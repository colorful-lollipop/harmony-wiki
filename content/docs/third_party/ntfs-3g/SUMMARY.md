# 文档阅读路线指南

本文档旨在帮助不同角色的读者快速找到所需信息。以下提供了针对不同使用场景的阅读建议和文档索引。

## 读者角色与阅读建议

### 系统集成者

如果您的任务是了解 NTFS-3G 库在 OpenHarmony 系统中的定位和使用方式，建议按照以下顺序阅读：

首先阅读 01_Overview.md，了解 NTFS-3G 库的基本功能和 OpenHarmony 定位，建立对该库的整体认知。

然后阅读 04_Usage_in_OH.md，这是系统集成者最核心的文档，详细描述了该库的依赖关系、被依赖情况以及在 OpenHarmony 系统中的典型使用场景。通过该文档，您可以了解如何在自己的模块中集成和使用该库的功能。

如果需要深入了解构建配置细节，可以进一步阅读 03_Build_Integration.md。

### 应用开发者

如果您的任务是开发使用 NTFS 存储设备的应用，建议重点关注以下内容：

阅读 04_Usage_in_OH.md 中的使用场景部分，了解 NTFS-3G 提供的工具程序和库接口，以及如何调用这些接口实现 NTFS 设备的读写操作。

查阅 01_Overview.md 中的功能特性部分，了解该库支持哪些 NTFS 操作，如文件读写、目录管理、权限处理等。

如果您的应用涉及底层集成，还需要了解 05_API_Differences.md 中描述的 API 使用注意事项。

### 构建系统开发者

如果您负责维护或修改该库的构建配置，03_Build_Integration.md 是您的核心参考文档。该文档详细说明了 GN 构建系统的配置细节，包括各模块的构建定义、编译选项设置以及与上游构建方式的差异。

在修改构建配置前，建议同时阅读 02_Patches.md，了解该库为何采用无 Patch 的适配策略，以及这一决策对构建配置的影响。

### 安全审计人员

如果您需要评估该库对 OpenHarmony 系统安全性的影响，请重点阅读 06_Security.md 文档。该文档提供了该库已知的安全漏洞信息、OpenHarmony 版本中的修复状态以及安全使用建议。

同时，建议阅读 03_Build_Integration.md 了解哪些安全相关功能被禁用，以及这些禁用决策的安全考量。

### 库维护者

如果您负责维护该库在 OpenHarmony 中的集成，以下文档都是必读内容：

02_Patches.md 包含了升级上游版本时的注意事项和建议，是版本升级的核心参考。

03_Build_Integration.md 详细记录了构建适配的所有细节，是排查构建问题的首选文档。

06_Security.md 提供了安全更新的建议和策略，帮助您制定该库的安全维护计划。

## 文档快速索引

### 概述性文档

README.md 提供了库的简介、适配概述和文档导航，是本文档集的入口文件。SUMMARY.md（即本文档）提供了阅读路线建议。01_Overview.md 介绍了原始库的功能和 OpenHarmony 定位。

### 技术细节文档

02_Patches.md 记录了所有 Patch 文件的分析（本库无 Patch）。03_Build_Integration.md 是构建适配的权威参考。04_Usage_in_OH.md 详细描述了依赖关系和使用场景。

### 专项分析文档

05_API_Differences.md 分析了 API 层面的差异和注意事项。06_Security.md 提供了安全风险分析和建议。

## 关键信息速查

### 版本信息速查

上游版本号：2022.10.3，OpenHarmony 版本号：3.1，许可证：GPL-2.0-or-later，所属子系统：thirdparty。

### 适配状态速查

Patch 状态：无 Patch，构建适配：已完成，功能完整性：完整，安全审查状态：请参考 06_Security.md。

### 核心模块速查

libntfs-3g：NTFS 核心库，静态链接。libfuse-lite：FUSE 接口库，静态链接。ntfsprogs：四个工具程序（fsck.ntfs、mount.ntfs、ntfsfix、ntfslabel）。

## 阅读流程图

```
开始
  │
  ▼
您是谁？
  │
  ├─ 系统集成者 ──→ 01_Overview.md → 04_Usage_in_OH.md → 03_Build_Integration.md
  │
  ├─ 应用开发者 ──→ 01_Overview.md → 04_Usage_in_OH.md → 使用场景部分
  │
  ├─ 构建开发者 ──→ 03_Build_Integration.md → 02_Patches.md → 04_Usage_in_OH.md
  │
  ├─ 安全审计员 ──→ 06_Security.md → 03_Build_Integration.md → 安全相关配置
  │
  └─ 库维护者 ────→ 02_Patches.md → 03_Build_Integration.md → 06_Security.md
```

## 常见问题快速定位

**问题：如何在应用中集成 NTFS 读写功能？**

请阅读 04_Usage_in_OH.md 中的依赖关系部分，了解静态库链接和工具程序调用的方式。

**问题：构建时出现编译错误如何处理？**

请阅读 03_Build_Integration.md 中的配置详情，对照检查您的构建环境配置是否正确。

**问题：升级上游版本需要注意什么？**

请阅读 02_Patches.md 中的升级建议部分，了解 Patch 迁移和兼容性检查的要点。

**问题：该库存在哪些安全风险？**

请阅读 06_Security.md，获取该库已知 CVE 和 OpenHarmony 版本中的修复状态。

**问题：该库在 OpenHarmony 中如何被使用？**

请阅读 04_Usage_in_OH.md 中的被依赖关系部分，了解该库在系统中的定位和使用方式。

## 文档更新历史

| 日期 | 版本 | 更新内容 |
|------|------|---------|
| 2026-02-07 | 1.0 | 初始版本，完成所有文档框架和核心内容 |
