# Telephony Core Service Wiki

## 简介

本 Wiki 面向 OpenHarmony `telephony_core_service` 模块，提供完整的项目工程文档，帮助新人快速理解项目架构、接口定义、编译系统和安全风险。

## 覆盖范围

- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构设计（组件图、数据流、线程模型）
- ✅ 对外 API（N-API / JS API）
- ✅ 内部 API（Inner Kits）
- ✅ GN 构建目标与编译产物
- ✅ 安全风险评审（基于代码证据）

## 未覆盖范围

- ❌ 测试代码（test/ 目录下的所有内容）
- ❌ 具体业务逻辑的详细实现
- ❌ 性能优化指南

## 阅读顺序建议

1. [项目概览](./00_Overview.md) - 了解项目定位和边界
2. [目录结构](./01_Directory_Structure.md) - 熟悉代码组织
3. [架构设计](./02_Architecture.md) - 理解组件关系和数据流
4. [N-API 接口](./03_NAPI_API.md) - JS API 清单和调用链
5. [内部 API](./04_Inner_API.md) - Native 接口定义
6. [GN 构建](./05_GN_Build.md) - 编译目标和产物
7. [安全风险](./06_Security.md) - 攻击面和风险点

## 更新维护

- **生成时间**: 2026-02-06
- **代码版本**: 基于仓库 HEAD 版本生成
- **更新方式**: 当代码结构或接口发生变更时，需要同步更新本文档

## 术语说明

| 术语 | 说明 |
|------|------|
| SA | System Ability，系统能力，OpenHarmony 的 IPC 服务框架 |
| N-API | Node-API，用于 C/C++ 编写 Node.js 原生扩展的 API |
| RIL | Radio Interface Layer，无线接口层 |
| IMS | IP Multimedia Subsystem，IP多媒体子系统 |
| eSIM | Embedded SIM，嵌入式 SIM 卡 |
| FFRT | Foundation Function Runtime，基础功能运行时 |

## 参考文档

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Telephony 子系统 README](../README.md)
- [Telephony 子系统 README(中文)](../README_zh.md)
