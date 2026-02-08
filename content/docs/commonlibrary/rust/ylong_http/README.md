# ylong_http Wiki

> 版本: 1.0
> 生成时间: 2026-02-06 09:27
> 项目: ylong_http (OpenHarmony Rust HTTP 库)

---

## 文档说明

本 Wiki 为 ylong_http 项目提供完整的技术文档，涵盖：

- **项目定位**：在 OpenHarmony 系统中的作用和边界
- **目录结构**：模块划分和职责说明
- **架构设计**：组件交互和数据流
- **API 接口**：对外暴露的 Rust API（无 N-API/JS 绑定）
- **构建系统**：GN targets 和编译产物
- **安全评审**：基于代码证据的安全风险分析

## 覆盖范围

- ✅ HTTP 协议实现（HTTP/1.1, HTTP/2, HTTP/3）
- ✅ HTTP 客户端（同步和异步）
- ✅ TLS/SSL 支持（OpenSSL 集成）
- ✅ 连接管理和连接池
- ✅ 代理和重定向
- ⚠️ HTTP/3 功能（代码存在但未在 GN 中启用）

## 未覆盖范围

- ❌ N-API/JS 绑定（项目为纯 Rust 库）
- ❌ IPC/System Ability（不涉及 OpenHarmony IPC）
- ❌ 应用层权限控制（仅有 TLS 层证书验证）
- ❌ 测试代码（所有 test/ 目录内容均不引用）

## 如何更新

当代码发生变化时，请按以下步骤更新 Wiki：

1. **同步代码变更**：更新对应模块的文档章节
2. **更新证据**：确保所有结论有代码路径和行号支持
3. **验证链接**：确保 SUMMARY.md 中的链接有效
4. **更新安全评审**：如有新功能，重新评估安全风险

## 维护者

- 当前维护者：自动生成（2026-02-06）
- 基于代码分支：master/main

## 文档约定

- 所有代码路径基于项目根目录
- 证据格式：`path:line` 或 `path/符号名`
- 无 N-API/JS 绑定
- 所有文档使用中文

---

## 相关资源

- **项目 README**: [README.md](../README.md) | [README_zh.md](../README_zh.md)
- **代码仓库**: https://gitcode.com/openharmony/commonlibrary_rust_ylong_http
- **许可证**: Apache License 2.0
