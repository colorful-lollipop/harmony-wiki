# 对外 API

> 本文档详细说明 ylong_http 对外暴露的所有公共 API

---

## 目的

本文档的目的是让读者在 40 分钟内：
- 了解项目对外暴露的所有公共 API
- 理解如何使用 HTTP 客户端
- 掌握配置选项和错误处理

## 适用范围

- ylong_http_client 的公共导出 API
- 不包含 N-API/JS 绑定（项目为纯 Rust 库）

---

## 关键结论

### 1. 接口类型说明

**ylong_http_client 是一个纯 Rust Library**，通过 `pub` 关键字对外暴露 API。

**无 N-API/JS 绑定证据**：
- 无 `napi_`, `NAPI_MODULE`, `napi_module_register` 等关键字
- 无 `*.d.ts` TypeScript 声明文件
- 无 C FFI 导出（`no_mangle` 未使用）
- 证据：全局代码搜索结果（详见 NOTES.md）

**使用方式**：
```toml
[dependencies]
ylong_http_client = { path = "../ylong_http_client" }
```

### 2. 核心类型导出

#### 2.1 HTTP 基础类型（从 ylong_http 重新导出）

**证据**: `ylong_http_client/src/lib.rs:24-35`

| 类型 | 定义位置 | 说明 |
|------|-----------|------|
| `EmptyBody` | ylong_http::body | 空消息体 |
| `TextBody` | ylong_http::body | 文本消息体 |
| `ReusableReader` | ylong_http::body | 可重用的 Body |
| `Method` | ylong_http::request | HTTP 方法枚举 |
| `Uri` | ylong_http::request | URI 结构体 |
| `Scheme` | ylong_http::request | URI 协议（http/https） |
| `Version` | ylong_http::version | HTTP 版本枚举 |
| `StatusCode` | ylong_http::response | HTTP 状态码枚举 |
| `Headers` | ylong_http::headers | HTTP 头部集合 |
| `Header` | ylong_http::headers | 单个 HTTP 头 |
| `HeaderName` | ylong_http::headers | HTTP 头名称 |
| `HeaderValue` | ylong_http::headers | HTTP 头值 |

#### 2.2 客户端类型（async_impl）

**证据**: `ylong_http_client/src/async_impl/mod.rs:61`

| 类型/别名 | 原始定义 | 说明 |
|-----------|-----------|------|
| `Client<HttpConnector>` | Client<HttpConnector> | 默认异步客户端类型 |
| `ClientBuilder` | client::ClientBuilder | 客户端构建器 |

**Client 主要方法**：
```rust
impl Client<HttpConnector> {
    // 发送请求（异步）
    pub async fn request<T: Body>(&self, request: Request<T>) -> Result<Response<T>, HttpClientError>;

    // 关闭客户端
    pub async fn close(self) -> Result<(), HttpClientError>;
}
```

**ClientBuilder 主要方法**：
```rust
impl ClientBuilder {
    // 设置连接超时
    pub fn timeout(mut self, timeout: Timeout) -> Self;

    // 设置连接池大小
    pub fn max_idle_connections(mut self, max: usize) -> Self;

    // 设置代理
    pub fn proxy(mut self, proxy: Proxy) -> Self;

    // 设置 TLS 配置
    pub fn tls_config(mut self, config: TlsConfig) -> Self;

    // 设置重定向策略
    pub fn redirect(mut self, redirect: Redirect) -> Self;

    // 构建 Client
    pub fn build<C: Connector>(self, connector: C) -> Result<Client<C>, HttpClientError>;
}
```

**证据**:
- `ylong_http_client/src/async_impl/client.rs:78-250` - Client 和 ClientBuilder 实现

#### 2.3 客户端类型（sync_impl）

**证据**: `ylong_http_client/src/sync_impl/mod.rs:81`

| 类型/别名 | 原始定义 | 说明 |
|-----------|-----------|------|
| `Client<HttpConnector>` | Client<HttpConnector> | 默认同步客户端类型 |
| `ClientBuilder` | client::ClientBuilder | 客户端构建器 |

**Client 主要方法**：
```rust
impl Client<C: Connector> {
    // 发送请求（同步，阻塞）
    pub fn request<T: Body>(&self, request: Request<T>) -> Result<Response<T>, HttpClientError>;

    // 关闭客户端
    pub fn close(self) -> Result<(), HttpClientError>;
}
```

**证据**:
- `ylong_http_client/src/sync_impl/client.rs:53-200` - Client 和 ClientBuilder 实现

### 3. 配置类型

#### 3.1 客户端配置

**证据**: `ylong_http_client/src/util/config/settings.rs:23-60`

| 配置项 | 类型 | 说明 |
|---------|------|------|
| `Timeout` | `Option<Duration>` | 连接、读取、写入超时 |
| `SpeedLimit` | - | 速度限制 |
| `Redirect` | `Redirect` | 重定向策略（自动/手动/禁用） |
| `Retry` | `Option<usize>` | 重试次数 |
| `Proxy` | `Proxy` | 代理配置 |

**使用示例**：
```rust
use ylong_http_client::async_impl::{Client, ClientBuilder};

async fn example() {
    let client = Client::builder()
        .timeout(Timeout::from_secs(30))
        .max_idle_connections(100)
        .proxy(Proxy::all("http://proxy.example.com:8080").build().unwrap())
        .build(HttpConnector::default())
        .unwrap();
}
```

**证据**:
- `ylong_http_client/src/util/config/settings.rs` - 配置结构定义

#### 3.2 TLS 配置

**证据**: `ylong_http_client/src/util/config/tls/adapter.rs:1-95`

| 配置项 | 类型 | 说明 |
|---------|------|------|
| `TlsConfig` | 结构体 | TLS 总配置 |
| `TlsConfigBuilder` | 结构体 | TLS 配置构建器 |
| `TlsVersion` | 枚举 | TLS 版本（V1_0, V1_1, V1_2, V1_3） |
| `TlsFileType` | 枚举 | 证书文件类型（PEM, DER） |
| `CertVerifier` | trait | 自定义证书验证器 |
| `ServerCerts` | 结构体 | 服务器证书信息 |
| `PubKeyPins` | 结构体 | 公钥固定配置 |

**TLS 配置项**：
```rust
pub struct TlsConfig {
    pub min_version: TlsVersion,
    pub max_version: TlsVersion,
    pub certificates: Option<Cert>,
    pub verifier: Option<Box<dyn CertVerifier>>,
    pub pins: Option<PubKeyPins>,
    pub accept_invalid_certs: bool,
    pub accept_invalid_hostnames: bool,
    pub danger_set_ca_list: bool,
}
```

**证据**:
- `ylong_http_client/src/util/config/tls/adapter.rs:21-95` - TLS 配置定义

### 4. Body 接口

#### 4.1 异步 Body

**证据**: `ylong_http_client/src/async_impl/mod.rs:29-30`

```rust
pub trait Body: Unpin + Sized {
    type Error: Into<Box<dyn Error + Send + Sync>>;

    // 读取 Body 数据（异步）
    fn poll_data(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>,
        buf: &mut [u8],
    ) -> Poll<Result<usize, Self::Error>>;

    // 读取 Trailer（异步）
    fn poll_trailer(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>,
    ) -> Poll<Result<Option<Headers>, Self::Error>>;
}
```

#### 4.2 同步 Body

**证据**: `ylong_http/body/mod.rs:59-172`

```rust
pub trait Body {
    type Error: Into<Box<dyn Error + Send + Sync>>;

    // 读取 Body 数据（同步）
    fn data(&mut self, buf: &mut [u8]) -> Result<usize, Self::Error>;

    // 读取 Trailer（同步）
    fn trailer(&mut self) -> Result<Option<Headers>, Self::Error>;
}
```

### 5. 错误处理

#### 5.1 HttpClientError

**证据**: `ylong_http_client/src/error.rs:31-71`

| ErrorKind | 说明 |
|-----------|------|
| `InvalidInput` | 无效输入参数 |
| `ConnectTimeout` | 连接超时 |
| `ReadTimeout` | 读取超时 |
| `WriteTimeout` | 写入超时 |
| `DnsError` | DNS 解析失败 |
| `RedirectError` | 重定向错误 |
| `TlsError` | TLS/SSL 错误 |
| `IoError` | I/O 错误 |
| `UserAborted` | 用户取消 |
| `Other` | 其他错误 |

**错误获取方法**：
```rust
pub fn error_kind(&self) -> ErrorKind;
pub fn io_error(&self) -> Option<io::Error>;
```

#### 5.2 HttpError

**证据**: `ylong_http/src/error.rs:82-100`

| ErrorKind | 说明 |
|-----------|------|
| `InvalidInput` | 无效输入 |
| `Uri(InvalidUri)` | URI 解析错误 |
| `H1(H1Error)` | HTTP/1.1 错误 |
| `H2(H2Error)` | HTTP/2 错误 |
| `H3(H3Error)` | HTTP/3 错误 |

### 6. 快速开始示例

#### 6.1 异步客户端

```rust
use ylong_http_client::async_impl::{Client, Request, Body};
use ylong_http::{Method, Uri, StatusCode};

async fn simple_get() -> Result<(), HttpClientError> {
    let client = Client::new();

    let request = Request::builder()
        .method(Method::GET)
        .url("https://example.com/api/data")
        .body(Body::empty())
        .unwrap();

    let response = client.request(request).await?;

    println!("Status: {}", response.status());
    println!("Body: {}", String::from_utf8_lossy(response.body()));

    Ok(())
}
```

**证据**: `ylong_http_client/examples/async_http.rs` - 示例代码

#### 6.2 同步客户端

```rust
use ylong_http_client::sync_impl::{Client, Request};
use ylong_http::{Method, Uri};

fn simple_get() -> Result<(), HttpClientError> {
    let client = Client::new();

    let request = Request::builder()
        .method(Method::GET)
        .url("https://example.com/api/data")
        .body(())  // EmptyBody
        .unwrap();

    let response = client.request(request)?;

    println!("Status: {}", response.status());
    println!("Body: {}", String::from_utf8_lossy(response.body()));

    Ok(())
}
```

**证据**: `ylong_http_client/examples/sync_http.rs` - 示例代码

---

## 相关跳转

- **[项目概览](00_Overview.md) - 项目定位和核心能力
- **[目录结构与模块职责](02_Directory_Structure.md)** - 代码组织和模块划分
- **[内部 API](05_Internal_API.md)** - 扩展点和 trait 定义

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
