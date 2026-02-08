# HiLog Wiki 文档

> 生成时间: 2026-02-06
> 最后更新: 2026-02-07（添加 ASSESSMENT.md 和优化 SUMMARY.md 双路线导航）
> 路径: /base/hiviewdfx/hilog

---

## 文档目的

本 Wiki 文档为 OpenHarmony HiLog 模块提供全面的工程文档，覆盖：
- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件、数据流、线程模型）
- 对外 API（JS N-API、NDK、Rust、ETS）
- 内部 API 与依赖关系
- GN Targets 与编译产物
- 安全风险评审（基于代码证据）

## 适用范围

本文档适用于 OpenHarmony HiLog 模块（版本 3.1），包括以下组件：
- hilogd 日志服务
- hilog 命令行工具
- libhilog 客户端库
- hilog_napi JS 接口
- hilog_ndk NDK 接口
- hilog_rust Rust 接口
- ani_hilog ETS 接口

## 更新方式

随着代码演进，本文档应保持更新：
1. 关键接口变更（N-API、NDK、内部 API）
2. 架构调整（线程模型、通信机制）
3. 新增/移除 GN targets
4. 安全机制变更（权限控制、流控策略）

更新建议：
- 同步更新 NOTES.md 中的证据索引
- 更新本文档中对应的章节
- 记录变更日期和原因

## 快速导航

新人阅读顺序：
1. [项目概览](01_Overview.md) - 了解 HiLog 定位和核心能力
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [架构说明](03_Architecture.md) - 理解组件交互和数据流
4. [N-API 接口](04_NAPI_Interface.md) - JS 调用 HiLog
5. [内部 API](05_Internal_API.md) - Native 开发使用
6. [GN Targets](06_GN_Targets.md) - 编译系统理解
7. [编译产物](07_Build_Artifacts.md) - 了解生成文件
8. [安全分析](08_Security_Analysis.md) - 安全风险与建议
9. [常见问题](09_Troubleshooting.md) - 问题定位与解决

## 相关跳转链接

- [N-API 接口详情](04_NAPI_Interface.md)
- [内部 API 详情](05_Internal_API.md)
- [GN 构建系统](06_GN_Targets.md)
- [安全风险列表](08_Security_Analysis.md)
- [调用链图](appendix/Callgraphs.md)
- [配置标志参考](appendix/Config_Flags.md)

---

## 已知限制

1. **测试内容不引用**: 本文档不引用测试相关内容（test/、unittest/、fuzz/）
2. **证据追溯**: 所有关键结论必须能在仓库内找到直接证据
3. **语言**: 默认中文，代码和技术术语保持英文

---

## 版本历史

| 日期 | 版本 | 变更说明 |
|------|-------|---------|
| 2026-02-06 | 3.1 | 初始版本，基于 hiviewdfx/hilog 源码生成 |
| 2026-02-07 | 3.1.1 | 添加 ASSESSMENT.md 项目评估文档，优化 SUMMARY.md 双路线导航（新人 + 安全研究员）|
