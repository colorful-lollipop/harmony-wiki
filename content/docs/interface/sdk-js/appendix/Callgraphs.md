# 关键调用链

## 概述

本附录描述 SDK 构建过程中的关键调用链，包括工具调用链和数据流。

## 工具调用链

### 1. SDK 构建主流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          GN 构建入口                                      │
│                       BUILD.gn:24-67                                    │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ohos_base_split (Action)                             │
│  脚本: //build/ohos/sdk/parse_interface_sdk.py                          │
│  依赖: build_ohos_ets_es2panda                                         │
│  输入: //interface/sdk-js/api/@ohos.*.d.ts                             │
│  输出: ${interface_sdk_path}                                            │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ohos_declaration_template                             │
│  脚本: //interface/sdk-js/remove_internal.py                           │
│  模板: BUILD.gn:131-166                                                 │
│  功能: 过滤 @internal 目录                                               │
│  输入: ${interface_sdk_path}/api                                        │
│  输出: ${root_out_dir}/ohos_declaration/${target_name}/                │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              ohos_handle_declaration_template (可选)                    │
│  脚本: //interface/sdk-js/delete_arkui_label.py                         │
│  模板: BUILD.gn:168-207                                                 │
│  功能: 处理 ArkUI noninterop 标签                                       │
│  输入: ohos_declaration_template 输出                                   │
│  输出: ${target_out_dir}/${target_name}/                                │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      ohos_copy (输出)                                    │
│  功能: 复制到最终 SDK 路径                                               │
│  输入: 处理后的声明文件                                                   │
│  输出: ${ohos_ets_api_path}, ${ohos_ets_arkts_path} 等                  │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据**: `BUILD.gn:24-67`, `BUILD.gn:132-166`, `BUILD.gn:169-207`, `BUILD.gn:581-606`

### 2. Interop 处理流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   build_static_sdk_interop (Action)                     │
│  脚本: //interface/sdk-js/compile_ets_ts.py                            │
│  依赖: build_ohos_ets                                                  │
│  输入: ${ohos_ets_static_path}                                          │
│  输出: ${interface_sdk_path}/static-interop/                           │
│        ├── declaration/                                                 │
│        └── bridge/                                                      │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 build_dynamic_sdk_interop_inner (Action)                  │
│  脚本: //interface/sdk-js/run_compile_declgen.py                        │
│  依赖: build_sdk_interop1_arkui, ohos_ets_dynamic                       │
│  输入: ${ohos_ets_dynamic_two_path}                                     │
│  输出: ${interface_sdk_path}/dynamic-interop/                           │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   ohos_ets_process_interop (Action)                     │
│  脚本: //interface/sdk-js/process_label_noninterop.py                   │
│  依赖: build_dynamic_sdk_interop_inner, build_static_sdk_interop        │
│  输入: ${interface_sdk_path}                                            │
│  输出: ${interface_sdk_path}/arkui_dummy_interop                        │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据**: `BUILD.gn:435-472`, `BUILD.gn:494-519`, `BUILD.gn:731-748`

### 3. 工具链调用关系

```
dts_parser (API 解析)
     │
     ├──► parser (解析 d.ts)
     │        └──► NodeProcessor.ts
     │        └──► JsDocProcessor.ts
     │
     ├──► checker (规范检查)
     │        └──► local_entry.ts
     │
     ├──► diff (差异比较)
     │        └──► DiffProcessor.ts
     │        └──► PermissionsProcessor.ts
     │
     └──► statistics (统计)
              └──► Statistics.ts

api_check_plugin (API 检查)
     │
     ├──► config/ (规则配置)
     │
     └──► src/ (检查逻辑)
              └──► entry.js
              └──► index.js

其他工具
     │
     ├──► arkui_transformer.py
     ├──► process_label_noninterop.py
     ├──► permissions_converter/
     └──► collect_application_api/
```

**证据**: `build-tools/dts_parser/README_zh.md:14-44`

## 数据流

### 1. API 声明数据流

```
Source (.d.ts) ──► Parse ──► Filter ──► Transform ──► Output (SDK)
     │                    │           │
     │                    │           └── delete_arkui_label.py
     │                    └── remove_internal.py
     └── parse_interface_sdk.py
```

### 2. Kit 处理数据流

```
kits/@kit.*.d.ts ──► ohos_copy_internal ──► ohos_copy ──► ohos_ets_kits_path
                          │
                          └── remove_list.json
```

**证据**: `BUILD.gn:330-342`

### 3. ArkTS 处理数据流

```
arkts/@arkts.*.d.ets ──► ohos_copy_internal ──► ohos_ets_*_arkts_path
                                │
                   bundle.json: inner_kits
```

## 关键文件位置

### 构建脚本

| 文件 | 用途 | 调用位置 |
|------|------|---------|
| `parse_interface_sdk.py` | SDK 解析拆分 | BUILD.gn:41 |
| `remove_internal.py` | 内部 API 过滤 | BUILD.gn:149 |
| `delete_arkui_label.py` | ArkUI 标签处理 | BUILD.gn:187 |
| `process_label_noninterop.py` | 互操作标签处理 | BUILD.gn:736 |
| `compile_ets_ts.py` | ETS 编译 | BUILD.gn:445 |

### 工具入口

| 工具 | 入口文件 |
|------|---------|
| dts_parser | `dts_parser/src/main.ts` |
| api_check_plugin | `api_check_plugin/src/index.js` |
| arkui_transformer | `arkui_transformer.py` |

## 相关文档

- [构建工具](../02_Build_Tools.md)
- [GN 配置](../03_Build_Configuration.md)
- [API 声明文件](../01_API_Declarations.md)
