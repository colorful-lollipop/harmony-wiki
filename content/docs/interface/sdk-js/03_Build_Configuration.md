# GN 构建配置

## 概述

本章节描述 `BUILD.gn` 和相关 GN 配置文件的结构与关键 targets。

## 关键配置文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | GN 构建主文件 (784行) |
| `interface_config.gni` | 接口配置 (路径、类型) |
| `bundle.json` | 组件配置 (子系统、部件) |
| `OAT.xml` | OSS 审计配置 |

## BUILD.gn 结构

```
BUILD.gn
├── 导入部分 (14-19行)
│   ├── ets2abc_config.gni
│   ├── ohos.gni
│   ├── notice.gni
│   ├── ohos_var.gni
│   └── metadata/module_info.gni
│
├── 基础处理 (24-80行)
│   ├── ohos_base_split      # 基础 SDK 拆分
│   └── build_ohos_ets_es2panda  # 依赖组
│
├── 模板定义 (82-207行)
│   ├── ohos_copy_internal        # 内部 API 复制
│   ├── ohos_declaration_template # 声明处理
│   └── ohos_handle_declaration_template  # ArkUI 处理
│
├── Dynamic SDK Targets (210-431行)
│   ├── ohos_build_dynamic_sdk_api
│   ├── ohos_build_dynamic_sdk_arkts
│   ├── ohos_build_dynamic_sdk_component
│   └── ohos_build_dynamic_sdk_kits
│
├── Static SDK Targets (229-432行)
│   ├── ohos_build_static_sdk_api
│   ├── ohos_build_static_sdk_arkts
│   └── ohos_build_static_sdk_kits
│
├── Interop 处理 (434-783行)
│   ├── build_static_sdk_interop
│   ├── build_dynamic_sdk_interop_inner
│   └── ohos_ets_process_interop
│
└── 输出 Targets (581-617行)
    ├── ohos_ets_api
    ├── ohos_ets_arkts
    └── ohos_ets_kits
```

## 关键 Targets

### 模板 Targets

#### ohos_declaration_template

**用途**: 处理 API 声明文件，过滤内部 API

**证据**: `BUILD.gn:131-166`

```gn
template("ohos_declaration_template") {
  forward_variables_from(invoker, "*")
  input_project_dir = invoker.input_project_dir

  action_with_pydeps(target_name) {
    deps = [ ":ohos_base_split" ]
    script = "//interface/sdk-js/remove_internal.py"
    args = [
      "--input", rebase_path(input_api_dir, root_build_dir),
      "--output", rebase_path(root_out_dir + "/ohos_declaration/${target_name}/", root_build_dir),
    ]
  }
}
```

#### ohos_copy_internal

**用途**: 复制内部 API，处理 @internal 目录

**证据**: `BUILD.gn:82-129`

```gn
template("ohos_copy_internal") {
  forward_variables_from(invoker, "*")
  iv_input = invoker.iv_input

  action_with_pydeps(target_name) {
    script = "//interface/sdk-js/process_internal.py"
    args = [
      "--input", rebase_path(iv_input, root_build_dir),
      "--remove", rebase_path("//interface/sdk-js/remove_list.json", root_build_dir),
      "--ispublic", "${sdk_build_public}",
    ]
  }
}
```

#### ohos_handle_declaration_template

**用途**: 处理 ArkUI 的 noninterop 标签

**证据**: `BUILD.gn:168-207`

```gn
template("ohos_handle_declaration_template") {
  forward_variables_from(invoker, "*")

  action_with_pydeps(target_name) {
    script = "//interface/sdk-js/delete_arkui_label.py"
    args = [
      "--root-build-dir", rebase_path("//", root_build_dir),
      "--input-interface-sdk", rebase_path(input_project_dir),
      "--output-arkui-interface-sdk", rebase_path("${target_out_dir}/${target_name}"),
    ]
  }
}
```

### SDK 构建 Targets

#### Dynamic SDK

| Target | 用途 |
|--------|------|
| `ohos_build_dynamic_sdk_api` | 动态 SDK API |
| `ohos_build_dynamic_sdk_arkts` | 动态 SDK ArkTS |
| `ohos_build_dynamic_sdk_component` | 动态 SDK 组件 |
| `ohos_build_dynamic_sdk_kits` | 动态 SDK Kits |

**证据**: `BUILD.gn:210-227`

#### Static SDK

| Target | 用途 |
|--------|------|
| `ohos_build_static_sdk_api` | 静态 SDK API |
| `ohos_build_static_sdk_arkts` | 静态 SDK ArkTS |
| `ohos_build_static_sdk_kits` | 静态 SDK Kits |

**证据**: `BUILD.gn:229-242`

### 输出 Targets

| Target | 类型 | 输出路径 | Part/Subsystem |
|--------|------|---------|----------------|
| `ohos_ets_api` | ohos_copy | `${ohos_ets_api_path}` | sdk |
| `ohos_ets_arkts` | ohos_copy | `${ohos_ets_arkts_path}` | sdk |
| `ohos_ets_kits` | ohos_copy | `${ohos_ets_kits_path}` | sdk |

**证据**: `BUILD.gn:581-606`

### Inner Kits (bundle.json)

**证据**: `bundle.json:35-76`

```json
{
  "inner_kits": [
    { "name": "//interface/sdk-js:ohos_ets_api" },
    { "name": "//interface/sdk-js:ohos_ets_arkts" },
    { "name": "//interface/sdk-js:ohos_ets_kits" },
    { "name": "//interface/sdk-js:ets_component" },
    { "name": "//interface/sdk-js:ohos_declaration_ets" },
    { "name": "//interface/sdk-js:bundle_kits" },
    { "name": "//interface/sdk-js:bundle_arkts" },
    { "name": "//interface/sdk-js:ets_internal_api" },
    { "name": "//interface/sdk-js:ohos_declaration_ets_api" }
  ]
}
```

## interface_config.gni 配置

### SDK 类型

```gn
sdk_type = "ets"
```

### SDK 路径配置

**Dynamic SDK 路径**:

| 变量 | 路径 | 用途 |
|------|------|------|
| `ohos_ets_dynamic_path` | `${root_build_dir}/ohos_dynamic` | 根路径 |
| `ohos_ets_dynamic_api_path` | `/api` | API 目录 |
| `ohos_ets_dynamic_arkts_path` | `/arkts` | ArkTS 目录 |
| `ohos_ets_dynamic_component_path` | `/component` | 组件目录 |
| `ohos_ets_dynamic_kits_path` | `/kits` | Kits 目录 |
| `ohos_ets_dynamic_two_path` | `${root_build_dir}/ohos_dynamic_two` | 第二路径 |

**Static SDK 路径**:

| 变量 | 路径 | 用途 |
|------|------|------|
| `ohos_ets_static_path` | `${root_build_dir}/ohos_static` | 根路径 |
| `ohos_ets_static_api_path` | `/api` | API 目录 |
| `ohos_ets_static_arkts_path` | `/arkts` | ArkTS 目录 |
| `ohos_ets_static_kits_path` | `/kits` | Kits 目录 |

**证据**: `interface_config.gni:31-42`

### 通用 API 源文件

```gn
common_api_src = [
  "@system.app.d.ts",
  "@system.configuration.d.ts",
  "@system.file.d.ts",
  "@system.mediaquery.d.ts",
  "@system.prompt.d.ts",
  "@system.router.d.ts",
]
```

**证据**: `interface_config.gni:15-22`

## 构建产物

### 产物类型

| 产物类型 | 说明 |
|---------|------|
| `.d.ts` 文件 | TypeScript 声明文件 |
| SDK 目录 | 完整的 SDK 包 |

### 产物路径

| SDK 类型 | 输出根路径 |
|---------|-----------|
| Dynamic SDK | `${root_build_dir}/ohos_dynamic/` |
| Static SDK | `${root_build_dir}/ohos_static/` |

### 产物结构

```
output/
├── api/                    # API 声明
│   ├── @ohos.*.d.ts
│   ├── @kit.*.d.ts
│   └── @internal/          # 内部 API (可选)
├── arkts/                  # ArkTS 内置
│   ├── @arkts.*.d.ets
│   └── ...
├── kits/                   # Kits 声明
│   └── @kit.*.d.ts
└── component/              # 组件 (仅动态)
    └── ...
```

## 构建流程

```
┌─────────────────────────────────────────────────────────────────┐
│ Phase 1: 基础拆分                                                 │
│ 脚本: parse_interface_sdk.py                                       │
│ 输入: api/@ohos.*.d.ts                                            │
│ 输出: interface_sdk_path (中间产物)                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 2: 声明处理                                                  │
│ 模板: ohos_declaration_template                                    │
│ 脚本: remove_internal.py                                          │
│ 功能: 过滤 @internal 目录                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 3: ArkUI 处理                                               │
│ 模板: ohos_handle_declaration_template                            │
│ 脚本: delete_arkui_label.py                                       │
│ 功能: 处理 noninterop 标签                                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 4: 输出复制                                                  │
│ 模板: ohos_copy                                                   │
│ 输出: 最终 SDK 目录                                                │
└─────────────────────────────────────────────────────────────────┘
```

**证据**: `BUILD.gn:24-67`, `BUILD.gn:132-166`, `BUILD.gn:169-207`

## 相关文档

- [API 声明文件](01_API_Declarations.md)
- [构建工具](02_Build_Tools.md)
- [安全评审](04_Security_Review.md)
