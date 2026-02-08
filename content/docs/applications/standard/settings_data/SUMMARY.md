# 文档导航

## 新人阅读路线

推荐阅读顺序：

1. **[首页](./index.md)** - 项目简介与快速开始
2. **[项目概览](./01_Project_Overview.md)** - 核心能力与技术选型
3. **[目录结构](./02_Directory_Structure.md)** - 代码组织方式
4. **[架构设计](./03_Architecture.md)** - DataAbility 架构详解
5. **[API 参考](./04_API_Reference.md)** - DataShare 接口配置
6. **[构建与编译](./05_Build_and_Compilation.md)** - hvigor 构建流程
7. **[安全评审](./06_Security_Review.md)** - 安全风险与修复建议

## 完整文档列表

### 入门指南

| 文档 | 说明 |
|------|------|
| [README](./README.md) | 文档覆盖范围与更新方式 |
| [首页](./index.md) | 项目快速介绍 |
| [项目概览](./01_Project_Overview.md) | 核心能力、运行环境、关键概念 |

### 架构与实现

| 文档 | 说明 |
|------|------|
| [目录结构](./02_Directory_Structure.md) | 模块划分与文件组织 |
| [架构设计](./03_Architecture.md) | 组件图、数据流、线程模型 |
| [API 参考](./04_API_Reference.md) | DataShare 接口与 URI 配置 |

### 工程实践

| 文档 | 说明 |
|------|------|
| [构建与编译](./05_Build_and_Compilation.md) | hvigor 构建配置与 HAP 产物 |
| [安全评审](./06_Security_Review.md) | 威胁模型与安全风险分析 |

## 代码证据索引

本文档所有关键结论均基于以下代码证据：

| 证据类型 | 位置 | 用途 |
|----------|------|------|
| DataAbility 核心逻辑 | `entry/src/main/ets/DataAbility/DataExtAbility.ets` | insert/update/query 实现 |
| 数据库操作 | `entry/src/main/ets/Utils/SettingsDBHelper.ets` | RDB 初始化与数据加载 |
| 模块配置 | `entry/src/main/module.json5` | ExtensionAbility 配置 |
| DataShare 配置 | `entry/src/main/resources/base/profile/data_share_config.json` | URI 与权限配置 |

## 术语表

| 术语 | 说明 |
|------|------|
| DataAbility | OpenHarmony 数据共享扩展能力 |
| DataShare | 数据共享框架 |
| RdbStore | 关系型数据库存储 |
| EL1/EL2 | 设备安全等级（EL2 为更高安全级别） |
| hvigor | OpenHarmony 构建系统 |
