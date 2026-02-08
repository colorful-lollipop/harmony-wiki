# GN 构建目标

## 目的

本文档详细说明 bundle_framework_lite 的 GN 构建配置，包括所有 targets、依赖关系和编译选项。

## 构建文件概览

| 文件路径 | 说明 |
|---------|------|
| `bundle_framework_lite.gni` | 全局配置和路径定义 |
| `frameworks/bundle_lite/BUILD.gn` | BundleKit 库构建 |
| `services/bundlemgr_lite/BUILD.gn` | BMS 服务构建 |
| `services/bundlemgr_lite/bundle_daemon/BUILD.gn` | Bundle Daemon 构建 |
| `services/bundlemgr_lite/tools/BUILD.gn` | bm 工具构建 |
| `interfaces/kits/bundle_lite/js/builtin/BUILD.gn` | JS API 构建 |

## 全局配置

### bundle_framework_lite.gni

**位置**: 根目录 `bundle_framework_lite.gni:1-32`

```gn
# 外部依赖路径
ace_engine_lite_path = "//foundation/arkui/ace_engine_lite"
appverify_lite_path = "//base/security/appverify/interfaces/innerkits/appverify_lite"
arkui_path = "//foundation/arkui"
communication_path = "//foundation/communication"
hilog_lite_path = "//base/hiviewdfx/hilog_lite"
permission_lite_path = "//base/security/permission_lite"
resource_management_lite_path = "//base/global/resource_management_lite"
samgr_lite_path = "//foundation/systemabilitymgr/samgr_lite"
startup_path = "//base/startup"
utils_lite_path = "//commonlibrary/utils_lite"

# Feature 开关
declare_args() {
  bundle_framework_lite_enable_ohos_bundle_manager_service = false
  bundle_framework_lite_enable_ohos_bundle_manager_service_permission = false
  bundle_framework_lite_enable_ohos_bundle_manager_service_parse_metadata = false
}
```

**Feature 开关说明**:

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `bundle_framework_lite_enable_ohos_bundle_manager_service` | false | 启用 BMS 服务 |
| `bundle_framework_lite_enable_ohos_bundle_manager_service_permission` | false | 启用权限管理 |
| `bundle_framework_lite_enable_ohos_bundle_manager_service_parse_metadata` | false | 启用元数据解析 |

## 主要 Targets

### 1. BundleKit 库 (frameworks/bundle_lite)

**位置**: `frameworks/bundle_lite/BUILD.gn:30-132`

#### Target: bundle

**类型**: `lite_library`

**条件编译**:
- `ohos_kernel_type == "liteos_m"`: 静态库 (`static_library`)
- 其他: 共享库 (`shared_library`)

**源文件** (LiteOS-A/Linux):
```gn
sources = [
  "src/ability_info.cpp",
  "src/ability_info_utils.cpp",
  "src/bundle_callback.cpp",
  "src/bundle_callback_utils.cpp",
  "src/bundle_info.cpp",
  "src/bundle_info_utils.cpp",
  "src/bundle_manager.cpp",
  "src/bundle_self_callback.cpp",
  "src/convert_utils.cpp",
  "src/element_name.cpp",
  "src/module_info.cpp",
  "src/module_info_utils.cpp",
  "src/token_generate.cpp",
]
```

**源文件** (LiteOS-M):
```gn
sources = [
  "src/ability_info.cpp",
  "src/ability_info_utils.cpp",
  "src/bundle_info.cpp",
  "src/bundle_info_utils.cpp",
  "src/element_name.cpp",
  "src/module_info.cpp",
  "src/module_info_utils.cpp",
  "src/slite/bundle_manager.cpp",
  "src/slite/bundle_manager_inner.cpp",
  "src/slite/bundlems_slite_client.cpp",
]
```

**依赖** (LiteOS-A/Linux):
```gn
deps = [
  "${aafwk_lite_path}/frameworks/want_lite:want",
  "${hilog_lite_path}/frameworks/featured:hilog_shared",
  "${permission_lite_path}/services/pms_client:pms_client",
]
```

**依赖** (LiteOS-M):
```gn
public_deps = [
  "${aafwk_lite_path}/frameworks/want_lite:want",
  "${hilog_lite_path}/frameworks/featured:hilog_static",
]
```

**包含目录**:
```gn
include_dirs = [
  "include",
  "${permission_lite_path}/interfaces/kits",
  "${permission_lite_path}/services/pms/include",
  "${aafwk_lite_path}/frameworks/want_lite/include",
  "${aafwk_lite_path}/interfaces/kits/want_lite",
  "${aafwk_lite_path}/interfaces/inner_api/abilitymgr_lite",
  "${appexecfwk_lite_path}/interfaces/inner_api/bundlemgr_lite",
  "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
  "${appexecfwk_lite_path}/utils/bundle_lite",
  "${communication_path}/ipc/interfaces/innerkits/c/ipc/include",
  "${samgr_lite_path}/interfaces/kits/samgr",
  "${samgr_lite_path}/interfaces/kits/registry",
  "//third_party/bounds_checking_function/include",
  "${utils_lite_path}/include",
  "//third_party/cJSON",
]
```

**编译选项**:
```gn
cflags = [
  "-fPIC",
  "-Wall",
  "-Wno-format",
]
cflags_cc = cflags
defines = [ "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER" ]
```

#### Target: appexecfwk_kits_lite

**类型**: `lite_component`

```gn
lite_component("appexecfwk_kits_lite") {
  features = [ ":bundle" ]
}
```

#### Target: bundle_notes (NDK)

**类型**: `ndk_lib`

```gn
ndk_lib("bundle_notes") {
  lib_extension = ".so"
  deps = [ ":bundle" ]
  head_files = [ "${appexecfwk_lite_path}/interfaces/kits/bundle_lite" ]
}
```

### 2. BMS 服务 (services/bundlemgr_lite)

**位置**: `services/bundlemgr_lite/BUILD.gn:1-185`

#### Target: bundlems

**类型**: 
- `ohos_kernel_type == "liteos_m"`: `static_library`
- 其他: `shared_library`

**配置** (`bundle_config`):
```gn
config("bundle_config") {
  defines = [ "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER" ]
  cflags_cc = [ "-std=c++14" ]
}
```

**源文件** (LiteOS-M):
```gn
sources = [
  "src/bundle_map.cpp",
  "src/bundle_mgr_service.cpp",
  "src/bundle_mgr_slite_feature.cpp",
  "src/bundle_util.cpp",
  "src/gt_bundle_extractor.cpp",
  "src/gt_bundle_installer.cpp",
  "src/gt_bundle_manager_service.cpp",
  "src/gt_bundle_parser.cpp",
  "src/gt_extractor_util.cpp",
]
```

**源文件** (LiteOS-A/Linux):
```gn
sources = [
  "src/bundle_daemon_client.cpp",
  "src/bundle_extractor.cpp",
  "src/bundle_info_creator.cpp",
  "src/bundle_inner_feature.cpp",
  "src/bundle_installer.cpp",
  "src/bundle_manager_service.cpp",
  "src/bundle_map.cpp",
  "src/bundle_ms_feature.cpp",
  "src/bundle_ms_host.cpp",
  "src/bundle_parser.cpp",
  "src/bundle_res_transform.cpp",
  "src/bundle_util.cpp",
  "src/extractor_util.cpp",
  "src/hap_sign_verify.cpp",
  "src/zip_file.cpp",
]
```

**编译选项** (LiteOS-A/Linux):
```gn
cflags = [
  "-Wall",
  "-Wno-format",
  "-Wno-format-extra-args",
]
```

**公共依赖** (LiteOS-A/Linux):
```gn
public_deps = [
  "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
  "${appverify_lite_path}:verify",
  "${hilog_lite_path}/frameworks/featured:hilog_shared",
  "${resource_management_lite_path}/frameworks/resmgr_lite:global_resmgr",
  "${samgr_lite_path}/samgr:samgr",
  "//build/lite/config/component/cJSON:cjson_shared",
  "//build/lite/config/component/zlib:zlib_shared",
]
```

**包含目录** (LiteOS-A/Linux):
```gn
include_dirs = [
  "${resource_management_lite_path}/interfaces/inner_api/include",
  "${aafwk_lite_path}/services/abilitymgr_lite/include",
  "${aafwk_lite_path}/interfaces/inner_api/abilitymgr_lite",
  "${aafwk_lite_path}/interfaces/kits/ability_lite",
  "${aafwk_lite_path}/interfaces/kits/want_lite",
  "${aafwk_lite_path}/frameworks/want_lite/include",
  "${appexecfwk_lite_path}/interfaces/inner_api/bundlemgr_lite",
  "${appexecfwk_lite_path}/frameworks/bundle_lite/include",
  "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
  "${appexecfwk_lite_path}/utils/bundle_lite",
  "${samgr_lite_path}/interfaces/kits/registry",
  "${samgr_lite_path}/interfaces/kits/samgr",
  "//third_party/cJSON",
  "//third_party/zlib",
  "//third_party/zlib/contrib/minizip",
  "${permission_lite_path}/interfaces/kits",
  "${permission_lite_path}/services/pms/include",
  "${appverify_lite_path}/include",
  "//third_party/bounds_checking_function/include",
  "${utils_lite_path}/include",
  "${utils_lite_path}/memory",
  "include",
]
```

#### Target: appexecfwk_services_lite

**类型**: `lite_component`

**特性** (LiteOS-M):
```gn
lite_component("appexecfwk_services_lite") {
  features = [ ":bundlems" ]
}
```

**特性** (LiteOS-A/Linux):
```gn
lite_component("appexecfwk_services_lite") {
  features = [
    ":bundlems",
    "tools:bm",
    "bundle_daemon:bundle_daemon",
  ]
}
```

### 3. Bundle Daemon

**位置**: `services/bundlemgr_lite/bundle_daemon/BUILD.gn`

#### Target: bundle_daemon

**类型**: `executable`

**源文件**:
```gn
sources = [
  "src/bundle_daemon.cpp",
  "src/bundle_daemon_handler.cpp",
  "src/bundle_file_utils.cpp",
  "src/bundlems_client.cpp",
  "src/main.cpp",
]
```

**依赖**:
```gn
deps = [
  "${hilog_lite_path}/frameworks/featured:hilog_shared",
  "${samgr_lite_path}/samgr:samgr",
  "//build/lite/config/component/zlib:zlib_shared",
]
```

### 4. bm 工具

**位置**: `services/bundlemgr_lite/tools/BUILD.gn`

#### Target: bm

**类型**: `executable`

**源文件**:
```gn
sources = [
  "src/command_parser.cpp",
  "src/main.cpp",
]
```

**依赖**:
```gn
deps = [
  "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
  "${hilog_lite_path}/frameworks/featured:hilog_shared",
]
```

### 5. JS API

**位置**: `interfaces/kits/bundle_lite/js/builtin/BUILD.gn`

#### Target: capability_api

**类型**: 
- `ohos_kernel_type == "liteos_m"`: `static_library`
- 其他: `shared_library`

**源文件**:
```gn
sources = [ "src/capability_module.cpp" ]
```

**依赖**:
```gn
deps = [
  "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
  "${hilog_lite_path}/frameworks/featured:hilog_shared",
]
```

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           依赖关系图                                     │
└─────────────────────────────────────────────────────────────────────────┘

appexecfwk_services_lite (组件)
    ├── bundlems (共享库/静态库)
    │       ├── bundle (来自 frameworks/bundle_lite)
    │       ├── verify (appverify_lite)
    │       ├── hilog_shared
    │       ├── global_resmgr
    │       ├── samgr
    │       ├── cjson_shared
    │       └── zlib_shared
    ├── bm (可执行文件)
    │       └── bundle
    └── bundle_daemon (可执行文件)
            ├── hilog_shared
            ├── samgr
            └── zlib_shared

appexecfwk_kits_lite (组件)
    └── bundle (共享库/静态库)
            ├── want (aafwk_lite)
            ├── hilog_shared / hilog_static
            └── pms_client (permission_lite)
```

## Target 汇总表

| Target | 类型 | 输出 | 路径 | 说明 |
|--------|------|------|------|------|
| bundle | shared_library/static_library | libbundle.so/libbundle.a | frameworks/bundle_lite | BundleKit 库 |
| appexecfwk_kits_lite | lite_component | - | frameworks/bundle_lite | 组件目标 |
| bundlems | shared_library/static_library | libbundlems.so/libbundlems.a | services/bundlemgr_lite | BMS 服务库 |
| appexecfwk_services_lite | lite_component | - | services/bundlemgr_lite | 组件目标 |
| bundle_daemon | executable | bundle_daemon | services/bundlemgr_lite/bundle_daemon | 守护进程 |
| bm | executable | bm | services/bundlemgr_lite/tools | 命令行工具 |
| capability_api | shared_library/static_library | libcapability_api.so | interfaces/kits/bundle_lite/js/builtin | JS API |

## 编译命令

```bash
# 编译整个组件
hb build -T bundle_framework_lite

# 编译特定目标
hb build -T //foundation/bundlemanager/bundle_framework_lite/services/bundlemgr_lite:bundlems
hb build -T //foundation/bundlemanager/bundle_framework_lite/frameworks/bundle_lite:bundle
hb build -T //foundation/bundlemanager/bundle_framework_lite/services/bundlemgr_lite/tools:bm

# 编译全量
hb build -f
```

---

**相关链接**:
- [编译产物](06_Build_Artifacts.md)
- [内部 API](04_Internal_API.md)
- [附录 - 配置开关](appendix/Config_Flags.md)
