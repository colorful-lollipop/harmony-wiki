# 内部 API

> 本文档说明 ylong_http 的内部接口、可扩展点和 trait 定义

---

## 目的

本文档的目的是让开发者了解：
- 可自定义和扩展的内部接口
- 主要 trait 定义和实现位置
- 如何实现自定义 Connector、CertVerifier 等
- 连接池和调度器的扩展机制

## 适用范围

- ylong_http 和 ylong_http_client 的 trait 定义
- 可扩展的接口点

---

## 关键结论

### 1. 扩展接口概览

| 接口 | 用途 | 定义位置 | 稳定性 |
|------|------|-----------|---------|
| `Connector` | 自定义连接创建逻辑 | `async_impl/connector/mod.rs`, `sync_impl/connector.rs` | ✅ 稳定 |
| `Body` (async) | 自定义异步 Body | `ylong_http/body/mod.rs` | ✅ 稳定 |
| `Body` (sync) | 自定义同步 Body | `ylong_http/body/mod.rs` | ✅ 稳定 |
| `CertVerifier` | 自定义证书验证逻辑 | `util/config/tls/verifier/mod.rs` | ✅ 稳定 |
| `Interceptor` | 连接生命周期拦截 | `util/interceptor/mod.rs` | ⚠️ 内部（pub(crate)） |
| `Pool` | 连接池接口 | `util/pool.rs` | ⚠️ 内部（pub(crate)） |
| `Resolver` | DNS 解析器 | `async_impl/dns/mod.rs` | ✅ 稳定 |

### 2. Connector Trait（异步）

**证据**: `ylong_http_client/src/async_impl/connector/mod.rs:1-150`

**定义**：
```rust
pub trait Connector: Send + Sync {
    type Stream: AsyncRead + AsyncWrite + Unpin;

    // 异步建立连接
    async fn connect(
        &self,
        uri: &Uri,
        tls: Option<&TlsConfig>,
    ) -> Result<Self::Stream, HttpClientError>;
}
```

**默认实现 - HttpConnector**：
- 支持 HTTP 和 HTTPS
- 自动处理 ALPN 协商
- 支持代理连接

**自定义示例**：
```rust
use ylong_http_client::async_impl::{Connector, TlsConfig};
use ylong_http::Uri;
use std::io;

struct MyConnector;

impl Connector for MyConnector {
    type Stream = MyStream;

    async fn connect(&self, uri: &Uri, tls: Option<&TlsConfig>)
        -> Result<Self::Stream, HttpClientError> {
        // 实现自定义连接逻辑
        Ok(MyStream)
    }
}
```

**证据**:
- `ylong_http_client/src/async_impl/connector/mod.rs` - HttpConnector 实现

### 3. Connector Trait（同步）

**证据**: `ylong_http_client/src/sync_impl/connector.rs:1-80`

**定义**：
```rust
pub trait Connector: Send + Sync {
    // 同步建立连接
    fn connect(
        &self,
        uri: &Uri,
        tls: Option<&TlsConfig>,
    ) -> Result<TcpStream, HttpClientError>;
}
```

**默认实现**：
- 基于标准 `std::net::TcpStream`
- 支持 TLS 包装
- 支持代理连接

**证据**:
- `ylong_http_client/src/sync_impl/connector.rs` - 默认 Connector 实现

### 4. Body Trait（异步）

**证据**: `ylong_http/body/mod.rs:173-254`

**定义**（已在对外 API 中说明）：

```rust
pub trait Body: Unpin + Sized {
    type Error: Into<Box<dyn Error + Send + Sync>>;

    fn poll_data(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>,
        buf: &mut [u8],
    ) -> Poll<Result<usize, Self::Error>>;

    fn poll_trailer(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>,
    ) -> Poll<Result<Option<Headers>, Self::Error>>;
}
```

**标准实现**：
- `EmptyBody` - 空 Body
- `TextBody` - 文本 Body
- `ChunkBody` - 分块传输 Body
- `ReusableReader` - 可重用 Body（实现 `AsyncRead`）

**自定义示例**：
```rust
use ylong_http::body::async_impl::Body;
use core::pin::Pin;
use core::task::{Context, Poll};

struct MyBody {
    data: Vec<u8>,
    position: usize,
}

impl Body for MyBody {
    type Error = std::io::Error;

    fn poll_data(
        mut self: Pin<&mut Self>,
        cx: &mut Context<'_>,
        buf: &mut [u8],
    ) -> Poll<Result<usize, Self::Error>> {
        let len = self.data.len() - self.position;
        if len == 0 {
            return Poll::Ready(Ok(0));
        }
        let to_copy = std::cmp::min(len, buf.len());
        buf[..to_copy].copy_from_slice(&self.data[self.position..self.position + to_copy]);
        self.position += to_copy;
        Poll::Ready(Ok(to_copy))
    }

    fn poll_trailer(
        self: Pin<&mut Self>,
        _cx: &mut Context<'_>,
    ) -> Poll<Result<Option<Headers>, Self::Error>> {
        Poll::Ready(Ok(None))
    }
}
```

**证据**:
- `ylong_http/src/body/mod.rs` - Body trait 定义和标准实现

### 5. CertVerifier Trait

**证据**: `ylong_http_client/src/util/config/tls/verifier/mod.rs:18-40`

**定义**：
```rust
pub trait CertVerifier: Send + Sync {
    // 验证服务器证书
    fn verify(&self, certs: &ServerCerts<'_>) -> Result<(), HttpClientError>;
}
```

**默认实现 - DefaultCertVerifier**：
- 执行标准证书链验证
- 验证证书有效期
- 验证主机名

**自定义示例**：
```rust
use ylong_http_client::util::config::CertVerifier;

struct MyVerifier;

impl CertVerifier for MyVerifier {
    fn verify(&self, certs: &ServerCerts<'_>) -> Result<(), HttpClientError> {
        // 自定义验证逻辑
        // 例如：检查证书黑名单
        // 例如：检查证书指纹
        Ok(())
    }
}
```

**使用方式**：
```rust
let config = TlsConfigBuilder::new()
    .verifier(Box::new(MyVerifier))
    .build();
```

**证据**:
- `ylong_http_client/src/util/config/tls/verifier/openssl.rs` - 默认验证器实现

### 6. Interceptor Trait

**注意**：此接口为内部使用（pub(crate)），不对外暴露

**证据**: `ylong_http_client/src/util/interceptor/mod.rs:1-60`

**定义**：
```rust
pub trait Interceptor: Send + Sync {
    // 连接建立前回调
    fn before_connect(&self, uri: &Uri) -> Option<HttpClientError>;

    // 连接建立后回调
    fn after_connect(&self, conn_info: &ConnInfo) -> Option<HttpClientError>;

    // 连接关闭前回调
    fn before_close(&self, conn_info: &ConnInfo);

    // 连接关闭后回调
    fn after_close(&self, conn_info: &ConnInfo);
}
```

**默认实现 - IdleInterceptor**：
- 不执行任何操作（空实现）
- 用于类型系统占位

**证据**:
- `ylong_http_client/src/util/interceptor/mod.rs` - Interceptor trait 和 IdleInterceptor

### 7. Resolver Trait

**证据**: `ylong_http_client/src/async_impl/dns/mod.rs:1-80`

**定义**：
```rust
pub trait Resolver: Send + Sync {
    // 异步解析 DNS
    fn resolve(&self, host: &str, port: u16) -> impl Future<Output = Result<Vec<SocketAddr>, HttpClientError>>;
}
```

**默认实现 - DefaultDnsResolver**：
- 使用系统 DNS 解析
- 支持缓存

**自定义示例**：
```rust
use ylong_http_client::async_impl::dns::Resolver;
use std::net::SocketAddr;

struct MyResolver;

impl Resolver for MyResolver {
    fn resolve(&self, host: &str, port: u16)
        -> impl Future<Output = Result<Vec<SocketAddr>, HttpClientError>> {
        async move {
            // 自定义 DNS 解析逻辑
            // 例如：使用 DoH (DNS-over-HTTPS)
            Ok(vec![])
        }
    }
}
```

**证据**:
- `ylong_http_client/src/async_impl/dns/default.rs` - DefaultDnsResolver 实现

### 8. 连接池接口（内部）

**证据**: `ylong_http_client/src/util/pool.rs:1-100`

**定义**：
```rust
pub trait Pool: Send + Sync {
    type Stream: Stream;

    // 从池中获取连接
    fn get(&self, uri: &Uri) -> Result<Self::Stream, HttpClientError>;

    // 归还连接到池中
    fn release(&self, stream: Self::Stream);
}
```

**异步连接池实现**（async_impl/pool.rs）：
- 支持最大连接数限制
- 支持空闲超时
- 自动关闭过时连接

**证据**:
- `ylong_http_client/src/async_impl/pool.rs` - ConnPool 实现和 Pool trait

### 9. 稳定性说明

| 接口 | 稳定性 | 兼容性保证 | 变更风险 |
|------|---------|------------|---------|
| `Connector` | ✅ 稳定 | 主版本内保证兼容 | 低（版本号变更可能影响） |
| `Body` | ✅ 稳定 | 主版本内保证兼容 | 低 |
| `CertVerifier` | ✅ 稳定 | 主版本内保证兼容 | 低 |
| `Resolver` | ✅ 稳定 | 主版本内保证兼容 | 低 |
| `Interceptor` | ⚠️ 内部 | 无兼容性保证 | 中（内部重构可能影响） |
| `Pool` | ⚠️ 内部 | 无兼容性保证 | 中（内部重构可能影响） |

**证据**:
- 各 trait 定义位置和注释说明
- `ylong_http_client/src/lib.rs` - 公共导出仅包含稳定接口

---

## 相关跳转

- **[对外 API](04_External_API.md)** - 公共 API 使用指南
- **[目录结构与模块职责](02_Directory_Structure.md)** - 代码组织和模块划分

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
