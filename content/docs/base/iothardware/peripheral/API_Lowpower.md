# Lowpower 接口详细文档

> Low Power Consumption 低功耗管理接口规范

## 模块概述

**头文件**: `lowpower.h`  
**功能**: 提供设备低功耗模式切换能力  
**依赖**: `iot_errno.h` (错误码)  
**函数数量**: 2 个公共函数

## 枚举类型

### LpcType - 低功耗模式

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `NO_SLEEP` | 0 | 不休眠（正常工作模式） |
| `LIGHT_SLEEP` | 1 | 浅睡眠（部分外设休眠） |
| `DEEP_SLEEP` | 2 | 深睡眠（大部分外设关闭） |

**证据**: `lowpower.h:44-51`

## API 接口

### LpcInit - 初始化低功耗模块

```c
unsigned int LpcInit(void);
```

**功能**: 初始化低功耗管理模块

**参数**: 无

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 初始化成功 |
| `IOT_FAILURE` | 初始化失败 |

**调用示例**:
```c
unsigned int ret = LpcInit();
if (ret != IOT_SUCCESS) {
    // 初始化失败处理
}
```

**证据**: `lowpower.h:54-61`

---

### LpcSetType - 设置低功耗模式

```c
unsigned int LpcSetType(LpcType type);
```

**功能**: 设置设备的低功耗模式

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `LpcType` | 低功耗模式枚举值 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**调用示例**:
```c
// 进入浅睡眠模式
LpcSetType(LIGHT_SLEEP);

// 恢复正常模式
LpcSetType(NO_SLEEP);

// 进入深睡眠模式
LpcSetType(DEEP_SLEEP);
```

**证据**: `lowpower.h:64-72`

## 低功耗模式说明

### NO_SLEEP - 正常工作模式

```
┌─────────────────────────────────────────────────────────────┐
│ CPU: 运行                                                  │
│ RAM: 保持供电                                              │
│ Flash: 保持供电                                            │
│ 外设: 全部使能                                              │
│ 功耗: 最高                                                  │
└─────────────────────────────────────────────────────────────┘
```

**适用场景**: 系统正常运行、处理任务

### LIGHT_SLEEP - 浅睡眠模式

```
┌─────────────────────────────────────────────────────────────┐
│ CPU: 暂停（可被中断唤醒）                                    │
│ RAM: 保持供电                                              │
│ Flash: 保持供电                                            │
│ 部分外设: 低功耗模式                                        │
│ 功耗: 中等                                                  │
└─────────────────────────────────────────────────────────────┘
```

**唤醒源**: GPIO 中断、UART 接收、定时器

**适用场景**: 等待外部事件、降低功耗但保持快速响应

### DEEP_SLEEP - 深睡眠模式

```
┌─────────────────────────────────────────────────────────────┐
│ CPU: 关闭                                                   │
│ RAM: 可选保持供电                                          │
│ Flash: 可选关闭                                            │
│ 外设: 仅 RTC/低功耗定时器运行                               │
│ 功耗: 最低                                                  │
└─────────────────────────────────────────────────────────────┘
```

**唤醒源**: RTC 定时器、特定 GPIO（需配置）

**适用场景**: 长时间待机、超低功耗应用

## 完整使用示例

```c
#include "lowpower.h"
#include "iot_gpio.h"
#include "iot_uart.h"

#define GPIO_WAKEUP 10

int lowpower_example(void) {
    unsigned int ret;

    // 1. 初始化低功耗模块
    ret = LpcInit();
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 2. 配置唤醒源（GPIO 中断）
    IoTGpioInit(GPIO_WAKEUP);
    IoTGpioSetDir(GPIO_WAKEUP, IOT_GPIO_DIR_IN);
    IoTGpioRegisterIsrFunc(GPIO_WAKEUP, IOT_INT_TYPE_EDGE,
                           IOT_GPIO_EDGE_RISE_LEVEL_HIGH,
                           wakeup_handler, NULL);

    // 3. 进入浅睡眠模式
    printf("Entering light sleep...\n");
    ret = LpcSetType(LIGHT_SLEEP);

    // 4. 唤醒后执行
    printf("Woken up!\n");

    // 5. 恢复正常模式
    ret = LpcSetType(NO_SLEEP);

    return IOT_SUCCESS;
}

void wakeup_handler(char *arg) {
    // 唤醒处理
    LpcSetType(NO_SLEEP);  // 恢复到正常工作模式
}
```

## 功耗对比（参考值）

| 模式 | 功耗 | 唤醒时间 |
|------|------|----------|
| NO_SLEEP | ~10-100 mA | 立即 |
| LIGHT_SLEEP | ~1-10 mA | < 1 ms |
| DEEP_SLEEP | ~1-100 µA | 1-10 ms |

**注意**: 具体功耗值取决于芯片和外围电路

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [GPIO 接口](API_GPIO.md)
- [Watchdog 接口](API_Watchdog.md)
- [Reset 接口](API_Reset.md)
