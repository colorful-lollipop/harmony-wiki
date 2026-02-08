# 05_API_Differences.md - OH 扩展 API 分析

## 概述

OpenHarmony 通过独立的 `codec_omx_ext.h` 文件对 OpenMAX IL 标准进行扩展，而非修改原标准头文件。

### 设计原则

1. **向后兼容**：不修改标准头文件，确保与上游 OMX 实现兼容
2. **Vendor 区域**：使用 Khronos 保留的 Vendor Extension 区域
3. **独立演进**：扩展可以独立于上游版本迭代

### 文件位置

```
third_party/openmax/api/1.1.2/codec_omx_ext.h
```

## 扩展分类

### 1. 视频编码格式扩展

#### 1.1 视频编码类型 (CodecVideoExType)

```c
enum CodecVideoExType {
    CODEC_OMX_VIDEO_CodingVP9  = 10,        /** VP9 在 Codec HDI 中的索引 */
    CODEC_OMX_VIDEO_CodingHEVC = 11,        /** HEVC 在 Codec HDI 中的索引 */
    CODEC_OMX_VIDEO_CodingVVC = 0x7F000007, /** VVC 在 Codec HDI 中的索引 */
};
```

**背景**：标准 OMX 1.1.2 缺乏对 HEVC/H.265、VVC/H.266 的定义

**使用场景**：
- 4K/8K 视频编解码
- HDR 视频处理

#### 1.2 AVC Profile 扩展

```c
enum CodecAVCProfileExt {
    OMX_VIDEO_AVC_LEVEL52  = 0x10000,  /**< Level 5.2 */
    OMX_VIDEO_AVC_LEVEL6   = 0x20000,  /**< Level 6 */
    OMX_VIDEO_AVC_LEVEL61  = 0x40000,  /**< Level 6.1 */
    OMX_VIDEO_AVC_LEVEL62  = 0x80000,  /**< Level 6.2 */
};
```

**说明**：扩展 AVC/H.264 的 Level 支持到 6.2

#### 1.3 HEVC Profile/Level 完整定义

```c
enum CodecHevcProfile {
    CODEC_HEVC_PROFILE_INVALID = 0x0,
    CODEC_HEVC_PROFILE_MAIN = 0x1,
    CODEC_HEVC_PROFILE_MAIN10 = 0x2,
    CODEC_HEVC_PROFILE_MAIN_STILL = 0x3,
    CODEC_HEVC_PROFILE_MAIN10_HDR10 = 0x1000,      // HDR SEI 支持
    CODEC_HEVC_PROFILE_MAIN10_HDR10_PLUS = 0x2000, // HDR10+ 支持
    CODEC_HEVC_PROFILE_MAX = 0x7FFFFFFF
};

enum CodecHevcLevel {
    CODEC_HEVC_LEVEL_INVALID = 0x0,
    CODEC_HEVC_MAIN_TIER_LEVEL1 = 0x1,
    CODEC_HEVC_HIGH_TIER_LEVEL1 = 0x2,
    // ... Level 1 ~ 6.2 的 Main/High Tier
    CODEC_HEVC_HIGH_TIER_MAX = 0x7FFFFFFF
};
```

**特性**：
- 完整支持 HEVC Main/High Tier
- 支持 HDR10 和 HDR10+

#### 1.4 VVC Profile/Level 完整定义

```c
enum CodecVvcProfile {
    CODEC_VVC_PROFILE_INVALID = 0x0,
    CODEC_VVC_PROFILE_MAIN10 = 0x1,
    CODEC_VVC_PROFILE_MAIN10_STILL = 0x2,
    CODEC_VVC_PROFILE_MAIN10_444 = 0x3,
    CODEC_VVC_PROFILE_MAIN10_444_STILL = 0x4,
    CODEC_VVC_PROFILE_MULTI_MAIN10 = 0x5,
    CODEC_VVC_PROFILE_MULTI_MAIN10_444 = 0x6,
    // ... Operation range extensions profiles
    CODEC_VVC_PROFILE_MAX = 0x7FFFFFFF
};

enum CodecVvcLevel {
    // Level 1 ~ 6.3, 15.5 的 Main/High Tier
    CODEC_VVC_HIGH_TIER_MAX = 0x7FFFFFFF
};
```

**VVC (H.266)**：下一代视频编码标准，相比 HEVC 节省 50% 码率

### 2. 颜色格式扩展

```c
enum CodecColorFormatExt {
    CODEC_COLOR_FORMAT_RGBA8888 = OMX_COLOR_FormatVendorStartUnused + 100,
};
```

**用途**：支持 RGBA 8888 颜色格式的视频处理

### 3. 错误类型扩展

```c
enum CodecErrorTypeExt {
    /** 参数集非法 */
    OMX_ErrorParameterSetsIllegal = OMX_ErrorVendorStartUnused + 1,
    /** 参数集丢失 */
    OMX_ErrorParameterSetsLost,
};
```

**场景**：
- H.264/H.265 解码时 SPS/PPS 参数集错误
- 网络传输导致的数据丢失

### 4. Buffer 类型扩展

#### 4.1 支持的 Buffer 类型

```c
enum CodecBufferType {
    CODEC_BUFFER_TYPE_INVALID = 0,
    CODEC_BUFFER_TYPE_VIRTUAL_ADDR = 0x1,        /** 虚拟地址 */
    CODEC_BUFFER_TYPE_AVSHARE_MEM_FD = 0x2,      /** 共享内存 FD */
    CODEC_BUFFER_TYPE_HANDLE = 0x4,              /** Buffer Handle */
    CODEC_BUFFER_TYPE_DYNAMIC_HANDLE = 0x8,      /** 动态 Handle */
    CODEC_BUFFER_TYPE_DMA_MEM_FD = 0x10,         /** DMA 内存 FD */
};
```

**说明**：
- `VIRTUAL_ADDR`：传统内存地址
- `AVSHARE_MEM_FD`：跨进程共享内存
- `HANDLE`：Surface Buffer Handle
- `DMA_MEM_FD`：DMA 硬件直通内存

#### 4.2 Buffer 能力查询结构

```c
struct SupportBufferType {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint32_t bufferTypes;  /** 支持的 Buffer 类型位图 */
};

struct UseBufferType {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint32_t bufferType;   /** 当前使用的 Buffer 类型 */
};
```

**使用场景**：
- 查询硬件支持的 Buffer 类型
- 配置零拷贝 (Zero-Copy) 传输

#### 4.3 Buffer Handle 使用参数

```c
struct GetBufferHandleUsageParams {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint64_t usage;  /** Buffer 用途标志 */
};
```

### 5. 视频端口格式扩展

```c
struct CodecVideoPortFormatParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint32_t codecColorIndex;      /** 颜色格式索引 */
    uint32_t codecColorFormat;     /** Display 定义的颜色格式 */
    uint32_t codecCompressFormat;  /** 压缩格式 */
    uint32_t framerate;            /** Q16 格式帧率 */
};
```

**改进**：
- 与 Display 子系统统一颜色格式定义
- 支持更丰富的视频格式配置

### 6. 码控参数扩展

#### 6.1 恒定质量码控

```c
struct ControlRateConstantQuality {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint32_t qualityValue;  /** 质量值 */
};
```

**应用场景**：
- 屏幕录制（质量优先）
- 专业视频编辑

#### 6.2 目标 QP 码控

```c
struct ControlQualityTargetQp {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint32_t targetQp;  /** 目标 QP 值 */
};
```

#### 6.3 稳定码控（低时延场景）

```c
struct StableControlRate {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    uint32_t sqrFactor;
    uint32_t sMaxBitrate;
    uint32_t sTargetBitrate;
    bool bitrateEnabled;
};
```

**特性**：
- 无线低时延视频编码优化
- 动态码率调整

### 7. ROI (Region of Interest) 编码支持

```c
struct CodecRoiRect {
    uint32_t x;
    uint32_t y;
    uint32_t w;
    uint32_t h;
};

struct CodecRoiParmas {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t portIndex;
    int32_t quantizationLevel;
    struct CodecRoiRect roiRect[ROI_QUANTITY];  // 最多 6 个 ROI 区域
};
```

**应用**：
- 视频会议（对人脸区域提高质量）
- 监控（对关键区域增强编码）

### 8. 低时延模式 (LPP)

#### 8.1 LPP 模式开关

```c
struct GetLppModeParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    bool enable;  /**< LPP 使能 */
};
```

#### 8.2 LPP 目标 PTS

```c
struct LppTargetPtsParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    int64_t targetPts;  /** LPP 目标时间戳 */
};
```

**LPP (Low Power Performance)**：
- 低功耗高性能模式
- 用于实时音视频场景
- 降低端到端延迟

### 9. 直通参数 (Passthrough)

```c
struct PassthroughParam {
    int32_t key;    /**< 参数类型索引 */
    void *val;      /**< 参数值指针 */
    int size;       /**< 参数值大小 */
};
```

**用途**：
- 厂商特定参数透传
- 无需定义新结构即可传递自定义参数

### 10. 工作频率控制

```c
struct WorkingFrequencyParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t level;  /** 工作频率级别 */
};
```

**场景**：
- 性能模式：提高频率降低延迟
- 省电模式：降低频率延长续航

### 11. 进程名称传递

```c
#define PROCESS_NAME_LEN 50

struct ProcessNameParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    char processName[PROCESS_NAME_LEN];
};
```

**用途**：
- QoS (Quality of Service) 调度
- 性能统计
- 调试诊断

### 12. 音频编解码参数

```c
struct AudioCodecParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    uint32_t sampleRate;
    uint32_t sampleFormat;
    uint32_t channels;
    uint32_t bitRate;
    uint32_t reserved;
};
```

### 13. 通道 ID 获取

```c
struct GetChannelIdParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    int32_t channelId;  /**< 通道 ID */
};
```

### 14. 扩展索引定义 (OmxIndexCodecExType)

```c
enum OmxIndexCodecExType {
    OMX_IndexExtBufferTypeStartUnused = OMX_IndexKhronosExtensions + 0x00a00000,
    
    // Buffer 相关
    OMX_IndexParamSupportBufferType,
    OMX_IndexParamUseBufferType,
    OMX_IndexParamGetBufferHandleUsage,
    
    // 视频格式
    OMX_IndexCodecVideoPortFormat,
    
    // 码控相关
    OMX_IndexParamControlRateConstantQuality,
    OMX_IndexParamPassthrough,
    OMX_IndexParamStableControlRate,
    OMX_IndexParamControlQualityTargetQp,
    
    // 性能/功耗
    OMX_IndexParamWorkingFrequency,
    OMX_IndexParamLppMode,
    OMX_IndexParamLppTargetPts,
    
    // ROI 编码
    OMX_IndexParamRoi,
    
    // 其他
    OMX_IndexParamProcessName,
    OMX_IndexParamAudioCodec,
    OMX_IndexParamGetChannelId,
    OMX_IndexParamCodecExtendedBufferHandles,
};
```

**索引范围**：
- 使用 `OMX_IndexKhronosExtensions + 0x00a00000` 起始
- 避免与标准 Khronos 扩展冲突

## API 使用示例

### 示例 1：配置 HEVC 编码参数

```c
#include <OMX_Core.h>
#include <OMX_Component.h>
#include <codec_omx_ext.h>

OMX_HANDLETYPE handle;
OMX_GetHandle(&handle, "OMX.hevc.encoder", NULL, &callbacks);

// 配置 HEVC Profile
OMX_VIDEO_PARAM_PROFILELEVELTYPE profileLevel;
profileLevel.nSize = sizeof(profileLevel);
profileLevel.nVersion.nVersion = OMX_VERSION;
profileLevel.nPortIndex = 1;
profileLevel.eProfile = CODEC_HEVC_PROFILE_MAIN10;
profileLevel.eLevel = CODEC_HEVC_MAIN_TIER_LEVEL51;
OMX_SetParameter(handle, OMX_IndexParamVideoProfileLevelCurrent, &profileLevel);
```

### 示例 2：查询 Buffer 支持类型

```c
SupportBufferType supportBuffer;
supportBuffer.size = sizeof(supportBuffer);
supportBuffer.version.nVersion = OMX_VERSION;
supportBuffer.portIndex = 1;
OMX_GetParameter(handle, OMX_IndexParamSupportBufferType, &supportBuffer);

if (supportBuffer.bufferTypes & CODEC_BUFFER_TYPE_DMA_MEM_FD) {
    // 支持 DMA 零拷贝
}
```

### 示例 3：配置 ROI 编码

```c
CodecRoiParmas roiParams;
roiParams.size = sizeof(roiParams);
roiParams.version.nVersion = OMX_VERSION;
roiParams.portIndex = 1;
roiParams.quantizationLevel = -5;  // 提高质量
roiParams.roiRect[0] = (CodecRoiRect){100, 100, 200, 200};  // 人脸区域
OMX_SetParameter(handle, OMX_IndexParamRoi, &roiParams);
```

### 示例 4：配置低时延码控

```c
StableControlRate stableRate;
stableRate.size = sizeof(stableRate);
stableRate.version.nVersion = OMX_VERSION;
stableRate.portIndex = 1;
stableRate.sqrFactor = 10;
stableRate.sMaxBitrate = 4000000;
stableRate.sTargetBitrate = 2000000;
stableRate.bitrateEnabled = OMX_TRUE;
OMX_SetParameter(handle, OMX_IndexParamStableControlRate, &stableRate);
```

## 与标准 OMX API 对比

| 功能 | 标准 OMX | OH 扩展 | 说明 |
|------|---------|---------|------|
| HEVC 支持 | ❌ | ✅ codec_omx_ext.h | 完整 Profile/Level 定义 |
| VVC 支持 | ❌ | ✅ codec_omx_ext.h | 下一代编码标准 |
| DMA Buffer | ❌ | ✅ codec_omx_ext.h | 零拷贝支持 |
| ROI 编码 | ❌ | ✅ codec_omx_ext.h | 感兴趣区域编码 |
| 低时延码控 | ❌ | ✅ codec_omx_ext.h | 无线场景优化 |
| HDR10+ | ❌ | ✅ codec_omx_ext.h | HDR 视频支持 |
| AVC Level 6+ | ❌ | ✅ codec_omx_ext.h | 高分辨率支持 |

## 回归风险

### 上游版本升级影响

| 扩展类型 | 风险等级 | 说明 |
|----------|----------|------|
| Profile/Level 定义 | 低 | 与上游无冲突 |
| Buffer 类型 | 中 | 需确认新版本的 Buffer 管理 |
| 索引定义 | 低 | 使用 Vendor 保留区域 |
| 结构体 | 中 | 需检查 OMX_VERSIONTYPE 兼容性 |

### 建议

1. 升级上游版本时，同时保留 `codec_omx_ext.h`
2. 检查新版本是否已包含类似扩展，考虑废弃 OH 特有定义
3. 测试所有依赖模块的编译和运行
