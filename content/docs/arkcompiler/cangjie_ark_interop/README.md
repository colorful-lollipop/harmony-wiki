# 仓颉-ArkTS 互操作 Wiki

## 项目简介

本 Wiki 记录 OpenHarmony `arkcompiler/cangjie_ark_interop` 子系统的工程文档，涵盖架构设计、API 接口、编译配置、安全评审等内容。

## 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ | 模块职责、目录结构 |
| N-API 接口 | ✅ | ark_interop 核心 API |
| 内部架构 | 🔄 | 模块依赖、线程模型 |
| GN 构建 | 🔄 | targets 与编译产物 |
| 安全评审 | 🔄 | 攻击面与风险分析 |

## 文档导航

建议阅读顺序（新人入门路径）：

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [01_API_Reference.md](./01_API_Reference.md) - N-API 接口参考
3. [02_Architecture.md](./02_Architecture.md) - 内部架构
4. [03_Build_System.md](./03_Build_System.md) - GN 构建系统
5. [04_Compilation_Products.md](./04_Compilation_Products.md) - 编译产物
6. [05_Security_Review.md](./05_Security_Review.md) - 安全风险评审

## 维护指南

### 何时更新文档

- 新增/删除/修改 N-API 接口时
- 修改模块依赖或架构时
- 新增编译 target 或构建开关时
- 发现或修复安全问题时

### 更新步骤

1. 在 `wiki/_work/NOTES.md` 中记录发现的事实
2. 更新对应的 Wiki 页面
3. 同步更新 `SUMMARY.md` 的导航链接
4. 运行 `lsp_diagnostics` 确保文档无格式错误

## 生成信息

- **生成时间**: 2025-02-06
- **代码版本**: 当前 HEAD
- **文档语言**: 中文 (默认)
