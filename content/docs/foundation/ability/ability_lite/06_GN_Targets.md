# GN 构建目标

## 目的

本文档描述 ability_lite 的 GN 构建系统配置，包括所有 targets、依赖关系和关键配置。

## 适用范围

- 需要理解构建系统的开发者
- 需要添加新模块的维护人员

## BUILD.gn 文件清单

| 文件路径 | 说明 |
|----------|------|
| `ability_lite.gni` | 全局变量定义 |
| `frameworks/ability_lite/BUILD.gn` | AbilityKit 库 |
| `frameworks/abilitymgr_lite/BUILD.gn` | AbilityManager 客户端库 |
| `frameworks/want_lite/BUILD.gn` | Want 库 |
| `services/abilitymgr_lite/BUILD.gn` | AMS 服务库 |
| `services/abilitymgr_lite/tools/BUILD.gn` | aa 命令行工具 |
| `interfaces/kits/js/napi/BUILD.gn` | N-API 绑定 |
| `interfaces/kits/js/declaration/BUILD.gn` | TypeScript 声明 |

## 关键 Targets

### 1. AbilityKit 库

**文件**: `frameworks/ability_lite/BUILD.gn`

**Target**: `ability`

| 属性 | 值 |
|------|-----|
| 类型 | `static_library` (liteos_m) / `shared_library` (其他) |
| 输出 | `libability.a` / `libability.so` |

**Sources** (liteos_m):
```
src/slite/ability_saved_data.cpp
src/slite/lite_context.cpp
src/slite/slite_ability.cpp
```

**Sources** (标准系统):
```
src/ability.cpp
src/ability_context.cpp
src/ability_env.cpp
src/ability_env_impl.cpp
src/ability_event_handler.cpp
src/ability_loader.cpp
src/ability_main.cpp
src/ability_scheduler.cpp
src/ability_thread.cpp
```

**条件编译** (当 `ability_lite_enable_ohos_appexecfwk_feature_ability == true`):
```
src/ability_slice.cpp
src/ability_slice_manager.cpp
src/ability_slice_route.cpp
src/ability_slice_scheduler.cpp
src/ability_slice_stack.cpp
src/ability_window.cpp
```

**依赖**:
```gn
deps = [
    "${aafwk_lite_path}/frameworks/abilitymgr_lite:abilitymanager",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "${communication_path}/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "${hilog_lite_path}/frameworks/featured:hilog_shared",
    "${kv_store_path}/interfaces/inner_api/kv_store:kv_store",
]
```

**Defines**:
```gn
defines = [
    "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
    "OPENHARMONY_FONT_PATH",
]

if (ability_lite_enable_ohos_appexecfwk_feature_ability == true) {
    defines += [ "ABILITY_WINDOW_SUPPORT" ]
}
```

### 2. AbilityManager 客户端库

**文件**: `frameworks/abilitymgr_lite/BUILD.gn`

**Target**: `abilitymanager`

| 属性 | 值 |
|------|-----|
| 类型 | `static_library` (liteos_m) / `shared_library` (其他) |
| 输出 | `libabilitymanager.a` / `libabilitymanager.so` |

**Sources** (liteos_m):
```
src/slite/ability_manager.cpp
src/slite/ability_manager_client.cpp
src/slite/ability_manager_inner.cpp
src/slite/ability_record_state_data.cpp
src/slite/abilityms_slite_client.cpp
src/slite/mission_info.cpp
```

**Sources** (标准系统):
```
src/ability_callback_utils.cpp
src/ability_manager.cpp
src/ability_self_callback.cpp
src/ability_service_manager.cpp
src/abilityms_client.cpp
```

**依赖**:
```gn
deps = [
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "${communication_path}/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "${hilog_lite_path}/frameworks/featured:hilog_shared",
]
```

### 3. Want 库

**文件**: `frameworks/want_lite/BUILD.gn`

**Target**: `want`

| 属性 | 值 |
|------|-----|
| 类型 | `static_library` |
| 输出 | `libwant.a` |

**Sources**:
```
src/want.cpp
```

**依赖**:
```gn
deps = [ "${hilog_lite_path}/frameworks/featured:hilog_shared" ]
```

### 4. AMS 服务库

**文件**: `services/abilitymgr_lite/BUILD.gn`

**Target**: `abilityms`

| 属性 | 值 |
|------|-----|
| 类型 | `static_library` (liteos_m) / `shared_library` (其他) |
| 输出 | `libabilityms.a` / `libabilityms.so` |

**Sources** (liteos_m):
```
src/slite/ability_list.cpp
src/slite/ability_mgr_service_slite.cpp
src/slite/ability_record.cpp
src/slite/ability_record_manager.cpp
src/slite/ability_record_observer_manager.cpp
src/slite/ability_thread.cpp
src/slite/ability_thread_loader.cpp
src/slite/bms_helper.cpp
src/slite/js_ability_thread.cpp
src/slite/native_ability_thread.cpp
src/slite/slite_ability_loader.cpp
```

**Sources** (标准系统):
```
src/ability_connect_mission.cpp
src/ability_inner_feature.cpp
src/ability_mgr_context.cpp
src/ability_mgr_feature.cpp
src/ability_mgr_handler.cpp
src/ability_mgr_service.cpp
src/ability_mission_record.cpp
src/ability_mission_stack.cpp
src/ability_stack_manager.cpp
src/ability_worker.cpp
src/app_manager.cpp
src/app_record.cpp
src/client/*.cpp
src/task/*.cpp
src/util/*.cpp
```

**条件编译**:
```gn
if (ability_lite_enable_ohos_appexecfwk_feature_ability == true) {
    deps += [ "${graphic_path}/surface_lite" ]
    defines += [ "ABILITY_WINDOW_SUPPORT" ]
}
```

**依赖**:
```gn
deps = [
    "${ability_lite_samgr_lite_path}/samgr:samgr",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "${communication_path}/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "${hilog_lite_path}/frameworks/featured:hilog_shared",
]
```

**配置 Defines**:
```gn
if (defined(ability_lite_config_ohos_aafwk_ams_task_size) &&
    ability_lite_config_ohos_aafwk_ams_task_size > 0) {
    defines += [ "AMS_TASK_STACK_SIZE=$ability_lite_config_ohos_aafwk_ams_task_size" ]
}

if (ability_lite_enable_ohos_aafwk_multi_tasks_feature == true) {
    defines += [ "_MINI_MULTI_TASKS_" ]
}
```

### 5. aa 命令行工具

**文件**: `services/abilitymgr_lite/tools/BUILD.gn`

**Target**: `aa`

| 属性 | 值 |
|------|-----|
| 类型 | `executable` |
| 输出 | `aa` |
| 输出目录 | `$root_out_dir/dev_tools` |

**Sources**:
```
src/ability_tool.cpp
src/main.cpp
```

**依赖**:
```gn
deps = [
    "${aafwk_lite_path}/frameworks/abilitymgr_lite:aafwk_abilityManager_lite",
    "${ability_lite_samgr_lite_path}/samgr:samgr",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "${communication_path}/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "${hilog_lite_path}/frameworks/featured:hilog_shared",
    "${kv_store_path}/interfaces/inner_api/kv_store:kv_store",
    "//build/lite/config/component/cJSON:cjson_shared",
]
```

### 6. N-API 绑定

**文件**: `interfaces/kits/js/napi/BUILD.gn`

**Target**: `aafwk`

| 属性 | 值 |
|------|-----|
| 类型 | `ohos_shared_library` |
| 输出 | `libaafwk.so` |
| 安装目录 | `module/` |

**Sources**:
```
js_aafwk.cpp
```

**依赖**:
```gn
deps = [
    "${ability_lite_path}/frameworks/abilitymgr_lite:abilitymanager",
    "${arkui_path}/napi/:ace_napi",
]

external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
]
```

## Component Targets

### 顶层组件

**定义在 bundle.json**:

```json
{
    "build": {
        "sub_component": [
            "//foundation/ability/ability_lite/frameworks/ability_lite:aafwk_abilitykit_lite",
            "//foundation/ability/ability_lite/frameworks/abilitymgr_lite:aafwk_abilityManager_lite",
            "//foundation/ability/ability_lite/services/abilitymgr_lite:aafwk_services_lite"
        ]
    }
}
```

| Component | 包含 Targets |
|-----------|-------------|
| `aafwk_abilitykit_lite` | `:ability` |
| `aafwk_abilityManager_lite` | `:abilitymanager` |
| `aafwk_services_lite` | `:abilityms`, `tools:aa` (非 liteos_m) |

## 关键配置参数

### Feature Flags

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `ability_lite_enable_ohos_appexecfwk_feature_ability` | bool | false | 启用 Page Ability 和窗口支持 |
| `ability_lite_enable_ohos_aafwk_multi_tasks_feature` | bool | false | 启用多任务支持 |
| `ability_lite_config_ohos_aafwk_ams_task_size` | int | 0 | AMS 任务栈大小 |
| `ability_lite_config_ohos_aafwk_aafwk_lite_task_stack_size` | int | 0 | 任务栈大小 |
| `ability_lite_config_ohos_aafwk_ability_list_capacity` | int | 0 | Ability 列表容量 |

### 全局变量 (ability_lite.gni)

```gn
hilog_lite_path = "//base/hiviewdfx/hilog_lite"
permission_lite_path = "//base/security/permission_lite"
appapawn_lite_path = "//base/startup/appspawn/lite"
utils_lite_path = "//commonlibrary/utils_lite"
ability_lite_path = "//foundation/ability/ability_lite"
graphic_path = "//foundation/graphic"
arkui_path = "//foundation/arkui"
ace_engine_lite_path = "${arkui_path}/ace_engine_lite"
ability_lite_samgr_lite_path = "//foundation/systemabilitymgr/samgr_lite"
kv_store_path = "//foundation/distributeddatamgr/kv_store"
communication_path = "//foundation/communication"
dmsfwk_lite_path = "//foundation/ability/dmsfwk_lite"
```

## Target 依赖图

```
aafwk_abilitykit_lite
    └── :ability
        ├── :abilitymanager
        ├── :bundle (bundle_framework_lite)
        ├── :ipc_single
        ├── :hilog_shared
        └── :kv_store

aafwk_abilityManager_lite
    └── :abilitymanager
        ├── :bundle
        ├── :ipc_single
        └── :hilog_shared

aafwk_services_lite
    ├── :abilityms
    │   ├── :samgr
    │   ├── :bundle
    │   ├── :ipc_single
    │   └── :hilog_shared
    └── tools:aa
        ├── aafwk_abilityManager_lite
        ├── :samgr
        ├── :bundle
        ├── :ipc_single
        ├── :hilog_shared
        └── :cjson_shared
```

## 编译命令示例

### 编译整个组件

```bash
gn gen out --args='...'
ninja -C out aafwk_abilitykit_lite aafwk_abilityManager_lite aafwk_services_lite
```

### 编译单个 target

```bash
# AbilityKit
ninja -C out //foundation/ability/ability_lite/frameworks/ability_lite:ability

# AMS 服务
ninja -C out //foundation/ability/ability_lite/services/abilitymgr_lite:abilityms

# aa 工具
ninja -C out //foundation/ability/ability_lite/services/abilitymgr_lite/tools:aa

# N-API
ninja -C out //foundation/ability/ability_lite/interfaces/kits/js/napi:aafwk
```

## 相关链接

- [编译产物](07_Build_Artifacts.md)
- [目录结构](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
