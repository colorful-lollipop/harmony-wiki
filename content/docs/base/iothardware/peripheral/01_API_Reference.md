# API 参考总览

> IoT Hardware Peripheral 子系统接口清单与快速索引

## API 清单表

### GPIO 接口 (iot_gpio.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `IoTGpioInit` | 初始化 GPIO | `unsigned int id` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:97` |
| `IoTGpioDeinit` | 去初始化 GPIO | `unsigned int id` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:108` |
| `IoTGpioSetDir` | 设置 GPIO 方向 | `unsigned int id, IotGpioDir dir` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:120` |
| `IoTGpioGetDir` | 获取 GPIO 方向 | `unsigned int id, IotGpioDir *dir` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:132` |
| `IoTGpioSetOutputVal` | 设置输出电平 | `unsigned int id, IotGpioValue val` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:144` |
| `IoTGpioGetOutputVal` | 获取输出电平 | `unsigned int id, IotGpioValue *val` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:156` |
| `IoTGpioGetInputVal` | 获取输入电平 | `unsigned int id, IotGpioValue *val` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:168` |
| `IoTGpioRegisterIsrFunc` | 注册中断回调 | `unsigned int id, IotGpioIntType intType, IotGpioIntPolarity intPolarity, GpioIsrCallbackFunc func, char *arg` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步注册 | `:185` |
| `IoTGpioUnregisterIsrFunc` | 注销中断回调 | `unsigned int id` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:197` |
| `IoTGpioSetIsrMask` | 设置中断屏蔽 | `unsigned int id, unsigned char mask` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:210` |
| `IoTGpioSetIsrMode` | 设置中断模式 | `unsigned int id, IotGpioIntType intType, IotGpioIntPolarity intPolarity` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:225` |

### I2C 接口 (iot_i2c.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `IoTI2cInit` | 初始化 I2C | `unsigned int id, unsigned int baudrate` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:56` |
| `IoTI2cDeinit` | 去初始化 I2C | `unsigned int id` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:67` |
| `IoTI2cWrite` | 写入 I2C 数据 | `unsigned int id, unsigned short deviceAddr, const unsigned char *data, unsigned int dataLen` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:83` |
| `IoTI2cRead` | 读取 I2C 数据 | `unsigned int id, unsigned short deviceAddr, unsigned char *data, unsigned int dataLen` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:99` |
| `IoTI2cSetBaudrate` | 设置 I2C 波特率 | `unsigned int id, unsigned int baudrate` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:111` |

### UART 接口 (iot_uart.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `IoTUartInit` | 初始化 UART | `unsigned int id, const IotUartAttribute *param` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:155` |
| `IoTUartRead` | 读取 UART 数据 | `unsigned int id, unsigned char *data, unsigned int dataLen` | 读取字节数/-1 | 同步 | `:169` |
| `IoTUartWrite` | 写入 UART 数据 | `unsigned int id, const unsigned char *data, unsigned int dataLen` | 写入字节数/-1 | 同步 | `:183` |
| `IoTUartDeinit` | 去初始化 UART | `unsigned int id` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:194` |
| `IoTUartSetFlowCtrl` | 设置流控 | `unsigned int id, IotFlowCtrl flowCtrl` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:208` |

### PWM 接口 (iot_pwm.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `IoTPwmInit` | 初始化 PWM | `unsigned int port` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:53` |
| `IoTPwmDeinit` | 去初始化 PWM | `unsigned int port` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:64` |
| `IoTPwmStart` | 启动 PWM 输出 | `unsigned int port, unsigned short duty, unsigned int freq` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:79` |
| `IoTPwmStop` | 停止 PWM 输出 | `unsigned int port` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:90` |

### Watchdog 接口 (iot_watchdog.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `IoTWatchDogEnable` | 使能看门狗 | 无 | void | 同步 | `:49` |
| `IoTWatchDogKick` | 喂狗 | 无 | void | 同步 | `:57` |
| `IoTWatchDogDisable` | 关闭看门狗 | 无 | void | 同步 | `:65` |

### Flash 接口 (iot_flash.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `IoTFlashRead` | 读取 Flash 数据 | `unsigned int flashOffset, unsigned int size, unsigned char *ramData` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:57` |
| `IoTFlashWrite` | 写入 Flash 数据 | `unsigned int flashOffset, unsigned int size, const unsigned char *ramData, unsigned char doErase` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:73` |
| `IoTFlashErase` | 擦除 Flash 数据 | `unsigned int flashOffset, unsigned int size` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:86` |
| `IoTFlashInit` | 初始化 Flash | 无 | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:96` |
| `IoTFlashDeinit` | 去初始化 Flash | 无 | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:106` |

### Reset 接口 (reset.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `RebootDevice` | 设备软复位 | `unsigned int cause` | void | 同步（阻塞） | `:51` |

### Lowpower 接口 (lowpower.h)

| 函数名 | 功能 | 参数 | 返回值 | 同步/异步 | 头文件位置 |
|--------|------|------|--------|-----------|------------|
| `LpcInit` | 初始化低功耗 | 无 | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:61` |
| `LpcSetType` | 设置低功耗模式 | `LpcType type` | `IOT_SUCCESS`/`IOT_FAILURE` | 同步 | `:72` |

### 错误码 (iot_errno.h)

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| `IOT_SUCCESS` | `0` | 操作成功 |
| `IOT_FAILURE` | `-1` | 操作失败 |

## 枚举类型汇总

### GPIO 枚举

| 枚举类型 | 取值 | 说明 |
|----------|------|------|
| `IotGpioValue` | `IOT_GPIO_VALUE0` | 低电平 |
| | `IOT_GPIO_VALUE1` | 高电平 |
| `IotGpioDir` | `IOT_GPIO_DIR_IN` | 输入模式 |
| | `IOT_GPIO_DIR_OUT` | 输出模式 |
| `IotGpioIntType` | `IOT_INT_TYPE_LEVEL` | 电平触发 |
| | `IOT_INT_TYPE_EDGE` | 边沿触发 |
| `IotGpioIntPolarity` | `IOT_GPIO_EDGE_FALL_LEVEL_LOW` | 下降沿/低电平 |
| | `IOT_GPIO_EDGE_RISE_LEVEL_HIGH` | 上升沿/高电平 |

### UART 枚举

| 枚举类型 | 取值 | 说明 |
|----------|------|------|
| `IotUartIdxDataBit` | `IOT_UART_DATA_BIT_5` ~ `8` | 数据位 5-8 位 |
| `IotUartStopBit` | `IOT_UART_STOP_BIT_1` | 1 位停止位 |
| | `IOT_UART_STOP_BIT_2` | 2 位停止位 |
| `IotUartParity` | `IOT_UART_PARITY_NONE` | 无校验 |
| | `IOT_UART_PARITY_ODD` | 奇校验 |
| | `IOT_UART_PARITY_EVEN` | 偶校验 |
| `IotUartBlockState` | `IOT_UART_BLOCK_STATE_NONE_BLOCK` | 非阻塞 |
| | `IOT_UART_BLOCK_STATE_BLOCK` | 阻塞 |
| `IotFlowCtrl` | `IOT_FLOW_CTRL_NONE` | 无流控 |
| | `IOT_FLOW_CTRL_RTS_CTS` | RTS/CTS 流控 |
| | `IOT_FLOW_CTRL_RTS_ONLY` | 仅 RTS |
| | `IOT_FLOW_CTRL_CTS_ONLY` | 仅 CTS |

### Lowpower 枚举

| 枚举类型 | 取值 | 说明 |
|----------|------|------|
| `LpcType` | `NO_SLEEP` | 不休眠 |
| | `LIGHT_SLEEP` | 浅睡眠 |
| | `DEEP_SLEEP` | 深睡眠 |

## 数据结构

### IotUartAttribute

```c
typedef struct {
    unsigned int baudRate;           // 波特率
    IotUartIdxDataBit dataBits;      // 数据位
    IotUartStopBit stopBits;         // 停止位
    IotUartParity parity;            // 校验位
    IotUartBlockState rxBlock;       // 接收阻塞状态
    IotUartBlockState txBlock;       // 发送阻塞状态
    unsigned char pad;               // 填充字节
} IotUartAttribute;
```

### GpioIsrCallbackFunc

```c
typedef void (*GpioIsrCallbackFunc)(char *arg);
```

## 头文件包含关系

```c
// 所有模块都需要包含错误码定义
#include "iot_errno.h"

// 各模块头文件独立，不存在跨模块依赖
#include "iot_gpio.h"
#include "iot_i2c.h"
#include "iot_uart.h"
#include "iot_pwm.h"
#include "iot_watchdog.h"
#include "iot_flash.h"
#include "reset.h"
#include "lowpower.h"
```

## 快速索引

### 按功能查找

| 需求 | 头文件 | 推荐函数 |
|------|--------|----------|
| 读取引脚电平 | `iot_gpio.h` | `IoTGpioGetInputVal` |
| 设置引脚输出 | `iot_gpio.h` | `IoTGpioSetOutputVal` |
| 配置引脚中断 | `iot_gpio.h` | `IoTGpioRegisterIsrFunc` |
| I2C 设备通信 | `iot_i2c.h` | `IoTI2cRead`/`IoTI2cWrite` |
| 串口数据收发 | `iot_uart.h` | `IoTUartRead`/`IoTUartWrite` |
| PWM 信号输出 | `iot_pwm.h` | `IoTPwmStart` |
| 系统看门狗 | `iot_watchdog.h` | `IoTWatchDogEnable` |
| Flash 存储操作 | `iot_flash.h` | `IoTFlashRead`/`Write` |
| 系统软复位 | `reset.h` | `RebootDevice` |
| 低功耗模式 | `lowpower.h` | `LpcSetType` |
