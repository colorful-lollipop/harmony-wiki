# 文档目录与阅读路线

## 📖 文档结构

```
css-what/wiki/
├── README.md              # 库概览与导航（入口文档）
├── SUMMARY.md             # 本文档，阅读路线指引
├── 01_Overview.md         # 原始库概述 + OH 定位
├── 02_Patches.md          # Patch 清单及详细分析
├── 03_Build_Integration.md # BUILD.gn 适配详解
├── 04_Usage_in_OH.md      # 依赖关系与使用场景
├── 05_API_Differences.md  # API 差异（如有）
├── 06_Security.md         # 安全风险分析（如有）
└── _work/
    ├── ASSESSMENT.md      # 项目评估报告
    ├── NOTES.md           # 分析过程记录
    └── PLAN.md            # 任务进度追踪
```

## 🎯 推荐阅读路线

### 场景一：快速了解

**目标**：在 5 分钟内了解 css-what 在 OH 中的基本情况

| 顺序 | 文档           | 预计时间 | 重点内容               |
| ---- | -------------- | -------- | ---------------------- |
| 1    | README.md      | 2 min    | 库概览、适配状态、导航 |
| 2    | 01_Overview.md | 3 min    | 功能介绍、OH 定位      |

### 场景二：深入适配细节

**目标**：了解库的 OH 适配工作原理

| 顺序 | 文档                    | 预计时间 | 重点内容           |
| ---- | ----------------------- | -------- | ------------------ |
| 1    | README.md               | 2 min    | 整体概览           |
| 2    | 03_Build_Integration.md | 5 min    | BUILD.gn 配置详解  |
| 3    | 04_Usage_in_OH.md       | 5 min    | 依赖关系与使用场景 |

### 场景三：Patch 分析（运维场景）

**目标**：了解库的所有修改，用于问题排查或升级评估

| 顺序 | 文档                   | 预计时间 | 重点内容        |
| ---- | ---------------------- | -------- | --------------- |
| 1    | README.md              | 2 min    | 快速概览        |
| 2    | 02_Patches.md          | 10 min   | 完整 Patch 清单 |
| →    | **结论**：本库无 Patch | -        | -               |

### 场景四：安全评估

**目标**：评估库的安全风险，指导安全更新策略

| 顺序 | 文档           | 预计时间 | 重点内容           |
| ---- | -------------- | -------- | ------------------ |
| 1    | README.md      | 2 min    | 快速概览           |
| 2    | 06_Security.md | 10 min   | CVE 列表与修复建议 |

## 📋 文档速查表

| 如果你想...        | 阅读此文档              |
| ------------------ | ----------------------- |
| 了解库的基本功能   | 01_Overview.md          |
| 查看有哪些 Patch   | 02_Patches.md           |
| 了解如何构建       | 03_Build_Integration.md |
| 了解谁在使用这个库 | 04_Usage_in_OH.md       |
| 查看 OH 特有的 API | 05_API_Differences.md   |
| 了解安全风险       | 06_Security.md          |
| 查看完整评估报告   | \_work/ASSESSMENT.md    |

## 🔍 关键结论速览

### 该库的核心特点

- ✅ **无 Patch**：纯上游源码集成，无任何 OH 特有修改
- ✅ **简单适配**：仅通过 BUILD.gn 导出源文件
- ✅ **类型安全**：纯 TypeScript 库，无原生代码

### 主要依赖者

1. **jsframework** - JS 框架核心
2. **ace_engine** - ArkUI 引擎

### 使用场景

CSS 选择器解析，用于 ArkUI 框架的样式选择功能。

## 📊 文档维护状态

| 文档                    | 状态              | 最后更新   |
| ----------------------- | ----------------- | ---------- |
| README.md               | ✅ 完成           | 2026-02-08 |
| SUMMARY.md              | ✅ 完成           | 2026-02-08 |
| 01_Overview.md          | ⏳ 待编写         | -          |
| 02_Patches.md           | ⏳ 待编写         | -          |
| 03_Build_Integration.md | ⏳ 待编写         | -          |
| 04_Usage_in_OH.md       | ⏳ 待编写         | -          |
| 05_API_Differences.md   | ⏳ 待编写（可选） | -          |
| 06_Security.md          | ⏳ 待编写（可选） | -          |

## 🔗 相关链接

- [上游 GitHub 仓库](https://github.com/fb55/css-what)
- [OpenHarmony 第三方库规范](../README.md)
- [贡献指南](../../../../CONTRIBUTING.md)

---

_文档版本：1.0_
_最后更新：2026-02-08_
