# openHiTLS 库简介

## 1. 基本信息

### 1.1 原始库信息

| 属性 | 值 |
|-----|-----|
| **名称** | openHiTLS |
| **版本** | 0.2.1 |
| **许可证** | Mulan Permissive Software License v2 |
| **上游地址** | https://gitcode.com/openhitls/openhitls |
| **官方网站** | https://openhitls.net |
| **维护组织** | openHiTLS 社区 / 华为 |
| **维护者邮箱** | zhangxiaozan1@huawei.com |

### 1.2 OpenHarmony 集成信息

| 属性 | 值 |
|-----|-----|
| **OH 组件名** | @ohos/openhitls |
| **OH 版本** | 4.0 |
| **所属子系统** | thirdparty |
| **支持系统类型** | standard |
| **功能特性** | openhitls_enabled |
| **安装镜像** | system, updater |

---

## 2. 原始库功能简介

### 2.1 一句话描述

openHiTLS 是一个**高度模块化、高性能的开源密码学和传输层安全(TLS)开发套件**，旨在为全场景提供高效、敏捷的密码学 SDK。

### 2.2 核心功能

#### 协议支持
- **TLS 1.3** - 最新 TLS 协议，支持 Hybrid Key Exchange
- **TLS 1.2** / **DTLS 1.2** - 标准 TLS/DTLS 协议
- **TLCP** / **DTLCP** - 中国国密 TLS 协议
- **TLS-Provider** - 提供者架构
- **Auth** - 基于 RFC9578 的 PrivPass Token 认证

#### 算法支持

**国密算法**：
- SM2 (椭圆曲线公钥算法)
- SM3 (哈希算法)
- SM4 (分组加密算法)
- TLCP (国密 TLS 协议)

**国际标准算法**：
- 对称加密：AES, ChaCha20
- 公钥算法：RSA, ECDSA, ECDH, DH, DSA, X25519
- 哈希算法：SHA1, SHA2, SHA3, MD5, SM3
- MAC/HMAC：HMAC, CMAC, GMAC
- KDF：HKDF, PBKDF2, SCRYPT

**后量子算法 (PQC)**：
- ML-DSA (CRYSTALS-Dilithium)
- ML-KEM (CRYSTALS-Kyber)
- SLH-DSA (SPHINCS+)

#### PKI 功能
- X509 证书解析与生成
- CRL (证书吊销列表) 处理
- 证书链验证
- 证书请求 (CSR) 生成
- PKCS#12 证书存储

### 2.3 架构特点

```
┌─────────────────────────────────────────┐
│           Applications                  │
├─────────────────────────────────────────┤
│  Auth  │  TLS  │  PKI  │  Crypto  │ BSL │
├─────────────────────────────────────────┤
│         Platform Abstraction            │
│         (SAL - OS Adapter Layer)        │
└─────────────────────────────────────────┘
```

**5 大组件**：
1. **BSL** (Base Support Layer) - 基础支持层，OS 适配
2. **Crypto** - 密码算法，高性能实现
3. **TLS** - TLS/DTLS 协议栈
4. **PKI** - 证书和 PKI 功能
5. **Auth** - 认证功能

---

## 3. 在 OpenHarmony 中的作用和定位

### 3.1 核心定位

openHiTLS 在 OpenHarmony 中的核心定位是：**国密 TLS 解决方案提供者**

### 3.2 解决的问题

| 问题 | openHiTLS 解决方案 |
|-----|-------------------|
| 国密 HTTPS 需求 | 提供 SM2/SM3/SM4/TLCP 完整支持 |
| TLS 1.3 支持 | 完整的 TLS 1.3 协议实现 |
| 后量子安全 | 支持 ML-DSA/ML-KEM 等 PQC 算法 |
| 高性能密码运算 | ARMv8/x86_64 汇编优化 |

### 3.3 使用场景

```
┌────────────────────────────────────────────┐
│              应用场景                       │
├────────────────────────────────────────────┤
│  1. 国密 HTTPS 网站访问 (curl)              │
│     - 支持 TLCP/SM2/SM3/SM4                 │
│     - 兼容国密 SSL VPN                      │
├────────────────────────────────────────────┤
│  2. 应用层密码学操作                        │
│     - 证书解析与验证                        │
│     - 签名/验签                            │
│     - 加密/解密                            │
├────────────────────────────────────────────┤
│  3. 系统安全服务                            │
│     - 安全随机数生成                        │
│     - 密钥派生                             │
└────────────────────────────────────────────┘
```

### 3.4 与 OpenSSL 的关系

openHiTLS 在 OpenHarmony 中**不替代 OpenSSL**，而是**补充国密能力**：

- **OpenSSL**: 处理标准 TLS/HTTPS
- **openHiTLS**: 处理国密 TLS/HTTPS

在 curl 中，通过 `lib/vtls/openhitls.c` 适配层同时使用两者。

---

## 4. 目录结构

```
third_party/openhitls/
├── auth/                  # 认证组件 (RFC9578 PrivPass)
│   └── privpass_token/
├── bsl/                   # 基础支持层
│   ├── asn1/              # ASN.1 编解码
│   ├── base64/            # Base64 编解码
│   ├── buffer/            # 缓冲区管理
│   ├── err/               # 错误处理
│   ├── hash/              # 哈希表
│   ├── init/              # 初始化
│   ├── list/              # 链表
│   ├── log/               # 日志
│   ├── obj/               # 对象管理
│   ├── params/            # 参数处理
│   ├── pem/               # PEM 编解码
│   ├── sal/               # OS 适配层 (关键)
│   ├── tlv/               # TLV 编解码
│   ├── uio/               # UIO 抽象
│   └── usrdata/           # 用户数据
├── codecs/                # 编解码支持
├── config/                # 配置
│   ├── json/              # JSON 配置
│   ├── macro_config/      # 宏定义配置
│   └── toolchain/         # 工具链
├── crypto/                # 密码算法 (44+ 模块)
│   ├── aes/               # AES 算法
│   ├── bn/                # 大数运算
│   ├── chacha20/          # ChaCha20
│   ├── drbg/              # 确定性随机数
│   ├── eal/               # 算法抽象层
│   ├── ecc/               # 椭圆曲线
│   ├── ecdh/              # ECDH
│   ├── ecdsa/             # ECDSA
│   ├── entropy/           # 熵源
│   ├── hkdf/              # HKDF
│   ├── hmac/              # HMAC
│   ├── md5/               # MD5
│   ├── mldsa/             # ML-DSA (PQC)
│   ├── mlkem/             # ML-KEM (PQC)
│   ├── modes/             # 分组模式
│   ├── rsa/               # RSA
│   ├── sha1/              # SHA1
│   ├── sha2/              # SHA2
│   ├── sha3/              # SHA3
│   ├── slh_dsa/           # SLH-DSA (PQC)
│   ├── sm2/               # SM2 (国密)
│   ├── sm3/               # SM3 (国密)
│   └── sm4/               # SM4 (国密)
├── docs/                  # 文档
├── include/               # 公共头文件
│   ├── auth/              # Auth 头文件
│   ├── bsl/               # BSL 头文件
│   ├── crypto/            # Crypto 头文件
│   ├── pki/               # PKI 头文件
│   └── tls/               # TLS 头文件
├── pki/                   # PKI 组件
│   ├── cms/               # CMS/PKCS#7
│   ├── pkcs12/            # PKCS#12
│   ├── print/             # 证书打印
│   ├── x509_cert/         # X509 证书
│   ├── x509_common/       # X509 通用
│   ├── x509_crl/          # CRL
│   ├── x509_csr/          # CSR
│   └── x509_verify/       # 证书验证
├── platform/              # 平台相关
│   └── Secure_C/          # 安全 C 库依赖
├── script/                # 脚本
├── testcode/              # 测试代码
│   ├── benchmark/         # 性能测试
│   ├── demo/              # 示例程序
│   ├── framework/         # 测试框架
│   ├── sdv/               # SDV 测试
│   └── testdata/          # 测试数据
└── tls/                   # TLS 协议
    ├── alert/             # 告警处理
    ├── app/               # 应用层
    ├── ccs/               # 密码变更
    ├── cert/              # 证书管理
    ├── cm/                # 连接管理
    ├── config/            # TLS 配置
    ├── crypt/             # 加密适配
    ├── feature/           # 特性 (ALPN, SNI 等)
    ├── handshake/         # 握手协议
    └── record/            # 记录层
```

---

## 5. 版本历史

### 当前版本 (0.2.1)

- 首次集成 OpenHarmony
- 支持完整的 TLS 1.3 和国密 TLCP
- 支持 ML-DSA, ML-KEM, SLH-DSA 后量子算法
- ARMv8/x86_64 性能优化

### 上游发展

openHiTLS 由华为在 2024-2025 年开源，目前处于快速发展期：
- 2024: 项目开源
- 2025.02: 版本 0.2.1，集成 OpenHarmony

---

## 6. 与其他库的关系

### 6.1 依赖库

| 库 | 用途 | 说明 |
|---|------|------|
| bounds_checking_function | 安全函数 | Secure C 库，字符串/内存安全操作 |

### 6.2 被依赖

| 库 | 用途 | 说明 |
|---|------|------|
| curl | HTTPS 客户端 | 国密 HTTPS 支持 |

---

## 7. 总结

openHiTLS 是 OpenHarmony 的国密安全基石：

1. **国密支持**: 完整的 SM2/SM3/SM4/TLCP 实现
2. **标准兼容**: 同时支持 TLS 1.3/1.2 国际标准
3. **未来就绪**: 支持 ML-DSA/ML-KEM 等后量子算法
4. **高性能**: ARMv8/x86_64 汇编优化
5. **易集成**: 5 个模块化组件可按需使用
6. **无侵入**: 原生支持 OpenHarmony，无需 Patch
