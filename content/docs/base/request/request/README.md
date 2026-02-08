# @ohos/request Wiki

**最后更新**: 2026-02-07  
**版本**: 3.1  
**维护者**: OpenHarmony Request Team

---

## 概述

本 Wiki 为 OpenHarmony `@ohos/request` 模块的完整技术文档，面向两类核心受众：

| 受众 | 目标 | 推荐阅读 |
|------|------|----------|
| **新人学习者** | 快速理解项目定位、掌握 API 使用、理解架构 | [01_Overview.md](01_Overview.md) → [03_CodeMap.md](03_CodeMap.md) → [04_Interface.md](04_Interface.md) |
| **安全研究员** | 识别攻击面、分析信任边界、评估漏洞风险 | [05_AttackSurface.md](05_AttackSurface.md) → [06_SecurityReview.md](06_SecurityReview.md) |

---

## 文档导航

### 新人学习路线

1. **[项目概览](01_Overview.md)** - 了解 Request 是什么、能做什么
2. **[代码地图](03_CodeMap.md)** - 找到核心代码位置
3. **[对外接口](04_Interface.md)** - 掌握 API 使用方法
4. **[架构与数据流](02_Architecture.md)** - 理解内部工作原理

预计学习时间：45 分钟

### 安全研究路线

1. **[攻击面分析](05_AttackSurface.md)** - 识别所有外部输入入口
2. **[安全风险评估](06_SecurityReview.md)** - 深度漏洞分析和修复建议
3. **[架构与数据流](02_Architecture.md)** - 理解信任边界和数据流
4. **[对外接口](04_Interface.md)** - 接口级安全分析

预计研究时间：60 分钟

---

## 文档清单

| 文档 | 内容 | 状态 |
|------|------|------|
| [01_Overview.md](01_Overview.md) | 项目定位、能力边界、快速开始 | ✅ 已完成 |
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型 | ✅ 已完成 |
| [03_CodeMap.md](03_CodeMap.md) | 目录结构、核心文件定位 | ✅ 已完成 |
| [04_Interface.md](04_Interface.md) | N-API/IPC 接口详细定义 | ✅ 已完成 |
| [05_AttackSurface.md](05_AttackSurface.md) | 外部输入、敏感操作、信任边界 | ✅ 已完成 |
| [06_SecurityReview.md](06_SecurityReview.md) | 风险评估、漏洞分析、修复建议 | ✅ 已完成 |
| [07_Build.md](07_Build.md) | GN 构建、产物、Feature 开关 | ✅ 已完成 |
| [08_Internals.md](08_Internals.md) | 核心类、内部 API、资源生命周期 | ✅ 已完成 |

---

## 覆盖范围

- ✅ N-API 接口（download, upload, agent）
- ✅ 框架实现（native, js, ets, cj）
- ✅ 服务实现（download_server Rust + C++）
- ✅ IPC 接口定义（22 个命令码）
- ✅ 安全攻击面分析（5 类风险）
- ✅ GN 构建配置（所有 targets）
- ✅ 代码证据（文件路径 + 行号）

## 未覆盖范围

- ⚠️ 测试相关内容（test/）
- ⚠️ 示例代码和教程
- ⚠️ 第三方依赖内部实现（curl, libuv 等）

---

## 证据标准

本文档遵循**证据优先原则**：

- 每个技术结论都有代码证据支撑
- 文件路径格式：`path/to/file.rs:line_number`
- 代码片段保留上下文
- 无法确认处标注 `TODO(证据不足)`

---

## 项目概况

| 属性 | 值 |
|------|-----|
| **组件名** | @ohos/request |
| **子系统** | request |
| **版本** | 3.1 |
| **SA ID** | 3706 |
| **进程名** | download_server |
| **技术栈** | Rust (核心) + C++ (N-API) |
| **许可证** | Apache License 2.0 |

---

## 更新说明

- **2026-02-07**: 初始版本，完成所有核心文档
  - 创建项目评估报告
  - 完成代码侦查和证据收集
  - 编写 8 篇核心文档
  - 完成安全风险评估（5 类风险）

---

## 相关链接

- [项目评估报告](_work/ASSESSMENT.md)
- [代码证据记录](_work/NOTES.md)
- [任务进度追踪](_work/PLAN.md)
- [文档导航](SUMMARY.md)
