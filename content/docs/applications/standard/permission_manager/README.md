# PermissionManager Wiki

> OpenHarmony 权限管理应用工程文档
> 生成时间: 2026-02-05

## 概述

本文档集为 PermissionManager 项目提供全面的工程分析，包括架构设计、API参考、安全评估和构建说明。

## 文档结构

### 入门指南
- [项目概览](00_Overview.md) - 项目定位、核心能力和运行环境
- [目录结构](01_Directory_Structure.md) - 源码组织方式和模块职责

### 架构与实现
- [架构说明](02_Architecture.md) - 组件图、数据流、线程模型、关键时序
- [对外API参考](03_API_Reference.md) - N-API/JS API、导出符号、权限参数
- [内部API说明](04_Inner_API.md) - 模块接口、依赖方向、稳定性

### 构建与部署
- [构建系统](05_Build_System.md) - GN Targets、编译产物、安装路径

### 安全与问题排查
- [安全风险评审](06_Security_Analysis.md) - 攻击面、信任边界、修复建议
- [常见问题排查](07_Troubleshooting.md) - 构建/运行/调试问题定位

### 附录
- [调用链](appendix/Callgraphs.md) - 关键调用链分析
- [配置标志](appendix/Config_Flags.md) - 关键宏和Feature Flags

## 快速导航

| 你想了解 | 阅读文档 |
|---------|---------|
| 这是什么项目？ | [00_Overview.md](00_Overview.md) |
| 代码怎么组织的？ | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构如何设计？ | [02_Architecture.md](02_Architecture.md) |
| 有哪些API？ | [03_API_Reference.md](03_API_Reference.md) |
| 如何构建？ | [05_Build_System.md](05_Build_System.md) |
| 安全吗？ | [06_Security_Analysis.md](06_Security_Analysis.md) |
| 遇到问题？ | [07_Troubleshooting.md](07_Troubleshooting.md) |

## 更新说明

本文档基于代码自动生成，最后更新时间: 2026-02-05

如需更新文档，请参考 `wiki/_work/` 目录下的工作笔记。
