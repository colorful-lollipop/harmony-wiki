# 文档导航

本文档是 `hidumper_lite` 项目的完整技术 Wiki 导航页面。

## 项目概述

- [01_Overview.md](01_Overview.md) - 项目定位、边界、核心能力、运行环境

## 架构设计

- [02_Architecture.md](02_Architecture.md) - 组件图、数据流、线程模型、关键时序

## 接口文档

- [03_API.md](03_API.md) - 对外 API（N-API）、内部接口、平台适配接口

## 构建配置

- [04_Build.md](04_Build.md) - GN Targets、编译产物、安装路径

## 安全评审

- [05_Security.md](05_Security.md) - 攻击面、信任边界、风险与修复建议

## 问题排查

- [06_Troubleshooting.md](06_Troubleshooting.md) - 常见构建、运行、调试问题

## 附录

- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 关键配置项

---

## 双路线导航

### 路线一：新人学习路线

#### 快速上手（30 分钟）

1. 阅读 [01_Overview.md](01_Overview.md#项目定位) 了解项目定位（5 分钟）
2. 阅读 [03_API.md](03_API.md#命令行工具) 掌握命令行使用（10 分钟）
3. 参考 [06_Troubleshooting.md](06_Troubleshooting.md) 解决常见问题（15 分钟）

#### 深入开发（2 小时）

1. 阅读 [01_Overview.md](01_Overview.md) 完整章节（15 分钟）
2. 阅读 [02_Architecture.md](02_Architecture.md) 理解架构设计（30 分钟）
3. 阅读 [03_API.md](03_API.md) 掌握所有接口（30 分钟）
4. 阅读 [04_Build.md](04_Build.md) 熟悉编译配置（15 分钟）
5. 浏览 [05_Security.md](05_Security.md) 了解安全考量（15 分钟）
6. 参考 [06_Troubleshooting.md](06_Troubleshooting.md) 和附录（15 分钟）

### 路线二：安全研究路线

#### 快速评估（1 小时）

1. 阅读 [01_Overview.md](01_Overview.md#安全特性) 安全特性概述（10 分钟）
2. 精读 [05_Security.md](05_Security.md#攻击面分析) 攻击面分析（20 分钟）
3. 查阅 [05_Security.md](05_Security.md#安全风险清单) 安全风险清单（20 分钟）
4. 参考 [02_Architecture.md](02_Architecture.md#数据流) 数据流向（10 分钟）

#### 深度审计（2 小时）

1. 阅读 [01_Overview.md](01_Overview.md) 项目全貌（15 分钟）
2. 精读 [05_Security.md](05_Security.md) 完整安全评审（45 分钟）
3. 阅读 [02_Architecture.md](02_Architecture.md#信任边界) 信任边界分析（20 分钟）
4. 阅读 [03_API.md](03_API.md) 所有输入接口（20 分钟）
5. 对照源码验证风险点（20 分钟）

---

## 快速索引

### 按功能索引

| 功能 | 文档位置 | 关键章节 |
|------|----------|----------|
| 项目介绍 | 01_Overview.md | 项目定位、核心能力 |
| 使用方法 | 03_API.md | 命令行工具、AT 命令 |
| 适配新平台 | 03_API.md | 平台适配接口 |
| 编译项目 | 04_Build.md | GN Targets、编译产物 |
| 安全问题 | 05_Security.md | 攻击面、风险清单 |
| 问题定位 | 06_Troubleshooting.md | 常见问题、调试方法 |

### 按角色索引

| 角色 | 推荐阅读章节 |
|------|--------------|
| 新加入开发者 | 01_Overview.md → 02_Architecture.md → 03_API.md |
| 平台适配开发者 | 03_API.md#平台适配接口 → 附录/调用链 |
| 构建工程师 | 04_Build.md → 附录/配置项 |
| 安全审计人员 | 05_Security.md → 02_Architecture.md#数据流 |
| 问题排查人员 | 06_Troubleshooting.md → 附录/调用链 |

---

## 文档更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本，完成所有章节框架 |

---

## 返回首页

- [README.md](README.md) - Wiki 首页，包含覆盖范围和更新方式
