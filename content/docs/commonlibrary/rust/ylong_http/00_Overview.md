# 项目概览

> 本文档提供 ylong_http 项目的整体概览和关键信息

---

## 目的

本文档的目的是让读者在 15 分钟内快速了解 ylong_http 项目的：
- 项目定位和目标
- 核心功能和能力
- 技术栈和依赖
- 运行环境要求
- 项目架构概览

## 适用范围

- ylong_http（HTTP 协议基础库）
- ylong_http_client（HTTP 客户端库）
- OpenHarmony 系统服务层集成者

## 不适用范围

- N-API/JS 绑定层（项目为纯 Rust 库）
- IPC/System Ability（不涉及）
- 应用层业务逻辑

---

## 关键结论

### 1. 项目定位

**ylong_http** 是 OpenHarmony 系统服务层的 **HTTP 协议栈组件**，使用 Rust 语言实现，为 `netstack` 模块提供 HTTP 协议支持。

**核心价值**：
- 完整的 HTTP 协议实现（HTTP/1.1, HTTP/2, HTTP/3）
- 高性能的 Rust 异步/同步客户端
- TLS/SSL 安全支持（OpenSSL 集成）
- 连接池和资源管理

**证据**: `bundle.json:17` - `part_name: "ylong_http"`, `subsystem_name: "commonlibrary"`

### 2. 核心能力

| 能力 | HTTP 版本 | 实现位置 |
|------|-----------|----------|
| 请求/响应编解码 | HTTP/1.1 | `ylong_http/src/h1/` |
| 请求/响应编解码 | HTTP/2 | `ylong_http/src/h2/` |
| 帧编解码、HPACK | HTTP/2 | `ylong_http/src/h2/hpack/` |
| 请求/响应编解码 | HTTP/3 | `ylong_http/src/h3/` |
| 帧编解码、QPACK | HTTP/3 | `ylong_http/src/h3/qpack/` |
| 哈夫曼编码 | HTTP/2/3 | `ylong_http/src/huffman/` |
| 异步客户端 | HTTP/1.1, HTTP/2 | `ylong_http_client/src/async_impl/` |
| 同步客户端 | HTTP/1.1, HTTP/2 | `ylong_http_client/src/sync_impl/` |
| TLS/SSL 支持 | HTTPS | `ylong_http_client/src/util/c_openssl/` |
| 连接池 | 所有版本 | `util/pool.rs`, `async_impl/pool.rs` |
| HTTP 代理 | HTTP/1.1, HTTP/2 | `util/proxy.rs` |
| 自动重定向 | 所有版本 | `util/redirect.rs` |
| DNS 解析 | 所有版本 | `async_impl/dns/` |

### 3. 技术栈

**语言**: Rust 2021 Edition

**证据**: `ylong_http/Cargo.toml:18` - `edition = "2021"`

**运行时选项**：
- `ylong_runtime` - OpenHarmony 异步运行时（默认）
- `tokio` - 第三方异步运行时（可选）

**证据**: `ylong_http_client/Cargo.toml:11-12`

**TLS 库**：
- OpenSSL 3.0（通过 FFI 集成）

**证据**: `ylong_http_client/src/util/c_openssl/ffi/` - `extern "C"` OpenSSL 绑定

**HTTP/3 实现**（可选）：
- quiche 0.22.0（QUIC 协议库）

**证据**: `ylong_http_client/Cargo.toml:15` - `quiche = { version = "0.22.0" }`

### 4. 运行环境要求

- **系统**: OpenHarmony (standard)
- **架构**: Linux/macOS（基于 libc 依赖）
- **RAM**: ~200KB（bundle.json:26）
- **ROM**: 100KB（bundle.json:25）

**证据**: `bundle.json:22-27`

### 5. 项目架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                  上层应用 (APP)                     │
│                                                        │
│                                                        ▼
│                  ┌─────────────────────────────┐              │
│                  │      netstack            │              │
│                  │ (系统服务层)            │              │
│                  └───────────┬─────────────┘              │
│                              │                             │
│                              ▼                             │
│                  ┌─────────────────────────────┐              │
│                  │     ylong_http           │              │
│                  │   (HTTP 协议栈)         │              │
│                  │  ┌────────────────────┐  │              │
│                  │  │ ylong_http      │  │              │
│                  │  │  (基础组件)     │  │              │
│                  │  │  - Request/Response│  │              │
│                  │  │  - Headers       │  │              │
│                  │  │  - Body          │  │              │
│                  │  │  - H1/H2/H3      │  │              │
│                  │  └────────┬────────┘  │              │
│                  │           │              │              │
│                  │           ▼              │              │
│                  │  ┌────────────────────┐  │              │
│                  │  │ ylong_http_client│  │              │
│                  │  │  (客户端层)     │  │              │
│                  │  │  ┌──────────┐  │  │              │
│                  │  │  │ async    │  │  │              │
│                  │  │  │ impl      │  │  │              │
│                  │  │  │  - Client │  │  │              │
│                  │  │  └──────────┘  │  │              │
│                  │  │  ┌──────────┐  │  │              │
│                  │  │  │ sync     │  │  │              │
│                  │  │  │ impl      │  │  │              │
│                  │  │  │  - Client │  │  │              │
│                  │  │  └──────────┘  │  │              │
│                  │  │                  │  │              │
│                  │  └───────────────────┘  │              │
│                  └─────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

**依赖层次**：
```
ylong_http_client
  └─→ ylong_http (协议基础组件)
      └─→ ylong_runtime (异步运行时)
ylong_http_client (TLS)
  └─→ openssl (libssl_shared, libcrypto_shared)
ylong_http_client (HTTP/3)
  └─→ quiche (QUIC 实现)
```

**证据**: `ylong_http_client/BUILD.gn:35-40` - `deps` 和 `external_deps`

### 6. 核心设计原则

1. **协议分层**: HTTP 协议实现与客户端实现分离
   - `ylong_http`: 提供 Request/Response/Headers/Body 等类型和编解码
   - `ylong_http_client`: 提供客户端逻辑、连接池、代理等
   - 证据: `ylong_http_client/src/lib.rs:24-35` - `pub use ylong_http::*`

2. **同步异步统一**: async_impl 和 sync_impl 提供相同的接口原型
   - 用户可轻松切换同步/异步实现
   - 证据: `ylong_http_client/src/async_impl/mod.rs` 和 `sync_impl/mod.rs` - 相似的 Client/Connector 接口

3. **运行时无关**: 支持 ylong_runtime 和 tokio 两种运行时
   - 通过 feature flags 切换
   - 证据: `ylong_http/Cargo.toml:18-21` - `tokio_base`, `ylong_base` features

4. **TLS 集成**: 通过 FFI 封装 OpenSSL，提供安全 HTTPS 支持
   - 支持 ALPN、证书验证、公钥固定
   - 证据: `ylong_http_client/src/util/c_openssl/` - 713 行 FFI 代码

5. **资源管理**: 使用连接池复用连接，减少开销
   - 异步连接池: `async_impl/pool.rs`
   - 通用连接池: `util/pool.rs`
   - 证据: `ylong_http_client/src/async_impl/pool.rs` - `ConnPool` 实现

### 7. 支持的 HTTP 特性

| 特性 | HTTP/1.1 | HTTP/2 | HTTP/3 | 实现方式 |
|------|----------|-------|-------|----------|
| 分块传输 (Chunked) | ✅ | ✅ | ✅ | `Body::ChunkBody` |
| 持久连接 (Keep-Alive) | ✅ | ✅ | ✅ | 连接池实现 |
| 头部压缩 (HPACK/QPACK) | ❌ | ✅ | ✅ | `h2/hpack/`, `h3/qpack/` |
| 服务端推送 (Server Push) | ❌ | ⚠️ | ✅ | 代码存在（h3/） |
| 多路复用 (Multiplexing) | ❌ | ✅ | ✅ | HTTP/2 流, HTTP/3 流 |
| 优先级 (Priority) | ❌ | ❌ | ✅ | HTTP/3 优先级帧 |
| 流量控制 (Flow Control) | ❌ | ✅ | ✅ | HTTP/2/3 流控 |
| TLS/SSL | ✅ | ✅ | ✅ | OpenSSL 集成 |
| 代理支持 | ✅ | ✅ | ✅ | `util/proxy.rs` |
| 自动重定向 | ✅ | ✅ | ✅ | `util/redirect.rs` |

**证据**: 各模块 README 和源代码

### 8. 项目限制

| 限制项 | 说明 |
|-------|------|
| **HTTP/3 支持** | 代码已实现（h3/），但 GN 配置中未启用 http3 feature |
| **N-API/JS 绑定** | 项目为纯 Rust 库，不提供 JS/N-API 接口 |
| **应用层权限控制** | 无应用层权限检查机制，仅有 TLS 层证书验证 |
| **双向流（Server Push）** | 当前实现为客户端，不支持服务端功能 |
| **WebSocket 支持** | 未实现 WebSocket 协议 |
| **HTTP/0.9 支持** | 仅支持 HTTP/1.1 及以上版本 |

**证据**: GN 配置和代码扫描结果

---

## 相关跳转

- **[项目定位与边界](01_Project_Position.md)** - 详细的定位说明和依赖关系
- **[架构说明](03_Architecture.md)** - 完整的架构设计文档
- **[对外 API](04_External_API.md)** - 公共 API 使用指南
- **[GN 目标梳理](06_GN_Targets.md)** - 构建系统详解

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
