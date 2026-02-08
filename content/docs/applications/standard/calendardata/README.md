# CalendarData Wiki

本文档为 OpenHarmony CalendarData 组件的工程 Wiki。

## 概述

CalendarData 是 OpenHarmony 系统中的预置应用，提供日历数据的增删改查能力。本文档面向开发者，提供完整的技术架构说明，包括对外 API、内部实现、构建系统和安全风险分析。

## 覆盖范围

### 已覆盖内容

✅ 目录结构与模块职责
✅ 对外 N-API 清单
✅ 权限机制说明
✅ GN 构建系统概览
✅ 数据流与架构说明
✅ 安全风险初步分析

### 未覆盖内容

⚠️ 详细错误码文档
⚠️ 完整的时序图
⚠️ 性能优化指南
⚠️ 第三方集成指南

## 如何使用本文档

### 新人阅读顺序

建议按以下顺序阅读文档：

1. [项目概览](00_Overview.md) - 了解项目定位和核心概念
2. [目录结构](01_Directory_Structure.md) - 熟悉代码组织方式
3. [架构说明](02_Architecture.md) - 理解组件关系和数据流
4. [对外 API](03_External_API.md) - 学习如何调用日历 API
5. [内部 API](04_Internal_API.md) - 深入理解内部实现
6. [GN 构建](05_GN_Build.md) - 了解构建系统
7. [编译产物](06_Build_Artifacts.md) - 了解运行时组件
8. [安全评审](07_Security_Review.md) - 了解安全考虑

### 按主题查阅

- **API 调用**: 查看 [对外 API](03_External_API.md)
- **调试问题**: 查看 [FAQ](08_FAQ.md)
- **安全审计**: 查看 [安全评审](07_Security_Review.md)
- **构建问题**: 查看 [GN 构建](05_GN_Build.md) 和 [编译产物](06_Build_Artifacts.md)

## 文档维护

### 生成时间

本文档于 2026-02-05 生成。

### 如何随代码更新文档

当代码发生重大变更时，建议按以下步骤更新文档：

1. 更新 `wiki/_work/NOTES.md` - 记录新的发现和变更
2. 更新对应的文档章节
3. 更新 `wiki/SUMMARY.md` - 如有新增章节
4. 验证所有链接有效
5. 更新本文档的"生成时间"

### 贡献指南

如果您发现文档错误或遗漏：

1. 验证问题确实存在于代码中
2. 更新相关文档章节
3. 在 `wiki/_work/NOTES.md` 中记录变更原因和证据
4. 提交变更并附上说明

## 版本信息

| 项目 | 值 |
|------|-----|
| 包名 | @ohos/calendar_data |
| 版本 | 3.1 |
| 子系统 | applications |
| 系统能力 | SystemCapability.Applications.CalendarData |
| API 级别 | API 10 |
| 开发语言 | ArkTS, C/C++ |
| 构建系统 | GN |
| 许可证 | Apache License 2.0 |

## 联系方式

如有问题或建议，请联系项目维护者。

---

**注意**: 本文档基于代码静态分析生成，部分实现细节可能随版本演进发生变化。使用时请以实际代码为准。
