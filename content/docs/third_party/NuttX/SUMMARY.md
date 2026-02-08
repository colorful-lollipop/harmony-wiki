# 阅读指南

本文档提供 NuttX Wiki 的完整阅读导航，帮助不同角色的读者快速找到所需信息。

---

## 推荐阅读路线

### 🚀 路线 1：快速概览（5 分钟）

适合：项目经理、技术决策者

**阅读顺序**：
1. [README.md](README.md) - 项目概述（必读）
2. [01_Overview.md](01_Overview.md) - 库简介（快速扫描）

**产出**：了解 NuttX 在 OH 中的定位和核心价值

---

### 👨‍💻 路线 2：技术集成（15 分钟）

适合：内核开发者、系统架构师

**阅读顺序**：
1. [01_Overview.md](01_Overview.md) - 原始库简介
2. [02_Patches.md](02_Patches.md) - Patch 分析（无 Patch 机制）
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建适配
4. [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用详情

**产出**：理解 NuttX 如何集成到 OH，构建配置和依赖关系

---

### 🔧 路线 3：深度技术（30 分钟）

适合：需要深入了解的开发者

**阅读顺序**：
1. 路线 2 的所有文档
2. [05_API_Differences.md](05_API_Differences.md) - API 差异
3. [06_Security.md](06_Security.md) - 安全考量
4. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 完整评估报告

**产出**：掌握 NuttX 在 OH 中的技术细节、安全风险和升级策略

---

### 📋 路线 4：维护者专用

适合：NuttX 模块维护者

**阅读顺序**：
1. [_work/NOTES.md](_work/NOTES.md) - 分析过程记录
2. [_work/PLAN.md](_work/PLAN.md) - 任务计划
3. 所有核心文档
4. [06_Security.md](06_Security.md) - 安全监控建议

**产出**：了解维护注意事项、变更记录和监控策略

---

## 按主题查找

### 了解基本概念

| 问题 | 答案 |
|-----|------|
| NuttX 是什么？ | [01_Overview.md](01_Overview.md) |
| OH 为什么使用 NuttX？ | [01_Overview.md#oh-定位](01_Overview.md#oh-定位) |
| NuttX 和 LiteOS 什么关系？ | [01_Overview.md#与-liteos-a-的关系](01_Overview.md#与-liteos-a-的关系) |

### 构建和配置

| 问题 | 答案 |
|-----|------|
| 如何配置 NuttX 模块？ | [03_Build_Integration.md](03_Build_Integration.md) |
| NuttX.gni 包含什么？ | [03_Build_Integration.md#nuttxgni-配置](03_Build_Integration.md#nuttxgni-配置) |
| 哪些模块被复用了？ | [03_Build_Integration.md#模块列表](03_Build_Integration.md#模块列表) |

### 使用和依赖

| 问题 | 答案 |
|-----|------|
| 哪些 OH 模块依赖 NuttX？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| VFS 如何工作？ | [04_Usage_in_OH.md#文件系统模块](04_Usage_in_OH.md#文件系统模块) |
| 驱动程序如何使用？ | [04_Usage_in_OH.md#驱动程序模块](04_Usage_in_OH.md#驱动程序模块) |

### 安全和维护

| 问题 | 答案 |
|-----|------|
| 有已知的安全漏洞吗？ | [06_Security.md](06_Security.md) |
| 如何升级 NuttX 版本？ | [02_Patches.md#升级建议](02_Patches.md#升级建议) |
| 需要关注哪些 CVE？ | [06_Security.md#cve-监控](06_Security.md#cve-监控) |

---

## 文档依赖关系图

```
                    README.md
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
  Overview.md     SUMMARY.md      _work/PLAN.md
        │               ▲
        │               │
        ▼               │
  01_Overview ──────────┤
        │               │
        ├──► 02_Patches ──┤
        │               │
        ├──► 03_Build_Integration ──┤
        │                           │
        ├──► 04_Usage_in_OH ────────┤
        │               │           │
        │               └──► 05_API_Differences ──► 06_Security
        │                           │
        └──► _work/ASSESSMENT ──────┘
                │
                └──► _work/NOTES
```

---

## 关键信息速查

### 基础信息

| 项目 | 值 |
|-----|-----|
| 上游版本 | 12.10.0 |
| OH 版本 | 3.1 |
| 许可证 | Apache 2.0 / BSD 3-Clause |
| 复用文件数 | 65 个源文件 |
| 依赖模块数 | 9 个 OH 模块 |

### 模块分类

| 模块类型 | 数量 | 示例 |
|---------|------|------|
| 文件系统 | 6 | VFS, RAMFS, ROMFS, NFS |
| 驱动程序 | 2 | 块缓存, 帧缓冲 |
| IPC | 1 | 管道 |

### 核心配置文件

| 文件 | 用途 |
|-----|------|
| `NuttX.gni` | 定义所有复用的源文件列表 |

---

## 版本历史

| 版本 | 日期 | 变更 |
|-----|------|------|
| 1.0 | 2025-02-07 | 初始版本 |

---

## 反馈和改进

如需补充或修正内容，请联系项目维护者或通过 OpenHarmony 社区渠道反馈。
