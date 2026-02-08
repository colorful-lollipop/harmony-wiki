# OpenHarmony IPC 组件 Wiki

## 概述

本文档为 OpenHarmony `foundation/communication/ipc` 组件的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 用法、构建方式以及安全注意事项。

**适用读者**：
- 需要使用 IPC/RPC 进行跨进程通信的应用开发者
- 需要维护或扩展 IPC 框架的系统开发者
- 需要进行安全评审的审计人员

## 覆盖范围

### 已覆盖内容

| 文档 | 内容 |
|------|------|
| [README](README.md) | 本 Wiki 使用说明与更新方式 |
| [SUMMARY](SUMMARY.md) | 全站导航与新人阅读路线 |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、约束与运行环境 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块职责划分 |
| [03_Architecture](03_Architecture.md) | 架构图、数据流、线程模型、关键时序 |
| [04_N-API](04_N-API.md) | JS/ArkTS N-API 接口清单与用法 |
| [05_Inner_API](05_Inner_API.md) | Native Inner API 与模块依赖 |
| [06_Build](06_Build.md) | GN Targets、编译产物、Feature Flags |
| [07_Security](07_Security.md) | 攻击面分析、风险清单、修复建议 |
| [08_Troubleshooting](08_Troubleshooting.md) | 常见问题、调试方法、日志定位 |

### 未覆盖内容

- 测试用例与测试方法（见 `test/` 目录）
- 特定设备平台的适配细节
- 性能基准测试数据

## 代码证据原则

本文档遵循**证据驱动**原则：
- 所有关键结论均标注代码来源（文件路径 + 行号）
- 接口定义均给出具体头文件位置
- 无法确认的信息标注 `TODO(需确认)` 并说明缺少的证据

示例：
```
- IPCSkeleton::GetCallingPid() 获取调用方 PID
- 证据：`interfaces/innerkits/ipc_core/include/ipc_skeleton.h:61`
```

## 更新方式

### 何时更新 Wiki

当发生以下变更时，应同步更新 Wiki：

1. 新增、删除或修改 N-API 接口
2. 改变构建配置或产物路径
3. 新增安全敏感功能或修改权限校验逻辑
4. 修改核心架构或模块依赖关系
5. 新增 Feature Flags 或修改其默认行为

### 更新步骤

1. 更新 `wiki/_work/NOTES.md` 添加发现的事实
2. 修改对应的 Wiki 文档
3. 更新 `wiki/SUMMARY.md`（如有必要）
4. 执行 Phase 7 一致性校验

## 快速入口

| 任务 | 跳转 |
|------|------|
| 快速入门 IPC 开发 | [01_Overview.md](./01_Overview.md) |
| 查找 JS API | [04_N-API.md](./04_N-API.md) |
| 理解编译配置 | [06_Build.md](./06_Build.md) |
| 安全开发指南 | [07_Security.md](./07_Security.md) |

---

*文档生成时间：2026-02-06*
*代码版本：OpenHarmony IPC Component v3.0*
