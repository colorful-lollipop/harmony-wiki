# 目录导航

> IoT Hardware Peripheral Wiki 全站导航

## 新人阅读顺序（推荐）

```
1. README.md          → 项目概述与快速入门
2. 00_Overview.md    → 详细项目定位与能力说明
3. 01_API_Reference.md → 接口规范（按模块分类）
4. 03_Architecture.md → 内部架构与模块依赖
5. 04_Build_Config.md → GN 构建配置与编译产物
6. 05_Security_Review.md → 安全风险评审
```

## 完整文档列表

### 基础文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README.md](README.md) | 项目概述、覆盖范围、更新方式 | ⭐⭐⭐ |
| [SUMMARY.md](SUMMARY.md) | 全站导航与阅读顺序 | ⭐⭐⭐ |

### 项目说明

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [00_Overview.md](00_Overview.md) | 项目定位、边界、核心能力、运行环境 | ⭐⭐⭐ |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | ⭐⭐ |

### 接口文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [01_API_Reference.md](01_API_Reference.md) | API 清单总表与快速索引 | ⭐⭐⭐ |
| [API_GPIO.md](API_GPIO.md) | GPIO 接口详细文档 | ⭐⭐⭐ |
| [API_I2C.md](API_I2C.md) | I2C 接口详细文档 | ⭐⭐⭐ |
| [API_UART.md](API_UART.md) | UART 接口详细文档 | ⭐⭐⭐ |
| [API_PWM.md](API_PWM.md) | PWM 接口详细文档 | ⭐⭐ |
| [API_Watchdog.md](API_Watchdog.md) | Watchdog 接口详细文档 | ⭐⭐ |
| [API_Flash.md](API_Flash.md) | Flash 接口详细文档 | ⭐⭐ |
| [API_Reset.md](API_Reset.md) | Reset 接口详细文档 | ⭐ |
| [API_Lowpower.md](API_Lowpower.md) | Lowpower 接口详细文档 | ⭐ |
| [API_Errno.md](API_Errno.md) | 错误码定义 | ⭐⭐ |

### 架构与构建

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [03_Architecture.md](03_Architecture.md) | 组件图、数据流、线程模型 | ⭐⭐⭐ |
| [04_Build_Config.md](04_Build_Config.md) | GN targets 与编译产物 | ⭐⭐⭐ |

### 安全与附录

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [05_Security_Review.md](05_Security_Review.md) | 攻击面、信任边界、风险点 | ⭐⭐⭐ |
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图示 | ⭐ |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏与 Feature Flags | ⭐ |

## 接口速查表

### 按功能分类

| 功能 | 头文件 | 主要函数 |
|------|--------|----------|
| 通用输入输出 | `iot_gpio.h` | `IoTGpioInit`, `IoTGpioSetDir`, `IoTGpioRead/Write` |
| I2C 总线 | `iot_i2c.h` | `IoTI2cInit`, `IoTI2cRead`, `IoTI2cWrite` |
| 串口通信 | `iot_uart.h` | `IoTUartInit`, `IoTUartRead`, `IoTUartWrite` |
| 脉冲调制 | `iot_pwm.h` | `IoTPwmInit`, `IoTPwmStart`, `IoTPwmStop` |
| 看门狗 | `iot_watchdog.h` | `IoTWatchDogEnable`, `IoTWatchDogKick` |
| Flash 存储 | `iot_flash.h` | `IoTFlashRead`, `IoTFlashWrite`, `IoTFlashErase` |
| 设备重置 | `reset.h` | `RebootDevice` |
| 低功耗管理 | `lowpower.h` | `LpcInit`, `LpcSetType` |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始 Wiki 版本 |

## 反馈与贡献

- 发现文档错误？请提交 PR 或 Issue
- 需要补充接口文档？请参考 [01_API_Reference.md](01_API_Reference.md) 模板
- 安全问题？请发送邮件至 security@openharmony.io
