# FaultLoggerd Wiki

本文档为 OpenHarmony faultloggerd 组件的技术文档，帮助开发者快速理解项目架构、API 接口、构建系统及安全特性。

## 文档概览

| 文档 | 说明 |
|------|------|
| [项目定位与核心能力](00_Overview.md) | 项目定位、边界、核心能力、运行环境 |
| [目录结构与模块职责](01_Directory_Structure.md) | 项目目录结构及各模块职责 |
| [架构设计](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [对外 API](03_External_API.md) | Native API 接口、参数、错误码 |
| [内部 API](04_Inner_API.md) | 模块接口、依赖关系、稳定性说明 |
| [GN 构建系统](05_GN_Targets.md) | 构建目标、依赖、产物、编译选项 |
| [编译产物](06_Build_Artifacts.md) | 产物清单、安装路径、运行时加载 |
| [攻击面分析](05_AttackSurface.md) | 外部输入入口、敏感操作、信任边界图 |
| [安全风险评审](07_Security_Review.md) | 风险分析、可被利用点、安全建议 |

## 重要说明

- **无 N-API 接口**：faultloggerd 是纯 C/C++ 原生服务，不提供 JavaScript/N-API 绑定
- **通信方式**：使用 Unix Domain Socket 通信，而非传统 IPC/SA
- **目标场景**：进程崩溃日志生成、堆栈抓取、故障诊断

## 更新记录

- 最新更新时间：2026-02-07
- 生成时间：2026-02-06
- 基于代码版本：main 分支
- 覆盖范围：完整生产代码（不含测试）
- 新增内容：攻击面分析文档（安全研究员专用）
