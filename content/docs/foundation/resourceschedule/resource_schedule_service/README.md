# Resource Schedule Service Wiki

## 简介

本文档是 OpenHarmony **resource_schedule_service**（资源调度服务）的完整工程 Wiki，基于代码分析生成，覆盖架构、接口、构建和安全等方面。

## 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ | 定位、边界、核心能力 |
| 目录结构 | ✅ | 非测试代码目录职责 |
| 架构说明 | ✅ | 组件图、数据流、线程模型 |
| N-API 接口 | ✅ | @ohos.resourceschedule.systemload<br>@ohos.resourceschedule.backgroundProcessManager |
| 内部 API | ✅ | C++ 客户端接口、服务接口 |
| GN 构建 | ✅ | Targets、依赖、产物 |
| 安全风险 | ✅ | 攻击面、可利用点分析 |
| 常见问题 | ✅ | 构建/运行/调试 |

## 更新方式

本文档通过代码分析自动生成。当代码发生变化时：

1. 检查 `wiki/_work/` 目录下的工作笔记
2. 更新相关章节
3. 验证链接有效性

## 生成信息

- **生成日期**: 2025-02-06
- **代码版本**: OpenHarmony 3.1+
- **仓库路径**: foundation/resourceschedule/resource_schedule_service

## 快速导航

- [总览](00_Overview.md) - 项目定位与核心概念
- [架构设计](01_Architecture.md) - 组件图与数据流
- [目录结构](02_Directory_Structure.md) - 代码组织
- [N-API 参考](03_NAPI_Reference.md) - JS API 详情
- [内部 API](04_Inner_API.md) - C++ 接口
- [GN 构建](05_GN_Targets.md) - 编译目标
- [编译产物](06_Build_Artifacts.md) - 输出文件
- [安全分析](07_Security_Analysis.md) - 风险评审
- [问题排查](08_Troubleshooting.md) - FAQ

## 附录

- [调用链](appendix/Callgraphs.md)
- [配置标志](appendix/Config_Flags.md)

---

> ⚠️ **注意**: 本文档基于源代码分析生成，所有结论均有代码证据支撑。如发现与代码不符之处，请以代码为准。
