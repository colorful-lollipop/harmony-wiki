# N-API 接口参考

## 1. 模块概述

### 模块名称
- **N-API 模块名**: `security.cert`
- **导入方式**: `import cert from '@ohos.security.cert'`

### 模块注册

**入口文件**: `frameworks/js/napi/certificate/src/napi_certificate_init.cpp:443-455`

```cpp
extern "C" __attribute__((constructor)) void RegisterCertModule(void)
{
    static napi_module cryptoFrameworkCertModule = {
        .nm_version = 1,
        .nm_flags = 0,
        .nm_filename = nullptr,
        .nm_register_func = CertModuleExport,
        .nm_modname = "security.cert",
        .nm_priv = nullptr,
        .reserved = { nullptr },
    };
    napi_module_register(&cryptoFrameworkCertModule);
}
```

## 2. API 清单

### 2.1 X509Cert 类

**创建方法**: `certificate.createX509Cert(encodingBlob)`

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `createX509Cert` |
| **C++ 实现** | `napi_certificate_init.cpp:1964` |
| **参数** | `EncodingBlob` - 证书编码数据 |
| **返回** | `X509Cert` 实例 (Promise) |
| **同步版本** | 无 |

**实例方法**:

| 方法名 | 类型 | 说明 | C++ 实现 |
|--------|------|------|----------|
| `verify` | 同步 | 验证证书签名 | napi_x509_certificate.cpp |
| `getEncoded` | 异步 | 获取编码数据 | napi_x509_certificate.cpp |
| `getPublicKey` | 同步 | 提取公钥 | napi_x509_certificate.cpp |
| `checkValidityWithDate` | 同步 | 检查有效期 | napi_x509_certificate.cpp |
| `getVersion` | 同步 | 获取版本号 | napi_x509_certificate.cpp |
| `getSerialNumber` | 同步 | 获取序列号 | napi_x509_certificate.cpp |
| `getCertSerialNumber` | 同步 | 获取证书序列号 | napi_x509_certificate.cpp |
| `getIssuerName` | 同步 | 获取颁发者名称 | napi_x509_certificate.cpp |
| `getSubjectName` | 同步 | 获取主题名称 | napi_x509_certificate.cpp |
| `getNotBeforeTime` | 同步 | 获取有效期起始 | napi_x509_certificate.cpp |
| `getNotAfterTime` | 同步 | 获取有效期结束 | napi_x509_certificate.cpp |
| `getSignature` | 同步 | 获取签名值 | napi_x509_certificate.cpp |
| `getSignatureAlgName` | 同步 | 获取签名算法名 | napi_x509_certificate.cpp |
| `getSignatureAlgOid` | 同步 | 获取签名算法 OID | napi_x509_certificate.cpp |
| `getSignatureAlgParams` | 同步 | 获取签名算法参数 | napi_x509_certificate.cpp |
| `getKeyUsage` | 同步 | 获取密钥用途 | napi_x509_certificate.cpp |
| `getExtKeyUsage` | 同步 | 获取扩展密钥用途 | napi_x509_certificate.cpp |
| `getBasicConstraints` | 同步 | 获取基本约束 | napi_x509_certificate.cpp |
| `getSubjectAltNames` | 同步 | 获取主题备用名称 | napi_x509_certificate.cpp |
| `getIssuerAltNames` | 同步 | 获取颁发者备用名称 | napi_x509_certificate.cpp |
| `getItem` | 同步 | 获取指定证书项 | napi_x509_certificate.cpp |
| `match` | 同步 | 匹配证书 | napi_x509_certificate.cpp |
| `toString` | 同步 | 转为字符串 | napi_x509_certificate.cpp |
| `hashCode` | 同步 | 获取哈希值 | napi_x509_certificate.cpp |
| `getExtensionsObject` | 同步 | 获取扩展对象 | napi_x509_certificate.cpp |
| `getIssuerX500DistinguishedName` | 同步 | 获取颁发者 X500 DN | napi_x509_certificate.cpp |
| `getSubjectX500DistinguishedName` | 同步 | 获取主题 X500 DN | napi_x509_certificate.cpp |
| `getCRLDistributionPoint` | 同步 | 获取 CRL 分布点 | napi_x509_certificate.cpp |

### 2.2 X509CertChain 类

**创建方法**:

| 方法名 | 类型 | 说明 | C++ 实现 |
|--------|------|------|----------|
| `createX509CertChain` | 异步 | 创建证书链 | napi_x509_cert_chain.cpp:1997 |
| `createTrustAnchorsWithKeyStore` | 异步 | 从密钥库创建信任锚 | napi_x509_cert_chain.cpp:1998 |
| `parsePkcs12` | 异步 | 解析 PKCS#12 | napi_x509_cert_chain.cpp:1999 |
| `createPkcs12Sync` | 同步 | 同步创建 PKCS#12 | napi_x509_cert_chain.cpp:2000 |
| `createPkcs12` | 异步 | 创建 PKCS#12 | napi_x509_cert_chain.cpp:2001 |
| `buildX509CertChain` | 异步 | 构建证书链 | napi_x509_cert_chain.cpp:2020 |

**实例方法**:

| 方法名 | 类型 | 说明 | C++ 实现 |
|--------|------|------|----------|
| `getCertList` | 同步 | 获取证书列表 | napi_x509_cert_chain.cpp |
| `validate` | 异步 | 校验证书链 | napi_x509_cert_chain.cpp |
| `toString` | 同步 | 转为字符串 | napi_x509_cert_chain.cpp |
| `hashCode` | 同步 | 获取哈希值 | napi_x509_cert_chain.cpp |

### 2.3 X509Crl / X509CRL 类

**创建方法**:

| 方法名 | 类型 | 说明 | C++ 实现 |
|--------|------|------|----------|
| `createX509Crl` | 异步 | 创建 CRL | napi_x509_crl.cpp:1582 |
| `createX509CRL` | 异步 | 创建 CRL (大写) | napi_x509_crl.cpp:1618 |

**X509Crl 实例方法**:

| 方法名 | 类型 | 说明 |
|--------|------|------|
| `isRevoked` | 同步 | 检查证书是否被吊销 |
| `getType` | 同步 | 获取 CRL 类型 |
| `getEncoded` | 异步 | 获取编码数据 |
| `verify` | 异步 | 验证签名 |
| `getVersion` | 同步 | 获取版本 |
| `getIssuerName` | 同步 | 获取颁发者名称 |
| `getLastUpdate` | 同步 | 获取最后更新 |
| `getNextUpdate` | 同步 | 获取下次更新 |
| `getSignature` | 同步 | 获取签名 |
| `getSignatureAlgName` | 同步 | 获取签名算法 |
| `getSignatureAlgOid` | 同步 | 获取签名算法 OID |
| `getSignatureAlgParams` | 同步 | 获取签名参数 |
| `getRevokedCert` | 同步 | 获取被吊销证书 |
| `getRevokedCerts` | 异步 | 获取所有被吊销证书 |
| `getRevokedCertWithCert` | 同步 | 用证书获取吊销信息 |
| `getTbsInfo` | 同步 | 获取 TBS 信息 |
| `toString` | 同步 | 转为字符串 |
| `hashCode` | 同步 | 获取哈希值 |
| `getExtensionsObject` | 同步 | 获取扩展对象 |
| `getIssuerX500DistinguishedName` | 同步 | 获取颁发者 X500 DN |

**X509CRL 额外方法**:

| 方法名 | 类型 | 说明 |
|--------|------|------|
| `getExtensions` | 同步 | 获取扩展 |
| `getTBSInfo` | 同步 | 获取 TBS 信息 |
| `match` | 同步 | 匹配 |

### 2.4 X500DistinguishedName 类

**创建方法**: `createX500DistinguishedName(encodingBlob)`

### 2.5 CertExtension 类

**创建方法**: `createCertExtension(encodingBlob)`

### 2.6 CertChainValidator 类

**创建方法**: `createCertChainValidator(algorithm)`

### 2.7 CertCRLCollection 类

**创建方法**: `createCertCRLCollection(encodingBlob)`

### 2.8 CmsGenerator 类

**创建方法**: `createCmsGenerator()`

**实例方法**:

| 方法名 | 类型 | 说明 |
|--------|------|------|
| `addSigner` | 同步 | 添加签名者 |
| `addCert` | 同步 | 添加证书 |
| `doFinal` | 异步 | 完成签名 |
| `doFinalSync` | 同步 | 同步完成签名 |
| `setRecipientEncryptionAlgorithm` | 同步 | 设置收件人加密算法 |
| `addRecipientInfo` | 同步 | 添加收件人信息 |
| `getEncryptedContentData` | 异步 | 获取加密内容数据 |

### 2.9 CmsParser 类

**创建方法**: `createCmsParser()`

**实例方法**:

| 方法名 | 类型 | 说明 |
|--------|------|------|
| `setRawData` | 同步 | 设置原始数据 |
| `getContentType` | 同步 | 获取内容类型 |
| `verifySignedData` | 异步 | 验证签名数据 |
| `getContentData` | 异步 | 获取内容数据 |
| `getCerts` | 异步 | 获取证书 |
| `decryptEnvelopedData` | 异步 | 解密信封数据 |

## 3. 常量定义

### 3.1 编码格式 (EncodingFormat)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `FORMAT_DER` | `CF_FORMAT_DER` | DER 编码 |
| `FORMAT_PEM` | `CF_FORMAT_PEM` | PEM 编码 |
| `FORMAT_PKCS7` | `CF_FORMAT_PKCS7` | PKCS#7 编码 |

### 3.2 错误码 (CertResult)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `INVALID_PARAMS` | `JS_ERR_CERT_INVALID_PARAMS` | 参数无效 |
| `NOT_SUPPORT` | `JS_ERR_CERT_NOT_SUPPORT` | 不支持 |
| `ERR_OUT_OF_MEMORY` | `JS_ERR_CERT_OUT_OF_MEMORY` | 内存不足 |
| `ERR_RUNTIME_ERROR` | `JS_ERR_CERT_RUNTIME_ERROR` | 运行时错误 |
| `ERR_PARAMETER_CHECK_FAILED` | `JS_ERR_CERT_PARAMETER_CHECK` | 参数检查失败 |
| `ERR_CRYPTO_OPERATION` | `JS_ERR_CERT_CRYPTO_OPERATION` | 加密操作错误 |
| `ERR_CERT_SIGNATURE_FAILURE` | `JS_ERR_CERT_SIGNATURE_FAILURE` | 签名验证失败 |
| `ERR_CERT_NOT_YET_VALID` | `JS_ERR_CERT_NOT_YET_VALID` | 证书未生效 |
| `ERR_CERT_HAS_EXPIRED` | `JS_ERR_CERT_HAS_EXPIRED` | 证书已过期 |
| `ERR_UNABLE_TO_GET_ISSUER_CERT_LOCALLY` | - | 无法获取颁发者证书 |
| `ERR_KEYUSAGE_NO_CERTSIGN` | - | 密钥用途无证书签名 |
| `ERR_KEYUSAGE_NO_DIGITAL_SIGNATURE` | - | 密钥用途无数字签名 |
| `ERR_MAYBE_WRONG_PASSWORD` | - | 可能是错误密码 |

### 3.3 证书项类型 (CertItemType)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `CERT_ITEM_TYPE_TBS` | `CF_ITEM_TBS` | TBS 证书 |
| `CERT_ITEM_TYPE_PUBLIC_KEY` | `CF_ITEM_PUBLIC_KEY` | 公钥 |
| `CERT_ITEM_TYPE_ISSUER_UNIQUE_ID` | `CF_ITEM_ISSUER_UNIQUE_ID` | 颁发者唯一标识 |
| `CERT_ITEM_TYPE_SUBJECT_UNIQUE_ID` | `CF_ITEM_SUBJECT_UNIQUE_ID` | 主题唯一标识 |
| `CERT_ITEM_TYPE_EXTENSIONS` | `CF_ITEM_EXTENSIONS` | 扩展 |

### 3.4 扩展 OID 类型 (ExtensionOidType)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `EXTENSION_OID_TYPE_ALL` | `CF_EXT_TYPE_ALL_OIDS` | 所有 OID |
| `EXTENSION_OID_TYPE_CRITICAL` | `CF_EXT_TYPE_CRITICAL_OIDS` | 关键 OID |
| `EXTENSION_OID_TYPE_UNCRITICAL` | `CF_EXT_TYPE_UNCRITICAL_OIDS` | 非关键 OID |

### 3.5 吊销检查选项 (RevocationCheckOptions)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `REVOCATION_CHECK_OPTION_PREFER_OCSP` | `CF_REVOCATION_CHECK_OPTION_PREFER_OCSP` | 优先 OCSP |
| `REVOCATION_CHECK_OPTION_ACCESS_NETWORK` | `CF_REVOCATION_CHECK_OPTION_ACCESS_NETWORK` | 允许访问网络 |
| `REVOCATION_CHECK_OPTION_FALLBACK_NO_PREFER` | `CF_REVOCATION_CHECK_OPTION_FALLBACK_NO_PREFER` | 回退非优先 |
| `REVOCATION_CHECK_OPTION_FALLBACK_LOCAL` | `CF_REVOCATION_CHECK_OPTION_FALLBACK_LOCAL` | 回退本地 |
| `REVOCATION_CHECK_OPTION_CHECK_INTERMEDIATE_CA_ONLINE` | 4 | 在线检查中间 CA |
| `REVOCATION_CHECK_OPTION_LOCAL_CRL_ONLY_CHECK_END_ENTITY_CERT` | 5 | 仅本地 CRL 检查终端实体 |
| `REVOCATION_CHECK_OPTION_IGNORE_NETWORK_ERROR` | 6 | 忽略网络错误 |

### 3.6 验证策略类型 (ValidationPolicyType)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `VALIDATION_POLICY_TYPE_X509` | `CF_VALIDATION_POLICY_TYPE_X509` | X.509 策略 |
| `VALIDATION_POLICY_TYPE_SSL` | `CF_VALIDATION_POLICY_TYPE_SSL` | SSL 策略 |

### 3.7 密钥用途类型 (KeyUsageType)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `KEYUSAGE_DIGITAL_SIGNATURE` | `CF_KEYUSAGE_DIGITAL_SIGNATURE` | 数字签名 |
| `KEYUSAGE_NON_REPUDIATION` | `CF_KEYUSAGE_NON_REPUDIATION` | 不可否认性 |
| `KEYUSAGE_KEY_ENCIPHERMENT` | `CF_KEYUSAGE_KEY_ENCIPHERMENT` | 密钥加密 |
| `KEYUSAGE_DATA_ENCIPHERMENT` | `CF_KEYUSAGE_DATA_ENCIPHERMENT` | 数据加密 |
| `KEYUSAGE_KEY_AGREEMENT` | `CF_KEYUSAGE_KEY_AGREEMENT` | 密钥协商 |
| `KEYUSAGE_KEY_CERT_SIGN` | `CF_KEYUSAGE_KEY_CERT_SIGN` | 证书签名 |
| `KEYUSAGE_CRL_SIGN` | `CF_KEYUSAGE_CRL_SIGN` | CRL 签名 |
| `KEYUSAGE_ENCIPHER_ONLY` | `CF_KEYUSAGE_ENCIPHER_ONLY` | 仅加密 |
| `KEYUSAGE_DECIPHER_ONLY` | `CF_KEYUSAGE_DECIPHER_ONLY` | 仅解密 |

### 3.8 通用名称类型 (GeneralNameType)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `GENERAL_NAME_TYPE_OTHER_NAME` | `CF_GENERAL_NAME_TYPE_OTHER_NAME` | 其他名称 |
| `GENERAL_NAME_TYPE_RFC822_NAME` | `CF_GENERAL_NAME_TYPE_RFC822_NAME` | 邮箱 |
| `GENERAL_NAME_TYPE_DNS_NAME` | `CF_GENERAL_NAME_TYPE_DNS_NAME` | DNS 名称 |
| `GENERAL_NAME_TYPE_X400_ADDRESS` | `CF_GENERAL_NAME_TYPE_X400_ADDRESS` | X400 地址 |
| `GENERAL_NAME_TYPE_DIRECTORY_NAME` | `CF_GENERAL_NAME_TYPE_DIRECTORY_NAME` | 目录名称 |
| `GENERAL_NAME_TYPE_EDI_PARTY_NAME` | `CF_GENERAL_NAME_TYPE_EDI_PARTY_NAME` | EDI 名称 |
| `GENERAL_NAME_TYPE_UNIFORM_RESOURCE_ID` | `CF_GENERAL_NAME_TYPE_UNIFORM_RESOURCE_ID` | URI |
| `GENERAL_NAME_TYPE_IP_ADDRESS` | `CF_GENERAL_NAME_TYPE_IP_ADDRESS` | IP 地址 |
| `GENERAL_NAME_TYPE_REGISTERED_ID` | `CF_GENERAL_NAME_TYPE_REGISTERED_ID` | 注册 ID |

## 4. 异步实现机制

### 4.1 Promise + Callback 双支持

**异步上下文结构** (`napi_common.h:28-38`):

```cpp
struct AsyncContext {
    AsyncType asyncType = ASYNC_TYPE_CALLBACK;  // 1: Callback, 2: Promise
    napi_value promise = nullptr;
    napi_ref callback = nullptr;
    napi_deferred deferred = nullptr;
    napi_async_work asyncWork = nullptr;
    napi_ref paramRef = nullptr;
    int32_t errCode = 0;
    const char *errMsg = nullptr;
};
```

### 4.2 异步工作创建流程

```cpp
// 创建 Promise
napi_create_promise(env, &context->deferred, &context->promise);

// 创建异步工作
napi_create_async_work(env, nullptr, resourceName,
    ExecuteCallback,      // 执行函数（工作线程）
    CompleteCallback,     // 完成回调（主线程）
    static_cast<void *>(context),
    &context->asyncWork);

// 队列执行
napi_queue_async_work(env, context->asyncWork);
```

## 5. 使用示例

### 5.1 创建和解析证书

```javascript
import certificate from '@ohos.security.cert';

let pemCert = `-----BEGIN CERTIFICATE-----
MIIB...
-----END CERTIFICATE-----`;

// 创建证书
let cert = await certificate.createX509Cert({
    encodingFormat: certificate.EncodingFormat.FORMAT_PEM,
    data: stringToUint8Array(pemCert)
});

// 获取证书信息
console.log('Version:', cert.getVersion());
console.log('Serial Number:', cert.getSerialNumber());
console.log('Issuer:', cert.getIssuerName());
console.log('Subject:', cert.getSubjectName());
```

### 5.2 验证证书有效期

```javascript
// 同步检查
try {
    cert.checkValidityWithDate('20250101000000Z');
    console.log('Certificate is valid');
} catch (e) {
    console.error('Certificate validation failed:', e);
}
```

### 5.3 证书链校验

```javascript
// 创建证书链
let chain = await certificate.createX509CertChain({
    encodingFormat: certificate.EncodingFormat.FORMAT_PEM,
    data: chainData
});

// 校验证书链
let result = await chain.validate({
    trustAnchors: anchors,
    revocationCheckOption: {
        checkOption: certificate.RevocationCheckOptions.REVOCATION_CHECK_OPTION_PREFER_OCSP
    }
});

if (result.isValid) {
    console.log('Chain is valid');
}
```

## 6. 错误处理

### 6.1 错误码处理

```javascript
try {
    let cert = await certificate.createX509Cert(data);
} catch (e) {
    if (e.code === certificate.CertResult.ERR_CERT_HAS_EXPIRED) {
        console.log('Certificate has expired');
    } else if (e.code === certificate.CertResult.ERR_CERT_NOT_YET_VALID) {
        console.log('Certificate not yet valid');
    } else {
        console.error('Other error:', e);
    }
}
```

### 6.2 参数校验

所有 API 在执行前都会进行参数校验：

- `null` / `undefined` 检查
- 数据类型检查
- 编码格式有效性检查
- 数据长度检查（最大 65536 字节）

## 7. 线程安全

- **同步方法**: 可从任意线程调用，内部会切换到正确的线程
- **异步方法**: 返回 Promise 或调用 Callback，确保在主线程执行回调

## 8. 相关文件

| 文件 | 说明 |
|------|------|
| `napi_certificate_init.cpp` | 模块入口和注册 |
| `napi_x509_certificate.cpp` | X509Cert 实现 |
| `napi_x509_cert_chain.cpp` | X509CertChain 实现 |
| `napi_x509_crl.cpp` | X509Crl 实现 |
| `napi_cert_cms_generator.cpp` | CMS 生成器/解析器实现 |
| `napi_common.cpp` | 异步上下文和回调处理 |
