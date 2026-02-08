# HarfBuzz 构建适配说明

本文档详细说明 HarfBuzz 在 OpenHarmony 中的构建系统集成，包括 BUILD.gn 配置、编译选项、工具链适配以及 Patch 集成方式。

## 1 构建系统概述

### 1.1 构建架构

HarfBuzz 在 OpenHarmony 中采用以下构建架构：

```
┌─────────────────────────────────────────────────────────────────────┐
│                         构建流程                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────┐                                               │
│  │ harfbuzz-11.0.0 │  原始源码归档                                   │
│  │    .tar.xz     │                                               │
│  └────────┬────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌─────────────────┐                                               │
│  │   install.py   │  解压 + Patch 应用                             │
│  └────────┬────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    生成源文件                                 │  │
│  │  hb-aat-layout.cc, hb-buffer.cc, hb-common.cc, ...         │  │
│  │  (72 个源文件)                                               │  │
│  └────────────────────────┬────────────────────────────────────┘  │
│                           │                                         │
│                           ▼                                         │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                   静态库编译                                 │  │
│  │     libharfbuzz_static.a / libharfbuzz.so                  │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                           │                                         │
│                           ▼                                         │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    OH 模块使用                               │  │
│  │  Rosen, Skia, UI Lite                                       │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 构建文件结构

```
third_party/harfbuzz/
├── BUILD.gn                    # 主构建配置
├── install.py                  # 安装脚本
├── harfbuzz-11.0.0.tar.xz     # 源码归档
├── huawei_harfbuzz.patch      # OH Patch
├── bundle.json                # OH 组件配置
├── COPYING                    # 许可证
└── README.OpenSource          # 开源声明
```

---

## 2 BUILD.gn 详解

### 2.1 整体结构

```gn
# Copyright (c) 2020-2021 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0

# 条件导入：Lite vs 标准系统
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")
} else {
  import("//build/ohos.gni")
}

# 头文件配置
config("harfbuzz_config") {
  include_dirs = [ "${target_gen_dir}/harfbuzz-11.0.0/src" ]
}

# 预处理 action：从 tar.xz 解压并应用 patch
action("harfbuzz_action") {
  ...
}

# Lite 系统构建
if (defined(ohos_lite)) {
  lite_library("harfbuzz") {
    ...
  }
} else {
  # 标准系统构建
  ohos_static_library("harfbuzz_static") {
    ...
  }
}
```

### 2.2 配置片段详解

#### 2.2.1 头文件配置

```gn
config("harfbuzz_config") {
  # 关键：生成目录下的头文件路径
  include_dirs = [ "${target_gen_dir}/harfbuzz-11.0.0/src" ]
  
  # 注意：不添加额外的 public_configs
  # 依赖模块需要显式声明依赖
}
```

#### 2.2.2 预处理 action

```gn
action("harfbuzz_action") {
  # 1. 脚本路径
  script = "//third_party/harfbuzz/install.py"
  
  # 2. 输出文件列表（所有生成的源文件）
  outputs = [
    "${target_gen_dir}/harfbuzz-11.0.0/src/hb-aat-layout.cc",
    "${target_gen_dir}/harfbuzz-11.0.0/src/hb-buffer.cc",
    "${target_gen_dir}/harfbuzz-11.0.0/src/hb-common.cc",
    # ... 共 72 个源文件
  ]
  
  # 3. 输入依赖
  inputs = [ "//third_party/harfbuzz/harfbuzz-11.0.0.tar.xz" ]
  
  # 4. 传递构建目录路径
  harfbuzz_path = rebase_path("${target_gen_dir}", root_build_dir)
  harfbuzz_source_path = rebase_path("//third_party/harfbuzz", root_build_dir)
  
  # 5. 脚本参数
  args = [
    "--gen-dir",
    "$harfbuzz_path",
    "--source-dir",
    "$harfbuzz_source_path",
  ]
}
```

**输出文件完整列表**:

| 类别 | 文件数 | 说明 |
|-----|-------|-----|
| **核心模块** | 15 | hb-buffer, hb-font, hb-shape 等 |
| **OpenType** | 35 | hb-ot-layout, hb-ot-cmap, hb-ot-font 等 |
| **塑形器** | 12 | Arabic, Indic, Thai, Khmer 等脚本 |
| **子集化** | 4 | hb-subset-cff*, hb-face-builder |
| **后备** | 6 | hb-fallback-shape, hb-ucd, hb-unicode |

---

## 3 标准系统构建

### 3.1 静态库定义

```gn
ohos_static_library("harfbuzz_static") {
  # 使用 action 生成的源文件
  sources = get_target_outputs(":harfbuzz_action")
  
  # 依赖 action：确保解压和 patch 先完成
  deps = [ ":harfbuzz_action" ]
  
  # 头文件搜索路径
  include_dirs = [ "${target_gen_dir}/harfbuzz-11.0.0/src" ]
  
  # 编译器定义
  defines = [ "HAVE_PTHREAD = 1" ]
  
  # 公开配置
  public_configs = [ ":harfbuzz_config" ]
  
  # OH 构建元数据
  part_name = "harfbuzz"
  subsystem_name = "thirdparty"
}
```

### 3.2 配置说明

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `part_name` | "harfbuzz" | 组件名称 |
| `subsystem_name` | "thirdparty" | 子系统归属 |
| `HAVE_PTHREAD = 1` | 定义 | 启用 POSIX 线程支持 |
| `include_dirs` | 生成目录 | 指向 patch 后的源文件 |

### 3.3 头文件结构

构建后，生成的头文件结构如下：

```
${target_gen_dir}/harfbuzz-11.0.0/src/
├── harfbuzz/
│   ├── hb.h
│   ├── hb-blob.h
│   ├── hb-buffer.h
│   ├── hb-font.h
│   ├── hb-face.h
│   ├── hb-ft.h
│   ├── hb-ot.h
│   ├── hb-unicode.h
│   ├── hb-version.h
│   └── ...
└── (72 个 .cc 源文件)
```

---

## 4 Lite 系统构建

### 4.1 Lite 构建条件

```gn
if (defined(ohos_lite)) {
  # ohos_lite 定义时，使用 lite_component.gni
  import("//build/lite/config/component/lite_component.gni")
  
  # ...
  lite_library("harfbuzz") {
    # ...
  }
}
```

### 4.2 Lite 静态库

```gn
lite_library("harfbuzz") {
  # 输出库名称
  output_name = "harfbuzz"
  
  # 源文件来源
  sources = get_target_outputs(":harfbuzz_action")
  
  # 依赖 action
  deps = [ ":harfbuzz_action" ]
  
  # 公开配置
  public_configs = [ ":harfbuzz_config" ]
  
  # 根据工具链类型进行差异化配置
  if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
    # ICCARM 工具链配置
    target_type = "static_library"
    defines = [
      "HB_TINY",           # 精简模式
      "ENABLE_ICCARM",    # ICCARM 启用
      "HB_CUSTOM_MALLOC", # 自定义内存分配
    ]
    cflags = [
      "--diag_suppress",
      "Pe068,Pa093,Pe111,Pa181,Pe128,Pe161,Pe177,Pe185,Pe186,Pe550,Pe554",
    ]
    cflags_cc = cflags
    
  } else if (defined(board_toolchain_type) && board_toolchain_type == "clang") {
    # Clang 工具链配置
    target_type = "static_library"
    defines = [
      "HB_TINY",
      "HB_CUSTOM_MALLOC",
      "HB_NO_PRAGMA_GCC_DIAGNOSTIC_WARNING",
      "HB_NO_PRAGMA_GCC_DIAGNOSTIC_ERROR",
    ]
    cflags = [
      "-Wall",
      "-Wno-error=unused-variable",
      "-Wno-unused-variable",
      "-Wno-error=extra-semi-stmt",
      "-Wno-extra-semi-stmt",
    ]
    cflags_cc = cflags
    
  } else {
    # GCC 默认配置
    target_type = "shared_library"
    defines = [ "HAVE_PTHREAD = 1" ]
  }
}
```

---

## 5 工具链适配

### 5.1 GCC 工具链

```gn
# 标准 GCC 配置
target_type = "shared_library"
defines = [ "HAVE_PTHREAD = 1" ]

# 无额外警告抑制
# 使用系统默认优化级别
```

### 5.2 Clang 工具链

```gn
# Clang 特定配置
defines = [
  "HB_TINY",                                  # 精简模式
  "HB_CUSTOM_MALLOC",                         # 自定义内存
  "HB_NO_PRAGMA_GCC_DIAGNOSTIC_WARNING",     # 禁用特定警告
  "HB_NO_PRAGMA_GCC_DIAGNOSTIC_ERROR",       # 禁用特定错误
]

cflags = [
  "-Wall",                                    # 启用警告
  "-Wno-error=unused-variable",              # 未使用变量降级
  "-Wno-unused-variable",                    # 抑制未使用变量警告
  "-Wno-error=extra-semi-stmt",              # 多余分号降级
  "-Wno-extra-semi-stmt",                    # 抑制多余分号警告
]

cflags_cc = cflags  # C++ 同样配置
```

### 5.3 ICCARM 工具链

```gn
# ICCARM 特定配置
defines = [
  "HB_TINY",           # 精简模式（必需）
  "ENABLE_ICCARM",    # ICCARM 模式（必需）
  "HB_CUSTOM_MALLOC", # 自定义内存分配（必需）
]

cflags = [
  "--diag_suppress",   # 诊断抑制
  "Pe068,Pa093,Pe111,Pa181,Pe128,Pe161,Pe177,Pe185,Pe186,Pe550,Pe554",
]
```

**ICCARM 警告代码说明**:

| 代码 | 类别 | 抑制原因 |
|-----|------|---------|
| Pe068 | Warning | 隐式类型转换警告 |
| Pa093 | Warning | 参数未使用警告 |
| Pe111 | Warning | 声明后未使用警告 |
| Pa181 | Warning | 未引用的形参 |
| Pe128 | Error | 需要用户定义的运算符 |
| Pe161 | Warning | 未知的 pragma |
| Pe177 | Warning | 多余的常量表达式 |
| Pe185 | Warning | 空语句 |
| Pe186 | Warning | 类似的语句 |
| Pe550 | Warning | 已声明但未使用的标签 |
| Pe554 | Warning | 直接访问 'this' |

---

## 6 编译选项详解

### 6.1 定义宏

| 宏定义 | 值 | 用途 | 适用场景 |
|-------|-----|------|---------|
| `HAVE_PTHREAD` | 1 | 启用 POSIX 线程支持 | GCC 标准构建 |
| `HB_TINY` | 定义 | 启用精简模式，减小二进制体积 | 所有 Lite 构建 |
| `ENABLE_ICCARM` | 定义 | 启用 ICCARM 特定代码路径 | ICCARM 工具链 |
| `HB_CUSTOM_MALLOC` | 定义 | 使用自定义内存分配器 | ICCARM / 嵌入式 |
| `HB_NO_PRAGMA_GCC_DIAGNOSTIC_WARNING` | 定义 | 禁用 GCC pragma 警告 | Clang |
| `HB_NO_PRAGMA_GCC_DIAGNOSTIC_ERROR` | 定义 | 禁用 GCC pragma 错误 | Clang |

### 6.2 HB_TINY 模式

启用 `HB_TINY` 会导致以下变化：

| 功能 | 标准模式 | HB_TINY 模式 |
|-----|---------|------------|
| 缓存 | 启用 | 禁用 |
| 内存池 | 启用 | 精简 |
| Unicode 数据 | 完整 | 精简 |
| 脚本支持 | 全部 | 常用脚本 |

### 6.3 HB_CUSTOM_MALLOC 模式

当定义 `HB_CUSTOM_MALLOC` 时，HarfBuzz 使用自定义内存分配：

```cpp
// hb-common.cc 中的条件编译
#ifdef HB_CUSTOM_MALLOC
// 使用自定义分配器
#define hb_alloc(T, n)      custom_malloc((n) * sizeof(T))
#define hb_free(P)          custom_free(P)
#else
// 使用标准 malloc/free
#define hb_alloc(T, n)      ((T*)malloc((n) * sizeof(T)))
#define hb_free(P)          free(P)
#endif
```

---

## 7 Patch 集成

### 7.1 Patch 应用流程

```
install.py 脚本流程:

1. untar_file(tar_file_path, extract_path)
   ├─ 解压 harfbuzz-11.0.0.tar.xz
   └─ 到 ${gen_dir} 目录
   
2. move_file(src_path, dst_path)
   ├─ 复制 huawei_harfbuzz.patch
   └─ 到目标目录
   
3. do_patch(target_dir)
   ├─ 读取 patch 文件
   └─ 执行: patch -p1 --fuzz=0 -i huawei_harfbuzz.patch
```

### 7.2 Patch 命令参数

```bash
patch -p1 --fuzz=0 --no-backup-if-mismatch -i huawei_harfbuzz.patch -d target_dir
```

| 参数 | 说明 |
|-----|------|
| `-p1` | 忽略 patch 中的第一个路径组件 |
| `--fuzz=0` | 不允许模糊匹配，必须精确匹配上下文 |
| `--no-backup-if-mismatch` | 不匹配时不创建备份 |
| `-i` | 输入 patch 文件 |
| `-d` | 指定目标目录 |

### 7.3 Patch 应用验证

```bash
# 检查 patch 是否成功应用
cd ${gen_dir}/harfbuzz-11.0.0

# 检查修改的文件
git status --short

# 检查特定修改
git diff src/OT/Color/COLR/COLR.hh | head -50
```

---

## 8 构建产物

### 8.1 标准系统产物

```
${target_out_dir}/obj/third_party/harfbuzz/
└── libharfbuzz_static.a
    ├── 大小: ~2-5 MB (取决于配置)
    ├── 符号: hb_* public API
    └── 依赖: libc, libpthread
```

### 8.2 Lite 系统产物

```
${target_out_dir}/obj/third_party/harfbuzz/
├── libharfbuzz.a          # GCC 构建 (共享库为 .so)
│   ├── 大小: ~500KB-1MB
│   └── 符号: 精简 public API
│
└── libharfbuzz.a          # ICCARM 构建
    ├── 大小: ~200-500KB
    └── 符号: 进一步精简
```

### 8.3 头文件产物

```
${target_gen_dir}/harfbuzz-11.0.0/src/
├── harfbuzz/
│   ├── hb.h              (核心 API)
│   ├── hb-version.h      (版本)
│   ├── hb-buffer.h       (缓冲区)
│   ├── hb-font.h         (字体)
│   ├── hb-ft.h           (FreeType 集成)
│   ├── hb-ot.h           (OpenType)
│   └── ...
└── (72 个源文件编译产物)
```

---

## 9 依赖关系

### 9.1 系统依赖

| 依赖 | 用途 | 链接方式 |
|-----|------|---------|
| libc | 标准 C 库 | 自动链接 |
| libpthread | 线程支持 | HAVE_PTHREAD 时 |

### 9.2 OH 构建依赖

```gn
# harfbuzz 无外部构建依赖
deps = [ ":harfbuzz_action" ]  # 仅内部依赖

# 但它被以下模块依赖:
# - foundation/graphic/graphic_2d/rosen
# - foundation/arkui/ui_lite
# - third_party/skia
```

### 9.3 组件声明

```json
// bundle.json
{
  "component": {
    "name": "harfbuzz",
    "subsystem": "thirdparty",
    "adapted_system_type": ["small", "mini", "standard"],
    "build": {
      "sub_component": ["//third_party/harfbuzz:harfbuzz_static"],
      "inner_kits": [
        {
          "name": "//third_party/harfbuzz:harfbuzz_static"
        }
      ]
    }
  }
}
```

---

## 10 常见构建问题

### 10.1 ICCARM 编译失败

**问题**: ICCARM 工具链编译时报错

**解决方案**:

```gn
# 确保定义正确的宏
defines = [
  "HB_TINY",
  "ENABLE_ICCARM",
  "HB_CUSTOM_MALLOC",
]

# 确保使用正确的 cflags
cflags = [
  "--diag_suppress",
  "Pe068,Pa093,Pe111,Pa181,Pe128,Pe161,Pe177,Pe185,Pe186,Pe550,Pe554",
]
```

### 10.2 头文件找不到

**问题**: 编译时提示找不到头文件

**解决方案**:

```gn
# 确保依赖模块添加了正确的依赖
deps = [ "//third_party/harfbuzz:harfbuzz_static" ]

# 或显式添加配置
public_configs = [ "//third_party/harfbuzz:harfbuzz_config" ]
```

### 10.3 Patch 应用失败

**问题**: patch 命令失败

**解决方案**:

```bash
# 1. 检查 patch 文件是否存在
ls -la third_party/harfbuzz/huawei_harfbuzz.patch

# 2. 检查源文件是否已解压
ls -la ${gen_dir}/harfbuzz-11.0.0/src/hb.h

# 3. 手动应用 patch 查看错误
cd ${gen_dir}/harfbuzz-11.0.0
patch -p1 --dry-run -i /path/to/huawei_harfbuzz.patch
```

---

## 11 构建最佳实践

### 11.1 增量构建

```bash
# 只重新构建修改的模块
hb build //third_party/harfbuzz

# 清理后重新构建
hb build //third_party/harfbuzz --clean
```

### 11.2 发布构建

```gn
# 发布配置建议
defines = [
  "HAVE_PTHREAD = 1",
  # 不定义 HB_TINY 以获得完整功能
]
```

### 11.3 调试构建

```gn
# 调试配置
defines = [
  "HAVE_PTHREAD = 1",
  "HB_DEBUG=1",  # 如果启用
]
```

---

*本文档最后更新: 2024年*
