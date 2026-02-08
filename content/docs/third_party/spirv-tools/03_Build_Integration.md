# OH 构建适配

本文档详细介绍 SPIRV-Tools 在 OpenHarmony 中的构建系统适配。

---

## 目录

- [构建系统概述](#构建系统概述)
- [BUILD.gn 结构说明](#buildgn-结构说明)
- [关键编译选项](#关键编译选项)
- [与上游构建系统的差异](#与上游构建系统的差异)
- [特殊处理](#特殊处理)
- [构建目标详解](#构建目标详解)
- [依赖关系](#依赖关系)
- [构建命令](#构建命令)

---

## 构建系统概述

### 使用 GN 构建系统

SPIRV-Tools 在 OpenHarmony 中使用 **GN（Generate Ninja）** 构建系统，而非上游默认的 CMake。

| 构建系统 | 用途 |
|---------|------|
| **CMake** | 上游默认构建系统 |
| **Bazel** | 上游支持的替代构建系统 |
| **GN** | OpenHarmony 采用的构建系统 |

### 构建配置位置

```
third_party/spirv-tools/
├── BUILD.gn                    # 主构建配置
├── source/
│   ├── opt/BUILD.gn           # 优化器模块
│   ├── val/BUILD.gn          # 验证器模块
│   ├── link/BUILD.gn          # 链接器模块
│   ├── reduce/BUILD.gn        # Reducer 模块
│   └── lint/BUILD.gn         # Linter 模块
└── test/
    └── fuzzers/BUILD.gn      # Fuzzer 测试
```

---

## BUILD.gn 结构说明

### 主配置文件结构

```gn
# Copyright (c) 2020 Google LLC
# Licensed under the Apache License, Version 2.0

import("//build/ohos.gni")
import("//third_party/vk-gl-cts/vk_gl_cts.gni")

# ==================== 配置部分 ====================
config("spv_headers_public_config") {
  include_dirs = [ "include" ]
}

config("deqp_spirvtool_config") {
  cflags_cc = deqp_common_cflags_cc
  cflags_cc += [ "-ftemplate-depth=1024", "-Wno-switch" ]
  defines = deqp_common_defines
  defines += [ "SPIRV_CHECK_CONTEXT", ... ]
}

# ==================== 模板部分 ====================
template("spvtools_core_tables") { ... }
template("spvtools_core_enums") { ... }
template("spvtools_glsl_tables") { ... }
template("spvtools_opencl_tables") { ... }

# ==================== 构建目标部分 ====================
ohos_source_set("spv_headers") { ... }
ohos_source_set("spvtools_headers") { ... }
ohos_source_set("deqp_spirvtool_source") { ... }

ohos_static_library("libdeqp_spirvtools") { ... }
ohos_static_library("libdeqp_spirvtools-opt") { ... }
ohos_static_library("libdeqp_spirvtools-val") { ... }
ohos_static_library("libdeqp_spirvtools-link") { ... }
ohos_static_library("libdeqp_spirvtools-reduce") { ... }
```

### 关键组成部分

| 部分 | 描述 |
|------|------|
| **import** | 引入 OHOS 基础配置和 vk-gl-cts 配置 |
| **config** | 定义编译配置（defines、flags、include_dirs） |
| **template** | 定义可复用的构建模板 |
| **ohos_source_set** | 源文件集合目标 |
| **ohos_static_library** | 静态库目标 |

---

## 关键编译选项

### 编译器标志

#### C/C++ 编译器标志

```gn
config("deqp_spirvtool_config") {
  # 基础标志
  cflags_cc = deqp_common_cflags_cc
  
  # 模板深度扩展
  cflags_cc += [ "-ftemplate-depth=1024" ]
  
  # 警告抑制
  cflags_cc += [ "-Wno-switch" ]
}
```

**标志说明**:

| 标志 | 说明 | 必要性 |
|------|------|--------|
| `-ftemplate-depth=1024` | 增加模板实例化深度限制 | 高：SPIRV-Tools 使用大量模板 |
| `-Wno-switch` | 抑制 switch 语句警告 | 中：代码风格相关 |
| `-std=c++17` | C++17 标准 | 高：代码使用 C++17 特性 |

#### 内部配置

```gn
config("spvtools_internal_config") {
  include_dirs = [ ".", "//third_party/spirv-headers/include" ]
  
  configs = [
    ":spv_headers_public_config",
    ":spvtools_include_gen_dirs",
  ]
  
  # 平台特定标志
  if (is_clang) {
    cflags += [
      "-Wno-implicit-fallthrough",
      "-Wno-newline-eof",
      "-Wno-unreachable-code-break",
      "-Wno-unreachable-code-return",
    ]
  } else if (!is_win) {
    cflags += [ "-Wno-format-truncation" ]
  }
}
```

### 编译宏定义

```gn
defines = [
  "SPIRV_CHECK_CONTEXT",     # 启用上下文检查
  "SPIRV_COLOR_TERMINAL",    # 启用彩色终端输出
  "SPIRV_LINUX",              # Linux 平台标识
  "SPIRV_TIMER_ENABLED",     # 启用计时器功能
  "SPIRV_TOOLS_SHAREDLIB",   # 构建共享库
  "SPIRV_Tools_shared_EXPORTS",  # 导出符号
]
```

**宏定义说明**:

| 宏 | 用途 |
|---|------|
| `SPIRV_CHECK_CONTEXT` | 启用 SPIR-V 上下文验证 |
| `SPIRV_COLOR_TERMINAL` | 启用命令行彩色输出 |
| `SPIRV_LINUX` | 指定目标平台为 Linux |
| `SPIRV_TIMER_ENABLED` | 启用性能计时功能 |
| `SPIRV_TOOLS_SHAREDLIB` | 构建为共享库（OH 使用静态库） |
| `SPIRV_Tools_shared_EXPORTS` | 导出符号定义 |

---

## 与上游构建系统的差异

### CMake vs GN

| 特性 | CMake (上游) | GN (OH) |
|------|--------------|---------|
| **构建系统** | CMakeLists.txt | BUILD.gn |
| **目标类型** | static_library, shared_library | ohos_static_library |
| **配置方式** | CMake 选项 | GN 变量和条件 |
| **依赖管理** | find_package | 直接路径引用 |
| **安装支持** | cmake --install | 无（OH 内置） |

### 主要差异对比

#### 1. 目标命名

```cmake
# CMake (上游)
add_library(SPIRV-Tools STATIC ...)
add_library(SPIRV-Tools-opt STATIC ...)

# GN (OH)
ohos_static_library("libdeqp_spirvtools") { ... }
ohos_static_library("libdeqp_spirvtools-opt") { ... }
```

#### 2. 编译配置

```cmake
# CMake (上游)
target_compile_definitions(SPIRV-Tools PRIVATE SPIRV_COLOR_TERMINAL)

# GN (OH)
config("deqp_spirvtool_config") {
  defines += [ "SPIRV_COLOR_TERMINAL" ]
}
```

#### 3. 包含路径

```cmake
# CMake (上游)
target_include_directories(SPIRV-Tools PRIVATE ${spirv-headers_INCLUDE_DIR})

# GN (OH)
ohos_source_set("deqp_spirvtool_source") {
  include_dirs = [
    "//third_party/spirv-headers/include",
    "//third_party/spirv-tools/include",
  ]
}
```

### OH 特有配置

#### 1. deqp 前缀命名

```gn
# 使用 deqp_ 前缀标识服务于 deqp 测试框架
ohos_source_set("deqp_spirvtool_source") { ... }
ohos_static_library("libdeqp_spirvtools") { ... }
```

#### 2. vk-gl-CTS 集成

```gn
# 生成文件路径指向 vk-gl-CTS 构建目录
core_insts_file = "//third_party/vk-gl-cts/build/external/spirv-tools/spirv-tools/core.insts-$version.inc"
operand_kinds_file = "//third_party/vk-gl-cts/build/external/spirv-tools/spirv-tools/operand.kinds-$version.inc"
```

---

## 特殊处理

### 1. 代码生成

SPIRV-Tools 需要从 JSON 语法文件生成 C++ 代码：

#### 核心表生成

```gn
template("spvtools_core_tables") {
  action("spvtools_core_tables_" + target_name) {
    script = "utils/generate_grammar_tables.py"
    
    sources = [
      core_json_file,
      debuginfo_insts_file,
      cldebuginfo100_insts_file,
    ]
    
    outputs = [
      core_insts_file,
      operand_kinds_file,
    ]
    
    args = [
      "--spirv-core-grammar",
      "--core-insts-output",
      "--extinst-debuginfo-grammar",
      "--operand-kinds-output",
    ]
  }
}
```

#### 生成文件

| 文件 | 用途 |
|------|------|
| `core.insts-<version>.inc` | 核心指令表 |
| `operand.kinds-<version>.inc` | 操作数类型表 |
| `glsl.std.450.insts.inc` | GLSL 标准指令 |
| `opencl.std.insts.inc` | OpenCL 标准指令 |
| `extension_enum.inc` | 扩展枚举 |
| `enum_string_mapping.inc` | 枚举字符串映射 |

### 2. Chromium 构建条件

```gn
if (build_with_chromium) {
  configs -= [ "//build/config/compiler:chromium_code" ]
  configs += [ "//build/config/compiler:no_chromium_code" ]
}
```

### 3. Fuzzer 构建

```gn
if (build_with_chromium && spvtools_build_executables) {
  proto_library("spvtools_fuzz_proto") { ... }
  
  ohos_static_library("spvtools_fuzz") {
    sources = [...fuzz源文件...]
    deps = [
      ":libdeqp_spirvtools",
      ":spvtools_fuzz_proto",
      "//third_party/spirv-tools/source/opt:libdeqp_spirvtools-opt",
    ]
    external_deps = [ "protobuf:protobuf_full" ]
  }
}
```

---

## 构建目标详解

### 核心库

#### libdeqp_spirvtools

```gn
ohos_static_library("libdeqp_spirvtools") {
  deps = [ ":deqp_spirvtool_source" ]
  part_name = "graphic_2d"
  subsystem_name = "graphic"
}
```

**功能**: SPIRV-Tools 核心库，包含汇编器、反汇编器

**源文件数量**: ~50 个核心文件

**依赖**: 无（自包含）

#### deqp_spirvtool_source

```gn
ohos_source_set("deqp_spirvtool_source") {
  sources = [
    "include/spirv-tools/libspirv.h",
    "include/spirv-tools/libspirv.hpp",
    "source/libspirv.cpp",
    "source/text.cpp",
    "source/binary.cpp",
    # ... 更多文件
  ]
  
  include_dirs = deqp_common_include_dirs
  include_dirs += [
    "//third_party/spirv-tools",
    "//third_party/vk-gl-cts/build/external/spirv-tools/spirv-tools",
    "//third_party/spirv-headers/include",
    "//third_party/spirv-tools/include",
  ]
}
```

### 扩展库

#### libdeqp_spirvtools-opt

```gn
ohos_static_library("libdeqp_spirvtools-opt") {
  deps = [
    ":libdeqp_spirvtools",
    "//third_party/spirv-tools/source/opt:spvtools_opt_source",
  ]
}
```

**功能**: 优化器库，包含 60+ 优化 pass

#### libdeqp_spirvtools-val

```gn
ohos_static_library("libdeqp_spirvtools-val") {
  deps = [
    ":libdeqp_spirvtools",
    "//third_party/spirv-tools/source/val:spvtools_val_source",
  ]
}
```

**功能**: 验证器库，包含完整的 SPIR-V 规范验证

#### libdeqp_spirvtools-link

```gn
ohos_static_library("libdeqp_spirvtools-link") {
  deps = [
    ":libdeqp_spirvtools",
    ":libdeqp_spirvtools-opt",
    ":libdeqp_spirvtools-val",
    "//third_party/spirv-tools/source/link:spvtools_link_source",
  ]
}
```

**功能**: 链接器库，合并多个 SPIR-V 模块

#### libdeqp_spirvtools-reduce

```gn
ohos_static_library("libdeqp_spirvtools-reduce") {
  deps = [
    ":libdeqp_spirvtools",
    ":libdeqp_spirvtools-opt",
    "//third_party/spirv-tools/source/reduce:spvtools_reduce_source",
  ]
}
```

**功能**: Reducer 库，简化测试用例

---

## 依赖关系

### 内部依赖

```
libdeqp_spirvtools (核心)
    ↓
libdeqp_spirvtools-opt (优化器)
    ↑
libdeqp_spirvtools-val (验证器) ← libdeqp_spirvtools
    ↑
libdeqp_spirvtools-link (链接器) ← libdeqp_spirvtools-val, opt
    ↑
libdeqp_spirvtools-reduce (Reducer) ← libdeqp_spirvtools-opt
```

### 外部依赖

| 依赖 | 用途 | 来源 |
|------|------|------|
| SPIRV-Headers | SPIR-V 头文件 | third_party/spirv-headers |
| vk-gl-CTS 构建产物 | 生成的代码表 | third_party/vk-gl-cts |

---

## 构建命令

### 标准构建

```bash
# 构建核心库
hb build //third_party/spirv-tools:libdeqp_spirvtools

# 构建优化器
hb build //third_party/spirv-tools/source/opt:libdeqp_spirvtools-opt

# 构建验证器
hb build //third_party/spirv-tools/source/val:libdeqp_spirvtools-val

# 构建全部
hb build //third_party/spirv-tools:all
```

### 构建所有模块

```bash
hb build //third_party/spirv-tools/...
```

### 运行测试

```bash
# 运行单元测试
hb test //third_party/spirv-tools/test/...
```

---

## 常见问题

### Q: 如何添加新的源文件？

**A**: 在对应的 `BUILD.gn` 文件中添加：

```gn
ohos_source_set("deqp_spirvtool_source") {
  sources = [
    "existing_file.cpp",
    "new_file.cpp",  # 添加新文件
  ]
}
```

### Q: 如何修改编译宏？

**A**: 修改 `config("deqp_spirvtool_config")` 中的 `defines`：

```gn
config("deqp_spirvtool_config") {
  defines += [ "NEW_MACRO" ]  # 添加新宏
  # 或
  defines -= [ "UNUSED_MACRO" ]  # 移除宏
}
```

### Q: 为什么使用 deqp_ 前缀？

**A**: 因为该库在 OH 中主要用于 **deqp**（Draw Quality Conformance Tests）测试框架。

---

## 参考信息

### 相关文档

- [Patch 分析](./02_Patches.md) - 无 Patch 适配说明
- [依赖关系与使用](./04_Usage_in_OH.md) - OH 使用场景
- [上游 CMakeLists.txt](../CMakeLists.txt) - CMake 配置参考

### 相关文件

- `BUILD.gn` - 主构建配置
- `source/opt/BUILD.gn` - 优化器构建配置
- `source/val/BUILD.gn` - 验证器构建配置
- `source/link/BUILD.gn` - 链接器构建配置
- `source/reduce/BUILD.gn` - Reducer 构建配置

---

## 版本历史

| 版本 | 日期 | 修改内容 |
|------|------|---------|
| 3.2 | 2026-02-07 | 初始版本，完整 BUILD.gn 适配 |

---

*最后更新: 2026-02-07*

*相关内容:*
- *上一章: [Patch 详细分析](./02_Patches.md)*
- *下一章: [依赖关系与使用](./04_Usage_in_OH.md)*
- *相关: [原始库简介](./01_Overview.md)*
