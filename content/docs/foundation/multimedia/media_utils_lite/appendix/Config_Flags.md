# 配置参数与宏定义

## 概述

本文档列出了 `media_utils_lite` 中所有的配置参数、宏定义和编译开关，供开发者参考。

## 构建配置参数

### config.gni

```gn
# 文件: foundation/multimedia/media_utils_lite/config.gni

declare_args() {
  # 媒体直通模式开关
  # 用途: 启用后跳过某些媒体处理步骤
  # 默认值: false
  # 建议: 仅在特定硬件平台上启用
  enable_media_passthrough_mode = false
}
```

**证据来源**: `config.gni:16-18`

---

## 编译宏定义

### 日志相关宏

```cpp
// 文件: interfaces/kits/media_log.h

// 日志域
#define LOG_DOMAIN 0xD002B00

// 日志标签
#define LOG_TAG "MultiMedia"

// 调试模式判断
#ifndef OHOS_DEBUG
  // 发布模式宏
  #define DECORATOR_HILOG(op, fmt, args...) \
      do { op(LOG_CORE, fmt, ##args); } while (0)
#else
  // 调试模式宏 (输出函数名、文件名、行号)
  #define DECORATOR_HILOG(op, fmt, args...) \
      do { op(LOG_CORE, "{%s()-%s:%d} " fmt, __FUNCTION__, __FILENAME__, __LINE__, ##args); } while (0)
#endif

// 日志级别宏
#define MEDIA_DEBUG_LOG(fmt, ...)   DECORATOR_HILOG(HILOG_DEBUG, fmt, ##__VA_ARGS__)
#define MEDIA_ERR_LOG(fmt, ...)     DECORATOR_HILOG(HILOG_ERROR, fmt, ##__VA_ARGS__)
#define MEDIA_WARNING_LOG(fmt, ...) DECORATOR_HILOG(HILOG_WARN, fmt, ##__VA_ARGS__)
#define MEDIA_INFO_LOG(fmt, ...)    DECORATOR_HILOG(HILOG_INFO, fmt, ##__VA_ARGS__)
#define MEDIA_FATAL_LOG(fmt, ...)   DECORATOR_HILOG(HILOG_FATAL, fmt, ##__VA_ARGS__)

// 返回值宏
#define MEDIA_OK                  0
#define MEDIA_INVALID_PARAM       (-1)
#define MEDIA_INIT_FAIL           (-2)
#define MEDIA_ERR                 (-3)
#define MEDIA_PERMISSION_DENIED   (-4)
#define MEDIA_IPC_FAILED          (-5)
```

**证据来源**: `interfaces/kits/media_log.h:22-52`

### 错误码宏

```cpp
// 文件: interfaces/kits/media_errors.h

// 模块和子系统标识
constexpr int MODULE_MEDIA = 1;
constexpr int SUBSYS_MEDIA = 30;

// 位域配置
constexpr int SUBSYSTEM_BIT_NUM = 21;
constexpr int MODULE_BIT_NUM = 16;

// 错误码生成
constexpr ErrCode ErrCodeOffset(unsigned int subsystem, unsigned int module = 0)
{
    return (subsystem << SUBSYSTEM_BIT_NUM) | (module << MODULE_BIT_NUM);
}

// 错误码基值
constexpr int32_t BASE_MEDIA_ERR_OFFSET = ErrCodeOffset(SUBSYS_MEDIA, MODULE_MEDIA);
// 结果: 0x3C10000
```

**证据来源**: `interfaces/kits/media_errors.h:44-65`

### HAL 宏

```c
// 文件: hals/hal_media.h

// 处理器句柄常量
typedef int32_t HalProcessorHdl;
#define HAL_INVALID_PROCESSOR (-1)
#define HAL_MAX_VPSS_NUM 10

// 返回值常量
#define HAL_MEDIA_OK 0
#define HAL_MEDIA_ERR 1
```

**证据来源**: `hals/hal_media.h:27-32`

### 相机 HAL 宏

```c
// 文件: hals/hal_camera.h

// 常量定义
#define CAMERA_FPS_MAX_NUM        16   // 最大帧率数量
#define CAMERA_DESC_MAX_LEN       32   // 描述最大长度
#define INFO_MAX_LEN           1024   // 信息最大长度
#define DESC_MAX_LEN             64   // 描述最大长度
#define AUTO_MODE_MAX_NUM        16   // 自动模式最大数量
#define PRIVATE_META_MAX_LEN     32   // 私有元数据最大长度
```

**证据来源**: `hals/hal_camera.h:27-32`

---

## 条件编译开关

### SURFACE_DISABLED

```cpp
// 用途: 在无 Surface 环境下禁用 Surface 相关代码
// 定义位置: BUILD.gn 的 config "media_common_public_config"
// 默认定义: 当 ohos_kernel_type == "liteos_m" 时定义

// 使用示例:
#ifndef SURFACE_DISABLED
#include "surface.h"
#endif

class StreamSource {
#ifndef SURFACE_DISABLED
    void SetSurface(Surface* surface);
    Surface* GetSurface();
    Surface* surface_;
    SurfaceBuffer* curBuffer_;
#endif
};
```

**证据来源**: `BUILD.gn:51-53`, `interfaces/kits/source.h:44-46`

### OHOS_DEBUG

```cpp
// 用途: 区分调试模式和发布模式
// 影响: 日志输出格式 (调试模式包含函数名、文件名、行号)

// 检查方式:
#ifndef OHOS_DEBUG
  // 发布模式
#else
  // 调试模式
#endif
```

**证据来源**: `interfaces/kits/media_log.h:29-39`

---

## BUILD.gn 配置参数

### ohos_kernel_type

```gn
// 用途: 指定目标内核类型
// 可选值: "liteos_m", "linux" 等
// 影响: 选择静态库还是动态库、选择不同的依赖

if (ohos_kernel_type == "liteos_m") {
    target_type = "static_library"
} else {
    target_type = "shared_library"
}
```

**证据来源**: `BUILD.gn:17-21`

### board_name

```gn
// 用途: 指定开发板名称
// 可选值: "hispark_taurus", "hispark_aries" 等
// 影响: 是否包含特定硬件适配依赖

if (board_name == "hispark_taurus" || board_name == "hispark_aries") {
    public_deps += [
        "$ohos_board_adapter_dir/media:hardware_media_sdk",
        "$ohos_board_adapter_dir/middleware:middleware_source_sdk",
    ]
}
```

**证据来源**: `BUILD.gn:42-47`

---

## Format 键值定义

### 预定义键

```cpp
// 文件: interfaces/kits/format.h

// 编解码 MIME 类型键
extern const char *CODEC_MIME;           // "mime"
extern const char *MIME_AUDIO_AAC;       // "audio/mp4a-latm"
extern const char *MIME_AUDIO_RAW;      // "audio/raw"
extern const char *PAUSE_AFTER_PLAY;    // "pause_after_play"
```

**证据来源**: `interfaces/kits/format.h:47-55`

### FormatDataType 枚举

```cpp
enum FormatDataType : uint32_t {
    FORMAT_TYPE_NONE = 0,
    FORMAT_TYPE_INT32,
    FORMAT_TYPE_INT64,
    FORMAT_TYPE_FLOAT,
    FORMAT_TYPE_DOUBLE,
    FORMAT_TYPE_STRING
};
```

**证据来源**: `interfaces/kits/format.h:63-76`

---

## StreamCallback 标志位

```cpp
// 文件: interfaces/kits/source.h

enum BufferFlags : uint32_t {
    STREAM_FLAG_SYNCFRAME = 1,        // 0x00000001 - 同步帧
    STREAM_FLAG_CODECCONFIG = 2,      // 0x00000002 - 编解码配置
    STREAM_FLAG_EOS = 4,               // 0x00000004 - 流结束
    STREAM_FLAG_PARTIAL_FRAME = 8,    // 0x00000008 - 帧的一部分
    STREAM_FLAG_ENDOFFRAME = 16,      // 0x00000010 - 帧结束
    STREAM_FLAG_MUXER_DATA = 32,      // 0x00000020 - 容器数据
};
```

**证据来源**: `interfaces/kits/source.h:83-96`

---

## 码率模式常量

```cpp
// 文件: interfaces/kits/media_info.h

const int BITRATE_MODE_CQ  = 0;   // 恒定质量模式
const int BITRATE_MODE_VBR = 1;   // 可变比特率模式
const int BITRATE_MODE_CBR = 2;   // 恒定比特率模式
```

**证据来源**: `interfaces/kits/media_info.h:45-55`

---

## 颜色格式常量

```cpp
// 文件: interfaces/kits/media_info.h

const int32_t COLOR_FORMAT_ARGB8888_32BIT = 16;   // ARGB8888 32位
const int32_t COLOR_FORMAT_YUV420SP = 21;        // YUV420SP
```

**证据来源**: `interfaces/kits/media_info.h:58-61`

---

## 内存类型枚举

```cpp
// 文件: interfaces/kits/data_stream.h

enum class MemoryType {
    VIRTUAL_ADDR = 0,    // 0 - 虚拟地址
    SURFACE_BUFFER,     // 1 - Surface 缓冲区
    SHARE_MEMORY,       // 2 - 共享内存 fd
};
```

**证据来源**: `interfaces/kits/data_stream.h:29-33`

---

## 配置参数速查表

| 参数/宏 | 类型 | 默认值 | 位置 | 用途 |
|---------|------|--------|------|------|
| `enable_media_passthrough_mode` | bool | false | config.gni | 媒体直通模式 |
| `LOG_DOMAIN` | uint32_t | 0xD002B00 | media_log.h | 日志域 |
| `LOG_TAG` | string | "MultiMedia" | media_log.h | 日志标签 |
| `SUBSYS_MEDIA` | int | 30 | media_errors.h | 子系统标识 |
| `MODULE_MEDIA` | int | 1 | media_errors.h | 模块标识 |
| `CAMERA_FPS_MAX_NUM` | int | 16 | hal_camera.h | 最大帧率数 |
| `HAL_MAX_VPSS_NUM` | int | 10 | hal_media.h | 最大 VPSS 数 |
| `SURFACE_DISABLED` | 宏 | 条件定义 | BUILD.gn | Surface 开关 |
| `OHOS_DEBUG` | 宏 | 条件定义 | media_log.h | 调试模式开关 |
