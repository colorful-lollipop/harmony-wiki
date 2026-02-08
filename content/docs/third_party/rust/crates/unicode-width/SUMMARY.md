# 阅读路线建议

本文档提供不同角色的推荐阅读顺序。

## 阅读路线

### 路线一：快速了解（5分钟）
**目标**：快速了解这是什么库，在 OH 中起什么作用

1. [README.md](./README.md) - 库概览和核心特点
2. [01_Overview.md](./01_Overview.md) - 功能定位和在 OH 中的作用
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 谁在用它

### 路线二：维护人员（15分钟）
**目标**：全面了解库的 OH 适配情况，为升级和维护做准备

1. [README.md](./README.md) - 概览
2. [02_Patches.md](./02_Patches.md) - **重点**：确认无 Patch 及原因
3. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 配置
4. [06_Security.md](./06_Security.md) - 安全评估
5. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 详细评估报告

### 路线三：开发者使用（10分钟）
**目标**：了解如何在 OH 项目中使用此库

1. [01_Overview.md](./01_Overview.md) - 功能介绍
2. [03_Build_Integration.md](./03_Build_Integration.md) - 依赖方式
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景示例
4. 参考上游文档：https://docs.rs/unicode-width

### 路线四：审计/安全评估（10分钟）
**目标**：评估第三方库的安全风险

1. [README.md](./README.md) - 基本信息
2. [02_Patches.md](./02_Patches.md) - 代码变更分析
3. [05_API_Differences.md](./05_API_Differences.md) - API 差异
4. [06_Security.md](./06_Security.md) - 安全分析

## 文档依赖关系

```
README.md (起点)
    ├── 01_Overview.md (基础)
    ├── 02_Patches.md (核心)
    ├── 03_Build_Integration.md (构建)
    ├── 04_Usage_in_OH.md (使用)
    ├── 05_API_Differences.md (API)
    └── 06_Security.md (安全)
```

## 关键文档说明

| 文档 | 阅读优先级 | 内容特点 |
|------|-----------|---------|
| README.md | ⭐⭐⭐⭐⭐ | 必读的起点和导航 |
| 01_Overview.md | ⭐⭐⭐⭐ | 功能介绍和 OH 定位 |
| 02_Patches.md | ⭐⭐⭐⭐⭐ | **核心文档**：本文档重点说明无 Patch 的情况 |
| 03_Build_Integration.md | ⭐⭐⭐ | 构建系统适配说明 |
| 04_Usage_in_OH.md | ⭐⭐⭐⭐ | 依赖关系和使用场景 |
| 05_API_Differences.md | ⭐⭐ | API 差异（与上游一致） |
| 06_Security.md | ⭐⭐⭐ | 安全风险评估 |
