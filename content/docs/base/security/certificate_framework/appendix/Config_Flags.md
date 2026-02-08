# 配置开关与宏定义

## 1. 构建配置

### 1.1 cf.gni 配置

**文件**: `cf.gni`

```gni
enable_coverage = false    # 是否启用代码覆盖率
```

### 1.2 根 BUILD.gn 配置

**文件**: `BUILD.gn`

```gn
declare_args() {
  certificate_framework_enabled = true  # 是否启用证书框架
}
```

## 2. 编译选项

### 2.1 Sanitizer 配置

**文件**: `frameworks/BUILD.gn`

```gn
sanitize = {
  cfi = true                    # 控制流完整性 (CFI)
  cfi_cross_dso = true          # 跨 DSO CFI 检查
  boundary_sanitize = true      # 边界检查
  debug = false                 # 调试模式
  integer_overflow = true       # 整数溢出检测
  ubsan = true                  # 未定义行为检测 (UBSan)
}
```

### 2.2 CFLAGS 配置

```gn
cflags = [
  "-DHILOG_ENABLE",    # 启用日志
  "-Wall",             # 启用所有警告
  "-Werror",           # 警告视为错误
]
```

### 2.3 LDFLAGS 配置

```gn
ldflags = [ "-Wl,--whole-archive" ]  # 全量链接
```

## 3. 功能宏定义

### 3.1 日志相关

| 宏 | 定义位置 | 说明 |
|---|---------|------|
| `HILOG_ENABLE` | frameworks/BUILD.gn | 启用 hilog 日志 |

### 3.2 API 导出

```c
// cf_type.h
#define CF_API_EXPORT __attribute__ ((visibility("default")))
```

### 3.3 能力注册

```c
// cf_ability.h
#define CF_ABILITY(abilityType, objType) (((abilityType) << 24) | (objType))
#define CF_MAGIC(type, obj) (((type) << 16) | (obj))
```

## 4. 运行时配置

### 4.1 常量定义

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_COUNT_OID` | 100 | 最大 OID 数量 |
| `MAX_LEN_OID` | 128 | 最大 OID 长度 |
| `MAX_COUNT_NID` | 1195 | 最大 NID 数量 |
| `MAX_LEN_CERTIFICATE` | 65536 | 最大证书长度 |
| `MAX_LEN_EXTENSIONS` | 65536 | 最大扩展长度 |

### 4.2 对象类型

```c
typedef enum {
    CF_OBJ_TYPE_CERT,        // 证书
    CF_OBJ_TYPE_EXTENSION,   // 扩展
    CF_OBJ_TYPE_CRL,         // CRL
    CF_OBJ_TYPE_LIST,        // 列表
} CfObjectType;
```

### 4.3 能力类型

```c
typedef enum {
    CF_ABILITY_TYPE_ADAPTER = 1,  // 适配器
    CF_ABILITY_TYPE_OBJECT = 2,   // 对象
} CfAbilityType;
```

### 4.4 标签类型

```c
typedef enum {
    CF_TAG_TYPE_INVALID = 0 << 28,
    CF_TAG_TYPE_INT = 1 << 28,
    CF_TAG_TYPE_UINT = 2 << 28,
    CF_TAG_TYPE_ULONG = 3 << 28,
    CF_TAG_TYPE_BOOL = 4 << 28,
    CF_TAG_TYPE_BYTES = 5 << 28,
} CfTagType;
```

### 4.5 编码格式

```c
typedef enum {
    CF_ENCODING_UTF8 = 0,
} CfEncodinigType;

typedef enum {
    PEM = 0,
    DER = 1,
} CfEncodinigBaseFormat;
```

## 5. 适配器版本

### 5.1 v1.0 架构 (SPI)

```c
// 文件: frameworks/core/v1.0/spi/*.h
struct HcfX509CertificateSpi {
    CfResult (*parseCert)(...);
    CfResult (*verifySignature)(...);
};
```

### 5.2 v2.0 架构 (Ability)

```c
// 文件: frameworks/core/cert/inc/cf_cert_adapter_ability_define.h
typedef struct CfCertAdapterAbilityFunc {
    CfAbilityBase base;
    CfResult (*adapterCreate)(const CfEncodingBlob *in, HcfX509CertificateSpi **spi);
    void (*adapterDestory)(HcfX509CertificateSpi **spi);
    CfResult (*adapterVerify)(HcfX509CertificateSpi *spi, void *key);
    CfResult (*adapterGetItem)(HcfX509CertificateSpi *spi, CfItemId itemId, CfBlob *out);
} CfCertAdapterAbilityFunc;
```

## 6. 异步类型

```c
typedef enum {
    ASYNC_TYPE_CALLBACK = 1,  // 回调模式
    ASYNC_TYPE_PROMISE = 2,   // Promise 模式
} AsyncType;
```

## 7. 验证策略

### 7.1 验证策略类型

```c
typedef enum {
    CF_VALIDATION_POLICY_TYPE_X509 = 0,  // X.509 策略
    CF_VALIDATION_POLICY_TYPE_SSL = 1,   // SSL 策略
} CfValidationPolicyType;
```

### 7.2 吊销检查选项

```c
typedef enum {
    CF_REVOCATION_CHECK_OPTION_PREFER_OCSP = 0,              // 优先 OCSP
    CF_REVOCATION_CHECK_OPTION_ACCESS_NETWORK = 1,           // 允许网络访问
    CF_REVOCATION_CHECK_OPTION_FALLBACK_NO_PREFER = 2,        // 回退非优先
    CF_REVOCATION_CHECK_OPTION_FALLBACK_LOCAL = 3,           // 回退本地
    CF_REVOCATION_CHECK_OPTION_CHECK_INTERMEDIATE_CA_ONLINE = 4,
    CF_REVOCATION_CHECK_OPTION_LOCAL_CRL_ONLY_CHECK_END_ENTITY_CERT = 5,
    CF_REVOCATION_CHECK_OPTION_IGNORE_NETWORK_ERROR = 6,     // 忽略网络错误
} CfRevocationCheckOptionsType;
```

### 7.3 密钥用途

```c
typedef enum {
    CF_KEYUSAGE_DIGITAL_SIGNATURE = 0,
    CF_KEYUSAGE_NON_REPUDIATION = 1,
    CF_KEYUSAGE_KEY_ENCIPHERMENT = 2,
    CF_KEYUSAGE_DATA_ENCIPHERMENT = 3,
    CF_KEYUSAGE_KEY_AGREEMENT = 4,
    CF_KEYUSAGE_KEY_CERT_SIGN = 5,
    CF_KEYUSAGE_CRL_SIGN = 6,
    CF_KEYUSAGE_ENCIPHER_ONLY = 7,
    CF_KEYUSAGE_DECIPHER_ONLY = 8,
} CfValidationKeyUsageType;
```

## 8. 部件配置 (bundle.json)

```json
{
  "component": {
    "name": "certificate_framework",
    "subsystem": "security",
    "syscap": [ "SystemCapability.Security.Cert" ],
    "features": [ "certificate_framework_enabled" ],
    "adapted_system_type": [ "standard" ],
    "rom": "1024KB",
    "ram": "5120KB"
  }
}
```

## 9. 依赖配置

### 9.1 内部依赖

```gn
deps = [
  "ability:libcertificate_framework_ability",
  "adapter:libcertificate_framework_adapter",
  "common:libcertificate_framework_common_static",
  "cert:libcertificate_framework_cert_object",
  "extension:libcertificate_framework_extension_object",
  "v1.0:libcertificate_framework_vesion1",
  "attestation:libcertificate_attestation"
]
```

### 9.2 外部依赖

```gn
external_deps = [
  "c_utils:utils",
  "crypto_framework:crypto_framework_lib",
  "hilog:libhilog",
  "napi:napi",
  "openssl:libcrypto_shared"
]
```
