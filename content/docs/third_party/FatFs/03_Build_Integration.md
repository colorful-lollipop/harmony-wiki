# OH 构建适配

> OpenHarmony 构建系统与 FatFs 的集成方式

---

## 构建系统概览

OpenHarmony 使用 **GN（Generate Ninja）** 作为主要构建系统。FatFs 的构建集成涉及以下几个文件：

| 文件 | 路径 | 作用 |
|------|------|------|
| **FatFs.gni** | `third_party/FatFs/FatFs.gni` | 定义 FatFs 源文件和包含目录 |
| **BUILD.gn** | `kernel/liteos_a/fs/fat/BUILD.gn` | 编译 FatFs 核心库和 OH 适配层 |
| **BUILD.gn** | `kernel/liteos_a/fs/fat/virpart/BUILD.gn` | 编译虚拟分区模块 |
| **BUILD.gn** | `kernel/liteos_a/BUILD.gn` | FatFs 组件引用 |
| **Kconfig** | `kernel/liteos_a/fs/fat/Kconfig` | FatFs 配置选项 |

---

## 1. FatFs.gni 文件

### 文件信息

- **路径**：`third_party/FatFs/FatFs.gni`
- **大小**：1.8KB
- **作用**：定义 FatFs 源文件列表和包含目录

### 文件内容

```gni
// FatFs.gni
# Copyright (c) 2022-2022 Huawei Device Co., Ltd. All rights reserved.
# ...

FATFS_SRC_FILES = [
  "//third_party/FatFs/source/diskio.c",
  "//third_party/FatFs/source/ff.c",
  "//third_party/FatFs/source/ffsystem.c",
  "//third_party/FatFs/source/ffunicode.c",
]

FATFS_INCLUDE_DIRS = [ "//third_party/FatFs/source" ]
```

### 说明

#### FATFS_SRC_FILES

定义 FatFs 核心源文件列表：

| 文件 | 大小 | 说明 |
|------|------|------|
| `source/diskio.c` | 6.1KB | 磁盘 I/O 示例实现 |
| `source/ff.c` | 234KB | FAT 文件系统核心实现 |
| `source/ffsystem.c` | 5.3KB | 系统相关函数（内存管理、时间等） |
| `source/ffunicode.c` | 1.97MB | Unicode 支持（中文、日文等） |

#### FATFS_INCLUDE_DIRS

定义 FatFs 头文件包含目录：

```gni
FATFS_INCLUDE_DIRS = [ "//third_party/FatFs/source" ]
```

### 使用方式

FatFs.gni 被其他 BUILD.gn 文件导入：

```gni
// kernel/liteos_a/fs/fat/BUILD.gn:31
import("$THIRDPARTY_FATFS_DIR/FatFs.gni")
```

然后使用 `FATFS_SRC_FILES` 和 `FATFS_INCLUDE_DIRS`：

```gni
// kernel/liteos_a/fs/fat/BUILD.gn:42
sources += FATFS_SRC_FILES
```

---

## 2. LiteOS-A Fatfs BUILD.gn

### 文件信息

- **路径**：`kernel/liteos_a/fs/fat/BUILD.gn`
- **大小**：2.1KB
- **作用**：编译 FatFs 核心库和 OH 适配层

### 文件内容

```gn
# kernel/liteos_a/fs/fat/BUILD.gn
# Copyright (c) 2013-2019 Huawei Technologies Co., Ltd. All rights reserved.
# ...

import("//kernel/liteos_a/liteos.gni")
import("$THIRDPARTY_FATFS_DIR/FatFs.gni")

module_switch = defined(LOSCFG_FS_FAT)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  sources = [
    "os_adapt/fat_shellcmd.c",
    "os_adapt/fatfs.c",
    "os_adapt/format.c",
  ]

  sources += FATFS_SRC_FILES

  include_dirs = [ "os_adapt" ]

  public_configs = [ ":public" ]
}

config("public") {
  include_dirs = FATFS_INCLUDE_DIRS
}
```

### 详细说明

#### 2.1 导入语句

```gn
// BUILD.gn:30
import("//kernel/liteos_a/liteos.gni")
```

**说明**：导入 LiteOS-A 的 GNI 模板和公共配置。

```gn
// BUILD.gn:31
import("$THIRDPARTY_FATFS_DIR/FatFs.gni")
```

**说明**：导入 FatFs 的源文件和包含目录定义。

- `$THIRDPARTY_FATFS_DIR` 指向 `third_party/FatFs`

#### 2.2 模块开关

```gn
// BUILD.gn:33
module_switch = defined(LOSCFG_FS_FAT)
```

**说明**：
- 只有启用 `LOSCFG_FS_FAT` 时才编译此模块
- `LOSCFG_FS_FAT` 由 Kconfig 配置（`FS_FAT`）

#### 2.3 模块名称

```gn
// BUILD.gn:34
module_name = get_path_info(rebase_path("."), "name")
```

**说明**：获取当前目录名称作为模块名称（"fat"）。

#### 2.4 源文件列表

```gn
// BUILD.gn:36-42
sources = [
  "os_adapt/fat_shellcmd.c",
  "os_adapt/fatfs.c",
  "os_adapt/format.c",
]

sources += FATFS_SRC_FILES
```

**说明**：
- 第一部分：OH 适配层文件（3 个）
- 第二部分：FatFs 核心文件（来自 `FATFS_SRC_FILES`）

**OH 适配层文件**：

| 文件 | 大小 | 说明 |
|------|------|------|
| `os_adapt/fatfs.c` | 61KB | **核心**：VFS 适配和 FatFs 封装 |
| `os_adapt/format.c` | 3.8KB | FAT 格式化工具 |
| `os_adapt/fat_shellcmd.c` | 3.4KB | Shell 命令（mkfs, fat 等） |

#### 2.5 包含目录

```gn
// BUILD.gn:44
include_dirs = [ "os_adapt" ]
```

**说明**：添加 OH 适配层头文件目录到编译搜索路径。

#### 2.6 公共配置

```gn
// BUILD.gn:46-47
public_configs = [ ":public" ]
```

```gn
// BUILD.gn:49-51
config("public") {
  include_dirs = FATFS_INCLUDE_DIRS
}
```

**说明**：
- 定义公共配置 `:public`
- 将 FatFs 的头文件目录（`FATFS_INCLUDE_DIRS`）导出为公共包含目录
- 依赖此模块的其他模块可以直接使用 FatFs 头文件

---

## 3. 虚拟分区 BUILD.gn

### 文件信息

- **路径**：`kernel/liteos_a/fs/fat/virpart/BUILD.gn`
- **大小**：1.8KB
- **作用**：编译虚拟分区模块

### 文件内容

```gn
# kernel/liteos_a/fs/fat/virpart/BUILD.gn
# Copyright (c) 2013-2019 Huawei Technologies Co., Ltd. All rights reserved.
# ...

import("//kernel/liteos_a/liteos.gni")

module_switch = defined(LOSCFG_FS_FAT_VIRTUAL_PARTITION)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  sources = [
    "src/virpart.c",
    "src/virpartff.c",
  ]

  include_dirs = [ "../os_adapt" ]

  public_configs = [ ":public" ]
}

config("public") {
  include_dirs = [ "include" ]
}
```

### 详细说明

#### 3.1 模块开关

```gn
// BUILD.gn:32
module_switch = defined(LOSCFG_FS_FAT_VIRTUAL_PARTITION)
```

**说明**：
- 只有启用 `LOSCFG_FS_FAT_VIRTUAL_PARTITION` 时才编译此模块
- `LOSCFG_FS_FAT_VIRTUAL_PARTITION` 由 Kconfig 配置（`FS_FAT_VIRTUAL_PARTITION`）

#### 3.2 源文件列表

```gn
// BUILD.gn:35-38
sources = [
  "src/virpart.c",
  "src/virpartff.c",
]
```

**虚拟分区文件**：

| 文件 | 说明 |
|------|------|
| `src/virpart.c` | 虚拟分区核心实现 |
| `src/virpartff.c` | 虚拟分区与 FatFs 集成 |

#### 3.3 包含目录

```gn
// BUILD.gn:40
include_dirs = [ "../os_adapt" ]
```

**说明**：添加 OH 适配层头文件目录（`../os_adapt`）。

#### 3.4 公共配置

```gn
// BUILD.gn:42-47
public_configs = [ ":public" ]

config("public") {
  include_dirs = [ "include" ]
}
```

**说明**：
- 将虚拟分区头文件目录（`include`）导出为公共包含目录
- 依赖此模块的其他模块可以使用虚拟分区接口

---

## 4. LiteOS-A 顶层 BUILD.gn

### 文件信息

- **路径**：`kernel/liteos_a/BUILD.gn`
- **作用**：引用 FatFs 组件

### 相关内容

```gn
# kernel/liteos_a/BUILD.gn
...
group("kernel_group") {
  deps = [
    ...
    "$LITEOSTHIRDPARTY/FatFs",
    ...
  ]
}
```

### 说明

- `$LITEOSTHIRDPARTY` 指向 `third_party`
- `FatFs` 指向 `third_party/FatFs/`
- 通过 `deps` 引用 FatFs 组件

---

## 5. Kconfig 配置

### 文件信息

- **路径**：`kernel/liteos_a/fs/fat/Kconfig`
- **大小**：1.1KB
- **作用**：定义 FatFs 配置选项

### 完整内容

```kconfig
# kernel/liteos_a/fs/fat/Kconfig
config FS_FAT
    bool "Enable FAT"
    default y
    depends on FS_VFS
    help
      Answer Y to enable LiteOS support fat filesystem.

config FS_FAT_CACHE
    bool "Enable FAT Cache"
    default y
    depends on FS_FAT
    help
      Answer Y to enable LiteOS fat filesystem support cache.

config FS_FAT_CACHE_SYNC_THREAD
    bool "Enable FAT Cache Sync Thread"
    default n
    depends on FS_FAT_CACHE
    help
      Answer Y to enable LiteOS fat filesystem support cache sync thread.

config FS_FAT_CHINESE
    bool "Enable Chinese"
    default y
    depends on FS_FAT
    help
      Answer Y to enable LiteOS fat filesystem support Chinese.

config FS_FAT_VIRTUAL_PARTITION
    bool "Enable Virtual Partition"
    default n
    depends on FS_FAT

config FS_FAT_VOLUMES
    int
    depends on FS_FAT
    default 32 if PLATFORM_HI3731
    default 16

config FS_FAT_DISK
    bool "Enable partinfo for storage device"
    depends on FS_VFS && (FS_FAT || DRIVERS_MMC || DRIVERS_USB)
    default y
```

### 配置项详解

#### 5.1 FS_FAT

```kconfig
config FS_FAT
    bool "Enable FAT"
    default y
    depends on FS_VFS
```

**说明**：
- 启用 FatFs 文件系统支持
- 默认值：`y`（启用）
- 依赖：`FS_VFS`（虚拟文件系统）

**影响**：
- 设置此选项为 `n` 会禁用 FatFs 整个模块
- `BUILD.gn` 中的 `module_switch = defined(LOSCFG_FS_FAT)` 会关闭编译

#### 5.2 FS_FAT_CACHE

```kconfig
config FS_FAT_CACHE
    bool "Enable FAT Cache"
    default y
    depends on FS_FAT
```

**说明**：
- 启用 FatFs 缓存功能
- 默认值：`y`（启用）
- 依赖：`FS_FAT`

**影响**：
- 提高文件系统性能
- 增加内存占用

#### 5.3 FS_FAT_CACHE_SYNC_THREAD

```kconfig
config FS_FAT_CACHE_SYNC_THREAD
    bool "Enable FAT Cache Sync Thread"
    default n
    depends on FS_FAT_CACHE
```

**说明**：
- 启用 FatFs 缓存同步线程
- 默认值：`n`（禁用）
- 依赖：`FS_FAT_CACHE`

**影响**：
- 异步刷新缓存，提高响应速度
- 增加系统复杂度

#### 5.4 FS_FAT_CHINESE

```kconfig
config FS_FAT_CHINESE
    bool "Enable Chinese"
    default y
    depends on FS_FAT
```

**说明**：
- 启用中文文件名支持
- 默认值：`y`（启用）
- 依赖：`FS_FAT`

**影响**：
- 在 `ffconf.h` 中设置 `FF_CODE_PAGE = 936`（GBK）
- 支持中文文件名和目录名

#### 5.5 FS_FAT_VIRTUAL_PARTITION

```kconfig
config FS_FAT_VIRTUAL_PARTITION
    bool "Enable Virtual Partition"
    default n
    depends on FS_FAT
```

**说明**：
- 启用虚拟分区功能
- 默认值：`n`（禁用）
- 依赖：`FS_FAT`

**影响**：
- 编译虚拟分区模块（`virpart/BUILD.gn`）
- 在 `ffconf.h` 中定义虚拟分区相关配置
- 在 `errcode_fat.h` 中定义虚拟分区错误码

#### 5.6 FS_FAT_VOLUMES

```kconfig
config FS_FAT_VOLUMES
    int
    depends on FS_FAT
    default 32 if PLATFORM_HI3731
    default 16
```

**说明**：
- 配置 FatFs 支持的卷数
- 默认值：
  - HI3731 平台：32
  - 其他平台：16
- 依赖：`FS_FAT`

**影响**：
- 在 `ffconf.h` 中设置 `FF_VOLUMES = LOSCFG_FS_FAT_VOLUMES`
- 支持同时挂载多个 FAT 卷

#### 5.7 FS_FAT_DISK

```kconfig
config FS_FAT_DISK
    bool "Enable partinfo for storage device"
    depends on FS_VFS && (FS_FAT || DRIVERS_MMC || DRIVERS_USB)
    default y
```

**说明**：
- 启用存储设备的分区信息功能
- 默认值：`y`（启用）
- 依赖：`FS_VFS` 且（`FS_FAT` 或 `DRIVERS_MMC` 或 `DRIVERS_USB`）

**影响**：
- 启用分区信息查询
- 支持多个存储设备（MMC、USB）

### Kconfig 配置表

| 配置项 | 类型 | 默认值 | 依赖 | 说明 |
|--------|------|--------|------|------|
| `FS_FAT` | bool | y | FS_VFS | 启用 FatFs 文件系统 |
| `FS_FAT_CACHE` | bool | y | FS_FAT | 启用 FatFs 缓存 |
| `FS_FAT_CACHE_SYNC_THREAD` | bool | n | FS_FAT_CACHE | 启用缓存同步线程 |
| `FS_FAT_CHINESE` | bool | y | FS_FAT | 启用中文支持 |
| `FS_FAT_VIRTUAL_PARTITION` | bool | n | FS_FAT | 启用虚拟分区 |
| `FS_FAT_VOLUMES` | int | 16/32 | FS_FAT | FAT 卷数配置 |
| `FS_FAT_DISK` | bool | y | FS_VFS && (...) | 启用分区信息 |

---

## 6. 条件编译

### LiteOS-M vs 标准系统

OH 的 FatFs 适配通过 `#ifndef __LITEOS_M__` 区分 LiteOS-M 和标准系统：

| 配置项 | LiteOS-M | 标准系统 |
|--------|----------|----------|
| 线程安全 | 禁用 (0) | 启用 (1) |
| 超时时间 | 1000ms | `LOS_WAIT_FOREVER` |
| 同步对象 | `HANDLE` | `LosMux` |
| 相对路径 | 启用 (1) | 禁用 (0) |
| Unicode | UTF-8 (2) | ANSI/OEM (0) |
| 卷字符串 ID | 启用 (2) | 禁用 (0) |
| TRIM | 启用 (1) | 禁用 (0) |
| 扇区大小 | `FS_MAX_SS` | 4096 |

### Kconfig 宏到 C 宏的映射

| Kconfig 宏 | C 宏 | 使用位置 |
|-----------|-------|---------|
| `FS_FAT` | `LOSCFG_FS_FAT` | BUILD.gn |
| `FS_FAT_CHINESE` | `LOSCFG_FS_FAT_CHINESE` | ffconf.h |
| `FS_FAT_VIRTUAL_PARTITION` | `LOSCFG_FS_FAT_VIRTUAL_PARTITION` | ffconf.h, errcode_fat.h |
| `FS_FAT_VOLUMES` | `LOSCFG_FS_FAT_VOLUMES` | ffconf.h |

---

## 7. 编译流程

### 7.1 FatFs 核心库编译

```
FatFs.gni
    ↓
kernel/liteos_a/fs/fat/BUILD.gn
    ↓
gn gen --root=...
    ↓
ninja
    ↓
libfat.a
```

**编译对象**：
- FatFs 核心文件：`ff.c`, `ffunicode.c`, `diskio.c`, `ffsystem.c`
- OH 适配层文件：`fatfs.c`, `format.c`, `fat_shellcmd.c`

**输出**：
- `libfat.a` - FatFs 静态库

### 7.2 虚拟分区模块编译

```
kernel/liteos_a/fs/fat/virpart/BUILD.gn
    ↓
gn gen --root=...
    ↓
ninja
    ↓
libvirpart.a
```

**编译对象**：
- 虚拟分区文件：`virpart.c`, `virpartff.c`

**输出**：
- `libvirpart.a` - 虚拟分区静态库

### 7.3 链接

```
libfat.a + libvirpart.a
    ↓
LiteOS Kernel
```

FatFs 库和虚拟分区库被链接到 LiteOS 内核镜像中。

---

## 8. 配置示例

### 示例 1：默认配置

```kconfig
# 默认配置
CONFIG_FS_FAT=y
CONFIG_FS_FAT_CACHE=y
CONFIG_FS_FAT_CACHE_SYNC_THREAD=n
CONFIG_FS_FAT_CHINESE=y
CONFIG_FS_FAT_VIRTUAL_PARTITION=n
CONFIG_FS_FAT_VOLUMES=16
CONFIG_FS_FAT_DISK=y
```

**结果**：
- 启用 FatFs
- 启用中文支持（GBK）
- 不启用虚拟分区
- 支持 16 个 FAT 卷

### 示例 2：启用虚拟分区

```kconfig
# 启用虚拟分区
CONFIG_FS_FAT=y
CONFIG_FS_FAT_CHINESE=y
CONFIG_FS_FAT_VIRTUAL_PARTITION=y
```

**结果**：
- 启用 FatFs
- 启用中文支持
- **启用虚拟分区**（编译 virpart 模块）
- 定义虚拟分区错误码（`errcode_fat.h`）

### 示例 3：LiteOS-M 配置

```kconfig
# LiteOS-M 配置（典型）
CONFIG_FS_FAT=y
CONFIG_FS_FAT_CHINESE=n
CONFIG_FS_FAT_VIRTUAL_PARTITION=n
CONFIG_FS_FAT_VOLUMES=4
```

**结果**：
- 启用 FatFs
- 不启用中文支持（节省资源）
- 不启用虚拟分区（LiteOS-M 资源受限）
- 支持 4 个 FAT 卷

### 示例 4：HI3731 平台配置

```kconfig
# HI3731 平台配置
CONFIG_FS_FAT=y
CONFIG_FS_FAT_CHINESE=y
CONFIG_FS_FAT_VOLUMES=32
```

**结果**：
- 启用 FatFs
- 启用中文支持
- **支持 32 个 FAT 卷**（HI3731 平台特性）

---

## 9. 构建调试

### 检查 FatFs 是否编译

```bash
# 查看 FatFs 模块是否被编译
ls -l out/.../obj/kernel/liteos_a/fs/fat/
```

**预期输出**：
```
obj/kernel/liteos_a/fs/fat/
├── fat_shellcmd.o
├── fatfs.o
├── format.o
├── diskio.o
├── ff.o
├── ffsystem.o
└── ffunicode.o
```

### 检查虚拟分区是否编译

```bash
# 查看虚拟分区模块是否被编译
ls -l out/.../obj/kernel/liteos_a/fs/fat/virpart/
```

**预期输出**（启用虚拟分区时）：
```
obj/kernel/liteos_a/fs/fat/virpart/
├── virpart.o
└── virpartff.o
```

### 检查配置是否生效

```bash
# 查看生成的配置头文件
grep -r "FF_CODE_PAGE\|FF_VOLUMES" out/.../gen/
```

**预期输出**：
```
#define FF_CODE_PAGE  936
#define FF_VOLUMES    16
```

---

## 10. 常见问题

### Q1: 如何启用中文文件名支持？

**A**: 在 Kconfig 中设置 `CONFIG_FS_FAT_CHINESE=y`。

或在 `.config` 文件中：
```
CONFIG_FS_FAT_CHINESE=y
```

### Q2: 如何启用虚拟分区？

**A**: 在 Kconfig 中设置 `CONFIG_FS_FAT_VIRTUAL_PARTITION=y`。

或在 `.config` 文件中：
```
CONFIG_FS_FAT_VIRTUAL_PARTITION=y
```

注意：虚拟分区会增加系统复杂度和内存占用。

### Q3: 如何修改支持的卷数？

**A**: 在 Kconfig 中设置 `CONFIG_FS_FAT_VOLUMES=N`。

或在 `.config` 文件中：
```
CONFIG_FS_FAT_VOLUMES=32
```

注意：卷数越多，内存占用越大。

### Q4: 如何禁用 FatFs？

**A**: 在 Kconfig 中设置 `CONFIG_FS_FAT=n`。

或在 `.config` 文件中：
```
CONFIG_FS_FAT=n
```

这会完全禁用 FatFs 模块，节省内存和代码大小。

### Q5: 如何检查当前配置？

**A**: 查看生成的配置头文件：

```bash
grep -r "FF_CODE_PAGE\|FF_VOLUMES\|FF_FS_REENTRANT" out/.../gen/kernel/liteos_a/fs/fat/
```

或查看 `.config` 文件：

```bash
grep "FS_FAT" .config
```

---

## 11. 总结

### OH 构建适配特点

1. **GNI 模块化**：通过 FatFs.gni 定义源文件和包含目录
2. **条件编译**：通过 Kconfig 控制功能开关
3. **模块化编译**：FatFs 核心库和虚拟分区模块独立编译
4. **公共配置**：通过 `public_configs` 导出头文件目录
5. **平台适配**：区分 LiteOS-M 和标准系统

### 关键文件总结

| 文件 | 作用 | 关键内容 |
|------|------|---------|
| **FatFs.gni** | 定义 FatFs 源文件 | `FATFS_SRC_FILES`, `FATFS_INCLUDE_DIRS` |
| **fat/BUILD.gn** | 编译 FatFs 核心库 | OH 适配层文件，导入 FatFs.gni |
| **virpart/BUILD.gn** | 编译虚拟分区模块 | 虚拟分区文件，条件编译 |
| **liteos_a/BUILD.gn** | 引用 FatFs 组件 | `$LITEOSTHIRDPARTY/FatFs` |
| **fat/Kconfig** | 定义配置选项 | 7 个配置项 |

---

## 参考资料

### 相关文档

- **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 项目评估结果
- **[01_Overview.md](01_Overview.md)** - 原始库简介
- **[02_Patches.md](02_Patches.md)** - Patch 详细分析
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用

### GN 构建系统

- **GN 官方文档**：https://gn.googlesource.com/gn/+/main/docs/reference.md
- **OpenHarmony 构建系统**：https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/subsystems/ohos-build-system.md

### Kconfig 配置系统

- **Kconfig 官方文档**：https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html
- **LiteOS Kconfig**：https://gitee.com/LiteOS/LiteOS/blob/master/doc/Kconfig-English.md

---

**文档版本**：1.0
**最后更新**：2026-02-08
