# 硬件驱动

## 驱动架构概览

本仓库采用 **HDF (Hardware Driver Foundation)** 框架，提供统一的硬件抽象层。

### 驱动分类

| 类型 | 位置 | 描述 |
|------|------|------|
| **音频驱动** | `*/audio_drivers/` | Codec/DAI/DSP/DMA |
| **音频 ALSA** | `*/audio_alsa/` | 标准音频接口适配 |
| **相机驱动** | `*/camera/vdi_impl/v4l2/` | V4L2 框架 |
| **WiFi 驱动** | `*/wifi/bcmdhd_wifi6/` | BCM WiFi6 |
| **系统配置** | `*/cfg/` | 初始化配置 |
| **分布式硬件** | `*/distributedhardware/` | 组件配置 |

---

## 音频驱动 (HDF)

### 目录结构

```
audio_drivers/
├── codec/rk809_codec/           # 编解码器
│   ├── include/
│   │   └── rk809_codec_impl.h
│   ├── src/
│   │   ├── rk809_codec_adapter.c   # HDF 驱动入口
│   │   └── rk809_codec_impl.c
├── dai/                           # 数字音频接口
│   ├── include/
│   │   └── rk3568_dai_ops.h
│   └── src/
│       ├── rk3568_dai_adapter.c
│       └── rk3568_dai_ops.c
├── dsp/                           # DSP
│   └── rk3568_dsp_adapter.c
├── headset_monitor/               # 耳机监测
│   └── analog_headset_core.c
└── soc/                          # DMA
    └── rk3568_dma_adapter.c
```

**证据**: `/rk3568/audio_drivers/` 目录

### HDF 驱动入口模式

```c
// rk809_codec_adapter.c
struct HdfDriverEntry g_Rk809DriverEntry = {
    .moduleVersion = 1,
    .moduleName = "CODEC_RK809",
    .Bind = Rk809DriverBind,
    .Init = Rk809DriverInit,
    .Release = RK809DriverRelease,
};
HDF_INIT(g_Rk809DriverEntry);
```

**证据**: `/rk3568/audio_drivers/codec/rk809_codec/src/rk809_codec_adapter.c`

### 关键接口

| 函数 | 用途 |
|------|------|
| `Rk809DriverBind()` | 绑定设备 |
| `Rk809DriverInit()` | 初始化资源 |
| `RK809DriverRelease()` | 释放资源 |

---

## ALSA 音频适配

### 目录结构

```
audio_alsa/
├── common.h
├── vendor_capture.c      # 音频采集 (录音)
└── vendor_render.c       # 音频播放
```

**证据**: `/rk3568/audio_alsa/` 目录

### 功能说明

| 文件 | 功能 |
|------|------|
| `vendor_capture.c` | ALSA 录音接口适配 |
| `vendor_render.c` | ALSA 播放接口适配 |

### 与 HDF 的关系

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Audio HAL     │     │   ALSA Interface│     │    Codec HW     │
│   (HDF)         │ ──▶ │   (vendor_*)   │ ──▶ │   (RK809)       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

## 相机驱动 (V4L2)

### 目录结构

```
camera/vdi_impl/v4l2/
├── demo/
│   └── ohos_camera_demo        # 演示程序
├── device_manager/              # 设备管理
│   ├── include/
│   │   ├── imx600.h           # 传感器配置
│   │   ├── rkispv5.h          # ISP V5 (RK3568)
│   │   ├── rkispv6.h          # ISP V6 (RK3588)
│   │   └── project_hardware.h
│   └── src/
│       ├── rkispv5.cpp         # ISP V5 实现
│       ├── rkispv6.cpp         # ISP V6 实现
│       └── camera_device_manager.cpp
└── pipeline_core/              # 流水线核心
    └── src/node/
        ├── rk_codec_node.cpp   # 编解码节点
        ├── rk_face_node.cpp    # 人脸检测节点
        └── rk_exif_node.cpp    # EXIF 处理
```

**证据**: `/rk3568/camera/vdi_impl/v4l2/` 目录

### ISP 版本对应

| ISP 版本 | 开发板 | 证据 |
|---------|--------|------|
| ISP V5 | RK3568 | `/rk3568/camera/vdi_impl/v4l2/device_manager/include/rkispv5.h` |
| ISP V6 | RK3588 | `/dayu210/camera/vdi_impl/v4l2/device_manager/include/rkispv6.h` |

### 关键类/命名空间

```cpp
// rk_codec_node.cpp
namespace OHOS::Camera {
    // 相机编解码处理
}
```

**证据**: `/rk3568/camera/vdi_impl/v4l2/pipeline_core/src/node/rk_codec_node.cpp`

### 图像处理节点

| 节点 | 功能 |
|------|------|
| `rk_codec_node.cpp` | JPEG 编解码 |
| `rk_face_node.cpp` | 人脸检测 |
| `rk_exif_node.cpp` | EXIF 元数据处理 |

---

## WiFi 驱动

### 目录结构

```
wifi/bcmdhd_wifi6/hdfadapt/
├── hdf_driver_bdh_register.c   # HDF 驱动注册
├── hdf_bdh_mac80211.c          # MAC 层实现
├── net_bdh_adpater.c           # 网络适配
└── hdf_wl_interface.h          # WiFi 接口定义
```

**证据**: `/rk3568/wifi/bcmdhd_wifi6/hdfadapt/` 目录

### 驱动功能

| 文件 | 功能 |
|------|------|
| `hdf_driver_bdh_register.c` | WiFi HDF 驱动入口 |
| `hdf_bdh_mac80211.c` | MAC 802.11 协议栈适配 |
| `net_bdh_adpater.c` | Linux 网络接口适配 |

---

## DAYU210 音频驱动

### ES8323 编解码器

```
audio_drivers/accessory/es8323/
├── include/
│   └── es8323_adapter.h
└── src/
    └── es8323_adapter.c        # HDF 驱动入口
```

**证据**: `/dayu210/audio_drivers/` 目录

### RK3588 DAI

```
audio_drivers/dai/
└── rk3588_dai_adapter.c        # DAI HDF 驱动
```

**证据**: `/dayu210/audio_drivers/dai/src/rk3588_dai_adapter.c`

---

## 系统配置

### 初始化配置

| 文件 | 开发板 | 证据 |
|------|--------|------|
| `init.rk3568.cfg` | RK3568 | `/rk3568/cfg/init.rk3568.cfg` |
| `init.dayu210.cfg` | DAYU210 | `/dayu210/cfg/init.dayu210.cfg` |

### 分布式硬件配置

```
distributedhardware/
└── distributed_hardware_components_cfg.json
```

**证据**: `/rk3568/distributedhardware/` 目录

---

## 启动控制 (DAYU210)

### 重启加载器

```
startup/reboot_loader/
├── BUILD.gn
└── reboot_loader.c              # 启动控制实现
```

**证据**: `/dayu210/startup/reboot_loader/` 目录

---

## 驱动编译配置

### RK3568 音频构建

```gn
# camera/vdi_impl/v4l2/BUILD.gn (示例)
ohos_shared_library("camera_board_vdi_impl") {
    sources = [
        "src/vdi_impl.cpp",
        # ...
    ]
    external_deps = [
        "drivers_peripheral_camera:peripheral_camera_*",
        "drivers_interface_camera:libcamera_proxy_1.0",
        # ...
    ]
}
```

**证据**: `/rk3568/camera/vdi_impl/v4l2/BUILD.gn`

---

## 相关文档

- [03_Architecture](03_Architecture.md) - 架构说明
- [05_Board_Configurations](05_Board_Configurations.md) - 开发板配置
- [04_GN_Build](04_GN_Build.md) - 构建配置
