# 阅读路线建议

本文档为不同角色提供针对性的阅读路线，帮助快速找到所需信息。

---

## 🎯 不同角色的阅读路线

### 1. OH 系统集成工程师

**目标**：了解 syn 在 OH 中的集成方式和升级策略

**推荐阅读路线**：
1. [README.md](README.md) - 快速了解 syn 的定位和集成状态
2. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 配置和 features 策略
3. [02_Patches.md](02_Patches.md) - 确认无 Patch，理解为什么不需要 Patch
4. [06_Security.md](06_Security.md) - 了解安全风险和升级建议

**关键信息**：
- syn 无需 Patch，可以直接升级
- 升级前需验证依赖者兼容性
- 启用所有 features，提供完整功能

---

### 2. OH 应用开发者

**目标**：理解如何使用 syn 开发过程宏

**推荐阅读路线**：
1. [README.md](README.md) - 了解 syn 的作用
2. [01_Overview.md](01_Overview.md) - 原始库功能简介
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看其他模块如何使用 syn
4. [03_Build_Integration.md](03_Build_Integration.md) - 了解如何在自己的模块中引用

**关键信息**：
- 添加依赖：`"//third_party/rust/crates/syn:lib"`
- 同时依赖 proc-macro2 和 quote
- 参考现有的 ani_rs_macros 实现

---

### 3. 第三方库维护者

**目标**：了解 OH 的集成方式，帮助自己的库集成到 OH

**推荐阅读路线**：
1. [03_Build_Integration.md](03_Build_Integration.md) - 学习 OH 构建系统集成
2. [README.md](README.md) - 了解 syn 的集成方式
3. [01_Overview.md](01_Overview.md) - 理解原始库的功能
4. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解 OH 如何使用第三方库

**关键信息**：
- syn 使用 `ohos_cargo_crate` 模板
- 保持版本与上游一致
- 无 Patch 的集成方式

---

### 4. 安全研究员

**目标**：评估 syn 的安全状态和风险

**推荐阅读路线**：
1. [06_Security.md](06_Security.md) - 安全风险分析和 CVE 状态
2. [01_Overview.md](01_Overview.md) - 了解 syn 的功能范围
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解影响范围
4. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建配置

**关键信息**：
- syn 是纯语法树解析库，攻击面相对较小
- 无 OH 特定代码，安全风险与上游一致
- 影响 OH 的所有过程宏

---

### 5. 架构师

**目标**：理解 syn 在 OH Rust 生态中的定位

**推荐阅读路线**：
1. [README.md](README.md) - 快速了解 syn 的定位
2. [01_Overview.md](01_Overview.md) - 原始库功能
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看依赖关系图
4. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建策略

**关键信息**：
- syn 是 OH Rust 生态的核心基础设施
- 支持所有过程宏的开发
- 与其他 crates（proc-macro2、quote）组成"过程宏三巨头"

---

## 📚 文档结构说明

### 必读文档（按优先级）

| 优先级 | 文档 | 说明 |
|-------|------|-----|
| ⭐⭐⭐ | [README.md](README.md) | 入门必读，快速了解全局 |
| ⭐⭐⭐ | [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配的核心文档 |
| ⭐⭐ | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系，理解影响力 |
| ⭐⭐ | [06_Security.md](06_Security.md) | 安全评估，升级决策 |
| ⭐ | [01_Overview.md](01_Overview.md) | 原始库简介，背景信息 |
| ⭐ | [02_Patches.md](02_Patches.md) | Patch 分析（本库无 Patch） |
| ⭐ | [05_API_Differences.md](05_API_Differences.md) | API 差异（本库无差异） |

### 工作文档

| 文档 | 说明 |
|------|-----|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | Phase 0 完整评估结果，包含所有原始发现 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程中的笔记和思考 |
| [_work/PLAN.md](./_work/PLAN.md) | 文档编写的进度追踪 |

---

## 🚀 快速入门

### 5 分钟了解 syn

1. **是什么**：Rust 语法树解析库，用于开发过程宏
2. **OH 集成**：无 Patch，直接集成，版本 2.0.48
3. **使用场景**：所有 OH 过程宏的基础，包括 ani_rs_macros、serde_derive 等
4. **升级策略**：直接升级，但需验证依赖兼容性

### 15 分钟深入理解

1. 阅读 [README.md](README.md) 的"库概览"和"OH 适配概述"
2. 查看 [03_Build_Integration.md](03_Build_Integration.md) 的 BUILD.gn 配置
3. 浏览 [04_Usage_in_OH.md](04_Usage_in_OH.md) 的依赖者清单
4. 检查 [06_Security.md](06_Security.md) 的升级建议

### 30 分钟全面掌握

1. 完整阅读 [README.md](README.md)
2. 阅读 [03_Build_Integration.md](03_Build_Integration.md) 和 [04_Usage_in_OH.md](04_Usage_in_OH.md)
3. 查看 [01_Overview.md](01_Overview.md) 了解原始库功能
4. 阅读 [06_Security.md](06_Security.md) 了解安全风险
5. 查看 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 了解评估过程

---

## 🔍 常见问题快速定位

| 问题 | 相关文档 |
|-----|---------|
| syn 是什么？ | [README.md](README.md) 库概览 |
| OH 如何集成 syn？ | [03_Build_Integration.md](03_Build_Integration.md) |
| 为什么不需要 Patch？ | [02_Patches.md](02_Patches.md) |
| 谁在使用 syn？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| 如何升级 syn？ | [06_Security.md](06_Security.md) |
| 如何在自己的模块中使用 syn？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| syn 有安全风险吗？ | [06_Security.md](06_Security.md) |
| BUILD.gn 如何配置 features？ | [03_Build_Integration.md](03_Build_Integration.md) |

---

## 📝 文档维护说明

### 更新频率

- **高频率**：[06_Security.md](06_Security.md) - 随时更新 CVE 信息
- **中频率**：[README.md](README.md) - 版本升级时更新
- **低频率**：其他文档 - 除非有重大变更

### 更新触发条件

- syn 版本升级
- 发现新的依赖者
- 识别到安全漏洞
- OH 构建系统变更
- 用户反馈需要补充信息

---

**文档最后更新**：2026-02-08
**适用版本**：syn 2.0.48
