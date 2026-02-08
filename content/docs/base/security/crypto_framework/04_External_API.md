# 对外 API (External API)

## 目的

本文档详细说明 crypto_framework 对外暴露的 API，包括 N-API 接口、Native Kits API、错误码定义等。

## 适用范围

- **目标读者**: 应用开发者、API 使用者、集成工程师
- **阅读时长**: 30 分钟

## N-API 接口

### 模块注册

**模块名称**: `security.cryptoFramework`

**注册入口**: `frameworks/js/napi/crypto/src/napi_init.cpp:243-254`

**引入方式**:
```javascript
import cryptoFramework from '@ohos.security.cryptoFramework';
```

### API 清单表

| JS 类/对象 | 说明 | 主要方法 | 文件位置 |
|------------|------|----------|----------|
| **CryptoMode** | 枚举：加密/解密模式 | ENCRYPT_MODE, DECRYPT_MODE | `napi_init.cpp` |
| **Result** | 枚举：错误码 | INVALID_PARAMS, NOT_SUPPORT, ... | `napi_init.cpp` |
| **AsyKeySpecItem** | 枚举：非对称密钥规格项 | RSA_N_BN, ECC_SK_BN, ... | `napi_init.cpp` |
| **AsyKeySpecType** | 枚举：非对称密钥规格类型 | COMMON_PARAMS_SPEC, PRIVATE_KEY_SPEC, ... | `napi_init.cpp` |
| **CipherSpecItem** | 枚举：加密规格项 | OAEP_MD_NAME_STR, GCM_TAG_LEN_NUM, ... | `napi_init.cpp` |
| **SignSpecItem** | 枚举：签名规格项 | PSS_MD_NAME_STR, SM2_USER_ID_UINT8ARR, ... | `napi_init.cpp` |
| **Random** | 随机数生成器 | generateRandom, generateRandomSync, setSeed, enableHardwareEntropy | `napi_rand.cpp` |
| **Md** | 消息摘要 | update, updateSync, digest, digestSync, getMdLength | `napi_md.cpp` |
| **Mac** | 消息认证码 | init, initSync, update, updateSync, doFinal, doFinalSync | `napi_mac.cpp` |
| **Sign** | 签名 | init, update, sign, initSync, updateSync, signSync, setSignSpec | `napi_sign.cpp` |
| **Verify** | 验签 | init, update, verify, recover, initSync, updateSync, verifySync, recoverSync | `napi_verify.cpp` |
| **Cipher** | 加密/解密 | init, update, doFinal, initSync, updateSync, doFinalSync | `napi_cipher.cpp` |
| **KeyAgreement** | 密钥协商 | generateSecret, generateSecretSync | `napi_key_agreement.cpp` |
| **AsyKeyGenerator** | 非对称密钥生成器 | generateKeyPair, generateKeyPairSync, convertKey, convertPemKey | `napi_asy_key_generator.cpp` |
| **AsyKeyGeneratorBySpec** | 基于规格的密钥生成器 | generateKeyPair, generatePriKey, generatePubKey | `napi_asy_key_spec_generator.cpp` |
| **SymKeyGenerator** | 对称密钥生成器 | generateSymKey, generateSymKeySync, convertKey | `napi_sym_key_generator.cpp` |
| **Kdf** | 密钥派生函数 | generateSecret, generateSecretSync | `napi_kdf.cpp` |
| **PriKey** | 私钥 | getEncoded, getEncodedDer, getEncodedPem, clearMem, getAsyKeySpec | `napi_pri_key.cpp` |
| **PubKey** | 公钥 | getEncoded, getEncodedDer, getEncodedPem, getAsyKeySpec | `napi_pub_key.cpp` |
| **SymKey** | 对称密钥 | getEncoded, clearMem | `napi_sym_key.cpp` |
| **HcfKey** | 密钥基类 | getEncoded | `napi_key.cpp` |
| **KeyPair** | 密钥对 | (基类) | `napi_key_pair.cpp` |
| **ECCKeyUtil** | ECC 密钥工具（静态） | genECCCommonParamsSpec, convertPoint, getEncodedPoint | `napi_ecc_key_util.cpp` |
| **DHKeyUtil** | DH 密钥工具（静态） | genDHCommonParamsSpec | `napi_dh_key_util.cpp` |
| **SM2CryptoUtil** | SM2 加密工具（静态） | genCipherTextBySpec, getCipherTextSpec | `napi_sm2_crypto_util.cpp` |
| **SignatureUtils** | 签名工具（静态） | genEccSignature, genEccSignatureSpec | `napi_sm2_ec_signature.cpp` |

### 工厂函数（模块级方法）

| JS 方法 | 对应 C++ 函数 | 说明 | 异步/同步 |
|----------|---------------|------|----------|
| `createRandom` | `NapiRand::CreateRand` | 创建随机数生成器 | 同步 |
| `createMd` | `NapiMd::CreateMd` | 创建消息摘要对象 | 同步 |
| `createMac` | `NapiMac::CreateMac` | 创建 MAC 对象 | 同步 |
| `createSign` | `NapiSign::CreateJsSign` | 创建签名对象 | 同步 |
| `createVerify` | `NapiVerify::CreateJsVerify` | 创建验签对象 | 同步 |
| `createCipher` | `NapiCipher::CreateCipher` | 创建加密对象 | 同步 |
| `createKeyAgreement` | `NapiKeyAgreement::CreateJsKeyAgreement` | 创建密钥协商对象 | 同步 |
| `createAsyKeyGenerator` | `NapiAsyKeyGenerator::CreateJsAsyKeyGenerator` | 创建非对称密钥生成器 | 同步 |
| `createAsyKeyGeneratorBySpec` | `NapiAsyKeyGeneratorBySpec::CreateJsAsyKeyGeneratorBySpec` | 基于规格创建密钥生成器 | 同步 |
| `createSymKeyGenerator` | `NapiSymKeyGenerator::CreateSymKeyGenerator` | 创建对称密钥生成器 | 同步 |
| `createKdf` | `NapiKdf::CreateJsKdf` | 创建密钥派生对象 | 同步 |

### 异步模式说明

所有耗时操作都支持异步模式，通过以下两种方式返回结果：

#### 1. Promise 模式

```javascript
// 示例：异步签名
let sign = cryptoFramework.createSign("RSA256");
await sign.init(priKey);
let signature = await sign.sign(data);
```

#### 2. Callback 模式

```javascript
// 示例：Callback 模式（如果支持）
let sign = cryptoFramework.createSign("RSA256");
sign.init(priKey, (err) => {
    if (err) {
        // 处理错误
        return;
    }
    sign.sign(data, (err, signature) => {
        if (err) {
            // 处理错误
            return;
        }
        // 使用签名
    });
});
```

### 同步模式说明

同步方法以 `Sync` 结尾，直接返回结果，可能阻塞线程：

```javascript
// 示例：同步签名
let sign = cryptoFramework.createSign("RSA256");
sign.initSync(priKey);
let signature = sign.signSync(data);
```

### 参数校验

crypto_framework 对所有输入参数进行严格校验：

| 校验类型 | 说明 | 校验位置 |
|----------|------|----------|
| **类型校验** | 检查参数类型（如 ArrayBuffer、DataBlob） | N-API 绑定层 |
| **范围校验** | 检查数值范围（如密钥长度、迭代次数） | 框架层 |
| **格式校验** | 检查数据格式（如 PEM 格式） | 框架层 |
| **空值校验** | 检查必需参数是否为 null/undefined | N-API 绑定层 |

**错误处理示例**:
```javascript
try {
    let sign = cryptoFramework.createSign("RSA256");
    await sign.init(null);  // 参数为空，抛出异常
} catch (error) {
    console.error("Error code:", error.code);  // 401
    console.error("Error message:", error.message);
}
```

## Native Kits API

### API 命名规范

- **函数前缀**: `OH_Crypto_`
- **类型前缀**: `Crypto_`
- **错误码前缀**: `CRYPTO_`

### API 清单表

| API 函数 | 功能 | 输入 | 输出 | 文件位置 |
|----------|------|------|------|----------|
| **OH_Crypto_FreeDataBlob** | 释放 DataBlob 内存 | `Crypto_DataBlob*` | `void` | `crypto_common.h:99` |
| **OH_Crypto_SymCipher_Create** | 创建对称加密对象 | 算法名称 | `Crypto_Cipher*` | `crypto_sym_cipher.h` |
| **OH_Crypto_SymCipher_Init** | 初始化加密操作 | Cipher, 模式, 密钥, 参数 | 错误码 | `crypto_sym_cipher.h` |
| **OH_Crypto_SymCipher_Update** | 更新数据 | Cipher, 输入, 输出 | 错误码 | `crypto_sym_cipher.h` |
| **OH_Crypto_SymCipher_DoFinal** | 完成加密 | Cipher, 输出 | 错误码 | `crypto_sym_cipher.h` |
| **OH_Crypto_AsymCipher_Create** | 创建非对称加密对象 | 算法名称 | `Crypto_Cipher*` | `crypto_asym_cipher.h` |
| **OH_Crypto_Digest_Create** | 创建摘要对象 | 算法名称 | `Crypto_Digest*` | `crypto_digest.h` |
| **OH_Crypto_Digest_Update** | 更新摘要 | Digest, 数据 | 错误码 | `crypto_digest.h` |
| **OH_Crypto_Digest_DoFinal** | 完成摘要 | Digest, 输出 | 错误码 | `crypto_digest.h` |
| **OH_Crypto_Sign_Create** | 创建签名对象 | 算法名称 | `Crypto_Sign*` | `crypto_signature.h` |
| **OH_Crypto_Sign_Init** | 初始化签名 | Sign, 私钥, 参数 | 错误码 | `crypto_signature.h` |
| **OH_Crypto_Sign_Update** | 更新签名数据 | Sign, 数据 | 错误码 | `crypto_signature.h` |
| **OH_Crypto_Sign_Sign** | 生成签名 | Sign, 输出 | 错误码 | `crypto_signature.h` |
| **OH_Crypto_Verify_Create** | 创建验签对象 | 算法名称 | `Crypto_Verify*` | `crypto_signature.h` |
| **OH_Crypto_Verify_Init** | 初始化验签 | Verify, 公钥, 参数 | 错误码 | `crypto_signature.h` |
| **OH_Crypto_Verify_Update** | 更新验签数据 | Verify, 数据 | 错误码 | `crypto_signature.h` |
| **OH_Crypto_Verify_Verify** | 验证签名 | Verify, 签名, 结果 | 错误码 | `crypto_signature.h` |
| **OH_Crypto_Mac_Create** | 创建 MAC 对象 | 算法名称 | `Crypto_Mac*` | `crypto_mac.h` |
| **OH_Crypto_Mac_Init** | 初始化 MAC | Mac, 密钥, 参数 | 错误码 | `crypto_mac.h` |
| **OH_Crypto_Mac_Update** | 更新 MAC | Mac, 数据 | 错误码 | `crypto_mac.h` |
| **OH_Crypto_Mac_DoFinal** | 完成 MAC | Mac, 输出 | 错误码 | `crypto_mac.h` |
| **OH_Crypto_Rand_Create** | 创建随机数生成器 | 无 | `Crypto_Rand*` | `crypto_rand.h` |
| **OH_Crypto_Rand_GenerateRandom** | 生成随机数 | Rand, 长度, 输出 | 错误码 | `crypto_rand.h` |
| **OH_Crypto_SymKey_Create** | 创建对称密钥 | 算法名称 | `Crypto_SymKey*` | `crypto_sym_key.h` |
| **OH_Crypto_AsymKey_Create** | 创建非对称密钥 | 算法名称 | `Crypto_AsymKey*` | `crypto_asym_key.h` |
| **OH_Crypto_Kdf_Create** | 创建 KDF 对象 | 算法名称 | `Crypto_Kdf*` | `crypto_kdf.h` |
| **OH_Crypto_Kdf_GenerateSecret** | 派生密钥 | Kdf, 密码, 盐值, 输出 | 错误码 | `crypto_kdf.h` |
| **OH_Crypto_KeyAgreement_Create** | 创建密钥协商对象 | 算法名称 | `Crypto_KeyAgreement*` | `crypto_key_agreement.h` |
| **OH_Crypto_KeyAgreement_GenerateSecret** | 生成共享密钥 | KA, 密钥对, 输出 | 错误码 | `crypto_key_agreement.h` |

### 数据类型

#### Crypto_DataBlob

```c
typedef struct Crypto_DataBlob {
    uint8_t *data;  // 数据缓冲区
    size_t len;      // 数据长度
} Crypto_DataBlob;
```

**位置**: `interfaces/kits/native/include/crypto_common.h:51-56`

#### Crypto_CipherMode

```c
typedef enum {
    CRYPTO_ENCRYPT_MODE = 0,  // 加密
    CRYPTO_DECRYPT_MODE = 1,  // 解密
} Crypto_CipherMode;
```

**位置**: `interfaces/kits/native/include/crypto_common.h:86-91`

## 错误码定义

### 内部错误码 (HcfResult)

| 错误码 | 值 | 说明 | 典型场景 |
|--------|-----|------|----------|
| `HCF_SUCCESS` | 0 | 成功 | 操作成功完成 |
| `HCF_INVALID_PARAMS` | -10001 | 参数无效 | 算法名称错误、参数为空 |
| `HCF_NOT_SUPPORT` | -10002 | 不支持 | 不支持的算法或操作 |
| `HCF_ERR_MALLOC` | -20001 | 内存分配失败 | 系统内存不足 |
| `HCF_ERR_NAPI` | -20002 | N-API 调用失败 | JS 引擎错误 |
| `HCF_ERR_ANI` | -20002 | ANI 调用失败 | ANI 引擎错误 |
| `HCF_ERR_PARAMETER_CHECK_FAILED` | -20003 | 参数校验失败 | 参数格式错误 |
| `HCF_ERR_CRYPTO_OPERATION` | -30001 | 加密操作错误 | 底层库操作失败 |

**位置**: `interfaces/inner_api/common/result.h:19-38`

### 对外错误码 (OH_Crypto_ErrCode)

| 错误码 | 值 | 说明 | 典型场景 |
|--------|-----|------|----------|
| `CRYPTO_SUCCESS` | 0 | 成功 | - |
| `CRYPTO_INVALID_PARAMS` | 401 | 参数无效 | 输入参数不合法 |
| `CRYPTO_NOT_SUPPORTED` | 801 | 不支持 | 不支持的算法 |
| `CRYPTO_MEMORY_ERROR` | 17620001 | 内存错误 | 内存分配失败 |
| `CRYPTO_PARAMETER_CHECK_FAILED` | 17620003 | 参数校验失败 | 参数格式错误 |
| `CRYPTO_OPERTION_ERROR` | 17630001 | 加密操作错误 | 底层操作失败 |

**位置**: `interfaces/kits/native/include/crypto_common.h:63-79`

### 错误码映射关系

```
底层库错误 (OpenSSL/MbedTLS)
    ↓
HCF_ERR_CRYPTO_OPERATION (-30001)
    ↓
CRYPTO_OPERTION_ERROR (17630001)
    ↓
JS Error 对象
```

## 权限说明

crypto_framework **不提供权限检查机制**。

### 权限管理位置

权限控制应在**应用框架层**或**系统服务层**实现：

```
应用调用 crypto_framework API
    ↑
    │ 应用层或系统服务
    │ 检查权限 ✓/✗
    │
    └── crypto_framework (无权限检查)
```

**证据**: 搜索 `AccessTokenKit`、`CheckPermission` 无结果（确认无权限检查代码）

## 相关跳转

- **架构设计**: [03_Architecture.md](03_Architecture.md)
- **内部 API**: [05_Internal_API.md](05_Internal_API.md)
- **目录结构**: [02_Directory_Structure.md](02_Directory_Structure.md)

## 更新记录

- **2026-02-06**: 创建文档，基于 N-API 和 Native API 分析生成

## TODO

- [ ] 补充每个 API 的详细参数说明
- [ ] 添加 API 使用示例代码
- [ ] 补充回调模式的使用说明
