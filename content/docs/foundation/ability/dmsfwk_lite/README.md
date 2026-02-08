# dmsfwk_lite Wiki

## 简介

本文档是 OpenHarmony **dmsfwk_lite**（轻量级分布式组件管理框架）的工程 Wiki，面向开发者、架构师和安全审计人员，提供从代码结构到安全分析的完整参考。

## 覆盖范围

| 文档 | 内容 | 目标读者 |
|------|------|----------|
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境 | 所有读者 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型、时序图 | 架构师、进阶开发者 |
| [03_CodeMap](03_CodeMap.md) | 目录结构、模块职责、代码导航 | 新接触代码的开发者 |
| [04_Interface](04_Interface.md) | 对外接口、错误码、使用示例 | 集成方开发者 |
| [05_AttackSurface](05_AttackSurface.md) | 攻击面、输入清单、信任边界 | 安全审计人员 |
| [06_SecurityReview](06_SecurityReview.md) | 风险点、利用路径、修复建议 | 安全审计人员 |
| [07_Build](07_Build.md) | 构建目标、依赖、产物 | 构建工程师 |
| [08_Internals](08_Internals.md) | 内部接口、模块依赖、生命周期 | 模块开发者 |

## 新人阅读路线

### 路线一：快速上手（15分钟）
1. [01_Overview](01_Overview.md) - 了解项目是什么、能做什么
2. [03_CodeMap](03_CodeMap.md) - 找到代码位置
3. [04_Interface](04_Interface.md) - 了解如何调用

### 路线二：安全研究（30分钟）
1. [01_Overview](01_Overview.md) - 了解项目定位
2. [05_AttackSurface](05_AttackSurface.md) - 识别攻击面
3. [06_SecurityReview](06_SecurityReview.md) - 评估安全风险

### 路线三：深度理解（1小时）
1. [01_Overview](01_Overview.md)
2. [02_Architecture](02_Architecture.md) - 理解架构设计
3. [08_Internals](08_Internals.md) - 理解内部实现
4. [06_SecurityReview](06_SecurityReview.md) - 理解安全机制

### 路线四：集成构建（30分钟）
1. [01_Overview](01_Overview.md)
2. [07_Build](07_Build.md) - 理解构建系统
3. [04_Interface](04_Interface.md) - 了解接口契约

## 更新方式

本文档基于代码生成，建议随代码迭代同步更新：

1. **代码变更时**: 同步更新相关 Wiki 页面
2. **接口变更时**: 必须更新 [04_Interface.md](04_Interface.md)
3. **架构变更时**: 必须更新 [02_Architecture.md](02_Architecture.md) 和 [08_Internals.md](08_Internals.md)
4. **构建变更时**: 必须更新 [07_Build.md](07_Build.md)
5. **安全修复时**: 必须更新 [05_AttackSurface.md](05_AttackSurface.md) 和 [06_SecurityReview.md](06_SecurityReview.md)

## 生成信息

- **生成时间**: 2026-02-07
- **代码版本**: OpenHarmony master (基于仓库 HEAD)
- **文档版本**: v1.0
- **维护者**: 工程 Wiki Agent

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

### 质量要求

1. 所有技术结论必须有代码证据支撑
2. 代码引用需标注文件路径和行号
3. 链接需验证有效性
4. 术语保持统一

## 参考链接

- [OpenHarmony 官方仓库](https://gitee.com/openharmony)
- [dmsfwk_lite 源码](https://gitee.com/openharmony/ability_dmsfwk_lite)

## 工作文件

| 文件 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据汇总（TODO） |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |
