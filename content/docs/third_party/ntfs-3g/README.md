# NTFS-3G 库 OpenHarmony 适配文档

## 简介

NTFS-3G 是一个开源的跨平台 NTFS 文件系统读写驱动程序，为 OpenHarmony 系统提供了完整的 Windows NTFS 文件系统支持能力。本文档详细记录了 NTFS-3G 库在 OpenHarmony 生态系统中的集成方式、构建适配、依赖关系以及使用场景。

NTFS（New Technology File System）是 Microsoft Windows 操作系统广泛使用的文件系统格式。从 Windows XP 到 Windows 2019 的所有 Windows 版本都采用 NTFS 作为其默认文件系统。在移动设备和桌面操作系统互联互通的场景中，支持 NTFS 读写能力对于数据交换至关重要。OpenHarmony 通过集成 NTFS-3G 库，使系统能够直接访问和操作 Windows 格式的存储设备，包括 USB 闪存盘、外置硬盘以及 SD 卡等。

本库的上游版本为 2022.10.3，源自 Tuxera 公司维护的 NTFS-3G 项目。在 OpenHarmony 中，该库被标记为版本 3.1，属于 thirdparty 子系统，适用于标准系统（standard）配置。

## OpenHarmony 适配概述

### 适配策略

NTFS-3G 在 OpenHarmony 中的集成采用了**构建适配优先、代码修改最小化**的核心策略。这一策略的关键特点包括：

**第一，构建系统转换**：将原生的 Autotools 构建系统（./configure && make）转换为 OpenHarmony 的 GN 构建系统。通过在库目录中添加四个 BUILD.gn 文件（分别位于根目录、libntfs-3g、libfuse-lite 和 ntfsprogs 子目录），实现了与 OpenHarmony 构建系统的无缝集成。

**第二，配置预生成**：在仓库中预置了 config.h 配置文件，该文件已在 OpenHarmony 构建环境中完成配置。这种方式避免了在不同平台运行 configure 脚本可能产生的差异，确保了构建的可重复性。

**第三，静态库集成**：将 libntfs-3g 和 libfuse-lite 构建为静态库，通过静态链接方式被各个工具程序使用。这种模式简化了依赖管理，减少了运行时依赖。

### 无 Patch 特性

与 OpenHarmony 中的许多第三方库不同，NTFS-3G 库**没有任何 Patch 文件**。这并不意味着该库未经适配，而是采用了更优雅的适配方式。所有必要的适配工作都通过 BUILD.gn 构建配置和预生成的 config.h 文件完成，源代码保持了与上游版本的高度一致性。

这种无 Patch 的适配策略带来了以下优势：源代码清晰易读，便于理解和使用；维护成本低，升级上游版本时无需处理 Patch 冲突；稳定性高，减少了因代码修改引入新 bug 的风险。

### 核心组件

该库由三个核心组件构成：

**libntfs-3g 静态库**：提供 NTFS 文件系统的核心读写功能，包括 MFT 解析、属性处理、簇分配、索引维护、安全描述符处理以及压缩文件支持等。这是整个库的核心实现，被所有工具程序所依赖。

**libfuse-lite 静态库**：提供 FUSE（Filesystem in Userspace）接口的轻量级实现。NTFS-3G 采用用户空间文件系统的设计，通过 libfuse-lite 与操作系统内核交互，实现文件系统操作。这种设计提高了系统的稳定性和安全性。

**ntfsprogs 工具集**：提供四个命令行工具程序，分别是 fsck.ntfs（文件系统检查）、mount.ntfs（文件系统挂载）、ntfsfix（文件系统修复）和 ntfslabel（卷标管理）。这些工具为用户和系统服务提供了完整的 NTFS 管理能力。

## 文档导航

本文档集由以下文档组成，按照从概述到细节的逻辑顺序组织：

| 文档 | 内容概述 | 建议阅读对象 |
|------|---------|-------------|
| **SUMMARY.md** | 文档阅读路线建议和快速索引 | 所有读者 |
| **01_Overview.md** | 原始库功能介绍和 OH 定位 | 需要了解背景知识的读者 |
| **02_Patches.md** | Patch 分析（本库无 Patch） | 开发者、维护者 |
| **03_Build_Integration.md** | GN 构建适配详解 | 构建系统开发者 |
| **04_Usage_in_OH.md** | 依赖关系和使用场景 | 系统集成者 |
| **05_API_Differences.md** | API 差异分析 | API 使用者 |
| **06_Security.md** | 安全风险分析 | 安全审计人员 |

## 快速开始

### 在 OpenHarmony 中使用 NTFS-3G

该库已预装在 OpenHarmony 标准系统中，用户和开发者无需额外配置即可使用。以下是典型使用场景：

**挂载 NTFS 存储设备**：

```bash
mount.ntfs /dev/sda1 /mnt/windows
```

**检查 NTFS 文件系统**：

```bash
fsck.ntfs /dev/sda1
```

**修复 NTFS 问题**：

```bash
ntfsfix /dev/sda1
```

### 开发者集成

对于需要在应用中集成 NTFS 读写能力的开发者，建议阅读以下章节：

需要了解 libntfs-3g 静态库的使用方式，请参考 04_Usage_in_OH.md 文档中的依赖关系部分。

需要自定义构建配置，请参考 03_Build_Integration.md 文档中的编译选项说明。

### 维护者指南

对于负责维护该库在 OpenHarmony 中集成的维护者，以下文档尤为重要：

**升级上游版本**：参考 02_Patches.md 文档中的升级建议部分，了解如何平滑升级到新版本的 NTFS-3G。

**构建问题排查**：参考 03_Build_Integration.md 文档，了解 GN 构建配置的具体细节。

**安全更新**：参考 06_Security.md 文档，获取已知 CVE 和安全修复建议。

## 版本信息

| 项目 | 版本 |
|------|-----|
| **上游版本** | 2022.10.3 |
| **OpenHarmony 版本** | 3.1 |
| **许可证** | GPL-2.0-or-later |
| **上游地址** | https://github.com/tuxera/ntfs-3g |

## 相关资源

- **上游项目**：Tuxera NTFS-3G（https://github.com/tuxera/ntfs-3g）
- **OpenHarmony 官方文档**：https://developer.harmonyos.com
- **构建系统文档**：GN 构建系统参考（https://gn.googlesource.com/gn/）

---

*本文档最后更新于 2026 年 2 月 7 日*
