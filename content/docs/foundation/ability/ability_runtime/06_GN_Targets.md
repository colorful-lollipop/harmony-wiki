# GN Targets 梳理

## 概述

ability_runtime 使用 GN（Generate Ninja）作为构建系统。本文档梳理主要 GN targets 的结构、依赖和输出产物。

## 根构建配置

### BUILD.gn

```gn
# ability_runtime/BUILD.gn
import("//build/ohos.gni")
```

**说明**：根目录 BUILD.gn 仅作为构建入口，实际配置在子目录。

### ability_runtime.gni

**位置**：`ability_runtime.gni`

**功能**：定义全局路径变量、编译选项开关。

**关键变量**：

```gn
# 路径变量
ability_runtime_path = "//foundation/ability/ability_runtime"
ability_runtime_napi_path = "${ability_runtime_path}/frameworks/js/napi"
ability_runtime_innerkits_path = "${ability_runtime_path}/interfaces/inner_api"
ability_runtime_native_path = "${ability_runtime_path}/frameworks/native"
ability_runtime_services_path = "${ability_runtime_path}/services"
```

**特性开关**：

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `ability_runtime_auto_fill` | true | 自动填充扩展 |
| `ability_runtime_child_process` | true | 子进程支持 |
| `ability_runtime_graphics` | true | 图形依赖 |
| `ability_runtime_power` | true | 电源管理 |
| `ability_runtime_ui_service_extension` | true | UI 服务扩展 |
| `ability_runtime_check_internet_permission` | false | 网络权限检查 |
| `ability_runtime_forbid_start_enabled` | false | 启动禁止 |

**代码证据**：`ability_runtime.gni` 第 14-260 行

## 主要 Targets 分类

### 1. 系统服务层

#### AbilityManagerService

**BUILD.gn 位置**：`services/abilitymgr/BUILD.gn`

**目标类型**：shared_library（系统服务）

**主要 deps**：

```gn
deps = [
  "//foundation/ability/ability_base/interfaces/inner_api:ability_base",
  "//foundation/bundlemanager/bundle_framework/interfaces/inner_api:bundlefwk_inner_kits",
  "//foundation/communication/ipc/interfaces:ipc_core",
  "//foundation/systemabilitymgr/samgr/interfaces:samgr_client",
  "//base/hiviewdfx/hilog/interfaces:native",
  "//utils/system/safwk/interfaces:safwk_core",
]
```

**输出产物**：
- `libability_manager_service.z.so`（32位）
- `libability_manager_service.so`（64位）

**SA ID**：`ABILITY_MGR_SERVICE_ID` (3701)

#### AppManagerService

**BUILD.gn 位置**：`services/appmgr/BUILD.gn`

**目标类型**：shared_library

**主要 deps**：

```gn
deps = [
  "//foundation/ability/ability_base/interfaces/inner_api:ability_base",
  "//foundation/bundlemanager/bundle_framework/interfaces/inner_api:bundlefwk_inner_kits",
  "//foundation/startup/appspawn/interfaces:appspawn",
  "//foundation/systemabilitymgr/samgr/interfaces:samgr_client",
  "//base/hiviewdfx/hilog/interfaces:native",
]
```

**输出产物**：
- `libapp_manager_service.z.so`
- `libapp_manager_service.so`

**SA ID**：`APP_MGR_SERVICE_ID` (1201)

### 2. N-API 模块

#### N-API 汇总

**BUILD.gn 位置**：`frameworks/js/napi/BUILD.gn`

**目标类型**：shared_library

```gn
napi_packages() {
  deps = [
    ":ability",
    ":ability_context",
    ":ability_manager",
    ":application",
    ":app_manager",
    ":callee",
    ":caller",
    # ... 更多模块
  ]
}
```

**输出产物**：
- `libability_napi.z.so`
- `libability_napi.so`

#### 各功能模块

**BUILD.gn 模式**：`frameworks/js/napi/{module}/BUILD.gn`

```gn
# 示例：ability_manager
module_name = "abilityManager"

napi_module("ability_manager") {
  sources = [
    "ability_manager_module.cpp",
    "ability_manager_impl.cpp",
    # ... 更多源文件
  ]
  
  deps = [
    "//foundation/ability/ability_runtime/interfaces/inner_api/ability_manager:ability_manager",
    "//foundation/ability/ability_runtime/frameworks/js/napi/inner/napi_ability_common:napi_ability_common",
    "//third_party/node:node",
    "//foundation/ability/ability_base/interfaces/inner_api:ability_base",
  ]
  
  public_deps = [
    "//foundation/ability/ability_runtime/interfaces/inner_api/ability_manager:ability_manager",
  ]
}
```

### 3. Native 框架

#### Native 模块

**BUILD.gn 位置**：`frameworks/native/ability/native/BUILD.gn`

**目标类型**：source_set + shared_library

```gn
# 静态库
component("ability_thread") {
  type = "source_set"
  sources = [
    "ability_thread.cpp",
    # ... 更多文件
  ]
  
  deps = [
    "//foundation/ability/ability_base/frameworks:native_base",
    "//foundation/ability/ability_runtime/interfaces/inner_api/runtime:runtime",
  ]
}

# 动态库
shared_library("uiabilitykit_native") {
  deps = [
    ":ability_thread",
    "//foundation/ability/ability_runtime/interfaces/kits/native/ability/native:abilitykit_native",
  ]
}
```

### 4. ETS/ArkTS 绑定

**BUILD.gn 位置**：`frameworks/ets/ani/BUILD.gn`

**目标类型**：shared_library

```gn
ani_packages() {
  deps = [
    ":ani_ability",
    ":ani_ability_context",
    ":ani_ability_manager",
    ":ani_application",
    # ... 更多模块
  ]
}
```

**输出产物**：
- `libability_ani.z.so`
- `libability_ani.so`

### 5. C API

**BUILD.gn 位置**：`frameworks/c/ability_runtime/BUILD.gn`

**目标类型**：static_library

```gn
source_set("ability_runtime") {
  sources = [
    "ability_runtime_common.cpp",
    # ... 更多文件
  ]
  
  deps = [
    "//foundation/ability/ability_runtime/interfaces/kits/c/ability_runtime:ability_runtime_headers",
  ]
}
```

## 依赖关系图

```
                              ┌─────────────────────────┐
                              │   services/abilitymgr   │
                              │   (AbilityManagerSvc)   │
                              └───────────┬─────────────┘
                                          │
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         frameworks/js/napi                                   │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐               │
│  │ :ability      │  │:ability_manager│  │ :application   │               │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘               │
└──────────┼───────────────────┼───────────────────┼─────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         frameworks/native                                    │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐               │
│  │:ability_thread │  │:appkit        │  │:child_process │               │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘               │
└──────────┼───────────────────┼───────────────────┼─────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         interfaces/inner_api                                 │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐               │
│  │:ability_manager│  │ :app_manager  │  │ :runtime      │               │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘               │
└──────────┼───────────────────┼───────────────────┼─────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         外部依赖                                            │
│                                                                             │
│  bundle_framework  │  ipc_core  │  samgr_client  │  hilog  │  safwk      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 构建命令

### 全量构建

```bash
./build.sh --product-name <product> --build-target ability_runtime
```

### 单独构建 AbilityManagerService

```bash
./build.sh --product-name <product> --build-target ability_manager_service
```

### 单独构建 N-API

```bash
./build.sh --product-name <product> --build-target ability_napi
```

## 构建产物位置

```
out/{product}/ability_runtime/
├── lib/
│   ├── libability_manager_service.so       # AbilityManagerService
│   ├── libapp_manager_service.so           # AppManagerService
│   ├── libability_napi.so                  # N-API 模块
│   ├── libability_ani.so                   # ANI 模块
│   └── ...
└── system/
    └── etc/
        └── sa/
            └── abilitymgr_sa.cfg         # SA 配置
```

## 相关文档

- [编译产物说明](07_Build_Artifacts.md)
- [架构说明](03_Architecture.md)
