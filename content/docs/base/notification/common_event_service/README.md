# 公共事件服务 Wiki

## 概述

本文档是 OpenHarmony 公共事件服务（Common Event Service, CES）的工程 Wiki，旨在帮助开发者快速理解项目架构、核心能力、接口规范及安全风险。

## 覆盖范围

本文档覆盖以下内容：

| 模块 | 说明 |
|------|------|
| [概览](00_Overview.md) | 项目定位、核心能力、运行环境 |
| [架构](01_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [N-API](02_N-API.md) | JS API 接口清单、参数校验、调用链 |
| [内部 API](03_Inner_API.md) | Native 接口、模块依赖、生命周期 |
| [构建与编译](04_Build.md) | GN Targets、编译产物、依赖关系 |
| [安全评审](05_Security.md) | 攻击面、风险点、修复建议 |
| [常见问题](06_FAQ.md) | 构建、运行、调试问题定位 |

## 更新方式

当代码发生以下变更时，需要同步更新 Wiki：

| 变更类型 | 需更新文档 |
|----------|------------|
| 新增/删除 N-API | `02_N-API.md` |
| 修改架构/模块 | `01_Architecture.md` |
| 修改构建配置 | `04_Build.md` |
| 新增安全风险 | `05_Security.md` |

**更新步骤**：
1. 修改对应 Wiki 文件
2. 确保所有 API 都有代码证据（路径 + 符号）
3. 验证 SUMMARY.md 链接有效性

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于当前仓库 HEAD
- **文档语言**: 中文（默认）

## 相关链接

- [OpenHarmony 通知子系统仓](https://gitee.com/openharmony/notification_common_event_service)
- [OpenHarmony 官方文档](https://docs.openharmony.cn)
