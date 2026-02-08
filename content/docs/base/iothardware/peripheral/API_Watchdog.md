# Watchdog 接口详细文档

> Watchdog Timer 看门狗定时器接口规范

## 模块概述

**头文件**: `iot_watchdog.h`  
**功能**: 提供系统看门狗的使能、喂狗和关闭能力  
**依赖**: 无（独立模块）  
**函数数量**: 3 个公共函数

## API 接口

### IoTWatchDogEnable - 使能看门狗

```c
void IoTWatchDogEnable(void);
```

**功能**: 使能系统看门狗定时器

**参数**: 无

**返回值**: 无 (void)

**注意**:
- 看门狗使能后，必须定期调用 `IoTWatchDogKick()` 喂狗
- 如果不喂狗，超时后系统将自动复位
- 喂狗间隔应小于看门狗超时时间

**调用示例**:
```c
// 使能看门狗
IoTWatchDogEnable();

// 进入主循环
while (1) {
    // 执行任务...
    IoTWatchDogKick();  // 定期喂狗
}
```

**证据**: `iot_watchdog.h:44-49`

---

### IoTWatchDogKick - 喂狗

```c
void IoTWatchDogKick(void);
```

**功能**: 喂狗（重置看门狗定时器）

**参数**: 无

**返回值**: 无 (void)

**注意**:
- 必须在看门狗超时前调用此函数
- 喂狗间隔应小于超时时间（通常设置为超时的 50%-80%）

**调用示例**:
```c
// 定期喂狗
while (1) {
    // 处理任务
    process_tasks();

    // 喂狗
    IoTWatchDogKick();
}
```

**证据**: `iot_watchdog.h:51-57`

---

### IoTWatchDogDisable - 关闭看门狗

```c
void IoTWatchDogDisable(void);
```

**功能**: 关闭系统看门狗定时器

**参数**: 无

**返回值**: 无 (void)

**注意**:
- 调试阶段可以关闭看门狗以方便调试
- **生产环境不建议关闭看门狗**

**调用示例**:
```c
// 调试时关闭看门狗
IoTWatchDogDisable();

// 调试完成后重新使能
IoTWatchDogEnable();
```

**证据**: `iot_watchdog.h:60-65`

## 完整使用示例

```c
#include "iot_watchdog.h"

void system_main_loop(void) {
    // 1. 使能看门狗
    IoTWatchDogEnable();

    // 2. 主循环
    while (system_running) {
        // 执行系统任务
        process_sensor_data();
        update_display();

        // 3. 定期喂狗
        IoTWatchDogKick();
    }

    // 4. 关闭看门狗（如果需要）
    IoTWatchDogDisable();
}
```

## 看门狗工作原理

```
┌─────────────────────────────────────────────────────┐
│                    系统运行                          │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│              IoTWatchDogEnable()                     │
│              (使能看门狗，开始计时)                    │
└─────────────────────────────────────────────────────┘
                    │
         ┌─────────┴─────────┐
         │                   │
    ┌────▼────┐         ┌────▼────┐
    │ 执行任务 │         │ 异常卡住 │
    └────┬────┘         └────┬────┘
         │                   │
         └─────────┬─────────┘
                   │
              IoTWatchDogKick()  ←──── 定期调用
                   │                   (重置计时器)
                   ▼
         ┌─────────────────────────────────┐
         │    正常：系统持续运行            │
         │    超时：系统自动复位           │
         └─────────────────────────────────┘
```

## 最佳实践

### 1. 喂狗时机

```c
// 推荐：任务循环末尾喂狗
while (1) {
    read_sensors();
    process_data();
    IoTWatchDogKick();  // ✓ 正确
}

// 不推荐：延迟喂狗
while (1) {
    IoTWatchDogKick();  // ✗ 错误：任务可能在喂狗后卡住
    read_sensors();
    process_data();
}
```

### 2. 喂狗间隔

| 场景 | 建议喂狗间隔 |
|------|-------------|
| 简单任务 | 超时时间的 50% |
| 复杂任务 | 超时时间的 80% |
| 网络通信 | 超时时间的 60% |

### 3. 调试技巧

```c
#ifdef DEBUG
    // 调试模式下关闭看门狗
    IoTWatchDogDisable();
#else
    // 生产模式使能看门狗
    IoTWatchDogEnable();
#endif
```

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [Reset 接口](API_Reset.md)
- [安全评审](05_Security_Review.md)
