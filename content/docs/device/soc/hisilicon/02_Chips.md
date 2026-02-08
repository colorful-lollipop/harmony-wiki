# 芯片型号说明

本文档描述 `device_soc_hisilicon` 仓库支持的各海思芯片型号及其特性。

## 芯片总览

| SoC 型号 | CPU 架构 | 主频 | 适用系统 | 领域 | 开发板 |
|---------|---------|------|---------|------|--------|
| Hi3516DV300 | Cortex-A7 双核 | 900MHz | 小型系统/标准系统 | 智慧视觉 | HiSpark Taurus |
| Hi3518EV300 | Cortex-A7 单核 | 900MHz | 小型系统 | 智慧视觉 | HiSpark Aries |
| Hi3751V350 | Cortex-A53 四核 | 1.4GHz | 标准系统 | 智慧媒体 | HiSpark Phoenix |
| Hi3861V100 | Cortex-M3 | 160MHz | 轻量系统 | 智慧 IoT | HiSpark Pegasus |
| WS63V100 | - | - | 轻量系统 | 智慧 IoT | NearLink_DK_WS63 |

## Hi3516DV300

### 芯片特性

**代码路径**: `hi3516dv300/`

| 特性 | 说明 |
|------|------|
| CPU | HiSilicon Hi3516DV300，双核 Cortex-A7 |
| 主频 | 900MHz |
| 内存 | DDR3/DDR4 接口 |
| 存储 | SPI Flash、NAND Flash、eMMC |
| 显示 | HDMI 1.4、CVBS、BT.656/BT.601 |
| 视频编码 | H.264/H.265 编码，最大 4MP@30fps |
| 视频解码 | H.264/H.265 解码 |
| 安全 | 安全启动、加密引擎 |

### SDK 结构

```
hi3516dv300/
├── sdk_liteos/           # LiteOS 轻量系统 SDK
│   ├── app/            # 应用代码
│   ├── boot/           # 启动加载器
│   ├── config/         # 配置文件
│   ├── hdf_config/     # HDF 驱动配置 (.hcs 文件)
│   ├── include/        # 头文件
│   ├── mpp/            # 媒体处理平台 (Media Process Platform)
│   │   ├── component/  # 组件层
│   │   ├── hal/        # 硬件抽象层
│   │   └── ko/         # 内核模块
│   ├── platform/       # 平台驱动
│   └── BUILD.gn        # GN 构建配置
├── sdk_linux/           # Linux 标准系统 SDK
│   ├── sample/         # 示例代码
│   ├── config.gni      # 构建配置
│   └── BUILD.gn        # GN 构建配置
└── uboot/              # U-Boot 引导程序
```

### 关键 SDK 头文件

| 文件 | 功能 |
|------|------|
| `sdk_liteos/include/hi_cipher.h` | 加密模块接口 |
| `sdk_liteos/include/hi_comm_video.h` | 视频通用接口 |
| `sdk_liteos/include/hi_comm_isp.h` | ISP 通用接口 |
| `sdk_liteos/include/hi_comm_region.h` | 区域管理接口 |
| `sdk_liteos/include/hi_defines.h` | 常量定义 |

### MPP 媒体处理

**证据**: `hi3516dv300/sdk_liteos/mpp/BUILD.gn` 定义了 MPP 组件

MPP (Media Process Platform) 是海思芯片的媒体处理框架：

| 模块 | 功能 |
|------|------|
| VI (Video Input) | 视频输入 |
| VPSS (Video Process Subsystem) | 视频处理子系统 |
| VENC (Video Encoder) | 视频编码 |
| VDEC (Video Decoder) | 视频解码 |
| VO (Video Output) | 视频输出 |
| REGION | OSD 区域管理 |
| ISP | 图像信号处理 |

## Hi3518EV300

### 芯片特性

**代码路径**: `hi3518ev300/`

| 特性 | 说明 |
|------|------|
| CPU | HiSilicon Hi3518EV300，单核 Cortex-A7 |
| 主频 | 900MHz |
| 视频编码 | H.264/H.265，最大 2MP@30fps |
| 视频解码 | H.264/H.265 解码 |
| 接口 | Ethernet、USB 2.0、SDIO |

### SDK 结构

```
hi3518ev300/
├── mpp/               # MPP 媒体处理库
│   ├── component/    # 组件层
│   ├── hal/          # 硬件抽象层
│   └── module_init/  # 模块初始化
├── hdf_config/        # HDF 驱动配置
│   └── hdf.hcs       # HDF 配置源文件
└── BUILD.gn          # 构建配置
```

### 特点

Hi3518EV300 是 Hi3516DV300 的精简版本，主要用于：
- Smart Camera（智能摄像机）
- 门铃、猫眼等视觉设备
- 低功耗视觉应用

## Hi3751V350

### 芯片特性

**代码路径**: `hi3751v350/`

| 特性 | 说明 |
|------|------|
| CPU | HiSilicon Hi3751V350，四核 Cortex-A53 |
| 主频 | 1.4GHz |
| GPU | Mali-450 MP4 |
| 显示 | HDMI 2.0、LVDS、MIPI DSI |
| 视频编码 | H.264/H.265/VP9 |
| 视频解码 | H.264/H.265/VP9/AVS2 |
| 音频 | Dolby Audio、DTS |

### SDK 结构

```
hi3751v350/
├── sdk_linux/          # Linux SDK
│   ├── out/           # 输出产物
│   ├── sample/        # 示例代码
│   └── BUILD.gn       # 构建配置
├── gpu/                # GPU 库
│   └── BUILD.gn       # 构建配置
└── soc.gni            # SoC 配置
```

### 适用场景

- Smart TV（智能电视）
- 智能机顶盒
- 智慧大屏设备

## Hi3861V100

### 芯片特性

**代码路径**: `hi3861v100/`

| 特性 | 说明 |
|------|------|
| CPU | HiSilicon Hi3861V100，Cortex-M3 |
| 主频 | 160MHz |
| 内存 | 288KB SRAM、2MB Flash |
| 无线 | WiFi 802.11 b/g/n |
| 外设 | GPIO、UART、I2C、SPI、ADC、PWM |
| 安全 | 安全启动、硬件加密 |

### SDK 结构

```
hi3861v100/
├── hi3861_adapter/     # OpenHarmony 适配层
│   ├── hals/         # 硬件抽象层 (HAL)
│   │   ├── update/  # OTA 升级
│   │   ├── utils/   # 工具类
│   │   └── iot_hardware/  # IoT 硬件 (GPIO/UART 等)
│   ├── kal/          # 内核抽象层 (Kernel Abstraction Layer)
│   │   ├── cmsis/    # CMSIS 适配
│   │   └── posix/    # POSIX 适配
│   └── BUILD.gn     # 构建配置
├── sdk_liteos/        # LiteOS SDK
│   ├── app/         # 应用代码
│   │   ├── demo/    # 示例应用
│   │   └── wifiiot_app/  # WiFi IoT 应用
│   ├── boot/        # 启动加载器
│   │   ├── commonboot/  # 通用启动代码
│   │   ├── flashboot/   # Flash 启动
│   │   └── loaderboot/  # 二级加载器
│   ├── components/  # 组件
│   ├── config/      # 配置
│   ├── include/     # 头文件
│   ├── platform/    # 平台驱动
│   └── BUILD.gn    # 构建配置
└── BUILD.gn         # 根构建配置
```

### 关键适配层

**代码证据**: `hi3861v100/hi3861_adapter/hals/` 和 `hi3861v100/hi3861_adapter/kal/`

| 模块 | 路径 | 功能 |
|------|------|------|
| IoT Hardware | `hals/iot_hardware/wifiiot_lite/` | GPIO、UART、I2C、SPI、Flash、PWM |
| WiFi Service | `hals/communication/wifi_lite/wifiservice/` | WiFi STA/AP |
| WiFi Aware | `hals/communication/wifi_lite/wifiaware/` | WiFi Aware |
| CMSIS | `kal/cmsis/` | CMSIS-RTOS2 适配 |
| POSIX | `kal/posix/` | POSIX 接口适配 |

### 启动流程

```
Loaderboot → Flashboot → Kernel → App
```

**代码证据**: `hi3861v100/sdk_liteos/boot/` 包含完整启动代码

## WS63V100

### 芯片特性

**代码路径**: `ws63v100/`

| 特性 | 说明 |
|------|------|
| 无线 | NearLink (星闪) 短距无线技术 |
| 系统 | 轻量系统 |
| 特点 | 低功耗、高速率、多协议 |

### SDK 结构

```
ws63v100/
├── adapter/           # OpenHarmony 适配层
│   ├── hals/        # HAL
│   │   ├── update/  # 升级
│   │   ├── utils/   # 工具
│   │   ├── iot_hardware/  # IoT 硬件
│   │   └── communication/ # 通信 (BLE/WiFi/SLE)
│   ├── kal/         # KAL
│   │   ├── cmsis/   # CMSIS
│   │   └── posix/   # POSIX
│   └── BUILD.gn    # 构建配置
├── sdk/              # SDK
│   ├── build/      # 构建配置
│   ├── open_source/  # 开源组件 (mbedtls 等)
│   └── BUILD.gn    # 构建配置
└── BUILD.gn         # 根构建配置
```

### 通信模块

| 模块 | 路径 | 功能 |
|------|------|------|
| BLE Lite | `adapter/hals/communication/ble_lite/` | BLE 通信 |
| SLE Lite | `adapter/hals/communication/sle_lite/` | 星闪 (SLE) 通信 |
| WiFi Service | `adapter/hals/communication/wifi_lite/wifiservice/` | WiFi 通信 |

## 芯片对比

### CPU 性能对比

| 芯片 | 架构 | 核数 | 主频 | 适用系统 |
|------|------|------|------|---------|
| Hi3751V350 | Cortex-A53 | 4 | 1.4GHz | 标准系统 |
| Hi3516DV300 | Cortex-A7 | 2 | 900MHz | 小型/标准系统 |
| Hi3518EV300 | Cortex-A7 | 1 | 900MHz | 小型系统 |
| Hi3861V100 | Cortex-M3 | 1 | 160MHz | 轻量系统 |
| WS63V100 | - | - | - | 轻量系统 |

### 多媒体能力对比

| 芯片 | 编码能力 | 解码能力 | ISP | GPU |
|------|---------|---------|-----|-----|
| Hi3751V350 | 4K@30fps | 4K@60fps | 支持 | Mali-450 |
| Hi3516DV300 | 4MP@30fps | 1080P@60fps | 支持 | - |
| Hi3518EV300 | 2MP@30fps | 1080P@60fps | 支持 | - |
| Hi3861V100 | - | - | - | - |
| WS63V100 | - | - | - | - |

### 外设接口对比

| 芯片 | GPIO | I2C | SPI | UART | ADC | PWM |
|------|------|-----|-----|------|-----|-----|
| Hi3751V350 | 多 | 多 | 多 | 多 | - | - |
| Hi3516DV300 | 多 | 多 | 多 | 多 | - | - |
| Hi3518EV300 | 多 | 多 | 多 | 多 | - | - |
| Hi3861V100 | 16+ | 2 | 2 | 3 | 6 | 6 |
| WS63V100 | 多 | 多 | 多 | 多 | - | - |

## 相关文档

- 目录结构: [03_Directory_Structure.md](03_Directory_Structure.md)
- HAL 模块: [04_HAL_Modules.md](04_HAL_Modules.md)
- 构建系统: [07_Build_System.md](07_Build_System.md)
