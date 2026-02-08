# 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenHarmony 系统层                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ System Services │ │   Apps     │  │   Framework            │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                      HDF (Hardware Driver Foundation)            │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    device_board_hihope                      │  │
│  │  ┌───────────┐ ┌───────────┐ ┌─────────────────────────┐  │  │
│  │  │audio_driv │ │  camera   │ │        wifi            │  │  │
│  │  │  ers      │ │ vdi/v4l2  │ │    bcmdhd_wifi6        │  │  │
│  │  └───────────┘ └───────────┘ └─────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────┐  ┌──────────────────────┐           │
│  │   device/soc/rockchip │  │ device/soc/winnermicro│          │
│  │   (RK3568/RK3588)    │  │    (W800)            │           │
│  └──────────────────────┘  └──────────────────────┘           │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────┐  ┌──────────────────────┐           │
│  │   Linux Kernel       │  │     LiteOS-M         │           │
│  │   (RK3568/RK3588)    │  │     (W800/WS63)      │           │
│  └──────────────────────┘  └──────────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

---

## HDF 驱动架构

### 驱动模型

OpenHarmony 采用 **HDF (Hardware Driver Foundation)** 框架：

```c
// 驱动入口定义
struct HdfDriverEntry {
    uint32_t moduleVersion;
    const char* moduleName;
    int (*Bind)(struct HdfDeviceObject* device);
    int (*Init)(struct HdfDeviceObject* device);
    void (*Release)(struct HdfDeviceObject* device);
};
```

**证据**: `/rk3568/audio_drivers/codec/rk809_codec/src/rk809_codec_adapter.c`

### 音频驱动结构

```
audio_drivers/
├── codec/rk809_codec/           # 编解码器驱动
│   ├── rk809_codec_adapter.c   # HDF 入口 (HdfDriverEntry)
│   └── rk809_codec_impl.c      # 具体实现
├── dai/                         # 数字音频接口
│   └── rk3568_dai_adapter.c    # DAI HDF 入口
├── dsp/                         # DSP 驱动
│   └── rk3568_dsp_adapter.c
├── headset_monitor/             # 耳机监测
│   └── analog_headset_core.c
└── soc/                        # DMA
    └── rk3568_dma_adapter.c
```

**证据**: `/rk3568/audio_drivers/` 目录结构

### WiFi 驱动结构

```
wifi/bcmdhd_wifi6/hdfadapt/
├── hdf_driver_bdh_register.c   # WiFi HDF 驱动注册
├── hdf_bdh_mac80211.c          # MAC 层实现
├── net_bdh_adpater.c           # 网络适配
└── hdf_wl_interface.h          # WiFi 接口定义
```

**证据**: `/rk3568/wifi/bcmdhd_wifi6/hdfadapt/` 目录结构

---

## V4L2 相机架构

```
camera/vdi_impl/v4l2/
├── device_manager/              # 设备管理层
│   ├── include/
│   │   ├── imx600.h            # 传感器配置
│   │   ├── rkispv5.h           # ISP V5 接口 (RK3568)
│   │   ├── rkispv6.h           # ISP V6 接口 (RK3588)
│   │   └── project_hardware.h  # 硬件配置
│   └── src/
│       ├── rkispv5.cpp         # ISP V5 实现
│       └── camera_device_manager.cpp
├── pipeline_core/               # 流水线核心
│   └── src/node/
│       ├── rk_codec_node.cpp   # 编解码处理节点
│       ├── rk_face_node.cpp    # 人脸检测节点
│       └── rk_exif_node.cpp     # EXIF 处理节点
└── demo/                        # 演示程序
    └── ohos_camera_demo        # 相机演示
```

**证据**: `/rk3568/camera/vdi_impl/v4l2/` 目录结构

### Pipeline 数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Sensor    │ ──▶ │  ISP V5/V6  │ ──▶ │  Pipeline    │ ──▶ │   Framework │
│  (V4L2)     │     │  (Image Sig │     │  Nodes       │     │   (Camera   │
│             │     │  Processing)│     │  (Codec/Face │     │   Service)  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

---

## 线程模型

### 音频子系统

```
┌─────────────────────────────────────────────────────────────┐
│                    Audio Subsystem                           │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Audio HAL    │  │  Audio DSP   │  │ ALSA Interface    │  │
│  │ (HDF)        │  │  (Codec)     │  │ (vendor_render/  │  │
│  │              │  │              │  │  vendor_capture)  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### WiFi 子系统

```
┌─────────────────────────────────────────────────────────────┐
│                    WiFi Subsystem                            │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ BCM WiFi     │  │ HDF Adapt    │  │ Network Stack    │  │
│  │ Firmware     │──▶│ Layer      ──▶│  (TCP/IP)         │  │
│  │              │  │              │  │                  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 启动流程

### RK3568 / DAYU210 (Linux)

```
1. Bootloader (U-Boot)
   └── Loads: kernel + ramdisk + dtb

2. Linux Kernel
   └── Initializes: drivers, subsystems

3. init (stage 1)
   └── Parses: init.cfg

4. HDF Framework
   └── Loads: audio_drivers, camera, wifi

5. init (stage 2)
   └── Starts: system services

6. Boot Animation
   └── Displays: (if is_support_boot_animation=true)
```

**证据**: `/rk3568/cfg/init.rk3568.cfg`

### Neptune100 (LiteOS-M)

```
1. Bootloader (ROM)
   └── Loads: kernel image

2. LiteOS-M Kernel
   └── Initializes: kernel subsystems

3. HDF Framework
   └── Loads: shield drivers

4. System Services
   └── Starts: OpenHarmony services
```

---

## 数据流图

### 音频数据流

```
┌──────────────────────────────────────────────────────────────────┐
│                        Audio Data Flow                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Capture (录音):                                                  │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────────────┐  │
│  │ Mic HW  │──▶│ Codec   │──▶│ DAI     │──▶│ Audio HAL (HDF) │  │
│  │ (ADC)   │   │ RK809   │   │ DMA     │   │                 │  │
│  └─────────┘   └─────────┘   └─────────┘   └────────┬────────┘  │
│                                                     │            │
│                                                     ▼            │
│                                              ┌─────────────────┐ │
│                                              │ Audio Service   │ │
│                                              │ (上层应用)       │ │
│                                              └─────────────────┘ │
│                                                                   │
│  Render (播放):                                                   │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────────────┐  │
│  │ Audio   │──▶│ DAI     │──▶│ Codec   │──▶│ Speaker/HP     │  │
│  │ Service │   │ DMA     │   │ RK809   │   │ (DAC)           │  │
│  └─────────┘   └─────────┘   └─────────┘   └─────────────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 关键时序

### HDF 驱动加载时序

```
1. init 进程读取 HDF 配置文件 (.hcs)
2. HDF 框架加载驱动模块
3. 调用驱动入口函数:
   HdfDriverEntry.Bind()   // 绑定设备
   HdfDriverEntry.Init()    // 初始化资源
4. 驱动就绪，等待上层调用
```

---

## 相关文档

- [04_GN_Build](04_GN_Build.md) - 构建配置
- [05_Board_Configurations](05_Board_Configurations.md) - 开发板配置
- [06_Hardware_Drivers](06_Hardware_Drivers.md) - 硬件驱动详情
- [appendix/Callchains](appendix/Callchains.md) - 关键调用链
