# OH 构建适配

本文档详细说明 zlib 在 OpenHarmony 中的构建系统适配，包括 BUILD.gn 结构、关键配置和与上游的差异。

## 构建系统对比

| 构建系统 | 用途 | OH 支持 |
|---------|-----|--------|
| CMakeLists.txt | 上游标准构建 | ⚠️ 仅通过 Patch 适配 |
| BUILD.gn | **OH 主构建系统** | ✅ 完整支持 |
| Makefile | Unix/Linux 通用 | ❌ 不支持 |

## BUILD.gn 结构

### 文件位置
```
third_party/zlib/BUILD.gn
```

### 完整内容解析

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd. All rights reserved.

import("//build/config/config.gni")
import("//build/ohos.gni")
import("//build/ohos/ndk/ndk.gni")
```

**导入的 GN 配置**:
- `config.gni` - 全局编译配置
- `ohos.gni` - OH 构建规则
- `ndk.gni` - NDK 导出配置

---

## 编译配置

### zlib_config

```gn
config("zlib_config") {
  cflags = [
    "-Wno-incompatible-pointer-types",    # 忽略不兼容指针类型警告
    "-Werror",                            # 将警告视为错误
    "-Wno-strict-prototypes",             # 忽略严格原型警告
    "-Wimplicit-function-declaration",    # 启用隐式函数声明警告
  ]
}
```

**配置说明**:

| 标志 | 作用 |
|-----|-----|
| `-Wno-incompatible-pointer-types` | zlib 源码中存在指针类型转换，需禁用此警告 |
| `-Werror` | 强制代码质量，确保无警告 |
| `-Wno-strict-prototypes` | 兼容旧式 C 函数声明 |
| `-Wimplicit-function-declaration` | 捕获未声明函数调用 |

### zlib_public_config

```gn
config("zlib_public_config") {
  include_dirs = [
    ".",                    # 主目录 (zlib.h, zconf.h)
    "contrib/minizip",      # minizip 头文件
  ]
}
```

**包含目录**:
- `.` - zlib 核心头文件 (zlib.h, zconf.h)
- `contrib/minizip` - ZIP 处理头文件 (zip.h, unzip.h, ioapi.h)

---

## 构建目标

### 1. libz (静态库)

```gn
ohos_static_library("libz") {
  sources = [
    "adler32.c",
    "compress.c",
    "contrib/minizip/ioapi.c",      # ★ OH 启用 minizip
    "contrib/minizip/unzip.c",      # ★ OH 启用 minizip
    "contrib/minizip/zip.c",        # ★ OH 启用 minizip
    "crc32.c",
    "crc32.h",
    "deflate.c",
    "deflate.h",
    "gzclose.c",
    "gzguts.h",
    "gzlib.c",
    "gzread.c",
    "gzwrite.c",
    "infback.c",
    "inffast.c",
    "inffast.h",
    "inffixed.h",
    "inflate.c",
    "inflate.h",
    "inftrees.c",
    "inftrees.h",
    "trees.c",
    "trees.h",
    "uncompr.c",
    "zconf.h",
    "zlib.h",
    "zutil.c",
    "zutil.h",
  ]
  configs = [ ":zlib_config" ]
  public_configs = [ ":zlib_public_config" ]

  part_name = "zlib"
  subsystem_name = "thirdparty"
}
```

**特性**:
- 包含 minizip (ZIP 文件处理)
- 静态链接到依赖模块
- 适用于需要独立 zlib 的模块

**使用模块示例**:
- `libpng` - PNG 图像库
- `arkcompiler` - 方舟编译器

### 2. shared_libz (共享库)

```gn
ohos_shared_library("shared_libz") {
  branch_protector_ret = "pac_ret"     # ★ ARM64 PAC 保护
  
  sources = [ ... ]  # 同 libz
  
  configs = [ ":zlib_config" ]
  public_configs = [ ":zlib_public_config" ]

  if (current_os == "ios") {
    ldflags = [
      "-Wl",
      "-install_name",
      "@rpath/libshared_libz.framework/libshared_libz",
    ]
  }

  install_images = [
    "system",      # 安装到 system 镜像
    "updater",     # 安装到 updater 镜像
  ]

  symlink_target_name = [ "libz.so" ]   # 创建符号链接 libz.so

  innerapi_tags = [
    "chipsetsdk_sp",
    "platformsdk",
  ]
  part_name = "zlib"
  subsystem_name = "thirdparty"
}
```

**特性**:
- 动态共享库 (.so)
- ARM64 PAC (Pointer Authentication) 保护
- 安装到 system 和 updater 镜像
- 创建 `libz.so` 符号链接便于查找

**使用模块示例**:
- `ArkUI ACE Engine`
- `Ability Runtime`
- `curl`

### 3. libz_crc (CRC 专用静态库)

```gn
# ARM64 CRC 优化配置
if (current_cpu == "arm64") {
  config("zlib_crc_config") {
    cflags = [
      "-Wno-incompatible-pointer-types",
      "-Werror",
      "-Wno-strict-prototypes",
      "-Wimplicit-function-declaration",
      "-march=armv8-a+crc",              # ★ 启用 ARM CRC32 指令
    ]
  }
} else {
  config("zlib_crc_config") {
    cflags = [
      "-Wno-incompatible-pointer-types",
      "-Werror",
      "-Wno-strict-prototypes",
      "-Wimplicit-function-declaration",
    ]
  }
}

ohos_static_library("libz_crc") {
  sources = [
    "crc32.c",
    "crc32.h",
    "zconf.h",
  ]
  configs = [ ":zlib_crc_config" ]
  public_configs = [ ":zlib_public_config" ]

  part_name = "zlib"
  subsystem_name = "thirdparty"
}
```

**特性**:
- 仅包含 CRC32 功能
- ARM64 平台启用硬件 CRC 指令 (`-march=armv8-a+crc`)
- 适用于仅需 CRC 校验的场景

**使用场景**:
- 需要高性能 CRC32 计算的模块
- 避免链接完整 zlib 减少体积

### 4. iOS 框架支持

```gn
if (current_os == "ios") {
  ohos_combine_darwin_framework("libshared_libz") {
    deps = [ ":shared_libz" ]
    subsystem_name = "thirdparty"
    part_name = "zlib"
  }
}
```

支持 iOS 平台打包为 Framework。

---

## 与上游 CMakeLists.txt 的差异

| 特性 | 上游 CMake | OH BUILD.gn |
|-----|-----------|-------------|
| 构建系统 | CMake | GN |
| 输出目标 | zlibstatic, zlib | libz, shared_libz, libz_crc |
| minizip | 可选 (默认关闭) | **强制启用** |
| 测试代码 | 构建 example, minigzip | **不构建** |
| ARM64 CRC | 需手动配置 | **自动检测启用** |
| iOS 框架 | 不支持 | 支持 |
| 安装路径 | 标准系统路径 | system/updater 镜像 |

### 关键差异说明

#### 1. minizip 强制启用

**上游 CMake** (默认关闭):
```cmake
option(ZLIB_BUILD_MINIZIP "Build minizip" OFF)
```

**OH BUILD.gn** (强制启用):
```gn
sources = [
  "contrib/minizip/ioapi.c",
  "contrib/minizip/unzip.c",
  "contrib/minizip/zip.c",
  ...
]
```

**原因**: OH Bundle Manager 需要 ZIP 功能解压 HAP 包。

#### 2. 测试代码禁用

**上游 CMake**:
```cmake
add_executable(example test/example.c)
add_executable(minigzip test/minigzip.c)
add_test(example example)
```

**OH BUILD.gn**: 不包含测试代码

**原因**: OH 使用 XTS 测试框架，不需要上游测试代码。

#### 3. ARM64 CRC 优化

**上游 CMake**: 需要手动添加编译选项

**OH BUILD.gn**:
```gn
if (current_cpu == "arm64") {
  cflags += [ "-march=armv8-a+crc" ]
}
```

**自动启用 ARM64 CRC32 硬件加速**。

#### 4. 多目标输出

**上游**: 仅 static/shared 两种

**OH**: 三种目标
- `libz` - 标准静态库
- `shared_libz` - 共享库 (带 PAC 保护)
- `libz_crc` - CRC 专用优化库

---

## 使用 BUILD.gn 依赖 zlib

### 依赖静态库 (libz)

```gn
ohos_static_library("my_module") {
  sources = [ "my_file.cpp" ]
  
  deps = [
    "//third_party/zlib:libz",
  ]
  
  # 自动继承 zlib_public_config 的 include_dirs
}
```

### 依赖共享库 (shared_libz)

```gn
ohos_shared_library("my_module") {
  sources = [ "my_file.cpp" ]
  
  external_deps = [
    "zlib:shared_libz",
  ]
}
```

### 依赖 CRC 库 (libz_crc)

```gn
ohos_static_library("my_module") {
  sources = [ "my_file.cpp" ]
  
  deps = [
    "//third_party/zlib:libz_crc",
  ]
}
```

---

## NDK 导出配置

### zlib.ndk.json

```json
[
    { "name": "_dist_code" },
    { "name": "_length_code" },
    ...
    { "name": "zlibVersion" }
]
```

导出 **94 个 API 函数**，完整支持：
- 压缩/解压: `compress`, `uncompress`, `deflate`, `inflate`
- gzip 文件: `gzopen`, `gzread`, `gzwrite`, `gzclose`
- 校验: `adler32`, `crc32`
- MiniZip: `zipOpen`, `unzOpen`, 等

---

## bundle.json 配置

```json
{
    "name": "@ohos/zlib",
    "description": "zlib 1.3.1 is a general purpose data compression library...",
    "version": "3.1",
    "license": "Zlib LICENSE",
    "component": {
        "name": "zlib",
        "subsystem": "thirdparty",
        "adapted_system_type": ["standard"],
        "build": {
            "inner_kits": [
                { "name": "//third_party/zlib:libz" },
                { "name": "//third_party/zlib:shared_libz" },
                { "name": "//third_party/zlib:libz_crc" }
            ]
        }
    }
}
```

**inner_kits**: 声明三个可导出目标供其他模块使用。

---

## 构建命令示例

```bash
# 构建 libz 静态库
gn gen out --args='...'
ninja -C out third_party/zlib:libz

# 构建 shared_libz 动态库
ninja -C out third_party/zlib:shared_libz

# 构建 libz_crc
ninja -C out third_party/zlib:libz_crc
```
