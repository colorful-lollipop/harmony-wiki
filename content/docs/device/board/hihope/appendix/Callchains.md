# 关键调用链

## 音频驱动调用链

### 录音路径

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Audio Capture Path                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Application (OpenHarmony)                                              │
│         │                                                               │
│         ▼                                                               │
│  Audio Service                                                          │
│         │                                                               │
│         ▼                                                               │
│  Audio Framework (HDF)                                                  │
│         │                                                               │
│         ▼                                                               │
│  ┌───────────────────────────────────────────────────────────────┐      │
│  │                    audio_drivers                              │      │
│  │  ├─ dai/ (rk3568_dai_adapter.c)                              │      │
│  │  ├─ codec/ (rk809_codec_adapter.c)  ◄── Codec 操作          │      │
│  │  └─ soc/ (rk3568_dai_adapter.c)                             │      │
│  └───────────────────────────────────────────────────────────────┘      │
│         │                                                               │
│         ▼                                                               │
│  ALSA Interface (vendor_capture.c)                                      │
│         │                                                               │
│         ▼                                                               │
│  Hardware (Microphone/ADC)                                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 播放路径

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Audio Render Path                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Application (OpenHarmony)                                              │
│         │                                                               │
│         ▼                                                               │
│  Audio Service                                                          │
│         │                                                               │
│         ▼                                                               │
│  Audio Framework (HDF)                                                  │
│         │                                                               │
│         ▼                                                               │
│  ┌───────────────────────────────────────────────────────────────┐      │
│  │                    audio_drivers                              │      │
│  │  ├─ dai/ (rk3568_dai_adapter.c)                             │      │
│  │  ├─ codec/ (rk809_codec_adapter.c)  ◄── Codec 操作          │      │
│  │  └─ soc/ (dma_adapter.c)                                    │      │
│  └───────────────────────────────────────────────────────────────┘      │
│         │                                                               │
│         ▼                                                               │
│  ALSA Interface (vendor_render.c)                                       │
│         │                                                               │
│         ▼                                                               │
│  Hardware (Speaker/DAC)                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 相机驱动调用链

### 预览路径

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Camera Preview Path                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Camera Application                                                     │
│         │                                                               │
│         ▼                                                               │
│  Camera Service (OHOS::Camera)                                         │
│         │                                                               │
│         ▼                                                               │
│  ┌───────────────────────────────────────────────────────────────┐      │
│  │              camera/vdi_impl/v4l2/                            │      │
│  │  ├─ device_manager/ (camera_device_manager.cpp)             │      │
│  │  │       │                                                    │      │
│  │  │       ▼                                                    │      │
│  │  │  ISP V5/V6 (rkispv5.cpp/rkispv6.cpp)                      │      │
│  │  └─ pipeline_core/ (node/)                                  │      │
│  │         │                                                    │      │
│  │         ▼                                                    │      │
│  │    ┌─────────────────────────────────────────┐               │      │
│  │    │  rk_codec_node.cpp (JPEG 编码)         │               │      │
│  │    │  rk_face_node.cpp (人脸检测)            │               │      │
│  │    │  rk_exif_node.cpp (EXIF 处理)          │               │      │
│  │    └─────────────────────────────────────────┘               │      │
│  └───────────────────────────────────────────────────────────────┘      │
│         │                                                               │
│         ▼                                                               │
│  V4L2 Kernel Driver                                                    │
│         │                                                               │
│         ▼                                                               │
│  Sensor Hardware (IMX600)                                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Demo 程序调用

```
demo/ohos_camera_demo
        │
        ▼
Camera Service APIs
        │
        ▼
camera/vdi_impl/v4l2/ (VDI 实现)
        │
        ▼
ISP + Pipeline Nodes
```

---

## WiFi 驱动调用链

### 网络发送路径

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       WiFi Tx Path                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Application (Network Stack)                                            │
│         │                                                               │
│         ▼                                                               │
│  Linux Network Stack                                                    │
│         │                                                               │
│         ▼                                                               │
│  ┌───────────────────────────────────────────────────────────────┐      │
│  │         wifi/bcmdhd_wifi6/hdfadapt/                           │      │
│  │  ├─ net_bdh_adpater.c (网络适配)                             │      │
│  │  ├─ hdf_bdh_mac80211.c (MAC 层)                              │      │
│  │  └─ hdf_driver_bdh_register.c (驱动注册)                     │      │
│  └───────────────────────────────────────────────────────────────┘      │
│         │                                                               │
│         ▼                                                               │
│  BCM WiFi Firmware                                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## HDF 驱动初始化调用链

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HDF Driver Initialization                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  init 进程                                                              │
│         │                                                               │
│         ▼                                                               │
│  HDF Framework                                                          │
│         │                                                               │
│         ▼                                                               │
│  读取 .hcs 配置                                                         │
│  (neptune100.hcs / init.*.cfg)                                         │
│         │                                                               │
│         ▼                                                               │
│  加载驱动模块                                                           │
│         │                                                               │
│         ▼                                                               │
│  ┌───────────────────────────────────────────────────────────────┐      │
│  │  For each driver:                                           │      │
│  │  ├─ HdfDriverEntry.Bind()   // 绑定设备                    │      │
│  │  ├─ HdfDriverEntry.Init()   // 初始化资源                  │      │
│  │  └─ HdfDriverEntry.Release() // 释放资源                   │      │
│  └───────────────────────────────────────────────────────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 启动调用链

### Linux 系统启动 (RK3568/DAYU210)

```
1. Bootloader (U-Boot)
   │
   ├─ Loads: kernel image
   ├─ Loads: device tree
   └─ Loads: ramdisk

2. Linux Kernel
   │
   ├─ Initializes: core subsystems
   ├─ Initializes: device drivers
   └─ Starts: init process

3. init (Stage 1)
   │
   └─ Parses: init.rk3568.cfg / init.dayu210.cfg
              │
              ├─ Loads: HDF drivers
              ├─ Mounts: filesystems
              └─ Starts: services

4. HDF Framework
   │
   └─ Loads: audio_drivers, camera, wifi

5. init (Stage 2)
   │
   └─ Starts: OpenHarmony services

6. Boot Animation
   │
   └─ Displays: (if is_support_boot_animation=true)

7. System Ready
   │
   └─ User can interact
```

**证据**: `/rk3568/cfg/init.rk3568.cfg`

### LiteOS-M 系统启动 (Neptune100)

```
1. Bootloader (ROM)
   │
   └─ Loads: kernel image

2. LiteOS-M Kernel
   │
   ├─ Initializes: kernel subsystems
   └─ Starts: init process

3. HDF Framework
   │
   └─ Loads: shield drivers

4. System Services
   │
   └─ Starts: OpenHarmony services

5. System Ready
```

---

## 关键入口点

### HDF 驱动入口

| 驱动 | 文件 | 函数 |
|------|------|------|
| Codec RK809 | rk809_codec_adapter.c | `Rk809DriverBind`, `Rk809DriverInit` |
| DAI RK3568 | rk3568_dai_adapter.c | `Rk3568DaiDriverInit` |
| WiFi | hdf_driver_bdh_register.c | `HdfDriverBdhRegister` |
| Camera | camera_device_manager.cpp | `CameraManagerInit` |

---

## 相关文档

- [03_Architecture](03_Architecture.md) - 架构说明
- [06_Hardware_Drivers](06_Hardware_Drivers.md) - 硬件驱动
- [08_Troubleshooting](08_Troubleshooting.md) - 问题排查
