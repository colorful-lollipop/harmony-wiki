# 配置标志

> 本文档说明 ylong_http 项目支持的所有 Feature Flags 和编译选项

---

## 目的

本文档详细说明 ylong_http 项目支持的：
- 所有 Feature Flags
- 编译时的行为差异
- 如何选择和组合 features

## 适用范围

- ylong_http/Cargo.toml
- ylong_http_client/Cargo.toml

---

## 关键结论

### 1. ylong_http Features

**来源**: `ylong_http/Cargo.toml:18-28`

| Feature | 说明 | 默认启用 | 依赖 features | GN 配置 |
|---------|------|----------|-------------|----------|
| `http1_1` | 启用 HTTP/1.1 支持 | ✅ 是 | 无 | `"http1_1"` |
| `http2` | 启用 HTTP/2 支持 | ✅ 是 | 无 | `"http2"` |
| `http3` | 启用 HTTP/3 支持 | ❌ 否 | 无 | 未配置 |
| `huffman` | 启用 Huffman 编码（用于 HPACK/QPACK） | ✅ 是 | 无 | `"huffman"` |
| `tokio_base` | 使用 tokio 作为异步运行时 | ❌ 否 | ylong_runtime | 未配置 |
| `ylong_base` | 使用 ylong_runtime 作为异步运行时 | ✅ 是 | 无 | `"ylong_base"` |

**Feature 依赖关系**：
- `http2` → `huffman` (HPACK 需要 Huffman)
- `http3` → `huffman` (QPACK 需要 Huffman)

**代码证据**: `ylong_http/Cargo.toml:23-27` - Feature 定义

### 2. ylong_http_client Features

**来源**: `ylong_http_client/Cargo.toml:11-26`

| Feature | 说明 | 默认启用 | 依赖 features | GN 配置 |
|---------|------|----------|-------------|----------|
| `async` | 启用异步客户端接口 | ✅ 是 | http1_1, http2 | `"async"` |
| `sync` | 启用同步客户端接口 | ❌ 否 | http1_1, http2 | 未配置 |
| `http1_1` | 启用 HTTP/1.1 支持 | ✅ 是 | ylong_base | `"http1_1"` |
| `http2` | 启用 HTTP/2 支持 | ✅ 是 | ylong_base, huffman | `"http2"` |
| `http3` | 启用 HTTP/3 支持 | ❌ 否 | http1_1, http2, huffman, quiche | 未配置 |
| `ylong_base` | 使用 ylong_runtime | ✅ 是 | async, http1_1, http2 | `"ylong_base"` |
| `tokio_base` | 使用 tokio | ❌ 否 | async, http1_1, http2 | 未配置 |
| `tls_default` | 使用默认 TLS 配置 | ❌ 否 | c_openssl_3_0, __tls | 未配置 |
| `c_openssl_3_0` | 使用 OpenSSL 3.0 TLS | ✅ 是 | __tls, __c_openssl | `"c_openssl_3_0"` |
| `c_openssl_1_1` | 使用 OpenSSL 1.1 TLS | ❌ 否 | __tls, __c_openssl | 未配置 |
| `c_boringssl` | 使用 BoringSSL | ❌ 否 | __tls | 未配置 |
| `__tls` | TLS 支持（内部标记） | ✅ 是 | __c_openssl | `"__tls"` |
| `__c_openssl` | OpenSSL 集成（内部标记） | ✅ 是 | __tls, libc | `"__c_openssl"` |

**运行时选择**：
- 默认：`ylong_base` (OpenHarmony 异步运行时)
- 可选：`tokio_base` (第三方 tokio 运行时)

**代码证据**: `ylong_http_client/Cargo.toml:11-26` - Feature 定义

### 3. Feature 组合示例

#### 3.1 使用默认异步客户端

```toml
[dependencies]
ylong_http_client = { path = "../ylong_http_client", features = ["async"] }

# 或在代码中
use ylong_http_client::async_impl::Client;
```

**说明**：启用 `async` 和 `ylong_base` features，使用 OpenHarmony 异步运行时

#### 3.2 使用 tokio 运行时

```toml
[dependencies]
ylong_http_client = { path = "../ylong_http_client", features = ["async", "tokio_base"] }

# GN 配置中需要额外配置
rustflags = [ "--cfg=feature=\"tokio_base\"" ]
```

**说明**：适用于非 OpenHarmony 环境，使用 tokio 而非 ylong_runtime

#### 3.3 同步客户端

```toml
[dependencies]
ylong_http_client = { path = "../ylong_http_client", features = ["sync"] }
```

**说明**：不启用 `async` feature，编译时仅包含同步实现

#### 3.4 启用 HTTP/2

```toml
[dependencies]
ylong_http_client = { path = "../ylong_http_client", features = ["async", "http2"] }
```

**说明**：同时启用 HTTP/2 和异步支持，需要 `ylong_base` 和 `huffman` features

#### 3.5 启用 TLS

```toml
[dependencies]
ylong_http_client = { path = "../ylong_http_client", features = ["async", "c_openssl_3_0"] }
```

**说明**：启用 TLS 支持，需要 `__tls` 和 `__c_openssl` features

### 4. GN 配置传递

#### 4.1 ylong_http 特性传递

```gn
# GN 配置中的 features 直接传递给 Rust 编译器
features = [
    "http1_1",
    "huffman",
    "http2",
    "ylong_base",
]
```

**等效的 Rustflags**：
```gn
rustflags = [
    "--cfg=feature=\"http1_1\"",
    "--cfg=feature=\"huffman\"",
    "--cfg=feature=\"http2\"",
    "--cfg=feature=\"ylong_base\""
]
```

#### 4.2 ylong_http_client 特性传递

```gn
# 客户端传递更多 features
features = [
    "async",
    "c_openssl_3_0",
    "http1_1",
    "http2",
    "ylong_base",
    "__c_openssl",
    "__tls",
]
```

**等效的 Rustflags**：
```gn
rustflags = [
    "--cfg=feature=\"async\"",
    "--cfg=feature=\"c_openssl_3_0\"",
    "--cfg=feature=\"http1_1\"",
    "--cfg=feature=\"http2\"",
    "--cfg=feature=\"ylong_base\"",
    "--cfg=feature=\"__c_openssl\"",
    "--cfg=feature=\"__tls\""
]
```

### 5. 环境变量

#### 5.1 日志级别控制

```bash
# 设置日志级别（RUST_LOG）
export RUST_LOG=ylong_http_client=info
export RUST_LOG=ylong_http=debug

# 查看详细日志
# RUST_LOG=trace 表示最详细，error 表示仅错误
```

**支持的日志**：
- `ylong_http_client` - 客户端日志
- `ylong_http` - 协议层日志

**代码证据**: ylong_http_client/src/lib.rs - 重新导出并使用环境日志

#### 5.2 OpenSSL 调试

```bash
# 启用 OpenSSL 详细错误日志
export OPENSSL_DEBUG=1024

# 查看错误堆栈
export RUST_BACKTRACE=1
```

### 6. 常见 Feature 组合

| 使用场景 | 推荐的 Features | 说明 |
|-----------|------------------|------|
| OpenHarmony 异步客户端 | `async`, `ylong_base`, `http1_1`, `http2` | 默认配置，性能最佳 |
| OpenHarmony HTTPS | `async`, `c_openssl_3_0`, `__tls`, `http1_1`, `http2` | 启用 TLS 支持 |
| 非 OH 环境异步 | `async`, `tokio_base`, `http1_1`, `http2` | 使用 tokio 运行时 |
| 同步客户端 | `sync`, `http1_1`, `http2` | 阻塞式 API |
| HTTP/3 (实验性) | `async`, `http3`, `quiche` | 需要额外依赖 |

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
