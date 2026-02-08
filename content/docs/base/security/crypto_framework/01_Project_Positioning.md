# 项目定位、边界与核心能力

## 目的

本文档明确 crypto_framework 的项目定位、职责边界和核心能力，帮助开发者理解该组件在整个 OpenHarmony 安全子系统中的角色和适用场景。

## 适用范围

- **目标读者**: 架构师、安全开发者、集成工程师
- **前置知识**: 了解 OpenHarmony 系统架构，熟悉加密算法基础概念
- **阅读时长**: 10 分钟

## 项目定位

### 在 OpenHarmony 中的位置

crypto_framework 是 **安全子系统** 的核心组件之一，属于 **CryptoArchitectureKit** 系统能力。

```
OpenHarmony 系统架构
└── 安全子系统 (security)
    ├── 认证与授权 (permission)
    ├── 加密算法框架 (crypto_framework) ← 当前组件
    ├── 密钥存储服务 (huks)
    ├── 证书框架 (certificate_framework)
    └── 安全策略管理 (security_policy)
```

### 核心定位

crypto_framework 的核心定位是：**提供统一的密码学算法接口，屏蔽底层实现差异**。

这一定位体现在以下三个方面：

1. **算法统一性**: 无论使用 OpenSSL 还是 MbedTLS，应用层看到的是相同的 API
2. **系统适配性**: 支持标准系统（OpenSSL）和轻量系统（MbedTLS）
3. **语言无关性**: 通过 N-API、Native API、CJ FFI 支持多种开发语言

## 职责边界

### ✅ 属于 crypto_framework 的职责

| 职责 | 说明 | 证据 |
|------|------|------|
| **密码算法实现** | 对称/非对称加密、哈希、签名、MAC 等 | `plugin/openssl_plugin/`, `plugin/mbedtls_plugin/` |
| **密钥材料管理** | 密钥生成、导入导出、格式转换 | `frameworks/key/` |
| **SPI 接口定义** | 定义插件需要实现的接口规范 | `frameworks/spi/` |
| **API 暴露** | 提供 N-API、Native C、CJ FFI 接口 | `frameworks/js/`, `frameworks/native/` |
| **参数解析与校验** | 解析算法参数、验证输入合法性 | `common/params_parser.c`, `common/params_parser.h` |

### ❌ 不属于 crypto_framework 的职责

| 职责 | 说明 | 负责方 |
|------|------|----------|
| **密钥长期存储** | 持久化存储密钥 | **HuKS** (Hardware Key Store) |
| **硬件加密加速** | 使用 SE/TEE 执行加密操作 | **HuKS** + 硬件适配层 |
| **权限控制** | 验证调用者是否有权使用加密功能 | **应用框架** (Ability/Service 层) |
| **IPC 通信** | 跨进程调用加密服务 | **系统服务层** (SAMGR) |
| **证书验证** | X.509 证书链验证、CRL/OCSP 检查 | **Certificate Framework** |
| **SSL/TLS 协议** | 安全通信协议实现 | **SSL/TLS 框架** |
| **密钥协商协议** | TLS 1.2/1.3 密钥交换 | **SSL/TLS 框架** |
| **应用层加密** | 文件加密、数据库加密等业务场景 | **应用层开发者** |

### 责任边界示例

#### 场景：应用需要加密文件

```
应用层:
  - 决定使用什么加密算法
  - 管理加密密钥的生命周期
  - 决定密钥如何存储 (明文/加密)
  - 决定加密后的数据如何存储

crypto_framework:
  - 提供加密算法实现 (如 AES-GCM)
  - 生成和管理临时密钥材料
  - 执行加密操作
  - 返回加密结果

HuKS:
  - (可选) 提供安全密钥存储
  - (可选) 提供硬件加密加速
```

## 核心能力

### 1. 密码算法能力

#### 对称加密 (Symmetric Cipher)

**支持的算法**:
- AES: 128/192/256 位密钥
- SM4: 128 位密钥

**支持的模式**:
- ECB, CBC, CTR, OFB, CFB, CFB1, CFB8, CFB128
- 认证模式: GCM, CCM

**证据位置**:
- AES: `plugin/openssl_plugin/crypto_operation/cipher/src/cipher_aes_openssl.c`
- SM4: `plugin/openssl_plugin/crypto_operation/cipher/src/cipher_sm4_openssl.c`

#### 非对称加密 (Asymmetric Cipher)

**支持的算法**:
- RSA: OAEP, PKCS1-v1_5
- SM2: 国密非对称加密

**证据位置**:
- RSA: `plugin/openssl_plugin/crypto_operation/cipher/src/cipher_rsa_openssl.c`
- SM2: `plugin/openssl_plugin/crypto_operation/signature/src/sm2_sign_openssl.c`

#### 消息摘要 (Message Digest)

**支持的算法**:
- SHA: SHA-1, SHA-224, SHA-256, SHA-384, SHA-512
- SM3
- MD5

**证据位置**:
- `plugin/openssl_plugin/crypto_operation/md/src/md_openssl.c`

#### 签名验签 (Signature/Verify)

**支持的算法**:
- RSA: RSASSA-PKCS1-v1_5, RSASSA-PSS
- ECDSA: secp256r1, secp384r1, secp521r1
- DSA: 1024/2048/3072 位
- SM2
- Ed25519

**证据位置**:
- RSA/ECDSA/DSA: `plugin/openssl_plugin/crypto_operation/signature/src/sign_openssl.c`
- SM2: `plugin/openssl_plugin/crypto_operation/signature/src/sm2_sign_openssl.c`

#### 消息认证码 (MAC)

**支持的算法**:
- HMAC: HMAC-SHA1, HMAC-SHA256, HMAC-SHA384, HMAC-SHA512, HMAC-SM3
- CMAC: AES-CMAC, SM4-CMAC

**证据位置**:
- HMAC: `plugin/openssl_plugin/crypto_operation/hmac/src/hmac_openssl.c`
- CMAC: `plugin/openssl_plugin/crypto_operation/hmac/src/cmac_openssl.c`

#### 密钥派生 (KDF)

**支持的算法**:
- PBKDF2: 支持多种哈希函数
- HKDF: HMAC-based KDF
- X963KDF: ANSI X9.63 KDF
- Scrypt: 密码派生函数

**证据位置**:
- `plugin/openssl_plugin/crypto_operation/kdf/src/kdf_openssl.c`

#### 密钥协商 (Key Agreement)

**支持的算法**:
- ECDH: P-256, P-384, P-521
- DH: 2048/3072/4096 位
- X25519: Curve25519

**证据位置**:
- `plugin/openssl_plugin/crypto_operation/key_agreement/src/key_agreement_openssl.c`

#### 随机数生成 (Random)

**能力**:
- 生成指定长度的随机字节
- 支持设置种子
- 支持硬件熵源

**证据位置**:
- `plugin/openssl_plugin/crypto_operation/rand/src/rand_openssl.c`

### 2. 密钥管理能力

#### 对称密钥 (Symmetric Key)

**功能**:
- 生成随机对称密钥
- 从字节数组导入密钥
- 导出密钥为字节数组

**证据位置**:
- `interfaces/inner_api/key/sym_key.h`
- `frameworks/key/sym_key_generator.c`

#### 非对称密钥 (Asymmetric Key)

**支持的密钥类型**:
- RSA: 1024/2048/3072/4096 位
- ECC: secp256r1, secp384r1, secp521r1
- DSA: 1024/2048/3072 位
- Ed25519, X25519
- SM2

**功能**:
- 生成密钥对
- 导入/导出 PEM、DER 格式
- 提取公钥
- 获取密钥规格参数

**证据位置**:
- `interfaces/inner_api/key/asy_key_generator.h`
- `frameworks/key/asy_key_generator.c`

#### 密钥编码格式

支持的编码:
- **PEM**: Base64 编码，带有 `-----BEGIN/END-----` 头尾
- **DER**: 二进制 ASN.1 编码

**证据位置**:
- `interfaces/kits/native/include/crypto_asym_key.h` (行 40-60)

### 3. 多语言支持能力

| 语言 | 绑定方式 | 编译产物 | 文件位置 |
|------|----------|----------|----------|
| **JS/ArkTS** | N-API | libcryptoframework_napi.so | `frameworks/js/napi/crypto/` |
| **JS/ArkTS** | ANI (Advanced Native Interface) | crypto_framework_ani | `frameworks/js/ani/` |
| **JS (Lite)** | JSI (JavaScript Interface) | libcryptoframework_jsi.a | `frameworks/js/jsi/` |
| **Native C** | C API | libohcrypto.so | `frameworks/native/` |
| **Cangjie** | FFI (Foreign Function Interface) | libcj_cryptoframework_ffi.so | `frameworks/cj/` |

**证据位置**:
- N-API: `frameworks/js/napi/crypto/src/napi_init.cpp` (行 243-254)
- Native: `frameworks/native/BUILD.gn` (行 18-66)
- CJ FFI: `frameworks/cj/BUILD.gn` (行 18-92)

### 4. 插件化能力

#### SPI 接口体系

crypto_framework 定义了一套完整的 SPI 接口，允许扩展新的算法库：

| SPI 接口 | 功能 | 文件位置 |
|----------|------|----------|
| HcfMdSpi | 摘要算法 | `frameworks/spi/md_spi.h` |
| HcfSignSpi | 签名 | `frameworks/spi/signature_spi.h` |
| HcfMacSpi | MAC | `frameworks/spi/mac_spi.h` |
| HcfCipherSpi | 加解密 | `frameworks/spi/cipher_factory_spi.h` |
| HcfKeyAgreementSpi | 密钥协商 | `frameworks/spi/key_agreement_spi.h` |
| HcfKdfSpi | 密钥派生 | `frameworks/spi/kdf_spi.h` |
| HcfRandSpi | 随机数 | `frameworks/spi/rand_spi.h` |
| HcfAsyKeyGeneratorSpi | 非对称密钥生成 | `frameworks/spi/asy_key_generator_spi.h` |

**已实现的插件**:
- **OpenSSL Plugin**: 标准系统，功能完整
- **MbedTLS Plugin**: 轻量系统，支持基础功能

**证据位置**:
- OpenSSL: `plugin/openssl_plugin/`
- MbedTLS: `plugin/mbedtls_plugin/`

## 运行环境

### 系统类型

| 系统类型 | os_level 配置 | 默认插件 | JS 绑定 |
|----------|---------------|----------|----------|
| **standard** | `os_level == "standard"` | OpenSSL | N-API, ANI |
| **mini** | `os_level == "mini"` | MbedTLS | JSI |

**证据位置**: `BUILD.gn` (行 19-34)

### 内存限制

**ROM 占用**: 2MB (来自 bundle.json `rom: "2048KB"`)

### 编译特性

**Standard 系统**:
- 启用 CFI (Control Flow Integrity)
- 启用 PAC (Pointer Authentication)
- 启用 Sanitizer

**证据位置**: `frameworks/js/napi/crypto/BUILD.gn` (行 27-32)

## 关键概念

### 1. Hcf 前缀

框架内部类和接口使用 `Hcf` (Harmony Crypto Framework) 前缀：
- `HcfKey`: 密钥基类
- `HcfResult`: 结果码
- `HcfBlob`: 二进制数据块
- `HcfSpi`: SPI 接口

### 2. 对象生命周期

所有继承自 `HcfObjectBase` 的对象支持：
- **类标识**: `HcfGetClass()` 获取对象类型
- **引用计数**: 部分对象实现引用计数
- **销毁**: `HcfDestroy()` 释放资源

**证据位置**: `interfaces/inner_api/common/object_base.h`

### 3. 错误处理

两层错误码体系：
1. **内部错误码** (`HcfResult`): 框架内部使用
2. **对外错误码** (`OH_Crypto_ErrCode`): Native API 暴露给应用

**错误码映射**:
- 成功: `0`
- 参数错误: `-10001` (内部) / `401` (对外)
- 不支持: `-10002` (内部) / `801` (对外)

**证据位置**:
- 内部: `interfaces/inner_api/common/result.h`
- 对外: `interfaces/kits/native/include/crypto_common.h`

## 与其他组件的关系

### 与 HuKS 的关系

| 组件 | 职责 | 交互方式 |
|------|------|----------|
| **crypto_framework** | 软件加密算法实现 | 通过 HuKS API 调用 |
| **HuKS** | 密钥存储和硬件加密 | (如果需要) 提供密钥 |

**当前状态**: crypto_framework 默认使用软件实现，HuKS 作为可选依赖

**证据位置**: `bundle.json` (行 45 行 `"huks"`)

### 与 Permission 的关系

crypto_framework **不负责**权限检查。权限控制应在调用方实现：

```
应用调用 crypto_framework API
  ↑
  │ 应用层
  │ 检查权限 ✓/✗
  │
  └── crypto_framework (无权限检查)
```

**证据位置**: 搜索 `AccessTokenKit`、`CheckPermission` 无结果（确认无权限检查代码）

### 与 SSL/TLS 的关系

crypto_framework 提供**底层加密原语**，SSL/TLS 框架使用这些原语实现安全协议：

```
SSL/TLS 框架
  ├── 密钥协商 → 使用 crypto_framework 的 KeyAgreement
  ├── 对称加密 → 使用 crypto_framework 的 Cipher
  ├── 签名验签 → 使用 crypto_framework 的 Signature
  └── 摘要算法 → 使用 crypto_framework 的 Md
```

## 适用场景

### ✅ 适合使用 crypto_framework 的场景

| 场景 | 说明 |
|------|------|
| **应用数据加密** | 加密文件、数据库、配置信息 |
| **数字签名** | 应用签名、数据完整性验证 |
| **密钥交换** | TLS、自定义协议中的密钥协商 |
| **密码派生** | 从用户密码派生加密密钥 |
| **随机数生成** | 生成 IV、Nonce、会话密钥 |
| **国密算法** | 需要符合国家密码法规的场景 |

### ❌ 不适合使用 crypto_framework 的场景

| 场景 | 原因 |
|------|------|
| **HTTPS 通信** | 应使用 SSL/TLS 框架，而非直接调用 crypto_framework |
| **长期密钥存储** | 应使用 HuKS 或安全存储，而非自行管理 |
| **证书验证** | 应使用 Certificate Framework |
| **硬件加密加速** | 需要直接对接 TEE/SE，或通过 HuKS |

## 相关跳转

- **目录结构**: [02_Directory_Structure.md](02_Directory_Structure.md)
- **架构设计**: [03_Architecture.md](03_Architecture.md)
- **对外 API**: [04_External_API.md](04_External_API.md)
- **项目概览**: [00_Overview.md](00_Overview.md)

## 更新记录

- **2026-02-06**: 创建文档，基于代码分析生成
