# PrintSpooler 工程 Wiki

> 本文档是 PrintSpooler 项目的工程文档，提供项目概览、架构说明、API 参考、构建配置和安全评审等内容。

## 文档覆盖范围

本 Wiki 涵盖以下内容：

- 项目概览与定位
- 目录结构说明
- 架构设计与数据流
- 对外 API（OpenHarmony 系统 JS API）
- 内部 API 与模块接口
- 构建配置与编译产物
- 安全风险评审
- 常见问题解答

## 未覆盖范围

由于项目特性，以下内容不在此文档范围：

- **N-API 绑定**: 项目为纯 ArkTS/TypeScript 应用，不涉及 N-API 绑定
- **C++ 源代码**: 项目不包含 C++ 代码
- **GN 构建配置**: 项目使用 hvigor 构建系统，而非 GN
- **单元测试**: 本文档不引用测试相关内容

## 文档生成时间

- 生成日期：2026-02-05
- 项目版本：1.0.0.10
- SDK 版本：API 18

## 如何随代码更新文档

当代码发生变化时，建议更新以下内容：

1. **API 变更**: 更新 `04_External_API.md` 和 `05_Internal_API.md`
2. **架构变更**: 更新 `03_Architecture.md` 和附录中的调用链图
3. **权限变更**: 更新 `08_Security_Review.md`
4. **模块结构变更**: 更新 `02_Directory_Structure.md`
5. **构建配置变更**: 更新 `06_GN_Build.md` 和 `07_Build_Artifacts.md`

## 贡献指南

如需改进本文档，请：

1. 确保所有结论都有代码证据支持（文件路径+行号+符号名）
2. 保持术语一致性
3. 更新相关章节的交叉引用
4. 在 `SUMMARY.md` 中更新导航链接

## 导航

- [全站导航](SUMMARY.md) - 推荐从这里开始阅读
- [项目概览](00_Overview.md) - 快速了解项目
- [项目定位与边界](01_Project_Scope.md) - 项目职责与范围
- [目录结构](02_Directory_Structure.md) - 代码组织结构
- [架构设计](03_Architecture.md) - 组件关系与数据流
- [对外 API](04_External_API.md) - OpenHarmony 系统 JS API 使用
- [内部 API](05_Internal_API.md) - 模块间接口
- [构建配置](06_GN_Build.md) - Hvigor 构建配置
- [编译产物](07_Build_Artifacts.md) - 编译输出文件
- [安全风险评审](08_Security_Review.md) - 安全分析与建议
- [常见问题](09_FAQ.md) - 构建运行调试问题
- [附录：关键调用链](appendix/Callgraphs.md) - 重要调用流程
- [附录：关键配置](appendix/Config_Flags.md) - 配置参数说明
