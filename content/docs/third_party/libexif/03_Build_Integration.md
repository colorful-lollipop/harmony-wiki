# OH 构建适配

## 概述

libexif 通过 `BUILD.gn` 文件集成到 OpenHarmony 构建系统，支持多种构建环境和安全加固措施。

### 构建文件位置

```
third_party/libexif/
├── BUILD.gn              # OH 构建配置
├── config.h              # 上游配置文件（0.6.24.1）
└── config.h.in           # 上游配置模板
```

## 构建目标

### OH Lite（轻量级系统）

#### 目标结构

```gn
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")

  config("libexif_config") {
    include_dirs = [...]
    libexif_source = [...]
    cflags = [...]
  }

  lite_library("libexif") {
    if (ohos_kernel_type == "liteos_m") {
      target_type = "static_library"    # LiteOS-M 使用静态库
    } else {
      target_type = "shared_library"   # LiteOS-A 使用共享库
    }
    sources = libexif_source
    public_configs = [ ":libexif_config" ]
  }
}
```

#### 关键特性

| 特性 | 说明 |
|------|------|
| **内核类型判断** | `ohos_kernel_type == "liteos_m"` 选择静态库，否则共享库 |
| **编译选项** | `-DGETTEXT_PACKAGE`, `-DLOCALEDIR` |
| **包含目录** | libexif 根目录 + 所有厂商子目录（包括 huawei） |
| **适用场景** | IP 摄像头、IoT 设备 |

### 标准系统

#### 目标结构

标准系统提供两个构建目标：

```gn
else {
  import("//build/ohos.gni")

  // 私有配置
  config("build_private_config") {
    cflags = [
      "-DGETTEXT_PACKAGE=\"libexif-12\"",
      "-DLOCALEDIR=\"//third_party/libexif/build/share/locale\"",
      "-Werror",                    # 将警告视为错误
      "-Wno-format",                # 禁用格式字符串警告
      "-Wno-sign-compare",          # 禁用符号比较警告
      "-Wno-unused-parameter",      # 禁用未使用参数警告
      "-DHAVE_CONFIG_H",            # 使用 config.h
    ]
  }

  // 公共配置
  config("build_public_config") {
    include_dirs = [ "//third_party/libexif" ]
  }

  // 静态库
  ohos_source_set("exif_static") {
    subsystem_name = "thirdparty"
    part_name = "libexif"

    // arkui_x 特殊处理
    if (is_arkui_x) {
      deps = [ "//third_party/bounds_checking_function:libsec_static" ]
    } else {
      external_deps = [ "bounds_checking_function:libsec_shared" ]
    }

    branch_protector_ret = "pac_ret"      # 分支保护
    sources = [
      // 包含所有 libexif 源文件 + huawei 源文件
      "//third_party/libexif/libexif/exif-byte-order.c",
      ...
      "//third_party/libexif/libexif/huawei/exif-mnote-data-huawei.c",
      ...
    ]
    include_dirs = [
      "//third_party/libexif",
      "//third_party/libexif/libexif",
      "//third_party/libexif/libexif/pentax",
      ...
      "//third_party/libexif/libexif/huawei",  # ← 包含 huawei 目录
    ]
    configs = [ ":build_private_config" ]
  }

  // 共享库
  ohos_shared_library("libexif") {
    branch_protector_ret = "pac_ret"
    deps = [ ":exif_static" ]

    if (is_arkui_x) {
      deps += [ "//third_party/bounds_checking_function:libsec_static" ]
    } else {
      external_deps = [ "bounds_checking_function:libsec_shared" ]
    }

    public_configs = [ ":build_public_config" ]
    install_images = [ system_base_dir ]     # 安装到系统基础目录
    subsystem_name = "thirdparty"
    innerapi_tags = [ "chipsetsdk" ]      # 芯片 SDK API
    part_name = "libexif"
  }
}
```

#### 双目标设计

| 目标 | 类型 | 用途 |
|------|------|------|
| **exif_static** | ohos_source_set | 内部静态库，被 libexif 共享库依赖 |
| **libexif** | ohos_shared_library | 最终的共享库，安装到系统 |

#### 依赖关系

```
libexif (共享库)
    ↓ 依赖
exif_static (静态库)
    ↓ 依赖
bounds_checking_function:libsec_shared (共享)
```

## arkui_x 支持

### 适配逻辑

arkui_x 是 OpenHarmony 的跨平台 UI 框架，需要特殊的依赖处理。

```gn
// exif_static 目标
if (is_arkui_x) {
  deps = [ "//third_party/bounds_checking_function:libsec_static" ]
} else {
  external_deps = [ "bounds_checking_function:libsec_shared" ]
}

// libexif 目标
if (is_arkui_x) {
  deps += [ "//third_party/bounds_checking_function:libsec_static" ]
} else {
  external_deps = [ "bounds_checking_function:libsec_shared" ]
}
```

### 差异对比

| 配置 | is_arkui_x = true | is_arkui_x = false |
|------|-------------------|---------------------|
| **依赖类型** | `deps` (静态) | `external_deps` (外部组件） |
| **bounds_checking** | `libsec_static` | `libsec_shared` |
| **原因** | arkui_x 不支持 external_deps | 标准 OH 系统使用 external_deps |

## 编译选项分析

### 核心编译标志

```gn
cflags = [
  "-DGETTEXT_PACKAGE=\"libexif-12\"",
  "-DLOCALEDIR=\"//third_party/libexif/build/share/locale\"",
  "-Werror",              # ← 严格模式：警告即错误
  "-Wno-format",          # 禁用格式字符串警告（兼容性）
  "-Wno-sign-compare",    # 禁用符号比较警告（兼容性）
  "-Wno-unused-parameter", # 禁用未使用参数警告（兼容性）
  "-DHAVE_CONFIG_H",       # 启用 config.h
]
```

### 选项说明

| 选项 | 作用 | OH 原因 |
|------|------|---------|
| `-DGETTEXT_PACKAGE` | 设置国际化包名 | 保持上游兼容 |
| `-DLOCALEDIR` | 设置本地化文件路径 | 虽未使用，但保留配置 |
| `-Werror` | 将警告视为错误 | **安全加固**，强制代码质量 |
| `-Wno-format` | 禁用格式字符串警告 | 兼容旧的格式字符串 |
| `-Wno-sign-compare` | 禁用符号比较警告 | 兼容旧代码风格 |
| `-Wno-unused-parameter` | 禁用未使用参数警告 | 兼容上游接口 |
| `-DHAVE_CONFIG_H` | 启用 config.h | 配置特性检测 |

### 严格编译模式

`-Werror` 是 OH 的安全加固措施之一，确保：
- 编译时发现所有潜在问题
- 不允许隐藏警告
- 与上游保持一致的代码质量

## 安全加固

### 1. 分支保护

```gn
branch_protector_ret = "pac_ret"
```

| 配置 | 说明 |
|------|------|
| `branch_protector_ret` | 返回地址保护（Return Address Protection） |
| `pac_ret` | 使用 PAC (Pointer Authentication) 保护返回地址 |

**作用**: 防止 ROP (Return-Oriented Programming) 攻击

### 2. 边界检查

```gn
// exif_static 目标
if (is_arkui_x) {
  deps = [ "//third_party/bounds_checking_function:libsec_static" ]
} else {
  external_deps = [ "bounds_checking_function:libsec_shared" ]
}

// libexif 目标
if (is_arkui_x) {
  deps += [ "//third_party/bounds_checking_function:libsec_static" ]
} else {
  external_deps = [ "bounds_checking_function:libsec_shared" ]
}
```

**bounds_checking_function** 是 OH 的边界检查库，提供：
- `memcpy_s` - 安全的内存拷贝
- `memmove_s` - 安全的内存移动
- `strcpy_s` - 安全的字符串拷贝
- 等等...

**作用**: 防止缓冲区溢出、越界读写

### 3. CFI (Control Flow Integrity)

虽然 libexif 自身的 BUILD.gn 未启用 CFI，但依赖它的模块（如 libjpegplugin）启用了：

```gn
// 来自 libjpegplugin/BUILD.gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  cfi_vcall_icall_only = true
  debug = false
}
```

**作用**: 防止控制流劫持攻击（如 JOP）

## 源文件包含

### 包含目录

```gn
include_dirs = [
  "//third_party/libexif",                # 根目录
  "//third_party/libexif/libexif",         # 主 libexif 目录
  "//third_party/libexif/libexif/pentax",  # Pentax Maker Note
  "//third_party/libexif/libexif/olympus", # Olympus Maker Note
  "//third_party/libexif/libexif/apple",   # Apple Maker Note
  "//third_party/libexif/libexif/canon",   # Canon Maker Note
  "//third_party/libexif/libexif/fuji",    # Fuji Maker Note
  "//third_party/libexif/libexif/apple",   # Apple Maker Note (重复，保留)
  "//third_party/libexif/contrib/watcom", # Watcom 编译器支持
  "//third_party/libexif/libexif/huawei",  # ← Huawei Maker Note
]
```

### 源文件列表（标准系统）

| 类别 | 文件 | 说明 |
|------|------|------|
| **核心 EXIF** | exif-byte-order.c, exif-content.c, exif-data.c, exif-entry.c, exif-format.c, exif-gps-ifd.c, exif-ifd.c, exif-loader.c, exif-log.c, exif-mem.c, exif-mnote-data.c, exif-tag.c, exif-utils.c | 13 个核心文件 |
| **Pentax** | exif-mnote-data-pentax.c, mnote-pentax-entry.c, mnote-pentax-tag.c | 宾得 Maker Note |
| **Olympus** | exif-mnote-data-olympus.c, mnote-olympus-entry.c, mnote-olympus-tag.c | 奥林巴斯 Maker Note |
| **Apple** | exif-mnote-data-apple.c, mnote-apple-entry.c, mnote-apple-tag.c | iOS Maker Note |
| **Canon** | exif-mnote-data-canon.c, mnote-canon-entry.c, mnote-canon-tag.c | 佳能 Maker Note |
| **Fuji** | exif-mnote-data-fuji.c, mnote-fuji-entry.c, mnote-fuji-tag.c | 富士 Maker Note |
| **Huawei** | exif-mnote-data-huawei.c, mnote-huawei-data-type.c, mnote-huawei-entry.c, mnote-huawei-tag.c | **华为 Maker Note (OH 新增)** |

**总计**: 13 + 15 + 15 + 9 + 9 + 9 + 4 = **74 个源文件**

### 源文件统计

| 类别 | 文件数 | 代码行数（估算） |
|------|--------|----------------|
| 核心库 | 13 | ~10,000 |
| 厂商 Maker Note | 18 | ~15,000 |
| **华为 Maker Note** | 4 | **~2,174** |
| **总计** | 35 | ~27,174 |

## 安装配置

### 标准系统

```gn
ohos_shared_library("libexif") {
  install_images = [ system_base_dir ]    # 安装到系统基础目录
  subsystem_name = "thirdparty"
  innerapi_tags = [ "chipsetsdk" ]      # 芯片 SDK API
  part_name = "libexif"
}
```

| 配置 | 说明 | 安装路径 |
|------|------|----------|
| `install_images` | 目标安装位置 | `/system/lib64/libexif.so` |
| `innerapi_tags` | API 标签 | "chipsetsdk"（芯片 SDK） |
| `part_name` | 组件名称 | "libexif" |
| `subsystem_name` | 子系统名称 | "thirdparty" |

### OH Lite

OH Lite 不安装 libexif，而是直接链接到目标应用或服务。

## 版本差异

### config.h 版本

```
#define PACKAGE_VERSION "0.6.24.1"
#define VERSION "0.6.24.1"
```

**问题**: config.h 显示 0.6.24.1，但 README.OpenSource 显示 0.6.25

**可能原因**:
1. config.h 未同步更新
2. OH 基于 0.6.24.1 定制

**建议**: TODO(需确认) 是否需要升级到 0.6.25

## 与上游构建系统的差异

### 上游构建系统

上游使用 **Autotools** (autoconf/automake)：

```
./configure --prefix=/usr --enable-nls
make
make install
```

**配置文件**: `configure.ac`, `Makefile.am`

### OH 构建系统

OH 使用 **GN (Generate Ninja)**：

```
gn gen out/ohos
ninja -C out/ohos
```

**配置文件**: `BUILD.gn`

### 差异对比

| 方面 | 上游 | OH |
|------|------|-----|
| **构建工具** | Autotools + Make | GN + Ninja |
| **配置方式** | configure 脚本 | GN 声明式配置 |
| **依赖管理** | pkg-config | GN external_deps |
| **交叉编译** | `--host=` | `target_cpu`, `target_os` |
| **安装** | `make install` | `install_images` |

### 移植要点

从上游 Autotools 移植到 GN 时：

1. **源文件列表**: 将 Makefile.am 中的 `*.c` 映射到 GN `sources`
2. **包含目录**: 将 configure 检测的头文件路径映射到 GN `include_dirs`
3. **编译选项**: 将 configure 启用的特性映射到 GN `defines` 和 `cflags`
4. **依赖关系**: 将 PKG_CHECK_MODULES 映射到 GN `external_deps`

## 使用示例

### 如何在 BUILD.gn 中依赖 libexif

#### 示例 1：静态链接

```gn
ohos_static_library("my_exif_parser") {
  sources = [ "exif_parser.cpp" ]
  deps = [
    "//third_party/libexif:exif_static",  # 静态链接 exif_static
  ]
}
```

#### 示例 2：动态链接（推荐）

```gn
ohos_shared_library("my_image_decoder") {
  sources = [ "decoder.cpp" ]
  external_deps = [
    "libexif:libexif",  # 动态链接 libexif 共享库
  ]
}
```

#### 示例 3：同时依赖多个版本（arkui_x）

```gn
ohos_source_set("my_decoder") {
  if (is_arkui_x) {
    deps = [ "//third_party/libexif:exif_static" ]
  } else {
    external_deps = [ "libexif:libexif" ]
  }
}
```

### 头文件包含

```cpp
// 推荐：使用完整的 include 路径
#include <libexif/exif-data.h>
#include <libexif/exif-entry.h>
#include <libexif/huawei/mnote-huawei-tag.h>  // 华为 Maker Note

// 不推荐：简化 include（可能冲突）
#include <exif-data.h>
```

## 总结

libexif 的 OH 构建适配具有以下特点：

### 关键配置

✅ **双目标设计**: `exif_static`（内部）+ `libexif`（最终共享库）
✅ **多环境支持**: OH Lite（静态/共享） + 标准系统
✅ **arkui_x 兼容**: 特殊处理 `is_arkui_x` 场景
✅ **严格编译**: `-Werror` 强制代码质量
✅ **安全加固**: `branch_protector_ret`, `bounds_checking_function`

### 构建统计

| 指标 | 数值 |
|------|------|
| **源文件数量** | 74 个 |
| **代码行数** | ~27,174 行 |
| **厂商支持** | 6 个（华为、Apple、Canon、Fuji、Olympus、Pentax） |
| **华为代码占比** | ~8% |
| **ROM 大小** | 196KB |
| **RAM 大小** | 392KB |

### 安装配置

- **标准系统**: 安装到 `/system/lib64/libexif.so`
- **OH Lite**: 直接链接到目标应用
- **API 标签**: chipsetsdk（芯片 SDK）

## 参考资料

- **OH 构建系统文档**: OH 官方文档
- **GN 语言参考**: https://gn.googlesource.com/gn/+/master/docs/reference.md
- **BUILD.gn**: `third_party/libexif/BUILD.gn`
- **bundle.json**: `third_party/libexif/bundle.json`
