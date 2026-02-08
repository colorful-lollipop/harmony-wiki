# Accessibility 子系统 Wiki

## 概述

本 Wiki 提供了 OpenHarmony Accessibility 子系统的完整技术文档，包括架构说明、API 参考、构建系统、安全评审等内容，旨在帮助新人快速理解项目。

## 文档生成信息

- **生成时间**: 2026-02-06
- **项目版本**: 4.0
- **覆盖范围**:
  - ✅ 项目概览和核心概念
  - ✅ 目录结构和模块职责
  - ✅ 架构说明（组件图、数据流、线程模型）
  - ✅ N-API 接口文档
  - ✅ 内部 API 和模块接口
  - ✅ GN 构建系统和编译产物
  - ✅ 安全风险评审
  - ⚪ 常见问题（待补充）
- **未覆盖范围**:
  - 测试相关内容
  - 性能优化建议
  - 详细的使用示例

## 如何更新文档

文档基于代码自动生成，如需更新：

1. **代码变更后**：重新运行文档生成工具（如有）
2. **手动更新**：
   - 检查证据是否仍然有效（文件路径、行号）
   - 更新变更的 API 或模块
   - 补充新增的组件或功能
3. **版本标记**：更新文档末尾的版本和日期

## 文档结构

所有文档链接请在 [SUMMARY.md](SUMMARY.md) 中查看。

## 快速导航

### 新人入门路线

1. 先读 [00_Overview.md](00_Overview.md) - 了解项目整体
2. 再读 [01_Project_Scope.md](01_Project_Scope.md) - 理解项目边界和核心能力
3. 继续读 [02_Directory_Structure.md](02_Directory_Structure.md) - 熟悉代码组织
4. 深入读 [03_Architecture.md](03_Architecture.md) - 理解架构设计
5. 参考 [04_N-API.md](04_N-API.md) - 学习对外 API
6. 查看 [08_Security_Review.md](08_Security_Review.md) - 了解安全考量

### 按 topic 查找

| 主题 | 文档 |
|------|------|
| 架构设计 | [03_Architecture.md](03_Architecture.md) |
| 对外 API | [04_N-API.md](04_N-API.md) |
| 内部接口 | [05_Inner_API.md](05_Inner_API.md) |
| 构建系统 | [06_GN_Targets.md](06_GN_Targets.md) |
| 编译产物 | [07_Build_Artifacts.md](07_Build_Artifacts.md) |
| 安全评审 | [08_Security_Review.md](08_Security_Review.md) |
| 常见问题 | [09_FAQ.md](09_FAQ.md) |
| 调用链图 | [appendix/Callgraphs.md](appendix/Callgraphs.md) |
| 配置标志 | [appendix/Config_Flags.md](appendix/Config_Flags.md) |

---

最后更新: 2026-02-06
