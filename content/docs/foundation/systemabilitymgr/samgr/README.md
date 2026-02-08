# Samgr - System Ability Manager Wiki

## 简介

本文档是 OpenHarmony System Ability Manager (Samgr) 组件的完整工程 Wiki，旨在帮助开发者快速理解项目架构、接口定义、构建系统和安全风险。

## 覆盖范围

本 Wiki 涵盖以下内容：

| 文档 | 内容描述 |
|------|----------|
| [项目概览](00_Overview.md) | 项目定位、核心能力、运行环境 |
| [目录结构](01_Directory_Structure.md) | 代码组织、模块职责 |
| [架构设计](02_Architecture.md) | 组件图、数据流、线程模型、时序图 |
| [对外 API](03_Public_API.md) | C++ 接口、Rust 绑定、API 清单 |
| [内部实现](04_Internal_Architecture.md) | 核心模块、接口依赖、状态机 |
| [GN 构建](05_GN_Targets.md) | 构建目标、依赖关系、配置选项 |
| [编译产物](06_Build_Artifacts.md) | 输出文件、安装路径、运行时加载 |
| [安全分析](07_Security_Analysis.md) | 攻击面、风险点、修复建议 |
| [问题排查](08_Troubleshooting.md) | 常见问题、调试方法、定位路径 |

## 更新说明

- **生成时间**: 2025-02-06
- **代码版本**: 基于仓库当前 HEAD
- **更新方式**: 当代码变更时，需同步更新相关文档

## 术语说明

| 术语 | 说明 |
|------|------|
| SA | System Ability，系统能力/系统服务 |
| SA ID | System Ability ID，系统能力标识符 |
| Samgr | System Ability Manager，系统能力管理器 |
| IPC | Inter-Process Communication，进程间通信 |
| DBinder | Distributed Binder，分布式 Binder |
| FFRT | Fast Function Runtime，快速函数运行时 |

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony)
- [SA 框架仓库](https://gitee.com/openharmony/systemabilitymgr_safwk)
- [Samgr 仓库](https://gitee.com/openharmony/systemabilitymgr_samgr)
