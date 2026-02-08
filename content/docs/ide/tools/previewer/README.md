# OpenHarmony Previewer Wiki

## 项目概述

**OpenHarmony Previewer** 是 DevEco Studio IDE 的核心组件，负责使用 ArkUI 渲染引擎实现实时页面预览功能。

- **项目名称**: `@ohos/previewer`
- **版本**: 3.1
- **License**: Apache License 2.0
- **子系统**: `ide`
- **组件名**: `previewer`

## 文档覆盖范围

本 Wiki 全面覆盖 Previewer 项目的以下方面：

| 文档 | 内容 |
|------|------|
| [README](README.md) | 本文档，使用指南 |
| [SUMMARY](SUMMARY.md) | 全站导航与阅读路线（双路径：新人/安全） |
| [项目评估](_work/ASSESSMENT.md) | 项目类型判定与文档策略 |
| [代码证据](_work/NOTES.md) | 代码证据汇总（技术结论追溯） |
| [工作计划](_work/PLAN.md) | Wiki 生成任务追踪 |

| 文档 | 内容 |
|------|------|
| [概览](00_Overview.md) | 项目定位、核心能力、运行环境 |
| [目录结构](01_Directory_Structure.md) | 模块划分与职责 |
| [架构设计](02_Architecture.md) | 组件图、数据流、线程模型 |
| [通信协议](03_Communication_Protocol.md) | 命名管道、WebSocket、JSON 格式 |
| [内部 API](04_Inner_API.md) | 内部 Kit 与模块接口 |
| [构建系统](05_Build_System.md) | GN Targets 与依赖 |
| [编译产物](06_Artifacts.md) | 二进制产物与安装路径 |
| [安全评审](07_Security_Review.md) | 攻击面、风险分析与修复建议 |

| 文档 | 内容 |
|------|------|
| [Mock 能力](08_Mock_Capabilities.md) | Mock 层设计与扩展指南 |
| [命令行参数](09_Command_Line_Params.md) | 完整参数列表与说明 |
| [开发指南](10_Development_Guide.md) | 构建、测试、调试指南 |
| [常见问题](appendix/FAQ.md) | 构建与运行时问题 |

## 不包含内容

- **N-API 文档**: Previewer 不是 N-API 模块，无 JS API 面暴露
- **测试代码**: 测试目录 (`test/`) 内容不在文档范围内
- **外部依赖**: arkui_ace_engine, libwebsockets 等依赖仓库的内部实现

## 关键结论

1. **架构**: C++ 桌面应用，通过命名管道 + WebSocket 与 DevEco Studio 通信
2. **双版本**: Rich (完整 ArkUI) 和 Lite (ACELite)
3. **无权限系统**: 不集成 OpenHarmony 权限框架，依赖 WebSocket SID 认证
4. **内部 Kit**: 对外通过 `libide_util.so` 和 `libide_extension.so` 提供 C++ 接口
5. **安全风险**: automock 模块存在 CRITICAL/HIGH 级别的安全风险，需立即修复

## 文档阅读建议

### 新人学习者
**目标**: 30 分钟内理解项目架构和核心功能
**推荐路径**: SUMMARY → 概览 → 目录结构 → 架构设计 → 通信协议 → Mock 能力

### 安全研究员
**目标**: 快速识别攻击面和潜在漏洞
**推荐路径**: SUMMARY → 安全评审 → 通信协议 → 代码证据 → Mock 能力

### 开发者
**目标**: 理解构建流程和代码组织，开始贡献代码
**推荐路径**: 构建系统 → 开发指南 → Mock 能力 → 代码证据

## 更新方式

当代码变更时，应同步更新相关 Wiki 章节：

1. 新增模块 → 更新 `01_Directory_Structure.md`
2. 新增命令 → 更新 `03_Communication_Protocol.md` 和 `09_Command_Line_Params.md`
3. 新增 GN target → 更新 `05_Build_System.md`
4. 新增安全风险 → 更新 `07_Security_Review.md`
5. 扩展 Mock 能力 → 更新 `08_Mock_Capabilities.md`
6. 修复安全漏洞 → 更新 `07_Security_Review.md` 和 `_work/PLAN.md`

## 质量保证

- ✅ **证据追溯**: 每个技术结论都有代码证据支撑（文件路径:行号）
- ✅ **链接有效**: 所有文档链接已验证可访问
- ✅ **双路径导航**: 提供新人学习和安全研究两条阅读路线
- ✅ **安全优先**: 识别 9 个安全风险，包括 CRITICAL/HIGH/MEDIUM/LOW 四个级别
- ✅ **可扩展性**: 提供 Mock 能力扩展指南，支持开发者自定义功能

## 贡献指南

欢迎通过以下方式贡献：
1. **修正错误**: 如果发现文档错误，请提交 Issue 或 PR
2. **补充内容**: 如果发现缺失的内容，请补充文档
3. **改进表达**: 如果某些部分难以理解，请改进表达
4. **修复安全漏洞**: 如果发现新的安全风险，请及时修复并更新文档

---

**最后更新**: 2026-02-07
**维护者**: Previewer 团队
**反馈**: 请通过 Issue 或 PR 反馈问题和建议
