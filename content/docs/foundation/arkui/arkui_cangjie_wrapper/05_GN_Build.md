# GN 构建系统 (GN Build)

## 概述

本文档描述 arkui_cangjie_wrapper 的 GN 构建系统，包括关键 targets、依赖关系和编译产物。

### 构建系统

| 系统 | 版本/模板 |
|------|------------|
| **构建工具** | GN (Generate Ninja) |
| **Cangjie 模板** | `//build/templates/cangjie/cjc.gni` |
| **目标平台** | OpenHarmony Standard |

---

## 根构建文件

### BUILD.gn

**路径**: `//foundation/arkui/arkui_cangjie_wrapper/BUILD.gn`

#### 主入口

```gn
group("arkui_cangjie_wrapper_package") {
  deps = [
    "ohos:ohos",
    "ohos/arkui:ohos.arkui",
    "ohos/arkui/component:ohos.arkui.component",
    "ohos/arkui/component_utils:ohos.arkui.component_utils",
    "ohos/arkui/shape:ohos.arkui.shape",
    "ohos/arkui/state_management:ohos.arkui.state_management",
    "ohos/arkui/ui_context:ohos.arkui.ui_context",
    "ohos/base:ohos.base",
    "ohos/curves:ohos.curves",
  ]
}
```

### bundle.json 配置

**路径**: `bundle.json`

#### 子组件定义

```json
{
  "name": "@ohos/arkui_cangjie_wrapper",
  "version": "6.1",
  "component": {
    "name": "arkui_cangjie_wrapper",
    "subsystem": "arkui",
    "features": [],
    "adapted_system_type": ["standard"],
    "rom": "8930KB",
    "ram": "8100KB"
  }
}
```

---

## Kit 构建 (kit/ArkUI)

### BUILD.gn

**路径**: `kit/ArkUI/BUILD.gn`

```gn
ohos_cangjie_shared_library("kit.ArkUI") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/arkui/component:ohos.arkui.component",
    "../../ohos/arkui/component_utils:ohos.arkui.component_utils",
    "../../ohos/arkui/shape:ohos.arkui.shape",
    "../../ohos/arkui/state_management:ohos.arkui.state_management",
    "../../ohos/arkui/ui_context:ohos.arkui.ui_context",
    "../../ohos/base:ohos.base",
    "../../ohos/curves:ohos.curves",
  ]

  cj_external_deps = [
    "window_cangjie_wrapper:ohos.display",
    "window_cangjie_wrapper:ohos.window",
  ]

  subsystem_name = "arkui"
  part_name = "arkui_cangjie_wrapper"
}
```

### 入口文件

**路径**: `kit/ArkUI/index.cj`

```cangjie
package kit.ArkUI

public import ohos.display.*
public import ohos.window.*
public import ohos.base.*
public import ohos.arkui.component.*
public import ohos.arkui.state_management.*
public import ohos.curves.*
public import ohos.arkui.component_utils.*
public import ohos.arkui.shape.*
public import ohos.arkui.ui_context.*
```

---

## ohos 构建

### 根 BUILD.gn

**路径**: `ohos/ohos.cj`

聚合导出所有 ohos 模块。

---

### ohos/arkui 构建

#### BUILD.gn

**路径**: `ohos/arkui/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.arkui") {
  sources = [ "arkui.cj" ]

  cj_deps = [
    "component:ohos.arkui.component",
    "component_utils:ohos.arkui.component_utils",
    "shape:ohos.arkui.shape",
    "state_macro_manage:ohos.arkui.state_macro_manage",
    "state_management:ohos.arkui.state_management",
    "ui_context:ohos.arkui.ui_context",
  ]

  subsystem_name = "arkui"
  part_name = "arkui_cangjie_wrapper"
}
```

---

### ohos/arkui/component 构建

#### BUILD.gn

**路径**: `ohos/arkui/component/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.arkui.component") {
  sources = [ "component.cj" ]

  cj_deps = [
    "action_sheet:ohos.arkui.component.action_sheet",
    "alert_dialog:ohos.arkui.component.alert_dialog",
    // ... 80+ components
    "video:ohos.arkui.component.video",
    "custom_component:ohos.arkui.component.custom_component",
    "web:ohos.arkui.component.web",
  ]

  external_deps = [ "ace_engine:cj_frontend_ohos" ]
  subsystem_name = "arkui"
  part_name = "arkui_cangjie_wrapper"
}
```

---

### ohos/base 构建

#### BUILD.gn

**路径**: `ohos/base/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.base") {
  sources = [
    "callback_type.cj",
    "cj_concurrency.cj",
    "collection.cj",
    "collection_extend.cj",
    "color.cj",
    "common_types.cj",
    "cstring_extend.cj",
    "length.cj",
    "main_context.cj",
    "resource.cj",
    "reuse_params.cj",
  ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.encoding.json",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.business_exception",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "ace_engine:cj_frontend_ohos" ]

  subsystem_name = "arkui"
  part_name = "arkui_cangjie_wrapper"
}
```

---

## Targets 清单

### 按模块分组

| 模块 | Target 类型 | 输出名 | 关键依赖 |
|------|-------------|--------|----------|
| **kit.ArkUI** | shared_library | kit.ArkUI | 7 个 cj_deps + 2 个 external |
| **ohos.arkui** | shared_library | ohos.arkui | 6 个子模块 |
| **ohos.arkui.component** | shared_library | ohos.arkui.component | 80+ 组件 + ace_engine |
| **ohos.arkui.shape** | shared_library | ohos.arkui.shape | 5 个形状组件 |
| **ohos.arkui.state_management** | shared_library | ohos.arkui.state_management | 11 个状态管理模块 |
| **ohos.arkui.ui_context** | shared_library | ohos.arkui.ui_context | 6 个上下文模块 |
| **ohos.base** | shared_library | ohos.base | 11 个基础类型 |
| **ohos.curves** | shared_library | ohos.curves | 曲线定义 |

### 按类型分组

#### Cangjie 共享库 (ohos_cangjie_shared_library)

| Target | Sources | 关键 cj_deps |
|--------|---------|--------------|
| kit.ArkUI | index.cj | 7 个模块依赖 |
| ohos.arkui | arkui.cj | 6 个子模块 |
| ohos.arkui.component | component.cj | 80+ 组件 |
| ohos.base | 11 个 .cj 文件 | cangjie_ark_interop, hiviewdfx |

#### SDK 复制任务 (copy_ohos_cangjie_sdk_api_lib)

```gn
copy_ohos_cangjie_sdk_api_lib("copy_sdk_arkui_cangjie_libs") {
  ohos_inputs = arkui_cangjie_wrapper_packages_ohos
  kit_inputs = arkui_cangjie_wrapper_packages_kit
}
```

---

## 依赖关系详解

### 内部依赖 (cj_deps)

```
kit.ArkUI
├── ohos.arkui.component
├── ohos.arkui.component_utils
├── ohos.arkui.shape
├── ohos.arkui.state_management
├── ohos.arkui.ui_context
├── ohos.base
└── ohos.curves

ohos.arkui
├── ohos.arkui.component
├── ohos.arkui.component_utils
├── ohos.arkui.shape
├── ohos.arkui.state_macro_manage
├── ohos.arkui.state_management
└── ohos.arkui.ui_context
```

### 外部依赖 (external_deps / cj_external_deps)

| 依赖项 | 类型 | 用途 |
|--------|------|------|
| `ace_engine:cj_frontend_ohos` | external_deps | Cangjie 前端引擎 |
| `window_cangjie_wrapper:ohos.display` | cj_external_deps | 显示管理 |
| `window_cangjie_wrapper:ohos.window` | cj_external_deps | 窗口管理 |
| `cangjie_ark_interop:ohos.encoding.json` | cj_external_deps | JSON 编码 |
| `cangjie_ark_interop:ohos.ffi` | cj_external_deps | FFI 支持 |
| `cangjie_ark_interop:ohos.labels` | cj_external_deps | 标签系统 |
| `cangjie_ark_interop:ohos.business_exception` | cj_external_deps | 业务异常 |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | cj_external_deps | 日志 |

---

## 编译产物

### 预期产物

| 产物类型 | 路径模式 | 说明 |
|----------|----------|------|
| **.so 库** | `out/{device}/libs/libarkui_cangjie_*.so` | Cangjie 运行时库 |
| **.cj_pkg** | `out/{device}/cjpkg/` | Cangjie 包 |
| **SDK** | `sdk/{version}/packages/` | 分发给开发者的 SDK |

### 产物映射

| Target | 产物 | 安装路径 |
|--------|------|----------|
| kit.ArkUI | libkit.ArkUI.so | system/lib/module/arkui/ |
| ohos.arkui | libohos.arkui.so | system/lib/module/arkui/ |
| ohos.base | libohos.base.so | system/lib/module/arkui/ |

---

## 构建命令

### 完整构建

```bash
# 构建整个 subsystem
hb set
hb build arkui_cangjie_wrapper

# 或使用 gn + ninja
gn gen out/{device}
ninja -C out/{device} arkui_cangjie_wrapper
```

### 模块构建

```bash
# 构建特定模块
ninja -C out/{device} //foundation/arkui/arkui_cangjie_wrapper/kit/ArkUI:kit.ArkUI

# 构建所有 ohos 模块
ninja -C out/{device} //foundation/arkui/arkui_cangjie_wrapper/ohos/arkui/component:ohos.arkui.component
```

---

## 代码证据

| 元素 | 证据位置 |
|------|----------|
| 根构建 | `BUILD.gn:14-28` |
| Kit 构建 | `kit/ArkUI/BUILD.gn:17-37` |
| 组件构建 | `ohos/arkui/component/BUILD.gn:16-107` |
| 基础构建 | `ohos/base/BUILD.gn:16-43` |
| bundle.json | `bundle.json:1-64` |

---

## 相关文档

- [01_Directory_Structure.md](./01_Directory_Structure.md) - 目录结构
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [07_Troubleshooting.md](./07_Troubleshooting.md) - 构建问题
