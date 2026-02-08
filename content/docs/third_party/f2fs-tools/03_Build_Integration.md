# 03 - OH 构建适配

## 3.1 BUILD.gn 结构总览

f2fs-tools 使用 GN（Generate Ninja）构建系统替代了上游的 autotools。项目包含 5 个 BUILD.gn 文件：

```
f2fs-tools/
├── BUILD.gn              # 根 BUILD.gn，定义 group 目标
├── lib/
│   └── BUILD.gn          # libf2fs 共享库
├── fsck/
│   └── BUILD.gn          # fsck.f2fs 可执行文件
├── mkfs/
│   └── BUILD.gn          # mkfs.f2fs 可执行文件
└── tools/
    └── BUILD.gn          # 辅助工具（f2fscrypt, f2fstat, fibmap.f2fs）
```

---

## 3.2 根 BUILD.gn 详解

### 文件内容
```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2 as
# published by the Free Software Foundation.

import("//build/ohos.gni")

group("f2fs-tools") {
  deps = [
    "//third_party/f2fs-tools/fsck:fsck.f2fs",
    "//third_party/f2fs-tools/lib:libf2fs",
    "//third_party/f2fs-tools/mkfs:mkfs.f2fs",
  ]
}

group("f2fs-tools_host_toolchain") {
  deps = [
    "//third_party/f2fs-tools/fsck:fsck.f2fs($host_toolchain)",
    "//third_party/f2fs-tools/mkfs:mkfs.f2fs($host_toolchain)",
  ]
}
```

### 目标说明

| 目标 | 类型 | 用途 |
|-----|------|------|
| `f2fs-tools` | group | 目标设备工具集合，构建所有目标平台工具 |
| `f2fs-tools_host_toolchain` | group | 主机工具链版本，用于构建系统镜像 |

### 主机工具链用途

`f2fs-tools_host_toolchain` 在以下场景使用：
- 构建 system.img 时需要在主机上运行 mkfs.f2fs
- 构建 updater.img 时需要格式化分区镜像
- 见 `build/ohos/images/BUILD.gn` 的引用

---

## 3.3 lib/BUILD.gn 详解

### 文件内容
```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2 as
# published by the Free Software Foundation.

import("//build/ohos.gni")

config("f2fs-defaults") {
  cflags = [
    "-Wall",
    "-Werror",
    "-Wno-incompatible-pointer-types",
    "-Wno-unused-function",
    "-Wno-unused-parameter",
    "-Wno-format",
  ]
}

config("libf2fs-headers") {
  include_dirs = [
    ".",
    "//third_party/f2fs-tools",
    "//third_party/f2fs-tools/include",
  ]
}

ohos_shared_library("libf2fs") {
  sources = [
    "libf2fs.c",
    "libf2fs_io.c",
    "libf2fs_zoned.c",
    "nls_utf8.c",
    "libf2fs_log.c",        # OH 新增
    "libf2fs_dmd.c"         # OH 新增
  ]

  include_dirs = [ "." ]

  external_deps = [
    "bounds_checking_function:libsec_shared",  # 安全函数库
  ]

  configs = [
    ":f2fs-defaults",
    ":libf2fs-headers",
  ]

  defines = [ "HAVE_CONFIG_H" ]
  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "f2fs-tools"
  install_images = [
    "system",
    "updater",
  ]
}
```

### 关键配置分析

#### 1. 编译器标志 (cflags)

| 标志 | 说明 |
|-----|------|
| `-Wall` | 启用所有警告 |
| `-Werror` | 将警告视为错误 |
| `-Wno-incompatible-pointer-types` | 禁用不兼容指针类型警告 |
| `-Wno-unused-function` | 禁用未使用函数警告 |
| `-Wno-unused-parameter` | 禁用未使用参数警告 |
| `-Wno-format` | 禁用格式字符串警告 |

**说明**: 禁用部分警告是为了兼容上游代码风格，同时确保在 OH 严格的编译环境下通过。

#### 2. 源文件

| 源文件 | 说明 |
|-------|------|
| `libf2fs.c` | 核心 F2FS 操作库 |
| `libf2fs_io.c` | I/O 操作封装 |
| `libf2fs_zoned.c` | 分区设备（Zoned Device）支持 |
| `nls_utf8.c` | UTF-8 字符集支持 |
| `libf2fs_log.c` | **OH 新增**: 增强日志系统 |
| `libf2fs_dmd.c` | **OH 新增**: DFX 诊断模块 |

#### 3. 外部依赖

```gn
external_deps = [
    "bounds_checking_function:libsec_shared",
]
```

**bounds_checking_function** 是 OH 的安全函数库，提供：
- `strncpy_s` - 安全的字符串拷贝
- `vsnprintf_s` - 安全的格式化输出
- `memset_s` - 安全的内存设置
- 其他安全边界检查函数

#### 4. 安装配置

```gn
install_enable = true
install_images = [
    "system",      # 安装到 system 分区
    "updater",     # 安装到 updater 分区（恢复模式）
]
```

---

## 3.4 fsck/BUILD.gn 详解

### 文件内容
```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.

import("//build/ohos.gni")

config("f2fs-defaults") {
  cflags = [
    "-Wno-pointer-sign",
    "-Wno-unused-variable",
    "-Wno-string-plus-int",
    "-Wno-error=format",
    "-Wno-unused-function",
    "-Wno-unused-parameter",
    "-Wno-incompatible-pointer-types",
  ]
  ldflags = [ "-lpthread" ]  # 链接 pthread 库
}

ohos_executable("fsck.f2fs") {
  configs = [ ":f2fs-defaults" ]
  sources = [
    "../lib/extra_fsck.c",           # 额外辅助功能
    "../tools/debug_tools/fsck_debug.c",
    "../tools/f2fs_tools/f2fs_tools.c",
    "compress.c",
    "dedup.c",
    "defrag.c",
    "dict.c",
    "dir.c",
    "dump.c",
    "fsck.c",
    "main.c",
    "mkquota.c",
    "mount.c",
    "node.c",
    "quotaio.c",
    "quotaio_tree.c",
    "quotaio_v2.c",
    "resize.c",
    "segment.c",
    "sload.c",
    "xattr.c",
    "queue.c"
  ]

  include_dirs = [
    ".",
    "../tools/debug_tools",
    "../tools/f2fs_tools",
    "//third_party/f2fs-tools",
    "//third_party/f2fs-tools/include",
    "//third_party/f2fs-tools/lib",
  ]

  deps = [ "//third_party/f2fs-tools/lib:libf2fs" ]

  external_deps = [
    "bounds_checking_function:libsec_shared",
    "e2fsprogs:libdacconfig",        # DAC 配置支持
  ]

  defines = [ "HAVE_CONFIG_H" ]

  symlink_target_name = [
    "resize.f2fs",
    "sload.f2fs",
  ]

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "f2fs-tools"
  install_images = [
    "system",
    "updater",
  ]
}
```

### 关键特性

#### 1. 符号链接
```gn
symlink_target_name = [
    "resize.f2fs",
    "sload.f2fs",
]
```

fsck.f2fs 通过 `argv[0]` 判断调用方式，实现多功能合一：
- `fsck.f2fs` - 文件系统检查
- `resize.f2fs` - 调整大小
- `sload.f2fs` - 加载文件到镜像

#### 2. 额外源文件
- `extra_fsck.c` - OH 添加的辅助功能
- `fsck_debug.c` - 调试支持
- `f2fs_tools.c` - 工具公共代码

#### 3. e2fsprogs 依赖
```gn
external_deps = [
    "e2fsprogs:libdacconfig",
]
```

`libdacconfig` 提供 DAC（Discretionary Access Control）配置支持。

---

## 3.5 mkfs/BUILD.gn 详解

### 文件内容
```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.

import("//build/ohos.gni")

config("f2fs-defaults") {
  cflags = [
    "-Wno-pointer-sign",
    "-Wno-unused-variable",
    "-Wno-string-plus-int",
    "-Wno-error=format",
  ]
}

ohos_executable("mkfs.f2fs") {
  configs = [ ":f2fs-defaults" ]
  sources = [
    "../lib/extra_fsck.c",
    "f2fs_format.c",
    "f2fs_format_main.c",
    "f2fs_format_utils.c",
  ]

  include_dirs = [
    ".",
    "//third_party/f2fs-tools",
    "//third_party/f2fs-tools/lib",
    "//third_party/f2fs-tools/include",
  ]

  deps = [ "//third_party/f2fs-tools/lib:libf2fs" ]

  external_deps = [
    "e2fsprogs:libext2_uuid",        # UUID 生成
    "e2fsprogs:libdacconfig"
  ]

  defines = [ "HAVE_CONFIG_H" ]

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "f2fs-tools"
  install_images = [
    "system",
    "updater",
  ]
}
```

### 关键依赖

```gn
external_deps = [
    "e2fsprogs:libext2_uuid",    # UUID 生成功能
    "e2fsprogs:libdacconfig",
]
```

`libext2_uuid` 用于生成 F2FS 文件系统的唯一标识符（UUID）。

---

## 3.6 tools/BUILD.gn 详解

### 文件内容
```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.

import("//build/ohos.gni")

config("f2fs-defaults") {
  cflags = [
    "-std=gnu89",
    "-Wno-implicit-function-declaration",
    "-Wno-pointer-sign",
  ]
}

# f2fscrypt - 加密管理工具
ohos_executable("f2fscrypt") {
  configs = [ ":f2fs-defaults" ]
  sources = [
    "f2fscrypt.c",
    "sha512.c",
  ]
  include_dirs = [
    ".",
    "//third_party/f2fs-tools",
    "//third_party/f2fs-tools/include",
  ]
  cflags = [ "-Wno-error=format-extra-args" ]

  external_deps = [ "e2fsprogs:libext2_uuid" ]

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "f2fs-tools"
  install_images = [ "system" ]
}

# f2fstat - 统计工具
ohos_executable("f2fstat") {
  configs = [ ":f2fs-defaults" ]
  sources = [ "f2fstat.c" ]

  include_dirs = [ "." ]
  cflags = [
    "-Wno-error=format",
    "-Wno-error=type-limits",
    "-Wno-format-extra-args",
  ]

  deps = []

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "f2fs-tools"
  install_images = [ "system" ]
}

# fibmap.f2fs - 块映射工具
ohos_executable("fibmap.f2fs") {
  configs = [ ":f2fs-defaults" ]
  sources = [ "fibmap.c" ]

  include_dirs = [
    ".",
    "//third_party/f2fs-tools",
    "//third_party/f2fs-tools/include",
    "//third_party/f2fs-tools/lib",
  ]

  cflags = [
    "-Wno-error=format",
    "-Wno-error=type-limits",
    "-Wno-format-extra-args",
  ]

  deps = [ "//third_party/f2fs-tools/lib:libf2fs" ]

  defines = [ "HAVE_CONFIG_H" ]

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "f2fs-tools"
  install_images = [ "system" ]
}
```

### 工具说明

| 工具 | 功能 | 依赖 |
|-----|------|------|
| `f2fscrypt` | F2FS 文件系统加密管理 | libext2_uuid |
| `f2fstat` | 显示 F2FS 统计信息 | 无 |
| `fibmap.f2fs` | 显示文件块映射 | libf2fs |

---

## 3.7 与上游构建系统对比

### 上游构建系统（autotools）

```bash
# 上游构建流程
./autogen.sh          # 生成 configure
./configure           # 检测系统环境
make                  # 编译
make install          # 安装到 /usr/local
```

**configure.ac 关键检测**:
- 检测 libuuid 是否存在
- 检测 libselinux 是否可用
- 检测 SCSI/SG_IO 支持
- 检测各种系统头文件

### OH 构建系统（GN）

```bash
# OH 构建流程
gn gen out            # 生成 Ninja 文件
ninja -C out          # 编译
```

**关键差异**:

| 特性 | 上游 (autotools) | OH (GN) |
|-----|-----------------|---------|
| 配置方式 | 运行时检测 | 编译时硬编码在 config.h |
| 依赖管理 | pkg-config | external_deps |
| 安装路径 | /usr/local/{bin,lib} | system/updater 镜像 |
| 日志系统 | 标准输出 | kmsg + 文件日志 |
| 安全加固 | 无 | bounds_checking_function |

### config.h 关键定义

```c
// OH 预生成的 config.h
#define HAVE_LIBUUID 1
#define HAVE_UUID_UUID_H 1
#define WITH_BLKDISCARD 1
#define HAVE_FSYNC 1
#define WITH_OHOS 1          // OH 特有
```

**说明**: OH 不运行 configure 脚本，而是提供预生成的 config.h，包含适用于 OH 环境的配置。

---

## 3.8 编译配置总结

### 编译器警告处理策略

| 警告类型 | 处理方式 | 理由 |
|---------|---------|------|
| incompatible-pointer-types | 禁用 | 上游代码风格 |
| unused-function | 禁用 | 上游代码存在未使用的函数 |
| unused-parameter | 禁用 | 回调函数接口需要 |
| format | 降级 | 格式字符串与参数匹配问题 |
| pointer-sign | 禁用 | 字符指针符号问题 |

### 安全措施

1. **Bounds Checking**: 所有字符串操作使用安全函数
2. **Werror**: 启用警告作为错误（除明确禁用的）
3. **编译器标志**: 使用标准 GNU C89

### 安装配置

| 组件 | system 分区 | updater 分区 |
|-----|------------|-------------|
| libf2fs.so | ✅ | ✅ |
| fsck.f2fs | ✅ | ✅ |
| mkfs.f2fs | ✅ | ✅ |
| f2fscrypt | ✅ | ❌ |
| f2fstat | ✅ | ❌ |
| fibmap.f2fs | ✅ | ❌ |
