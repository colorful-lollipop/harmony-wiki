# Security Component Manager - Wiki

> 工程文档仓库：OpenHarmony 安全组件管理服务

---

## 概述

本 Wiki 为 security_component_manager 项目提供完整的技术文档，覆盖：

- 项目定位与核心概念
- 架构设计与交互流程
- 对外 API 参考（C++ Native SDK）
- 内部模块接口与依赖关系
- GN 构建系统详解
- 编译产物与运行时加载
- 安全风险分析与评审

**重要提示**：本项目是纯 C++ System Ability 服务，**无 N-API 绑定**。JavaScript/ArkTS 应用通过 Ace Engine 的安全组件（PasteButton、SaveButton、LocationButton）间接调用本服务，不直接暴露 JS API。

---

## 快速开始

### 5 分钟了解项目

**[00_Overview](./00_Overview.md)** - 项目概览与核心概念

### 深入学习

按照新人入门路线阅读：

1. [00_Overview](./00_Overview.md) - 项目定位与边界
2. [01_Directory_Structure](./01_Directory_Structure.md) - 目录结构与模块职责
3. [02_Architecture](./02_Architecture.md) - 架构设计与数据流
4. [03_Public_APIs](./03_Public_APIs.md) - C++ SDK API 参考
5. [04_Internal_APIs](./04_Internal_APIs.md) - 内部 API 与模块接口
6. [05_GN_Targets](./05_GN_Targets.md) - GN 构建系统详解
7. [06_Build_Artifacts](./06_Build_Artifacts.md) - 编译产物与运行时加载
8. [07_Security_Analysis](./07_Security_Analysis.md) - 安全风险分析与评审

### 常见问题

[08_Common_Issues](./08_Common_Issues.md) - 常见构建、运行、调试问题

### 附录

- [附录：调用链分析](./appendix/Callgraphs.md) - 关键 API 调用链
- [附录：配置与开关](./appendix/Config_Flags.md) - 关键编译宏与特性开关

---

## 完整导航

[查看完整导航目录](./SUMMARY.md)

---

## 文档范围

### 已覆盖 ✅

- ✅ 项目核心功能与定位
- ✅ 目录结构与模块职责
- ✅ 三层架构设计（Ace UI → Security Component Service → Permission Manager）
- ✅ C++ Native SDK API 完整参考
- ✅ 内部模块接口与依赖关系
- ✅ GN 构建系统（targets、依赖、输出）
- ✅ 编译产物与安装路径
- ✅ 安全风险分析（攻击面、信任边界、可被利用点）
- ✅ 关键调用链
- ✅ 常见构建与调试问题

### 未覆盖 ⏳

- ⏳ **N-API/JS API 绑定** - 本项目为纯 C++ 服务，无 N-API。JS/ArkTS 应用通过 Ace Engine 的组件（PasteButton/SaveButton/LocationButton）调用，N-API 绑定在 arkui_ace_engine 仓库
- ⏳ **Ace Engine ArkTS 组件实现** - 位于 arkui_ace_engine 仓库
- ⏳ **Permission Manager 应用实现** - 位于 applications/standard/permission_manager
- ⏳ **厂商增强库开发指南** - 需要额外的厂商文档
- ⏳ **性能优化建议** - 需要进一步性能测试数据

---

## 更新信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于 OpenHarmony master 分支
- **维护方式**: 随代码更新同步更新 Wiki

---

## 如何贡献

当更新以下内容时，请同步更新 Wiki：

| 变更类型 | 更新页面 |
|---------|---------|
| 新增/修改 API | [03_Public_APIs](./03_Public_APIs.md) |
| 新增/删除文件 | [01_Directory_Structure](./01_Directory_Structure.md) |
| 架构变更 | [02_Architecture](./02_Architecture.md) |
| 新增/删除 target | [05_GN_Targets](./05_GN_Targets.md) |
| 新增/修复安全问题 | [07_Security_Analysis](./07_Security_Analysis.md) |

---

## 工作笔记

详细的探索过程、证据记录、疑问与 TODO 请查看：

[📝 wiki/_work/NOTES.md](./_work/NOTES.md) - 工作笔记与证据记录
[📋 wiki/_work/PLAN.md](./_work/PLAN.md) - 任务计划与进度跟踪

---

## 相关仓库

- [arkui_ace_engine](https://gitee.com/openharmony/arkui_ace_engine) - Ace Engine 实现（包含安全组件的 ArkTS 绑定）
- [applications_standard_permission_manager](https://gitee.com/openharmony/applications_standard_permission_manager) - Permission Manager 应用

---

## 许可证

本文档遵循与主项目相同的 Apache License 2.0 许可证。
