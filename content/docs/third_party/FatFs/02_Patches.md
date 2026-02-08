# Patch 详细分析

> OH 对 FatFs 的配置修改和新增文件分析

---

## Patch 概述

**重要声明**：OpenHarmony **没有使用传统的 `.patch` 文件**来修改 FatFs 源代码。

OH 采用**非侵入式适配策略**，通过以下方式实现集成：

| 适配方式 | 说明 | 优势 |
|----------|------|------|
| **配置文件修改** | 修改 `ffconf.h` 添加 OH 配置 | 易于上游版本升级 |
| **新增头文件** | 添加 `errcode_fat.h` 等 OH 特有文件 | 不修改原始代码 |
| **独立适配层** | 在内核中创建独立的 VFS 适配层 | 职责分离，易于维护 |
| **GNI 构建集成** | 通过 GNI 文件控制编译 | 灵活的构建配置 |

### 未修改的文件（原始代码）

以下 FatFs 核心文件**未被修改**：

```
third_party/FatFs/source/
├── ff.c           ← 未修改（FAT 核心实现）
├── ffunicode.c    ← 未修改（Unicode 支持）
├── diskio.c       ← 未修改（磁盘 I/O 示例）
├── ffsystem.c     ← 未修改（系统相关函数）
├── ff.h           ← 未修改（公共头文件）
└── integer.h      ← 未修改（整数类型）
```

### 被修改的文件（OH 适配）

以下文件被 OH 修改或新增：

```
third_party/FatFs/source/
├── ffconf.h       ← 被修改：OH 配置
├── diskio.h       ← 被修改：新增接口
└── errcode_fat.h ← OH 新增：虚拟分区错误码
```

---

## 1. ffconf.h 配置详解

### 文件信息

- **路径**：`third_party/FatFs/source/ffconf.h`
- **大小**：12.9KB
- **状态**：被 OH 大量修改
- **修改类型**：条件编译 + 新增配置

### OH 配置宏

#### 1.1 LiteOS 系统头文件引用

**原始代码**：无

**OH 修改**：
```c
// ffconf.h:8-14
#ifndef __LITEOS_M__
#include "los_mux.h"
#include "los_config.h"
#else
#include "fs_config.h"
#include "vfs_config.h"
#endif
```

**目的**：区分 LiteOS-M 和标准系统，引用不同的系统头文件。

**影响**：
- 标准系统使用 LiteOS 互斥锁（`LosMux`）
- LiteOS-M 使用自定义配置

#### 1.2 中文文件名支持

**原始代码**：`FF_CODE_PAGE` 默认为 437（US）

**OH 修改**：
```c
// ffconf.h:91-95
#ifdef LOSCFG_FS_FAT_CHINESE
#define FF_CODE_PAGE  936
#else
#define FF_CODE_PAGE  437
#endif
```

**说明**：
- **437** - US (OEM) 代码页
- **936** - 简体中文 (GBK) 代码页

**OH 需求**：
- 支持 GBK 编码的中文文件名
- 符合中国市场用户习惯

**Kconfig 配置**：
```kconfig
// kernel/liteos_a/fs/fat/Kconfig:22-27
config FS_FAT_CHINESE
    bool "Enable Chinese"
    default y
    depends on FS_FAT
    help
      Answer Y to enable LiteOS fat filesystem support Chinese.
```

#### 1.3 设备名称定义

**原始代码**：无设备名称定义

**OH 修改**：
```c
// ffconf.h:124-133
#ifndef __LITEOS_M__
#define MMC0_DEVNAME  "mmcblk0"
#define MMC1_DEVNAME  "mmcblk1"
#define USB_DEVNAME   "sda"
#define MMC_LENGTH    7
enum STORAGE {
    MMC0,
    MMC1,
};
#endif
```

**OH 需求**：
- 统一不同类型存储设备的命名规范
- 简化设备识别和挂载

**设备说明**：
- `mmcblk0` - 第一个 MMC/SD 设备
- `mmcblk1` - 第二个 MMC/SD 设备
- `sda` - 第一个 USB 存储设备

#### 1.4 卷数配置

**原始代码**：`FF_VOLUMES` 为固定值

**OH 修改**：
```c
// ffconf.h:207-211
#ifndef __LITEOS_M__
#define FF_VOLUMES  LOSCFG_FS_FAT_VOLUMES
#else
#define FF_VOLUMES  4
#endif
```

**说明**：标准系统的卷数可配置，LiteOS-M 固定为 4。

**Kconfig 配置**：
```kconfig
// kernel/liteos_a/fs/fat/Kconfig:34-38
config FS_FAT_VOLUMES
    int
    depends on FS_FAT
    default 32 if PLATFORM_HI3731
    default 16
```

**OH 需求**：
- 支持多卷（SD 卡 + USB + ...）
- 根据平台配置不同卷数（HI3731 平台支持 32 个卷）

#### 1.5 虚拟分区支持

**原始代码**：无虚拟分区相关配置

**OH 修改**：
```c
// ffconf.h:214-218
#ifdef LOSCFG_FS_FAT_VIRTUAL_PARTITION
#define _DEFAULT_VIRVOLUEMS  4
#define _MIN_CLST              0x4000
#define _FLOAT_ACC            0.00000001
#endif
```

**说明**：定义虚拟分区的默认参数。

**OH 需求**：
- 在单一 FAT 卷上创建多个逻辑分区
- 提供基础的访问控制机制

**Kconfig 配置**：
```kconfig
// kernel/liteos_a/fs/fat/Kconfig:29-32
config FS_FAT_VIRTUAL_PARTITION
    bool "Enable Virtual Partition"
    default n
    depends on FS_FAT
```

#### 1.6 线程安全配置

**原始代码**：线程安全由用户自定义

**OH 修改**：
```c
// ffconf.h:324-332
#ifndef __LITEOS_M__
#define FF_FS_REENTRANT  1
#define FF_FS_TIMEOUT    LOS_WAIT_FOREVER
#define FF_SYNC_t        LosMux
#else
#define FF_FS_REENTRANT  0
#define FF_FS_TIMEOUT    1000
#define FF_SYNC_t        HANDLE
#endif
```

**说明**：
- **标准系统**：启用线程安全，使用 `LosMux`，超时为 `LOS_WAIT_FOREVER`（无限等待）
- **LiteOS-M**：禁用线程安全，使用 `HANDLE`，超时为 1000ms

**OH 需求**：
- 确保多线程安全访问文件系统
- 与 LiteOS 调度机制深度集成

#### 1.7 扇区大小支持

**原始代码**：`FF_MAX_SS` 默认为 512

**OH 修改**：
```c
// ffconf.h:247-252
#ifndef __LITEOS_M__
#define FF_MAX_SS  4096
#else
#define FF_MAX_SS  FS_MAX_SS
#endif
```

**说明**：标准系统支持 4096 字节扇区，LiteOS-M 使用配置。

**OH 需求**：
- 支持大扇区设备（如某些 SSD）
- 提高大容量设备的访问效率

#### 1.8 TRIM 支持

**原始代码**：`FF_USE_TRIM` 为可选配置

**OH 修改**：
```c
// ffconf.h:261-265
#ifndef __LITEOS_M__
#define FF_USE_TRIM  0
#else
#define FF_USE_TRIM  1
#endif
```

**说明**：
- **标准系统**：禁用 TRIM
- **LiteOS-M**：启用 TRIM（Flash 优化）

**OH 需求**：
- LiteOS-M 通常使用 Flash 存储，TRIM 可以延长 Flash 寿命
- 标准系统通常使用 SD 卡或 USB，TRIM 支持有限

#### 1.9 文件锁数量

**原始代码**：`FF_FS_LOCK` 为固定值或用户配置

**OH 修改**：
```c
// ffconf.h:308-312
#ifndef __LITEOS_M__
#define FF_FS_LOCK  CONFIG_NFILE_DESCRIPTORS
#else
#define FF_FS_LOCK  (CONFIG_NFILE_DESCRIPTORS + LOSCFG_MAX_OPEN_DIRS - MIN_START_FD)
#endif
```

**说明**：文件锁数量基于系统配置。

**OH 需求**：
- 根据系统资源动态配置文件锁数量
- 支持同时打开多个文件和目录

#### 1.10 相对路径支持

**原始代码**：`FF_FS_RPATH` 默认为 0

**OH 修改**：
```c
// ffconf.h:190-194
#ifndef __LITEOS_M__
#define FF_FS_RPATH  0
#else
#define FF_FS_RPATH  1
#endif
```

**说明**：
- **标准系统**：禁用相对路径
- **LiteOS-M**：启用相对路径

**OH 需求**：
- LiteOS-M 资源受限，相对路径简化了路径解析

#### 1.11 其他条件编译

OH 还通过 `#ifndef __LITEOS_M__` 修改了以下配置：

| 配置项 | 标准系统 | LiteOS-M | OH 需求 |
|--------|----------|----------|---------|
| `FF_USE_EXPAND` | 1 | 0 | 文件快速分配（标准系统） |
| `FF_USE_CHMOD` | 1 | 0 | 文件属性修改（标准系统） |
| `FF_USE_LABEL` | 1 | 0 | 卷标签功能（标准系统） |
| `FF_LFN_UNICODE` | 0 | 2 | Unicode 支持（LiteOS-M 使用 UTF-8） |
| `FF_STR_VOLUME_ID` | 0 | 2 | 卷字符串 ID（LiteOS-M） |

### ffconf.h 配置对比表

| 配置项 | 标准 FatFs | OH 适配（标准系统） | OH 适配（LiteOS-M） | OH 需求 |
|--------|-----------|-------------------|-------------------|---------|
| `FF_CODE_PAGE` | 437 | 936 | 437 | 中文支持 |
| `FF_VOLUMES` | 固定 | `LOSCFG_FS_FAT_VOLUMES` | 4 | 多卷支持 |
| `FF_FS_REENTRANT` | 用户自定义 | 1 | 0 | 线程安全 |
| `FF_FS_TIMEOUT` | 1000ms | `LOS_WAIT_FOREVER` | 1000ms | 等待策略 |
| `FF_SYNC_t` | 用户自定义 | `LosMux` | `HANDLE` | LiteOS 集成 |
| `FF_MAX_SS` | 512 | 4096 | `FS_MAX_SS` | 大扇区支持 |
| `FF_USE_TRIM` | 可选 | 0 | 1 | Flash 优化 |
| `FF_FS_LOCK` | 用户自定义 | `CONFIG_NFILE_DESCRIPTORS` | 动态计算 | 资源适配 |
| `FF_FS_RPATH` | 0 | 0 | 1 | 路径解析 |
| `FF_LFN_UNICODE` | 用户自定义 | 0 | 2 (UTF-8) | Unicode 支持 |
| `FF_STR_VOLUME_ID` | 用户自定义 | 0 | 2 | 卷字符串 ID |

---

## 2. diskio.h 新增接口分析

### 文件信息

- **路径**：`third_party/FatFs/source/diskio.h`
- **大小**：3.4KB
- **状态**：被 OH 修改（新增接口）
- **修改类型**：新增函数声明

### 新增接口

#### 2.1 disk_read_readdir()

**原型**：
```c
// diskio.h:43
DRESULT disk_read_readdir(BYTE pdrv, BYTE* buff, LBA_t sector, UINT count);
```

**说明**：目录读取专用接口。

**OH 需求**：
- 优化目录读取性能
- 减少重复的磁盘 I/O 操作

**条件编译**：
```c
// diskio.h:42-43
#ifndef __LITEOS_M__
DRESULT disk_read_readdir(BYTE pdrv, BYTE* buff, LBA_t sector, UINT count);
#endif
```

**可用性**：仅标准系统可用，LiteOS-M 不可用。

#### 2.2 disk_raw_read()

**原型**：
```c
// diskio.h:44
DRESULT disk_raw_read(int id, void* buff, LBA_t sector, UINT32 count);
```

**说明**：原始扇区读取接口。

**OH 需求**：
- 提供底层扇区访问能力
- 支持特殊存储设备的优化
- 绕过 FatFs 缓存直接读取原始数据

**条件编译**：
```c
// diskio.h:42-44
#ifndef __LITEOS_M__
DRESULT disk_read_readdir(BYTE pdrv, BYTE* buff, LBA_t sector, UINT count);
DRESULT disk_raw_read(int id, void* buff, LBA_t sector, UINT32 count);
#endif
```

**可用性**：仅标准系统可用，LiteOS-M 不可用。

#### 2.3 disk_raw_write()

**原型**：
```c
// diskio.h:45
DRESULT disk_raw_write(int id, void* buff, LBA_t sector, UINT32 count);
```

**说明**：原始扇区写入接口。

**OH 需求**：
- 提供底层扇区写入能力
- 支持特殊存储设备的优化
- 绕过 FatFs 缓存直接写入原始数据

**条件编译**：
```c
// diskio.h:42-45
#ifndef __LITEOS_M__
DRESULT disk_read_readdir(BYTE pdrv, BYTE* buff, LBA_t sector, UINT count);
DRESULT disk_raw_read(int id, void* buff, LBA_t sector, UINT32 count);
DRESULT disk_raw_write(int id, void* buff, LBA_t sector, UINT32 count);
#endif
```

**可用性**：仅标准系统可用，LiteOS-M 不可用。

### diskio.h 新增接口对比表

| 接口 | 参数 | 用途 | 可用性 |
|------|------|------|--------|
| `disk_read_readdir()` | `pdrv, buff, sector, count` | 目录读取优化 | 标准系统 |
| `disk_raw_read()` | `id, buff, sector, count` | 原始扇区读取 | 标准系统 |
| `disk_raw_write()` | `id, buff, sector, count` | 原始扇区写入 | 标准系统 |

---

## 3. errcode_fat.h 新增文件分析

### 文件信息

- **路径**：`third_party/FatFs/source/errcode_fat.h`
- **大小**：1.6KB
- **状态**：OH 新增文件
- **用途**：定义虚拟分区相关的错误码

### 版权信息

```c
// errcode_fat.h:1-29
/*
 * Copyright (c) 2013-2019, Huawei Technologies Co., Ltd. All rights reserved.
 * Copyright (c) 2020, Huawei Device Co., Ltd. All rights reserved.
 * ...
 */
```

### 错误码定义

#### 3.1 错误码范围

```c
// errcode_fat.h:40-41
#ifdef LOSCFG_FS_FAT_VIRTUAL_PARTITION
#define VIRERR_BASE    0x10000000
```

**说明**：虚拟分区错误码基地址为 `0x10000000`，与标准 FatFs 错误码区分。

#### 3.2 错误码列表

| 错误码 | 值（十六进制） | 说明 |
|--------|----------------|------|
| `VIRERR_OK` | 0x00000000 | 操作成功 |
| `VIRERR_BASE` | 0x10000000 | 虚拟分区错误码基地址 |
| `VIRERR_MODIFIED` | 0x10000001 | 分区已被修改 |
| `VIRERR_CHAIN_ERR` | 0x10000002 | 链表错误 |
| `VIRERR_OCCUPIED` | 0x10000003 | 分区已被占用 |
| `VIRERR_NOTCLEAR` | 0x10000004 | 分区未清理 |
| `VIRERR_NOTFIT` | 0x10000005 | 空间不足 |
| `VIRERR_NOTMOUNT` | 0x10000006 | 分区未挂载 |
| `VIRERR_INTER_ERR` | 0x10000007 | 内部错误 |
| `VIRERR_NOPARAM` | 0x10000008 | 参数错误 |
| `VIRERR_PARMLOCKED` | 0x10000009 | 参数已锁定 |
| `VIRERR_PARMNUMERR` | 0x1000000A | 参数数量错误 |
| `VIRERR_PARMPERCENTERR` | 0x1000000B | 百分比参数错误 |
| `VIRERR_PARMNAMEERR` | 0x1000000C | 参数名称错误 |
| `VIRERR_PARMDEVERR` | 0x1000000D | 设备参数错误 |

#### 3.3 错误码使用示例

```c
// kernel/liteos_a/fs/fat/os_adapt/fatfs.c
#ifdef LOSCFG_FS_FAT_VIRTUAL_PARTITION
int fatfs_2_vfs(int result)
{
    if (result < 0 || result >= VIRERR_BASE) {
        return result;  // 虚拟分区错误码直接返回
    }
    // ... FatFs 错误码转换
}
#endif
```

### 条件编译

```c
// errcode_fat.h:40-56
#ifdef LOSCFG_FS_FAT_VIRTUAL_PARTITION
#define VIRERR_OK          0x00000000
#define VIRERR_BASE         0x10000000
#define VIRERR_MODIFIED     0x10000001
// ... 其他错误码
#endif
#endif
```

**说明**：只有启用 `LOSCFG_FS_FAT_VIRTUAL_PARTITION` 时才定义错误码。

---

## 4. OH 特有功能文件分析

### 4.1 虚拟分区功能

#### 实现文件

```
kernel/liteos_a/fs/fat/virpart/
├── BUILD.gn
├── Makefile
├── include/
│   ├── virpart.h
│   └── virpartff.h
└── src/
    ├── virpart.c
    └── virpartff.c
```

#### 文件说明

| 文件 | 大小 | 说明 |
|------|------|------|
| `virpart.c` | - | 虚拟分区核心实现 |
| `virpartff.c` | - | 虚拟分区与 FatFs 集成 |
| `virpart.h` | 2.7KB | 虚拟分区头文件 |
| `virpartff.h` | 2.4KB | 虚拟分区 FatFs 接口 |

#### 构建配置

```gn
// kernel/liteos_a/fs/fat/virpart/BUILD.gn:32-47
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
```

**说明**：只有启用 `LOSCFG_FS_FAT_VIRTUAL_PARTITION` 时才编译虚拟分区模块。

### 4.2 VFS 适配层

#### 实现文件

```
kernel/liteos_a/fs/fat/os_adapt/
├── fatfs.c         (61KB) - VFS 适配层核心
├── fatfs.h         - 适配层头文件
├── format.c        (3.8KB) - FAT 格式化工具
└── fat_shellcmd.c  (3.4KB) - Shell 命令
```

#### 文件说明

| 文件 | 大小 | 说明 |
|------|------|------|
| `fatfs.c` | 61KB | **核心**：VFS 适配和 FatFs 封装 |
| `fatfs.h` | - | 适配层头文件 |
| `format.c` | 3.8KB | FAT 格式化工具 |
| `fat_shellcmd.c` | 3.4KB | Shell 命令（mkfs, fat 等） |

#### fatfs.c 核心功能

**文件操作**：
- `fatfs_open()` - 打开文件
- `fatfs_read()` - 读取文件
- `fatfs_write()` - 写入文件
- `fatfs_close()` - 关闭文件

**目录操作**：
- `fatfs_opendir()` - 打开目录
- `fatfs_readdir()` - 读取目录
- `fatfs_closedir()` - 关闭目录

**文件系统操作**：
- `fatfs_mount()` - 挂载文件系统
- `fatfs_unmount()` - 卸载文件系统
- `fatfs_statfs()` - 查询文件系统状态

**错误码转换**：
```c
// kernel/liteos_a/fs/fat/os_adapt/fatfs.c
int fatfs_2_vfs(int result)
{
    int status = ENOERR;
    switch (result) {
        case FR_OK:
            break;
        case FR_NO_FILE:
        case FR_NO_PATH:
            status = ENOENT;
            break;
        // ... 其他错误码转换
    }
    return status;
}
```

---

## 5. 升级建议

### 5.1 上游版本升级步骤

由于 OH 采用非侵入式适配，升级相对简单：

1. **替换 FatFs 核心文件**
   - 下载新版本 FatFs 源码
   - 替换以下文件：
     - `source/ff.c`
     - `source/ffunicode.c`
     - `source/diskio.c`
     - `source/ffsystem.c`
     - `source/ff.h`
     - `source/integer.h`

2. **重新应用 OH 配置到 ffconf.h**
   - 查看新版本的 `ffconf.h`
   - 重新应用以下 OH 配置：
     - LiteOS 系统头文件引用（`#ifndef __LITEOS_M__`）
     - 中文支持（`FF_CODE_PAGE = 936`）
     - 设备名称定义（`MMC0_DEVNAME` 等）
     - 卷数配置（`FF_VOLUMES = LOSCFG_FS_FAT_VOLUMES`）
     - 虚拟分区支持（`#ifdef LOSCFG_FS_FAT_VIRTUAL_PARTITION`）
     - 线程安全配置（`FF_FS_REENTRANT = 1`, `FF_SYNC_t = LosMux`）
     - 其他条件编译分支

3. **验证新增接口兼容性**
   - 确保 `disk_read_readdir()`, `disk_raw_read()`, `disk_raw_write()` 仍然可用
   - 如果新版本不支持，需要在 `diskio.c` 中实现

4. **测试 OH 特有功能**
   - 测试中文文件名支持
   - 测试虚拟分区功能（如果启用）
   - 测试 LiteOS 线程安全

5. **构建和测试**
   - 完整编译 OH 系统
   - 运行 FatFs 相关测试用例
   - 验证所有功能正常

### 5.2 配置变更追踪

#### 需要重新应用的配置项

| 配置项 | ffconf.h 位置 | 说明 |
|--------|--------------|------|
| LiteOS 头文件 | 8-14 | `#include "los_mux.h"` 等 |
| 中文支持 | 91-95 | `FF_CODE_PAGE = 936` |
| 设备名称 | 124-133 | `MMC0_DEVNAME` 等 |
| 卷数配置 | 207-211 | `FF_VOLUMES = LOSCFG_FS_FAT_VOLUMES` |
| 虚拟分区 | 214-218 | `#ifdef LOSCFG_FS_FAT_VIRTUAL_PARTITION` |
| 线程安全 | 324-332 | `FF_FS_REENTRANT = 1`, `FF_SYNC_t = LosMux` |
| 扇区大小 | 247-252 | `FF_MAX_SS = 4096` |
| TRIM 支持 | 261-265 | LiteOS-M 启用 TRIM |
| 文件锁 | 308-312 | `FF_FS_LOCK = CONFIG_NFILE_DESCRIPTORS` |
| 相对路径 | 190-194 | LiteOS-M 启用相对路径 |

### 5.3 风险与注意事项

#### 风险 1：API 变更

**问题**：新版本 FatFs 可能修改 API

**应对**：
- 仔细阅读新版本的 `ff.h` 和 API 文档
- 更新 VFS 适配层（`fatfs.c`）以匹配新 API
- 运行完整的测试套件

#### 风险 2：配置项变更

**问题**：新版本可能移除或重命名配置项

**应对**：
- 对比新旧版本的 `ffconf.h`
- 更新 OH 的配置应用逻辑
- 更新 Kconfig 配置项

#### 风险 3：虚拟分区不兼容

**问题**：虚拟分区功能依赖特定版本的 FatFs

**应对**：
- 测试虚拟分区功能是否正常
- 如有必要，更新 `virpartff.c` 以适配新版本
- 考虑将虚拟分区功能独立于 FatFs 版本

#### 风险 4：性能退化

**问题**：新版本可能引入性能问题

**应对**：
- 运行性能测试
- 对比旧版本和新版本的性能指标
- 调整配置选项以优化性能

### 5.4 升级检查清单

升级前检查：

- [ ] 备份当前版本的 FatFs 和 OH 适配层
- [ ] 阅读新版本的 CHANGELOG
- [ ] 查看新版本的 API 文档
- [ ] 确认 OH 配置项在新版本中仍然有效
- [ ] 准备测试用例

升级后验证：

- [ ] 编译成功，无警告
- [ ] 所有单元测试通过
- [ ] 功能测试通过（中文文件名、虚拟分区等）
- [ ] 性能测试通过（无明显退化）
- [ ] 兼容性测试通过（现有应用正常）

---

## 6. 总结

### OH 适配策略总结

OpenHarmony 采用**非侵入式适配策略**集成 FatFs：

| 策略 | 说明 | 优势 |
|------|------|------|
| **配置文件修改** | 修改 `ffconf.h` | 易于上游版本升级 |
| **新增头文件** | 添加 `errcode_fat.h` | 不修改原始代码 |
| **独立适配层** | 创建 VFS 适配层 | 职责分离 |
| **GNI 构建集成** | 通过 GNI 控制编译 | 灵活配置 |

### 主要适配内容

1. **ffconf.h 配置修改**
   - LiteOS 系统头文件引用
   - 中文文件名支持（GBK）
   - 设备名称标准化
   - 线程安全集成（LosMux）
   - 虚拟分区支持
   - 扇区大小支持（4096）
   - TRIM 支持（LiteOS-M）

2. **diskio.h 新增接口**
   - `disk_read_readdir()` - 目录读取优化
   - `disk_raw_read()` - 原始扇区读取
   - `disk_raw_write()` - 原始扇区写入

3. **errcode_fat.h 新增文件**
   - 虚拟分区错误码（13 个）
   - 错误码基地址 `0x10000000`

4. **VFS 适配层**（独立于 FatFs）
   - `fatfs.c` - VFS 适配（61KB）
   - `virpart/` - 虚拟分区功能

### 证据链

所有适配修改都有明确的 OH 需求依据：

- ✅ 中文支持 → `LOSCFG_FS_FAT_CHINESE` Kconfig 选项
- ✅ 虚拟分区 → `LOSCFG_FS_FAT_VIRTUAL_PARTITION` Kconfig 选项
- ✅ LiteOS 集成 → `los_mux.h`, `LosMux` 等引用
- ✅ 多卷支持 → `LOSCFG_FS_FAT_VOLUMES` Kconfig 选项

---

## 参考资料

### 相关文档

- **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 项目评估结果
- **[01_Overview.md](01_Overview.md)** - 原始库简介
- **[03_Build_Integration.md](03_Build_Integration.md)** - OH 构建适配
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用
- **[05_API_Differences.md](05_API_Differences.md)** - API/接口差异

### 外部资源

- **FatFs 官方文档**：http://elm-chan.org/fsw/ff/00index_e.html
- **FatFs API 文档**：http://elm-chan.org/fsw/ff/00index_e.html
- **OpenHarmony 文档**：https://gitee.com/openharmony/docs

---

**文档版本**：1.0
**最后更新**：2026-02-08
