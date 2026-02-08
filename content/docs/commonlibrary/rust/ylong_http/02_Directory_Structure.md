# 目录结构与模块职责

> 本文档详细说明 ylong_http 项目的代码组织、模块划分和职责

---

## 目的

本文档的目的是让读者在 20 分钟内：
- 理解项目的代码组织结构
- 了解各模块的职责和边界
- 熟悉关键模块的位置和功能

## 适用范围

- ylong_http 模块
- ylong_http_client 模块
- 不包含 test/ 目录

---

## 关键结论

### 1. 总体目录结构

```
ylong_http/
├── wiki/                    # Wiki 文档
├── ylong_http/              # HTTP 协议基础库
│   ├── src/
│   │   ├── body/            # Body trait 和实现
│   │   ├── h1/              # HTTP/1.1 编解码
│   │   ├── h2/              # HTTP/2 编解码和帧处理
│   │   ├── h3/              # HTTP/3 编解码和帧处理
│   │   ├── huffman/         # Huffman 编码
│   │   ├── request/         # HTTP Request 类型
│   │   ├── response/        # HTTP Response 类型
│   │   ├── headers.rs        # HTTP 头处理（41KB）
│   │   ├── error.rs         # HTTP 错误定义
│   │   ├── pseudo.rs        # HTTP/2/3 伪头（17KB）
│   │   ├── util/            # 内部工具
│   │   ├── version.rs        # HTTP 版本管理（4KB）
│   │   └── lib.rs          # 库入口（62 行）
│   ├── examples/            # 使用示例
│   ├── BUILD.gn            # GN 构建配置（静态库）
│   └── Cargo.toml          # Rust crate 配置
└── ylong_http_client/       # HTTP 客户端库
    ├── src/
    │   ├── async_impl/        # 异步客户端实现
    │   │   ├── client.rs       # Client 主类型（约 2000+ 行）
    │   │   ├── conn/           # 连接管理
    │   │   ├── connector/      # 连接创建
    │   │   ├── downloader/     # 下载器
    │   │   ├── ssl_stream/      # TLS 流
    │   │   ├── uploader/        # 上传器
    │   │   ├── dns/            # DNS 解析
    │   │   ├── pool.rs         # 连接池
    │   │   ├── request.rs       # 请求构建
    │   │   ├── response.rs      # 响应处理
    │   │   ├── http_body.rs     # Body 处理
    │   │   ├── timeout.rs      # 超时
    │   │   ├── mix.rs          # 混合实现
    │   │   └── mod.rs          # 模块入口
    │   ├── sync_impl/          # 同步客户端实现
    │   │   ├── client.rs       # Client 主类型（约 800 行）
    │   │   ├── conn/           # 连接管理
    │   │   ├── connector.rs     # 连接创建
    │   │   ├── pool.rs         # 连接池
    │   │   ├── reader.rs       # Body 读取
    │   │   └── mod.rs          # 模块入口
    │   ├── util/               # 通用组件
    │   │   ├── c_openssl/      # OpenSSL FFI（713 行）
    │   │   │   ├── ffi/          # FFI 定义（8 个文件）
    │   │   │   ├── ssl/          # SSL 封装
    │   │   │   ├── x509.rs       # X509 证书
    │   │   │   └── bio.rs        # BIO 操作
    │   │   ├── config/         # 配置
    │   │   │   ├── tls/         # TLS 配置
    │   │   │   │   ├── verifier/   # 证书验证器
    │   │   │   │   │   ├── openssl.rs
    │   │   │   │   │   └── alpn/       # ALPN 配置
    │   │   │   │   ├── adapter.rs    # TLS 适配器
    │   │   │   │   └── settings.rs  # 各种配置
    │   │   ├── dispatcher.rs    # 连接调度器
    │   │   ├── pool.rs         # 通用连接池接口
    │   │   ├── proxy.rs        # HTTP 代理
    │   │   ├── redirect.rs      # 自动重定向
    │   │   ├── normalizer.rs    # 请求规范化
    │   │   ├── interceptor.rs  # 拦截器
    │   │   ├── monitor.rs       # 性能监控
    │   │   ├── progress.rs     # 进度追踪
    │   │   ├── information.rs  # 连接信息
    │   │   ├── h2/            # HTTP/2 连接
    │   │   ├── h3/            # HTTP/3 连接
    │   │   ├── test_utils.rs    # 测试工具
    │   │   ├── alt_svc.rs      # Alt-SVC 头
    │   │   ├── base64.rs       # Base64 编码
    │   │   ├── data_ref.rs     # 数据引用
    │   │   └── mod.rs          # 模块入口
    │   ├── error.rs             # 客户端错误定义
    │   └── lib.rs              # 库入口（119 行）
    ├── examples/                # 使用示例
    ├── BUILD.gn                # GN 构建配置（共享库）
    └── Cargo.toml              # Rust crate 配置
```

**证据**: `find . -type d | head -20 | grep -v test` 命令输出

### 2. ylong_http 模块详解

#### 2.1 body/ 模块 - HTTP 消息体

**职责**: 定义 HTTP 消息体抽象和具体实现

| 文件 | 类型/职责 | 证据 |
|------|-----------|------|
| `mod.rs` | Body trait 定义，sync_impl 和 async_impl 子模块 | `body/mod.rs:59` |
| `empty.rs` | EmptyBody - 空消息体 | `body/mod.rs:51` |
| `text.rs` | TextBody - 文本消息体 | `body/mod.rs:52` |
| `chunk.rs` | ChunkBody - 分块传输 | `body/mod.rs:50` |
| `mime/` | Multipart 消息体 | `body/mod.rs:53-55` |

**公共导出**（body/mod.rs:49-56）:
- `EmptyBody`, `TextBody`, `ChunkBody`, `ChunkExt`, `Chunks`
- `MimeMulti`, `MimeMultiBuilder`, `MimePart`, `MimePartBuilder`, `MimePartEncoder`
- `Body` (sync_impl), `Body` (async_impl)

**Body trait**:
```rust
pub trait Body {
    type Error: Into<Box<dyn Error + Send + Sync>>;
    fn data(&mut self, buf: &mut [u8]) -> Result<usize, Self::Error>;
    fn trailer(&mut self) -> Result<Option<Headers>, Self::Error>;
}
```

#### 2.2 h1/ 模块 - HTTP/1.1

**职责**: HTTP/1.1 请求编码和响应解码

| 文件 | 职责 | 证据 |
|------|------|------|
| `request/encoder.rs` | HTTP/1.1 请求编码器 | `h1/mod.rs:24` |
| `response/decoder.rs` | HTTP/1.1 响应解码器 | `h1/mod.rs:25` |
| `error.rs` | HTTP/1.1 错误定义 | `h1/mod.rs:26` |

#### 2.3 h2/ 模块 - HTTP/2

**职责**: HTTP/2 帧编解码、HPACK、流管理

| 文件 | 职责 | 大小估算 |
|------|------|---------|
| `encoder.rs` | HTTP/2 帧编码器 | ~80KB |
| `decoder.rs` | HTTP/2 帧解码器 | ~65KB |
| `frame.rs` | HTTP/2 帧定义 | ~23KB |
| `hpack/` | HPACK 头部压缩 | ~20KB |
| `mod.rs` | 模块入口 | ~3KB |

#### 2.4 h3/ 模块 - HTTP/3

**职责**: HTTP/3 帧编解码、QPACK

| 文件 | 职责 | 大小估算 |
|------|------|---------|
| `encoder.rs` | HTTP/3 帧编码器 | ~30KB |
| `decoder.rs` | HTTP/3 帧解码器 | ~34KB |
| `frame.rs` | HTTP/3 帧定义 | ~8KB |
| `qpack/` | QPACK 头部压缩 | ~20KB |
| `mod.rs` | 模块入口 | ~1KB |

#### 2.5 其他核心模块

| 模块 | 职责 | 证据 |
|------|------|------|
| `request/` | HTTP Request 结构、Method、Uri、RequestBuilder | `request/mod.rs:1-239` |
| `response/` | HTTP Response 结构、StatusCode | `response/mod.rs:1-57` |
| `headers.rs` | HTTP 头处理（Header, HeaderName, HeaderValue, Headers） | `headers.rs:1-1168` |
| `error.rs` | HTTP 错误定义（HttpError, ErrorKind） | `error.rs:1-100` |
| `version.rs` | HTTP 版本枚举（HTTP1_0, HTTP1_1, HTTP2, HTTP3） | `version.rs:1-85` |

### 3. ylong_http_client 模块详解

#### 3.1 async_impl/ 模块 - 异步客户端

**职责**: 基于 ylong_runtime/tokio 的非阻塞 HTTP 客户端实现

| 子模块 | 职责 | 主要类型 |
|--------|------|----------|
| `client.rs` | 客户端主类型和构建器 | `Client<C>`, `ClientBuilder` |
| `connector/` | 连接创建和管理 | `HttpConnector`, `TlsConnector` |
| `conn/` | 连接对象（H1, H2, H3） | `HttpConn`, `H2Conn`, `H3Conn` |
| `pool.rs` | 连接池管理 | `ConnPool` |
| `downloader/` | 文件下载器 | `Downloader`, `DownloadOperator` |
| `uploader/` | 文件上传器 | `Uploader`, `UploadOperator` |
| `ssl_stream/` | TLS 流封装 | `SslStream` |
| `dns/` | DNS 解析 | `Resolver` trait, `DefaultDnsResolver` |
| `request.rs` | 请求构建和 Body 处理 | `Request`, `Body`, `PercentEncoder` |
| `response.rs` | 响应处理 | `Response` |
| `http_body.rs` | Body 处理工具 | `HttpBody` trait |
| `timeout.rs` | 超时处理 | `Timeout` trait |

**公共导出**（async_impl/mod.rs）:
- `ClientBuilder`, `Client<HttpConnector>` (type alias)
- `Connector`, `HttpConnector`
- `Downloader`, `Uploader` 及相关 Builder
- `Body`, `Request`, `Response`
- DNS 相关类型

#### 3.2 sync_impl/ 模块 - 同步客户端

**职责**: 阻塞式 HTTP 客户端实现，无运行时依赖

| 子模块 | 职责 | 主要类型 |
|--------|------|----------|
| `client.rs` | 客户端主类型和构建器 | `Client<C>`, `ClientBuilder` |
| `connector.rs` | 连接创建 | `Connector` trait 和实现 |
| `conn/` | 连接对象管理 | `HttpConn`, `H2Conn` |
| `pool.rs` | 连接池管理 | `ConnPool` |
| `reader.rs` | Body 读取处理 | `BodyProcessor`, `BodyReader` |

**公共导出**（sync_impl/mod.rs）:
- `Client`, `ClientBuilder`
- `Connector`
- `HttpBody` trait
- Body 相关类型（从 ylong_http 重新导出）

#### 3.3 util/ 模块 - 通用组件

**职责**: 同步/异步客户端共享的工具和配置

| 子模块 | 职责 | 主要类型 |
|--------|------|----------|
| `c_openssl/` | OpenSSL FFI 适配器（713 行 unsafe 代码） | `TlsConfig`, `CertVerifier`, `PubKeyPins` |
| `config/` | 各种配置选项 | `ClientConfig`, `Proxy`, `Redirect`, `Retry`, `Timeout` |
| `dispatcher.rs` | 连接调度器，管理单连接生命周期 | `Dispatcher` trait, `Conn` enum |
| `pool.rs` | 通用连接池接口 | `Pool` trait |
| `proxy.rs` | HTTP 代理实现 | `Proxy`, `ProxyBuilder` |
| `redirect.rs` | 自动重定向处理 | `Redirect`, `Trigger` |
| `normalizer.rs` | 请求规范化（Host 大小写等） | `RequestFormatter` |
| `interceptor.rs` | 连接拦截器 | `Interceptor` trait, `IdleInterceptor` |
| `monitor/` | 性能监控和时间统计 | `TimeGroup` |
| `progress.rs` | 上传下载进度追踪 | - |
| `information.rs` | 连接信息和协商信息 | `ConnInfo`, `ConnData`, `NegotiateInfo` |
| `h2/` | HTTP/2 连接管理 | - |
| `h3/` | HTTP/3 连接管理 | - |

**TLS 相关导出**（util/mod.rs:53-66）:
- `TlsConfig`, `TlsConfigBuilder`
- `CertVerifier`, `ServerCerts`
- `Cert`, `Certificate`, `PubKeyPins`, `PubKeyPinsBuilder`
- `TlsFileType`, `TlsVersion`

**配置相关导出**（util/mod.rs:62）:
- `Proxy`, `ProxyBuilder`
- `Redirect`, `Retry`, `Timeout`, `SpeedLimit`

---

## 相关跳转

- **[对外 API](04_External_API.md)** - 公共 API 使用指南
- **[内部 API](05_Internal_API.md)** - 内部接口和可扩展点
- **[架构说明](03_Architecture.md)** - 模块交互和数据流

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
