# DLP 权限管理服务工程 Wiki

> 本 Wiki 文档提供 DLP 权限管理服务 (`dlp_permission_service`) 的完整工程文档，帮助新人快速理解项目架构、API 接口、构建系统和安全特性。

---

## 文档覆盖范围

| 文档 | 说明 |
|-------|------|
| [项目概览](00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| [目录结构与模块职责](01_Directory_Structure.md) | 顶层目录结构、各模块职责 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [对外 N-API](03_NAPI.md) | JavaScript API 面向的三方接口 |
| [内部 API](04_Internal_API.md) | 模块间接口、依赖方向、稳定性标注 |
| [GN Targets](05_GN_Targets.md) | 构建目标、类型、依赖、产物、开关 |
| [编译产物](06_Build_Artifacts.md) | .so/.a/.hap 文件、安装路径、运行时加载 |
| [安全风险评审](07_Security_Review.md) | 攻击面、信任边界、可被利用点、修复建议 |
| [工作区 - ASSESSMENT.md](_work/ASSESSMENT.md) | **项目评估文档**（项目类型判定、受众需求、文档策略） |

---

## 新人阅读顺序建议

推荐按以下顺序阅读：

1. **[项目概览](00_Overview.md)** - 了解项目定位和核心能力
2. **[目录结构与模块职责](01_Directory_Structure.md)** - 熟悉代码组织
3. **[架构说明](02_Architecture.md)** - 理解整体架构和数据流
4. **[对外 N-API](03_NAPI.md)** - 了解如何调用 DLP 服务（三方应用开发）
5. **[安全风险评审](07_Security_Review.md)** - 了解安全特性和风险点

---

## 如何更新文档

### 文档与代码同步

本 Wiki 基于代码生成，每次代码变更后建议更新对应章节：

| 变更类型 | 建议更新文档 |
|----------|------------|
| N-API 变更 | 03_NAPI.md |
| IPC 接口变更 | 02_Architecture.md, 04_Internal_API.md |
| 构建配置变更 | 05_GN_Targets.md, 06_Build_Artifacts.md |
| 安全相关变更 | 07_Security_Review.md |

### 更新流程

1. 修改相关代码
2. 更新 `wiki/_work/NOTES.md` 中的事实记录
3. 更新对应的 Wiki 文档
4. 更新本文档中的"生成时间"

### 证据追溯原则

所有关键结论都应包含代码证据：
- 文件路径（含行号，如 `path:line`）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

无法确认的内容应标注 `TODO(需确认)` 并说明缺少的证据。

---

## 生成信息

- **生成时间**：2026-02-07（含 ASSESSMENT.md）
- **代码版本**：OpenHarmony base/security/dlp_permission_service
- **生成方式**：代码扫描 + 文档生成 Agent
- **工作区**：`wiki/_work/` (ASSESSMENT.md, NOTES.md, PLAN.md)

---

## 反馈与改进

如有文档不准确或需要补充的内容，请联系项目维护者或提交 Issue。
