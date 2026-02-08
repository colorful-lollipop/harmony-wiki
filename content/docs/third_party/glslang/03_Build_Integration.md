# OH 构建适配

## 1. 构建系统概述

### 1.1 双重构建系统

OpenHarmony 中的 glslang 使用**双重 BUILD.gn** 结构：

```
third_party/glslang/
├── BUILD.gn              # Chromium/Fuchsia 风格（根目录）
├── glslang/
│   └── BUILD.gn          # OH 风格（子目录）
├── SPIRV/
│   └── BUILD.gn          # OH 风格（子目录）
└── CMakeLists.txt        # 上游原始构建文件（未使用）
```

### 1.2 构建目标映射

| 上游目标 (CMake) | OH 目标 (GN) | 类型 |
|------------------|--------------|------|
| `glslang` | `:glslang` | 静态库 |
| `glslangValidator` | `:glslang_validator` | 可执行文件 |
| `spirv-remap` | `:spirv-remap` | 可执行文件 |
| `GenericCodeGen` | `glslang:libdeqp_GenericCodeGen` | 静态库 |
| `MachineIndependent` | `glslang:libdeqp_MachineIndependent` | 静态库 |
| `OSDependent` | `glslang:libdeqp_OSDependent` | 静态库 |
| `SPIRV` | `SPIRV:SPIRV_source` / `SPIRV:libdeqp_spirv` | 源集/静态库 |
| `SPVRemapper` | `SPIRV:libdeqp_spvremapper` | 静态库 |

---

## 2. 根目录 BUILD.gn 分析

### 2.1 文件头与导入

```gn
# Copyright (C) 2018 Google, Inc.
#
# All rights reserved.
# ... (BSD-3 许可证)

import("//build/ohos.gni")
import("build_overrides/glslang.gni")
```

**说明**:
- 使用 Google 版权头（继承自 Chromium 项目）
- 导入 OH 构建系统定义和本地覆盖配置

### 2.2 警告配置处理

```gn
# Both Chromium and Fuchsia use by default a set of warning errors
# that is far too strict to compile this project.
if (defined(is_fuchsia_tree) && is_fuchsia_tree) {
  _configs_to_remove = [ "//build/config:default_warnings" ]
  _configs_to_add = []
} else {
  _configs_to_remove = [ "//build/config/compiler:chromium_code" ]
  _configs_to_add = [ "//build/config/compiler:no_chromium_code" ]
}
```

**说明**:
- 移除 Chromium/Fuchsia 的严格警告配置
- 使用 `no_chromium_code` 配置以兼容第三方代码

### 2.3 Action 目标：生成文件

#### glslang_build_info

```gn
action("glslang_build_info") {
  script = "build_info.py"
  
  src_dir = "."
  changes_file = "CHANGES.md"
  template_file = "build_info.h.tmpl"
  out_file = "${target_gen_dir}/include/glslang/build_info.h"
  
  inputs = [
    changes_file,
    script,
    template_file,
  ]
  outputs = [ out_file ]
  args = [
    rebase_path(src_dir, root_build_dir),
    "-i",
    rebase_path(template_file, root_build_dir),
    "-o",
    rebase_path(out_file, root_build_dir),
  ]
}
```

**功能**: 从 `CHANGES.md` 提取版本信息，生成 `build_info.h` 头文件

#### glslang_extension_headers

```gn
action("glslang_extension_headers") {
  script = "gen_extension_headers.py"
  
  out_file = "${target_gen_dir}/include/glslang/glsl_intrinsic_header.h"
  
  sources = [ "glslang/ExtensionHeaders/GL_EXT_shader_realtime_clock.glsl" ]
  inputs = [ script ]
  outputs = [ out_file ]
  args = [
    "-i",
    rebase_path("glslang/ExtensionHeaders", root_build_dir),
    "-o",
    rebase_path(out_file, root_build_dir),
  ]
}
```

**功能**: 生成 GLSL 扩展内联头文件

### 2.4 公共配置

```gn
config("glslang_public") {
  include_dirs = [ "." ]
  if (!is_win || is_clang) {
    cflags = [ "-Wno-conversion" ]
  }
}

config("glslang_hlsl") {
  defines = [ "ENABLE_HLSL=1" ]
}
```

### 2.5 源文件模板

```gn
template("glslang_sources_common") {
  source_set(target_name) {
    public_configs = [ ":glslang_public" ]
    
    if (invoker.enable_hlsl) {
      public_configs += [ ":glslang_hlsl" ]
    }
    
    sources = [
      # ... 大量源文件列表 ...
    ]
    
    if (is_win) {
      sources += [ "glslang/OSDependent/Windows/ossource.cpp" ]
      defines += [ "GLSLANG_OSINCLUDE_WIN32" ]
    } else {
      sources += [ "glslang/OSDependent/Unix/ossource.cpp" ]
      defines += [ "GLSLANG_OSINCLUDE_UNIX" ]
    }
    
    # ... 编译器选项 ...
    
    if (build_ohos_sdk) {
      defines += [ "OH_SDK" ]
      cflags += [ "-std=c++17" ]
    } else {
      configs -= _configs_to_remove
      configs += _configs_to_add
    }
  }
}
```

**关键配置**:
- `enable_hlsl` - 控制 HLSL 支持
- `enable_opt` - 控制 SPIRV-Tools 优化（OH 中禁用）
- `OH_SDK` 宏定义（仅当 `build_ohos_sdk` 时）

### 2.6 可执行文件目标

#### glslang_validator

```gn
ohos_executable("glslang_validator") {
  sources = [
    "StandAlone/DirStackFileIncluder.h",
    "StandAlone/StandAlone.cpp",
  ]
  
  if (!is_win) {
    cflags = [
      "-Woverflow",
      "-std=c++17",
    ]
  } else {
    cflags = [ "-std=c++17" ]
  }
  
  defines = [
    "ENABLE_OPT=1",    # ← 启用优化接口
    "OH_SDK",          # ← OH 特定宏
  ]
  
  deps = [
    ":glslang_build_info",
    ":glslang_default_resource_limits_sources",
    ":glslang_extension_headers",
    ":glslang_sources",
  ]
  
  public_configs = [ ":glslang_hlsl" ]
  
  include_dirs = [
    "${target_gen_dir}/include",
    "${spirv_tools_dir}/include",
  ]
  
  install_enable = true
  part_name = "graphic_2d"
  subsystem_name = "graphic"
}
```

**注意**: 虽然定义了 `ENABLE_OPT=1`，但由于 `glslang_sources` 中 `enable_opt = false`，实际优化器并未链接。

#### spirv-remap

```gn
ohos_executable("spirv-remap") {
  sources = [ "StandAlone/spirv-remap.cpp" ]
  defines = [
    "ENABLE_OPT=1",
    "OH_SDK",
  ]
  deps = [ ":glslang_sources" ]
  cflags = [ "-std=c++17" ]
  include_dirs = [ "${spirv_tools_dir}/include" ]
  install_enable = true
  part_name = "graphic_2d"
  subsystem_name = "graphic"
}
```

### 2.7 主静态库

```gn
ohos_static_library("glslang") {
  deps = [
    ":glslang_default_resource_limits_sources",
    "//third_party/glslang/SPIRV:SPIRV_source",
    "//third_party/glslang/glslang:libdeqp_glslang",
  ]
  
  if (build_xts) {
    deps += [
      "//third_party/glslang/SPIRV:libdeqp_spirv",
      "//third_party/glslang/SPIRV:libdeqp_spvremapper",
    ]
  }
  
  public_configs = [ ":glslang_public" ]
  part_name = "glslang"
  subsystem_name = "thirdparty"
}
```

---

## 3. glslang/BUILD.gn 分析（OH 风格）

### 3.1 配置定义

```gn
config("src_glslang_config") {
  cflags_cc = [
    "-fPIC",
    "-std=c++17",
    "-Wno-reorder",
    "-fno-rtti",
    "-fno-exceptions",
    "-Wno-sign-compare",
    "-Wno-missing-field-initializers",
    "-Wno-unused-parameter",
    "-Wno-unused-variable",
  ]
  
  if (is_mingw) {
    cflags_cc -= [ "-fPIC" ]
  }
  
  defines = [
    "ENABLE_HLSL",
    "ENABLE_OPT=0",
    "GLSLANG_OSINCLUDE_UNIX",
  ]
}
```

**关键选项**:
- `-fno-rtti` / `-fno-exceptions` - 嵌入式优化
- `-fPIC` - 位置无关代码（MinGW 除外）
- `ENABLE_OPT=0` - 禁用 SPIRV-Tools 优化

### 3.2 模块化构建

OH 风格 BUILD.gn 将 glslang 拆分为多个子模块：

```
OSDependent_source → libdeqp_OSDependent
GenericCodeGen_source → libdeqp_GenericCodeGen
MachineIndependent_source → libdeqp_MachineIndependent
glslang_source → libdeqp_glslang
```

**优势**:
- 细粒度依赖管理
- 便于 CTS (deqp) 按需链接
- 避免链接未使用的代码

### 3.3 源集示例

```gn
ohos_source_set("MachineIndependent_source") {
  sources = [
    "//third_party/glslang/glslang/HLSL/hlslAttributes.cpp",
    "//third_party/glslang/glslang/HLSL/hlslGrammar.cpp",
    # ... 更多源文件 ...
  ]
  
  include_dirs = [
    "//third_party/glslang/glslang/Include",
    "//third_party/glslang/glslang/Public",
    "//third_party/glslang",
  ]
  
  deps = [
    ":libdeqp_GenericCodeGen",
    ":libdeqp_OSDependent",
  ]
  
  configs = [ ":src_glslang_config" ]
  part_name = "glslang"
  subsystem_name = "thirdparty"
}
```

---

## 4. SPIRV/BUILD.gn 分析

### 4.1 配置

```gn
config("SPIRV_config") {
  cflags_cc = [
    "-fPIC",
    "-std=c++17",
    "-Wno-reorder",
    "-fno-rtti",
    "-fno-exceptions",
    "-Wno-sign-compare",
    "-Wno-unused-parameter",
  ]
  
  defines = [
    "ENABLE_HLSL",
    "ENABLE_OPT=0",
    "GLSLANG_OSINCLUDE_UNIX",
  ]
}
```

### 4.2 目标

```gn
ohos_source_set("SPIRV_source") {
  sources = [
    "//third_party/glslang/SPIRV/CInterface/spirv_c_interface.cpp",
    "//third_party/glslang/SPIRV/GlslangToSpv.cpp",
    # ... 更多 ...
  ]
  
  deps = [
    "//third_party/glslang/glslang:libdeqp_GenericCodeGen",
    "//third_party/glslang/glslang:libdeqp_MachineIndependent",
    "//third_party/glslang/glslang:libdeqp_OSDependent",
  ]
  ...
}

ohos_shared_library("libdeqp_spirv") { ... }
ohos_shared_library("libdeqp_spvremapper") { ... }
```

---

## 5. 构建配置对比

### 5.1 上游 vs OH

| 配置项 | 上游 CMake | OH GN | 说明 |
|--------|-----------|-------|------|
| **ENABLE_HLSL** | `ON` (可选) | 强制定义 | HLSL 支持 |
| **ENABLE_OPT** | `ON` (可选) | `0` | 禁用优化器 |
| **GLSLANG_OSINCLUDE_UNIX** | 自动检测 | 强制定义 | Unix OS 层 |
| **-std=c++17** | 必需 | 强制 | C++ 标准 |
| **-fno-rtti** | 通常无 | 有 | 禁用 RTTI |
| **-fno-exceptions** | 通常无 | 有 | 禁用异常 |
| **-fPIC** | 通常无 | 有 (非 MinGW) | 位置无关代码 |

### 5.2 产物差异

| 产物 | 上游 | OH |
|------|------|-----|
| glslang 库 | 动态/静态可选 | 静态库 |
| 可执行文件 | glslangValidator | glslang_validator |
| 命名 | 标准 | `libdeqp_*` 前缀 |
| 安装 | 标准路径 | part/subsystem |

---

## 6. 使用指南

### 6.1 依赖 glslang

在 GN 文件中添加依赖：

```gn
# 使用根目录目标
 deps = [ "//third_party/glslang:glslang" ]

# 使用子目录目标（CTS 风格）
deps = [
  "//third_party/glslang/glslang:libdeqp_glslang",
  "//third_party/glslang/SPIRV:libdeqp_spirv",
]
```

### 6.2 头文件引用

```cpp
// C++ 接口
#include <glslang/Public/ShaderLang.h>
#include <glslang/Include/glslang_c_interface.h>

// C 接口
#include <glslang/Public/resource_limits_c.h>
```

### 6.3 编译选项继承

依赖 glslang 的目标会自动继承以下配置：
- `glslang_public` config - 包含路径
- `glslang_hlsl` config - `ENABLE_HLSL=1` 定义

---

## 7. 常见问题

### Q1: 为什么有两套 BUILD.gn？

A: 根目录 BUILD.gn 继承自 Chromium/Fuchsia，用于兼容性；子目录 BUILD.gn 是 OH 专用，提供更细粒度的控制。

### Q2: 为什么禁用 SPIRV-Tools 优化？

A: 减少依赖链大小。CTS 测试通常不需要优化后的 SPIR-V，且可以单独使用 spirv-opt 工具进行优化。

### Q3: `libdeqp_` 前缀的含义？

A: 表示 "dEQP"（drawElements Quality Program），这是 CTS 测试框架的前身。前缀用于兼容性。

### Q4: 如何启用 SPIRV-Tools 优化？

A: 需要：
1. 将 `enable_opt = false` 改为 `true`
2. 确保 `spirv_tools_dir` 指向有效的 spirv-tools 源码
3. 添加 spirv-tools 依赖

---

## 8. 构建命令示例

```bash
# 构建 glslang 库
hb build //third_party/glslang:glslang

# 构建所有目标
hb build //third_party/glslang/...

# 构建验证器工具
hb build //third_party/glslang:glslang_validator

# 构建 spirv-remap 工具
hb build //third_party/glslang:spirv-remap
```
