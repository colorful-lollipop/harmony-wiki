# SUMMARY - dmsfwk_lite 工程 Wiki 导航

## 文档概览

| 编号 | 文档名称 | 内容摘要 | 受众 |
|-----|---------|---------|------|
| **必读** | [README.md](./README.md) | 项目概览、文档导航、维护说明 | 所有读者 |
| **必读** | [SUMMARY.md](./SUMMARY.md) | 全站导航、阅读路线 | 所有读者 |
| 01 | [01_Overview.md](./01_Overview.md) | 项目定位、功能边界、运行环境 | 新人学习者 |
| 02 | [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、线程模型 | 架构师、进阶开发者 |
| 03 | [03_CodeMap.md](./03_CodeMap.md) | 目录结构、代码导航 | 新接触代码的开发者 |
| 04 | [04_Interface.md](./04_Interface.md) | 对外接口、参数说明、调用示例 | 集成方开发者 |
| 05 | [05_AttackSurface.md](./05_AttackSurface.md) | 输入清单、敏感操作、信任边界 | 安全研究员 |
| 06 | [06_SecurityReview.md](./06_SecurityReview.md) | 风险分析、漏洞评估、修复建议 | 安全研究员 |
| 07 | [07_Build.md](./07_Build.md) | GN targets、编译产物、Feature 开关 | 构建工程师 |
| 08 | [08_Internals.md](./08_Internals.md) | 核心模块、资源生命周期 | 模块开发者 |

---

## 推荐阅读路线

### 路线一：新人学习路线 ⭐

**目标**：5 分钟理解项目，15 分钟定位代码，30 分钟掌握架构

| 顺序 | 文档 | 阅读时间 | 关键收获 |
|-----|------|---------|---------|
| 1 | [01_Overview.md](./01_Overview.md) | 5 分钟 | 项目定位、能做什么、怎么用 |
| 2 | [03_CodeMap.md](./03_CodeMap.md) | 10 分钟 | 代码在哪里、核心文件有哪些 |
| 3 | [04_Interface.md](./04_Interface.md) | 10 分钟 | API 调用方式、参数说明 |
| 4 | [02_Architecture.md](./02_Architecture.md) | 15 分钟 | 整体架构、数据流向 |
| 5 | [07_Build.md](./07_Build.md) | 5 分钟 | 编译方式、产物位置 |
| 6 | [08_Internals.md](./08_Internals.md) | 10 分钟 | 内部实现细节 |

**预计总时间**：55 分钟

---

### 路线二：安全研究路线 🔒

**目标**：5 分钟理解边界，15 分钟识别攻击面，30 分钟评估风险

| 顺序 | 文档 | 阅读时间 | 关键收获 |
|-----|------|---------|---------|
| 1 | [01_Overview.md](./01_Overview.md) | 5 分钟 | 项目定位、信任边界概览 |
| 2 | [05_AttackSurface.md](./05_AttackSurface.md) | 15 分钟 | 所有外部输入点、敏感操作 |
| 3 | [06_SecurityReview.md](./06_SecurityReview.md) | 15 分钟 | 风险点、利用路径、修复建议 |
| 4 | [04_Interface.md](./04_Interface.md) | 10 分钟 | API 接口契约、输入验证 |
| 5 | [02_Architecture.md](./02_Architecture.md) | 10 分钟 | 数据流、调用链 |
| 6 | [08_Internals.md](./08_Internals.md) | 10 分钟 | 权限校验、资源管理 |

**预计总时间**：65 分钟

---

## 按角色阅读（旧版索引，保留兼容）

### 👤 新接触项目的开发者
阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 了解项目背景
2. [03_CodeMap.md](./03_CodeMap.md) - 熟悉代码组织
3. [02_Architecture.md](./02_Architecture.md) - 理解架构设计
4. [04_Interface.md](./04_Interface.md) - 学习接口使用

### 👤 模块开发者
阅读顺序：
1. [02_Architecture.md](./02_Architecture.md) - 理解整体架构
2. [08_Internals.md](./08_Internals.md) - 掌握内部接口
3. [06_SecurityReview.md](./06_SecurityReview.md) - 了解安全约束
4. [07_Build.md](./07_Build.md) - 理解构建依赖

### 👤 安全审计人员
阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 了解项目范围
2. [02_Architecture.md](./02_Architecture.md) - 理解信任边界
3. [05_AttackSurface.md](./05_AttackSurface.md) - 详细攻击面分析
4. [06_SecurityReview.md](./06_SecurityReview.md) - 安全风险评估
5. [04_Interface.md](./04_Interface.md) - 检查接口安全

### 👤 构建/集成工程师
阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 了解项目定位
2. [07_Build.md](./07_Build.md) - 掌握构建系统
3. [08_Internals.md](./08_Internals.md) - 理解依赖关系

---

## 关键文档速查

| 需求 | 文档位置 |
|------|---------|
| 错误码速查 | [04_Interface.md#错误码定义](./04_Interface.md#错误码定义) |
| 调用链速查 | [02_Architecture.md#关键调用链](./02_Architecture.md#关键调用链) |
| 安全风险速查 | [06_SecurityReview.md#风险清单](./06_SecurityReview.md#风险清单) |
| 构建目标速查 | [07_Build.md#targets-列表](./07_Build.md#targets-列表) |

---

## 术语表

| 术语 | 说明 |
|------|------|
| DMS | Distributed Management Service，分布式管理服务 |
| FA | Feature Ability，元能力 |
| TLV | Type-Length-Value，一种数据编码格式 |
| SoftBus | 分布式软总线，提供设备间通信能力 |
| SAMGR | System Ability Manager，系统服务管理框架 |
| BMS | Bundle Manager Service，包管理服务 |
| DSoftBus | OpenHarmony 分布式软总线组件 |

---

## 相关资源

- [Wiki README](README.md) - Wiki 使用说明
- [工作笔记](_work/NOTES.md) - 生成过程中的事实记录
- [项目评估](_work/ASSESSMENT.md) - 项目类型与文档策略

---

## 版本信息

| 属性 | 值 |
|------|-----|
| **文档版本** | v1.0 |
| **生成日期** | 2026-02-07 |
| **代码版本** | OpenHarmony master |
| **最后更新** | 2026-02-07 |

---

## 贡献指南

### 发现问题？

| 问题类型 | 处理方式 |
|---------|---------|
| 文档与代码不符 | 提交 Issue 或 PR |
| 缺少关键信息 | 提交 Issue 说明需求 |
| 代码错误 | 直接提交 PR 修复 |

### 贡献方式

1. Fork 本 Wiki 仓库
2. 创建分支：`git checkout -b wiki-update`
3. 修改文档并提交
4. 创建 Pull Request

---

*最后更新：2026-02-07*
