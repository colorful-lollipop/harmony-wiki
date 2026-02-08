# 后台任务管理模块 Wiki

## 文档概述

本Wiki提供OpenHarmony后台任务管理模块(background_task_mgr)的完整工程文档，帮助开发者快速理解项目架构、接口使用和安全风险。

## 覆盖范围

✅ **已覆盖内容**：
- 项目定位与核心能力
- 目录结构与模块职责
- N-API对外接口（短时任务、长时任务、能效资源）
- 内部架构与组件关系
- GN构建目标与编译产物
- 安全风险评审

❌ **未覆盖内容**：
- 详细的性能优化建议
- 各芯片平台适配细节
- 历史版本变更记录

## 阅读顺序（新人推荐）

1. **[首页概览](index.md)** - 了解模块定位和核心能力
2. **[架构说明](02_Architecture.md)** - 理解组件关系和数据流
3. **[对外N-API](03_NAPI_Reference.md)** - 学习JS/ArkTS接口使用
4. **[内部API](04_Inner_API.md)** - 了解C++内部接口
5. **[GN构建](05_GN_Build.md)** - 掌握构建配置
6. **[安全风险](06_Security.md)** - 了解安全注意事项

## 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02-06 | v1.0 | 初始版本创建 |

## 如何更新本文档

当代码发生变更时，请同步更新以下文件：
1. `wiki/_work/NOTES.md` - 记录新发现的事实
2. 相应专题文档 - 更新具体技术内容
3. `wiki/README.md` - 更新版本号和更新记录

## 相关资源

- 代码仓库：`foundation/resourceschedule/background_task_mgr`
- 子系统：ResourceSchedule（资源调度）
- SystemCapability：
  - SystemCapability.ResourceSchedule.BackgroundTaskManager.ContinuousTask
  - SystemCapability.ResourceSchedule.BackgroundTaskManager.TransientTask
  - SystemCapability.ResourceSchedule.BackgroundTaskManager.EfficiencyResourcesApply
