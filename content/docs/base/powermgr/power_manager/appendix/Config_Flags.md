# Feature Flags 配置清单

本文档列出 Power Manager 模块的所有 Feature Flags 及其配置说明。

## 1. 特性开关列表

### 1.1 核心功能

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_runninglock` | true | 启用运行锁功能 |
| `power_manager_feature_shutdown_reboot` | true | 启用关机/重启功能 |
| `power_manager_feature_screen_on_off` | true | 启用亮灭屏功能 |
| `power_manager_feature_power_state` | true | 启用电源状态管理 |
| `power_manager_feature_power_mode` | true | 启用电源模式功能 |

### 1.2 电源键

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_allow_interrupting_powerkey_off` | true | 允许电源键中断关机 |
| `power_manager_feature_block_long_press` | false | 阻止电源键长按 |

### 1.3 待机休眠

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_enable_s4` | false | 启用 S4 休眠状态 |
| `power_manager_feature_enable_suspend_with_tag` | false | 启用带标签挂起 |
| `power_manager_feature_force_sleep_broadcast` | false | 强制休眠广播 |

### 1.4 唤醒源

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_doubleclick` | true | 双击唤醒 |
| `power_manager_feature_pickup` | true | 抬腕唤醒 |
| `power_manager_feature_movement` | true | 运动唤醒 |
| `power_manager_feature_wakeup_action` | false | 唤醒动作 |

### 1.5 屏幕相关

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_disable_auto_displayoff` | false | 禁用自动灭屏 |
| `power_manager_feature_report_screenoff_invalid` | true | 报告无效灭屏 |
| `power_manager_feature_screen_on_timeout_check` | false | 屏幕超时检查 |

### 1.6 电源模式

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_tv_dreaming` | false | TV Dreaming |
| `power_manager_feature_poweroff_charge` | false | 关机充电模式 |

### 1.7 运行锁

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_audio_lock_unproxy` | false | 音频锁不代理 |
| `power_manager_feature_runninglock_background_user_idle_permissive_permission_mode` | false | 后台用户空闲锁宽松权限模式 |

### 1.8 关机相关

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_judging_takeover_shutdown` | false | 判断接管关机 |
| `power_manager_feature_surport_takeover_suspend` | false | 支持接管挂起 |
| `power_manager_feature_watch_limit_screen_common_event_publish` | false | 限屏公共事件发布 |
| `power_manager_feature_watch_update_adapt` | false | 手表更新适配 |

### 1.9 设备状态

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_enable_lid_check` | false | 启用翻盖检测 |
| `power_manager_feature_lid_fold` | false | 翻盖状态 |
| `power_manager_feature_external_screen_management` | false | 外屏管理 |
| `power_manager_feature_proximity_controller_override` | false | 距离控制器覆盖 |

### 1.10 ULSR 插件

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_enable_ulsr_plugin` | false | 启用 ULSR 插件 |

### 1.11 性能模式

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_enable_performance_mode` | false | 启用性能模式 |
| `power_manager_feature_charging_type_setting` | false | 充电类型设置 |

### 1.12 其他

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_watch_boot_completed` | false | 手表启动完成 |
| `power_manager_feature_support_filtering_proximity_event` | false | 支持过滤距离事件 |

---

## 2. 配置示例

### 2.1 在 product_name.gni 中配置

```gni
# 启用所有功能
power_manager_feature_runninglock = true
power_manager_feature_shutdown_reboot = true
power_manager_feature_screen_on_off = true
power_manager_feature_power_state = true
power_manager_feature_power_mode = true

# 启用唤醒特性
power_manager_feature_doubleclick = true
power_manager_feature_pickup = true
power_manager_feature_movement = true

# 禁用关机充电
power_manager_feature_poweroff_charge = false
```

### 2.2 条件编译

```gni
# 启用 S4 休眠
if (power_manager_feature_enable_s4) {
  defines += [ "POWER_MANAGER_POWER_ENABLE_S4" ]
}

# 启用带标签挂起
if (power_manager_feature_enable_suspend_with_tag) {
  defines += [ "POWER_MANAGER_ENABLE_SUSPEND_WITH_TAG" ]
}

# 音频锁不代理
if (power_manager_feature_audio_lock_unproxy) {
  defines += [ "POWER_MANAGER_AUDIO_LOCK_UNPROXY" ]
}

# 距离控制器覆盖
if (power_manager_feature_proximity_controller_override) {
  defines += [ "POWER_MANAGER_PROXIMITY_CONTROLLER_OVERRIDE" ]
}

# 翻盖检测
if (power_manager_feature_enable_lid_check) {
  defines += [ "POWER_MANAGER_ENABLE_LID_CHECK" ]
}

# 后台用户空闲锁宽松权限
if (power_manager_feature_runninglock_background_user_idle_permissive_permission_mode) {
  defines += [ "POWER_MANAGER_RUNNINGLOCK_BACKGROUND_USER_IDLE_PERMISSION_PERMISSIVE_MODE" ]
}

# 支持过滤距离事件
if (power_manager_feature_support_filtering_proximity_event) {
  defines += [ "POWER_MANAGER_SUPPORT_FILTERING_PROXIMITY_EVENT" ]
}
```

---

## 3. 依赖检测

### 3.1 组件检测

```gni
# 检测 display_manager 是否存在
if (!defined(global_parts_info) ||
    defined(global_parts_info.powermgr_display_manager)) {
  has_display_manager_part = true
  defines += [ "HAS_DISPLAY_MANAGER_PART" ]
} else {
  has_display_manager_part = false
}

# 检测 device_standby 是否存在
if (!defined(global_parts_info) ||
    defined(global_parts_info.resourceschedule_device_standby)) {
  has_device_standby_part = true
  defines += [ "HAS_DEVICE_STANDBY_PART" ]
} else {
  has_device_standby_part = false
}

# 检测 sensor 是否存在
if (!defined(global_parts_info) || defined(global_parts_info.sensors_sensor)) {
  has_sensors_sensor_part = true
  defines += [ "HAS_SENSORS_SENSOR_PART" ]
} else {
  has_sensors_sensor_part = false
}

# 检测 input 是否存在
if (!defined(global_parts_info) ||
    defined(global_parts_info.multimodalinput_input)) {
  has_multimodalinput_input_part = true
  defines += [ "HAS_MULTIMODALINPUT_INPUT_PART" ]
} else {
  has_multimodalinput_input_part = false
}
```

### 3.2 条件编译头文件

```cpp
// 显示管理
#ifdef HAS_DISPLAY_MANAGER_PART
#include "display_manager.h"
#endif

// 设备待机
#ifdef HAS_DEVICE_STANDBY_PART
#include "standby_service_client.h"
#endif

// 传感器
#ifdef HAS_SENSORS_SENSOR_PART
#include "sensor_manager.h"
#endif
```

---

## 4. 默认配置文件位置

```
powermgr.gni                    # 主配置
product_name/product_name.gni   # 产品配置
board/board_name/config.gni     # 板级配置
```

---

## 5. 验证配置

### 5.1 查看当前配置

```bash
# 生成构建后查看
gn args out/ --list | grep power_manager
```

### 5.2 验证编译产物

```bash
# 检查特定功能是否编译
nm -C out/libpowermgrservice.so | grep "S4\|SuspendWithTag"
```
