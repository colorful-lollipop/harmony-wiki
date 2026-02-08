# OpenHarmony appspawn 应用孵化器 Wiki

## 概述

本文档为 OpenHarmony `appspawn` 应用孵化器组件的工程Wiki，提供新人快速理解项目所需的所有技术文档。

**重要说明**: appspawn 是**纯Native C/C++系统服务**，**不提供JavaScript/N-API接口**。所有对外接口均为C语言Inner API，通过IPC/Socket调用。

## 覆盖范围

### 已覆盖内容
- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- Inner API 接口文档（Native C API）
- GN 构建系统与编译产物
- 安全风险评审
- 常见问题与调试指南

### 未覆盖内容
- 测试代码（按规范忽略）
- N/A (本组件无N-API)

## 更新方式

当代码发生以下变更时，需要同步更新Wiki：

| 变更类型 | 更新文档 |
|---------|---------|
| 新增/删除/修改Inner API | `02_API.md` |
| 新增/删除模块 | `01_Architecture.md`, `03_Build.md` |
| 修改构建配置 | `03_Build.md` |
| 新增安全机制 | `04_Security.md` |
| 新增Spawner类型 | `01_Architecture.md` |

## 文档索引

### 快速入门
- [README](README.md) - 本文档
- [SUMMARY](SUMMARY.md) - 全站导航

### 核心文档
1. [00_Overview.md](00_Overview.md) - 项目概览
2. [01_Architecture.md](01_Architecture.md) - 架构说明
3. [02_API.md](02_API.md) - Inner API 接口
4. [03_Build.md](03_Build.md) - 构建与编译产物
5. [04_Security.md](04_Security.md) - 安全风险评审
6. [05_Troubleshooting.md](05_Troubleshooting.md) - 常见问题

### 附录
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链

## 生成信息

- **代码版本**: 基于当前仓库
- **生成时间**: 2026-02-06
- **维护者**: OpenHarmony start subsystem
