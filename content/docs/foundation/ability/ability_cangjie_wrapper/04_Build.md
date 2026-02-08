# GN 构建配置

## 概述

本文档描述 ability_cangjie_wrapper 子系统的 GN 构建配置，包括所有 targets、依赖关系和编译产物。

## 构建入口

### 根构建文件

**文件路径**：`//foundation/ability/ability_cangjie_wrapper/BUILD.gn`

**主要目标**：
- `copy_sdk_ability_cangjie_libs`：SDK 库复制目标
- `ability_cangjie_wrapper_packages_ohos`：所有 ohos 模块列表
- `ability_cangjie_wrapper_packages_kit`：kit 模块列表

**证据来源**：`BUILD.gn:16-48`

---

## GN Targets 完整清单

### 聚合目标

| Target 名称 | 类型 | 目录 | 主要依赖 | 输出 |
|------------|------|-----|---------|-----|
| `ability_package` | group | `ohos/` | 23 个子模块 | 元目标 |
| `kit.AbilityKit` | ohos_cangjie_shared_library | `kit/AbilityKit/` | 14 个内部模块 | `.so` 包 |

### 核心模块 Targets

| Target 名称 | 类型 | 源码文件 | cj_deps | cj_external_deps | external_deps | 输出 |
|------------|------|---------|---------|------------------|-------------|-----|
| `ohos.app` | ohos_cangjie_shared_library | `app.cj` | - | - | - | `.so` |
| `ohos.ability` | ohos_cangjie_shared_library | `ability.cj` | - | - | - | `.so` |
| `ohos.application` | ohos_cangjie_shared_library | `application.cj` | - | - | - | `.so` |
| `ohos.app.ability` | ohos_cangjie_shared_library | `ability.cj`, `base_context.cj` | - | cangjie_ark_interop | - | `.so` |

### Ability 模块 Targets

| Target 名称 | 类型 | 源码文件 | cj_deps | cj_external_deps | external_deps | 输出 |
|------------|------|---------|---------|------------------|-------------|-----|
| `ohos.app.ability.ability_constant` | ohos_cangjie_shared_library | `ability_constant.cj`, `hilog.cj` | - | cangjie_ark_interop, hiviewdfx_cangjie_wrapper | - | `.so` |
| `ohos.app.ability.common` | ohos_cangjie_shared_library | `common.cj` | ohos.ability.*, ohos.application.* | - | - | `.so` |
| `ohos.app.ability.configuration` | ohos_cangjie_shared_library | `configuration.cj` | - | cangjie_ark_interop | - | `.so` |
| `ohos.app.ability.context_constant` | ohos_cangjie_shared_library | `context_constant.cj` | - | cangjie_ark_interop | - | `.so` |
| `ohos.app.ability.completion_handler` | ohos_cangjie_shared_library | `completion_handler.cj` | - | cangjie_ark_interop | - | `.so` |
| `ohos.app.ability.dialog_request` | ohos_cangjie_shared_library | `dialog_request.cj`, `hilog.cj` | ohos.app.ability.want | cangjie_ark_interop, hiviewdfx_cangjie_wrapper | - | `.so` |
| `ohos.app.ability.open_link_options` | ohos_cangjie_shared_library | `open_link_options.cj` | ohos.app.ability.want | cangjie_ark_interop | - | `.so` |
| `ohos.app.ability.start_options` | ohos_cangjie_shared_library | `start_options.cj` | ohos.app.ability.* | bundlemanager_*, multimedia_*, window_* | - | `.so` |
| `ohos.app.ability.want_constant` | ohos_cangjie_shared_library | `want_constant.cj` | - | cangjie_ark_interop | - | `.so` |

### UIAbility 核心 Target（最复杂）

| Target 名称 | 类型 |
|------------|------|
| `ohos.app.ability.ui_ability` | ohos_cangjie_shared_library |

**源码文件**（12 个）：
- `ability.cj`
- `ability_errorcode.cj`
- `ability_stage_context.cj`
- `application_context.cj`
- `callback_utils.cj`
- `callee.cj`
- `context.cj`
- `context_interop.cj`
- `hilog.cj`
- `system_object_interop_type_to_js.cj`
- `ui_ability.cj`
- `ui_ability_context.cj`

**cj_deps**：
- `ohos.app.ability`
- `ohos.ability.ability_result`
- `ohos.ability.connect_optionsos.application.event_hub`
- ``
- `ohohos.app.ability.ability_constant`
- `ohos.app.ability.completion_handler`
- `ohos.app.ability.configuration`
- `ohos.app.ability.context_constant`
- `ohos.app.ability.dialog_request`
- `ohos.app.ability.open_link_options`
- `ohos.app.ability.start_options`
- `ohos.app.ability.want`

**cj_external_deps**：
- `arkui_cangjie_wrapper:ohos.base`
- `bundlemanager_cangjie_wrapper:ohos.bundle.bundle_manager`
- `cangjie_ark_interop:*`
- `communication_cangjie_wrapper:ohos.rpc`
- `global_cangjie_wrapper:ohos.resource_manager`
- `hiviewdfx_cangjie_wrapper:ohos.hilog`
- `multimedia_cangjie_wrapper:ohos.multimedia.image`
- `window_cangjie_wrapper:ohos.window`

**external_deps**：
- `ability_runtime:abilitykit_native`
- `ability_runtime:appkit_native`
- `ability_runtime:cj_ability_context_native`
- `ability_runtime:cj_ability_ffi`
- `ability_runtime:cj_abilitykit_native_ffi`
- `ability_runtime:cj_context_ffi`
- `ability_runtime:cj_extensionkit_native`
- `ability_runtime:cj_insight_intent_executor`
- `ability_runtime:cj_insightintentcontext`
- `ability_runtime:cj_photo_editor_extension`
- `ability_runtime:cj_ui_extension`
- `ability_runtime:cj_want_agent_ffi`
- `ability_runtime:uiabilitykit_native`
- `access_token:cj_ability_access_ctrl_ffi`

**证据来源**：`ohos/app/ability/ui_ability/BUILD.gn`

### AbilityStage Target

| Target 名称 | 类型 |
|------------|------|
| `ohos.app.ability.ability_stage` | ohos_cangjie_shared_library |

**源码文件**：
- `ability_stage.cj`
- `hilog.cj`

**cj_deps**：
- `ohos.application.event_hub`
- `ohos.app.ability.configuration`
- `ohos.app.ability.context_constant`
- `ohos.app.ability.ui_ability`
- `ohos.app.ability.want`

**cj_external_deps**：
- `bundlemanager_cangjie_wrapper:ohos.bundle.bundle_manager`
- `cangjie_ark_interop:*`
- `global_cangjie_wrapper:ohos.resource_manager`
- `hiviewdfx_cangjie_wrapper:ohos.hilog`

**external_deps**：
- `ability_runtime:appkit_native`

**证据来源**：`ohos/app/ability/ability_stage/BUILD.gn`

### ErrorManager Target

| Target 名称 | 类型 |
|------------|------|
| `ohos.app.ability.error_manager` | ohos_cangjie_shared_library |

**cj_deps**：
- `ohos.application.error_observer`

**external_deps**：
- `ability_runtime:cj_errormanager_ffi`

**证据来源**：`ohos/app/ability/error_manager/BUILD.gn`

### AbilityDelegatorRegistry Target

| Target 名称 | 类型 |
|------------|------|
| `ohos.app.ability.ability_delegator_registry` | ohos_cangjie_shared_library |

**cj_deps**：
- `ohos.app.ability.ui_ability`
- `ohos.app.ability.want`
- `ohos.app.ability.ability_stage`

**external_deps**：
- `ability_runtime:cj_ability_ffi`

**证据来源**：`ohos/app/ability/ability_delegator_registry/BUILD.gn`

### AppRecovery Target

| Target 名称 | 类型 |
|------------|------|
| `ohos.app.ability.app_recovery` | ohos_cangjie_shared_library |

**cj_deps**：
- `ohos.app.ability.want`

**external_deps**：
- `ability_runtime:cj_app_recovery_ffi`

**证据来源**：`ohos/app/ability/app_recovery/BUILD.gn`

### Application 模块 Targets

| Target 名称 | 类型 | external_deps |
|------------|------|--------------|
| `ohos.application.event_hub` | ohos_cangjie_shared_library | - |
| `ohos.application.error_observer` | ohos_cangjie_shared_library | - |
| `ohos.application.test_runner` | ohos_cangjie_shared_library | `ability_runtime:appkit_delegator` |

---

## 依赖关系详解

### 内部模块依赖

#### 核心依赖链

```
kit.AbilityKit
├── ohos.app.ability.ability_constant
├── ohos.app.ability.ability_stage
│   ├── ohos.app.ability.ui_ability (引入完整 UIAbility 依赖树)
│   └── ohos.app.ability.want
├── ohos.app.ability.app_recovery
│   └── ohos.app.ability.want
├── ohos.app.ability.common
│   ├── ohos.ability.ability_result
│   ├── ohos.ability.connect_options
│   ├── ohos.application.error_observer
│   └── ohos.app.ability
├── ohos.app.ability.completion_handler
├── ohos.app.ability.configuration
├── ohos.app.ability.context_constant
├── ohos.app.ability.dialog_request
│   └── ohos.app.ability.want
├── ohos.app.ability.error_manager
│   └── ohos.application.error_observer
├── ohos.app.ability.open_link_options
│   └── ohos.app.ability.want
├── ohos.app.ability.start_options
│   ├── ohos.app.ability.ability_constant
│   ├── ohos.app.ability.completion_handler
│   └── ohos.app.ability.context_constant
├── ohos.app.ability.ui_ability (完整依赖树)
├── ohos.app.ability.want
└── ohos.app.ability.want_constant
```

### 外部子系统依赖

| 子系统 | 依赖模块 | FFI Targets |
|-------|---------|------------|
| **ability_runtime** | 所有模块 | `cj_ability_ffi`, `cj_context_ffi`, `appkit_native`, `abilitykit_native`, `cj_errormanager_ffi`, `cj_app_recovery_ffi` |
| **access_token** | ui_ability | `cj_ability_access_ctrl_ffi` |
| **cangjie_ark_interop** | 所有模块 | `ohos.ffi`, `ohos.labels`, `ohos.business_exception`, `ohos.encoding.json` |
| **arkui_cangjie_wrapper** | ui_ability | `ohos.base` |
| **bundlemanager_cangjie_wrapper** | ui_ability, want, start_options | `ohos.bundle.bundle_manager`, `ohos.element_name` |
| **communication_cangjie_wrapper** | ui_ability, connect_options | `ohos.rpc` |
| **global_cangjie_wrapper** | ui_ability, ability_stage | `ohos.resource_manager` |
| **hiviewdfx_cangjie_wrapper** | 所有日志模块 | `ohos.hilog` |
| **window_cangjie_wrapper** | ui_ability, start_options | `ohos.window` |
| **multimedia_cangjie_wrapper** | ui_ability, start_options | `ohos.multimedia.image` |
| **testfwk_cangjie_wrapper** | test_runner | `ohos.ui_test` |

---

## 编译产物

### 产物清单

| 产物类型 | 说明 | 安装路径 |
|---------|------|---------|
| `.so` 共享库 | 各模块的编译产物 | `system/lib/` |
| `.cj` 包 | Cangjie 源码包 | SDK 目录 |
| SDK 聚合包 | SDK 级别聚合 | `sdk/` |

### 产物与 Target 映射

| Target | 产物 | 说明 |
|--------|-----|------|
| `kit.AbilityKit` | `libkit.AbilityKit.z.so` | Kit 级别聚合库 |
| `ohos.app.ability.ui_ability` | `libohos.app.ability.ui_ability.z.so` | UIAbility 核心库 |
| `ohos.app.ability.ability_stage` | `libohos.app.ability.ability_stage.z.so` | AbilityStage 库 |
| `ohos.app.ability.error_manager` | `libohos.app.ability.error_manager.z.so` | 错误管理库 |
| `ohos.app.ability.app_recovery` | `libohos.app.ability.app_recovery.z.so` | 应用恢复库 |

### 运行时加载关系

```
应用启动
    │
    ▼
加载 lib AbilityRuntime 库
    │
    ├── 加载 cj_ability_ffi (FFI 接口)
    │   │
    │   ├── 加载 libohos.app.ability.ui_ability.z.so (按需)
    │   │   │
    │   │   ├── 依赖 libkit.AbilityKit.z.so
    │   │   │   │
    │   │   │   ├── 依赖 libohos.app.ability.*.z.so (各子模块)
    │   │   │   └── 依赖其他子系统库
    │   │   │
    │   │   └── 依赖 abilitykit_native
    │   │
    │   └── 依赖 cj_context_ffi, cj_want_agent_ffi 等
    │
    └── 加载 appkit_native
        │
        └── 加载 libohos.app.ability.ability_stage.z.so (按需)
            │
            └── 依赖 appkit_native
```

---

## 构建配置详情

### 构建模板

所有模块使用 `ohos_cangjie_shared_library` 模板：

```gn
ohos_cangjie_shared_library("target_name") {
    sources = [...]
    cj_deps = [...]
    cj_external_deps = [...]
    external_deps = [...]
    subsystem_name = "ability"
    part_name = "ability_cangjie_wrapper"
}
```

### 配置参数说明

| 参数 | 类型 | 说明 |
|-----|------|-----|
| `sources` | list | 源码文件列表 |
| `cj_deps` | list | 内部 Cangjie 模块依赖 |
| `cj_external_deps` | list | 外部 Cangjie 包装器依赖 |
| `external_deps` | list | 原生 FFI 库依赖 |
| `subsystem_name` | string | 子系统名称 |
| `part_name` | string | 部件名称 |

### 条件编译

```gn
if (is_mingw || is_mac) {
    # Windows/Mac 平台使用 mock 源码
    sources = [ "mock/..." ]
} else {
    # Linux 平台使用实际实现
    sources = [ "real/..." ]
}
```

**证据来源**：`BUILD.gn:14` - 模板导入与条件编译

---

## 组件配置

**文件**：`bundle.json`

```json
{
    "name": "@ohos/ability_cangjie_wrapper",
    "subsystem": "ability",
    "adapted_system_type": ["standard"],
    "rom": "1500KB",
    "ram": "1536KB",
    "deps": {
        "components": [
            "ability_runtime",
            "access_token",
            "cangjie_ark_interop",
            "arkui_cangjie_wrapper",
            "hiviewdfx_cangjie_wrapper",
            "bundlemanager_cangjie_wrapper",
            "communication_cangjie_wrapper",
            "global_cangjie_wrapper",
            "multimedia_cangjie_wrapper",
            "window_cangjie_wrapper",
            "accesscontrol_cangjie_wrapper",
            "testfwk_cangjie_wrapper"
        ]
    },
    "build": {
        "sub_component": [
            "//foundation/ability/ability_cangjie_wrapper/ohos:ability_package",
            "//foundation/ability/ability_cangjie_wrapper/kit/AbilityKit:kit.AbilityKit"
        ]
    }
}
```

**证据来源**：`bundle.json:1-74`

---

## 常见构建问题

### 问题 1：FFI 依赖缺失

**症状**：`ninja: error: unknown target '//foundation/ability/ability_cangjie_wrapper/...'`

**原因**：`external_deps` 中引用的 FFI target 未构建

**解决**：
```bash
# 确保 ability_runtime 子系统已构建
hb build -p ability_runtime

# 或同步构建
hb build -T all
```

### 问题 2：循环依赖

**症状**：`GN detected a cycle in dependencies`

**原因**：模块间存在环形依赖

**证据来源**：UIAbility ↔ AbilityStage 存在潜在循环（依赖链验证中）

### 问题 3：平台条件编译

**症状**：Windows/Mac 平台构建失败

**原因**：某些模块在非 Linux 平台不可用

**解决**：
```bash
# 使用 Linux 构建环境
hb set -p <linux_device>
hb build
```
