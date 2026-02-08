# OpenHarmony N-API 组件 Wiki

## 项目概述

本 Wiki 旨在为 OpenHarmony `arkui/napi` 组件提供完整的技术文档，帮助开发者理解 N-API 框架的架构设计、接口规范、构建系统和安全机制。

**项目定位**：N-API（Native API）组件是一套对外接口基于 Node.js N-API 规范开发的原生模块扩展开发框架，用于实现 JS 与 C/C++ 代码的互相访问。

## 覆盖范围

### 已覆盖内容

| 文档 | 描述 |
|------|------|
| [README](README.md) | 本文档，说明覆盖范围和更新方式 |
| [SUMMARY](SUMMARY.md) | 全站导航和新人阅读路线 |
| [概览](01_Overview.md) | 项目定位、核心能力、运行环境 |
| [目录结构](02_Directory_Structure.md) | 模块职责和代码组织 |
| [架构说明](03_Architecture.md) | 组件图、数据流、线程模型 |
| [N-API 接口参考](04_NAPI_Reference.md) | 完整 API 清单和使用示例 |
| [构建系统](05_Build_System.md) | GN Targets 和编译产物 |
| [安全风险评审](06_Security.md) | 攻击面分析和风险评估 |
| [故障排查](07_Troubleshooting.md) | 常见问题和定位路径 |

### 未覆盖内容

- 特定业务模块的实现细节
- 测试代码（根据约束不引用测试代码）
- 第三方依赖的内部实现

## 代码证据原则

本 Wiki 所有关键结论均提供代码证据：
- 文件路径（必要时含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

**示例**：
> 模块通过 `napi_module_register()` 注册（证据：`native_node_api.h:58`）

## 更新方式

当代码变更时，需同步更新相关文档：

1. **API 变更**：更新 `04_NAPI_Reference.md` 中的 API 清单
2. **架构变更**：更新 `03_Architecture.md` 中的组件 **构建变更**：更新 `05_Build_System.md`图
3. 中的 targets
4. **安全发现**：更新 `06_Security.md` 中的风险清单

## 文档规范

- 默认中文（除非另有要求）
- 禁止引用测试代码作为业务证据
- 仅修改 `wiki/**` 目录
- 所有文档必须包含：目的、适用范围、关键结论、相关跳转链接

---

**生成时间**：2026-02-06
**文档版本**：1.0
