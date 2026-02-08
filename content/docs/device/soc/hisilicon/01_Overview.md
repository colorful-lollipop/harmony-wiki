# 项目概览

## 项目定位

`device_soc_hisilicon` 是 OpenHarmony 系统中**设备层（Device Layer）**的芯片适配仓库，负责：

1. **HDI 实现**: 提供 media、display、camera、codec、audio、ai 等硬件驱动的 OpenHarmony 接口定义（HDI）实现
2. **芯片 SDK**: 提供海思芯片的 SDK 库和头文件
3. **外设驱动**: 实现 GPIO、I2C、SPI 等外设的硬件抽象层
4. **启动与升级**: 包含启动加载器、OTA 升级等系统级功能

## 关键特征

### 芯片覆盖矩阵

| 芯片 | CPU 架构 | 主频 | 适用系统 | 典型应用 |
|------|---------|------|---------|---------|
| Hi3516DV300 | Cortex-A7 双核 | 900MHz | 小型系统/标准系统 | IP Camera |
| Hi3518EV300 | Cortex-A7 单核 | 900MHz | 小型系统 | Smart Camera |
| Hi3751V350 | Cortex-A53 四核 | 1.4GHz | 标准系统 | Smart TV |
| Hi3861V100 | Cortex-M3 | 160MHz | 轻量系统 | WiFi IoT |
| WS63V100 | - | - | 轻量系统 | NearLink IoT |

### 代码规模

- **代码语言**: C / C++
- **构建系统**: GN (Generate Ninja)
- **主要模块**: HAL、Platform Drivers、SDK、Boot

## 架构位置

```
┌─────────────────────────────────────────────┐
│           OpenHarmony 系统框架层              │
├─────────────────────────────────────────────┤
│         设备抽象层 (HDI/SDK 本仓库)            │
│  ┌─────────────────────────────────────────┐│
│  │ common/hal/ (显示/媒体/AI/USB)          ││
│  │ common/platform/ (GPIO/I2C/SPI等)       ││
│  └─────────────────────────────────────────┘├─────────────────────────────────────────────┤
│           芯片硬件层                          │
│  ┌─────────────────────────────────────────┐│
│  │ 芯片寄存器、外设驱动、SDK 实现             ││
│  └─────────────────────────────────────────┘│
└─────────────────────────────────────────────┘
```

## 核心能力

### 1. 硬件抽象层 (HAL)

位于 `common/hal/` 目录：

| 模块 | 功能 | 关键代码路径 |
|------|------|-------------|
| display | 显示硬件抽象 | `common/hal/display/` |
| media | 媒体处理 | `common/hal/media/` |
| ai | AI 硬件加速 | `common/hal/ai/` |
| usb | USB 主机/设备 | `common/hal/usb/` |
| multimedia | 多媒体编解码 | `common/hal/multimedia/` |
| update | OTA 升级 | `common/hal/update/` |

**代码证据**: `common/hal/BUILD.gn` 定义了各模块的构建目标

### 2. 平台驱动 (Platform Drivers)

位于 `common/platform/` 目录，涵盖外设驱动：

| 驱动类型 | 功能 | 关键目录 |
|---------|------|---------|
| GPIO | 通用输入输出 | `common/platform/gpio/` |
| I2C | I2C 通信 | `common/platform/i2c/` |
| SPI | SPI 通信 | `common/platform/spi/` |
| UART | 串口通信 | `common/platform/uart/` |
| ADC | 模数转换 | `common/platform/adc/` |
| PWM | 脉冲宽度调制 | `common/platform/pwm/` |
| RTC | 实时时钟 | `common/platform/rtc/` |
| Watchdog | 看门狗定时器 | `common/platform/watchdog/` |
| DMA | 直接内存访问 | `common/platform/dmac/` |

### 3. 芯片 SDK

各芯片的专用 SDK 实现：

| 芯片 | SDK 路径 | 说明 |
|------|---------|------|
| Hi3861V100 | `hi3861v100/sdk_liteos/` | WiFi IoT SDK (LiteOS) |
| Hi3516DV300 | `hi3516dv300/sdk_liteos/` | 轻量系统 SDK |
| Hi3516DV300 | `hi3516dv300/sdk_linux/` | Linux 标准系统 SDK |
| Hi3751V350 | `hi3751v350/sdk_linux/` | Linux SDK |

### 4. 启动与升级

位于各芯片 SDK 的 `boot/` 目录：

| 组件 | 功能 | 路径模式 |
|------|------|---------|
| Loaderboot | 二级加载器 | `{chip}/sdk_*/boot/loaderboot/` |
| Flashboot | 闪存启动加载器 | `{chip}/sdk_*/boot/flashboot/` |
| Commonboot | 通用启动代码 | `{chip}/sdk_*/boot/commonboot/` |
| 升级系统 | OTA 升级 | `{chip}/sdk_*/platform/system/upg/` |

**代码证据**: `hi3861v100/sdk_liteos/boot/flashboot/` 包含完整的启动和升级实现

## 依赖关系

### 上游依赖

- **OpenHarmony 系统框架**: 提供 HDI 接口定义
- **device_board_hisilicon**: 开发板适配
- **vendor_hihope**: 厂商配置

### 下游依赖

- **芯片硬件**: 寄存器映射、外设规格
- **第三方库**: mbedtls (安全)、lwip (网络) 等

## 开发流程

### 代码组织

```
//device/soc/hisilicon
├── common/           # 通用 HAL 和驱动（跨芯片复用）
│   ├── hal/         # 硬件抽象层
│   └── platform/    # 平台驱动
├── hi3516dv300/     # Hi3516DV300 专用代码
├── hi3518ev300/     # Hi3518EV300 专用代码
├── hi3751v350/      # Hi3751V350 专用代码
├── hi3861v100/      # Hi3861V100 专用代码
└── ws63v100/        # WS63V100 专用代码
```

### 适配新芯片的步骤

1. 在根目录创建 `{chip}/` 目录
2. 添加 `BUILD.gn` 或 `{chip}.gni` 构建配置
3. 创建适配层代码（`adapter/` 或 `sdk_*/`）
4. 如有新增外设，在 `common/platform/` 添加驱动
5. 如有新增 HDI 模块，在 `common/hal/` 添加实现

## 相关文档

- 目录结构: [03_Directory_Structure.md](03_Directory_Structure.md)
- HAL 模块: [04_HAL_Modules.md](04_HAL_Modules.md)
- 构建系统: [07_Build_System.md](07_Build_System.md)
- 安全评审: [09_Security_Review.md](09_Security_Review.md)
