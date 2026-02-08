# 开发板配置详解

## Neptune100

### 芯片规格

| 参数 | 值 |
|------|-----|
| **芯片** | 联盛德 W800 |
| **CPU** | 32位 XT804 处理器 |
| **架构** | ck803 |
| **CPU 型号** | ck804ef |
| **内核** | LiteOS-M |
| **工具链** | csky-elfabiv2-gcc |
| **存储** | 2MB Flash, 288KB RAM |

**证据**: `/neptune100/liteos_m/config.gni:15-18`

### 关键特性

| 特性 | 描述 |
|------|------|
| **Wi-Fi** | 802.11 b/g/n, Station/Soft-AP 双模 |
| **蓝牙** | BT/BLE 4.2 双模 |
| **安全** | AES128/DES/3DES/SHA1/RSA/CRC, 真随机数 |
| **接口** | GPIO×18, I2C×1, UART×5, PWM×5, ADC×2, I2S×1 |

**证据**: `/neptune100/README_zh.md:17-29`

### 通信能力

- Wi-Fi Station 模式
- Wi-Fi Soft-AP 模式
- 蓝牙 BT/BLE 双模
- 分布式软总线

### 配置结构

```
neptune100/
├── BUILD.gn                    # module_group
├── liteos_m/
│   ├── BUILD.gn              # kernel_module
│   └── config.gni            # CPU/工具链配置
├── shields/
│   └── neptune100/
│       └── neptune100.hcs    # HDF 设备配置
└── ohos.build
```

### 相关文档

- [neptune100/README_zh.md](../neptune100/README_zh.md)

---

## DAYU200 (RK3568)

### 芯片规格

| 参数 | 值 |
|------|-----|
| **芯片** | 瑞芯微 RK3568 |
| **CPU** | 4×Cortex-A55 |
| **架构** | armv8-a |
| **主频** | 2.0GHz |
| **工艺** | 22nm |
| **内核** | Linux |
| **工具链** | clang |
| **FPU** | neon-fp-armv8 |

**证据**: `/rk3568/config.gni:15-18`

### 处理器规格

| 组件 | 规格 |
|------|------|
| **CPU** | Quad-core Cortex-A55 @2.0GHz |
| **GPU** | 双核心架构 |
| **NPU** | 高效能神经网络处理器 |
| **内存** | 2GB LPDDR4 |
| **存储** | 32GB eMMC |

**证据**: `/rk3568/README_zh.md:45-72`

### 外设接口

| 类型 | 接口 |
|------|------|
| **显示** | HDMI 2.0×1 (4K@60fps), MIPI×2 (1080p@60fps), eDP×1 (2K@60fps) |
| **音频** | I2S/TDM/PDM×8ch, HDMI 音频, 喇叭, 耳机, 麦克风 |
| **网络** | 2×GMAC (10/100/1000M) |
| **无线** | SDIO Wi-Fi6, BT 4.2 |
| **摄像头** | MIPI-CSI2 (1×4-lane/2×2-lane@2.5Gbps/lane) |
| **USB** | USB2.0×2, USB3.0×1, USB3.0 OTG×1 |
| **存储** | Micro SD Card 3.0, SATA 3.0×1 |
| **扩展** | 20Pin (ADC×2, I2C×2, GPIO×7, 电源) |

**证据**: `/rk3568/README_zh.md:96-117`

### 功能特性

- 双网口：内外网数据访问和传输
- 多屏异显：最多三屏异显
- 多系统支持：OpenHarmony, Linux

### 配置结构

```
rk3568/
├── BUILD.gn                      # group
├── device.gni                    # SoC 配置
├── config.gni                    # 板级配置
├── audio_drivers/                # HDF 音频驱动
│   ├── codec/rk809_codec/
│   ├── dai/
│   ├── dsp/
│   ├── headset_monitor/
│   └── soc/
├── audio_alsa/                   # ALSA 适配
├── camera/vdi_impl/v4l2/         # V4L2 相机
├── wifi/bcmdhd_wifi6/hdfadapt/  # WiFi6
├── cfg/
│   └── init.rk3568.cfg
├── distributedhardware/
├── kernel/
└── updater/
```

### 编译命令

```bash
hb set    # 选择 hihope -> rk3568
hb build -f
```

**产物**: `out/rk3568/packages/phone/images/`

### 相关文档

- [rk3568/README_zh.md](../rk3568/README_zh.md)

---

## DAYU210 (RK3588)

### 芯片规格

| 参数 | 值 |
|------|-----|
| **芯片** | 瑞芯微 RK3588 |
| **CPU** | 4×Cortex-A76 + 4×Cortex-A55 |
| **架构** | armv8-a |
| **主频** | 2.4GHz |
| **内核** | Linux |
| **工具链** | clang |
| **FPU** | neon-fp-armv8 |

**证据**: `/dayu210/config.gni:15-18`

### 处理器规格

| 组件 | 规格 |
|------|------|
| **CPU** | 4×Cortex-A76 + 4×Cortex-A55 @2.4GHz |
| **GPU** | G610, OpenGLES 1.1/2.0/3.2, OpenCL 2.2, Vulkan 1.2 |
| **NPU** | 6.0 TOPs |
| **视频解码** | 8K |
| **视频编码** | 8K |
| **JPEG** | 高质量编解码 |
| **内存** | 8GB LPDDR4 (最大 32GB) |
| **存储** | 16GB/32GB/64GB/128GB eMMC |

**证据**: `/dayu210/README_zh.md:57-63`

### 核心板规格

| 参数 | 值 |
|------|-----|
| **SoC** | RK3588 |
| **RAM** | LPDDR4x 4GB/8GB/16GB (最大 32GB) |
| **ROM** | 32GB/64GB/128GB eMMC |
| **尺寸** | 85mm×50mm×4mm (连接器高度) |

**证据**: `/dayu210/README_zh.md:85-86`

### 外设接口

| 类型 | 接口 |
|------|------|
| **显示** | MIPI DSI×2 (4 Lane), HDMI2.1×2, eDP×1 (4 Lane), DP×2 (4 Lane) |
| **视频输入** | MIPI-CSI×4 (4 Lane), HDMI 2.0 IN |
| **音频** | Speaker×1 (1.3W), Headphone×1, Mic×2, I2S×8ch |
| **网络** | 2×RGMII (千兆以太网), Wi-Fi 6 |
| **扩展** | PCIe 3.0/2.1, SATA 3.3, USB 3.0×1, USB 2.0×2, Type-C×2 |
| **其他** | SDIO 3.0, SDMMC, ADC×8, PWM, UART×9, SPI×2, I2C×3, GPIO |

**证据**: `/dayu210/README_zh.md:47-84`

### 应用场景

- 智能 NVR
- 云终端
- 物联网网关
- 工业控制
- 信息发布终端
- 多媒体广告机
- 嵌入式人工智能

### 配置结构

```
dayu210/
├── BUILD.gn                      # group
├── device.gni                    # SoC 配置
├── config.gni                    # 板级配置
├── audio_drivers/                # HDF 音频驱动
│   ├── accessory/es8323/
│   ├── dai/
│   └── soc/
├── camera/vdi_impl/v4l2/         # V4L2 相机
├── cfg/
│   └── init.dayu210.cfg
├── distributedhardware/
├── kernel/                       # Linux 5.10 + 补丁
│   └── kernel_patch/linux-5.10/
├── loader/
│   └── uboot/
├── startup/reboot_loader/        # 启动控制
└── updater/
```

### 编译命令

```bash
./build.sh --product-name dayu210
```

**产物**: `out/rk3588/packages/phone/images/`

### 相关文档

- [dayu210/README_zh.md](../dayu210/README_zh.md)

---

## nearlink_dk_3863 (星闪)

### 芯片规格

| 参数 | 值 |
|------|-----|
| **芯片** | 海思 WS63 (星闪 NearLink) |
| **架构** | rv32imfc |
| **CPU** | RISC-V |
| **内核** | LiteOS-M |
| **工具链** | riscv32-linux-musl-gcc |

**证据**: `/nearlink_dk_3863/liteos_m/config.gni`

### 特点

- 基于星闪 (NearLink) 无线连接技术
- 轻量级 IoT 场景
- 配置型开发板，驱动代码在 `device/soc/hisilicon/ws63v100`

### 配置结构

```
nearlink_dk_3863/
├── README_zh.md
├── doc/figures/
├── liteos_m/
│   └── config.gni                # 板级配置
└── ohos.build
```

---

## 开发板对比

| 特性 | Neptune100 | RK3568 | DAYU210 | NearLink |
|------|-----------|--------|---------|----------|
| **芯片** | W800 | RK3568 | RK3588 | WS63 |
| **架构** | ck803 | armv8-a | armv8-a | rv32imfc |
| **内核** | LiteOS-M | Linux | Linux | LiteOS-M |
| **CPU** | XT804 | A55×4 | A76×4+A55×4 | RISC-V |
| **主频** | - | 2.0GHz | 2.4GHz | - |
| **Wi-Fi** | 802.11 b/g/n | Wi-Fi 6 | Wi-Fi 6 | NearLink |
| **蓝牙** | BT/BLE 4.2 | BT 4.2 | - | - |
| **NPU** | - | 有 | 6.0 TOPs | - |
| **定位** | 轻量 IoT | AI 多媒体 | 高性能 AI | 星闪 IoT |
| **代码量** | 0 | ~11,562 | ~4,877 | 0 |

---

## 相关文档

- [03_Architecture](03_Architecture.md) - 架构说明
- [04_GN_Build](04_GN_Build.md) - 构建配置
- [06_Hardware_Drivers](06_Hardware_Drivers.md) - 硬件驱动
