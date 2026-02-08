# 安全风险评审

> 本文档基于代码证据，对 ylong_http 项目进行全面的安全风险分析

---

## 目的

本文档的目的是让读者在 20 分钟内了解：
- 项目的攻击面和威胁模型
- 潜在的安全风险和可利用点
- TLS/SSL 配置和证书验证机制
- 不安全代码位置和注意事项

## 适用范围

- TLS/SSL 安全
- 输入验证和边界检查
- 内存安全和并发
- 网络安全

---

## 关键结论

### 1. 威胁模型

```
┌─────────────────────────────────────────────────────────────┐
│                    威胁模型                              │
├─────────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────────────────────────────┐         │
│  │       外部攻击面                        │         │
│  │                                        │  │
│  │  ┌──────────────────────────────┐          │  │
│  │  │  网络数据           │          │  │
│  │  │  - HTTP 请求/响应     │          │  │
│  │  │  - TLS 握手           │          │  │
│  │  └──────────────────────────────┘          │  │
│  │                                        │  │
│  │  ┌──────────────────────────────┐          │  │
│  │  │  本地攻击面             │          │  │
│  │  │  - HTTP 头注入         │          │  │
│  │  │  - 证书验证            │          │  │
│  │  │  - 代理认证            │          │  │
│  │  └──────────────────────────────┘          │  │
│  └──────────────────────────────────────────────┘         │
│                                                         │
└─────────────────────────────────────────────────────────────┘
```

### 2. TLS/SSL 证书验证机制

#### 2.1 证书验证器 Trait

**证据**: `ylong_http_client/src/util/config/tls/verifier/mod.rs:18`

```rust
pub trait CertVerifier: Send + Sync {
    fn verify(&self, certs: &ServerCerts<'_>) -> Result<(), HttpClientError>;
}
```

**默认实现** - DefaultCertVerifier:
- 执行标准证书链验证
- 检查证书有效期
- 验证主机名（SNI）

**自定义验证支持**：
- 开发者可实现 `CertVerifier` trait 进行自定义验证
- 可用于实现证书黑名单、白名单、证书指纹验证等

#### 2.2 公钥固定（Public Key Pinning）

**证据**: `ylong_http_client/src/util/c_openssl/verify/pinning.rs`

**策略**：
- `RootCertificate` - 固定根证书公钥
- `LeafCertificate` - 固定叶证书公钥

**实现**：
```rust
pub struct PubKeyPins {
    pins: Vec<[u8; 32]>,  // SHA256 哈希列表
}

pub fn sha256_digest(cert: &[u8], size: usize, digest: &mut [u8; 32])
    -> Result<(), HttpClientError>
```

**安全性**：
- 防止中间人攻击（MITM）
- 即使证书链有效，如公钥不匹配则拒绝连接
- SHA256 哈希实现基于 OpenSSL（验证代码）

**风险**：
- 公钥轮换时必须及时更新 pins
- 配置不当会导致合法连接被拒绝

**证据**:
- `ylong_http_client/src/util/c_openssl/verify/pinning.rs:1-372` - 公钥固定实现

### 3. Unsafe 代码统计

**总 unsafe 块数量**: 约 140+ 处

**主要位置**：
- `ylong_http_client/src/util/c_openssl/ffi/` - OpenSSL FFI 定义（8 个文件）
- `ylong_http_client/src/util/c_openssl/ssl/` - SSL 封装
- `ylong_http_client/src/util/c_openssl/x509.rs` - X509 证书处理
- `ylong_http_client/src/util/c_openssl/bio.rs` - BIO 操作

**FFI 相关 unsafe**：
- FFI 函数调用：`unsafe { OpenSSL_function(...) }`
- 指针转换：`unsafe { ptr as *mut Type }`
- 字符串转换：`unsafe { CStr::from_ptr(...) }`

**非 FFI unsafe**：
- 请求处理中的 UTF-8 转换
- HTTP 头部解析中的字符串操作
- 连接池中的锁操作

**证据**:
- `ylong_http_client/src/util/c_openssl/ffi/*.rs` - 大量 `unsafe` 关键字

### 4. 可被利用点

#### 4.1 证书验证绕过

**风险等级**: 🔴 高

**证据**: `ylong_http_client/src/util/config/tls/adapter.rs`

```rust
pub struct TlsConfig {
    pub accept_invalid_certs: bool,      // ⚠️ 危险选项
    pub accept_invalid_hostnames: bool,  // ⚠️ 危险选项
    pub danger_set_ca_list: bool,         // ⚠️ 危险选项
    // ...
}
```

**触发条件**：用户设置 `TlsConfig` 中的危险标志

**影响**：
- `accept_invalid_certs`: 接受过期、自签名、吊销的证书
- `accept_invalid_hostnames`: 接受主机名不匹配的证书
- `danger_set_ca_list`: 不使用系统 CA 列表

**攻击路径**：
1. 攻击者搭建恶意 HTTPS 服务器，使用自签名证书
2. 诱导用户使用上述危险配置（可能通过环境变量或配置文件）
3. 成功绕过证书验证，建立到受信任域名的连接

**修复建议**：
1. 移除危险选项或默认设置为 `false`
2. 在 API 文档中明确标注危险配置的风险
3. 考虑在 `ClientBuilder` 中禁止设置危险选项

#### 4.2 HTTP 代理认证泄露

**风险等级**: 🟡 中

**证据**: `ylong_http_client/src/util/proxy.rs`

```rust
pub struct Proxy {
    pub username: Option<String>,
    pub password: Option<String>,
    // ...
}

// Basic 认证头生成
pub fn basic_auth_header(&self) -> Option<HeaderValue> {
    match (&self.username, &self.password) {
        (Some(user), Some(pass)) => {
            let credentials = format!("{}:{}", user, pass);
            let encoded = base64::encode(credentials.as_bytes());
            Some(HeaderValue::from_bytes(format!("Basic {}", encoded).as_bytes()))
        }
        _ => None,
    }
}
```

**触发条件**：代理配置的凭证在日志中泄露

**攻击路径**：
1. 用户配置代理时将凭证写入日志文件
2. 恶意程序读取日志文件，提取 Base64 编码的凭证
3. 使用提取的凭证访问代理

**修复建议**：
1. 在日志输出中脱敏代理凭证（显示为 `***`）
2. 在 API 文档中说明安全日志最佳实践
3. 考虑使用加密存储或密钥管理

#### 4.3 Host 头注入

**风险等级**: 🟡 中

**证据**: `ylong_http_client/src/util/proxy.rs` - 代理请求中的 Host 处理

```rust
// 代理连接时覆盖 Host 头
headers.insert("Host", proxy_authority)?;
```

**触发条件**：用户控制的 Host 值直接用于 HTTP 头

**攻击路径**：
1. 攻击者通过 `Proxy` 设置 Host 头为恶意域名
2. 后续请求使用该 Host 头，导致请求发送到恶意服务器
3. 可能绕过 CSP 或同源策略

**修复建议**：
1. 验证代理 Host 头的合法性
2. 在文档中明确代理模式下 Host 头的覆盖行为
3. 考虑添加 Host 头白名单机制

#### 4.4 不安全的 FFI 调用

**风险等级**: 🟠 低中

**证据**: `ylong_http_client/src/util/c_openssl/ffi/ssl.rs`

```rust
pub unsafe fn SSL_get0_param(ssl: *mut SSL) -> *mut X509_VERIFY_PARAM {
    unsafe { SSL_get0_param(ssl as *mut _) }
}
```

**潜在问题**：
- FFI 函数返回的指针生命周期管理不当可能导致 UAF
- `*mut _` 类型转换绕过 Rust 的借用检查
- 错误的字符串转换可能导致缓冲区溢出

**缓解措施**：
1. 使用封装函数（如 `SslParamRef`）管理指针生命周期
2. 在 unsafe 块中添加详细的安全注释
3. 尽可能使用 `check_ptr` 等安全封装函数

**证据**:
- `ylong_http_client/src/util/c_openssl/foreign.rs` - 安全封装函数

#### 4.5 连接池耗尽攻击

**风险等级**: 🟡 中

**证据**: `ylong_http_client/src/async_impl/pool.rs` - 连接池实现

```rust
pub struct ConnPool<C, S> {
    pool: Vec<Connection<C, S>>,
    max_idle_connections: usize,
    // ...
}

async fn get(&self, _uri: &Uri) -> Result<Connection<C, S>, HttpClientError> {
    // 从池中获取连接，未找到时创建新连接
    Ok(Connection::new(...))
}
```

**触发条件**：攻击者快速创建大量短生命周期连接

**攻击路径**：
1. 攻击者发起大量 HTTP 请求，每个请求使用新连接
2. 连接池达到上限后继续创建连接（绕过池机制）
3. 大量半开连接消耗服务器资源

**缓解措施**：
1. 实施连接数限制和速率限制
2. 监控连接池大小和创建速率
3. 添加连接清理机制，定期关闭空闲连接

#### 4.6 HTTP 头解析整数溢出

**风险等级**: 🟠 低

**证据**: `ylong_http/src/headers.rs` - 头部解析

```rust
// 数值解析（可能溢出）
let value = u64::from_str(value_str).map_err(|_| HttpErrorKind::InvalidInput)?;
```

**触发条件**：超长的数值字符串

**攻击路径**：
1. 攻击者发送超长 `Content-Length` 头
2. 解析时整数溢出导致内存分配异常
3. 可能导致拒绝服务或缓冲区溢出

**缓解措施**：
1. 使用 `saturating_*` 方法防止溢出
2. 添加数值范围验证
3. 返回解析错误而非 panic

**证据**: `ylong_http/src/headers.rs` - `from_str` 等解析方法实现

### 5. 检查范围与局限性

#### 5.1 已检查的安全机制

✅ **TLS/SSL 证书验证**：
- 证书链验证
- 证书有效期检查
- 主机名验证（SNI）
- 公钥固定支持

✅ **连接池管理**：
- 最大连接数限制
- 空闲超时机制

✅ **错误处理**：
- 统一的 `HttpClientError` 和 `HttpError`
- 不使用 `unwrap()` 或 `expect()` 处理用户输入

✅ **内存安全**：
- 使用 Rust 的所有权和借用检查
- unsafe 代码仅在 FFI 层使用 OpenSSL

#### 5.2 未覆盖的安全检查

⚠️ **路径遍历**：
- 无专门验证 URL 中 `../`, `%2e` 等特殊字符的逻辑
- 依赖 `Uri` 的解析和规范化

⚠️ **输入大小限制**：
- 无统一的请求体大小限制
- 无 Header 大小限制
- 无超长 URI 长度检查

⚠️ **速率限制**：
- 无客户端级别的请求速率限制
- 无 DNS 查询频率限制

⚠️ **HTTP/3 安全**：
- HTTP/3 实现存在但未在 GN 中启用
- QUIC 安全特性（如 0-RTT 握手保护）未验证

⚠️ **资源限制**：
- 连接池大小可配置，默认值可能过大
- 无并发连接数的硬性限制

### 6. 安全最佳实践建议

1. **使用默认 TLS 配置**：
   - 不要设置 `accept_invalid_certs` 或 `danger_set_ca_list` 为 true
   - 使用系统的 CA 证书存储

2. **启用证书验证**：
   - 使用公钥固定（Pinning）增强安全性
   - 实现自定义 `CertVerifier` 进行额外验证

3. **配置连接池大小**：
   - 根据设备资源限制最大连接数
   - 设置合理的空闲超时

4. **日志安全**：
   - 避免记录敏感信息（密码、Token、Cookie）
   - 对代理凭证进行脱敏处理

5. **输入验证**：
   - 验证 Host、URL、Content-Length 的格式和范围
   - 限制单个请求和响应的大小

---

## 相关跳转

- **[目录结构与模块职责](02_Directory_Structure.md)** - unsafe 代码位置
- **[对外 API](04_External_API.md)** - TLS 配置和危险选项

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
