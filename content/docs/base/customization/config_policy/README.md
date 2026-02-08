# Config Policy 组件 Wiki

## 概述

本 Wiki 是 OpenHarmony `config_policy` 配置策略组件的技术文档，面向希望理解、使用或扩展该组件的开发者。

### 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目定位与架构 | ✅ | 组件职责、边界、核心能力、跨平台支持 |
| 目录结构 | ✅ | 各模块职责划分（不含测试） |
| C++ 内部 API | ✅ | `config_policy_utils.h` 接口 |
| N-API 接口 | ✅ | JS/ETS 接口定义与绑定 |
| GN 构建配置 | ✅ | targets、依赖、产物 |
| 攻击面分析 | ✅ | 外部输入清单、敏感操作、信任边界（新增） |
| 安全风险评审 | ✅ | 攻击面与风险点分析 |

### 未覆盖范围

- 测试代码相关文档（单元测试、模糊测试等）
- 详细的运行时配置参数（由系统其他组件管理）

### 最近更新

**更新时间**：2026-02-07

**改进内容**：
1. 改进 01_Overview.md：添加能力边界、语言绑定对比、跨平台支持、性能说明
2. 创建 08_AttackSurface.md：独立的攻击面分析文档，包含信任边界图和攻击路径
3. 改进 SUMMARY.md：添加安全研究员路线，提供双路线导航（新人学习 vs 安全研究）

### 更新方式

当代码发生以下变更时，需要同步更新 Wiki：
1. 新增/删除/修改 N-API 接口
2. 修改 GN 构建配置（新增 target、改变依赖等）
3. 发现新的安全风险或修复现有问题

### 文档结构

| 文档 | 路径 | 说明 |
|------|------|------|
| 项目概述 | [01_Overview.md](./01_Overview.md) | 组件定位、能力边界、快速开始 |
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) | 代码组织方式和文件定位 |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) | 组件架构、数据流、FollowX 机制 |
| N-API 接口 | [04_N-API_Reference.md](./04_N-API_Reference.md) | JavaScript N-API 接口详细说明 |
| C++ 内部 API | [05_Cpp_Inner_API.md](./05_Cpp_Inner_API.md) | C/C++ 内部 API 详细说明 |
| GN 构建 | [06_GN_Build.md](./06_GN_Build.md) | 构建目标、依赖、编译产物 |
| 攻击面分析 | [08_AttackSurface.md](./08_AttackSurface.md) | 外部输入清单、敏感操作、信任边界 |
| 安全风险评审 | [07_Security_Review.md](./07_Security_Review.md) | 安全分析与修复建议 |
| 导航 | [SUMMARY.md](./SUMMARY.md) | 双路线导航（新人学习 vs 安全研究） |

### 工作区

| 文档 | 路径 | 说明 |
|------|------|------|
| 项目评估 | [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目类型判定、受众分析、文档策略 |
| 代码证据 | [_work/NOTES.md](./_work/NOTES.md) | 代码证据汇总和关键发现 |
