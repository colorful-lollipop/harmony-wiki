# SUMMARY - Cast+ Stream Wiki 导航

> 本文档提供全站导航和新人推荐阅读顺序

---

## 推荐阅读路线

### 路线一：快速入门（15分钟）
适合：初次接触本项目，需要快速了解整体架构

1. [README.md](./README.md) - 了解文档覆盖范围和项目简介
2. [00_Overview.md](./00_Overview.md) - 项目概览与核心概念
3. [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构速览
4. [03_Architecture.md](./03_Architecture.md) - 架构设计（重点看组件图）

### 路线二：接口开发（30分钟）
适合：需要调用本模块接口的开发者

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [04_External_API.md](./04_External_API.md) - 对外 IPC API 详解
3. [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 编译产物与加载关系
4. [09_Troubleshooting.md](./09_Troubleshooting.md) - 常见问题

### 路线三：深度开发（60分钟）
适合：需要修改或扩展本模块的开发者

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [01_Project_Positioning.md](./01_Project_Positioning.md) - 项目定位与边界
3. [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
4. [03_Architecture.md](./03_Architecture.md) - 架构设计
5. [05_Internal_API.md](./05_Internal_API.md) - 内部模块接口
6. [06_GN_Targets.md](./06_GN_Targets.md) - GN 构建目标
7. [08_Security_Review.md](./08_Security_Review.md) - 安全风险评审
8. [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 关键调用链

### 路线四：安全审计（45分钟）
适合：进行安全评审或漏洞分析的开发者

1. [08_Security_Review.md](./08_Security_Review.md) - 安全风险评审
2. [05_Internal_API.md](./05_Internal_API.md) - 内部模块接口（关注信任边界）
3. [03_Architecture.md](./03_Architecture.md) - 架构设计（关注数据流）
4. [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 配置与宏定义

---

## 全站文档索引

### 核心文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [README.md](./README.md) | 文档首页 | 覆盖范围、更新方式、项目简介 |
| [SUMMARY.md](./SUMMARY.md) | 本文件 | 导航与阅读路线 |

### 概览文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [00_Overview.md](./00_Overview.md) | 项目概览 | 核心概念、运行环境、关键术语 |
| [01_Project_Positioning.md](./01_Project_Positioning.md) | 项目定位 | 功能边界、核心能力、非功能特性 |

### 架构文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构 | 模块职责、文件组织 |
| [03_Architecture.md](./03_Architecture.md) | 架构设计 | 组件图、数据流、线程模型、状态机 |

### 接口文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [04_External_API.md](./04_External_API.md) | 对外 API | IPC 接口清单、调用方式、权限要求 |
| [05_Internal_API.md](./05_Internal_API.md) | 内部 API | 模块接口、依赖方向、稳定性 |

### 构建文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [06_GN_Targets.md](./06_GN_Targets.md) | GN 构建目标 | Targets 列表、依赖关系、配置选项 |
| [07_Build_Artifacts.md](./07_Build_Artifacts.md) | 编译产物 | 产物清单、安装路径、加载关系 |

### 运维文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [08_Security_Review.md](./08_Security_Review.md) | 安全风险评审 | 攻击面、信任边界、可被利用点 |
| [09_Troubleshooting.md](./09_Troubleshooting.md) | 问题定位 | 常见问题、调试方法、日志分析 |

### 附录

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链 | 入口到核心逻辑的调用路径 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 配置与宏定义 | 关键宏、Feature Flags、配置项 |

---

## 关键术语速查

| 术语 | 说明 | 相关文档 |
|------|------|----------|
| Cast+ | OpenHarmony 投屏协议/框架 | [00_Overview.md](./00_Overview.md) |
| Mirror Cast | 镜像投屏模式 | [01_Project_Positioning.md](./01_Project_Positioning.md) |
| Stream Cast | 流媒体投屏模式 | [01_Project_Positioning.md](./01_Project_Positioning.md) |
| RTSP | 实时流协议，用于会话控制 | [03_Architecture.md](./03_Architecture.md) |
| SoftBus | OpenHarmony 分布式软总线 | [03_Architecture.md](./03_Architecture.md) |
| VTP | 虚拟传输协议 | [03_Architecture.md](./03_Architecture.md) |
| Session | 投屏会话 | [03_Architecture.md](./03_Architecture.md) |
| Source | 投屏源端（发送方） | [00_Overview.md](./00_Overview.md) |
| Sink | 投屏接收端（接收方） | [00_Overview.md](./00_Overview.md) |

---

## 文档维护

- **最后更新**: 2026-02-07
- **维护者**: 工程 Wiki 生成 Agent
- **更新方式**: 基于代码分析自动生成
- **本次更新**: 安全评审文档优化，补充代码证据
