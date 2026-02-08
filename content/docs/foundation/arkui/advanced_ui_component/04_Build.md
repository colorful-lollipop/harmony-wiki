# 04_Build - GN 构建配置与产物

## 构建系统概述

**advanced_ui_component** 使用 OpenHarmony 标准 GN 构建系统，结合 es2abc 工具链将 ArkTS 编译为 ABC 字节码。

### 构建环境

| 环境 | 要求 |
|------|------|
| **构建系统** | GN + Ninja |
| **编译器** | Clang (C++17) |
| **目标平台** | OpenHarmony Standard |
| **构建工具** | hb (OpenHarmony Build) |

---

## GN 配置文件结构

### 根级 BUILD.gn

**文件**: `BUILD.gn:14-26`

```gn
group("advanced_ui_component") {
  deps = [
    "atomicservicenavigation/interfaces:atomicservicenavigation",
    "atomicservicesearch/interfaces:atomicservicesearch",
    "atomicservicetabs/interfaces:atomicservicetabs",
    "atomicserviceweb/interfaces:atomicserviceweb",
    "customappbar/atomicservicemenubar:atomicservicemenubar",
    "customappbar/interfaces:custom_app_bar",
    "halfscreenlaunchcomponent/interfaces:halfscreenlaunchcomponent",
    "innerfullscreenlaunchcomponent/interfaces:innerfullscreenlaunchcomponent",
    "interstitialdialogaction/interfaces:interstitialdialogaction",
  ]
}
```

**功能**: 聚合所有组件的接口模块。

---

## 组件构建配置

### 标准组件 BUILD.gn 模式

**证据来源**: `atomicservicenavigation/interfaces/BUILD.gn`

```gn
# 1. 导入配置
import("//build/config/components/ets_frontend/es2abc_config.gni")
import("//build/ohos.gni")
import("//foundation/arkui/advanced_ui_component/atomicservice_config.gni")

# 2. ArkTS -> ABC 字节码编译
es2abc_gen_abc("gen_componentname_abc") {
  src_js = "componentname.js"                    # ArkTS 入口文件
  dst_file = target_out_dir + "/componentname.abc"  # 输出路径
  in_puts = [ "componentname.js" ]              # 输入
  out_puts = [ target_out_dir + "/componentname.abc" ]  # 输出
  extra_args = [ "--module" ]                   # 模块模式
}

# 3. ABC -> C 源码 (用于预览/模拟器)
gen_obj("componentname_abc_preview") {
  input = get_label_info(":gen_componentname_abc", "target_out_dir") + "/componentname.abc"
  output = target_out_dir + "/componentname_abc.c"
  snapshot_dep = [ ":gen_componentname_abc" ]
}

# 4. ABC -> 对象文件 (用于真机)
gen_js_obj("componentname_abc") {
  input = get_label_info(":gen_componentname_abc", "target_out_dir") + "/componentname.abc"
  output = target_out_dir + "/componentname_abc.o"
  dep = ":gen_componentname_abc"
}

# 5. 构建共享库
ohos_shared_library("componentname") {
  # 源文件
  sources = [ "componentname.cpp" ]

  # 条件依赖
  if (use_mingw_win || use_mac || use_linux) {
    deps = [ ":gen_obj_src_componentname_abc_preview" ]  # 预览模式
  } else {
    deps = [ ":componentname_abc" ]                       # 真机模式
  }

  # 外部依赖
  external_deps = [
    "hilog:libhilog",
    "napi:ace_napi",
  ]

  # 安装配置
  relative_install_dir = "module/atomicservice"
  subsystem_name = "arkui"
  part_name = "advanced_ui_component"
}
```

---

## 构建配置详解

### 1. es2abc_gen_abc

将 ArkTS/JS 源码编译为 ABC 字节码。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| src_js | string | ArkTS 入口文件路径 |
| dst_file | string | 输出 ABC 文件路径 |
| in_puts | array | 输入文件列表 |
| out_puts | array | 输出文件列表 |
| extra_args | array | 额外编译参数 |

**证据来源**: `atomicservice_config.gni`

### 2. gen_js_obj

将 ABC 字节码转换为 C 数组对象文件。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| input | string | ABC 文件路径 |
| output | string | 输出 .o 文件路径 |
| dep | label | 依赖的 es2abc 目标 |

### 3. gen_obj

生成 C 源码文件 (预览模式)。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| input | string | ABC 文件路径 |
| output | string | 输出 .c 文件路径 |
| snapshot_dep | array | 快照依赖 |

### 4. ohos_shared_library

构建共享库 (.so)。

**关键配置**:
| 配置项 | 类型 | 说明 |
|--------|------|------|
| sources | array | C++ 源文件列表 |
| deps | array | 内部依赖 |
| external_deps | array | 外部系统依赖 |
| include_dirs | array | 头文件搜索路径 |
| defines | array | 预定义宏 |
| relative_install_dir | string | 安装子目录 |
| subsystem_name | string | 子系统名 |
| part_name | string | 部件名 |

---

## 各组件 Targets 清单

| 组件 | Target 路径 | 类型 | 输出 |
|------|-------------|------|------|
| AtomServiceNavigation | `//foundation/arkui/advanced_ui_component/atomicservicenavigation/interfaces:atomicservicenavigation` | ohos_shared_library | `libatomicservicenavigation.so` |
| AtomServiceSearch | `.../atomicservicesearch/interfaces:atomicservicesearch` | ohos_shared_library | `libatomicservicesearch.so` |
| AtomServiceTabs | `.../atomicservicetabs/interfaces:atomicservicetabs` | ohos_shared_library | `libatomicservicetabs.so` |
| AtomServiceWeb | `.../atomicserviceweb/interfaces:atomicserviceweb` | ohos_shared_library | `libatomicserviceweb.so` |
| AtomServiceMenuBar | `.../customappbar/atomicservicemenubar:atomicservicemenubar` | ohos_shared_library | `libatomicservicemenubar.so` |
| CustomAppBar | `.../customappbar/interfaces:custom_app_bar` | ohos_shared_library | `libcustom_app_bar.so` |
| HalfScreenLaunch | `.../halfscreenlaunchcomponent/interfaces:halfscreenlaunchcomponent` | ohos_shared_library | `liblevelscreenlaunchcomponent.so` |
| InnerFullScreen | `.../innerfullscreenlaunchcomponent/interfaces:innerfullscreenlaunchcomponent` | ohos_shared_library | `libinnerfullscreenlaunchcomponent.so` |
| InterstitialDialog | `.../interstitialdialogaction/interfaces:interstitialdialogaction` | ohos_shared_library | `libinterstitialdialogaction.so` |
| NavPushPathHelper | `.../navpushpathhelper:*` | ohos_shared_library | `libnavpushpathhelper.so` |

**证据来源**: `bundle.json:31-42`

---

## 外部依赖配置

### 通用依赖 (所有组件)

```gn
external_deps = [
  "hilog:libhilog",    # 日志库
  "napi:ace_napi",     # N-API 框架
]
```

### NavPushPathHelper 完整依赖

**证据来源**: `navpushpathhelper/BUILD.gn:59-69`

```gn
external_deps = [
  "ability_base:want",           # Want 能力
  "ability_runtime:abilitykit_native",  # Ability 原生接口
  "bundle_framework:appexecfwk_base",  # 应用框架基础
  "bundle_framework:appexecfwk_core",  # 应用框架核心
  "c_utils:utils",               # C 工具库
  "hilog:libhilog",              # 日志
  "ipc:ipc_core",                # IPC 核心
  "napi:ace_napi",               # N-API
  "samgr:samgr_proxy",           # Samgr 代理
]
```

### AtomServiceMenuBar 额外依赖

```gn
external_deps = [
  "hilog:libhilog",
  "napi:ace_napi",
  "ace_engine:ace_ndk",          # ACE NDK
  "ace_engine:ace_uicontent",    # UI 内容
]
```

---

## 编译产物

### 安装路径

| 产物 | 安装路径 |
|------|----------|
| .so 共享库 | `/system/lib64/module/atomicservice/` |
| 符号文件 | `out/.../lib64/module/atomicservice/` |

### 产物清单

```
out/{product}/lib64/module/atomicservice/
├── libatomicservicenavigation.so
├── libatomicservicesearch.so
├── libatomicservicetabs.so
├── libatomicserviceweb.so
├── libatomicservicemenubar.so
├── libcustom_app_bar.so
├── libhalfscreenlaunchcomponent.so
├── libinnerfullscreenlaunchcomponent.so
├── libinterstitialdialogaction.so
└── libnavpushpathhelper.so
```

### 构建产物验证

```bash
# 查看构建产物
ls -la out/{product}/lib64/module/atomicservice/

# 检查符号表
nm -D out/{product}/lib64/module/atomicservice/libatomicserviceweb.so | grep GetABCCode
```

---

## 条件编译

### 预览/模拟器模式

```gn
if (use_mingw_win || use_mac || use_linux) {
  deps = [ ":gen_obj_src_componentname_abc_preview" ]
} else {
  deps = [ ":componentname_abc" ]
}
```

**说明**:
- `use_mingw_win`: Windows (MinGW)
- `use_mac`: macOS
- `use_linux`: Linux
- 默认: 真机模式 (OpenHarmony)

### 配置开关

**证据来源**: `atomicservice_config.gni:49`

```gn
declare_args() {
  advanced_ui_component_feature_pc = false  # PC 特性开关
}
```

---

## 构建命令

### 完整构建

```bash
# 设置构建目标
hb set

# 构建所有组件
hb build -f
```

### 单独构建

```bash
# 构建单个组件
hb build //foundation/arkui/advanced_ui_component/atomicservicenavigation/interfaces:atomicservicenavigation

# 构建所有组件
hb build //foundation/arkui/advanced_ui_component:advanced_ui_component
```

### 清理构建

```bash
# 清理并重新构建
hb build -f -c

# 清理特定组件
hb build -c //foundation/arkui/advanced_ui_component/atomicserviceweb/interfaces:atomicserviceweb
```

---

## 常见构建问题

### 1. es2abc 编译失败

**现象**: `es2abc_gen_abc` 目标编译错误

**排查**:
1. 检查 ArkTS 语法
2. 确认入口文件路径
3. 验证 extra_args 参数

### 2. 符号未定义

**现象**: 链接时找不到 `NAPI_*_GetABCCode`

**排查**:
1. 确认 `gen_js_obj` 依赖正确
2. 检查 `deps` 配置
3. 验证 ABC 文件生成

### 3. 外部依赖缺失

**现象**: 找不到系统组件

**排查**:
1. 检查 `external_deps` 拼写
2. 确认子系统已构建
3. 验证依赖链

---

## 附录: BUILD.gn 模板

```gn
# Copyright (c) 2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/config/components/ets_frontend/es2abc_config.gni")
import("//build/ohos.gni")
import("//foundation/arkui/advanced_ui_component/atomicservice_config.gni")

es2abc_gen_abc("gen_componentname_abc") {
  src_js = "componentname.js"
  dst_file = target_out_dir + "/componentname.abc"
  in_puts = [ "componentname.js" ]
  out_puts = [ target_out_dir + "/componentname.abc" ]
  extra_args = [ "--module" ]
}

gen_js_obj("componentname_abc") {
  input = get_label_info(":gen_componentname_abc", "target_out_dir") +
          "/componentname.abc"
  output = target_out_dir + "/componentname_abc.o"
  dep = ":gen_componentname_abc"
}

ohos_shared_library("componentname") {
  sources = [ "componentname.cpp" ]
  deps = [ ":componentname_abc" ]
  external_deps = [
    "hilog:libhilog",
    "napi:ace_napi",
  ]
  relative_install_dir = "module/atomicservice"
  subsystem_name = "arkui"
  part_name = "advanced_ui_component"
}
```