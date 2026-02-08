# Pasteboard 剪贴板服务 Wiki

## 概述

本文档是 OpenHarmony Pasteboard（剪贴板）组件的工程 Wiki，旨在帮助开发者快速理解项目架构、接口设计、构建系统以及安全风险。

## 覆盖范围

本文档涵盖以下内容：

- **项目概览**: 定位、边界、核心能力、运行环境
- **架构说明**: 组件图、数据流、线程模型、关键时序
- **目录结构**: 模块职责、文件组织
- **N-API 参考**: JS API 清单、参数校验、调用链
- **内部 API**: 模块接口、依赖方向、稳定性
- **GN 构建**: Targets、依赖关系、编译产物
- **安全评审**: 攻击面、风险点、修复建议
- **常见问题**: 构建/运行/调试问题定位

## 更新方式

本文档基于代码分析自动生成，建议随代码迭代同步更新：

1. **接口变更**: 修改 N-API 或 IPC 接口时，同步更新 `03_NAPI_Reference.md` 和 `04_Inner_API.md`
2. **架构调整**: 模块重构时，更新 `01_Architecture.md` 和 `02_Directory_Structure.md`
3. **新增依赖**: 添加外部依赖时，更新 `05_GN_Targets.md`
4. **安全修复**: 修复安全问题时，更新 `06_Security.md`

## 生成信息

- **生成时间**: 2025-02-06
- **代码版本**: OpenHarmony master (commit: 待确认)
- **分析范围**: `/foundation/distributeddatamgr/pasteboard`
- **排除内容**: test/、unittest/、fuzztest/ 等测试代码

## 阅读指南

**新人推荐阅读顺序**:
1. [概览](00_Overview.md) - 了解项目基本信息
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [架构说明](01_Architecture.md) - 理解系统架构
4. [N-API 参考](03_NAPI_Reference.md) - 了解对外接口
5. [安全评审](06_Security.md) - 理解安全模型

**快速查阅**:
- [SUMMARY.md](SUMMARY.md) - 全站导航
- [GN 构建](05_GN_Targets.md) - 构建配置
- [问题排查](07_Troubleshooting.md) - 常见问题

## 术语表

| 术语 | 说明 |
|------|------|
| SA | System Ability，系统能力，OpenHarmony 的系统服务框架 |
| IPC | Inter-Process Communication，进程间通信 |
| N-API | Node-API，用于 JS/TS 调用 C/C++ 代码的接口 |
| NDK | Native Development Kit，原生开发工具包 |
| TLV | Type-Length-Value，一种数据序列化格式 |
| DLP | Data Loss Prevention，数据防泄漏 |
| UDMF | Unified Data Management Framework，统一数据管理框架 |
| FFRT | Foundation Function Runtime，基础功能运行时 |

## 参考链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [Pasteboard API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-pasteboard.md)
- [源码仓库](https://gitee.com/openharmony/distributeddatamgr_pasteboard)

---

**注意**: 本文档所有结论均基于代码分析，关键证据已标注文件路径和行号。如有疑问，请以代码为准。
