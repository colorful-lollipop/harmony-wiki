# HDI/VDI 接口文档

## 文档信息

- **目的**: 详细说明 Rockchip OpenHarmony 仓库的对外 HDI/VDI 接口
- **适用范围**: RK3566/RK3568/RK3588 平台（Linux 内核）
- **关键结论**: 本仓库通过 VDI 接口实现 OpenHarmony HDI 标准，无 N-API 接口

## 接口概述

### 接口架构

本仓库作为 SoC 硬件适配层，通过 **VDI (Vendor Driver Interface)** 实现 **HDI (Hardware Driver Interface)** 标准接口：

```
┌─────────────────────────────────────────────────────────────┐
│              OpenHarmony HDI 标准接口                        │
│         (定义于 drivers/peripheral 仓库)                    │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │ IDisplayComposer│  │ IDisplayBuffer  │                  │
│  │     Vdi         │  │     Vdi         │                  │
│  └────────┬────────┘  └────────┬────────┘                  │
└───────────┼────────────────────┼────────────────────────────┘
            │                    │
┌───────────┼────────────────────┼────────────────────────────┐
│           │     VDI 实现层      │                            │
│  ┌────────┴────────┐  ┌────────┴────────┐                   │
│  │ DisplayComposer │  │ DisplayBuffer   │                   │
│  │   VdiImpl       │  │   VdiImpl       │                   │
│  │   (本仓库)       │  │   (本仓库)       │                   │
│  └─────────────────┘  └─────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

### 接口类型

| 接口类别 | 接口名称 | 说明 | 位置 |
|----------|----------|------|------|
| Display | IDisplayComposerVdi | 显示合成器 | `*/hardware/display/` |
| Display | IDisplayBufferVdi | 显示缓冲区 | `*/hardware/display/` |
| Codec | ICodecComponent | 编解码器 | `*/hardware/codec/` |
| MPP | RKMppApi | MPP 媒体接口 | `*/hardware/mpp/` |

## Display Composer VDI 接口

### 接口定义

**文件**: `rk3568/hardware/display/src/display_device/display_composer_vdi_impl.h`

```cpp
namespace OHOS {
namespace HDI {
namespace DISPLAY {

class DisplayComposerVdiImpl : public IDisplayComposerVdi {
public:
    static DisplayComposerVdiImpl& GetVdiInstance();
    
    // 显示设备管理
    virtual int32_t RegHotPlugCallback(HotPlugCallback cb, void* data) override;
    virtual int32_t GetDisplayCapability(uint32_t devId, DisplayCapability& info) override;
    virtual int32_t GetDisplaySupportedModes(uint32_t devId, std::vector<DisplayModeInfo>& modes) override;
    virtual int32_t GetDisplayMode(uint32_t devId, uint32_t& modeId) override;
    virtual int32_t SetDisplayMode(uint32_t devId, uint32_t modeId) override;
    
    // 电源管理
    virtual int32_t GetDisplayPowerStatus(uint32_t devId, DispPowerStatus& status) override;
    virtual int32_t SetDisplayPowerStatus(uint32_t devId, DispPowerStatus status) override;
    
    // 背光控制
    virtual int32_t GetDisplayBacklight(uint32_t devId, uint32_t& level) override;
    virtual int32_t SetDisplayBacklight(uint32_t devId, uint32_t level) override;
    
    // VSync 控制
    virtual int32_t SetDisplayVsyncEnabled(uint32_t devId, bool enabled) override;
    virtual int32_t RegDisplayVBlankCallback(uint32_t devId, VBlankCallback cb, void* data) override;
    
    // 图层管理
    virtual int32_t CreateLayer(uint32_t devId, const LayerInfo& layerInfo, uint32_t& layerId) override;
    virtual int32_t DestroyLayer(uint32_t devId, uint32_t layerId) override;
    virtual int32_t PrepareDisplayLayers(uint32_t devId, bool& needFlushFb) override;
    
    // 图层属性
    virtual int32_t SetLayerAlpha(uint32_t devId, uint32_t layerId, const LayerAlpha& alpha) override;
    virtual int32_t SetLayerRegion(uint32_t devId, uint32_t layerId, const IRect& rect) override;
    virtual int32_t SetLayerCrop(uint32_t devId, uint32_t layerId, const IRect& rect) override;
    virtual int32_t SetLayerZorder(uint32_t devId, uint32_t layerId, uint32_t zorder) override;
    virtual int32_t SetLayerTransformMode(uint32_t devId, uint32_t layerId, TransformType type) override;
    virtual int32_t SetLayerBuffer(uint32_t devId, uint32_t layerId, 
                                   const BufferHandle& buffer, int32_t fence) override;
    virtual int32_t SetLayerCompositionType(uint32_t devId, uint32_t layerId, CompositionType type) override;
    virtual int32_t SetLayerBlendType(uint32_t devId, uint32_t layerId, BlendType type) override;
    
    // 显示提交
    virtual int32_t Commit(uint32_t devId, int32_t& fence) override;
};

} // DISPLAY
} // HDI
} // OHOS
```

### 工厂函数

**文件**: `rk3568/hardware/display/src/display_device/display_composer_vdi_impl.cpp`

```cpp
// VDI 工厂函数 - 创建 Composer VDI 实例
extern "C" IDisplayComposerVdi* CreateComposerVdi() {
    return &OHOS::HDI::DISPLAY::DisplayComposerVdiImpl::GetVdiInstance();
}

// VDI 工厂函数 - 销毁 Composer VDI 实例
extern "C" void DestroyComposerVdi(IDisplayComposerVdi* vdi) {
    // 单例模式，无需显式销毁
}
```

### API 清单

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| RegHotPlugCallback | cb, data | int32_t | 注册热插拔回调 |
| GetDisplayCapability | devId, info | int32_t | 获取显示能力 |
| GetDisplaySupportedModes | devId, modes | int32_t | 获取支持的模式列表 |
| GetDisplayMode | devId, modeId | int32_t | 获取当前显示模式 |
| SetDisplayMode | devId, modeId | int32_t | 设置显示模式 |
| GetDisplayPowerStatus | devId, status | int32_t | 获取电源状态 |
| SetDisplayPowerStatus | devId, status | int32_t | 设置电源状态 |
| GetDisplayBacklight | devId, level | int32_t | 获取背光亮度 |
| SetDisplayBacklight | devId, level | int32_t | 设置背光亮度 |
| SetDisplayVsyncEnabled | devId, enabled | int32_t | 启用/禁用 VSync |
| RegDisplayVBlankCallback | devId, cb, data | int32_t | 注册 VBlank 回调 |
| CreateLayer | devId, layerInfo, layerId | int32_t | 创建图层 |
| DestroyLayer | devId, layerId | int32_t | 销毁图层 |
| PrepareDisplayLayers | devId, needFlushFb | int32_t | 准备显示图层 |
| SetLayerAlpha | devId, layerId, alpha | int32_t | 设置图层透明度 |
| SetLayerRegion | devId, layerId, rect | int32_t | 设置图层区域 |
| SetLayerCrop | devId, layerId, rect | int32_t | 设置图层裁剪 |
| SetLayerZorder | devId, layerId, zorder | int32_t | 设置图层 Z 序 |
| SetLayerTransformMode | devId, layerId, type | int32_t | 设置变换模式 |
| SetLayerBuffer | devId, layerId, buffer, fence | int32_t | 设置图层缓冲区 |
| SetLayerCompositionType | devId, layerId, type | int32_t | 设置合成类型 |
| SetLayerBlendType | devId, layerId, type | int32_t | 设置混合类型 |
| Commit | devId, fence | int32_t | 提交显示帧 |

### 调用链

```
应用层
  ↓
OHOS::HDI::Display::Composer::V1_0::IDisplayComposer (HDI 接口)
  ↓
CreateComposerVdi() → DisplayComposerVdiImpl (VDI 实现)
  ↓
HDI::DISPLAY::HdiSession::GetInstance()
  ↓
HDI::DISPLAY::HdiDisplay → DRM 设备
  ↓
drmModeAtomicCommit() → 内核 DRM 驱动
```

## Display Buffer VDI 接口

### 接口定义

**文件**: `rk3568/hardware/display/src/display_gralloc/display_buffer_vdi_impl.h`

```cpp
namespace OHOS {
namespace HDI {
namespace DISPLAY {

class DisplayBufferVdiImpl : public IDisplayBufferVdi {
public:
    DisplayBufferVdiImpl();
    virtual ~DisplayBufferVdiImpl();

    // 内存分配
    virtual int32_t AllocMem(const AllocInfo& info, BufferHandle*& handle) const override;
    virtual void FreeMem(const BufferHandle& handle) const override;
    
    // 内存映射
    virtual void *Mmap(const BufferHandle& handle) const override;
    virtual int32_t Unmap(const BufferHandle& handle) const override;
    
    // 缓存管理
    virtual int32_t FlushCache(const BufferHandle& handle) const override;
    virtual int32_t InvalidateCache(const BufferHandle& handle) const override;
    
    // 分配验证
    virtual int32_t IsSupportedAlloc(const std::vector<VerifyAllocInfo>& infos,
                                     std::vector<bool>& supporteds) const override;
    
    // 缓冲区注册
    virtual int32_t RegisterBuffer(const BufferHandle& handle) override;
    
    // 元数据管理
    virtual int32_t SetMetadata(const BufferHandle& handle, uint32_t key, 
                                const std::vector<uint8_t>& value) override;
    virtual int32_t GetMetadata(const BufferHandle& handle, uint32_t key, 
                                std::vector<uint8_t>& value) override;
    virtual int32_t ListMetadataKeys(const BufferHandle& handle, std::vector<uint32_t>& keys) override;
    virtual int32_t EraseMetadataKey(const BufferHandle& handle, uint32_t key) override;
    
    // 图像布局
    virtual int32_t GetImageLayout(const BufferHandle& handle, 
                                   Display::Buffer::V1_2::ImageLayout& layout) const override;
};

} // DISPLAY
} // HDI
} // OHOS
```

### 工厂函数

```cpp
// VDI 工厂函数 - 创建 Buffer VDI 实例
extern "C" IDisplayBufferVdi* CreateDisplayBufferVdi() {
    return new DisplayBufferVdiImpl();
}

// VDI 工厂函数 - 销毁 Buffer VDI 实例
extern "C" void DestroyDisplayBufferVdi(IDisplayBufferVdi* vdi) {
    delete vdi;
}
```

### API 清单

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| AllocMem | info, handle | int32_t | 分配内存 |
| FreeMem | handle | void | 释放内存 |
| Mmap | handle | void* | 内存映射 |
| Unmap | handle | int32_t | 解除映射 |
| FlushCache | handle | int32_t | 刷新缓存 |
| InvalidateCache | handle | int32_t | 使缓存失效 |
| IsSupportedAlloc | infos, supporteds | int32_t | 验证分配支持 |
| RegisterBuffer | handle | int32_t | 注册缓冲区 |
| SetMetadata | handle, key, value | int32_t | 设置元数据 |
| GetMetadata | handle, key, value | int32_t | 获取元数据 |
| ListMetadataKeys | handle, keys | int32_t | 列出元数据键 |
| EraseMetadataKey | handle, key | int32_t | 删除元数据 |
| GetImageLayout | handle, layout | int32_t | 获取图像布局 |

## MPP (Media Process Platform) 接口

### 接口定义

**文件**: `common/hardware/mpp/include/rk_mpi.h`

```c
typedef struct MppApi_t {
    unsigned int size;
    unsigned int version;

    // 简单数据流接口 - 解码
    MPP_RET (*decode)(MppCtx ctx, MppPacket packet, MppFrame *frame);
    MPP_RET (*decode_put_packet)(MppCtx ctx, MppPacket packet);
    MPP_RET (*decode_get_frame)(MppCtx ctx, MppFrame *frame);
    
    // 简单数据流接口 - 编码
    MPP_RET (*encode)(MppCtx ctx, MppFrame frame, MppPacket *packet);
    MPP_RET (*encode_put_frame)(MppCtx ctx, MppFrame frame);
    MPP_RET (*encode_get_packet)(MppCtx ctx, MppPacket *packet);
    
    // 高级任务接口
    MPP_RET (*poll)(MppCtx ctx, MppPortType type, MppPollType timeout);
    MPP_RET (*dequeue)(MppCtx ctx, MppPortType type, MppTask *task);
    MPP_RET (*enqueue)(MppCtx ctx, MppPortType type, MppTask task);
    
    // 控制接口
    MPP_RET (*reset)(MppCtx ctx);
    MPP_RET (*control)(MppCtx ctx, MpiCmd cmd, MppParam param);
} MppApi;

// 上下文管理
MPP_RET mpp_create(MppCtx *ctx, MppApi **mpi);
MPP_RET mpp_init(MppCtx ctx, MppCtxType type, MppCodingType coding);
MPP_RET mpp_start(MppCtx ctx);
MPP_RET mpp_stop(MppCtx ctx);
MPP_RET mpp_destroy(MppCtx ctx);
```

### API 清单

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| mpp_create | ctx, mpi | MPP_RET | 创建 MPP 上下文 |
| mpp_init | ctx, type, coding | MPP_RET | 初始化 MPP |
| mpp_start | ctx | MPP_RET | 启动 MPP |
| mpp_stop | ctx | MPP_RET | 停止 MPP |
| mpp_destroy | ctx | MPP_RET | 销毁 MPP |
| decode | ctx, packet, frame | MPP_RET | 同步解码 |
| decode_put_packet | ctx, packet | MPP_RET | 发送解码包 |
| decode_get_frame | ctx, frame | MPP_RET | 获取解码帧 |
| encode | ctx, frame, packet | MPP_RET | 同步编码 |
| encode_put_frame | ctx, frame | MPP_RET | 发送编码帧 |
| encode_get_packet | ctx, packet | MPP_RET | 获取编码包 |
| poll | ctx, type, timeout | MPP_RET | 轮询端口 |
| dequeue | ctx, type, task | MPP_RET | 出队任务 |
| enqueue | ctx, type, task | MPP_RET | 入队任务 |
| reset | ctx | MPP_RET | 重置 MPP |
| control | ctx, cmd, param | MPP_RET | 控制命令 |

### 支持的编解码格式

| 类型 | 格式 | 说明 |
|------|------|------|
| 解码 | H.264 | 视频解码 |
| 解码 | H.265/HEVC | 视频解码 |
| 解码 | VP9 | 视频解码 |
| 解码 | AV1 | 视频解码 (RK3588) |
| 编码 | H.264 | 视频编码 |
| 编码 | H.265/HEVC | 视频编码 |

## Codec HDI 接口

### 接口定义

**文件**: `rk3568/hardware/codec/include/hdi_mpp.h`

```c
// 编解码器组件结构
typedef struct {
    CODEC_HANDLETYPE handle;
    RKHdiCodecMimeSetup setup;
    RKHdiEncodeSetup encodeParam;
    // ... 其他成员
} RKHdiBaseComponent;

// 组件管理接口
int32_t InitMppConfig(RKHdiBaseComponent *pBaseComponent);
int32_t DeinitMppConfig(RKHdiBaseComponent *pBaseComponent);
int32_t SetEncCfg(RKHdiBaseComponent *pBaseComponent);
int32_t SetDecCfg(RKHdiBaseComponent *pBaseComponent);
```

### JPEG 解码接口

**文件**: `rk3568/hardware/codec/jpeg/include/codec_jpeg_decoder.h`

```cpp
class CodecJpegDecoder {
public:
    int32_t Init();
    int32_t Deinit();
    int32_t Decode(const BufferHandle& inBuffer, const BufferHandle& outBuffer);
    int32_t GetImageInfo(const BufferHandle& buffer, ImageInfo& info);
};
```

## 错误码定义

### Display HDI 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| DISPLAY_SUCCESS | 0 | 成功 |
| DISPLAY_FAILURE | -1 | 失败 |
| DISPLAY_EINVAL | -2 | 无效参数 |
| DISPLAY_ENOMEM | -3 | 内存不足 |
| DISPLAY_EIO | -4 | IO 错误 |

### MPP 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| MPP_OK | 0 | 成功 |
| MPP_ERR_UNKNOW | -1 | 未知错误 |
| MPP_ERR_NULL_PTR | -2 | 空指针 |
| MPP_ERR_MALLOC | -3 | 内存分配失败 |
| MPP_ERR_OPEN_CODEC | -4 | 打开编解码器失败 |
| MPP_ERR_TIMEOUT | -5 | 超时 |

## 权限与前置条件

### Display VDI 权限

| 操作 | 权限要求 | 说明 |
|------|----------|------|
| AllocMem | 无 | 分配图形内存 |
| SetDisplayMode | system | 修改显示模式需要系统权限 |
| SetDisplayPowerStatus | system | 电源管理需要系统权限 |
| RegHotPlugCallback | 无 | 注册热插拔回调 |

### MPP 权限

| 操作 | 权限要求 | 说明 |
|------|----------|------|
| mpp_create | 无 | 创建上下文 |
| 访问 /dev/mpp_service | system | 访问 MPP 设备节点需要系统权限 |

## 相关链接

- [架构说明](01_Architecture.md) - 系统架构详解
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [GN 构建系统](04_GN_Build.md) - 构建配置
