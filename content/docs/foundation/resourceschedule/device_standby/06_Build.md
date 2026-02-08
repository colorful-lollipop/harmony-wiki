# 编译构建

## 概述

Device Standby 部件使用 GN (Generate Ninja) 作为构建系统，通过 `bundle.json` 和 `BUILD.gn` 文件定义构建规则。

## 根构建配置

### bundle.json

**路径**：`bundle.json`

```json
{
  "name": "@ohos/device_standby",
  "subsystem": "resourceschedule",
  "version": "4.0",
  "syscap": [
    "SystemCapability.ResourceSchedule.DeviceStandby"
  ],
  "features": [
    "device_standby_plugin_enable",
    "device_standby_realtime_timer_enable",
    "device_standby_firewall_timer_no_wakeup"
  ]
}
```

### standby_service.gni

**路径**：`standby_service.gni`

定义全局变量和 feature flags：

```gni
# 路径变量
standby_service_root_path = "//foundation/resourceschedule/device_standby"
standby_interfaces_path = "${standby_service_root_path}/interfaces"
standby_plugins_path = "${standby_service_root_path}/plugins"
# ... 更多路径变量

# Feature Flags
device_standby_plugin_enable = true
device_standby_realtime_timer_enable = false
device_standby_firewall_timer_no_wakeup = false
```

## GN Targets 清单

### Base Group

基础组件：

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `//utils/common:standby_utils_common` | 静态库 | `.a` | 通用工具 |
| `//utils/policy:standby_utils_policy` | 静态库 | `.a` | 策略配置 |

### Framework Group

框架组件：

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `//frameworks:standby_fwk` | 共享库 | `.so` | API 框架 |
| `//interfaces:standby_interfaces` | 静态库 | `.a` | 接口定义 |
| `//interfaces/kits/ani:device_standby_taihe` | 共享库 | `.so`/`.ani` | ArkTS API |

### Service Group

服务组件：

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `//sa_profile:device_standby_sa_profile` | SA 配置 | `.json` | SA 配置 |
| `//interfaces/innerkits:standby_innerkits` | 静态库 | `.a` | Inner Kits |
| `//plugins:standby_plugin_group` | 共享库组 | `.so` | 插件组 |
| `//services:standby_service` | SA 共享库 | `.z.so` | 主服务 |
| `//utils/policy:standby_service_config` | 静态库 | `.a` | 服务配置 |

## 关键 Targets 详解

### standby_service

**路径**：`services/BUILD.gn`

**类型**：共享库 (SA)

**产物**：`libstandby_service.z.so`

**Sources**：

```gn
sources = [
  "common/src/device_standby_switch.cpp",
  "common/src/time_provider.cpp",
  "common/src/timed_task.cpp",
  "core/src/ability_manager_helper.cpp",
  "core/src/allow_record.cpp",
  "core/src/app_mgr_helper.cpp",
  "core/src/app_state_observer.cpp",
  "core/src/bundle_manager_helper.cpp",
  "core/src/common_event_observer.cpp",
  "core/src/standby_service.cpp",
  "core/src/standby_service_impl.cpp",
  "notification/src/standby_state_subscriber.cpp",
]
```

**Deps**：

```gn
deps = [
  "${standby_innerkits_path}:standby_innerkits",
  "${standby_service_frameworks_path}:standby_fwk",
  "${standby_utils_common_path}:standby_utils_common",
  "${standby_utils_policy_path}:standby_utils_policy",
]
```

**External Deps**：

```gn
external_deps = [
  "ability_base:want",
  "ability_runtime:app_manager",
  "access_token:libaccesstoken_sdk",
  "access_token:libtokenid_sdk",
  "bundle_framework:appexecfwk_base",
  "common_event_service:cesfwk_innerkits",
  "eventhandler:libeventhandler",
  "hilog:libhilog",
  "ipc:ipc_single",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
  # ... 更多依赖
]
```

### standby_plugin

**路径**：`plugins/BUILD.gn`

**类型**：共享库

**产物**：`libstandby_plugin.z.so`

**Sources**：

```gn
StandbyPluginSrc = [
  # ext
  "${standby_plugins_path}/ext/src/base_state.cpp",
  "${standby_plugins_path}/ext/src/istate_manager_adapter.cpp",
  # standby_state
  "${standby_plugins_path}/standby_state/src/dark_state.cpp",
  "${standby_plugins_path}/standby_state/src/maintenance_state.cpp",
  "${standby_plugins_path}/standby_state/src/nap_state.cpp",
  "${standby_plugins_path}/standby_state/src/sleep_state.cpp",
  "${standby_plugins_path}/standby_state/src/state_manager_adapter.cpp",
  "${standby_plugins_path}/standby_state/src/working_state.cpp",
  # strategy
  "${standby_plugins_path}/strategy/src/network_strategy.cpp",
  "${standby_plugins_path}/strategy/src/running_lock_strategy.cpp",
  "${standby_plugins_path}/strategy/src/timer_strategy.cpp",
  # message_listener
  "${standby_plugins_path}/message_listener/src/listener_manager_adapter.cpp",
]
```

### device_standby_ani

**路径**：`interfaces/kits/ani/BUILD.gn`

**类型**：Taihe 共享库

**产物**：
- `.ani.cpp` (自动生成)
- `.abi.c` (自动生成)
- `.so` 库

**Sources** (自动生成 + 手动)：

```gn
sources = get_target_outputs(":run_taihe")  // 自动生成文件
sources += [
  "src/ani_constructor.cpp",
  "src/ohos.deviceStandby.impl.cpp",
]
```

### device_standby_abc

**路径**：`interfaces/kits/ani/BUILD.gn`

**类型**：静态 ABC

**产物**：`device_standby_abc.abc`

**安装路径**：`/system/framework/device_standby_abc.abc`

## Feature Flags

### 编译时开关

| 开关 | 默认值 | 定义位置 | 说明 |
|------|--------|----------|------|
| `device_standby_plugin_enable` | true | `standby_service.gni:102` | 是否编译插件 |
| `device_standby_realtime_timer_enable` | false | `standby_service.gni:103` | 实时定时器支持 |
| `device_standby_firewall_timer_no_wakeup` | false | `standby_service.gni:104` | 防火墙定时器唤醒 |

### 条件编译

```gni
# 背景任务管理
if (enable_background_task_mgr) {
  sources += [ "common/src/background_task_helper.cpp" ]
  defines += [ "ENABLE_BACKGROUND_TASK_MGR" ]
}

# 电源管理
if (standby_power_manager_enable) {
  external_deps += [ "power_manager:powermgr_client" ]
  defines += [ "STANDBY_POWER_MANAGER_ENABLE" ]
}

# 电池管理
if (standby_battery_manager_enable) {
  external_deps += [ "battery_manager:batterysrv_client" ]
  defines += [ "STANDBY_BATTERY_MANAGER_ENABLE" ]
}
```

## 产物清单

| 产物 | 路径 | 类型 | 用途 |
|------|------|------|------|
| `libstandby_service.z.so` | system/lib64/ | SA 库 | 主服务 |
| `libstandby_plugin.z.so` | system/lib64/ | 插件库 | 状态/策略 |
| `device_standby_abc.abc` | system/framework/ | ABC | ArkTS API |
| `device_standby_ani.z.so` | system/lib64/ | Taihe | ArkTS native |

## 编译命令

```bash
# 编译整个子系统
./build.sh --product <product_name> --build-target resourceschedule_device_standby

# 单独编译
hb build -p <product_name> -T //foundation/resourceschedule/device_standby/services:standby_service

# 编译插件
hb build -p <product_name> -T //foundation/resourceschedule/device_standby/plugins:standby_plugin
```

## 资源消耗

| 指标 | 估算值 |
|------|--------|
| ROM | 2048 KB |
| RAM | 10240 KB |

## 注意事项

1. **SA 库命名**：服务产物必须以 `.z.so` 结尾
2. **版本脚本**：`standby_service` 使用 `libstandby_service.versionscript`
3. **CFI 保护**：所有库启用 CFI (Control Flow Integrity) 保护
4. **PAC/RET**：启用分支保护
