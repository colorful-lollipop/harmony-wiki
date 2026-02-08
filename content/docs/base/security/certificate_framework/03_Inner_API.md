# 内部 C API

## 1. 概述

内部 C API 是框架核心层对外提供的接口，主要供 N-API 层和 FFI 层调用。定义在 `interfaces/inner_api/` 目录下。

### API 分类

| 分类 | 主要文件 | 职责 |
|------|---------|------|
| 核心 API | cf_api.h | 对象创建 |
| 类型定义 | cf_type.h | 数据结构、枚举、常量 |
| 参数处理 | cf_param.h | 参数集操作 |
| 证书接口 | certificate/*.h | 证书相关接口 |
| CRL 接口 | crl.h | CRL 相关接口 |
| 证书链 | x509_cert_chain.h | 证书链接口 |

## 2. 核心 API (cf_api.h)

**文件路径**: `interfaces/inner_api/include/cf_api.h`

### CfCreate - 对象创建入口

```c
CF_API_EXPORT int32_t CfCreate(CfObjectType objType, const CfEncodingBlob *in, CfObject **object);
```

**功能**: 创建证书框架对象的统一入口

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `objType` | `CfObjectType` | 对象类型枚举 |
| `in` | `const CfEncodingBlob *` | 编码数据输入 |
| `object` | `CfObject **` | 输出对象指针 |

**返回值**: `CfResult` 错误码

**对象类型枚举** (`cf_type.h:27-32`):

```c
typedef enum {
    CF_OBJ_TYPE_CERT,        // 证书对象
    CF_OBJ_TYPE_EXTENSION,   // 扩展对象
    CF_OBJ_TYPE_CRL,         // CRL 对象
    CF_OBJ_TYPE_LIST,        // 列表对象
} CfObjectType;
```

**使用示例**:

```c
CfEncodingBlob in = {
    .encodingFormat = PEM,
    .data = (uint8_t *)pemCert,
    .dataSize = strlen(pemCert)
};

CfObject *certObj = NULL;
int32_t ret = CfCreate(CF_OBJ_TYPE_CERT, &in, &certObj);
if (ret != CF_SUCCESS) {
    // 错误处理
}
```

### CfObject 结构体

```c
struct CfObjectInner {
    int32_t (*get)(const CfObject *object, const CfParamSet *paramSetIn, CfParamSet **paramSetOut);
    int32_t (*check)(const CfObject *object, const CfParamSet *paramSetIn, CfParamSet **paramSetOut);
    void (*destroy)(CfObject **object);
};
```

**方法说明**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `get` | 对象、输入参数集、输出参数集 | CfResult | 获取对象属性 |
| `check` | 对象、输入参数集、输出参数集 | CfResult | 检查对象属性 |
| `destroy` | 对象指针 | void | 销毁对象 |

## 3. 类型定义 (cf_type.h)

### 3.1 基础类型

```c
#define CF_API_EXPORT __attribute__ ((visibility("default")))

typedef struct {
    unsigned long type;
} CfBase;
```

### 3.2 证书项 ID (CfItemId)

```c
typedef enum {
    CF_ITEM_TBS = 0,              // TBS 证书
    CF_ITEM_PUBLIC_KEY,           // 公钥
    CF_ITEM_ISSUER_UNIQUE_ID,     // 颁发者唯一标识
    CF_ITEM_SUBJECT_UNIQUE_ID,    // 主题唯一标识
    CF_ITEM_EXTENSIONS,           // 扩展
    CF_ITEM_ENCODED,              // 编码数据
    CF_ITEM_VERSION,              // 版本
    CF_ITEM_SERIAL_NUMBER,        // 序列号
    CF_ITEM_ISSUE_NAME,           // 颁发者名称
    CF_ITEM_SUBJECT_NAME,         // 主题名称
    CF_ITEM_NOT_BEFORE,           // 有效期起始
    CF_ITEM_NOT_AFTER,            // 有效期结束
    CF_ITEM_SIGNATURE,            // 签名
    CF_ITEM_SIGNATURE_ALG_NAME,   // 签名算法名
    CF_ITEM_INVALID,
} CfItemId;
```

### 3.3 二进制数据 (CfBlob)

```c
typedef struct {
    uint8_t *data;
    uint32_t size;
} CfBlob;
```

### 3.4 参数集 (CfParamSet)

```c
typedef struct {
    uint32_t paramSetSize;
    uint32_t paramsCnt;
    CfParam params[];
} CfParamSet;
```

### 3.5 标签类型 (CfTagType)

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

### 3.6 编码格式

```c
typedef enum {
    CF_ENCODING_UTF8 = 0,
} CfEncodinigType;

typedef enum {
    PEM = 0,
    DER = 1,
} CfEncodinigBaseFormat;
```

### 3.7 常量定义

```c
#define MAX_COUNT_OID          100      // 最大 OID 数量
#define MAX_LEN_OID            128     // 最大 OID 长度
#define MAX_COUNT_NID          1195    // 最大 NID 数量
#define MAX_LEN_CERTIFICATE    65536   // 最大证书长度
#define MAX_LEN_EXTENSIONS     65536   // 最大扩展长度
```

## 4. 参数处理 API (cf_param.h)

### 4.1 CfInitParamSet

```c
int32_t CfInitParamSet(CfParamSet **paramSet);
```

**功能**: 初始化参数集

### 4.2 CfAddParams

```c
int32_t CfAddParams(CfParamSet *paramSet, const CfParam *params, uint32_t paramCnt);
```

**功能**: 添加参数到参数集

### 4.3 CfBuildParamSet

```c
int32_t CfBuildParamSet(CfParamSet **paramSet);
```

**功能**: 构建参数集

### 4.4 CfFreeParamSet

```c
void CfFreeParamSet(CfParamSet **paramSet);
```

**功能**: 释放参数集

### 4.5 CfGetParam

```c
int32_t CfGetParam(const CfParamSet *paramSet, uint32_t tag, CfParam **param);
```

**功能**: 从参数集获取参数

### 4.6 CfGetTagType

```c
CfTagType CfGetTagType(CfTag tag);
```

**功能**: 获取标签类型

## 5. 证书相关接口

### 5.1 HcfCertificate (certificate.h)

```c
typedef struct HcfCertificate HcfCertificate;

struct HcfCertificate {
    struct CfObjectBase base;

    CfResult (*verify)(HcfCertificate *self, void *key);
    CfResult (*getEncoded)(HcfCertificate *self, CfEncodingBlob *encodedByte);
    CfResult (*getPublicKey)(HcfCertificate *self, void **keyOut);
};
```

### 5.2 HcfX509Certificate (x509_certificate.h)

```c
typedef struct HcfX509Certificate HcfX509Certificate;

struct HcfX509Certificate {
    HcfCertificate base;

    CfResult (*checkValidityWithDate)(HcfX509Certificate *self, const char *date);
    long (*getVersion)(HcfX509Certificate *self);
    CfResult (*getSerialNumber)(HcfX509Certificate *self, CfBlob *out);
    CfResult (*getIssuerName)(HcfX509Certificate *self, CfBlob *out);
    CfResult (*getSubjectName)(HcfX509Certificate *self, CfBlob *out);
    CfResult (*getNotBeforeTime)(HcfX509Certificate *self, CfBlob *outDate);
    CfResult (*getNotAfterTime)(HcfX509Certificate *self, CfBlob *outDate);
    CfResult (*getSignature)(HcfX509Certificate *self, CfBlob *sigOut);
    CfResult (*getSignatureAlgName)(HcfX509Certificate *self, CfBlob *outName);
    CfResult (*getSignatureAlgOid)(HcfX509Certificate *self, CfBlob *out);
    CfResult (*getKeyUsage)(HcfX509Certificate *self, CfBlob *boolArr);
    CfResult (*getExtKeyUsage)(HcfX509Certificate *self, CfArray *keyUsageOut);
    int32_t (*getBasicConstraints)(HcfX509Certificate *self);
    CfResult (*match)(HcfX509Certificate *self, 
                      const HcfX509CertMatchParams *matchParams, bool *out);
    // ... 更多方法
};
```

**创建函数**:

```c
CfResult HcfX509CertificateCreate(const CfEncodingBlob *inStream, 
                                  HcfX509Certificate **returnObj);
```

### 5.3 HcfCertChain (x509_cert_chain.h)

```c
typedef struct HcfCertChain HcfCertChain;

struct HcfCertChain {
    struct CfObjectBase base;

    CfResult (*getCertList)(HcfCertChain *self, HcfX509CertificateArray *out);
    CfResult (*validate)(HcfCertChain *self, 
                        const HcfX509CertChainValidateParams *params, 
                        HcfX509CertChainValidateResult *result);
    CfResult (*toString)(HcfCertChain *self, CfBlob *out);
    CfResult (*hashCode)(HcfCertChain *self, CfBlob *out);
};
```

**创建函数**:

```c
CfResult HcfCertChainCreate(const CfEncodingBlob *inStream, 
                            const HcfX509CertificateArray *inCerts, 
                            HcfCertChain **returnObj);
```

### 5.4 HcfX509Crl (x509_crl.h)

```c
typedef struct HcfX509Crl HcfX509Crl;

struct HcfX509Crl {
    struct CfObjectBase base;

    bool (*isRevoked)(HcfX509Crl *self, const HcfX509Certificate *cert);
    CfResult (*getEncoded)(HcfX509Crl *self, CfEncodingBlob *encodedByte);
    CfResult (*verify)(HcfX509Crl *self, void *key);
    long (*getVersion)(HcfX509Crl *self);
    // ... 更多方法
};
```

**创建函数**:

```c
CfResult HcfX509CrlCreate(const CfEncodingBlob *inStream, HcfX509Crl **returnObj);
```

## 6. 能力注册中心 (cf_ability.h)

### 6.1 RegisterAbility

```c
int32_t RegisterAbility(uint32_t abilityId, const CfAbilityBase *abilityFunc);
```

**功能**: 注册能力实现

### 6.2 GetAbility

```c
const CfAbilityBase *GetAbility(uint32_t abilityId);
```

**功能**: 获取能力实现

**能力 ID 编码**:

```c
#define CF_ABILITY(abilityType, objType) (((abilityType) << 24) | (objType))

// 能力类型
#define CF_ABILITY_TYPE_ADAPTER    1  // 适配器类型
#define CF_ABILITY_TYPE_OBJECT     2  // 对象类型
```

## 7. 对象生命周期

### 7.1 创建流程

```
CfCreate()
    │
    ▼
GetAbility(CF_ABILITY_TYPE_ADAPTER, objType)
    │
    ▼
适配器实现 (CfOpensslCreateCert)
    │
    ▼
返回 CfObject*
```

### 7.2 获取属性流程

```
object->get()
    │
    ▼
适配器实现 (内部解析 OpenSSL 结构)
    │
    ▼
填充 CfParamSet 返回
```

### 7.3 销毁流程

```
object->destroy()
    │
    ▼
释放内部资源
    │
    ▼
释放 CfObject 结构
```

## 8. 稳定性标注

### 稳定接口（对外公开）

| 头文件 | 稳定性 | 说明 |
|--------|--------|------|
| cf_api.h | 稳定 | 核心 API |
| cf_type.h | 稳定 | 类型定义 |
| cf_param.h | 稳定 | 参数处理 |
| certificate/*.h | 稳定 | 证书接口 |
| crl.h | 稳定 | CRL 接口 |
| x509_cert_chain.h | 稳定 | 证书链接口 |

### 内部使用接口

| 头文件 | 稳定性 | 说明 |
|--------|--------|------|
| cf_ability.h | 内部 | 能力注册 |
| cf_object_base.h | 内部 | 对象基类 |
| cf_blob.h | 内部 | 二进制数据 |

## 9. 相关文件索引

| 文件 | 路径 | 说明 |
|------|------|------|
| cf_api.h | `interfaces/inner_api/include/` | 核心 API |
| cf_type.h | `interfaces/inner_api/include/` | 类型定义 |
| cf_param.h | `interfaces/inner_api/include/` | 参数处理 |
| cf_ability.h | `frameworks/ability/inc/` | 能力注册 |
| cf_object_base.h | `interfaces/inner_api/common/` | 对象基类 |
| certificate.h | `interfaces/inner_api/certificate/` | 证书基类 |
| x509_certificate.h | `interfaces/inner_api/certificate/` | X509 证书 |
| x509_cert_chain.h | `interfaces/inner_api/certificate/` | 证书链 |
