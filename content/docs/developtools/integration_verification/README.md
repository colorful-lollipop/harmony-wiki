# OpenHarmony 集成验证部件 Wiki

## 项目概述

本文档是 OpenHarmony 集成验证部件（`developtools/integration_verification`）的 Wiki 文档，旨在帮助开发者快速理解项目结构、测试框架和工具使用。

## 文档覆盖范围

### 已覆盖内容

- 项目定位与核心能力
- 目录结构与模块职责
- 用例组织与管理
- 工具集说明
- 部署与使用指南
- 架构设计与工作流程

### 未覆盖内容

- N-API 接口文档（本项目不涉及）
- GN 构建配置（本项目不使用）
- IPC/System Ability 文档（本项目不涉及）
- 安全风险详细评审（本项目为测试框架，风险有限）

## 文档更新方式

### 何时更新文档

当发生以下情况时，应同步更新 Wiki 文档：

1. 新增测试用例模块或工具
2. 修改用例配置格式（如 `repo_cases_matrix.csv`）
3. 更新工具功能或命令行参数
4. 变更架构或工作流程

### 更新流程

1. 修改对应模块的 Markdown 文档
2. 更新 `SUMMARY.md` 以反映新增内容
3. 在 `CHANGELOG.md` 中记录变更
4. 提交更改至代码仓库

## 相关资源

- **代码仓库**：https://gitee.com/openharmony/developtools_integration_verification
- **项目 README**：`../README.md`
- **许可证**：Apache License 2.0

## 文档语言

本文档默认使用简体中文编写。如需英文版本，请参考根目录下的 `README.en.md`。

---

*文档最后更新时间：2026年2月*
