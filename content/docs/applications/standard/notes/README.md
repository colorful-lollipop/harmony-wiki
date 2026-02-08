# OpenHarmony Notes 应用工程文档

## 文档概述

本文档为 OpenHarmony 备忘录（Notes）应用的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、核心模块与构建流程。

### 覆盖范围

| 分类 | 状态 | 说明 |
|------|------|------|
| 项目定位与边界 | ✅ 已覆盖 | 详见 [00_Overview.md](00_Overview.md) |
| 目录结构与模块职责 | ✅ 已覆盖 | 详见 [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构说明 | ✅ 已覆盖 | 详见 [02_Architecture.md](02_Architecture.md) |
| 对外 API (N-API) | ❌ 不适用 | 本项目为纯 ArkTS 应用，无 N-API 导出 |
| 内部 API | ✅ 已覆盖 | 详见 [03_Inner_API.md](03_Inner_API.md) |
| GN 目标梳理 | ✅ 已覆盖 | 详见 [04_GN_Targets.md](04_GN_Targets.md) |
| 编译产物 | ✅ 已覆盖 | 详见 [05_Build_Artifacts.md](05_Build_Artifacts.md) |
| 安全风险评审 | ✅ 已覆盖 | 详见 [06_Security_Review.md](06_Security_Review.md) |
| 常见问题与调试 | ✅ 已覆盖 | [07_Troubleshooting.md](07_Troubleshooting.md) |

### 不包含内容

- **测试相关**：本 Wiki 不引用测试代码作为业务证据
- **N-API**：本项目为 ArkTS 应用，不涉及原生 N-API 导出
- **第三方库**：仅记录项目直接依赖的系统能力

### 更新方式

本文档基于代码静态分析生成。如需更新：

1. 修改代码后检查相关文档章节
2. 运行文档检查脚本（待实现）
3. 提交 PR 时附带文档更新

### 生成信息

- **生成时间**: 2025-02-05
- **分析范围**: `/applications/standard/notes`
- **代码版本**: 基于当前 HEAD 分析
