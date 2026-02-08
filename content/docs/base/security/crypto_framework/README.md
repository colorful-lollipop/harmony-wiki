# Crypto Framework Wiki

## 概述

本 Wiki 文档提供了 OpenHarmony `crypto_framework`（加解密算法库框架）的完整技术文档，面向希望深入理解该项目架构和实现细节的开发者。

**项目路径**: `/Volumes/lexar/code/d/work/oh/base/security/crypto_framework`

**版本**: 3.2

**子系统**: security

**许可证**: Apache License 2.0

## 覆盖范围

### 已覆盖内容

- ✅ **项目概览**：项目定位、边界、核心能力
- ✅ **目录结构**：模块职责和代码组织
- ✅ **架构设计**：分层架构、组件关系、数据流
- ✅ **对外 API**：N-API 绑定、Native Kits API
- ✅ **内部 API**：框架内部接口、SPI 接口
- ✅ **构建系统**：GN targets、依赖关系、编译产物
- ✅ **安全分析**：攻击面、信任边界、潜在风险

### 未覆盖内容

- ❌ **测试相关**：单元测试、模糊测试（根据要求忽略）
- ❌ **第三方库**：OpenSSL、MbedTLS 内部实现细节
- ❌ **性能分析**：算法性能基准测试
- ❌ **使用教程**：应用程序使用示例

## 文档阅读顺序

建议按以下顺序阅读文档：

1. [SUMMARY.md](SUMMARY.md) - 全站导航和阅读路线图
2. [00_Overview.md](00_Overview.md) - 项目概览和快速入门
3. [01_Project_Positioning.md](01_Project_Positioning.md) - 理解项目定位和边界
4. [02_Directory_Structure.md](02_Directory_Structure.md) - 了解代码组织
5. [03_Architecture.md](03_Architecture.md) - 深入架构设计
6. [04_External_API.md](04_External_API.md) - 对外 API 参考
7. [05_Internal_API.md](05_Internal_API.md) - 内部接口设计
8. [06_GN_Targets.md](06_GN_Targets.md) - 构建系统详解
9. [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物说明
10. [08_Security_Review.md](08_Security_Review.md) - 安全风险分析
11. [09_Troubleshooting.md](09_Troubleshooting.md) - 常见问题解答

## 文档维护

### 更新方式

本文档是基于源代码分析生成的静态文档。当代码更新时，建议：

1. 检查 `wiki/_work/NOTES.md` 中的文件路径是否仍然有效
2. 验证关键 API 和接口定义是否发生变化
3. 更新错误码和常量定义
4. 补充新添加的算法和功能

### 生成时间

- **生成日期**: 2026-02-06
- **代码版本**: 基于 2026-02-05 的代码快照

## 贡献指南

如需补充或修正本文档：

1. 确保所有结论都有代码证据支持（文件路径 + 符号名）
2. 更新相关文档的交叉引用
3. 保持与代码结构一致
4. 测试所有 Markdown 链接

## 相关资源

- **官方文档**: [加解密算法库框架开发指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/CryptoArchitectureKit/Readme-CN.md)
- **API 参考**: [CryptoArchitectureKit API](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-crypto-architecture-kit/js-apis-cryptoFramework.md)
- **仓库地址**: [security_crypto_framework](https://gitcode.com/openharmony/security_crypto_framework)
- **子系统概览**: [安全子系统](https://gitcode.com/openharmony/docs/blob/master/zh-cn/readme/安全子系统.md)
