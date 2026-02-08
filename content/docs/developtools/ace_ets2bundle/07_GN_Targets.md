# GN 构建目标

## 根 BUILD.gn 概述

**文件位置**：`BUILD.gn`

**主要 Targets**：

| Target | 类型 | 产物 | 职责 |
|--------|------|------|------|
| `build_ets_loader_library` | action | 目录 | 构建 ETS 加载器库 |
| `build_ets_sysResource` | action | sysResource.js | 生成系统资源 |
| `ets_loader` | ohos_copy | 目录 | 复制加载器文件 |
| `ets_loader_component_config` | ohos_copy | .json | 组件配置 |
| `ets_loader_form_config` | ohos_copy | .json | 表单配置 |
| `ets_loader_library` | ohos_copy | 目录 | 库文件 |
| `ets_loader_declaration` | ohos_copy | 目录 | 声明文件 |
| `ets_loader_ark` | ohos_copy | 目录 | Ark 运行时加载器 |
| `ets_loader_ark_hap` | ohos_prebuilt_etc | .xml | HAP 配置 |
| `arkui-plugins` | ohos_copy | 目录 | UI 插件 |
| `ohos_ets_koala_wrapper` | ohos_copy | 目录 | Koala 包装器 |

## build_ets_loader_library

**类型**：action

**职责**：执行构建脚本生成 ETS 加载器库。

**关键代码位置**：`BUILD.gn:46-160`

**依赖**：

```gn
deps = [
  ":components",
  ":form_components",
  ":insight_intents",
  ":install_arkguard_tsc_declgen",
  ":server",
]
```

**输入脚本**：

| 脚本 | 说明 |
|------|------|
| `compiler/node_modules/@babel/cli/bin/babel.js` | Babel 转译 |
| `compiler/babel.config.js` | Babel 配置 |
| `compiler/uglify-source.js` | 代码压缩 |
| `compiler/build_declarations_file.js` | 声明文件生成 |
| `compiler/build_kitConfigs_file.js` | Kit 配置生成 |

**输出产物**：

| 产物 | 说明 |
|------|------|
| `lib/` | 转译后的 JS 库文件 |
| `declarations/` | 类型声明文件 |
| `component_config.json` | 组件配置 |
| `form_config.json` | 表单配置 |
| `kit_configs/` | Kit 配置目录 |

## ets_loader_ark_hap_inner

**类型**：group

**职责**：聚合 Ark HAP 相关的所有产物。

**关键代码位置**：`BUILD.gn:390-402`

**依赖**：

```gn
deps = [
  ":ets_loader_ark",
  ":ets_loader_ark_codegen",
  ":ets_loader_ark_components",
  ":ets_loader_ark_declaration",
  ":ets_loader_ark_form_components",
  ":ets_loader_ark_insight_intents",
  ":ets_loader_ark_lib",
  ":ets_loader_ark_server",
  ":ohos_declaration_ets_ark",
]
```

## arkui-plugins

**构建配置**：arkui-plugins/BUILD.gn

**Targets**：

| Target | 职责 |
|--------|------|
| `gen_ui_plugins` | 生成 UI 插件 |
| `build_ets_sysResource` | 生成系统资源 |
| `ui_plugin` | 复制 UI 插件 |
| `ohos_ets_ui_plugins` | 复制到 Ohos 路径 |

**构建脚本**：`build_ui_plugins.py`

## koala-wrapper

**构建配置**：koala-wrapper/BUILD.gn

**Targets**：

| Target | 类型 | 职责 |
|--------|------|------|
| `gen_sdk_ts_wrapper` | action | 生成 TS 包装器 |
| `ets2panda_koala_wrapper` | ohos_copy | 复制 Koala 包装器 |
| `ohos_ets_koala_wrapper` | ohos_copy | Ohos 路径 Koala 包装器 |

**依赖**：

```gn
deps = [ "./native:es2panda" ]
```

## install_arkguard_tsc_declgen

**类型**：action

**职责**：安装 ArkGuard TSC 声明生成工具。

**关键代码位置**：`BUILD.gn:407-424`

**依赖**：

```gn
external_deps = [
  "ets_frontend:build_arkguard_etc",
  "runtime_core:build_declgen_etc",
  "typescript:build_typescript_etc",
]
```

## 条件编译配置

### is_arkui_x 条件

```gn
if (defined(is_arkui_x) && is_arkui_x) {
  # ArkUI X 模式
  deps += [
    "//interface/sdk-js:bundle_arkts",
    "//interface/sdk-js:bundle_kits",
    "//interface/sdk-js:ohos_build_dynamic_sdk_component",
    "//interface/sdk-js:ets_internal_api",
    "//interface/sdk-js:ohos_declaration_ets",
  ]
} else {
  # 标准模式
  external_deps = [
    "sdk:bundle_arkts_etc",
    "sdk:bundle_kits_etc",
    "sdk:ets_component_etc",
    "sdk:ets_internal_api_etc",
    "sdk:ohos_declaration_ets_api",
  ]
}
```

### is_standard_system 条件

```gn
if (is_standard_system) {
  _ace_config_dir = "compiler"
} else {
  _ace_config_dir = "//prebuilts/ace-toolkit/ets-loader/compiler"
}
```

## 产物路径配置

### 动态路径获取

```gn
# 获取构建输出目录
ets_loader_lib_dir = get_label_info(":build_ets_loader_library", "target_out_dir") + "/lib"
ets_loader_declarations_dir = get_label_info(":build_ets_loader_library", "target_out_dir") + "/declarations"

# SDK 构建产物路径
ets_component_out_dir = get_label_info(ets_component_dep, "target_out_dir")
_arkts_apis_file_dir = ets_component_out_dir + "/bundle_arkts"
_kit_apis_file_dir = ets_component_out_dir + "/bundle_kits"
```

## bundle.json 配置

**文件位置**：`bundle.json`

**组件定义**：

```json
{
  "name": "@ohos/ace_ets2bundle",
  "version": "3.1",
  "component": {
    "name": "ace_ets2bundle",
    "subsystem": "developtools",
    "adapted_system_type": ["standard"],
    "build": {
      "sub_component": [
        "//developtools/ace_ets2bundle:ets_loader_component_config",
        "//developtools/ace_ets2bundle:ets_loader",
        "//developtools/ace_ets2bundle:ets_loader_library",
        "//developtools/ace_ets2bundle:components",
        "//developtools/ace_ets2bundle:server",
        "//developtools/ace_ets2bundle:codegen",
        "//developtools/ace_ets2bundle:ets_loader_declaration",
        "//developtools/ace_ets2bundle:ets_loader_ark_hap",
        "//developtools/ace_ets2bundle/koala-wrapper:ohos_ets_koala_wrapper"
      ],
      "inner_kits": [
        {"name": "//developtools/ace_ets2bundle/arkui-plugins:ohos_ets_ui_plugins"},
        {"name": "//developtools/ace_ets2bundle/koala-wrapper:ohos_ets_koala_wrapper"},
        {"name": "//developtools/ace_ets2bundle:ets_loader_ark_hap"}
      ]
    }
  }
}
```

## 相关文档

- [架构说明](02_Architecture.md)
- [目录结构](03_Directory_Structure.md)
- [编译产物](08_Build_Artifacts.md)
