# Reset 接口详细文档

> Device Reboot 设备重置接口规范

## 模块概述

**头文件**: `reset.h`  
**功能**: 提供设备软复位能力  
**依赖**: 无（独立模块）  
**函数数量**: 1 个公共函数

## API 接口

### RebootDevice - 设备软复位

```c
void RebootDevice(unsigned int cause);
```

**功能**: 使用指定的复位原因触发设备软复位

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `cause` | `unsigned int` | 复位原因码 |

**返回值**: 无 (void)

**注意**:
- 此函数会立即触发设备复位
- 复位后系统将重新启动
- 应用程序应在此之前保存关键数据

**调用示例**:
```c
#include "reset.h"

// 正常重启
RebootDevice(0);

// OTA 升级后重启
RebootDevice(1);

// 错误恢复重启
RebootDevice(2);
```

**证据**: `reset.h:43-51`

## 复位原因码定义

复位原因码由系统定义，具体含义如下：

| 原因码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `RESET_CAUSE_NONE` | 无特定原因 |
| 1 | `RESET_CAUSE_OTA` | OTA 升级完成 |
| 2 | `RESET_CAUSE_ERROR` | 错误恢复 |
| 3 | `RESET_CAUSE_POWER` | 电源相关 |
| ... | ... | 具体值取决于芯片实现 |

**注意**: 具体的原因码定义需参考 Hi3861 芯片手册或 HAL 实现。

## 使用场景

### 1. OTA 升级后重启

```c
void ota_complete_callback(void) {
    // 保存升级状态
    save_ota_status(OTA_STATUS_COMPLETE);

    // 触发重启以应用新固件
    RebootDevice(1);  // OTA 升级完成
}
```

### 2. 错误恢复

```c
void fatal_error_handler(void) {
    // 记录错误日志
    log_error_to_flash();

    // 尝试恢复
    if (!attempt_recovery()) {
        // 恢复失败，触发复位
        RebootDevice(2);  // 错误恢复
    }
}
```

### 3. 电源管理

```c
void power_button_long_press(void) {
    // 关机前保存状态
    save_system_state();

    // 触发复位
    RebootDevice(3);  // 电源相关
}
```

## 注意事项

### 1. 数据保存

```c
void safe_reboot(unsigned int cause) {
    // 1. 保存关键数据到 Flash
    save_critical_data();

    // 2. 同步文件系统
    sync_filesystem();

    // 3. 触发复位
    RebootDevice(cause);
}
```

### 2. 复位前清理

```c
void clean_reboot(void) {
    // 关闭外设
    IoTGpioDeinit(GPIO_LED);
    IoTUartDeinit(UART_ID);
    IoTFlashDeinit();

    // 触发复位
    RebootDevice(0);
}
```

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [Watchdog 接口](API_Watchdog.md)
- [Lowpower 接口](API_Lowpower.md)
- [安全评审](05_Security_Review.md)
