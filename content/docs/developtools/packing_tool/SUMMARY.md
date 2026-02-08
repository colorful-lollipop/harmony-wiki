# SUMMARY - 全站导航

## 快速导航

- [首页](index.md)
- [项目概述](01_Overview.md)
- [架构说明](02_Architecture.md)
- [目录结构](03_Directory_Structure.md)
- [对外 API 参考](04_API_Reference.md)
- [内部 API 说明](05_Inner_API.md)
- [GN 构建目标](06_GN_Targets.md)
- [编译产物说明](07_Build_Artifacts.md)
- [安全风险评审](08_Security_Review.md)
- [常见问题与调试](09_Troubleshooting.md)

## 附录

- [关键调用链](appendix/Callgraphs.md)
- [配置标志说明](appendix/Config_Flags.md)

---

## 新人阅读路线

### 第一阶段：快速入门（15 分钟）

1. [首页](index.md) - 了解项目是什么、能做什么
2. [项目概述 - 核心能力](01_Overview.md#核心能力) - 了解支持的包类型
3. [对外 API 参考 - 快速开始](04_API_Reference.md#快速开始) - 学习基本用法

### 第二阶段：理解架构（30 分钟）

1. [架构说明 - 组件架构图](02_Architecture.md#组件架构图) - 理解整体架构
2. [目录结构](03_Directory_Structure.md) - 熟悉代码组织方式
3. [架构说明 - 数据流](02_Architecture.md#数据流) - 理解包处理流程

### 第三阶段：实践应用（按需阅读）

- **打包功能**: [对外 API - 打包接口](04_API_Reference.md#打包接口)
- **拆包功能**: [对外 API - 拆包接口](04_API_Reference.md#拆包接口)
- **解析功能**: [对外 API - 解析接口](04_API_Reference.md#解析接口)
- **扫描功能**: [对外 API - 扫描接口](04_API_Reference.md#扫描接口)

## 开发者深入路线

### 架构与设计

1. [内部 API - 核心类设计](05_Inner_API.md#核心类设计)
2. [内部 API - 数据模型](05_Inner_API.md#数据模型)
3. [附录 - 关键调用链](appendix/Callgraphs.md)

### 构建与部署

1. [GN 构建目标](06_GN_Targets.md) - 理解构建系统
2. [编译产物说明](07_Build_Artifacts.md) - 了解输出产物
3. [附录 - 配置标志](appendix/Config_Flags.md)

### 安全与质量

1. [安全风险评审 - 攻击面分析](08_Security_Review.md#攻击面分析)
2. [安全风险评审 - 可利用点](08_Security_Review.md#可被利用点)
3. [安全风险评审 - 修复建议](08_Security_Review.md#修复建议)

## 问题排查速查

| 问题类型 | 参考文档 |
|---------|---------|
| 打包失败 | [常见问题 - 打包问题](09_Troubleshooting.md#打包问题) |
| 拆包失败 | [常见问题 - 拆包问题](09_Troubleshooting.md#拆包问题) |
| 验证失败 | [常见问题 - 验证问题](09_Troubleshooting.md#验证问题) |
| 构建问题 | [常见问题 - 构建问题](09_Troubleshooting.md#构建问题) |
| 性能问题 | [常见问题 - 性能优化](09_Troubleshooting.md#性能优化) |

## 术语表

| 术语 | 说明 |
|-----|------|
| HAP | HarmonyOS Ability Package，应用能力包 |
| HSP | HarmonyOS Shared Package，共享包 |
| APP | 多 HAP/HSP 组合的应用包 |
| HQF | HarmonyOS Quick Fix，快速修复包 |
| APPQF | 多 HQF 组合的快速修复包 |
| HAR | HarmonyOS Archive，静态共享库 |
| Stage 模型 | OpenHarmony 应用开发模型（推荐） |
| FA 模型 | OpenHarmony 传统应用开发模型 |
