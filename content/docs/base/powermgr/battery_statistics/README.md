# Battery Statistics 模块 Wiki

## 概述

本 Wiki 是 OpenHarmony `battery_statistics`（电池统计）模块的工程文档，旨在帮助开发者快速理解项目结构、架构设计、API 接口、编译构建和安全机制。

## 覆盖范围

| 文档 | 内容 |
|------|------|
| [README](README.md) | 本 Wiki 的使用说明、贡献指南 |
| [SUMMARY](SUMMARY.md) | 全站导航与新人阅读路线 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、运行环境 |
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型、时序图 |
| [03_NAPI](03_NAPI.md) | JS API 清单、参数、返回值、错误码 |
| [04_Inner_API](04_Inner_API.md) | Inner API、IPC 接口、内部模块 |
| [05_GN_Build](05_GN_Build.md) | BUILD.gn targets、依赖关系、feature flags |
| [06_Build_Outputs](06_Build_Outputs.md) | 编译产物、路径、运行时加载 |
| [07_Security](07_Security.md) | 安全风险分析、攻击面、修复建议 |
| [附录/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |

## 版本信息

| 属性 | 值 |
|------|-----|
| **模块版本** | 3.1 |
| **OpenHarmony 版本** | 标准系统 |
| **SysCap** | SystemCapability.PowerManager.BatteryStatistics |
| **文档生成时间** | 2026-02-06 |

## 如何更新文档

1. **修改代码后更新 Wiki**：
   - 若修改 N-API：更新 `03_NAPI.md`
   - 若修改构建配置：更新 `05_GN_Build.md` 和 `06_Build_Outputs.md`
   - 若修改安全机制：更新 `07_Security.md`

2. **添加新 API**：
   - 在 `03_NAPI.md` 添加 API 清单表
   - 在 `02_Architecture.md` 更新调用链
   - 在 `SUMMARY.md` 添加链接

3. **文档验证**：
   - 检查 SUMMARY.md 链接有效性
   - 验证所有代码证据（路径+符号）准确性
   - 确认无测试代码引用

## 贡献者

- Wiki 由 OpenHarmony 工程 Agent 自动生成
- 欢迎通过 PR 补充和修正文档内容
