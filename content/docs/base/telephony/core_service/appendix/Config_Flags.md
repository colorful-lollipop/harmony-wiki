# 附录：配置与开关

## 目的

本文档汇总 `telephony_core_service` 的所有编译开关、Feature Flags 和运行时配置。

---

## 编译开关 (GN Args)

### core_service_support_esim

| 属性 | 说明 |
|------|------|
| 默认值 | `false` |
| 位置 | `telephony_core_service.gni` |
| 用途 | 启用 eSIM 功能支持 |

**影响的代码**:
```cpp
#ifdef CORE_SERVICE_SUPPORT_ESIM
    esimManager_ = std::make_shared<EsimManager>(telRilManager_);
    esimManager_->OnInit(slotCount);
#endif
```

**影响的源文件**:
- `services/sim/src/esim_*.cpp`
- `utils/codec/*.cpp`
- `utils/vcard/*.cpp`

### core_service_low_power_class_2

| 属性 | 说明 |
|------|------|
| 默认值 | `false` |
| 位置 | `telephony_core_service.gni` |
| 用途 | 启用低功耗模式 |

**影响**:
```gn
if (!core_service_low_power_class_2) {
  if (deps_netmanager_ext) {
    external_deps += [ "netmanager_ext:net_tether_manager_if" ]
  }
}
```

### core_service_satellite

| 属性 | 说明 |
|------|------|
| 默认值 | `false` |
| 位置 | `telephony_core_service.gni` |
| 用途 | 启用卫星通信支持 |

**影响的源文件**:
- `services/satellite_service_interaction/src/*.cpp`

### telephony_hicollie_able

| 属性 | 说明 |
|------|------|
| 默认值 | `false` (推断) |
| 位置 | `BUILD.gn:258` |
| 用途 | 启用 HiCollie 卡顿检测 |

```gn
if (telephony_hicollie_able) {
  external_deps += [ "hicollie:libhicollie" ]
  defines += [ "HICOLLIE_ENABLE" ]
}
```

---

## 全局部件依赖开关

### ABILITY_POWER_SUPPORT

**位置**: `BUILD.gn:236-239`

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.powermgr_power_manager) &&
    global_parts_info.powermgr_power_manager) {
  external_deps += [ "power_manager:powermgr_client" ]
  defines += [ "ABILITY_POWER_SUPPORT" ]
}
```

### ABILITY_BATTERY_SUPPORT

**位置**: `BUILD.gn:241-246`

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.powermgr_battery_manager) &&
    global_parts_info.powermgr_battery_manager) {
  external_deps += [ "battery_manager:batterysrv_client" ]
  defines += [ "ABILITY_BATTERY_SUPPORT" ]
}
```

### ABILITY_LOCATION_SUPPORT

**位置**: `BUILD.gn:248-256`

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.location_location) &&
    global_parts_info.location_location) {
  external_deps += [ "location:lbsservice_common", "location:locator_sdk" ]
  defines += [ "ABILITY_LOCATION_SUPPORT" ]
}
```

---

## 运行时配置

### FFRT 线程数配置

**代码位置**: `services/core/src/core_service.cpp:46,72`

```cpp
const int32_t MAX_FFRT_THREAD_NUM = 32;
// OnStart 中设置
ffrt_set_cpu_worker_max_num(ffrt::qos_default, MAX_FFRT_THREAD_NUM);
```

### 日志级别

**代码位置**: `utils/log/include/telephony_log_wrapper.h`

```cpp
#define TELEPHONY_LOGV(...) HILOG_DEBUG(LOG_CORE, __VA_ARGS__)
#define TELEPHONY_LOGD(...) HILOG_DEBUG(LOG_CORE, __VA_ARGS__)
#define TELEPHONY_LOGI(...) HILOG_INFO(LOG_CORE, __VA_ARGS__)
#define TELEPHONY_LOGW(...) HILOG_WARN(LOG_CORE, __VA_ARGS__)
#define TELEPHONY_LOGE(...) HILOG_ERROR(LOG_CORE, __VA_ARGS__)
```

**日志 Domain**: `0xD001F04`  
**日志 Tag**: 各模块不同，如 `CoreService`, `CoreServiceApi`

### 系统参数

**telephony 相关参数** (`services/etc/param/telephony.para`):

```
# 示例参数
telephony.debug.enable=false
telephony.test.mode=false
```

---

## 版本脚本符号控制

### libtel_core_service.versionscript

控制动态库导出符号:

```ld
{
  global:
    *;
  local:
    *;
};
```

### libtel_core_service_api.versionscript

控制 API 库导出符号。

---

## Syscap 定义

**bundle.json:22-25**:

```json
"syscap": [
    "SystemCapability.Telephony.CoreService",
    "SystemCapability.Telephony.CoreService.Esim"
]
```

---

## 相关链接

- [GN 构建](../05_GN_Build.md)
- [安全风险](../06_Security.md)
