# ace_js2bundle Wiki

## 简介

本文档是 OpenHarmony 项目 `developtools/ace_js2bundle` 的工程 Wiki，提供项目架构、API、构建系统和安全风险的完整分析。

## 文档生成信息

- **生成时间**: 2025-02-06
- **代码版本**: 基于仓库 HEAD 提交
- **文档版本**: 1.0
- **覆盖范围**: 
  - ✅ 项目概览与定位
  - ✅ 目录结构与模块职责
  - ✅ 架构设计与数据流
  - ✅ 内部 API 与模块接口
  - ✅ GN 构建系统与产物
  - ✅ 安全风险评估
  - ❌ N-API 对外接口（本项目为纯构建工具，无 N-API）

## 快速导航

| 文档 | 说明 |
|------|------|
| [SUMMARY.md](./SUMMARY.md) | 完整导航与阅读路线 |
| [index.md](./index.md) | 项目首页/概览 |
| [01_Overview.md](./01_Overview.md) | 项目定位、边界、核心能力 |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构与模块职责 |
| [03_Architecture.md](./03_Architecture.md) | 架构说明、数据流、插件链 |
| [04_External_API.md](./04_External_API.md) | 对外 API（N-API）说明 |
| [05_Internal_API.md](./05_Internal_API.md) | 内部 API 与模块接口 |
| [06_Build_System.md](./06_Build_System.md) | GN 构建系统与产物 |
| [07_Security.md](./07_Security.md) | 安全风险评估 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 常见问题与定位 |

## 更新方式

本文档基于代码证据自动生成。当代码发生变更时：

1. 重新运行扫描工具更新 `wiki/_work/NOTES.md`
2. 根据代码变更更新相关 Wiki 页面
3. 确保所有结论都有代码证据支持（文件路径+行号）

## 证据规范

所有技术结论均标注代码证据：
- 文件路径 + 行号: `path/to/file.js:42`
- 关键符号名: `functionName`, `ClassName`
- 代码片段（必要时）

未找到证据的标注为 `TODO(需确认)`。

## 免责声明

本文档基于静态代码分析生成，仅供参考。生产环境使用请以官方文档为准。
