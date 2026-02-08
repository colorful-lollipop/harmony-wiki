# GN Targets 与编译配置

> **目的**: 了解 powermgr_cangjie_wrapper 的构建目标、依赖关系和配置
> **适用范围**: 构建调试、依赖分析、配置修改
> **最后更新**: 2025-02-06

---

## GN 构建系统概览

**构建工具**: GN (Generate Ninja) + Ninja

**模板**: `import("//build/templates/cangjie/cjc.gni")`

**证据**: `BUILD.gn:14`

---

## 关键 BUILD.gn 文件

### 1. 根 BUILD.gn

**路径**: `BUILD.gn`

**证据**: `BUILD.gn:1-21`

#### 内容分析

```gn
import("//build/templates/cangjie/cjc.gni")

powermgr_cangjie_wrapper_packages_ohos = [
  "//base/powermgr/powermgr_cangjie_wrapper/ohos/battery_info:ohos.battery_info"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_powermgr_cangjie_libs") {
  ohos_inputs = powermgr_cangjie_wrapper_packages_ohos
}
```

**说明**:
- **导入模板**: `cjc.gni` - Cangjie 编译模板
- **定义包列表**: `powermgr_cangjie_wrapper_packages_ohos`
- **创建 copy target**: `copy_sdk_powermgr_cangjie_libs` - 复制 SDK 文件

---

### 2. ohos/battery_info/BUILD.gn

**路径**: `ohos/battery_info/BUILD.gn`

**证据**: `ohos/battery_info/BUILD.gn:1-40`

#### 主要 Target: ohos.battery_info

```gn
ohos_cangjie_shared_library("ohos.battery_info") {
  if (is_mingw || is_mac){
    sources = [ "../../mock/ohos.battery_info.cj" ]
  } else {
    sources = [
      "battery_info.cj",
      "native.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.labels",
  ]

  external_deps = [ "battery_manager:cj_battery_info_ffi" ]

  subsystem_name = "powermgr"
  part_name = "powermgr_cangjie_wrapper"
}
```

**说明**:
- **Target 类型**: `ohos_cangjie_shared_library` - Cangjie 共享库
- **Target 名称**: `ohos.battery_info`
- **条件编译**: Windows/macOS 使用 mock 实现
- **子系统**: `powermgr`
- **组件**: `powermgr_cangjie_wrapper`

---

## Targets 列表

### 1. ohos.battery_info

| 属性 | 值 |
|-----|-----|
| **类型** | `ohos_cangjie_shared_library` |
| **名称** | `ohos.battery_info` |
| **位置** | `ohos/battery_info/BUILD.gn:19` |
| **输出** | Cangjie 共享库 (.so) |
| **源文件** | `battery_info.cj`, `native.cj` |
| **Mock 源文件** | `mock/ohos.battery_info.cj` |
| **Cangjie 依赖** | `cangjie_ark_interop:ohos.business_exception`, `cangjie_ark_interop:ohos.labels` |
| **C 依赖** | `battery_manager:cj_battery_info_ffi` |
| **子系统** | `powermgr` |
| **组件** | `powermgr_cangjie_wrapper` |
| **条件编译** | `is_mingw || is_mac` → mock |

**证据**: `ohos/battery_info/BUILD.gn:19-39`

### 2. copy_sdk_powermgr_cangjie_libs

| 属性 | 值 |
|-----|-----|
| **类型** | `copy_ohos_cangjie_sdk_api_lib` |
| **名称** | `copy_sdk_powermgr_cangjie_libs` |
| **位置** | `BUILD.gn:19` |
| **输出** | SDK 文件复制 |
| **输入** | `["//base/powermgr/powermgr_cangjie_wrapper/ohos/battery_info:ohos.battery_info"]` |
| **用途** | 将编译产物复制到 SDK 目录 |

**证据**: `BUILD.gn:19-21`

---

## 依赖关系图

```
ohos.battery_info (ohos_cangjie_shared_library)
  │
  ├── cj_external_deps (Cangjie 库)
  │     ├── cangjie_ark_interop:ohos.business_exception
  │     └── cangjie_ark_interop:ohos.labels
  │
  ├── external_deps (C 库)
  │     └── battery_manager:cj_battery_info_ffi
  │
  └── sources (源文件)
        ├── battery_info.cj (生产)
        ├── native.cj (生产)
        └── ohos.battery_info.cj (mock, 条件编译)

copy_sdk_powermgr_cangjie_libs (copy_ohos_cangjie_sdk_api_lib)
  │
  └── ohos_inputs
        └── ohos.battery_info
```

**证据**: 综合自 `BUILD.gn` 和 `ohos/battery_info/BUILD.gn`

---

## 条件编译

### Mock 实现

**条件**: `is_mingw || is_mac`

**生效环境**:
- Windows (MinGW)
- macOS

**源文件**: `mock/ohos.battery_info.cj`

**生产实现源文件**: `battery_info.cj`, `native.cj`

**证据**: `ohos/battery_info/BUILD.gn:21-28`

### 编译选择流程

```
编译开始
  │
  ├─→ is_mingw || is_mac ?
  │     ├─ 是 ─→ 使用 mock/ohos.battery_info.cj
  │     └─ 否 ─→ 使用 battery_info.cj + native.cj
  │
  └─→ 编译输出 ohos.battery_info
```

---

## Target 详细配置

### ohos.battery_info 详细配置

#### sources (源文件)

**生产环境**:
```gn
sources = [
  "battery_info.cj",    # 450 行，主 API 定义
  "native.cj",          # 41 行，FFI 声明
]
```

**Mock 环境**:
```gn
sources = [ "../../mock/ohos.battery_info.cj" ]
```

**证据**: `ohos/battery_info/BUILD.gn:22-28`

#### cj_external_deps (Cangjie 依赖)

```gn
cj_external_deps = [
  "cangjie_ark_interop:ohos.business_exception",
  "cangjie_ark_interop:ohos.labels",
]
```

**说明**:
- `ohos.business_exception`: 提供业务异常类
- `ohos.labels`: 提供 API Level 标注

**证据**: `ohos/battery_info/BUILD.gn:30-33`

#### external_deps (C 依赖)

```gn
external_deps = [ "battery_manager:cj_battery_info_ffi" ]
```

**说明**:
- `battery_manager`: 电池管理服务组件
- `cj_battery_info_ffi`: Cangjie FFI 接口库

**证据**: `ohos/battery_info/BUILD.gn:35`

#### subsystem_name & part_name

```gn
subsystem_name = "powermgr"
part_name = "powermgr_cangjie_wrapper"
```

**说明**:
- `subsystem_name`: 电源管理子系统
- `part_name`: 组件名称

**证据**: `ohos/battery_info/BUILD.gn:37-38`

---

## 组件声明 (bundle.json)

**路径**: `bundle.json`

**证据**: `bundle.json:1-43`

### 基本信息

| 属性 | 值 |
|-----|-----|
| **name** | `@ohos/powermgr_cangjie_wrapper` |
| **description** | Display of charge-discharge, battery, and state of charge information |
| **version** | `6.1` |
| **license** | `Apache-2.0` |
| **publishAs** | `code-segment` |

**证据**: `bundle.json:2-6`

### 组件配置

| 属性 | 值 |
|-----|-----|
| **subsystem** | `powermgr` |
| **syscap** | `[]` (空) |
| **adapted_system_type** | `["standard"]` |
| **rom** | `100KB` |
| **ram** | `72KB` |
| **deps.components** | `["cangjie_ark_interop", "battery_manager"]` |

**证据**: `bundle.json:14-26`

### 构建配置

#### sub_component (子组件)

```json
"build": {
  "sub_component": [
    "//base/powermgr/powermgr_cangjie_wrapper/ohos/battery_info:ohos.battery_info"
  ]
}
```

**证据**: `bundle.json:29-31`

#### inner_kits (内部套件)

```json
"inner_kits": [
  {
    "name": "//base/powermgr/powermgr_cangjie_wrapper/ohos/battery_info:ohos.battery_info"
  },
  {
    "name": "//base/powermgr/powermgr_cangjie_wrapper:copy_sdk_powermgr_cangjie_libs"
  }
]
```

**证据**: `bundle.json:32-38`

---

## Target ↔ 产物映射

### 预期编译产物

| Target | 类型 | 预期输出 | 安装路径 |
|--------|------|---------|---------|
| `ohos.battery_info` | `ohos_cangjie_shared_library` | `libohos.battery_info.so` | `/system/lib64/` 或 `/system/lib/` |
| `copy_sdk_powermgr_cangjie_libs` | `copy_ohos_cangjie_sdk_api_lib` | SDK 文件 | SDK 目录 |

**说明**:
- Cangjie 共享库通常输出 `.so` 文件
- 具体路径和名称需查看构建输出确认

**TODO(需确认)**: 实际编译产物的完整路径和名称

---

## 构建命令示例

### 编译整个组件

```bash
# 编译 powermgr_cangjie_wrapper 组件
./build.sh --product-name <product> --build-target powermgr_cangjie_wrapper

# 或编译子系统
./build.sh --product-name <product> --build-target powermgr
```

### 仅编译 battery_info 模块

```bash
./build.sh --product-name <product> --build-target ohos.battery_info
```

---

## 特性开关

### 当前状态

**无特性开关**: 项目中没有定义任何特性开关（feature flags）

**证据**: 搜索 `.gni` 配置文件无结果，BUILD.gn 中无 `defines` 或 `configs`

### 条件编译

唯一的条件编译是 Mock 实现：

```gn
if (is_mingw || is_mac){
  sources = [ "../../mock/ohos.battery_info.cj" ]
}
```

**证据**: `ohos/battery_info/BUILD.gn:21-28`

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [模块职责](01_Module_Responsibilities.md) - 目录结构和模块划分
- [编译产物](06_Build_Artifacts.md) - 产物清单和安装路径
- [内部 API](04_Internal_API.md) - FFI 接口详解
