# 项目定位与边界

> 本文档说明 ylong_http 在 OpenHarmony 系统中的定位、边界和核心能力

---

## 目的

本文档的目的是明确 ylong_http 项目的：
- 在 OpenHarmony 系统服务层中的位置
- 与其他组件的依赖关系
- 功能边界和职责范围
- 运行环境和资源要求

## 适用范围

- ylong_http 库（HTTP 协议基础组件）
- ylong_http_client 库（HTTP 客户端）

---

## 关键结论

### 1. 系统定位

**ylong_http 是 OpenHarmony 系统服务层的 HTTP 协议栈组件**，属于 `commonlibrary` 子系统的 `ylong_http` 部件。

**位置**: `commonlibrary/rust/ylong_http`

**证据**:
- `bundle.json:19` - `"subsystem": "commonlibrary"`, `"part_name": "ylong_http"`
- `bundle.json:20` - `"destPath": "commonlibrary/rust/ylong_http"`

**依赖的下游组件**：
- `netstack` - 系统网络协议栈模块
- `request` - 系统上传下载组件

**证据**: `README.md:13-15` - 架构图中显示 ylong_http 为 netstack 提供 HTTP 协议支持

### 2. 功能边界

**ylong_http 提供的功能**（在边界内）：
- ✅ HTTP 协议编解码（HTTP/1.1, HTTP/2, HTTP/3）
- ✅ HTTP 请求/响应结构（Request, Response, Headers, Body）
- ✅ TLS/SSL 支持（通过 OpenSSL FFI）
- ✅ 连接管理（TCP, TLS 连接）
- ✅ 连接池（连接复用）
- ✅ HTTP 代理支持
- ✅ 自动重定向（3xx 状态码）
- ✅ DNS 解析
- ✅ 证书验证和公钥固定
- ✅ 文件上传下载（通过 ylong_http_client）

**ylong_http 不提供的功能**（在边界外）：
- ❌ N-API/JS 绑定层（需要上层封装）
- ❌ IPC/System Ability 通信（不涉及 OpenHarmony IPC）
- ❌ 应用层权限控制（如 permission check）
- ❌ 服务端功能（WebSocket, Server Push, HTTP/0.9）
- ❌ 其他网络协议（FTP, SMTP, WebSocket）

### 3. 职责边界

```
┌─────────────────────────────────────────────────────────────┐
│                   ylong_http (边界)                 │
├─────────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────────────────────────────┐         │
│  │           HTTP 协议实现                   │         │
│  │  - Request/Response/Headers            │         │
│  │  - H1/H2/H3 编解码器              │         │
│  │  - Body 类型                          │         │
│  │  - HTTP 语义（方法、状态码、URI）    │         │
│  │                                       │         │
│  │  职责：提供 HTTP 协议能力             │         │
│  └──────────────────────────────────────────────┘         │
│                                                         │
│  ┌──────────────────────────────────────────────┐         │
│  │       HTTP 客户端实现                   │         │
│  │  - Client 接口（同步/异步）         │         │
│  │  - 连接池                          │         │
│  │  - 代理和重定向                     │         │
│  │  - TLS 配置和验证                   │         │
│  │  - DNS 解析                         │         │
│  │                                       │         │
│  │  职责：使用 HTTP 协议能力         │         │
│  └──────────────────────────────────────────────┘         │
│                                                         │
└─────────────────────────────────────────────────────────────┘
```

**证据**:
- `ylong_http/src/lib.rs` - 协议基础模块入口
- `ylong_http_client/src/lib.rs` - 客户端模块入口
- 模块职责在源代码中清晰分离

### 4. 核心能力

#### 4.1 HTTP 协议支持

| HTTP 版本 | 状态 | Feature | 主要实现 | GN Target |
|-----------|------|----------|----------|------------|
| HTTP/1.1 | ✅ 完全支持 | `http1_1` | ylong_http |
| HTTP/2 | ✅ 完全支持 | `http2` | ylong_http |
| HTTP/3 | ⚠️ 代码存在，未启用 | `http3` | ylong_http |

**证据**:
- `ylong_http/src/h1/mod.rs` - HTTP/1.1 模块
- `ylong_http/src/h2/mod.rs` - HTTP/2 模块
- `ylong_http/src/h3/mod.rs` - HTTP/3 模块（代码存在）
- `ylong_http/BUILD.gn:24-26` - features 配置（无 http3）

#### 4.2 TLS/SSL 支持

| 功能 | 实现方式 | 证据 |
|------|----------|------|
| TLS 加密 | OpenSSL FFI 集成 | `ylong_http_client/src/util/c_openssl/ffi/ssl.rs` |
| 证书验证 | 可自定义验证器 | `util/config/tls/verifier/mod.rs:18` - `CertVerifier` trait |
| 公钥固定 | SHA256 固定 | `util/c_openssl/verify/pinning.rs` |
| ALPN 协商 | 支持 HTTP/1.1, HTTP/2 | `util/config/tls/alpn/mod.rs` |
| 主机名验证 | SNI 支持 | `ylong_http_client/src/util/c_openssl/ssl/stream.rs:131` - `ssl_set_tlsext_host_name` |

#### 4.3 连接管理

| 功能 | 实现位置 | 证据 |
|------|----------|------|
| 连接池 | `util/pool.rs`, `async_impl/pool.rs` | 连接复用和生命周期管理 |
| 异步连接器 | `async_impl/connector/mod.rs` | `HttpConnector` trait 和实现 |
| 同步连接器 | `sync_impl/connector.rs` | `Connector` trait 和实现 |
| 调度器 | `util/dispatcher.rs` | 单连接管理，根据版本分发到 H1/H2/H3 |

#### 4.4 其他功能

| 功能 | 实现位置 | 证据 |
|------|----------|------|
| HTTP 代理 | `util/proxy.rs` | 支持 HTTP/HTTPS 代理和 Basic 认证 |
| 自动重定向 | `util/redirect.rs` | 3xx 状态码自动重定向 |
| DNS 解析 | `async_impl/dns/` | 支持默认解析器和 DoH |
| 进度追踪 | `util/progress.rs` | 上传下载进度回调 |
| 连接信息 | `util/information.rs` | 连接详情和协商信息 |
| 拦截器 | `util/interceptor.rs` | 连接生命周期拦截 |

### 5. 与其他组件的依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│             上层应用 (App Layer)                   │
│                                                        │
│  ┌──────────────────────────────────────────────┐         │
│  │       request (上传下载)                 │         │
│  │                                        │         │
│  └──────────────────────────────────────────────┘         │
│                                                        │
│                   ┌──────────────────────────────┐         │
│                   │      netstack (系统服务)    │         │
│                   └──────────────────────────────┘         │
│                                                        │
│  ┌──────────────────────────────────────────────┐         │
│  │       ylong_http (协议栈)            │         │
│  │  ┌────────────────────────────┐          │         │
│  │  │  ylong_http (基础组件) │          │         │
│  │  └────────────────────────────┘          │         │
│  │                                        │         │
│  │  ┌────────────────────────────┐          │         │
│  │  │ ylong_http_client (客户端)  │          │         │
│  │  └────────────────────────────┘          │         │
│  └──────────────────────────────────────────────┘         │
│                                                        │
└─────────────────────────────────────────────────────────────┘
```

**依赖组件**（来自 bundle.json:27-32）:
- `ylong_runtime` - OpenHarmony 异步运行时
- `openssl` - TLS/SSL 库
- `rust_libc` - Rust libc 绑定

### 6. 运行环境和资源

**系统要求**：
- 操作系统: OpenHarmony (standard)
- 架构: Linux/macOS（基于 libc）
- RAM: ~200KB 运行时占用（bundle.json:26）

**编译产物**：
- ylong_http: `libylong_http.a` (静态库)
- ylong_http_client: `libylong_http_client_inner.so` (共享库)

**证据**: GN Targets 分析（详见 [GN 目标梳理](06_GN_Targets.md)）

### 7. 与其他 HTTP 库的对比

| 特性 | ylong_http | curl | hyper | reqwest |
|------|-----------|------|--------|---------|
| 语言 | Rust | C | Rust | Rust |
| 运行时 | ylong_runtime/tokio | 无 | tokio | tokio |
| 同步 API | ✅ | ✅ | ❌ | ✅ |
| 异步 API | ✅ | ✅ | ✅ | ✅ |
| HTTP/2 | ✅ | ✅ | ✅ | ✅ |
| HTTP/3 | ⚠️ 代码存在 | ❌ | ❌ | ❌ |
| OpenHarmony 集成 | ✅ | ❌ | ❌ | ❌ |
| OpenSSL 集成 | ✅ | ✅ | ⚠️ (native-tls) | ⚠️ (native-tls) |

**定位差异**：ylong_http 专为 OpenHarmony 系统设计，支持 ylong_runtime，而其他库主要为通用用途。

---

## 相关跳转

- **[架构说明](03_Architecture.md)** - 详细的架构设计和组件交互
- **[目录结构与模块职责](02_Directory_Structure.md)** - 代码组织和模块划分
- **[对外 API](04_External_API.md)** - 公共 API 使用指南
- **[GN 目标梳理](06_GN_Targets.md)** - 构建系统和产物详解

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
