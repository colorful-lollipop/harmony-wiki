# 安全风险分析

> **重要**: 本文档分析 rust-openssl 在 OpenHarmony 中的安全相关考虑。

## 概述

rust-openssl 本身是一个 **安全的 FFI 绑定层**，其安全性主要取决于：

1. **上游 rust-openssl 代码质量**
2. **底层 OpenSSL C 库的安全性**
3. **OH 构建配置的影响**

## 上游安全状态

### CVE 跟踪

| CVE 编号 | 影响版本 | 状态 | 说明 |
|---------|---------|------|------|
| N/A | rust-openssl | ✅ 无已知 CVE | rust-openssl 是绑定层，无独立漏洞 |

**注意**: rust-openssl 本身没有 CVE，因为它是 FFI 绑定。漏洞存在于 OpenSSL C 库。

### OpenSSL CVE 状态

rust-openssl 链接的 OpenSSL C 库可能存在 CVE：

| CVE 状态 | OH 系统 | 说明 |
|---------|--------|------|
| **已修复** | ✅ | OH OpenSSL 版本会同步上游安全修复 |
| **待修复** | ⚠️ | 需检查 OH OpenSSL 版本 |

## OH 构建配置安全影响

### 禁用的算法（安全收益）

rustflags 配置禁用了以下不安全的算法：

| 禁用算法 | 风险等级 | 禁用原因 |
|---------|---------|---------|
| **SSLv3** | 🔴 高 | POODLE 攻击 (CVE-2014-3566) |
| **RC4** | 🔴 高 | 多方面攻击 (CVE-2013-4366) |
| **MD5** | 🟠 中 | 碰撞攻击 (CVE-2004-2761) |
| **3DES** | 🟠 中 | 暴力攻击可行 |
| **Blowfish** | 🟡 低 | 密钥调度问题 |

```gn
# 安全相关的 rustflags 配置
rustflags = [
  "--cfg=osslconf=\"OPENSSL_NO_SSL3_METHOD\"",  # 禁用 SSLv3
  "--cfg=osslconf=\"OPENSSL_NO_RMD160\"",       # 禁用 RMD160
  # ... 其他禁用
]
```

### 启用的安全特性

| 特性 | 启用标志 | 安全性 |
|-----|---------|-------|
| **TLS 1.3** | `--cfg=ossl111` | ✅ 前向保密 |
| **AES-GCM** | 默认 | ✅ 认证加密 |
| **ChaCha20-Poly1305** | 默认 | ✅ 现代算法 |

## 安全使用建议

### 1. TLS 配置最佳实践

```rust
use openssl::ssl::{SslConnector, SslMethod, SslVerifyMode};

fn create_secure_connector() -> Result<SslConnector, openssl::error::ErrorStack> {
    let mut builder = SslConnector::builder(SslMethod::tls())?;
    
    // 1. 设置最低 TLS 版本（TLS 1.2+）
    builder.set_min_proto_version(Some(openssl::ssl::SslVersion::TLS1_2))?;
    
    // 2. 启用证书验证
    builder.set_verify(SslVerifyMode::PEER)?;
    
    // 3. 加载系统信任链
    // builder.set_ca_file("/etc/ssl/certs/ca-certificates.crt")?;
    
    // 4. 禁用压缩（CRIME 攻击）
    // builder.set_options(openssl::ssl::SslOptions::NO_COMPRESSION)?;
    
    Ok(builder.build())
}
```

### 2. 密码套件配置

```rust
use openssl::ssl::{SslConnector, SslMethod};

fn create_fips_connector() -> Result<SslConnector, openssl::error::ErrorStack> {
    let builder = SslConnector::builder(SslMethod::tls())?;
    
    // 推荐的安全套件（示例）
    // 注意：实际配置应遵循最新的安全建议
    
    // 启用前向保密的套件
    // - ECDHE-RSA-AES256-GCM-SHA384
    // - ECDHE-RSA-AES128-GCM-SHA256
    // - ECDHE-RSA-CHACHA20-POLY1305
    
    Ok(builder.build())
}
```

### 3. 证书验证

```rust
use openssl::x509::{X509, X509Store, X509StoreContext};
use openssl::error::ErrorStack;

fn verify_certificate_chain(
    leaf: &X509,
    intermediates: &[X509],
    root: &X509
) -> Result<bool, ErrorStack> {
    // 1. 构建证书存储
    let mut store = X509Store::new()?;
    store.add_cert(root.clone())?;
    
    // 2. 添加中间证书
    for intermediate in intermediates {
        store.add_cert(intermediate.clone())?;
    }
    
    // 3. 验证证书链
    let mut ctx = X509StoreContext::new()?;
    ctx.init(&store, leaf, |ctx| {
        ctx.verify_cert()
    })
}
```

## 已知安全限制

### 1. OpenSSL 版本依赖

rust-openssl 依赖于系统 OpenSSL，如果系统 OpenSSL 版本存在漏洞，会受到影响。

**建议**: 保持 OH 系统 OpenSSL 版本与上游同步。

### 2. FFI 安全边界

```rust
// 安全的 FFI 使用模式
use openssl_sys;

// 安全：使用高层 API
let result = openssl::hash::MessageDigest::sha256();

// 注意：避免直接使用 openssl_sys 中的底层函数
// 除非必要，否则使用 openssl crate 的安全封装
```

### 3. 错误处理安全

```rust
use openssl::error::ErrorStack;

// 不要泄露敏感信息
fn handle_error(error: ErrorStack) {
    // ✅ 正确：记录通用错误
    log::error!("TLS connection failed");
    
    // ❌ 错误：不要打印详细错误（可能包含敏感信息）
    // println!("{:?}", error);
}
```

## 安全审计清单

### 定期检查项目

- [ ] **OpenSSL 版本**: 确保 OH 系统 OpenSSL 是最新稳定版
- [ ] **TLS 配置**: 审查 TLS 版本和密码套件配置
- [ ] **证书验证**: 检查证书验证逻辑
- [ ] **日志泄露**: 确保不泄露敏感信息
- [ ] **密钥管理**: 审计密钥存储和传输

### 升级注意事项

```bash
# 升级 rust-openssl 时的安全检查
# 1. 检查上游版本的安全公告
# 2. 验证新版本的 CVE 修复
# 3. 测试 TLS 兼容性
# 4. 更新文档中的安全建议
```

## 相关资源

### 安全标准参考

- **[Mozilla TLS 指南](https://wiki.mozilla.org/Security/Server_Side_TLS)** - TLS 配置最佳实践
- **[OpenSSL 安全公告](https://www.openssl.org/news/vulnerabilities.html)** - CVE 跟踪
- **[CWE](https://cwe.mitre.org/)** - 常见弱点枚举

### 工具推荐

| 工具 | 用途 |
|-----|------|
| **testssl.sh** | TLS 配置审计 |
| **sslscan** | SSL/TLS 漏洞扫描 |
| **OWASP** | Web 应用安全 |

## 联系与报告

### 安全问题报告

如果发现 rust-openssl 相关的安全问题：

1. **上游问题**: 报告到 [rust-openssl GitHub](https://github.com/sfackler/rust-openssl/issues)
2. **OH 适配问题**: 联系维护者 (xuelei3@huawei.com)
3. **OpenSSL 问题**: 报告到 [OpenSSL](https://www.openssl.org/news/vulnerabilities.html)

---

**最后审查**: 2024年  
**维护者**: xuelei3@huawei.com  
**安全评级**: 🟢 低风险（主要风险来自 OpenSSL C 库）
