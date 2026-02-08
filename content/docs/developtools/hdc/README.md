# hdc 工程文档

## 概述

本文档集为 OpenHarmony hdc 项目提供完整的工程化文档，旨在帮助新开发者快速理解项目架构、API 接口、构建系统和安全风险。

- **项目名称**: hdc (OpenHarmony Device Connector)
- **仓库**: developtools/hdc
- **版本**: 3.1
- **许可证**: Apache License 2.0
- **子系统**: developtools
- **组件**: hdc

## 文档覆盖范围

本文档覆盖以下内容：

- ✅ 项目定位与边界
- ✅ 目录结构与模块职责
- ✅ 架构说明（组件图、数据流、线程模型、关键时序）
- ✅ 对外 API（JDWP 注册）
- ✅ 内部 API（模块接口、依赖方向、稳定性）
- ✅ GN 目标梳理（targets 列表、类型、依赖、产物、开关）
- ✅ 编译产物（.so/.a/.hap/可执行文件、安装路径、运行时加载关系）
- ✅ 安全风险评审（攻击面、信任边界、可被利用点）
- ✅ 常见构建/运行/调试问题

## 未覆盖范围

- ❌ 测试用例和测试框架（test/ 目录）
- ❌ 其他 OpenHarmony 组件（Ability 框架、系统服务等）
- ❌ 用户使用手册（见 README_zh.md）

## 文档更新方式

本文档基于代码证据生成，所有结论均可追溯到具体文件路径和代码行号。

**更新建议**：
1. 代码变更后，同步更新相关文档章节
2. 使用 `git blame` 查找代码变更对应的提交者
3. 检查文档中的代码片段是否仍与最新代码一致

## 生成信息

- **生成时间**: 2026-02-06 09:48
- **生成工具**: OpenHarmony Wiki 生成 Agent
- **基于代码版本**: 当前 master 分支

---

## 快速导航

- [完整目录导航](./SUMMARY.md) - 按阅读顺序组织
- [项目概览](./00_Overview.md) - 项目定位、核心能力、运行环境
- [目录结构](./02_Directory_Structure.md) - 模块职责、文件组织
- [架构说明](./03_Architecture.md) - 组件图、数据流、线程模型
- [对外 API](./04_External_API.md) - JDWP 注册、API 清单
- [内部 API](./05_Internal_API.md) - 模块接口、依赖方向
- [GN 目标](./06_GN_Targets.md) - 构建系统、targets、产物
- [编译产物](./07_Build_Artifacts.md) - 安装路径、运行时加载
- [安全评审](./08_Security_Review.md) - 攻击面、风险点、修复建议
- [常见问题](./09_FAQ.md) - 构建/运行/调试问题

## 工作笔记

详细的探索记录、代码证据、待确认事项，请查看 [wiki/_work/NOTES.md](./_work/NOTES.md)。

## 任务计划

详细的执行计划和进度跟踪，请查看 [wiki/_work/PLAN.md](./_work/PLAN.md)。
