# gRPC OpenHarmony Wiki

## 概述

本文档详细描述 **gRPC** 在 **OpenHarmony (OH)** 中的集成、适配和使用方式。

gRPC 是 OpenHarmony 的标准第三方库，提供高性能的 RPC（远程过程调用）能力，支持多种语言和平台间的服务通信。

## 文档导航

### 快速入门

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | gRPC 简介及其在 OH 中的定位 |
| [02_Patches.md](./02_Patches.md) | OH Patch 详细分析 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

### 关键信息

- **上游版本**: v1.73.0
- **OH Bundle 版本**: 3.1
- **许可证**: Apache License V2.0
- **主要功能**: RPC 框架、HTTP/2 通信、SSL/TLS 安全传输

### 主要输出

| 库 | 说明 |
|----|----|
| `libgpr.so` | gRPC 平台运行时库 |
| `libgrpc.so` | gRPC C 核心库 |
| `libgrpcxx.so` | gRPC C++ 库 |
| `grpc_cpp_plugin` | Protocol Buffers C++ 插件 |

## 核心特点

### 在 OH 中的适配

1. **最小化 Patch**: gRPC 在 OH 中没有大量定制化，主要是上游代码的直接移植
2. **构建系统**: 使用 GN 构建系统替代 Bazel/CMake
3. **依赖管理**: 与 OH 组件系统深度集成
4. **日志集成**: 使用 hilog 作为日志后端

### 与上游的差异

- 禁用 c-ares DNS 解析，使用系统原生 DNS
- 使用 protobuf lite 减少 ROM 占用
- 使用 OpenSSL 替代 BoringSSL
- 使用 OH 的 hilog 日志系统

## 使用建议

### 适用场景

- 分布式系统服务间通信
- 云同步服务
- AI 模型服务调用
- 微服务架构

### 注意事项

- gRPC 依赖 protobuf，需要正确处理 proto 文件
- 使用 SSL/TLS 时注意证书管理
- 注意版本兼容性，与上游保持同步

## 文档维护

本文档基于 gRPC v1.73.0 创建，如有更新请参考上游发布说明。

---

**最后更新**: 2026-02-08
