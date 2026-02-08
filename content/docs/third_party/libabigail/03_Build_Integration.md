# 03 - OH 构建适配

## 3.1 构建系统概述

### 3.1.1 上游构建系统

libabigail 上游使用 **GNU Autotools** 构建系统：

```
上游构建流程
    │
    ├── configure.ac    # 配置定义
    ├── Makefile.am     # 构建规则
    ├── autogen.sh      # 生成配置脚本
    ├── configure       # 配置脚本（生成）
    └── Makefile        # 构建文件（生成）
```

**上游构建命令**:
```bash
./autogen.sh          # 生成配置脚本
./configure           # 配置（检测依赖、生成 Makefile）
make                  # 编译
make install          # 安装
```

### 3.1.2 OH 构建系统

OH 使用 **GN + Ninja** 构建系统：

```
OH 构建文件
    │
    ├── BUILD.gn        # 根目录配置
    ├── src/
    │   └── BUILD.gn    # 静态库构建
    └── tools/
        └── BUILD.gn    # 工具构建
```

**OH 构建命令**:
```bash
gn gen out --args='...'    # 生成 Ninja 文件
ninja -C out libabigail    # 编译
```

### 3.1.3 两种构建系统对比

| 特性 | 上游 (Autotools) | OH (GN) |
|------|------------------|---------|
| **配置方式** | 运行时检测 | 声明式配置 |
| **灵活性** | 高（多种选项） | 中（固定配置） |
| **构建速度** | 较慢 | 快（Ninja） |
| **增量构建** | 支持 | 优秀 |
| **跨平台** | 优秀 | 优秀 |
| **依赖管理** | pkg-config | GN deps |
| **输出类型** | 共享库 + 工具 | 静态库 + 工具 |

## 3.2 BUILD.gn 详细分析

### 3.2.1 根目录 BUILD.gn

**路径**: `third_party/libabigail/BUILD.gn`

**完整内容**:
```gni
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ...

import("//build/ohos.gni")

config("libabigail_defaults") {
  cflags_cc = [
    "-fexceptions",           # 启用 C++ 异常处理
    "-frtti",                 # 启用运行时类型信息
    "-Wno-unused-variable",   # 忽略未使用变量警告
    "-Wno-unused-value",
    "-Wno-overloaded-virtual",
    "-Wno-defaulted-function-deleted",
    "-Wno-parentheses",
    "-Wno-infinite-recursion",
  ]
}

group("libabigail-tools_host_toolchain") {
  deps = [
    "//third_party/libabigail/tools:abidiff($host_toolchain)",
    "//third_party/libabigail/tools:abidw($host_toolchain)",
  ]
}
```

**关键配置解析**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cflags_cc` | `-fexceptions` | 启用 C++ 异常，libabigail 使用异常处理错误 |
| `cflags_cc` | `-frtti` | 启用 RTTI，用于动态类型识别 |
| `cflags_cc` | `-Wno-*` | 禁用特定警告，上游代码存在这些警告 |
| `deps` | `$host_toolchain` | 强制使用 host 工具链，不编译为目标设备代码 |

**与上游对比**:

上游 `configure.ac` 中相关选项：
```bash
# 上游使用 AX_CXX_COMPILE_STDCXX 检测 C++ 标准
AX_CXX_COMPILE_STDCXX(14, noext, mandatory)

# 编译警告选项
AX_CHECK_COMPILE_FLAG([-Wall], [CXXFLAGS="$CXXFLAGS -Wall"])
```

OH 直接使用固定的编译器标志，无需运行时检测。

### 3.2.2 src/BUILD.gn

**路径**: `third_party/libabigail/src/BUILD.gn`

**完整内容**:
```gni
# Copyright (c) 2023 Huawei Device Co., Ltd.
# ...

import("//build/ohos.gni")

ohos_static_library("libabigail_static") {
  configs = [ "//third_party/libabigail:libabigail_defaults" ]

  sources = [
    "abg-comp-filter.cc",
    "abg-comparison.cc",
    "abg-config.cc",
    "abg-corpus.cc",
    "abg-default-reporter.cc",
    "abg-diff-utils.cc",
    "abg-dwarf-reader.cc",
    "abg-elf-based-reader.cc",
    "abg-elf-helpers.cc",
    "abg-elf-reader.cc",
    "abg-fe-iface.cc",
    "abg-hash.cc",
    "abg-ini.cc",
    "abg-ir.cc",
    "abg-leaf-reporter.cc",
    "abg-libxml-utils.cc",
    "abg-reader.cc",
    "abg-regex.cc",
    "abg-reporter-priv.cc",
    "abg-suppression.cc",
    "abg-symtab-reader.cc",
    "abg-tools-utils.cc",
    "abg-traverse.cc",
    "abg-writer.cc",
    "xxhash.c",
  ]

  include_dirs = [
    "//third_party/libabigail",
    "//third_party/libabigail/include",
    "//third_party/libabigail/src",
    "/usr/include/libxml2"
  ]

  external_deps = [
    "elfutils:libdw_static",
  ]
  ldflags = [ "-lxml2" ]

  defines = [ "ABIGAIL_ROOT_SYSTEM_LIBDIR=\"lib\"" ]

  license_file = "../LICENSE.txt"
  subsystem_name = "thirdparty"
  part_name = "libabigail"
}
```

**源文件分析**:

| 源文件 | 功能 | 重要性 |
|--------|------|--------|
| `abg-dwarf-reader.cc` | DWARF 调试信息读取 | ⭐⭐⭐ 核心 |
| `abg-elf-reader.cc` | ELF 文件读取 | ⭐⭐⭐ 核心 |
| `abg-comparison.cc` | ABI 比较逻辑 | ⭐⭐⭐ 核心 |
| `abg-writer.cc` | ABIXML 写入 | ⭐⭐⭐ 核心 |
| `abg-reader.cc` | ABIXML 读取 | ⭐⭐⭐ 核心 |
| `abg-ir.cc` | 中间表示 | ⭐⭐⭐ 核心 |
| `abg-corpus.cc` | ABI 语料库管理 | ⭐⭐⭐ 核心 |
| `abg-comp-filter.cc` | 兼容性过滤 | ⭐⭐⭐ 核心 |
| `abg-suppression.cc` | 抑制规则处理 | ⭐⭐ 重要 |
| `abg-symtab-reader.cc` | 符号表读取 | ⭐⭐ 重要 |
| `abg-libxml-utils.cc` | XML 工具 | ⭐⭐ 重要 |
| 其他 | 辅助功能 | ⭐ 一般 |

**依赖分析**:

```
libabigail_static
    │
    ├── external_deps
    │   └── elfutils:libdw_static    # DWARF 解析库
    │
    ├── ldflags
    │   └── -lxml2                   # XML 处理库
    │
    └── include_dirs
        ├── //third_party/libabigail         # 根目录
        ├── //third_party/libabigail/include # 头文件
        ├── //third_party/libabigail/src     # 源码
        └── /usr/include/libxml2             # 系统 libxml2
```

**宏定义**:

- `ABIGAIL_ROOT_SYSTEM_LIBDIR="lib"`: 指定系统库目录为 `lib`（而非 `lib64`）

**与上游对比**:

上游 `src/Makefile.am` 中相关配置：
```makefile
# 上游构建为共享库
lib_LTLIBRARIES = libabigail.la
libabigail_la_SOURCES = $(sources)
libabigail_la_LIBADD = $(LIBXML2_LIBS) $(ELFUTILS_LIBS)
libabigail_la_CPPFLAGS = $(LIBXML2_CFLAGS) $(ELFUTILS_CFLAGS)
```

OH 改为静态库，直接链接依赖。

### 3.2.3 tools/BUILD.gn

**路径**: `third_party/libabigail/tools/BUILD.gn`

**完整内容**:
```gni
# Copyright (c) 2023 Huawei Device Co., Ltd.
# ...

import("//build/ohos.gni")

ohos_executable("abidiff") {
  configs = [ "//third_party/libabigail:libabigail_defaults" ]
  sources = [ "abidiff.cc" ]

  include_dirs = [
    "//third_party/libabigail",
    "//third_party/libabigail/include",
  ]

  deps = [ "//third_party/libabigail/src:libabigail_static" ]
  external_deps = [ "elfutils:libdw_static" ]

  ldflags = [
    "-lxml2",
    "-llzma",
  ]
  install_enable = false
  subsystem_name = "thirdparty"
  part_name = "libabigail"
  license_file = "../LICENSE.txt"
}

ohos_executable("abidw") {
  configs = [ "//third_party/libabigail:libabigail_defaults" ]
  sources = [ "abidw.cc" ]

  include_dirs = [
    "//third_party/libabigail",
    "//third_party/libabigail/include",
  ]

  deps = [ "//third_party/libabigail/src:libabigail_static" ]
  external_deps = [ "elfutils:libdw_static" ]

  ldflags = [
    "-lxml2",
    "-llzma",
  ]
  install_enable = false
  subsystem_name = "thirdparty"
  part_name = "libabigail"
  license_file = "../LICENSE.txt"
}
```

**工具说明**:

| 工具 | 源文件 | 功能 | OH 用途 |
|------|--------|------|---------|
| `abidiff` | `abidiff.cc` | 比较两个 ABI 的差异 | ✅ 核心工具 |
| `abidw` | `abidw.cc` | 提取 ABI 到 XML | ✅ 核心工具 |

**关键配置**:

- `install_enable = false`: 不安装到系统目录（仅作为构建工具使用）
- `ldflags`: 链接 libxml2 和 lzma（用于 XZ 压缩）

**与上游对比**:

上游 `tools/Makefile.am`:
```makefile
bin_PROGRAMS = abidiff abidw abilint abicompat abipkgdiff ...
abidiff_SOURCES = abidiff.cc
abidiff_LDADD = $(top_builddir)/src/libabigail.la
```

OH 仅构建两个核心工具，且为静态链接。

## 3.3 关键编译选项详解

### 3.3.1 编译器标志

| 标志 | 说明 | 来源 |
|------|------|------|
| `-fexceptions` | 启用 C++ 异常处理 | libabigail 使用异常处理错误条件 |
| `-frtti` | 启用运行时类型信息 | 需要 dynamic_cast/typeid |
| `-Wno-unused-variable` | 忽略未使用变量警告 | 上游代码存在此类警告 |
| `-Wno-unused-value` | 忽略未使用值警告 | 上游代码存在此类警告 |
| `-Wno-overloaded-virtual` | 忽略虚函数重载警告 | 上游代码存在此类警告 |
| `-Wno-defaulted-function-deleted` | 忽略默认函数删除警告 | 上游代码存在此类警告 |
| `-Wno-parentheses` | 忽略括号警告 | 上游代码存在此类警告 |
| `-Wno-infinite-recursion` | 忽略无限递归警告 | 上游代码存在此类警告 |

### 3.3.2 预处理器宏

| 宏 | 定义 | 作用 |
|----|------|------|
| `ABIGAIL_ROOT_SYSTEM_LIBDIR` | `"lib"` | 指定系统库目录路径 |

### 3.3.3 链接选项

| 标志 | 说明 |
|------|------|
| `-lxml2` | 链接 libxml2 库（XML 解析） |
| `-llzma` | 链接 lzma 库（XZ 压缩支持） |

## 3.4 与上游 configure.ac 的对比

### 3.4.1 上游可选功能

上游 `configure.ac` 支持的功能（OH 未启用）：

```bash
# 可选工具
--enable-rpm              # RPM 包支持（未启用）
--enable-fedabipkgdiff    # Fedora 包差异（未启用）
--enable-abidb            # ABIDB 数据库（未启用）
--enable-manual           # 手册页（未启用）
--enable-apidoc           # API 文档（未启用）

# 可选特性
--enable-inlined-xxhash   # 内联 xxhash（未启用）
--with-libxml2            # 指定 libxml2 路径（使用系统默认）
--enable-dependency-tracking  # 依赖跟踪（未启用）
```

### 3.4.2 OH 简化配置

OH 采用固定配置：

| 功能 | 上游 | OH |
|------|------|-----|
| abidiff | ✅ 启用 | ✅ 启用 |
| abidw | ✅ 启用 | ✅ 启用 |
| abilint | ✅ 启用 | ❌ 禁用 |
| abicompat | ✅ 启用 | ❌ 禁用 |
| abipkgdiff | ✅ 启用 | ❌ 禁用 |
| fedabipkgdiff | ✅ 启用 | ❌ 禁用 |
| abidb | 可选 | ❌ 禁用 |
| 手册页 | ✅ 启用 | ❌ 禁用 |
| API 文档 | 可选 | ❌ 禁用 |

### 3.4.3 简化原因

1. **功能聚焦**: OH 仅需 ABI 检查和提取功能
2. **减少依赖**: 禁用 RPM、Fedora 相关功能，减少依赖
3. **加快构建**: 减少编译目标，加快构建速度
4. **Host 工具**: 不随镜像发布，无需完整功能

## 3.5 依赖管理

### 3.5.1 依赖关系图

```
libabigail
    │
    ├── 直接依赖
    │   ├── elfutils:libdw_static    # GN 外部依赖
    │   ├── libxml2                  # 系统库
    │   └── lzma                     # 系统库
    │
    └── 间接依赖（通过 elfutils）
        ├── libelf
        └── 其他 elfutils 组件
```

### 3.5.2 bundle.json 依赖声明

```json
{
    "deps": {
        "components": [
            "elfutils"
        ],
        "third_party": []
    }
}
```

### 3.5.3 依赖版本要求

| 依赖 | 最低版本 | 说明 |
|------|----------|------|
| elfutils | 0.178 | 从 NEWS 文件推断 |
| libxml2 | 2.9 | 系统默认通常满足 |
| lzma | 5.0 | 系统默认通常满足 |

## 3.6 构建输出

### 3.6.1 构建产物

```
out/clang_x64/thirdparty/libabigail/
├── src/
│   └── libabigail_static.a          # 静态库
└── tools/
    ├── abidiff                      # 可执行工具
    └── abidw                        # 可执行工具
```

### 3.6.2 使用方式

在 OH 构建系统中，通过以下方式引用：

```gni
# 在 check_abi_and_copy_deps 模板中
deps += [ "//third_party/libabigail/tools:abidiff($host_toolchain)" ]
deps += [ "//third_party/libabigail/tools:abidw($host_toolchain)" ]
```

## 3.7 升级指南

### 3.7.1 升级前检查

升级 libabigail 上游版本时，检查以下 BUILD.gn 相关内容：

```markdown
1. 源文件列表 (src/BUILD.gn)
   - [ ] 检查是否有新增源文件
   - [ ] 检查是否有删除的源文件
   - [ ] 检查源文件重命名

2. 头文件路径 (src/BUILD.gn)
   - [ ] 检查 include_dirs 是否需要调整

3. 依赖变化 (所有 BUILD.gn)
   - [ ] 检查 elfutils 依赖是否变化
   - [ ] 检查 libxml2 依赖是否变化
   - [ ] 检查新增的系统依赖

4. 编译选项 (根目录 BUILD.gn)
   - [ ] 检查是否需要调整警告抑制选项
   - [ ] 检查是否需要新增编译标志

5. 工具变化 (tools/BUILD.gn)
   - [ ] 检查是否需要新增工具
   - [ ] 检查工具源文件变化
```

### 3.7.2 升级步骤

```bash
# 1. 备份当前 BUILD.gn 文件
cp BUILD.gn BUILD.gn.bak
cp src/BUILD.gn src/BUILD.gn.bak
cp tools/BUILD.gn tools/BUILD.gn.bak

# 2. 替换上游源码
git checkout upstream/master -- .

# 3. 恢复 BUILD.gn 文件
mv BUILD.gn.bak BUILD.gn
mv src/BUILD.gn.bak src/BUILD.gn
mv tools/BUILD.gn.bak tools/BUILD.gn

# 4. 验证编译
gn gen out
ninja -C out libabigail-tools_host_toolchain

# 5. 功能测试
./out/clang_x64/thirdparty/libabigail/tools/abidw --help
./out/clang_x64/thirdparty/libabigail/tools/abidiff --help
```

---

**上一章**: [02_Patches.md](02_Patches.md)  
**下一章**: [04_Usage_in_OH.md](04_Usage_in_OH.md)
