# Sonic Wiki 阅读路线建议

本文档提供针对不同需求和角色的阅读路线建议，帮助您快速找到所需信息。

---

## 推荐阅读路线

### 路线 1: 快速概览（5 分钟）

适合：初次接触，想了解基本情况

1. [**README.md**](README.md) - 查看关键信息速览表
2. [**01_Overview.md**](01_Overview.md) 第 1、2 节 - 库简介和在 OH 中的定位

### 路线 2: 深入了解 OH 适配（15 分钟）

适合：需要全面了解 OH 定制化内容的开发者

1. [**01_Overview.md**](01_Overview.md) - 了解库的整体情况
2. [**02_Patches.md**](02_Patches.md) - **重点** - 了解 OH 的关键 Bug 修复
3. [**03_Build_Integration.md**](03_Build_Integration.md) - 了解构建适配
4. [**04_Usage_in_OH.md**](04_Usage_in_OH.md) - 了解依赖关系

### 路线 3: 开发使用指南（20 分钟）

适合：需要在代码中使用 sonic 的开发者

1. [**01_Overview.md**](01_Overview.md) 第 1.3-1.4 节 - 算法原理和 API 概述
2. [**05_API_Differences.md**](05_API_Differences.md) - **重点** - API 详细说明和使用示例
3. [**03_Build_Integration.md**](03_Build_Integration.md) 第 5 节 - 如何在 BUILD.gn 中声明依赖
4. [**04_Usage_in_OH.md**](04_Usage_in_OH.md) 第 3 节 - 使用场景分析

### 路线 4: 升级维护指南（25 分钟）

适合：负责维护和升级 sonic 库的工程师

1. [**02_Patches.md**](02_Patches.md) - **重点** - 了解需要保留的 OH 特有修改
2. [**03_Build_Integration.md**](03_Build_Integration.md) - 了解 BUILD.gn 配置
3. [**05_API_Differences.md**](05_API_Differences.md) - 检查 API 兼容性
4. [**06_Security.md**](06_Security.md) - 了解安全注意事项

### 路线 5: 安全审计（15 分钟）

适合：进行安全评估或审计的工程师

1. [**06_Security.md**](06_Security.md) - **重点** - 完整的安全分析
2. [**02_Patches.md**](02_Patches.md) 第 2.1 节 - 双声道修复的安全评估
3. [**03_Build_Integration.md**](03_Build_Integration.md) 第 2.3.1 节 - 分支保护安全特性

---

## 按角色阅读

### 架构师

**关注重点**: 整体架构、依赖关系、风险评估

1. [01_Overview.md](01_Overview.md) 第 2 节 - OH 架构位置和定位
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) 第 4 节 - 依赖关系图
3. [06_Security.md](06_Security.md) 第 1、7 节 - 安全概况和结论

### 应用开发者

**关注重点**: 如何使用 sonic API

1. [01_Overview.md](01_Overview.md) 第 1.4 节 - API 概述
2. [05_API_Differences.md](05_API_Differences.md) 第 1、5 节 - API 说明和示例
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) 第 5 节 - 依赖声明方式

### 系统开发者

**关注重点**: 集成到音频框架

1. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 完整了解依赖关系
2. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建配置
3. [02_Patches.md](02_Patches.md) - 了解 OH 特有修改

### 维护工程师

**关注重点**: 日常维护、升级、问题排查

1. [02_Patches.md](02_Patches.md) - 了解所有 OH 修改
2. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建系统
3. [06_Security.md](06_Security.md) - 了解安全注意事项

---

## 按问题查找

### 问题: 这个库是做什么的？
→ [01_Overview.md](01_Overview.md) 第 1.1-1.2 节

### 问题: OH 对这个库做了什么修改？
→ [02_Patches.md](02_Patches.md) 第 2 节

### 问题: 如何在 BUILD.gn 中添加 sonic 依赖？
→ [03_Build_Integration.md](03_Build_Integration.md) 第 5 节
→ [04_Usage_in_OH.md](04_Usage_in_OH.md) 第 5.3 节

### 问题: 哪些模块使用了 sonic？
→ [04_Usage_in_OH.md](04_Usage_in_OH.md) 第 1、2 节

### 问题: 如何使用 sonic API？
→ [05_API_Differences.md](05_API_Differences.md) 第 1、5 节

### 问题: sonic 有安全漏洞吗？
→ [06_Security.md](06_Security.md) 第 1、2、7 节

### 问题: 升级上游版本要注意什么？
→ [02_Patches.md](02_Patches.md) 第 3 节
→ [05_API_Differences.md](05_API_Differences.md) 第 4 节
→ [06_Security.md](06_Security.md) 第 6 节

### 问题: sonic 在 OH 架构中的位置？
→ [01_Overview.md](01_Overview.md) 第 2.2 节
→ [04_Usage_in_OH.md](04_Usage_in_OH.md) 第 4 节

---

## 文档关系图

```
README.md (入口)
    │
    ├── 01_Overview.md (基础)
    │       ├── 02_Patches.md (Patch 详情)
    │       └── 05_API_Differences.md (API 详情)
    │
    ├── 03_Build_Integration.md (构建)
    │       └── 04_Usage_in_OH.md (依赖)
    │
    └── 06_Security.md (安全)
```

---

## 关键文档速查

| 想了解什么 | 查看文档 | 具体章节 |
|------------|----------|----------|
| 库的基本信息 | 01_Overview.md | 第 1 节 |
| 在 OH 中的作用 | 01_Overview.md | 第 2 节 |
| OH 做了什么修改 | 02_Patches.md | 完整阅读 |
| BUILD.gn 怎么配 | 03_Build_Integration.md | 第 1、5 节 |
| 谁依赖了 sonic | 04_Usage_in_OH.md | 第 1、2 节 |
| API 怎么用 | 05_API_Differences.md | 第 1、5 节 |
| 安全吗 | 06_Security.md | 第 1、7 节 |
| 怎么升级 | 02_Patches.md + 06_Security.md | Patch 分析 + 升级策略 |

---

**提示**: 所有文档均使用 Markdown 格式，支持快速搜索关键字查找所需内容。
