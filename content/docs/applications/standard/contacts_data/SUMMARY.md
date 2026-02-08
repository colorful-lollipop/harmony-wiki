# contacts_data Wiki 导航

> 全站文档索引和阅读路线图

## 文档概览

本文档提供 contacts_data 子系统所有 Wiki 页面的索引，帮助开发者快速定位所需信息。

## 快速开始

### 如果你是新人

建议按照以下顺序阅读，建立对项目的完整认知：

1. **[项目概览](00_Overview.md)** - 了解项目定位和核心能力
2. **[目录结构](01_Directory_Structure.md)** - 掌握模块划分
3. **[架构设计](02_Architecture.md)** - 理解整体架构
4. **[API 参考](03_API_Reference.md)** - 学习接口使用
5. **[构建文档](04_Build.md)** - 了解编译流程

### 如果你有特定需求

| 需求 | 跳转页面 |
|------|----------|
| 查找 JS API | `03_API_Reference.md` → N-API 章节 |
| 查找 C++ 接口 | `03_API_Reference.md` → Inner API 章节 |
| 了解如何构建 | `04_Build.md` |
| 了解安全要求 | `05_Security_Review.md` |
| 查看调用关系 | `appendix/Callgraphs.md` |
| 查看配置参数 | `appendix/Config_Flags.md` |

## 文档索引

### 核心文档

| 文档 | 描述 | 适用角色 |
|------|------|----------|
| [README](README.md) | Wiki 使用指南和贡献说明 | 所有开发者 |
| [项目概览](00_Overview.md) | 项目定位、能力、运行环境 | 新人 |
| [目录结构](01_Directory_Structure.md) | 模块职责和文件组织 | 所有人 |
| [架构设计](02_Architecture.md) | 组件图、数据流、时序图 | 架构师 |
| [API 参考](03_API_Reference.md) | N-API 和 Inner API 详解 | 开发者 |
| [构建文档](04_Build.md) | GN targets 和编译产物 | 构建工程师 |
| [安全评审](05_Security_Review.md) | 安全风险和修复建议 | 安全工程师 |

### 附录

| 文档 | 描述 |
|------|------|
| [关键调用链](appendix/Callgraphs.md) | 入口→核心逻辑调用链 |
| [配置参数](appendix/Config_Flags.md) | 关键宏和 feature flags |

## 阅读路线图

### 路线一：JS 开发者快速上手

```
README.md → 00_Overview.md → 03_API_Reference.md (N-API)
```

目标：快速掌握 JS API 使用方法

### 路线二：C++ 开发者深入开发

```
README.md → 00_Overview.md → 01_Directory_Structure.md → 
02_Architecture.md → 03_API_Reference.md (Inner API)
```

目标：理解内部实现，进行二次开发

### 路线三：构建工程师

```
README.md → 04_Build.md → 01_Directory_Structure.md
```

目标：掌握构建配置和产物

### 路线四：安全工程师

```
README.md → 05_Security_Review.md → 02_Architecture.md
```

目标：理解安全边界和风险点

## 相关资源

### 内部资源

| 资源 | 链接 |
|------|------|
| 项目代码 | `//applications/standard/contacts_data` |
| 代码仓库 | [Gitee](https://gitee.com/openharmony/applications_contacts_data) |
| 工作笔记 | `_work/NOTES.md` |
| 生成计划 | `_work/PLAN.md` |

### 外部资源

| 资源 | 链接 |
|------|------|
| OpenHarmony 官网 | https://www.openharmony.cn/ |
| N-API 参考 | OpenHarmony N-API 文档 |
| GN 构建系统 | GN 构建文档 |

## 术语表

| 术语 | 解释 |
|------|------|
| N-API | Native API，OpenHarmony 提供的 JS 调用 C++ 的接口 |
| Inner API | 内部 C++ 接口，供模块间调用 |
| SA | System Ability，系统能力 |
| DataAbility | 数据访问能力 |
| DataShare | 数据共享机制 |

## 更新日志

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0.0 | 2024-XX-XX | 初始版本 |
