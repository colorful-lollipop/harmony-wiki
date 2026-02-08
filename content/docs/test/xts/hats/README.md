# OpenHarmony HATS Wiki

## 文档概述

本文档为 OpenHarmony HATS（Hardware Abstract Test Suite，硬件抽象测试套件）仓库的工程文档，旨在帮助开发者快速理解项目结构、架构设计、编译构建和安全评估。

### 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | HATS 定位、目标、系统类型 |
| 目录结构 | ✅ 完成 | 9 大子系统模块 |
| 架构说明 | ✅ 完成 | HDI 测试模式、组件交互 |
| N-API 接口 | ⚠️ 有限 | HATS 主要测试 HAL 层，不涉及 N-API |
| 内部 API | ✅ 完成 | HDI 服务接口、依赖关系 |
| GN 构建 | ✅ 完成 | 505 个构建目标解析 |
| 编译产物 | ✅ 完成 | 静态库、可执行文件、配置 |
| 安全评审 | ✅ 完成 | 威胁模型与风险点 |

### 文档语言

- **默认语言**：中文（简体）
- **代码注释**：保持原始英文
- **API 命名**：保持原始英文标识符

### 更新方式

本文档随代码仓库同步更新。文档生成基于以下版本：

- **仓库版本**：4.0
- **生成时间**：2026-02-06
- **最后更新**：随 BUILD.gn 和源码变更同步

如需更新文档，请参考：

1. 修改源码后检查 `wiki/_work/NOTES.md` 记录变更
2. 更新对应的模块文档
3. 运行 `lsp_diagnostics` 验证文档格式

### 相关链接

- **主仓库**：`/Volumes/lexar/code/d/work/oh/test/xts/hats`
- **README**：[README.md](/Volumes/lexar/code/d/work/oh/test/xts/hats/README.md)
- **构建配置**：[BUILD.gn](/Volumes/lexar/code/d/work/oh/test/xts/hats/BUILD.gn)
- **测试框架**：GoogleTest (gtest)

---

## 快速导航

| 主题 | 入口文件 |
|------|----------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 模块架构 | [01_Architecture.md](./01_Architecture.md) |
| 子系统详解 | [02_Modules.md](./02_Modules.md) |
| N-API 说明 | [03_N-API.md](./03_N-API.md) |
| 构建系统 | [04_Build.md](./04_Build.md) |
| 安全评审 | [05_Security.md](./05_Security.md) |
| 附录 | [06_Appendix.md](./06_Appendix.md) |

---

## 贡献指南

如需改进本文档：

1. 在对应文档中添加/修改内容
2. 确保结论可追溯到代码证据（路径 + 行号）
3. 避免引用测试文件作为业务证据
4. 更新 SUMMARY.md 导航链接
