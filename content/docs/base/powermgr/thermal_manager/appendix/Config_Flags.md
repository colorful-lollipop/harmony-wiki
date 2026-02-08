# 配置宏与 Feature Flags

## 目的

本文档说明 thermal_manager 的所有配置宏、Feature flags 和编译选项。

## 适用范围

- OpenHarmony thermal_manager 模块
- GN 构建配置
- 条件编译特性

## 相关文档

- [06_GN_Targets.md](06_GN_Targets.md) - GN Targets 详解
- [00_Overview.md](00_Overview.md) - 项目概览

---

## 全局 Feature Flags

### build flag: thermal_manager_audio_framework_enable

**定义位置**: `thermalmgr.gni:17`

```gni
declare_args() {
  thermal_manager_audio_framework_enable = false
}
```

**用途**: 控制是否启用音频框架集成

**影响代码**:
- `HAS_THERMAL_AUDIO_FRAMEWORK_PART` 宏定义
- 音频控制动作的编译

**依赖**: `audio_framework` subsystem

---

## 条件编译宏

### HAS_HIVIEWDFX_HISYSEVENT_PART

**定义位置**: `thermalmgr.gni:20-26`

```gni
if (!defined(global_parts_info) ||
    defined(global_parts_info.hiviewdfx_hisysevent)) {
  has_hiviewdfx_hisysevent_part = true
  defines += [ "HAS_HIVIEWDFX_HISYSEVENT_PART" ]
} else {
  has_hiviewdfx_hisysevent_part = false
}
```

**用途**: 控制是否启用 HiSysEvent 支持

**影响代码**:
- `#ifdef HAS_HIVIEWDFX_HISYSEVENT_PART`
- HiSysEvent 订阅和发布

**依赖**: `hisysevent` subsystem

---

### HAS_THERMAL_AIRPLANE_MANAGER_PART

**定义位置**: `thermalmgr.gni:28-33`

```gni
if (!defined(global_parts_info) ||
    defined(global_parts_info.communication_netmanager_base)) {
  has_thermal_airplane_manager_part = true
} else {
  has_thermal_airplane_manager_part = false
}
```

**用途**: 控制是否启用飞行模式管理

**影响代码**:
- `HAS_THERMAL_AIRPLANE_MANAGER_PART` 宏定义
- `AirplaneCommonEventSubscriber` 类编译
- 飞行模式相关动作编译

**依赖**: `netmanager_base` subsystem

---

### HAS_THERMAL_AUDIO_FRAMEWORK_PART

**定义位置**: `thermalmgr.gni:36-40`

```gni
if (!defined(global_parts_info) ||
    defined(global_parts_info.multimedia_audio_framework)) {
  has_thermal_audio_framework_part = true
} else {
  has_thermal_audio_framework_part = false
}

if (has_thermal_audio_framework_part &&
    thermal_manager_audio_framework_enable) {
  defines += [ "HAS_THERMAL_AUDIO_FRAMEWORK_PART" ]
  external_deps += [ "audio_framework:audio_client" ]
}
```

**用途**: 控制是否启用音频框架集成

**影响代码**:
- 音频控制动作（`ActionVolume`）编译
- `HAS_THERMAL_AUDIO_FRAMEWORK_PART` 宏定义

**依赖**: `audio_framework` subsystem + `thermal_manager_audio_framework_enable` flag

---

### HAS_THERMAL_DISPLAY_MANAGER_PART

**定义位置**: `thermalmgr.gni:43-48`

```gni
if (!defined(global_parts_info) ||
    defined(global_parts_info.powermgr_display_manager)) {
  has_thermal_display_manager_part = true
  defines += [ "HAS_THERMAL_DISPLAY_MANAGER_PART" ]
} else {
  has_thermal_display_manager_part = false
}
```

**用途**: 控制是否启用显示管理器集成

**影响代码**:
- 显示控制动作（`ActionDisplay`）编译
- `HAS_THERMAL_DISPLAY_MANAGER_PART` 宏定义

**依赖**: `display_manager` subsystem

---

### HAS_THERMAL_CONFIG_POLICY_PART

**定义位置**: `thermalmgr.gni:51-55`

```gni
if (!defined(global_parts_info) ||
    defined(global_parts_info.customization_config_policy)) {
  has_thermal_config_policy_part = true
  defines += [ "HAS_THERMAL_CONFIG_POLICY_PART" ]
} else {
  has_thermal_config_policy_part = false
}
```

**用途**: 控制是否启用配置策略框架

**影响代码**:
- 配置文件路径解析
- 使用 `configpolicy_util` 获取配置

**依赖**: `config_policy` subsystem

---

### SOC_PERF_ENABLE

**定义位置**: `services/BUILD.gn:51-55`

```gni
if (defined(global_parts_info) &&
    defined(global_parts_info.resourceschedule_soc_perf)) {
  external_deps += [ "soc_perf:socperf_client" ]
  defines += [ "SOC_PERF_ENABLE" ]
}
```

**用途**: 控制 SOC 性能动作的编译

**影响代码**:
- SOC 性能动作（CPU/GPU 提升、频率调节）编译

**依赖**: `resourceschedule_soc_perf` subsystem

---

### BATTERY_MANAGER_ENABLE

**定义位置**: `services/BUILD.gn:57-61`

```gni
if (defined(global_parts_info) &&
    defined(global_parts_info.powermgr_battery_manager)) {
  defines += [ "BATTERY_MANAGER_ENABLE" ]
  external_deps += [ "battery_manager:batterysrv_client" ]
}
```

**用途**: 控制是否启用电池管理器集成

**影响代码**:
- 充电状态收集
- 电池信息获取

**依赖**: `powermgr_battery_manager` subsystem

---

### DRIVERS_INTERFACE_BATTERY_ENABLE

**定义位置**: `services/BUILD.gn:64-67`

```gni
if (defined(global_parts_info) &&
    defined(global_parts_info.hdf_drivers_interface_battery)) {
  defines += [ "DRIVERS_INTERFACE_BATTERY_ENABLE" ]
  external_deps += [ "drivers_interface_battery:libbattery_proxy_2.0.z.so" ]
}
```

**用途**: 控制是否启用电池 HDI 集成

**影响代码**:
- Battery HDI v2.0 接口编译

**依赖**: `hdf_drivers_interface_battery` subsystem

---

### THERMAL_USER_VERSION

**定义位置**: `services/BUILD.gn:69-71`

```gni
if (build_variant == "user") {
  defines += [ "THERMAL_USER_VERSION" ]
}
```

**用途**: 区分用户版本和系统版本

**影响代码**:
- 温度上报开关
- Mock 功能限制

**证据**: `services/native/src/thermal_service.cpp:675-678`
```cpp
int32_t ThermalService::HandleThermalCallbackEvent(const HdfThermalCallbackInfo& event)
{
#ifndef THERMAL_USER_VERSION
    if (!isTempReport_) {
        return ERR_OK;
    }
#endif
    // ...
}
```

---

## 编译配置宏

### SANITIZE 配置

**定义位置**: `frameworks/napi/BUILD.gn:22-25`, `services/BUILD.gn:31-34`

```python
sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
}
```

**说明**:
- `cfi` - Control Flow Integrity 保护
- `cfi_cross_dso` - 跨 DSO CFI 保护
- `debug` - 禁用调试符号

---

### BRANCH_PROTECTION 配置

**定义位置**: `services/BUILD.gn:35`

```python
branch_protector_ret = "pac_ret"
```

**说明**: 返回地址保护（Pointer Authentication）

---

## 路径常量

### 系统配置文件路径

**定义位置**: `services/native/src/thermal_service.cpp:54-56`

```cpp
const std::string THERMAL_SERVICE_CONFIG_PATH = "etc/thermal_config/thermal_service_config.xml";
const std::string VENDOR_THERMAL_SERVICE_CONFIG_PATH = "/vendor/etc/thermal_config/thermal_service_config.xml";
const std::string SYSTEM_THERMAL_SERVICE_CONFIG_PATH = "/system/etc/thermal_config/thermal_service_config.xml";
```

**说明**: 配置文件搜索优先级（从代码推断）
1. `/vendor/etc/` - 厂商配置（最高优先级）
2. `/system/etc/` - 系统配置
3. `etc/` - 相对路径

### 工作队列配置

**定义位置**: `services/native/src/thermal_service.cpp:58`

```cpp
FFRTQueue g_queue("thermal_service");
```

**说明**: FFRT（Foundation Function Runtime）工作队列

---

### 错误码常量

**定义位置**: `services/native/src/thermal_service.cpp:59-60`

```cpp
constexpr uint32_t RETRY_TIME = 1000;
constexpr int32_t ERR_FAIL = -1;
constexpr int32_t ERR_OK = 0;
```

**说明**: 统一错误码定义

---

## SA 配置常量

### SA ID 和配置

**定义位置**: `sa_profile/3303.json:5`

```json
{
    "name": 3303,
    "libpath": "libthermalservice.z.so",
    "run-on-create": true,
    "distributed": false,
    "dump_level": 1
}
```

**说明**:
- SA ID: 3303 (POWER_MANAGER_THERMAL_SERVICE_ID)
- 自动启动: true
- 非分布式: false
- 支持 dump: 是

---

## 配置文件选项

### XML 解析选项

**证据**: `services/native/src/thermal_policy/thermal_srv_config_parser.cpp:74, 77`

```cpp
docParse = xmlReadMemory(
    result.c_str(), int(result.size()), NULL, NULL, XML_PARSE_NOBLANKS);
// 或
docParse = xmlReadFile(path.c_str(), nullptr, XML_PARSE_NOBLANKS);
```

**建议的安全选项**:
```cpp
// 禁用外部实体（推荐）
XML_PARSE_NOENT | XML_PARSE_NONET

// 或更严格
XML_PARSE_NOBLANKS | XML_PARSE_NOENT | XML_PARSE_NONET | XML_PARSE_NOCDATA
```

---

## Feature Flag 使用指南

### 启用音频集成

```bash
# 编译时启用
gn args --thermal_manager_audio_framework_enable=true

# 运行时启用（部分功能）
hdc shell param set persist.thermal.audio.enable 1
```

### 启用调试模式

```bash
# 启用详细日志
hdc shell param set persist.thermal.debug 1

# 启用 dump
hdc shell param set persist.thermal.dumplevel 1
```

---

## 总结

Thermal Manager 的配置系统包括：

1. **Feature Flags** (4 个):
   - `thermal_manager_audio_framework_enable`
   - `HAS_THERMAL_AIRPLANE_MANAGER_PART`
   - `HAS_THERMAL_AUDIO_FRAMEWORK_PART`
   - `HAS_THERMAL_DISPLAY_MANAGER_PART`

2. **条件编译宏** (8 个):
   - 子系统依赖相关
   - 控制特定功能编译

3. **编译安全配置**:
   - CFI 保护
   - 返回地址保护

4. **运行时配置**:
   - 配置文件路径
   - SA 配置
   - 错误码定义

**配置特点**:
- ✅ 模块化设计：通过宏控制特性
- ✅ 条件编译：根据子系统自动启用/禁用功能
- ✅ 安全编译：启用 CFI 等现代编译保护
