# OpenHarmony Build Wiki

## 简介

本文档是 OpenHarmony **build** 仓库（编译构建子系统）的工程 Wiki，旨在帮助开发者快速理解和使用 OpenHarmony 的构建系统。

## 覆盖范围

本 Wiki 涵盖以下内容：

- **项目概览**: 定位、边界、核心能力
- **目录结构**: 各模块职责说明
- **架构说明**: 构建流程、数据流、关键时序
- **构建系统**: GN/Ninja 构建原理、工具链配置
- **GN Targets**: 模板定义、编译产物
- **安全风险**: 攻击面分析与修复建议
- **常见问题**: 构建/调试问题定位

## 未覆盖范围

- 测试相关内容（`test/` 目录）
- 特定产品的构建配置
- 第三方库详细说明

## 阅读顺序

建议新用户按以下顺序阅读：

1. [项目概览](01_Overview.md) - 了解项目定位和核心能力
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [构建系统详解](04_Build_System.md) - 理解构建原理
4. [GN Targets 梳理](05_GN_Targets.md) - 掌握模板使用
5. [安全风险评审](07_Security.md) - 了解安全注意事项

## 更新方式

本文档基于代码分析自动生成，建议随代码版本更新同步更新。

- **生成时间**: 2025-02-06
- **代码版本**: 4.0.2
- **仓库路径**: `//build`

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Build 仓库 README](README_zh.md)

---

*本 Wiki 由 OpenHarmony 工程 Agent 生成*
