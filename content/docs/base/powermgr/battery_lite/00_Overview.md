# OpenHarmony Lite Battery Manager 概览

> **目的**: 为 OpenHarmony 轻量级设备提供电池管理能力，支持 mini 和 small 系统。  
> **适用范围**: 基于 LiteOS-M 内核的轻量设备以及小型嵌入式系统。  
> **关键结论**: 本组件是电源管理子系统的轻量级实现，通过 SAMgr 提供 IPC 服务，支持 JS API 绑定。

---

## 项目定位

### 在电源管理子系统中的位置

```
电源管理子系统
├── powermgr_power_manager    # 电源管理（设备电源策略）
├── powermgr_display_manager  # 显示管理（屏幕亮度等）
├── powermgr_battery_manager  # 完整电池管理（标准系统）
├── powermgr_battery_lite     # 轻量电池管理 ← 本组件
├── powermgr_thermal_manager  # 温控管理
└── powermgr_battery_statistics # 电池统计
```

**证据**: `README_zh.md:13-18` 列出电池服务组件提供四项核心功能。

### 与标准电池管理器的差异

| 特性 | battery_lite | battery_manager |
|------|--------------|-----------------|
| 适用系统 | mini/small | standard |
| 架构 | SAMgr + Service | 更多 IPC 机制 |
| API 风格 | 简单查询 | 完整回调/监听 |
| 资源占用 | ~32KB ROM/RAM | 更大资源占用 |

---

## 核心能力

### 1. 电池信息查询

| 功能 | API | 返回类型 | 说明 |
|------|-----|----------|------|
| 获取电池电量 | `GetBatSoc()` | `int32_t` | 0-100% |
| 获取电池电压 | `GetBatVoltage()` | `int32_t` | 单位 mV |
| 获取电池温度 | `GetBatTemperature()` | `int32_t` | 单位 0.1℃ |
| 获取电池技术 | `GetBatTechnology()` | `char*` | 电池型号字符串 |

**证据**: `interfaces/kits/battery_info.h:100-106` 定义了七个公共查询接口。

### 2. 充放电状态监测

| 功能 | API | 枚举值 | 说明 |
|------|-----|--------|------|
| 充电状态 | `GetChargingStatus()` | `CHARGE_STATE_NONE/ENABLE/DISABLE/FULL` | 放电/充电/未充满/充满 |
| 连接类型 | `GetPluggedType()` | `PLUGGED_TYPE_NONE/AC/USB/WIRELESS` | 无/交流/USB/无线 |

**证据**: `interfaces/kits/battery_info.h:23-98` 定义了三个枚举类型。

### 3. 电池健康监控

| 功能 | API | 枚举值 | 说明 |
|------|-----|--------|------|
| 健康状态 | `GetHealthStatus()` | `HEALTH_STATE_UNKNOWN/GOOD/OVERHEAT/OVERVOLTAGE/COLD/DEAD` | 未知/良好/过热/过压/过冷/失效 |

**证据**: `interfaces/kits/battery_info.h:46-75` 定义了六种健康状态。

### 4. 充电指示灯控制

| 功能 | API | 参数 | 返回值 |
|------|-----|------|--------|
| 点亮 LED | `TurnOnLed()` | 红、绿、蓝强度 (0-255) | `int` 错误码 |
| 关闭 LED | `TurnOffLed()` | 无 | `int` 错误码 |
| 设置颜色 | `SetLedColor()` | 红、绿、蓝强度 | `int` 错误码 |
| 获取颜色 | `GetLedColor()` | 输出指针参数 | `int` 错误码 |

**证据**: `services/include/battery_device.h:55-58` 定义了四个 LED 控制接口。

---

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| 内核 | LiteOS-M |
| 系统类型 | mini / small |
| 依赖组件 | utils_lite, samgr_lite, ipc, hilog_lite |

**证据**: `bundle.json:27-30` 指定了适配的 `adapted_system_type` 为 `mini` 和 `small`。

### 资源占用

| 资源 | 大小 |
|------|------|
| ROM | 约 22KB |
| RAM | 约 10KB |

**证据**: `bundle.json:31-32` 声明了 `rom: "22KB"` 和 `ram: "~10KB"`。

### 系统能力

```
SystemCapability.PowerManager.BatteryManager.Lite
```

**证据**: `bundle.json:24` 声明了 `syscap` 为 `SystemCapability.PowerManager.BatteryManager.Lite`。

---

## 关键概念

### SAMgr (Service Ability Manager)

SAMgr 是 OpenHarmony Lite 系统的核心服务框架，负责管理服务和特征的注册、发现、调用。

**电池服务的 SAMgr 配置**:
- **Service 名称**: `"battery_service"`
- **Feature 名称**: `"battery_feature"`
- **Task 配置**: `LEVEL_HIGH`, `PRIORITY_BELOW_NORMAL`, `STACK_SIZE: 0x800`, `QUEUE_SIZE: 20`

**证据**: 
- `frameworks/native/include/battery_mgr.h:24-25` 定义服务名常量
- `services/include/battery_device.h:42-43` 定义任务配置常量

### IBattery 接口

电池服务对外暴露的抽象接口，包含所有电池操作的函数指针。

```c
typedef struct IBattery {
    int32_t (*GetSoc)();
    BatteryChargeState (*GetChargingStatus)();
    BatteryHealthState (*GetHealthStatus)();
    BatteryPluggedType (*GetPluggedType)();
    int32_t (*GetVoltage)();
    char* (*GetTechnology)();
    int32_t (*GetTemperature)();
    int (*TurnOnLed)(int, int, int);
    int (*TurnOffLed)();
    int (*SetLedColor)(int, int, int);
    int (*GetLedColor)(int*, int*, int*);
    void (*ShutDown)();
    void (*UpdateBatInfo)(BatInfo*);
} IBattery;
```

**证据**: `services/include/ibattery.h:45-59` 定义了完整的 IBattery 接口结构。

### BatInfo 数据结构

电池信息的完整数据结构，用于在服务间传递电池状态。

```c
typedef struct {
    int32_t batSoc;              // 电池电量
    int32_t batVoltage;          // 电池电压
    int32_t BatTemp;             // 电池温度
    int32_t batCapacity;         // 电池容量
    BatteryChargeState chargingStatus;  // 充电状态
    BatteryPluggedType pluggedType;     // 连接类型
    char BatTechnology[64];      // 电池技术
    BatteryHealthState healthStatus;     // 健康状态
} BatInfo;
```

**证据**: `services/include/ibattery.h:26-43` 定义了完整的 BatInfo 结构。

---

## 快速开始

### Native API 调用

```c
#include "battery_info.h"

// 获取电池电量
int32_t soc = GetBatSoc();  // 返回 0-100

// 获取充电状态
BatteryChargeState state = GetChargingStatus();

// 获取电池健康状态
BatteryHealthState health = GetHealthStatus();
```

### JS API 调用

```javascript
import battery from '@system.battery';

// 获取电池电量
battery.BatterySOC({
    success: (data) => {
        console.log('Battery SoC:', data.batterySoc);
    },
    fail: (data, code) => {
        console.error('Failed:', data, code);
    }
});
```

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [目录结构](01_Directory_Structure.md) | 详细代码组织结构 |
| [架构说明](02_Architecture.md) | 组件图和数据流 |
| [N-API 接口](03_N_API.md) | 完整 API 清单 |
| [GN 构建](05_GN_Build.md) | 构建 Targets 和产物 |
| [安全评审](06_Security_Review.md) | 安全风险分析 |
