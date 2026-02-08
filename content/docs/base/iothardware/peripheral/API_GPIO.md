# GPIO 接口详细文档

> General Purpose Input/Output 通用输入输出接口规范

## 模块概述

**头文件**: `iot_gpio.h`  
**功能**: 提供 GPIO 引脚的初始化、方向配置、电平读写和中断管理能力  
**依赖**: `iot_errno.h` (错误码)  
**函数数量**: 10 个公共函数

## 枚举类型

### IotGpioValue - 电平值

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_GPIO_VALUE0` | 0 | 低电平 (Logic Low) |
| `IOT_GPIO_VALUE1` | 1 | 高电平 (Logic High) |

**证据**: `iot_gpio.h:45-50`

### IotGpioDir - 方向

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_GPIO_DIR_IN` | 0 | 输入模式 (Input) |
| `IOT_GPIO_DIR_OUT` | 1 | 输出模式 (Output) |

**证据**: `iot_gpio.h:55-60`

### IotGpioIntType - 中断类型

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_INT_TYPE_LEVEL` | 0 | 电平触发型中断 (Level-sensitive) |
| `IOT_INT_TYPE_EDGE` | 1 | 边沿触发型中断 (Edge-sensitive) |

**证据**: `iot_gpio.h:65-70`

### IotGpioIntPolarity - 中断极性

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_GPIO_EDGE_FALL_LEVEL_LOW` | 0 | 下降沿/低电平触发 |
| `IOT_GPIO_EDGE_RISE_LEVEL_HIGH` | 1 | 上升沿/高电平触发 |

**证据**: `iot_gpio.h:75-80`

### GpioIsrCallbackFunc - 中断回调函数类型

```c
typedef void (*GpioIsrCallbackFunc)(char *arg);
```

**参数说明**:
- `arg`: 中断触发时传递给回调函数的参数

**证据**: `iot_gpio.h:86`

## API 接口

### IoTGpioInit - 初始化 GPIO

```c
unsigned int IoTGpioInit(unsigned int id);
```

**功能**: 初始化指定的 GPIO 引脚

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 初始化成功 |
| `IOT_FAILURE` | 初始化失败 |

**调用示例**:
```c
unsigned int ret = IoTGpioInit(0);  // 初始化 GPIO 0
if (ret != IOT_SUCCESS) {
    // 处理错误
}
```

**证据**: `iot_gpio.h:89-97`

---

### IoTGpioDeinit - 去初始化 GPIO

```c
unsigned int IoTGpioDeinit(unsigned int id);
```

**功能**: 去初始化指定的 GPIO 引脚，释放相关资源

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 去初始化成功 |
| `IOT_FAILURE` | 去初始化失败 |

**证据**: `iot_gpio.h:100-108`

---

### IoTGpioSetDir - 设置 GPIO 方向

```c
unsigned int IoTGpioSetDir(unsigned int id, IotGpioDir dir);
```

**功能**: 设置指定 GPIO 引脚的方向（输入或输出）

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `dir` | `IotGpioDir` | 方向枚举值 (`IOT_GPIO_DIR_IN` / `IOT_GPIO_DIR_OUT`) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**调用示例**:
```c
// 设置 GPIO 0 为输出模式
IoTGpioSetDir(0, IOT_GPIO_DIR_OUT);

// 设置 GPIO 1 为输入模式
IoTGpioSetDir(1, IOT_GPIO_DIR_IN);
```

**证据**: `iot_gpio.h:111-120`

---

### IoTGpioGetDir - 获取 GPIO 方向

```c
unsigned int IoTGpioGetDir(unsigned int id, IotGpioDir *dir);
```

**功能**: 获取指定 GPIO 引脚的方向

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `dir` | `IotGpioDir *` | 指向方向枚举变量的指针，用于输出结果 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 获取成功 |
| `IOT_FAILURE` | 获取失败 |

**调用示例**:
```c
IotGpioDir dir;
IoTGpioGetDir(0, &dir);
if (dir == IOT_GPIO_DIR_OUT) {
    // 当前为输出模式
}
```

**证据**: `iot_gpio.h:123-132`

---

### IoTGpioSetOutputVal - 设置输出电平

```c
unsigned int IoTGpioSetOutputVal(unsigned int id, IotGpioValue val);
```

**功能**: 设置指定 GPIO 输出引脚的电平值

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `val` | `IotGpioValue` | 电平枚举值 (`IOT_GPIO_VALUE0` / `IOT_GPIO_VALUE1`) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**调用示例**:
```c
// 设置 GPIO 0 输出高电平
IoTGpioSetOutputVal(0, IOT_GPIO_VALUE1);

// 设置 GPIO 0 输出低电平
IoTGpioSetOutputVal(0, IOT_GPIO_VALUE0);
```

**证据**: `iot_gpio.h:135-144`

---

### IoTGpioGetOutputVal - 获取输出电平

```c
unsigned int IoTGpioGetOutputVal(unsigned int id, IotGpioValue *val);
```

**功能**: 获取指定 GPIO 输出引脚的当前电平值

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `val` | `IotGpioValue *` | 指向电平枚举变量的指针，用于输出结果 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 获取成功 |
| `IOT_FAILURE` | 获取失败 |

**证据**: `iot_gpio.h:147-156`

---

### IoTGpioGetInputVal - 获取输入电平

```c
unsigned int IoTGpioGetInputVal(unsigned int id, IotGpioValue *val);
```

**功能**: 读取指定 GPIO 输入引脚的当前电平值

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `val` | `IotGpioValue *` | 指向电平枚举变量的指针，用于输出读取结果 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 读取成功 |
| `IOT_FAILURE` | 读取失败 |

**调用示例**:
```c
IotGpioValue input;
IoTGpioGetInputVal(1, &input);
if (input == IOT_GPIO_VALUE1) {
    // 引脚输入为高电平
}
```

**证据**: `iot_gpio.h:159-168`

---

### IoTGpioRegisterIsrFunc - 注册中断回调

```c
unsigned int IoTGpioRegisterIsrFunc(unsigned int id, IotGpioIntType intType,
                                     IotGpioIntPolarity intPolarity,
                                     GpioIsrCallbackFunc func, char *arg);
```

**功能**: 为指定 GPIO 引脚注册中断服务例程

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `intType` | `IotGpioIntType` | 中断触发类型 (电平/边沿) |
| `intPolarity` | `IotGpioIntPolarity` | 中断极性 (上升沿/下降沿等) |
| `func` | `GpioIsrCallbackFunc` | 中断回调函数指针 |
| `arg` | `char *` | 传递给回调函数的参数 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 注册成功 |
| `IOT_FAILURE` | 注册失败 |

**调用示例**:
```c
void gpio_isr_handler(char *arg) {
    printf("GPIO interrupt triggered, arg: %s\n", arg);
}

IoTGpioRegisterIsrFunc(2, IOT_INT_TYPE_EDGE,
                      IOT_GPIO_EDGE_RISE_LEVEL_HIGH,
                      gpio_isr_handler, "ISR_ARG");
```

**证据**: `iot_gpio.h:171-186`

---

### IoTGpioUnregisterIsrFunc - 注销中断回调

```c
unsigned int IoTGpioUnregisterIsrFunc(unsigned int id);
```

**功能**: 注销指定 GPIO 引脚的中断服务例程

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 注销成功 |
| `IOT_FAILURE` | 注销失败 |

**证据**: `iot_gpio.h:189-197`

---

### IoTGpioSetIsrMask - 设置中断屏蔽

```c
unsigned int IoTGpioSetIsrMask(unsigned int id, unsigned char mask);
```

**功能**: 设置指定 GPIO 引脚的中断屏蔽状态

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `mask` | `unsigned char` | 屏蔽标志 (1=屏蔽, 0=不屏蔽) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**证据**: `iot_gpio.h:200-210`

---

### IoTGpioSetIsrMode - 设置中断模式

```c
unsigned int IoTGpioSetIsrMode(unsigned int id, IotGpioIntType intType,
                                IotGpioIntPolarity intPolarity);
```

**功能**: 配置指定 GPIO 引脚的中断触发模式

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | GPIO 引脚编号 |
| `intType` | `IotGpioIntType` | 中断触发类型 |
| `intPolarity` | `IotGpioIntPolarity` | 中断极性 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**证据**: `iot_gpio.h:213-225`

## 完整使用示例

```c
#include "iot_gpio.h"
#include "iot_errno.h"

#define GPIO_LED 0
#define GPIO_BUTTON 1

int gpio_example(void) {
    unsigned int ret;

    // 1. 初始化 LED 引脚（输出）
    ret = IoTGpioInit(GPIO_LED);
    if (ret != IOT_SUCCESS) return ret;

    IoTGpioSetDir(GPIO_LED, IOT_GPIO_DIR_OUT);

    // 2. 初始化按钮引脚（输入）
    ret = IoTGpioInit(GPIO_BUTTON);
    if (ret != IOT_SUCCESS) return ret;

    IoTGpioSetDir(GPIO_BUTTON, IOT_GPIO_DIR_IN);

    // 3. 控制 LED
    IoTGpioSetOutputVal(GPIO_LED, IOT_GPIO_VALUE1);  // LED 亮

    // 4. 读取按钮状态
    IotGpioValue btn_val;
    IoTGpioGetInputVal(GPIO_BUTTON, &btn_val);

    // 5. 清理
    IoTGpioDeinit(GPIO_LED);
    IoTGpioDeinit(GPIO_BUTTON);

    return IOT_SUCCESS;
}
```

## 错误码说明

| 错误码 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 操作成功 |
| `IOT_FAILURE` | 操作失败（通用错误） |

**注意**: HAL 层可能返回芯片特定的错误码，具体请参考 Hi3861 芯片手册。

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [UART 接口](API_UART.md)
- [I2C 接口](API_I2C.md)
- [架构说明](03_Architecture.md)
