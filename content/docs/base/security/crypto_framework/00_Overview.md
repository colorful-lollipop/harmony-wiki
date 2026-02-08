# 项目概览

## 目的

本文档提供 crypto_framework（加解密算法库框架）的快速概览，帮助开发者在 15 分钟内理解项目的基本情况、核心能力和使用场景。

## 适用范围

本文档适用于：
- 新加入 OpenHarmony 安全子系统的开发者
- 需要集成 crypto_framework 的应用开发者
- 安全审计人员
- 对加密算法框架感兴趣的研究人员

## 项目简介

crypto_framework 是 OpenHarmony 系统中提供的加解密算法库框架，其核心目标是**屏蔽不同第三方密码学算法库的实现差异**，为上层应用提供统一的加密接口。

### 核心价值

| 价值点 | 说明 |
|--------|------|
| **接口统一** | 无论底层使用 OpenSSL 还是 MbedTLS，应用层使用相同的 API |
| **算法丰富** | 支持对称加密、非对称加密、签名验签、消息摘要、密钥派生等多种算法 |
| **多语言支持** | 支持 JS/ArkTS（通过 N-API/ANI）、Native C、Cangjie 语言调用 |
| **系统适配** | 同时支持标准系统（standard）和轻量系统（mini） |
| **插件化** | 通过 SPI 机制实现插件化架构，便于扩展新的算法库 |

## 核心能力

### 1. 密码操作

| 操作类型 | 说明 | 典型算法 |
|----------|------|----------|
| **对称加密** | 加密/解密数据 | AES (ECB/CBC/CTR/GCM/CCM), SM4 |
| **非对称加密** | 公钥加密/私钥解密 | RSA, SM2 |
| **消息摘要** | 计算数据哈希 | SHA-1/256/384/512, SM3, MD5 |
| **签名验签** | 数字签名和验证 | RSA-PSS/RSASSA-PKCS1-v1_5, ECDSA, SM2 |
| **消息认证码** | 数据完整性和真实性 | HMAC, CMAC |
| **密钥派生** | 从密码/密钥派生新密钥 | PBKDF2, HKDF, Scrypt |
| **密钥协商** | 建立共享密钥 | ECDH, DH |
| **随机数** | 生成安全随机数 | 支持硬件熵源 |

### 2. 密钥管理

- **对称密钥生成**: 生成 AES、SM4 等对称密钥
- **非对称密钥生成**: 生成 RSA、ECC、DSA、ED25519、X25519 等非对称密钥对
- **密钥导入导出**: 支持 PEM、DER 格式
- **密钥转换**: 在不同格式之间转换密钥

### 3. 国密算法支持

完全支持中国国家密码算法（SM 系列）：
- **SM2**: 非对称加密和签名
- **SM3**: 哈希算法
- **SM4**: 对称加密

### 4. 异步操作

所有耗时操作均支持异步调用模式：
- **Promise 模式**: 返回 Promise 对象
- **Callback 模式**: 传入回调函数
- **同步模式**: 直接返回结果

## 运行环境

### 系统支持

| 系统类型 | 底层插件 | JS 接口 | 适用场景 |
|----------|----------|----------|----------|
| **standard** | OpenSSL | N-API, ANI | 标准系统（如手机、平板） |
| **mini** | MbedTLS | JSI | 轻量系统（如 IoT 设备） |

### 依赖组件

**标准系统**:
- OpenSSL (`openssl:libcrypto_shared`)
- HuKS (`huks:libhukssdk`)
- N-API (`napi:ace_napi`)
- 日志库 (`hilog:libhilog`)

**轻量系统**:
- MbedTLS (`mbedtls`)
- 轻量级 N-API (`napi`)
- 轻量级日志 (`hilog_lite:hilog_lite`)

### 系统能力

crypto_framework 依赖以下系统能力（定义在 `bundle.json` 中）：

- `SystemCapability.Security.CryptoFramework`
- `SystemCapability.Security.CryptoFramework.Key`
- `SystemCapability.Security.CryptoFramework.Key.SymKey`
- `SystemCapability.Security.CryptoFramework.Key.AsymKey`
- `SystemCapability.Security.CryptoFramework.Signature`
- `SystemCapability.Security.CryptoFramework.Cipher`
- `SystemCapability.Security.CryptoFramework.KeyAgreement`
- `SystemCapability.Security.CryptoFramework.MessageDigest`
- `SystemCapability.Security.CryptoFramework.Mac`
- `SystemCapability.Security.CryptoFramework.Kdf`
- `SystemCapability.Security.CryptoFramework.Rand`

## 通用概念

### 分层架构

crypto_framework 采用三层架构设计：

```
┌─────────────────────────────────────────┐
│   API 层 (应用直接调用)            │
│  - N-API (JS/ArkTS)                │
│  - Native C API                     │
│  - CJ FFI (Cangjie)               │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   框架层 (统一接口实现)            │
│  - 密码操作封装                      │
│  - 密钥材料管理                      │
│  - SPI 接口定义                      │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   插件层 (第三方库适配)            │
│  - OpenSSL Plugin                   │
│  - MbedTLS Plugin                  │
└─────────────────────────────────────┘
```

### 对象模型

crypto_framework 使用基于对象的 API 设计：

- **HcfObjectBase**: 所有对象的基类，提供类标识和销毁机制
- **HcfBlob**: 二进制数据块，用于传递密钥数据、加密结果等
- **HcfResult**: 操作结果码，表示成功或失败原因
- **HcfKey**: 密钥基类，派生出的子类包括：
  - `HcfSymKey`: 对称密钥
  - `HcfPriKey`: 私钥
  - `HcfPubKey`: 公钥
  - `HcfKeyPair`: 密钥对

### SPI 机制

SPI (Service Provider Interface) 是 crypto_framework 的核心扩展机制：

- **接口定义**: `frameworks/spi/` 目录定义了所有 SPI 接口
- **插件实现**: `plugin/` 目录下实现了基于 OpenSSL 和 MbedTLS 的插件
- **动态加载**: 框架在运行时根据配置加载相应的插件

## 关键特性

### 1. 国密算法完整支持

代码证据: `plugin/openssl_plugin/` 下有完整的 SM2/SM3/SM4 实现

### 2. 多种编码格式支持

- **ASN.1/DER**: 二进制编码
- **PEM**: Base64 编码 + 头尾标记

### 3. 算法参数灵活配置

通过参数对象（`HcfAlgorithmParameter`）配置算法行为：
- RSA: OAEP 模式参数、PSS 盐长度
- ECC: 曲线参数、域参数
- AES: IV、GCM 认证标签长度
- KDF: 迭代次数、盐值

### 4. 内存安全管理

- **敏感数据清除**: 提供密钥清除接口（`clearMem`）
- **安全随机数**: 支持硬件熵源
- **内存检查**: 使用 `bounds_checking_function` 防止缓冲区溢出

## 不支持的功能

以下功能**不在** crypto_framework 的职责范围内：

| 功能 | 说明 | 负责方 |
|------|------|----------|
| **密钥持久化存储** | 密钥的长期存储 | HuKS (Hardware Key Store) |
| **权限控制** | 调用加密 API 的权限检查 | 应用框架层（Ability/Service） |
| **IPC 通信** | 跨进程调用 | 系统服务层 |
| **硬件加密** | 使用 SE/TEE 硬件加速 | HuKS 和硬件适配层 |
| **证书验证** | X.509 证书链验证 | Certificate Framework |
| **SSL/TLS** | 安全通信协议 | SSL/TLS 框架 |

## 使用场景示例

### 场景 1: 应用数据加密

```
应用层调用 crypto_framework
  ↓
创建 AES 对称密钥 (SymKeyGenerator)
  ↓
使用 Cipher 加密用户数据
  ↓
保存加密后的数据到存储
```

### 场景 2: 数字签名

```
应用层调用 crypto_framework
  ↓
生成 ECC 密钥对 (AsyKeyGenerator)
  ↓
使用 Sign 签名敏感数据
  ↓
将签名和公钥发送到验证方
```

### 场景 3: 密钥协商

```
客户端                          服务端
  │                              │
  ├─ 生成 ECDH 密钥对          ├─ 生成 ECDH 密钥对
  │                              │
  ├─ 发送公钥 ──────────────────> 交换公钥
  │                              │
  │                              ├─ 计算共享密钥
  ├─ 计算共享密钥              │
  │                              │
  └─ 使用共享密钥加密会话      └─ 使用共享密钥解密会话
```

## 相关跳转

- **详细架构**: [03_Architecture.md](03_Architecture.md)
- **API 参考**: [04_External_API.md](04_External_API.md)
- **目录结构**: [02_Directory_Structure.md](02_Directory_Structure.md)
- **安全分析**: [08_Security_Review.md](08_Security_Review.md)

## 更新记录

- **2026-02-06**: 创建文档，基于代码分析生成

## TODO

- [ ] 补充具体算法的性能对比
- [ ] 添加更多使用示例代码
- [ ] 补充与 HuKS 的集成说明
