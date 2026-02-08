# 平台驱动说明

本文档描述 `common/platform/` 目录下各平台驱动的功能和接口。

## 驱动总览

| 驱动类型 | 路径 | 功能 | 适用系统 |
|---------|------|------|---------|
| GPIO | `common/platform/gpio/` | 通用输入输出 | 全部 |
| I2C | `common/platform/i2c/` | I2C 通信 | 全部 |
| SPI | `common/platform/spi/` | SPI 通信 | 全部 |
| UART | `common/platform/uart/` | 串口通信 | 全部 |
| ADC | `common/platform/adc/` | 模数转换 | 全部 |
| PWM | `common/platform/pwm/` | 脉冲宽度调制 | 全部 |
| RTC | `common/platform/rtc/` | 实时时钟 | 全部 |
| Watchdog | `common/platform/watchdog/` | 看门狗定时器 | 全部 |
| DMA | `common/platform/dmac/` | 直接内存访问 | 全部 |
| MIPI CSI | `common/platform/mipi_csi/` | 摄像头接口 | 小型/标准 |
| MIPI DSI | `common/platform/mipi_dsi/` | 显示接口 | 小型/标准 |
| MMC | `common/platform/mmc/` | 存储卡接口 | 全部 |
| MTD | `common/platform/mtd/` | 存储设备 | 全部 |
| I2S | `common/platform/i2s/` | 音频接口 | 全部 |
| Pin | `common/platform/pin/` | 引脚复用 | 全部 |
| WiFi | `common/platform/wifi/` | WiFi 驱动 | 全部 |
| Ethernet | `common/platform/hieth-sf/` | 以太网驱动 | 全部 |

## GPIO 驱动

**路径**: `common/platform/gpio/`

### 功能

提供通用输入输出接口，支持：

- 设置引脚方向 (输入/输出)
- 读取引脚电平
- 设置输出电平
- 中断配置

### 头文件

| 文件 | 功能 |
|------|------|
| `platform/gpio/inc/hi_gpio.h` | GPIO 接口定义 |

### 关键 API

| API | 功能 |
|------|------|
| `hi_gpio_set_dir()` | 设置方向 |
| `hi_gpio_read()` | 读取电平 |
| `hi_gpio_write()` | 设置电平 |
| `hi_gpio_register_irq()` | 注册中断 |

## I2C 驱动

**路径**: `common/platform/i2c/`

### 功能

提供 I2C 主从设备通信支持：

- I2C 主机读写
- 从机模式支持
- 10位地址支持

### 关键 API

| API | 功能 |
|------|------|
| `hi_i2c_write()` | I2C 写操作 |
| `hi_i2c_read()` | I2C 读操作 |
| `hi_i2c_write_read()` | 写后读操作 |
| `hi_i2c_set_baudrate()` | 设置波特率 |

## SPI 驱动

**路径**: `common/platform/spi/`

### 功能

提供 SPI 全双工通信支持：

- 主/从模式
- 多片选支持
- 可配置时钟极性和相位
- DMA 传输支持

### 关键 API

| API | 功能 |
|------|------|
| `hi_spi_write()` | SPI 写 |
| `hi_spi_read()` | SPI 读 |
| `hi_spi_transfer()` | 同步传输 |
| `hi_spi_config()` | 配置参数 |

## UART 驱动

**路径**: `common/platform/uart/`

### 功能

提供串口通信支持：

- 波特率配置
- 数据位/停止位/校验位
- 接收/发送 FIFO
- 硬件流控 (RTS/CTS)

### 关键 API

| API | 功能 |
|------|------|
| `hi_uart_write()` | 串口发送 |
| `hi_uart_read()` | 串口接收 |
| `hi_uart_set_attribute()` | 配置参数 |

## ADC 驱动

**路径**: `common/platform/adc/`

### 功能

提供模数转换支持：

- 多通道 ADC
- 单次/连续转换
- 中断/DMA 读取

### 关键 API

| API | 功能 |
|------|------|
| `hi_adc_read()` | 读取 ADC 值 |
| `hi_adc_config()` | 配置通道和模式 |

## PWM 驱动

**路径**: `common/platform/pwm/`

### 功能

提供脉冲宽度调制支持：

- PWM 输出
- 占空比配置
- 频率配置

### 关键 API

| API | 功能 |
|------|------|
| `hi_pwm_start()` | 启动 PWM |
| `hi_pwm_stop()` | 停止 PWM |
| `hi_pwm_set_duty()` | 设置占空比 |
| `hi_pwm_set_freq()` | 设置频率 |

## RTC 驱动

**路径**: `common/platform/rtc/`

### 功能

提供实时时钟支持：

- 时间读写
- 闹钟设置
- 秒中断

### 关键 API

| API | 功能 |
|------|------|
| `hi_rtc_write_time()` | 设置时间 |
| `hi_rtc_read_time()` | 读取时间 |
| `hi_rtc_set_alarm()` | 设置闹钟 |

## Watchdog 驱动

**路径**: `common/platform/watchdog/**

### 功能

提供看门狗定时器支持：

- 启动看门狗
- 喂狗
- 配置超时时间

### 关键 API

| API | 功能 |
|------|------|
| `hi_watchdog_start()` | 启动看门狗 |
| `hi_watchdog_feed()` | 喂狗 |
| `hi_watchdog_stop()` | 停止看门狗 |

## DMA 驱动

**路径**: `common/platform/dmac/`

### 功能

提供直接内存访问支持：

- 内存到内存传输
- 外设到内存传输
- 链表模式
- 环形模式

### 关键 API

| API | 功能 |
|------|------|
| `hi_dmac_start()` | 启动 DMA |
| `hi_dmac_config()` | 配置传输参数 |
| `hi_dmac_wait_done()` | 等待传输完成 |

## MIPI CSI 驱动

**路径**: `common/platform/mipi_csi/`

### 功能

提供 MIPI CSI 摄像头接口支持：

- 摄像头数据接收
- 通道配置
- 时序参数配置

## MIPI DSI 驱动

**路径**: `common/platform/mipi_dsi/`

### 功能

提供 MIPI DSI 显示接口支持：

- 显示屏数据发送
- DSI 命令模式
- 视频模式

## MMC 驱动

**路径**: `common/platform/mmc/`

### 功能

提供 eMMC/SD 卡支持：

- 块设备读写
- 高速模式支持
- 硬件 ECC

## MTD 驱动

**路径**: `common/platform/mtd/**

### 功能

提供存储设备抽象：

- NOR/NAND Flash
- 分区管理
- 读写擦操作

## I2S 驱动

**路径**: `common/platform/i2s/`

### 功能

提供 I2S 数字音频接口：

- 主/从模式
- 立体声/单声道
- 采样率配置

## Pin 驱动

**路径**: `common/platform/pin/`

### 功能

提供引脚复用配置：

- 引脚功能选择
- 电气特性配置
- 上拉/下拉配置

## WiFi 驱动

**路径**: `common/platform/wifi/`

### 功能

提供 WiFi 芯片驱动支持：

- STA 模式
- AP 模式
- WiFi Aware

## Ethernet 驱动

**路径**: `common/platform/hieth-sf/`

### 功能

提供以太网驱动支持：

- 10/100M 以太网
- PHY 驱动
- 网络协议栈接口

## 构建配置

### 顶层 BUILD.gn

**代码证据**: `common/platform/BUILD.gn`

```gn
# 平台驱动构建配置
config("platform_config") {
  include_dirs = [
    "inc/",
    "include/",
  ]
}

# 各驱动模块定义
if (ohos_kernel_type == "liteos_a") {
  # LiteOS A 内核驱动
} else if (ohos_kernel_type == "liteos_m") {
  # LiteOS M 内核驱动
}
```

### Kconfig

**代码证据**: `common/platform/Kconfig`

提供内核配置菜单选项：

```
Platform Drivers  --->
    GPIO Support  --->
    I2C Support  --->
    SPI Support  --->
    ...
```

### lite.mk

**代码路径**: `common/platform/lite.mk`

用于 LiteOS Makefile 构建系统。

## 相关文档

- HAL 模块: [04_HAL_Modules.md](04_HAL_Modules.md)
- 芯片 SDK: [06_SDK_Architecture.md](06_SDK_Architecture.md)
