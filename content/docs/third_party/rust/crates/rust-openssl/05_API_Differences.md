# API/接口差异

> **说明**: rust-openssl 本身没有 OH 特有的 API 差异，所有 API 与上游保持一致。本文档说明使用时的注意事项。

## API 一致性声明

rust-openssl 的 API **与上游版本完全一致**，没有添加、修改或删除任何 API。

### 无差异证明

| 检查项 | 状态 | 说明 |
|-------|------|------|
| **API 签名** | ✅ 一致 | 函数签名与上游相同 |
| **返回类型** | ✅ 一致 | 无任何类型修改 |
| **错误处理** | ✅ 一致 | 错误类型和传播机制相同 |
| **特性标志** | ✅ 一致 | Cargo features 完全相同 |

## 使用注意事项

### 1. OpenSSL 版本兼容性

虽然 API 一致，但功能可用性取决于系统的 OpenSSL 版本。

```rust
// 示例：使用 OpenSSL 3.0 新特性
// 在支持 OpenSSL 3.0 的系统上可用

use openssl::pkey::PKey;
use openssl::pkey_ctx::PkeyCtxRef;

// Provider 支持 (OpenSSL 3.0+)
fn use_provider() -> Result<(), openssl::error::ErrorStack> {
    let ctx = PkeyCtxRef::new()?;
    // 使用 Provider API...
    Ok(())
}
```

### 2. 特性检测

rust-openssl 使用条件编译检测可用特性：

```rust
// 自动检测 OpenSSL 版本
#[cfg(ossl111)]
fn tls_1_3_features() {
    // OpenSSL 1.1.1+ 特性
}

#[cfg(ossl300)]
fn openssl_3_features() {
    // OpenSSL 3.0+ 特性
}
```

### 3. 可用算法

受 rustflags 配置影响，以下算法不可用：

```rust
// 以下代码将编译失败，因为这些算法被禁用
#[cfg(not(osslconf = "OPENSSL_NO_BF"))]
fn use_blowfish() {
    // Blowfish 不可用
}

// 可用的现代算法
fn available_algorithms() {
    // AES - 可用 ✅
    // ChaCha20 - 可用 ✅
    // RSA - 可用 ✅
    // EC (P-256, P-384) - 可用 ✅
    // SHA-256 - 可用 ✅
}
```

## 常见使用模式

### TLS 配置

```rust
use openssl::ssl::{SslConnector, SslMethod, SslAcceptor, SslFiletype};
use openssl::pkey::PKey;
use openssl::x509::X509;

// TLS 客户端配置
fn create_client_connector() -> Result<SslConnector, openssl::error::ErrorStack> {
    let mut builder = SslConnector::builder(SslMethod::tls())?;
    
    // 设置最低 TLS 版本
    builder.set_min_proto_version(Some(openssl::ssl::SslVersion::TLS1_2))?;
    
    // 启用证书验证
    builder.set_verify(openssl::ssl::SslVerifyMode::PEER)?;
    
    Ok(builder.build())
}

// TLS 服务端配置
fn create_server_acceptor(
    cert_path: &str,
    key_path: &str
) -> Result<SslAcceptor, openssl::error::ErrorStack> {
    let mut acceptor = SslAcceptor::builder(SslMethod::tls())?;
    
    // 加载证书和私钥
    acceptor.set_certificate_file(cert_path, SslFiletype::PEM)?;
    acceptor.set_private_key_file(key_path, SslFiletype::PEM)?;
    
    // 配置会话
    acceptor.set_session_id_context("ohos_hdc")?;
    
    Ok(acceptor.build())
}
```

### 证书处理

```rust
use openssl::x509::{X509, X509Name, X509Store, X509StoreContext};
use openssl::pkey::PKey;
use openssl::hash::MessageDigest;
use openssl::nid::Nid;
use std::time::{SystemTime, Duration};

// 证书创建
fn generate_self_signed_cert() -> Result<(X509, PKey), openssl::error::ErrorStack> {
    let rsa = PKey::from_rsa(openssl::rsa::Rsa::generate(2048)?)?;
    
    let mut name = X509Name::builder()?;
    name.append_entry_by_nid(Nid::COMMON_NAME, "localhost")?;
    let name = name.build();
    
    let mut cert = X509::builder()?;
    cert.set_subject_name(&name)?;
    cert.set_issuer_name(&name)?;
    cert.set_pubkey(&rsa)?;
    
    // 设置有效期
    cert.set_not_before(&SystemTime::UNIX_EPOCH)?;
    cert.set_not_after(&SystemTime::UNIX_EPOCH + Duration::from_secs(365 * 24 * 60 * 60))?;
    
    cert.sign(&rsa, MessageDigest::sha256())?;
    
    Ok((cert.build(), rsa))
}

// 证书验证
fn verify_certificate(
    cert: &X509,
    ca_store: &X509Store
) -> Result<bool, openssl::error::ErrorStack> {
    let mut ctx = X509StoreContext::new()?;
    ctx.init(ca_store, cert, |ctx| {
        ctx.verify_cert()
    })
}
```

### 加密操作

```rust
use openssl::symm::{Cipher, Crypter, Mode};
use openssl::hash::MessageDigest;
use openssl::hmac::{Hmac, HmacEngine};
use openssl::pkey_ctx::PkeyCtxRef;

// 对称加密 - AES-256-GCM
fn encrypt_aes_256_gcm(
    key: &[u8],
    iv: &[u8],
    plaintext: &[u8]
) -> Result<Vec<u8>, openssl::error::ErrorStack> {
    let cipher = Cipher::aes_256_gcm();
    let mut crypter = Crypter::new(Mode::Encrypt, cipher, key, Some(iv))?;
    
    // 输入可能需要填充
    let mut output = vec![0u8; plaintext.len() + cipher.block_size()];
    let count = crypter.update(plaintext, &mut output)?;
    
    // 获取认证标签
    let mut tag = vec![0u8; cipher.tag_len()];
    crypter.finalize(&mut output[count..], &mut tag)?;
    
    // 合并密文和标签
    output.truncate(count);
    output.extend_from_slice(&tag);
    
    Ok(output)
}

// HMAC 签名
fn hmac_sha256(key: &[u8], data: &[u8]) -> Result<Vec<u8>, openssl::error::ErrorStack> {
    let mut hmac = Hmac::new(MessageDigest::sha256(), key)?;
    hmac.update(data)?;
    hmac.finish()
}
```

## 限制说明

### 不支持的算法

由于 `rustflags` 配置，以下算法不可用：

| 算法 | 原因 | 建议替代 |
|-----|------|---------|
| **Blowfish** | 禁用 | ChaCha20 |
| **IDEA** | 禁用 | AES |
| **DES** | 禁用 | AES |
| **RC4** | 禁用 | AES, ChaCha20 |
| **SSLv3** | 禁用 | TLS 1.2+ |
| **MD5** | 不安全 | SHA-256 |

### 使用限制

```rust
// 以下操作将导致编译错误
#[cfg(osslconf = "OPENSSL_NO_BF")]
fn use_blowfish_cipher() {
    let cipher = Cipher::bf_cbc(); // 不存在！
}

// 正确做法：使用可用算法
fn use_available_cipher() {
    let cipher = Cipher::aes_256_cbc(); // ✅ 可用
}
```

## 迁移指南

### 从其他加密库迁移

如果从其他加密库迁移到 rust-openssl：

```rust
// 从 ring 迁移
// ring:
// use ring::{signature, digest};

// rust-openssl:
use openssl::pkey::PKey;
use openssl::signature::{Signature, KeyPair};
use openssl::hash::MessageDigest;

// RSA 签名
fn sign_rsa(private_key: &PKey, data: &[u8]) -> Result<Vec<u8>, openssl::error::ErrorStack> {
    let mut signature = Signature::sign_with_context(private_key, MessageDigest::sha256())?;
    signature.update(data)?;
    signature.sign_rsa_padded()
}
```

## 相关文档

- **[01_Overview.md](./01_Overview.md)** - 原始库功能介绍
- **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 使用场景
- **[上游 API 文档](https://docs.rs/openssl)** - 完整 API 参考
