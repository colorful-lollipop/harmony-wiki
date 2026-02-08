# 原始库简介

> FatFs 原始库的简要介绍及其在 OpenHarmony 中的作用和定位

---

## FatFs 概述

FatFs 是一个通用的 FAT/exFAT 文件系统模块，专为小型嵌入式系统设计。它由 ChaN（日本嵌入式系统专家）开发和维护，是业界广泛使用的开源 FAT 文件系统实现。

### 核心特点

| 特性 | 说明 |
|------|------|
| **平台无关** | 使用 ANSI C (C89) 编写，与平台完全解耦 |
| **资源占用小** | 适合资源受限的微控制器（8051, PIC, AVR, ARM 等） |
| **模块化设计** | FAT 核心与磁盘 I/O 层分离，易于移植 |
| **高度可配置** | 通过配置文件可以裁剪功能和调整参数 |
| **完整支持** | 支持 FAT12、FAT16、FAT32、exFAT |

### 许可证

**BSD 3-Clause License** - 宽松的开源许可证，允许商业使用和修改。

---

## 功能列表

FatFs 提供了完整的 FAT 文件系统功能：

### 基础文件操作

- 文件创建、打开、读取、写入、关闭
- 文件属性修改（读/写/隐藏等）
- 文件移动、重命名、删除

### 目录操作

- 目录创建、删除、遍历
- 相对路径支持（可选）

### 文件系统操作

- 格式化（mkfs）
- 挂载/卸载
- 卷信息查询
- 空间查询（f_getfree）

### 高级功能

- 长文件名支持（LFN）
- Unicode 文件名（UTF-8/UTF-16/UTF-32）
- 多卷支持（最多 10 个）
- 文件锁（防止并发冲突）
- 相对路径（可选）

---

## FatFs 在 OpenHarmony 中的定位

### 主要用途

在 OpenHarmony 系统中，FatFs 主要用于**可移动存储设备**的 FAT 文件系统支持：

| 存储设备 | 文件系统 | 用途 |
|----------|---------|------|
| **SD 卡** | FAT32/exFAT | 相机、手机、开发板的数据存储 |
| **USB 存储** | FAT32/exFAT | U 盘、移动硬盘的数据交换 |
| **NAND Flash** | FAT32 | 某些嵌入式设备的用户数据存储 |
| **eMMC** | FAT32 | 特定场景的兼容性分区 |

### 系统架构定位

FatFs 在 OpenHarmony 文件系统层次中的位置：

```
┌─────────────────────────────────────┐
│       应用层                      │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────────┐
│    VFS (Virtual File System)      │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────────┐
│     文件系统抽象层                 │
│  ┌──────────┬──────────┐        │
│  │ FatFs    │ ext4     │        │
│  │ (可移动)  │ (内置)    │        │
│  └──────────┴──────────┘        │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────────┐
│     磁盘驱动层                    │
│  ┌────┬────┬────┬────┐        │
│  │SD  │USB │NAND│...│        │
│  └────┴────┴────┴────┘        │
└─────────────────────────────────────┘
```

**FatFs vs ext4**：

| 特性 | FatFs | ext4 |
|------|-------|------|
| **使用场景** | 可移动存储（SD、USB） | 系统内置存储 |
| **兼容性** | 跨平台兼容性好 | 主要用于 Linux |
| **功能复杂度** | 相对简单 | 功能丰富 |
| **日志/恢复** | 不支持 | 支持 |
| **访问控制** | 基本 | 完整的权限系统 |

---

## OpenHarmony 的集成方式

### 非侵入式适配策略

OpenHarmony 采用**非侵入式**的方式集成 FatFs，不修改 FatFs 核心代码：

| 适配方式 | 说明 | 优势 |
|----------|------|------|
| **配置文件修改** | 修改 `ffconf.h` 添加 OH 配置 | 易于上游版本升级 |
| **新增头文件** | 添加 `errcode_fat.h` 等 OH 特有文件 | 职责分离 |
| **独立适配层** | 在内核中创建独立的 VFS 适配层 | 便于维护 |
| **GNI 构建集成** | 通过 GNI 文件控制编译 | 灵活的构建配置 |

### 主要适配文件

#### 1. FatFs 核心库（未修改）

```
third_party/FatFs/source/
├── ff.c           ← FAT 核心实现（未修改）
├── ffunicode.c    ← Unicode 支持（未修改）
├── diskio.c       ← 磁盘 I/O 示例（未修改）
├── ffsystem.c     ← 系统相关函数（未修改）
├── ffconf.h       ← ⚠️ 被修改：OH 配置
├── diskio.h       ← ⚠️ 被修改：新增接口
└── errcode_fat.h ← ✅ OH 新增：虚拟分区错误码
```

#### 2. OH 适配层（新增）

```
kernel/liteos_a/fs/fat/
├── os_adapt/
│   ├── fatfs.c        ← VFS 适配层（61KB）
│   ├── fatfs.h        ← 适配层头文件
│   ├── format.c       ← FAT 格式化工具
│   └── fat_shellcmd.c ← Shell 命令
├── virpart/
│   ├── src/
│   │   ├── virpart.c   ← 虚拟分区核心
│   │   └── virpartff.c ← 虚拟分区 FatFs 集成
│   └── include/
│       ├── virpart.h   ← 虚拟分区头文件
│       └── virpartff.h ← 虚拟分区接口
├── BUILD.gn
├── Kconfig
└── Makefile
```

---

## OpenHarmony 的定制化内容

### 1. 虚拟分区功能

**说明**：OH 独有的功能，允许在单一 FAT 卷上创建多个逻辑分区。

**价值**：
- 在不支持多分区的存储设备上实现逻辑隔离
- 为不同用户或应用提供独立的目录空间
- 提供基础的访问控制机制

**实现**：
- 虚拟分区核心：`kernel/liteos_a/fs/fat/virpart/`
- 错误码定义：`source/errcode_fat.h`
- 配置开关：`LOSCFG_FS_FAT_VIRTUAL_PARTITION`

### 2. 中文文件名支持

**说明**：默认启用中文文件名支持。

**配置**：
```c
// source/ffconf.h
#ifdef LOSCFG_FS_FAT_CHINESE
#define FF_CODE_PAGE  936  // GBK
#else
#define FF_CODE_PAGE  437  // US
#endif
```

**价值**：
- 支持中文文件名
- 符合中国市场用户习惯
- 可通过 Kconfig 关闭（`FS_FAT_CHINESE`）

### 3. 增强的磁盘 I/O

**说明**：新增优化的磁盘 I/O 接口。

**新增接口**（`source/diskio.h`）：
```c
// 目录读取优化（非 LiteOS-M）
DRESULT disk_read_readdir(BYTE pdrv, BYTE* buff, LBA_t sector, UINT count);

// 原始扇区访问（非 LiteOS-M）
DRESULT disk_raw_read(int id, void* buff, LBA_t sector, UINT32 count);
DRESULT disk_raw_write(int id, void* buff, LBA_t sector, UINT32 count);
```

**价值**：
- 优化目录读取性能
- 提供底层扇区访问能力
- 支持特殊存储设备的优化

### 4. LiteOS 线程安全

**说明**：FatFs 的线程安全机制与 LiteOS 互斥锁集成。

**配置**：
```c
// source/ffconf.h
#ifndef __LITEOS_M__
#define FF_FS_REENTRANT   1
#define FF_FS_TIMEOUT     LOS_WAIT_FOREVER
#define FF_SYNC_t         LosMux
#else
#define FF_FS_REENTRANT   0
#define FF_FS_TIMEOUT     1000
#define FF_SYNC_t         HANDLE
#endif
```

**价值**：
- 确保多线程安全访问文件系统
- 与 LiteOS 调度机制深度集成
- 提供可配置的超时策略

### 5. 设备名称标准化

**说明**：统一不同类型存储设备的命名规范。

**配置**：
```c
// source/ffconf.h
#ifndef __LITEOS_M__
#define MMC0_DEVNAME  "mmcblk0"
#define MMC1_DEVNAME  "mmcblk1"
#define USB_DEVNAME   "sda"
#endif
```

**价值**：
- 统一设备命名
- 简化设备识别
- 提高跨平台一致性

---

## 技术规格

### 版本信息

| 项目 | 值 |
|------|-----|
| **FatFs 原始版本** | R0.15a |
| **OH 组件名称** | @ohos/FatFs |
| **OH 组件版本** | 3.1 |
| **许可证** | BSD 3-Clause License |
| **上游地址** | http://elm-chan.org/fsw/ff/00index_e.html |

### 资源占用

| 资源 | 占用 |
|------|------|
| **ROM** | 336KB |
| **RAM** | 672KB |

### 支持的文件系统

- ✅ FAT12（小型存储设备）
- ✅ FAT16（中等容量存储）
- ✅ FAT32（大容量存储）
- ✅ exFAT（超大容量存储）

### 支持的字符编码

- ✅ ANSI/OEM（代码页 437 等）
- ✅ UTF-8（代码页 0）
- ✅ UTF-16LE/BE
- ✅ UTF-32
- ✅ GBK（OH 默认，代码页 936）

---

## 适用系统类型

OpenHarmony 的 FatFs 组件适配以下系统类型：

| 系统类型 | 支持情况 | 说明 |
|---------|---------|------|
| **mini 系统** | ✅ 支持 | 资源受限系统，完整功能 |
| **small 系统** | ✅ 支持 | 标准系统，完整功能 |
| **标准系统** | ✅ 支持 | 完整系统，完整功能 |

---

## 与其他文件系统的关系

### FatFs 在 OH 文件系统生态中的位置

```
OpenHarmony 文件系统
    ├─ VFS (Virtual File System)
    │   ├─ FatFs ← 本文档关注
    │   ├─ ext4
    │   ├─ NFS
    │   └─ ...
    └─ 存储管理
        ├─ 分区管理
        ├─ 卷管理
        └─ ...
```

### FatFs vs 其他文件系统

| 特性 | FatFs | ext4 | NFS |
|------|-------|------|-----|
| **用途** | 可移动存储 | 系统存储 | 网络存储 |
| **兼容性** | 跨平台 | Linux | 跨平台 |
| **性能** | 中等 | 高 | 中等 |
| **功能** | 基础 | 丰富 | 中等 |
| **日志/恢复** | 不支持 | 支持 | 不支持 |
| **访问控制** | 基本 | 完整 | 依赖 NFS |

---

## 参考资料

### 官方文档

- **FatFs 官方网站**：http://elm-chan.org/fsw/ff/00index_e.html
- **FatFs API 文档**：http://elm-chan.org/fsw/ff/00index_e.html
- **FatFs 源码仓库**：http://elm-chan.org/fsw/ff/00index_e.html

### OH 相关文档

- **OH 文档中心**：https://gitee.com/openharmony/docs
- **LiteOS-A 文档**：https://gitee.com/openharmony/kernel_liteos_a
- **FatFs 组件**：`third_party/FatFs/`

### 技术标准

- **FAT 文件系统规范**：Microsoft FAT Specification
- **exFAT 文件系统规范**：Microsoft exFAT Specification
- **FAT32 规范**：Microsoft FAT32 Specification

---

## 后续阅读

- **[02_Patches.md](02_Patches.md)** - OH 的配置文件修改和新增文件详细分析
- **[03_Build_Integration.md](03_Build_Integration.md)** - OH 的构建系统和配置方式
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - FatFs 在 OH 中的使用场景和架构
- **[05_API_Differences.md](05_API_Differences.md)** - OH 新增的 API 和接口差异

---

**文档版本**：1.0
**最后更新**：2026-02-08
