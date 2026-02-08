# 附录：关键调用链

## 文档信息

- **目的**: 记录 Rockchip OpenHarmony 仓库的关键调用链，帮助理解代码执行流程
- **适用范围**: RK3568/RK3588 平台

## Display Composer 调用链

### 1. 初始化流程

```
CreateComposerVdi() [display_composer_vdi_impl.cpp:xx]
  └── DisplayComposerVdiImpl::GetVdiInstance()
      └── DisplayComposerVdiImpl::DisplayComposerVdiImpl()
          └── HdiSession::GetInstance()
              └── HdiSession::HdiSession()
                  ├── drmOpen()
                  ├── drmModeGetResources()
                  ├── DrmDevice::Create()
                  │   └── DrmDevice::DrmDevice()
                  │       ├── DiscoverDisplay()
                  │       └── HdiDisplay::Create()
                  └── HdiNetLinkMonitor::GetInstance()
```

### 2. 显示提交流程

```
Commit(devId, fence) [display_composer_vdi_impl.cpp:xx]
  └── HdiSession::CallDisplayFunction()
      └── HdiDisplay::Commit()
          └── HdiComposer::Prepare()
              ├── HdiComposition::SetLayers()
              │   └── HdiDrmComposition::SetLayers()
              │       └── DrmPlane::SetPlane()
              └── drmModeAtomicCommit()
                  └── ioctl(DRM_IOCTL_MODE_ATOMIC)
```

### 3. 图层设置流程

```
SetLayerBuffer(devId, layerId, buffer, fence) [display_composer_vdi_impl.cpp:xx]
  └── HdiSession::CallLayerFunction()
      └── HdiDisplay::SetLayerBuffer()
          └── HdiLayer::SetLayerBuffer()
              ├── HdiLayerBuffer::Create()
              │   └── drmPrimeFDToHandle()
              └── HdiDrmLayer::Init()
```

### 4. VSync 回调流程

```
DRM VBlank 中断
  └── drm_handle_vblank()
      └── VBlankCallback()
          └── HdiSession::VBlankEvent()
              └── VBlankCallback (注册回调)
                  └── 应用层回调处理
```

## Display Buffer 调用链

### 1. 缓冲区分配流程

```
CreateDisplayBufferVdi()
  └── DisplayBufferVdiImpl::DisplayBufferVdiImpl()

AllocMem(info, handle) [display_buffer_vdi_impl.cpp:xx]
  └── display_gralloc_gbm.cpp
      └── gbm_bo_create()
          ├── drmModeCreateDumbBuffer()
          └── drmPrimeHandleToFD()
```

### 2. 缓冲区映射流程

```
Mmap(handle) [display_buffer_vdi_impl.cpp:xx]
  └── display_gralloc_gbm.cpp
      └── gbm_bo_map()
          └── drmModeMapDumbBuffer()
              └── mmap()
```

## MPP 调用链

### 1. 初始化流程

```
mpp_create(ctx, mpi) [rk_mpi.h:229]
  └── mpp_create_impl()
      ├── MppCtxImpl::MppCtxImpl()
      └── MppApi::init()

mpp_init(ctx, type, coding) [rk_mpi.h:240]
  └── mpp_init_impl()
      ├── MppCtxImpl::init()
      │   ├── MppDecImpl::init() (解码)
      │   └── MppEncImpl::init() (编码)
      └── mpp_control_setup()

mpp_start(ctx) [rk_mpi.h:254]
  └── MppCtxImpl::start()
      └── 启动编解码线程
```

### 2. 解码流程

```
同步解码:
decode(ctx, packet, frame) [rk_mpi.h:93]
  └── MppDecImpl::decode()
      ├── 发送数据包
      │   └── put_packet()
      └── 接收解码帧
          └── get_frame()

异步解码:
decode_put_packet(ctx, packet) [rk_mpi.h:102]
  └── MppDecImpl::put_packet()
      └── MppThread::send()

decode_get_frame(ctx, frame) [rk_mpi.h:111]
  └── MppDecImpl::get_frame()
      └── MppThread::recv()
```

### 3. 编码流程

```
同步编码:
encode(ctx, frame, packet) [rk_mpi.h:122]
  └── MppEncImpl::encode()
      ├── 发送帧
      │   └── put_frame()
      └── 接收编码包
          └── get_packet()

异步编码:
encode_put_frame(ctx, frame) [rk_mpi.h:131]
  └── MppEncImpl::put_frame()
      └── MppThread::send()

encode_get_packet(ctx, packet) [rk_mpi.h:140]
  └── MppEncImpl::get_packet()
      └── MppThread::recv()
```

## Codec HDI 调用链

### 1. 组件创建流程

```
GetCodecJpegHwi() [codec_jpeg_impl.h:xx]
  └── CodecJpegDecoder::Init()
      ├── hdiMppCreate()
      │   └── mpp_create()
      ├── hdiMppInit()
      │   └── mpp_init()
      └── InitMppConfig()
          └── SetDefaultFps()
```

### 2. 解码流程

```
CodecJpegDecoder::Decode(inBuffer, outBuffer)
  └── hdiMppDecode()
      ├── hdiMppPacketInit()
      ├── hdiMppFrameInit()
      ├── hdiMppDecode()
      │   └── mpi->decode()
      └── hdiMppFrameDeinit()
```

## 内核驱动调用链

### 1. DRM 初始化流程

```
rockchip_drm_probe() [rockchip_drm_drv.c]
  ├── drm_dev_alloc()
  ├── rockchip_drm_create_properties()
  ├── rockchip_drm_load()
  │   ├── component_bind_all()
  │   │   ├── vop_component_bind()
  │   │   └── hdmi_component_bind()
  │   └── rockchip_drm_fbdev_init()
  └── drm_dev_register()
```

### 2. MPP 驱动流程

```
mpp_service_probe() [mpp_service.c]
  ├── mpp_service_create()
  ├── mpp_taskqueue_init()
  └── mpp_dev_init()

mpp_task_submit() [mpp_service.c]
  ├── mpp_taskqueue_push_task()
  ├── mpp_dev_ioctl()
  │   └── 硬件寄存器操作
  └── mpp_taskqueue_pop_task()
```

## HDF 驱动调用链 (RK2206)

### 1. GPIO 初始化流程

```
GpioDriverBind() [gpio_driver.c]
  └── HDF_INIT(gpio)

GpioDriverInit() [gpio_driver.c]
  ├── GpioGetConfig()
  │   └── 读取 HDF 配置
  ├── HAL_GPIO_INIT()
  │   └── 初始化 GPIO 硬件
  └── GpioRegisterOps()
      └── 注册 GPIO 操作接口
```

### 2. GPIO 操作流程

```
GpioRead() [gpio_driver.c]
  └── HAL_GPIO_ReadPin()
      └── 读取 GPIO 寄存器

GpioWrite() [gpio_driver.c]
  └── HAL_GPIO_WritePin()
      └── 写入 GPIO 寄存器
```

## 关键文件路径

### Display 相关

| 函数 | 文件路径 |
|------|----------|
| CreateComposerVdi | `rk3568/hardware/display/src/display_device/display_composer_vdi_impl.cpp` |
| HdiSession::GetInstance | `common/hardware/display/src/display_device/hdi_session.cpp` |
| drmModeAtomicCommit | `common/sdk_linux/drivers/gpu/drm/drm_atomic.c` |

### MPP 相关

| 函数 | 文件路径 |
|------|----------|
| mpp_create | `common/hardware/mpp/mpp/hdi_mpp/hdi_mpp_mpi.cpp` |
| mpp_init | `common/hardware/mpp/mpp/hdi_mpp/hdi_mpp_mpi.cpp` |
| mpp_service_probe | `rk3588/kernel/drivers/video/rockchip/mpp/mpp_service.c` |

### Codec 相关

| 函数 | 文件路径 |
|------|----------|
| GetCodecJpegHwi | `rk3568/hardware/codec/jpeg/src/codec_jpeg_impl.cpp` |
| hdiMppCreate | `rk3568/hardware/codec/src/hdi_mpp.c` |

## 相关链接

- [架构说明](../01_Architecture.md) - 系统架构详解
- [HDI/VDI 接口](../03_HDI_Interfaces.md) - 接口文档
- [目录结构](../02_Directory_Structure.md) - 代码组织方式
