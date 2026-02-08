# FatFs OpenHarmony 适配文档

> FatFs 在 OpenHarmony 中的集成、适配与使用指南

**FatFs 版本**：R0.15a
**OpenHarmony 组件**：@ohos/FatFs v3.1
**文档更新时间**：2026-02-08

---

## 快速导航

### 新手入门

如果你是第一次了解 FatFs 在 OH 中的使用，建议按以下顺序阅读：

1. **[01_Overview.md](01_Overview.md)** - 了解 FatFs 及其在 OH 中的作用
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解 FatFs 在 OH 中的使用场景和架构
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 了解 OH 如何构建和配置 FatFs

### 深入研究

如果你需要详细了解 OH 的定制化内容：

4. **[02_Patches.md](02_Patches.md)** - OH 的配置文件修改和新增文件分析
5. **[05_API_Differences.md](05_API_Differences.md)** - OH 新增的 API 和接口差异
6. **[06_Security.md](06_Security.md)** - 安全性分析和风险评估

### 维护与升级

如果你需要升级 FatFs 版本或维护 OH 适配：

- 阅读 [02_Patches.md](02_Patches.md) 了解 OH 的适配策略
- 参考 [ASSESSMENT.md](wiki/_work/ASSESSMENT.md) 的升级建议
- 查看 [SUMMARY.md](SUMMARY.md) 获取完整的阅读路线

---

## 文档说明

本文档重点说明 OpenHarmony 对 FatFs 的适配和定制化内容，包括：

- ✅ OH 的 Patch 和配置修改
- ✅ OH 特有的功能扩展（虚拟分区、中文支持等）
- ✅ 构建系统适配（GNI、BUILD.gn、Kconfig）
- ✅ 在 OH 系统中的使用方式和依赖关系
- ⚠️ FatFs 的原始功能和 API（仅简要说明）

**不包含**：
- ❌ FatFs 的完整 API 文档（请参考 [官方文档](http://elm-chan.org/fsw/ff/00index_e.html)）
- ❌ FatFs 的使用教程和示例代码
- ❌ 磁盘驱动开发指南

---

## 项目概览

### FatFs 是什么？

FatFs 是一个通用的 FAT/exFAT 文件系统模块，专为小型嵌入式系统设计。它具有以下特点：

- **平台无关**：使用 ANSI C (C89) 编写，不依赖特定平台
- **资源占用小**：适合资源受限的微控制器
- **功能完整**：支持 FAT12、FAT16、FAT32、exFAT
- **可配置**：通过配置文件可以裁剪功能以满足不同需求

### FatFs 在 OpenHarmony 中的作用

在 OpenHarmony 中，FatFs 主要用于：

- **可移动存储设备**：SD 卡、USB 存储的 FAT 文件系统支持
- **NAND Flash**：某些 NAND Flash 存储的文件系统
- **用户数据存储**：作为可插拔存储的通用解决方案
- **虚拟分区**：支持在单一 FAT 卷上创建多个逻辑分区（OH 特有功能）

### OH 适配策略

OpenHarmony 采用**非侵入式适配**策略：

- ❌ **不使用 Patch 文件**：没有传统的 `.patch` 文件
- ✅ **配置文件适配**：修改 `ffconf.h` 添加 OH 特定配置
- ✅ **新增头文件**：添加 OH 特有头文件（如 `errcode_fat.h`）
- ✅ **独立适配层**：在内核中创建独立的 VFS 适配层
- ✅ **GNI 构建集成**：通过 GNI 文件控制编译

这种策略的优势是**易于上游版本升级**，因为 FatFs 核心代码未被修改。

---

## OH 适配亮点

### 1. 中文文件名支持

OH 默认启用中文文件名支持（代码页 936 - GBK），通过 `LOSCFG_FS_FAT_CHINESE` 配置项控制。

### 2. 虚拟分区功能

OH 独有的虚拟分区功能，允许在单一 FAT 卷上创建多个逻辑分区。相关文件：

- `kernel/liteos_a/fs/fat/virpart/` - 虚拟分区实现
- `source/errcode_fat.h` - 虚拟分区错误码

### 3. 增强的磁盘 I/O

新增优化的磁盘 I/O 接口：

- `disk_read_readdir()` - 目录读取优化
- `disk_raw_read()` / `disk_raw_write()` - 原始扇区访问

### 4. LiteOS 线程安全

FatFs 的线程安全机制与 LiteOS 互斥锁（`LosMux`）集成，确保多线程安全访问文件系统。

---

## 依赖关系

```
应用层
  ↓
LiteOS VFS
  ↓
FatFs 适配层 (kernel/liteos_a/fs/fat/)
  ├─ fatfs.c         - VFS 适配
  ├─ virpart/        - 虚拟分区
  ├─ format.c        - 格式化工具
  └─ fat_shellcmd.c - Shell 命令
  ↓
FatFs 核心库 (third_party/FatFs/source/)
  ├─ ff.c           - FAT 核心实现
  ├─ ffunicode.c    - Unicode 支持
  └─ diskio.c       - 磁盘 I/O
  ↓
底层磁盘驱动
```

详细依赖关系请参考 [04_Usage_in_OH.md](04_Usage_in_OH.md)。

---

## 构建配置

FatFs 的功能通过 LiteOS Kconfig 控制，主要配置项：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `FS_FAT` | y | 启用 FAT 文件系统 |
| `FS_FAT_CHINESE` | y | 启用中文支持 |
| `FS_FAT_VIRTUAL_PARTITION` | n | 启用虚拟分区 |
| `FS_FAT_VOLUMES` | 16 | FAT 卷数 |

详细配置说明请参考 [03_Build_Integration.md](03_Build_Integration.md)。

---

## 技术规格

| 项目 | 值 |
|------|-----|
| **原始版本** | FatFs R0.15a |
| **OH 组件版本** | @ohos/FatFs v3.1 |
| **许可证** | BSD 3-Clause License |
| **ROM 占用** | 336KB |
| **RAM 占用** | 672KB |
| **支持的文件系统** | FAT12, FAT16, FAT32, exFAT |
| **支持的字符编码** | ANSI/OEM, UTF-8, UTF-16, UTF-32 |
| **线程安全** | 是（LiteOS LosMux） |
| **代码页支持** | GBK (936), Unicode 等 |

---

## 相关链接

- **FatFs 官方文档**：http://elm-chan.org/fsw/ff/00index_e.html
- **OpenHarmony 文档**：https://gitee.com/openharmony/docs
- **FatFs 源码仓库**：third_party/FatFs/
- **OH 适配层源码**：kernel/liteos_a/fs/fat/

---

## 文档索引

- **[SUMMARY.md](SUMMARY.md)** - 阅读路线建议
- **[01_Overview.md](01_Overview.md)** - 原始库简介
- **[02_Patches.md](02_Patches.md)** - Patch 详细分析
- **[03_Build_Integration.md](03_Build_Integration.md)** - OH 构建适配
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用
- **[05_API_Differences.md](05_API_Differences.md)** - API/接口差异
- **[06_Security.md](06_Security.md)** - 安全风险分析

- **工作文件**：
  - **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 项目评估结果
  - **[NOTES.md](wiki/_work/NOTES.md)** - 分析过程记录
  - **[PLAN.md](wiki/_work/PLAN.md)** - 任务进度

---

**维护者**：OpenHarmony 社区
**最后更新**：2026-02-08
