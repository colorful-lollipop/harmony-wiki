# GN 构建系统 - display_manager

> 本文档说明 display_manager 模块的 GN 构建系统配置

---

## 文档目的

本文档提供：
- 所有 GN 构建目标的详细说明
- 目标之间的依赖关系
- 编译配置和特性开关
- 源文件组织

## 适用范围

- **适用对象**：构建工程师、系统集成人员、模块开发者
- **前置知识**：熟悉 GN 构建系统、OpenHarmony 组件构建

---

## 构建文件概览

### 构建文件列表

| 文件 | 用途 |
|------|------|
| `displaymgr.gni` | 全局配置变量和特性开关 |
| `state_manager/service/BUILD.gn` | 服务层构建（主服务、Proxy/Stub） |
| `state_manager/interfaces/inner_api/BUILD.gn` | 内部 API 库构建 |
| `state_manager/frameworks/napi/BUILD.gn` | N-API 模块构建 |
| `brightness_manager/BUILD.gn` | 亮度管理静态库构建 |

---

## 全局配置（displaymgr.gni）

**文件位置**：`displaymgr.gni`

### 构建参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `has_sensors_sensor_part` | bool | true | 启用传感器集成 |
| `display_manager_feature_brightnessext` | string | "" | 亮度扩展功能（空字符串表示未启用） |
| `display_manager_feature_poweroff_strategy` | bool | false | 屏幕关闭策略功能 |
| `has_hiviewdfx_hisysevent_part` | bool | true | Hisysevent 支持 |
| `has_dfx_hiview_part` | bool | true | HiView DFX 支持 |

### 条件编译宏

| 条件 | 宏定义 | 说明 |
|------|--------|------|
| `has_hiviewdfx_hisysevent_part` | `HAS_HIVIEWDFX_HISYSEVENT_PART` | 启用 Hisysevent 日志 |
| `has_dfx_hiview_part` | `HAS_DFX_HIVIEW_PART` | 启用 HiView DFX 功能 |
| `display_manager_feature_poweroff_strategy` | `ENABLE_SCREEN_POWER_OFF_STRATEGY` | 启用关屏策略 |

### 路径变量

```gn
displaymgr_part_name = "display_manager"
displaymgr_root_path = "//base/powermgr/display_manager/state_manager"
displaymgr_framework_path = "${displaymgr_root_path}/frameworks"
displaymgr_service_zidl = "${displaymgr_root_path}/service/zidl"
displaymgr_inner_api = "${displaymgr_root_path}/interfaces/inner_api"
displaymgr_utils_path = "${displaymgr_root_path}/utils"
brightnessmgr_root_path = "//base/powermgr/display_manager/brightness_manager"
taihe_generated_file_path = "${root_out_dir}/taihe/out/powermgr/display_manager"
```

---

## 服务层构建（state_manager/service/BUILD.gn）

### 目标清单

#### 1. displaymgrservice（主服务共享库）

| 属性 | 值 |
|------|------|
| **类型** | `ohos_shared_library` |
| **输出** | `libdisplaymgrservice.z.so` |
| **库类型** | `shlib_type = "sa"` (System Ability) |
| **子系统** | `powermgr` |
| **部件** | `display_manager` |

**安全配置**：
```gn
sanitize = {
  cfi = true              # Control Flow Integrity
  cfi_cross_dso = true
  debug = false
}
branch_protector_ret = "pac_ret"  # Pointer Authentication
```

**源文件**（11 个）：
```
${displaymgr_utils_path}/native/src/display_xcollie.cpp
native/src/display_auto_brightness.cpp
native/src/display_common_event_mgr.cpp
native/src/display_param_helper.cpp
native/src/display_power_mgr_service.cpp
native/src/display_setting_helper.cpp
native/src/display_system_ability.cpp
native/src/gradual_animator.cpp
native/src/screen_action.cpp
native/src/screen_controller.cpp
zidl/src/display_brightness_callback_proxy.cpp
zidl/src/display_brightness_listener_proxy.cpp
zidl/src/display_power_callback_proxy.cpp
```

**配置**：
```gn
configs = [
  "${displaymgr_utils_path}:utils_config",
  ":displaymgr_private_config",
  "${displaymgr_utils_path}:coverage_flags",
]

public_configs = [
  ":displaymgr_public_config",
  "${brightnessmgr_root_path}:brightness_manager_config",
]
```

**依赖**：
```gn
deps = [
  "${displaymgr_root_path}/service:displaymgr_stub",
]

public_deps = [
  "${brightnessmgr_root_path}:brightness_manager",
]

external_deps = [
  "power_manager:power_permission",
  "ability_base:zuri",
  "ability_runtime:ability_manager",
  "cJSON:cjson",
  "c_utils:utils",
  "data_share:datashare_consumer",
  "eventhandler:libeventhandler",
  "ffrt:libffrt",
  "graphic_2d:librender_service_base",
  "hicollie:libhicollie",
  "hilog:libhilog",
  "image_framework:image_native",
  "ipc:ipc_core",
  "power_manager:power_ffrt",
  "power_manager:power_setting",
  "power_manager:power_sysparam",
  "power_manager:powermgr_client",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
  "skia:skia_canvaskit",
  "window_manager:libdm_lite",
]
```

**条件编译**：
```gn
# 传感器支持
if (has_sensors_sensor_part) {
  external_deps += [ "sensor:sensor_interface_native" ]
  defines += [ "ENABLE_SENSOR_PART" ]
}

# Hisysevent 支持
if (has_hiviewdfx_hisysevent_part) {
  external_deps += [ "hisysevent:libhisysevent" ]
}

# Fuzz 测试覆盖
if (use_clang_coverage) {
  defines += [ "FUZZ_COV_TEST" ]
}

# 关屏策略（可选）
if (display_manager_feature_poweroff_strategy) {
  sources += [ "native/src/miscellaneous_display_power_strategy.cpp" ]
}
```

---

#### 2. displaymgr_interface（IDL 接口生成）

| 属性 | 值 |
|------|------|
| **类型** | `idl_gen_interface` |
| **子系统** | `powermgr` |
| **部件** | `display_manager` |

**IDL 源文件**：
```
IDisplayPowerMgr.idl
DisplayPowerMgrIdlTypes.idl
```

**日志配置**：
```gn
log_domainid = "0xD002982"
log_tag = "DisplayPowerSvc"
```

---

#### 3. displaymgr_proxy（客户端 Proxy）

| 属性 | 值 |
|------|------|
| **类型** | `ohos_source_set` |
| **源文件** | 从 `displaymgr_interface` 输出过滤 `*_proxy.cpp` |

**安全配置**：
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
}
```

**依赖**：
```gn
deps = [ ":displaymgr_interface" ]

external_deps = [
  "c_utils:utils",
  "hicollie:libhicollie",
  "hilog:libhilog",
  "ipc:ipc_core",
  "power_manager:power_setting",
  "safwk:system_ability_fwk",
]
```

---

#### 4. displaymgr_stub（服务端 Stub）

| 属性 | 值 |
|------|------|
| **类型** | `ohos_source_set` |
| **源文件** | 从 `displaymgr_interface` 输出过滤 `*_stub.cpp` |

**依赖**：与 `displaymgr_proxy` 相同

---

## 内部 API 构建（state_manager/interfaces/inner_api/BUILD.gn）

### displaymgr（客户端共享库）

| 属性 | 值 |
|------|------|
| **类型** | `ohos_shared_library` |
| **输出** | `libdisplaymgr.so` |
| **标签** | `platformsdk` |
| **安装位置** | `system_base_dir/platformsdk/` |

**安全配置**：
```gn
branch_protector_ret = "pac_ret"
sanitize = {
  cfi = true
  cfi_cross_dso = true
}
```

**源文件**（4 个）：
```
${displaymgr_framework_path}/native/display_power_mgr_client.cpp
${displaymgr_service_zidl}/src/display_brightness_callback_stub.cpp
${displaymgr_service_zidl}/src/display_power_callback_stub.cpp
${displaymgr_service_zidl}/src/display_brightness_listener_stub.cpp
```

**配置**：
```gn
configs = [
  ":displaymgr_private_config",
  "${displaymgr_utils_path}:coverage_flags",
  "${displaymgr_root_path}/service:displaymgr_public_config",
]

public_configs = [
  ":displaymgr_public_config",
  "${displaymgr_root_path}/service:displaymgr_public_config",
]
```

**依赖**：
```gn
deps = [
  "${displaymgr_root_path}/service:displaymgr_proxy",
]

external_deps = [
  "c_utils:utils",
  "hicollie:libhicollie",
  "hilog:libhilog",
  "ipc:ipc_core",
  "power_manager:powermgr_client",
  "samgr:samgr_proxy",
]
```

---

## N-API 构建（state_manager/frameworks/napi/BUILD.gn）

### brightness（N-API 模块）

| 属性 | 值 |
|------|------|
| **类型** | `ohos_shared_library` |
| **输出** | `libbrightness.so` |
| **安装位置** | `module/` |
| **子系统** | `powermgr` |
| **部件** | `display_manager` |

**源文件**（2 个）：
```
${displaymgr_framework_path}/napi/brightness.cpp
${displaymgr_framework_path}/napi/brightness_module.cpp
```

**配置**：
```gn
configs = [
  "${displaymgr_utils_path}:utils_config",
  "${displaymgr_utils_path}:coverage_flags",
  "${displaymgr_root_path}/service:displaymgr_public_config",
]
```

**依赖**：
```gn
deps = [
  "${brightnessmgr_root_path}:brightness_manager",
  "${displaymgr_inner_api}:displaymgr",
]

external_deps = [
  "c_utils:utils",
  "ets_runtime:libark_jsruntime",
  "hilog:libhilog",
  "ipc:ipc_core",
  "napi:ace_napi",
  "power_manager:powermgr_client",
]
```

**条件依赖**：
```gn
if (has_dfx_hiview_part) {
  external_deps += [ "hiview:libxpower_event_js" ]
}
```

---

## 亮度管理构建（brightness_manager/BUILD.gn）

### brightness_manager（静态库）

| 属性 | 值 |
|------|------|
| **类型** | `ohos_static_library` |
| **输出** | `libbrightness_manager.a` |
| **子系统** | `powermgr` |
| **部件** | `display_manager` |

**安全配置**：
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
}
branch_protector_ret = "pac_ret"
```

**源文件**（16 个）：
```
src/brightness_action.cpp
src/brightness_config_parser.cpp
src/brightness_dimming.cpp
src/brightness_manager.cpp
src/brightness_manager_ext.cpp
src/brightness_param_helper.cpp
src/brightness_service.cpp
src/brightness_setting_helper.cpp
src/calculation_config_parser.cpp
src/calculation_curve.cpp
src/calculation_manager.cpp
src/config_parser.cpp
src/config_parser_base.cpp
src/light_lux_buffer.cpp
src/light_lux_manager.cpp
src/lux_filter_config_parser.cpp
src/lux_threshold_config_parser.cpp
```

**配置**：
```gn
configs = [
  ":brightness_manager_config",
  "${displaymgr_utils_path}:utils_config",
  "${displaymgr_inner_api}:displaymgr_public_config",
  "${displaymgr_root_path}/service:displaymgr_public_config",
]
```

**外部依赖**（16 个）：
```
ability_base:zuri
ability_runtime:ability_manager
cJSON:cjson
c_utils:utils
data_share:datashare_consumer
eventhandler:libeventhandler
ffrt:libffrt
graphic_2d:librender_service_base
hicollie:libhicollie
hilog:libhilog
image_framework:image_native
ipc:ipc_core
power_manager:power_ffrt
power_manager:power_setting
power_manager:power_sysparam
safwk:system_ability_fwk
samgr:samgr_proxy
skia:skia_canvaskit
window_manager:libdm_lite
```

**条件编译**：
```gn
# Fuzz 测试
if (use_libfuzzer) {
  defines += [ "FUZZ_TEST" ]
}

# 传感器支持
if (has_sensors_sensor_part) {
  defines += [ "ENABLE_SENSOR_PART" ]
  external_deps += [ "sensor:sensor_interface_native" ]
}

# 亮度扩展
if (display_manager_feature_brightnessext != "") {
  defines += [ "OHOS_BUILD_ENABLE_BRIGHTNESS_WRAPPER" ]
}

# Hisysevent 支持
if (has_hiviewdfx_hisysevent_part) {
  external_deps += [ "hisysevent:libhisysevent" ]
}
```

---

## 依赖关系图

```
libdisplaymgrservice.z.so (displaymgrservice)
    ├── displaymgr_stub (source_set)
    │   └── displaymgr_interface (idl_gen)
    ├── displaymgr_proxy (source_set)
    │   └── displaymgr_interface (idl_gen)
    ├── libbrightness_manager.a (brightness_manager)
    └── libdisplaymgr.so (displaymgr) [client lib]
        └── displaymgr_proxy

libbrightness.so (brightness)
    ├── libbrightness_manager.a
    └── libdisplaymgr.so
```

---

## 特性开关使用

### 启用传感器支持

默认自动检测，如需显式控制：

```gn
# 在组件配置中覆盖
has_sensors_sensor_part = true
defines += [ "ENABLE_SENSOR_PART" ]
```

### 启用关屏策略

```gn
display_manager_feature_poweroff_strategy = true
defines += [ "ENABLE_SCREEN_POWER_OFF_STRATEGY" ]
```

### 启用亮度扩展

```gn
display_manager_feature_brightnessext = "wrapper_name"
defines += [ "OHOS_BUILD_ENABLE_BRIGHTNESS_WRAPPER" ]
```

---

## 相关链接

- **目录结构**：[02_Directory_Structure.md](02_Directory_Structure.md)
- **编译产物**：[07_Build_Artifacts.md](07_Build_Artifacts.md)
- **配置标志**：[appendix/Config_Flags.md](appendix/Config_Flags.md)

---

## 文档更新记录

- **2026-02-07**：初始版本 v1.0，基于 BUILD.gn 文件分析生成
