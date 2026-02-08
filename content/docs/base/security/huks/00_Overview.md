# HUKS 项目概览

> 了解 HUKS 的定位、核心能力、运行环境和关键概念

**目的**: 快速理解 HUKS 是什么、能做什么、如何使用
**适用范围**: 所有人（新人、应用开发者、系统开发者）
**相关文档**: [目录结构](./01_Directory_Structure.md) | [架构说明](./02_Architecture.md) | [对外 API](./03_External_API.md)

---

## 1. 项目定位

### 1.1 什么是 HUKS

**HUKS** (Harmony Universal KeyStore / Universal Keystore Kit) 是 OpenHarmony 提供的**系统级密钥管理服务**，向应用提供统一的密钥管理和密码学操作能力。

**证据**:
- `README_zh.md:11` - "HUKS（OpenHarmony Universal KeyStore，OpenHarmony通用密钥库系统）"
- `bundle.json:3` - "The provider of key and certificate manangement capbility"

### 1.2 HUKS 的边界

**HUKS 负责**:
- 密钥的生成、导入、导出、删除
- 密钥的加密、解密、签名、验签
- 密钥的存储和访问控制
- 密钥证明和证书链验证

**HUKS 不负责**:
- 加密算法的底层实现（由 Crypto Framework 提供）
- 证书管理和颁发（由 Certificate Manager 提供）
- 证书的存储和检索

**相关仓库**:
- [security_crypto_framework](https://gitcode.com/openharmony/security_crypto_framework) - 加解密算法库
- [security_certificate_manager](https://gitcode.com/openharmony/security_certificate_manager) - 证书管理

### 1.3 与其他组件的关系

```
┌─────────────────────────────────────────────────────────┐
│  应用层 (JS/TS/C/C++)                                 │
│  - 应用调用 HUKS API                                  │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  HUKS (密钥管理)                                      │
│  - 密钥生成/存储/访问控制                              │
│  - 调用 Crypto Framework 进行加密操作                   │
└─────────────────────────────────────────────────────────┘
           ↓                              ↓
┌────────────────────┐    ┌──────────────────────────┐
│ Crypto Framework   │    │ Certificate Manager      │
│ (加密算法实现)      │    │ (证书管理)               │
└────────────────────┘    └──────────────────────────┘
```

---

## 2. 核心能力

### 2.1 密钥管理能力

| 能力 | 说明 | 证据 |
|-----|------|------|
| 密钥生成 | 支持多种算法（RSA/ECC/AES/HMAC 等） | `hks_api.h:HksGenerateKey()` |
| 密钥导入 | 支持明文导入和加密导入 | `hks_api.h:HksImportKey()` |
| 密钥导出 | 导出公钥和证书链 | `hks_api.h:HksExportPublicKey()` |
| 密钥删除 | 删除指定密钥 | `hks_api.h:HksDeleteKey()` |
| 密钥列表 | 列出所有密钥别名 | `hks_api.h:HksGetKeyInfoList()` |

### 2.2 密码学操作能力

| 能力 | 说明 | 证据 |
|-----|------|------|
| 加密/解密 | 对称加密和非对称加密 | `hks_api.h:HksEncrypt()`, `HksDecrypt()` |
| 签名/验签 | 多种签名算法支持 | `hks_api.h:HksSign()`, `HksVerify()` |
| MAC | 消息认证码 | `hks_api.h:HksMac()` |
| 密钥派生 | HKDF, PBKDF2 等派生算法 | `hks_api.h:HksDeriveKey()` |
| 密钥协商 | ECDH 等协商算法 | `hks_api.h:HksAgreeKey()` |

### 2.3 密钥证明能力

| 能力 | 说明 | 证据 |
|-----|------|------|
| 密钥证明 | 证明密钥由可信环境生成 | `hks_api.h:HksAttestKey()` |
| 证书链验证 | 验证证书链的完整性 | `hks_api.h:HksGetCertificateChain()` |

### 2.4 扩展能力

| 能力 | 说明 | 证据 |
|-----|------|------|
| UKey 支持 | 支持 USB Key 密钥操作 | `interfaces/kits/napi/src/huks_napi_ukey_module.cpp` |
| 密钥包装 | 密钥包装/解包装 | `hks_api.h:HksWrapKey()`, `HksUnwrapKey()` |

---

## 3. 运行环境

### 3.1 支持的系统类型

**证据**: `bundle.json:58-62`

| 系统类型 | 说明 | HUKS 实现 |
|---------|------|-----------|
| **Standard** (L2) | 标准系统 | 完整功能，使用 Binder IPC + SA 框架 |
| **Small** (L1) | 小型系统 | 轻量功能，使用 Samgr Lite 框架 |
| **Mini** (L0) | 轻量系统 | 最小功能，单二进制模式 |

### 3.2 安全环境要求

**标准系统**:
- **要求**: HUKS Core 层必须在安全环境（TEE 或具备安全能力的芯片）中运行
- **开源实现**: 由于安全环境需要特定硬件支持，开源代码中为模拟实现
- **证据**: `README_zh.md:19`

**小型/轻量系统**:
- **实现**: 仅提供根密钥保护方案的模拟实现
- **商用场景**: 必须根据产品能力适配硬件根密钥或使用其他根密钥保护方案
- **证据**: `README_zh.md:19`

### 3.3 依赖的系统组件

**证据**: `bundle.json:68-95`

| 组件 | 用途 |
|-----|------|
| access_token | Access Token 权限管理 |
| bundle_framework | Bundle 管理服务 |
| ipc | IPC 通信 |
| napi | Node-API 支持 |
| safwk | System Ability 框架 |
| samgr | Samgr Lite 框架 |
| user_auth_framework | 用户认证框架 |
| drivers_interface_huks | HUKS 驱动接口 |
| openssl / mbedtls | 加密库 |

---

## 4. 架构概览

### 4.1 三层架构

**证据**: `README_zh.md:13-19`

```
┌─────────────────────────────────────────────────────────┐
│  HUKS SDK 层                                          │
│  - 提供 HUKS API 供应用调用                          │
│  - N-API / C API / CJ API                            │
└─────────────────────────────────────────────────────────┘
                        ↓ IPC
┌─────────────────────────────────────────────────────────┐
│  HUKS Service 层                                      │
│  - 实现 HUKS 密钥管理、存储等功能                    │
│  - System Ability (SA ID: 3510)                      │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  HUKS Core 层                                         │
│  - HUKS 核心模块，负责密钥生成以及加解密等工作        │
│  - 运行在安全环境（TEE）                             │
└─────────────────────────────────────────────────────────┘
```

### 4.2 核心组件

**SDK 层** (`interfaces/`):
- `interfaces/kits/napi/` - JS/TS N-API
- `interfaces/kits/c/` - Native C API (NDK)
- `interfaces/kits/cj/` - Cangjie FFI

**Service 层** (`services/huks_standard/huks_service/`):
- `main/core/` - 密钥管理和存储
- `main/os_dependency/sa/` - System Ability 框架
- `main/systemapi_wrap/` - 系统 API 包装
- `extension/` - UKey 和其他扩展

**Core 层** (`services/huks_standard/huks_engine/`):
- `main/core/` - 加密引擎核心
- `frameworks/huks_standard/main/crypto_engine/` - 加密引擎实现（mbedtls/openssl）

---

## 5. 关键概念

### 5.1 密钥别名 (Key Alias)

密钥的字符串标识符，应用通过别名引用密钥。

**证据**: `hks_type.h` - `HksBlob keyAlias`

### 5.2 参数集 (ParamSet)

密钥属性和操作参数的集合，使用 Tag-Value 结构存储。

**证据**: `hks_param.h` - `struct HksParamSet`

### 5.3 三阶段操作 (Init/Update/Finish)

用于大数据量加密/解密/签名/验签的操作模式。

**证据**: `hks_api.h` - `HksInit()`, `HksUpdate()`, `HksFinish()`

### 5.4 安全级别 (Security Level)

| 安全级别 | 说明 | 证据 |
|---------|------|------|
| Software | 软件安全级别 | `huks_security_level = "software"` |
| Trusted Environment | 可信环境（TEE） | 未开源 |

### 5.5 存储级别 (Storage Level)

| 存储级别 | 说明 | 证据 |
|---------|------|------|
| DE (Device Encrypted) | 设备加密 | `HKS_AUTH_STORAGE_LEVEL_DE` |
| CE (Credential Encrypted) | 凭证加密 | `HKS_AUTH_STORAGE_LEVEL_CE` |
| ECE (Enhanced CE) | 增强凭证加密 | `HKS_AUTH_STORAGE_LEVEL_ECE` |

### 5.6 用户认证 (User Authentication)

密钥可以配置需要用户认证才能使用。

**证据**: `hks_tag.h` - `HKS_TAG_USER_AUTH_TYPE`

---

## 6. 系统能力

**证据**: `bundle.json:21-26`

```json
"syscap": [
    "SystemCapability.Security.Huks.Extension",
    "SystemCapability.Security.Huks.Core",
    "SystemCapability.Security.Cipher",
    "SystemCapability.Security.Huks.CryptoExtension = false"
]
```

---

## 7. 快速开始

### 7.1 应用使用 HUKS

```typescript
import huks from '@ohos.security.huks';

// 生成密钥
let keyAlias = 'myKey';
let properties = [
  { tag: huks.HuksTag.HUKS_TAG_ALGORITHM, value: huks.HuksKeyAlgorithm.HUKS_ALG_AES },
  { tag: huks.HuksTag.HUKS_TAG_KEY_SIZE, value: 256 },
  { tag: huks.HuksTag.HUKS_TAG_PURPOSE, value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_ENCRYPT | huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_DECRYPT },
  { tag: huks.HuksTag.HUKS_TAG_BLOCK_MODE, value: huks.HuksCipherMode.HUKS_MODE_CBC },
  { tag: huks.HuksTag.HUKS_TAG_PADDING, value: huks.HuksKeyPadding.HUKS_PADDING_PKCS7 }
];

huks.generateKeyItem(keyAlias, properties, (error, data) => {
  if (error) {
    console.error(`Generate key failed, code: ${error.code}, msg: ${error.message}`);
    return;
  }
  console.log('Generate key success');
});
```

### 7.2 C/C++ 使用 HUKS

```c
#include "hks_api.h"

// 生成密钥
struct HksBlob keyAlias = { .size = strlen("myKey"), .data = (uint8_t *)"myKey" };
struct HksParamSet *genParamSet = NULL;
HksInitParamSet(&genParamSet);

struct HksParam algParam = { .tag = HKS_TAG_ALGORITHM, .value = { .uint32Param = HKS_ALG_AES } };
HksAddParams(genParamSet, &algParam, 1);

int32_t ret = HksGenerateKey(&keyAlias, genParamSet, NULL);

HksFreeParamSet(&genParamSet);
```

---

## 8. 资源与链接

### 8.1 官方文档

- [HUKS 接口文档](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-universal-keystore-kit/Readme-CN.md)
- [HUKS 开发指导](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/UniversalKeystoreKit/Readme-CN.md)

### 8.2 源码位置

- [HUKS 仓库](https://gitcode.com/openharmony/security_huks)
- 本地路径: `/Volumes/lexar/code/d/work/oh/base/security/huks`

### 8.3 相关文档

- [目录结构](./01_Directory_Structure.md) - 代码组织详解
- [架构说明](./02_Architecture.md) - 架构设计和数据流
- [对外 API](./03_External_API.md) - API 接口详细说明

---

## 9. 版本信息

| 项目 | 版本 |
|-----|------|
| HUKS | 4.0.2 |
| OpenHarmony | N/A |
| 许可证 | Apache License 2.0 |

**证据**: `bundle.json:4-5`
