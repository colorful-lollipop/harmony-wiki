# 配置开关说明

## 概述

本文档详细说明 Device Standby 部件的所有 Feature Flags 和编译配置选项。

## Feature Flags 分类

| 分类 | 数量 | 说明 |
|------|------|------|
| 功能开关 | 3 | 控制主要功能模块 |
| 条件编译 | 9+ | 根据依赖组件启用功能 |
| 构建配置 | 5+ | 构建系统配置 |

## 功能开关 (Feature Flags)

### device_standby_plugin_enable

| 属性 | 值 |
|------|-----|
| 默认值 | `true` |
| 定义位置 | `standby_service.gni:102` |
| 影响范围 | 插件编译和加载 |

```gni
device_standby_plugin_enable = true
```

**作用**：
- 控制 `standby_plugin` 是否编译
- 影响 `//plugins/BUILD.gn:190-193`

```gn
group("standby_plugin_group") {
  deps = []
  if (device_standby_plugin_enable) {
    deps += [ ":standby_plugin" ]
  }
}
```

### device_standby_realtime_timer_enable

| 属性 | 值 |
|------|-----|
| 默认值 | `false` |
| 定义位置 | `standby_service.gni:103` |
| 影响范围 | 实时定时器支持 |

```gni
device_standby_realtime_timer_enable = false
```

**作用**：
- 启用实时定时器支持
- 定义宏：`STANDBY_REALTIME_TIMER_ENABLE`

### device_standby_firewall_timer_no_wakeup

| 属性 | 值 |
|------|-----|
| 默认值 | `false` |
| 定义位置 | `standby_service.gni:104` |
| 影响范围 | 防火墙定时器唤醒 |

```gni
device_standby_firewall_timer_no_wakeup = false
```

**作用**：
- 禁用防火墙定时器唤醒
- 定义宏：`STANDBY_FIREWALL_TIMER_NO_WAKEUP`

## 条件编译配置

### 配置文件策略

| 配置项 | 默认值 | 定义位置 | 说明 |
|--------|--------|----------|------|
| `enable_standby_configpolicy` | `true` | `standby_service.gni:47-51` | 启用配置策略 |

```gni
declare_args() {
  enable_standby_configpolicy = true
  if (defined(global_parts_info) &&
      !defined(global_parts_info.customization_config_policy)) {
    enable_standby_configpolicy = false
  }
}
```

**作用**：
- 启用 `config_policy` 组件依赖
- 定义宏：`STANDBY_CONFIG_POLICY_ENABLE`

### 后台任务管理

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `enable_background_task_mgr` | `true` | `standby_service.gni:53-57` |

```gni
if (enable_background_task_mgr) {
  external_deps += [ "background_task_mgr:bgtaskmgr_innerkits" ]
  StandbyPluginSrc += [ "message_listener/src/background_task_listener.cpp" ]
  defines += [ "ENABLE_BACKGROUND_TASK_MGR" ]
}
```

### 电源管理

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `standby_power_manager_enable` | `true` | `standby_service.gni:59-63` |

### 电池管理

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `standby_battery_manager_enable` | `true` | `standby_service.gni:65-69` |

### 多模输入

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `standby_multimodalinput_input_enable` | `true` | `standby_service.gni:71-75` |

### 传感器

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `standby_sensors_sensor_enable` | `true` | `standby_service.gni:77-81` |

### 网络管理

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `standby_communication_netmanager_base_enable` | `true` | `standby_service.gni:83-87` |

### 工作调度

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `standby_rss_work_scheduler_enable` | `true` | `standby_service.gni:89-93` |

### 访问令牌

| 配置项 | 默认值 | 定义位置 |
|--------|--------|----------|
| `device_standby_access_token_enable` | `true` | `standby_service.gni:95-99` |

## 构建配置

### CFI 保护

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sanitize.cfi` | `true` | 启用 Control Flow Integrity |
| `sanitize.cfI_cross_dso` | `true` | 跨 DSO CFI 保护 |
| `branch_protector_ret` | `"pac_ret"` | PACRET 分支保护 |

```gn
ohos_shared_library("standby_service") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
}
```

### 编译优化选项

```gn
cflags_cc = [
  "-fdata-sections",
  "-ffunction-sections",
  "-fno-rtti",
  "-fno-exceptions",
  "-fno-unwind-tables",
  "-fstack-protector-strong",
  "-Os",
]
```

## 配置组合矩阵

| 场景 | plugin_enable | realtime_timer | firewall_no_wakeup |
|------|---------------|----------------|-------------------|
| 标准版 | true | false | false |
| 实时版 | true | true | false |
| 精简版 | false | false | false |

## 配置验证

### 检查配置是否生效

```bash
# 查看编译时定义的宏
grep -r "define\|ifdef" out/compile_commands.json | grep STANDBY

# 验证插件是否加载
hilog | grep standby_plugin
```
