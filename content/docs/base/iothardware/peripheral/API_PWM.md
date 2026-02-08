# PWM 接口详细文档

> Pulse Width Modulation 脉冲宽度调制接口规范

## 模块概述

**头文件**: `iot_pwm.h`  
**功能**: 提供 PWM 信号输出的初始化、启动、停止能力  
**依赖**: `iot_errno.h` (错误码)  
**函数数量**: 4 个公共函数

## API 接口

### IoTPwmInit - 初始化 PWM

```c
unsigned int IoTPwmInit(unsigned int port);
```

**功能**: 初始化指定的 PWM 端口

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `port` | `unsigned int` | PWM 端口号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 初始化成功 |
| `IOT_FAILURE` | 初始化失败 |

**调用示例**:
```c
unsigned int ret = IoTPwmInit(0);  // 初始化 PWM 端口 0
```

**证据**: `iot_pwm.h:45-53`

---

### IoTPwmDeinit - 去初始化 PWM

```c
unsigned int IoTPwmDeinit(unsigned int port);
```

**功能**: 去初始化指定的 PWM 端口，释放相关资源

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `port` | `unsigned int` | PWM 端口号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 去初始化成功 |
| `IOT_FAILURE` | 去初始化失败 |

**证据**: `iot_pwm.h:56-64`

---

### IoTPwmStart - 启动 PWM 输出

```c
unsigned int IoTPwmStart(unsigned int port, unsigned short duty, unsigned int freq);
```

**功能**: 在指定端口启动 PWM 信号输出，设置占空比和频率

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `port` | `unsigned int` | PWM 端口号 |
| `duty` | `unsigned short` | 占空比 (1-99)，表示高电平占比百分比 |
| `freq` | `unsigned int` | 输出频率 (单位: Hz) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 启动成功 |
| `IOT_FAILURE` | 启动失败 |

**调用示例**:
```c
// 启动 PWM0，50% 占空比，1kHz 频率
unsigned int ret = IoTPwmStart(0, 50, 1000);

// 启动 PWM0，25% 占空比，10kHz 频率
ret = IoTPwmStart(0, 25, 10000);
```

**注意**: `duty` 参数范围为 1-99，不支持 0% 或 100% 占空比（此时应使用普通 GPIO 输出）

**证据**: `iot_pwm.h:67-79`

---

### IoTPwmStop - 停止 PWM 输出

```c
unsigned int IoTPwmStop(unsigned int port);
```

**功能**: 停止指定端口的 PWM 信号输出

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `port` | `unsigned int` | PWM 端口号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 停止成功 |
| `IOT_FAILURE` | 停止失败 |

**调用示例**:
```c
// 停止 PWM0 输出
IoTPwmStop(0);
```

**证据**: `iot_pwm.h:82-90`

## 完整使用示例

```c
#include "iot_pwm.h"
#include "iot_errno.h"

#define PWM_PORT 0

int pwm_led_dimming_example(void) {
    unsigned int ret;

    // 1. 初始化 PWM
    ret = IoTPwmInit(PWM_PORT);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 2. LED 呼吸灯效果 - 渐变占空比
    for (int i = 1; i < 100; i++) {
        ret = IoTPwmStart(PWM_PORT, i, 1000);  // 1kHz
        if (ret != IOT_SUCCESS) break;
        // 延时...
    }

    // 3. 停止 PWM
    IoTPwmStop(PWM_PORT);

    // 4. 清理
    IoTPwmDeinit(PWM_PORT);

    return IOT_SUCCESS;
}
```

## PWM 参数说明

### 占空比 (Duty Cycle)

占空比表示一个周期内高电平时间所占的比例：

| 占空比 | 效果 |
|--------|------|
| 1% | 几乎关闭 |
| 25% | 25% 亮/75% 暗 |
| 50% | 半亮（最常见） |
| 75% | 75% 亮/25% 暗 |
| 99% | 几乎全亮 |

### 频率 (Frequency)

| 频率范围 | 应用场景 |
|----------|----------|
| < 100 Hz | 人眼可见闪烁（照明控制） |
| 100 Hz - 1 kHz | LED 调光、电机调速 |
| 1 kHz - 100 kHz | 音频信号、精密控制 |
| > 100 kHz | 超声波应用、电力电子 |

## 注意事项

1. **占空比限制**: `duty` 必须为 1-99 之间的整数
2. **频率限制**: 具体频率范围取决于 Hi3861 芯片 PWM 硬件支持
3. **初始化顺序**: 必须先调用 `IoTPwmInit` 才能调用 `IoTPwmStart`
4. **资源释放**: 不使用 PWM 时调用 `IoTPwmDeinit` 释放资源

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [GPIO 接口](API_GPIO.md)
- [架构说明](03_Architecture.md)
