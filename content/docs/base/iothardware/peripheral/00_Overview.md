# 项目概览

> IoT Hardware Peripheral 子系统定位、能力与运行环境说明

## 项目定位

### 子系统职责

**IoT Hardware Peripheral** 是 OpenHarmony 物联网子系统的硬件抽象层（HAL）接口定义模块，提供对常见 IoT 硬件外设的操作能力。

**证据**：`README.md:1-21`
```
The Internet of Things (IoT) hardware subsystem provides APIs for
operating IoT devices, including flash, GPIO, I2C, PWM, UART, and watchdog APIs.
```

### 边界与范围

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                          │
├─────────────────────────────────────────────────────────────┤
│  应用层 (JavaScript/ArkTS)                                   │
├─────────────────────────────────────────────────────────────┤
│  Native 层 (C/C++)                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  @ohos/iothardware_peripheral (本模块)               │    │
│  │  - 提供 C 语言接口给 Native 应用                      │    │
│  │  - 定义 HAL 接口规范                                  │    │
│  └─────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│  HAL 层 (Board Adapter)                                      │
│  - wifiiot_lite:hal_iothardware                             │
│  - Hi3861 芯片实现                                           │
├─────────────────────────────────────────────────────────────┤
│  硬件层                                                      │
│  - Hi3861 SoC                                                │
└─────────────────────────────────────────────────────────────┘
```

### 核心能力

| 能力 | 说明 | 头文件 |
|------|------|--------|
| **GPIO 控制** | 数字输入输出、中断触发 | `iot_gpio.h` |
| **I2C 通信** | 主从设备数据收发 | `iot_i2c.h` |
| **UART 串口** | 串行通信、波特率配置 | `iot_uart.h` |
| **PWM 输出** | 脉冲宽度调制、占空比控制 | `iot_pwm.h` |
| **Watchdog** | 系统看门狗、故障恢复 | `iot_watchdog.h` |
| **Flash 操作** | 读写擦除、持久化存储 | `iot_flash.h` |
| **设备重置** | 系统软复位 | `reset.h` |
| **低功耗管理** | 睡眠模式切换 | `lowpower.h` |

## 运行环境

### 支持平台

| 属性 | 说明 |
|------|------|
| **目标芯片** | Hi3861 |
| **内核类型** | LiteOS-M |
| **系统类型** | mini |
| **语言** | C (非 C++) |
| **代码规模** | 9 个头文件，约 1000 行接口定义 |

**证据**：`bundle.json:18-22`
```json
"adapted_system_type": ["mini"],
"features": [],
"syscap": []
```

### 依赖关系

```
本模块 ──依赖──> HAL 层实现
    │
    └── $ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware
```

### 不涉及的范围

| 范围 | 说明 | 原因 |
|------|------|------|
| N-API | JavaScript/TypeScript 接口 | 本项目为 Native C 接口 |
| IPC/SA | 进程间通信/系统能力 | 纯 HAL 接口，无 IPC |
| 权限管理 | 访问控制/鉴权 | 直接硬件操作，无需权限 |
| 多线程同步 | 互斥/信号量 | HAL 层实现决定 |

## 关键概念

### 接口设计模式

本模块采用 **头文件声明 + HAL 实现** 模式：

1. **接口定义层** (`interfaces/inner_api/*.h`)
   - 定义函数原型、数据结构、错误码
   - 提供给应用层调用

2. **HAL 实现层** (`hals/iot_hardware/wifiiot_lite`)
   - 芯片厂商提供的底层实现
   - 对接具体硬件寄存器

### 命名规范

| 前缀 | 说明 | 示例 |
|------|------|------|
| `IoT` | IoT 硬件通用前缀 | `IoTGpioInit`, `IoTI2cRead` |
| `Iot` | 枚举/结构体前缀 | `IotGpioValue`, `IotUartAttribute` |
| `IOT_` | 宏定义前缀 | `IOT_SUCCESS`, `IOT_FAILURE` |
| `Lpc` | 低功耗相关 | `LpcInit`, `LpcSetType` |
| `Reboot` | 重置相关 | `RebootDevice` |

### 返回值约定

| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS (0)` | 操作成功 |
| `IOT_FAILURE (-1)` | 操作失败 |
| **其他正值** | 特定操作的成功结果（如 `IoTUartRead` 返回读取字节数） |

**证据**：`iot_errno.h:45-51`
```c
#define IOT_SUCCESS    0
#define IOT_FAILURE   (-1)
```

## 版本历史

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 3.1 | 2022-06 | 当前版本（基于 bundle.json） |
| 2.2 | 2021-12 | 接口规范版本（基于头文件注释） |

## 相关资源

- **官方文档**: https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/IoT硬件子系统.md
- **源码仓库**: https://gitee.com/openharmony/base_iothardware
- **Issue 反馈**: https://gitee.com/openharmony/base_iothardware/issues
