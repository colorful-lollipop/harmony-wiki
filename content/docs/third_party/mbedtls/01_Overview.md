# mbedtls 库概述

## 原始库简介

### 库信息

| 属性 | 值 |
|------|-----|
| **库名称** | Mbed TLS (原名 mbed TLS) |
| **版本** | v3.6.5 |
| **许可证** | Apache License V2.0 (双许可: Apache-2.0 OR GPL-2.0-or-later) |
| **上游地址** | https://www.trustedfirmware.org/projects/mbed-tls/ |
| **代码托管** | https://github.com/Mbed-TLS/mbedtls |
| **首次发布** | 2009 年 |
| **维护者** | Trusted Firmware Project |

### 主要功能

Mbed TLS 是一个用 C 语言实现的**轻量级加密库**，提供以下核心功能：

#### 1. 加密原语 (Cryptographic Primitives)

| 类别 | 功能 |
|------|------|
| **对称加密** | AES、DES/3DES、ARC4、CHACHA20、POLY1305、CAMELLIA、ARIA |
| **非对称加密** | RSA、ECC (ECDSA, ECDH)、SM2 |
| **哈希算法** | SHA-1、SHA-256、SHA-512、SHA-3、MD5、RIPEMD-160 |
| **消息认证** | HMAC、CMAC、GMAC |
| **密钥派生** | HKDF、PBKDF2、ECJPAKE |

#### 2. X.509 证书处理

- X.509 证书解析与验证
- CSR (证书签名请求) 生成
- 证书链验证
- CRL (证书吊销列表) 支持

#### 3. SSL/TLS/DTLS 协议

| 协议版本 | 支持情况 |
|----------|----------|
| **TLS 1.0** | 已废弃 (建议禁用) |
| **TLS 1.1** | 已废弃 (建议禁用) |
| **TLS 1.2** | 完全支持 |
| **TLS 1.3** | 完全支持 |
| **DTLS 1.0** | 历史版本 |
| **DTLS 1.2** | 完全支持 |

#### 4. PSA 加密 API

Arm 的 Platform Security Architecture (PSA) 加密 API 实现，提供：

- 统一的加密接口
- 密钥管理抽象
- 加密驱动支持
- 安全存储集成

### 设计特点

1. **可移植性**: 纯 C 实现，支持多种硬件架构和操作系统
2. **可配置性**: 通过配置文件选择性启用功能，减少代码体积
3. **代码体积小**: 适合嵌入式系统
4. **易于使用**: 清晰的 API 设计
5. **安全性**: 恒定时间实现，防止时序攻击

### 与其他库的对比

| 特性 | mbedtls | OpenSSL | wolfSSL |
|------|---------|---------|---------|
| **代码体积** | 小 | 大 | 中等 |
| **许可证** | Apache-2.0 | Apache-2.0 | GPL-3.0/商用 |
| **TLS 1.3** | 支持 | 支持 | 支持 |
| **PSA API** | 原生支持 | 不支持 | 不支持 |
| **适用场景** | 嵌入式/IoT | 通用服务器 | 嵌入式/服务器 |

## 在 OpenHarmony 中的定位

### 系统安全基础设施

mbedtls 在 OpenHarmony 系统中扮演**核心安全基础设施**的角色：

```
┌─────────────────────────────────────────────────────────┐
│              OpenHarmony 安全架构                       │
├─────────────────────────────────────────────────────────┤
│  应用层安全 (Ability 权限、IPC 权限)                    │
├─────────────────────────────────────────────────────────┤
│  框架层安全 (HUKS 密钥管理、证书管理)                   │
├─────────────────────────────────────────────────────────┤
│  基础库安全 (mbedtls 加密、SSL/TLS)         ◄── 核心   │
├─────────────────────────────────────────────────────────┤
│  系统安全 (SELinux、安全启动)                          │
└─────────────────────────────────────────────────────────┘
```

### 主要应用场景

#### 1. 网络通信安全

```c
// HTTPS 通信加密
mbedtls_ssl_context ssl;
mbedtls_ssl_config conf;
mbedtls_x509_crt cacert;

// TLS 握手、加密传输、解密接收
```

**使用模块**:
- dsoftbus (分布式软总线)
- curl (HTTP/HTTPS 客户端)
- 网络框架

#### 2. 证书验证

```c
// X.509 证书解析
mbedtls_x509_crt_parse();
mbedtls_x509_crt_verify();

// 证书链验证
mbedtls_x509_crt_verify_chain();
```

**使用模块**:
- appverify_lite (应用验证)
- device_attest_lite (设备认证)

#### 3. 数据加密

```c
// AES 加密
mbedtls_cipher_context_t ctx;
mbedtls_cipher_setup();
mbedtls_cipher_encrypt();

// 哈希计算
mbedtls_md_context_t md_ctx;
mbedtls_md_init();
mbedtls_md_setup();
```

**使用模块**:
- huks (密钥管理)
- sys_installer_lite (系统升级)
- hiviewdfx (日志安全)

### 与 OH 组件的集成关系

```
                    ┌──────────────────┐
                    │   OpenHarmony    │
                    │    应用框架      │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
    ┌─────────▼──────┐ ┌────▼─────┐ ┌───────▼──────┐
    │   dsoftbus     │ │  huks   │ │  appverify   │
    │ (分布式通信)   │ │(密钥管理)│ │  (应用验证)  │
    └───────┬────────┘ └────┬────┘ └──────┬───────┘
            │               │              │
            └───────────────┼──────────────┘
                            │
                   ┌────────▼─────────┐
                   │    mbedtls       │
                   │ (加密安全基础)   │
                   └──────────────────┘
```

### 版本与升级策略

| 项目 | 详情 |
|------|------|
| **当前版本** | v3.6.5 |
| **上游最新** | 请查阅 https://github.com/Mbed-TLS/mbedtls |
| **升级策略** |跟随上游 LTS 版本，OH 适配层独立维护 |
| **适配层版本**| 5.0 |

### 组件配置 (bundle.json)

```json
{
  "name": "@ohos/mbedtls",
  "version": "5.0",
  "subsystem": "thirdparty",
  "features": [
    "mbedtls_porting_path",
    "mbedtls_enable_ssl_srv"
  ],
  "adapted_system_type": [
    "mini",
    "small",
    "standard"
  ],
  "deps": {
    "components": ["bounds_checking_function"]
  }
}
```
