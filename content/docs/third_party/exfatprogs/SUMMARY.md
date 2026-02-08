# 阅读路线指南

本文档为不同角色的读者提供针对性的阅读建议，帮助您快速定位所需信息。

## 适用角色

### 1. 系统集成者

**目标**：了解如何在 OpenHarmony 项目中使用 exfatprogs

**推荐阅读顺序**：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | README.md | 库概述、快速开始 |
| 2 | 04_Usage_in_OH.md | 依赖添加方式、API 使用示例 |
| 3 | 03_Build_Integration.md | 构建配置细节 |

**关键问题**：
- 如何添加对 exfatprogs 的依赖？
- libexfat 提供了哪些 API？
- 编译时需要链接哪些库？

### 2. 构建系统维护者

**目标**：理解 exfatprogs 在 OH 构建系统中的集成方式

**推荐阅读顺序**：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | 03_Build_Integration.md | BUILD.gn 配置详解 |
| 2 | README.md | 构建适配概述 |
| 3 | 02_Patches.md | Patch 状态（确认无 Patch） |

**关键问题**：
- BUILD.gn 的配置结构是怎样的？
- 如何添加新的构建目标？
- 与上游构建系统的差异是什么？

### 3. 库维护者

**目标**：了解 exfatprogs 的 OH 适配状态和升级策略

**推荐阅读顺序**：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | 02_Patches.md | Patch 分析和升级建议 |
| 2 | 03_Build_Integration.md | 构建配置适配 |
| 3 | ASSESSMENT.md | 项目整体评估 |
| 4 | _work/NOTES.md | 分析过程记录 |

**关键问题**：
- 上游版本升级时需要处理哪些 Patch？
- OH 特有的配置变更有哪些？
- 版本同步策略是什么？

### 4. 安全审计人员

**目标**：评估 exfatprogs 在 OH 中的安全风险

**推荐阅读顺序**：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | 02_Patches.md | OH 引入的代码变更 |
| 2 | 04_Usage_in_OH.md | 使用场景和依赖链 |
| 3 | ASSESSMENT.md | 安全相关评估 |

**关键问题**：
- OH 引入的 Patch 是否增加新的攻击面？
- 依赖关系是否存在安全风险？
- CVE 修复状态如何？

## 文档依赖关系

```
README.md (必读)
    │
    ├──► 01_Overview.md (背景知识)
    │
    ├──► 02_Patches.md (维护升级必读)
    │       │
    │       └──► ASSESSMENT.md (详细评估)
    │
    ├──► 03_Build_Integration.md (构建相关)
    │       │
    │       └──► ASSESSMENT.md (技术细节)
    │
    └──► 04_Usage_in_OH.md (集成使用)
            │
            └──► 03_Build_Integration.md (构建依赖)
```

## 快速问答

| 问题 | 答案 |
|------|------|
| exfatprogs 有 OH 特有的 Patch 吗？ | 没有。该库为干净导入，所有源码与上游一致。 |
| 如何在项目中添加依赖？ | 在 BUILD.gn 中添加 `deps += [ "//third_party/exfatprogs:libexfat" ]` |
| libexfat 提供哪些功能？ | exFAT 文件系统的挂载、读取、写入、创建、检查、修复等操作。 |
| 该库是否稳定？ | 稳定。exfatprogs 是 Linux 官方 exFAT 用户空间工具，经过充分验证。 |

## 版本信息

| 项目 | 值 |
|------|-----|
| **本文档版本** | 1.0 |
| **创建日期** | 2026-02-07 |
| **适用版本** | exfatprogs OH 4.1 (上游 1.2.5) |
