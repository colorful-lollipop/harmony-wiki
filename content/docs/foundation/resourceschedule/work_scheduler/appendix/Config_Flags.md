# 配置标志与宏定义

> **目的**: 列出 Work Scheduler 模块的所有编译开关和宏定义
> **适用范围**: 构建系统定制、功能裁剪、平台适配

---

## Feature Flags 汇总

### 全局开关

| Flag | 定义位置 | 默认值 | 控制的功能 |
|-------|----------|---------|-----------|
| `work_scheduler_device_enable` | `workscheduler.gni:42` | true | 全局开关 |
| `bundle_active_enable` | `workscheduler.gni:44` | depends | 应用活跃度频率控制 |
| `device_standby_enable` | `workscheduler.gni:50` | depends | 待机状态监听 |
| `resourceschedule_bgtaskmgr_enable` | `workscheduler.gni:56` | depends | 后台任务订阅 |
| `workscheduler_with_communication_netmanager_base_enable` | `workscheduler.gni:62` | depends | 网络状态监听 |
| `powermgr_battery_manager_enable` | `workscheduler.gni:68` | depends | 电池状态监听 |
| `powermgr_thermal_manager_enable` | `workscheduler.gni:74` | depends | 温度策略过滤 |
| `powermgr_power_manager_enable` | `workscheduler.gni:80` | depends | 功耗模式策略 |
| `workscheduler_hicollie_enable` | `workscheduler.gni:86` | depends | HiCollie 故障收集 |

---

## 详细 Flag 说明

### work_scheduler_device_enable

**定义**:
```gni
declare_args() {
  work_scheduler_device_enable = true
}
```

**影响**:
- ✅ 为 `true`: 编译所有 Work Scheduler 组件
- ❌ 为 `false`: 跳过所有 Work Scheduler 编译

**使用位置**:
- 根 `BUILD.gn` 的所有 `if (work_scheduler_device_enable)` 检查

---

### bundle_active_enable

**定义**:
```gni
bundle_active_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.resourceschedule_device_usage_statistics)) {
  bundle_active_enable = false
}
```

**影响**:
- ✅ 为 `true`: 启用应用活跃度分组和频率控制
- ❌ 为 `false`: 禁用频率控制，所有应用使用默认间隔

**相关代码**:
```cpp
// services/BUILD.gn:17-20
if (bundle_active_enable) {
  external_deps += [ "device_usage_statistics:usagestatsinner" ]
  defines += [ "DEVICE_USAGE_STATISTICS_ENABLE" ]
}
```

**功能**:
- 根据 Device Usage Statistics 返回的应用分组（active/daily/fixed/rare/restricted/unused）
- 应用不同的最小执行间隔（2/4/24/48/禁止）

**证据**: `services/BUILD.gn:17-20` + `services/native/src/work_sched_config.cpp`

---

### device_standby_enable

**定义**:
```gni
device_standby_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.resourceschedule_device_standby)) {
  device_standby_enable = false
}
```

**影响**:
- ✅ 为 `true`: 监听设备待机状态变化
- ❌ 为 `false`: 禁用待机协同

**相关代码**:
```cpp
// services/BUILD.gn:21-24
if (device_standby_enable) {
  external_deps += [ "device_standby:standby_innerkits" ]
  defines += [ "DEVICE_STANDBY_ENABLE" ]
}

// services/native/src/work_scheduler_service.cpp
#ifdef DEVICE_STANDBY_ENABLE
  AddSystemAbilityListener(DEVICE_STANDBY_SERVICE_SYSTEM_ABILITY_ID);
#endif
```

**功能**:
- 待机时不触发任务
- 设备退出待机后恢复任务检查

---

### resourceschedule_bgtaskmgr_enable

**定义**:
```gni
resourceschedule_bgtaskmgr_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.resourceschedule_background_task_mgr)) {
  resourceschedule_bgtaskmgr_enable = false
}
```

**影响**:
- ✅ 为 `true`: 订阅后台任务状态变化
- ❌ 为 `false`: 禁用后台任务订阅

**相关代码**:
```cpp
// services/BUILD.gn:25-28
if (resourceschedule_bgtaskmgr_enable) {
  external_deps += [ "background_task_mgr:bgtaskmgr_innerkits" ]
  defines += [ "RESOURCESCHEDULE_BGTASKMGR_ENABLE" ]
}
```

**功能**:
- 后台任务运行时暂停 Work Scheduler 任务
- 后台任务结束时恢复任务

---

### workscheduler_with_communication_netmanager_base_enable

**定义**:
```gni
workscheduler_with_communication_netmanager_base_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.communication_netmanager_base)) {
  workscheduler_with_communication_netmanager_base_enable = false
}
```

**影响**:
- ✅ 为 `true`: 监听网络状态变化
- ❌ 为 `false`: 禁用网络条件检查

**相关代码**:
```cpp
// services/BUILD.gn:42-45
if (workscheduler_with_communication_netmanager_base_enable) {
  defines += [ "COMMUNICATION_NETMANAGER_BASE_ENABLE" ]
  external_deps += [ "netmanager_base:net_conn_manager_if" ]
}
```

**功能**:
- 监听网络类型变化（WiFi/移动网络/蓝牙等）
- 支持的网络类型过滤

---

### powermgr_battery_manager_enable

**定义**:
```gni
powermgr_battery_manager_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.powermgr_battery_manager)) {
  powermgr_battery_manager_enable = false
}
```

**影响**:
- ✅ 为 `true`: 监听电池状态和电量变化
- ❌ 为 `false`: 禁用电池条件检查

**相关代码**:
```cpp
// services/BUILD.gn:29-32
if (powermgr_battery_manager_enable) {
  external_deps += [ "battery_manager:batterysrv_client" ]
  defines += [ "POWERMGR_BATTERY_MANAGER_ENABLE" ]
}
```

**功能**:
- 支持 batteryStatus 条件
- 支持 batteryLevel 条件

---

### powermgr_thermal_manager_enable

**定义**:
```gni
powermgr_thermal_manager_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.powermgr_thermal_manager)) {
  powermgr_thermal_manager_enable = false
}
```

**影响**:
- ✅ 为 `true`: 启用温度策略过滤
- ❌ 为 `false`: 禁用温度检查

**相关代码**:
```cpp
// services/BUILD.gn:33-36
if (powermgr_thermal_manager_enable) {
  external_deps += [ "thermal_manager:thermalsrv_client" ]
  defines += [ "POWERMGR_THERMAL_MANAGER_ENABLE" ]
}
```

**功能**:
- 根据温度决定是否执行任务
- 避免在设备过热时执行任务

---

### powermgr_power_manager_enable

**定义**:
```gni
powermgr_power_manager_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.powermgr_power_manager)) {
  powermgr_power_manager_enable = false
}
```

**影响**:
- ✅ 为 `true`: 启用功耗模式策略过滤
- ❌ 为 `false`: 禁用功耗模式检查

**相关代码**:
```cpp
// services/BUILD.gn:37-41
if (powermgr_power_manager_enable) {
  external_deps += [ "power_manager:powermgr_client" ]
  defines += [ "POWERMGR_POWER_MANAGER_ENABLE" ]
  sources += [ "native/src/policy/power_mode_policy.cpp" ]
}
```

**功能**:
- 监听功耗模式变化（节能/性能模式）
- 节能模式时拒绝任务执行

---

### workscheduler_hicollie_enable

**定义**:
```gni
workscheduler_hicollie_enable = true
if (defined(global_parts_info) &&
    !defined(global_parts_info.hiviewdfx_hicollie)) {
  workscheduler_hicollie_enable = false
}
```

**影响**:
- ✅ 为 `true`: 启用 HiCollie 故障收集
- ❌ 为 `false`: 禁用 HiCollie

**相关代码**:
```cpp
// services/BUILD.gn:46-49
if (workscheduler_hicollie_enable) {
  external_deps += [ "hicollie:libhicollie" ]
  defines += [ "HICOLLIE_ENABLE" ]
}
```

**功能**:
- 收集任务执行统计
- 收集故障信息
- 用于性能分析和问题定位

---

## 常量定义

### 时间常量

| 常量名 | 值 | 说明 | 定义位置 |
|---------|-----|------|----------|
| `WATCHDOG_TIME` | 120 (秒) | 任务最大执行时长 | `work_sched_constants.h` |
| `WORKSCHEDULER_SERVICE_NAME` | "WorkScheduler" | 服务名称 | `work_sched_constants.h` |
| `WORK_SCHEDULE_SERVICE_ID` | 1904 | SA ID | `work_sched_constants.h` |
| `MAX_REPEAT_CYCLE_TIME` | - | 最大重复周期（待确认） | TODO |

### 日志常量

| 常量名 | 值 | 说明 | 定义位置 |
|---------|-----|------|----------|
| `WORK_SCHED_HILOG_DOMAIN` | 0xD001712 | 日志域 | `work_sched_hilog.h` |
| `WORK_SCHED_HILOG_TAG` | "WORK_SCHEDULER" | 日志标签 | `work_sched_hilog.h` |

### 路径常量

| 常量名 | 值 | 说明 | 定义位置 |
|---------|-----|------|----------|
| `WORK_SCHED_LIB_NAME` | "libworkscheduler.so" | 主 N-API 库名 | `work_sched_constants.h` |
| `WORK_SCHED_SA_LIB_NAME` | "libworkschedservice.z.so" | SA 库名 | `work_sched_constants.h` |
| `DEFAULT_WORK_SCHEDULER_SERVICE_CONFIG` | "/system/profile/work_sched_config.json" | 配置文件路径 | `work_sched_config.cpp` |

---

## 编译宏定义

### 条件编译宏

这些宏在服务代码中用于功能裁剪：

| 宏 | 定义位置 | 控制的功能 |
|-----|----------|-----------|
| `DEVICE_USAGE_STATISTICS_ENABLE` | `services/BUILD.gn:19` | 应用活跃度 |
| `DEVICE_STANDBY_ENABLE` | `services/BUILD.gn:23` | 待机监听 |
| `RESOURCESCHEDULE_BGTASKMGR_ENABLE` | `services/BUILD.gn:27` | 后台任务订阅 |
| `COMMUNICATION_NETMANAGER_BASE_ENABLE` | `services/BUILD.gn:44` | 网络监听 |
| `POWERMGR_BATTERY_MANAGER_ENABLE` | `services/BUILD.gn:31` | 电池监听 |
| `POWERMGR_THERMAL_MANAGER_ENABLE` | `services/BUILD.gn:35` | 温度策略 |
| `POWERMGR_POWER_MANAGER_ENABLE` | `services/BUILD.gn:39` | 功耗模式 |
| `HICOLLIE_ENABLE` | `services/BUILD.gn:48` | HiCollie |

**使用示例**:
```cpp
// services/native/src/work_scheduler_service.cpp
#ifdef DEVICE_USAGE_STATISTICS_ENABLE
  // 应用活跃度相关代码
#endif

#ifdef DEVICE_STANDBY_ENABLE
  // 待机相关代码
#endif
```

---

## 配置文件

### SA 配置

**文件**: `sa_profile/1904.json`

```json
{
  "process": "resource_schedule_service",
  "systemability": [{
    "name": 1904,
    "libpath": "libworkschedservice.z.so",
    "run-on-create": true,
    "distributed": false,
    "dump_level": 1
  }]
}
```

**配置项说明**:
- `process`: SA 运行在的进程名
- `name`: SA ID
- `libpath`: 动态库路径
- `run-on-create`: 系统启动时自动创建
- `distributed`: 是否支持分布式
- `dump_level`: dump 级别

### HiSysEvent 配置

**文件**: `hisysevent.yaml`

**事件域**: `WORK_SCHEDULER`
**事件定义**:
- `WORK_SCHEDULER_START`: 任务启动
- `WORK_SCHEDULER_STOP`: 任务停止
- `WORK_SCHEDULER_TIMEOUT`: 任务超时
- `WORK_SCHEDULER_CONDITION_CHANGE`: 条件变化
- `WORK_SCHEDULER_POLICY_CHECK`: 策略检查

---

## 平台适配

### 依赖模块检查

Work Scheduler 通过 Feature Flags 自动适配不同平台的能力：

| Feature Flag | 依赖的 System Ability | 原因 |
|-------------|---------------------|------|
| `bundle_active_enable` | Device Usage Statistics (2704) | 应用活跃度统计 |
| `device_standby_enable` | Device Standby (2802) | 待机状态管理 |
| `resourceschedule_bgtaskmgr_enable` | Background Task Manager (2701) | 后台任务协同 |
| `workscheduler_with_communication_netmanager_base_enable` | Network Manager (601) | 网络状态 |
| `powermgr_battery_manager_enable` | Battery Manager (2301) | 电池状态 |
| `powermgr_thermal_manager_enable` | Thermal Manager (2302) | 温度状态 |
| `powermgr_power_manager_enable` | Power Manager (2304) | 功耗状态 |
| `workscheduler_hicollie_enable` | HiCollie (故障收集) | 故障上报 |

### 最小功能集

即使所有可选依赖禁用，Work Scheduler 仍能提供基础功能：

| 基础功能 | 说明 |
|-----------|------|
| 手动条件设置 | 网络、充电、屏幕、定时器（不依赖监听器） |
| 任务队列管理 | 添加、移除、查询任务 |
| Ability 启动 | 启动指定 Ability |
| Extension 回调 | 通知应用任务开始/停止 |
| Watchdog | 任务超时保护 |

---

## 自定义配置建议

### 添加新的 Feature Flag

1. 在 `workscheduler.gni` 中声明：
```gni
declare_args() {
  custom_feature_enable = true  # 添加自定义开关
}
```

2. 在 `services/BUILD.gn` 中使用：
```gn
if (custom_feature_enable) {
  external_deps += [ "custom_module:custom_kits" ]
  defines += [ "CUSTOM_FEATURE_ENABLE" ]
  sources += [ "native/src/custom_feature.cpp" ]
}
```

3. 在服务代码中使用条件编译：
```cpp
#ifdef CUSTOM_FEATURE_ENABLE
  // 自定义功能代码
#endif
```

---

## 相关跳转

- [06_GN_Targets.md](06_GN_Targets.md) - 构建系统详解
- [00_Overview.md](00_Overview.md) - 模块概览
- [03_Architecture.md](03_Architecture.md) - 架构设计

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| Feature Flags 定义 | `workscheduler.gni:42-91` |
| 编译宏使用 | `services/BUILD.gn:17-49` |
| 常量定义 | `utils/native/include/work_sched_constants.h` |
| SA 配置 | `sa_profile/1904.json` |
| HiSysEvent 配置 | `hisysevent.yaml` |
