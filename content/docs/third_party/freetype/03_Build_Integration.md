# OpenHarmony 构建适配

本文档详细说明 FreeType 在 OpenHarmony 构建系统（GN）中的适配方案。

---

## 1. 构建系统概览

### 1.1 构建架构

```
FreeType 构建流程
│
├── 源码包管理
│   └── freetype-2.13.3.tar.xz (上游源码)
│
├── Patch 应用
│   └── install.py (自动化补丁脚本)
│
├── 源文件生成
│   └── freetype_action (生成器动作)
│
├── 库构建
│   ├── ohos_static_library (标准系统)
│   └── lite_library (轻量级系统)
│
└── 头文件配置
    └── ftconfig.h (多架构配置)
```

### 1.2 构建目标

| 构建目标 | 类型 | 适用系统 | 主要差异 |
|----------|------|----------|----------|
| freetype_static | static_library | 标准 OHOS | 支持完整功能 |
| freetype | lite_library | LiteOS-M | 精简配置 |
| freetype_shared | shared_library | LiteOS (可选) | 动态链接 |

---

## 2. 构建脚本详解

### 2.1 install.py

`install.py` 是 OH 定制的构建脚本，负责：

1. **源码解压**: 解压 `freetype-2.13.3.tar.xz`
2. **补丁应用**: 按顺序应用 7 个 patches
3. **头文件复制**: 复制 `ftconfig.h` 和配置
4. **模块组织**: 准备源文件列表

#### 核心代码流程

```python
def main():
    # 1. 解压源码
    untar_file(tar_file_path, target_dir)
    
    # 2. 复制配置和补丁
    move_file(source_dir, target_dir)
    
    # 3. 复制头文件
    move_include(source_dir, include_dir)
    
    # 4. 应用补丁
    do_patch(target_dir)
```

#### Patch 应用顺序

| 顺序 | Patch 文件 | 用途 |
|------|-----------|------|
| 1 | backport-freetype-2.2.1-enable-valid.patch | 验证模块启用 |
| 2 | backport-freetype-2.3.0-enable-spr.patch | 子像素渲染 |
| 3 | backport-freetype-2.6.5-libtool.patch | libtool 修复 |
| 4 | backport-freetype-2.8-multilib.patch | 多库支持 |
| 5 | backport-freetype-2.10.0-internal-outline.patch | ABI 兼容 |
| 6 | backport-freetype-2.10.1-debughook.patch | API 修复 |
| 7 | backport-freetype-2.12.1-enable-funcs.patch | 函数导出 |

---

## 3. BUILD.gn 配置详解

### 3.1 基础配置 (freetype_config)

```gn
config("freetype_config") {
  defines = [ "FT2_BUILD_LIBRARY" ]
  include_dirs = [ "${target_gen_dir}/freetype/include" ]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `FT2_BUILD_LIBRARY` | 必需 | 标记库构建模式 |
| `include_dirs` | 生成目录 | 指向补丁后的头文件目录 |

### 3.2 标准系统构建 (freetype_static)

```gn
ohos_static_library("freetype_static") {
  visibility = [
    ":*",  # 默认对所有模块可见
    "//foundation/arkui/ui_lite:*",
    "//foundation/graphic/graphic_2d/rosen:*",
    "//third_party/skia/m133:*",
    # ... 其他可见模块
  ]
  
  sources = get_target_outputs(":freetype_action")
  include_dirs = [ "${target_gen_dir}/freetype/src/base" ]
  public_configs = [ ":freetype_config" ]
  
  deps = [ ":freetype_action" ]
  external_deps = [ "zlib:libz" ]
  defines = [ "FT_CONFIG_OPTION_SYSTEM_ZLIB" ]
  
  # 条件配置
  if (current_os == "ohos") {
    defines += [ "FT_CONFIG_OPTION_USE_PNG" ]
    external_deps += [ "libpng:libpng" ]
  } else if (is_arkui_x) {
    defines += [ "FT_CONFIG_OPTION_USE_PNG" ]
    deps += [ "//third_party/skia/m133/third_party/libpng" ]
  } else if (product_name == "ohos-sdk") {
    defines += [ "FT_CONFIG_OPTION_USE_PNG" ]
    external_deps += [ "libpng:libpng_static" ]
  }
  
  part_name = "freetype"
  subsystem_name = "thirdparty"
}
```

#### 关键配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | 动作输出 | 自动生成的文件列表 |
| `external_deps` | zlib:libz | 系统 zlib 依赖 |
| `FT_CONFIG_OPTION_SYSTEM_ZLIB` | 启用 | 使用系统 zlib |
| `FT_CONFIG_OPTION_USE_PNG` | 条件启用 | PNG 位图支持 |

### 3.3 轻量级系统构建 (lite_library)

```gn
lite_library("freetype") {
  if (ohos_kernel_type == "liteos_m") {
    target_type = "static_library"
  } else {
    target_type = "shared_library"
  }
  
  freetype_sources = get_target_outputs(":freetype_action")
  
  deps = [ "//third_party/libpng:libpng" ]
  sources = freetype_sources
  include_dirs = [
    "${target_gen_dir}/freetype/src/base",
    "//third_party/libpng",
  ]
  
  public_configs = [ ":freetype_config" ]
  defines = [ "FT_CONFIG_OPTION_USE_PNG" ]
  
  if (target_type == "static_library") {
    deps += [ "//build/lite/config/component/zlib:zlib_static" ]
    defines += [ "FT_CONFIG_OPTION_SYSTEM_ZLIB" ]
  }
  
  deps += [ ":freetype_action" ]
  
  # ICCARM 编译器特殊处理
  if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
    cflags = [
      "--diag_suppress",
      "Pa082,Pa084,Pa128,Pe128,Pa134,Pa137,Pe550",
    ]
    cflags_cc = cflags
  }
}
```

---

## 4. 多架构支持

### 4.1 ftconfig.h 架构配置

```c
#ifndef __FTCONFIG_H__MULTILIB
#define __FTCONFIG_H__MULTILIB

#include <bits/wordsize.h>

#if __WORDSIZE == 32
# include "ftconfig-32.h"
#elif __WORDSIZE == 64
# include "ftconfig-64.h"
#else
# error "unexpected value for __WORDSIZE macro"
#endif

#endif
```

### 4.2 架构配置差异

| 配置项 | 32 位 | 64 位 | 说明 |
|--------|-------|-------|------|
| `FT_CONFIG_OPTION_OLD_INTL` | 可能启用 | 禁用 | 国际化支持 |
| 指针大小 | 4 字节 | 8 字节 | 影响内存布局 |
| long 大小 | 4 字节 | 8 字节 | 影响 API 返回值 |

---

## 5. 条件编译选项

### 5.1 功能开关

| 宏定义 | 状态 | 说明 |
|--------|------|------|
| `FT2_BUILD_LIBRARY` | 必需 | 库构建模式 |
| `FT_CONFIG_OPTION_USE_PNG` | 可选 | PNG 位图支持 |
| `FT_CONFIG_OPTION_SYSTEM_ZLIB` | 可选 | 系统 zlib |
| `FT_CONFIG_OPTION_SUBPIXEL_RENDERING` | 启用 | 子像素渲染 |

### 5.2 平台特定配置

```gn
# 标准 OHOS
if (current_os == "ohos") {
  defines += [ "FT_CONFIG_OPTION_USE_PNG" ]
  external_deps += [ "libpng:libpng" ]
}

# ArkUI X
if (is_arkui_x) {
  defines += [ "FT_CONFIG_OPTION_USE_PNG" ]
  deps += [ "//third_party/skia/m133/third_party/libpng" ]
}

# OHOS SDK
if (product_name == "ohos-sdk") {
  defines += [ "FT_CONFIG_OPTION_USE_PNG" ]
  external_deps += [ "libpng:libpng_static" ]
}
```

---

## 6. 与上游构建差异

### 6.1 构建系统差异

| 方面 | 上游 (Make/CMake) | OH (GN) |
|------|-------------------|---------|
| 源文件管理 | 手动指定 | 自动生成 |
| 补丁应用 | 用户手动 | 自动化 |
| 头文件配置 | configure 脚本 | ftconfig.h |
| 多架构 | 自动检测 | 显式配置 |

### 6.2 主要差异点

1. **源文件生成**: 上游使用模块配置文件，OH 通过 `freetype_action` 生成
2. **补丁集成**: 上游用户手动打补丁，OH 自动化集成
3. **配置方式**: 上游使用 ./configure，OH 使用条件编译

### 6.3 兼容性考虑

- OH 的 `freetype_action` 生成的文件列表与上游一致
- 补丁应用后，源代码与上游补丁版本兼容
- 条件编译确保不同平台正确配置

---

## 7. 常见构建问题

### 7.1 编译错误排查

#### 错误: 头文件找不到

```bash
# 检查 ftconfig.h 是否正确复制
ls -la ${target_gen_dir}/freetype/include/freetype2/

# 检查 include_dirs 配置
grep -A5 "include_dirs" BUILD.gn
```

#### 错误: 符号未定义

```bash
# 检查依赖是否正确
nm -g libfreetype.a | grep FT_Stream_New

# 检查是否启用了 enable-funcs patch
grep -r "FT_EXPORT.*FT_Stream_New" ${target_gen_dir}/
```

### 7.2 性能优化

#### 禁用不需要的模块

```gn
# 在 lite_library 中
defines -= [ "FT_CONFIG_OPTION_SUBPIXEL_RENDERING" ]
```

#### 启用优化

```gn
# 确保使用系统 zlib
defines += [ "FT_CONFIG_OPTION_SYSTEM_ZLIB" ]
```

---

*文档版本: 1.0*
*最后更新: 2025-02-08*
