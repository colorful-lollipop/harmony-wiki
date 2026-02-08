# 编译产物说明

## 1. 产物概述

证书算法库框架编译后产生的主要产物包括动态库、静态库和头文件。

## 2. 产物清单

### 2.1 动态库 (.so)

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `libcertificate_framework_lib.so` | system/lib64/ | 主框架库，导出 C API |
| `libcertificate_framework_adapter_openssl.so` | system/lib64/ | OpenSSL 适配器实现 |
| `libcertificate_napi.so` | system/lib64/ | N-API 接口层 |

### 2.2 静态库 (.a)

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `libcertificate_framework_ability.a` | system/lib64/ | 能力注册中心 |
| `libcertificate_framework_cert_object.a` | system/lib64/ | 证书对象管理 |
| `libcertificate_framework_extension_object.a` | system/lib64/ | 扩展对象管理 |
| `libcertificate_framework_vesion1.a` | system/lib64/ | v1.0 业务实现 |
| `libcertificate_attestation.a` | system/lib64/ | 证书证明模块 |

### 2.3 头文件

头文件导出位置: `interfaces/inner_api/`

| 头文件分类 | 头文件列表 |
|-----------|-----------|
| 核心类型 | cf_api.h, cf_type.h, cf_param.h |
| 证书接口 | certificate.h, x509_certificate.h, x509_cert_chain.h |
| CRL 接口 | crl.h, x509_crl.h, x509_crl_entry.h |
| 公共类型 | cf_blob.h, cf_object_base.h, cf_result.h |
| 证书属性 | x509_cert_match_parameters.h, x509_crl_match_parameters.h |
| 验证结果 | x509_cert_chain_validate_params.h, x509_cert_chain_validate_result.h |
| 信任锚 | x509_trust_anchor.h, x509_distinguished_name.h |

## 3. 运行时加载关系

### 3.1 库依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                    application (JS/TS)                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  libcertificate_napi.so                      │
│              (N-API 胶水层, 依赖 libnapi.so)                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              libcertificate_framework_lib.so                  │
│              (框架主库, 导出 C API)                           │
└─────────────────────────────────────────────────────────────┘
         │              │              │              │
         ▼              ▼              ▼              ▼
┌─────────────────┐ ┌──────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ ability.a       │ │ adapter.so   │ │ cert_object.a   │ │ extension_obj.a │
└─────────────────┘ └──────────────┘ └─────────────────┘ └─────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 libcrypto.so (OpenSSL)                      │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 加载顺序

1. **系统启动**: `libcrypto.so` (OpenSSL) 由系统加载
2. **模块加载**: `libcertificate_framework_adapter_openssl.so` 被按需加载
3. **框架初始化**: `libcertificate_framework_lib.so` 被 N-API 层加载
4. **应用调用**: 应用通过 `libcertificate_napi.so` 调用证书功能

### 3.3 dlopen 加载

适配器通过 `dlsym` 动态加载:

```c
// 适配器注册时 (cf_adapter_ability.c)
__attribute__((constructor)) static void LoadAdapterAbility(void)
{
    RegisterAbility(CF_ABILITY(CF_ABILITY_TYPE_ADAPTER, CF_OBJ_TYPE_CERT), 
                    &g_certAdapterFunc.base);
}
```

## 4. 符号导出

### 4.1 导出符号 (nm -D)

```
libcertificate_framework_lib.so:
                 U BIO_new_mem_buf
                 U EVP_DigestInit_ex
                 U EVP_DigestUpdate
                 U OPENSSL_free
                 U OPENSSL_malloc
                 U OPENSSL_zalloc
                 U RAND_bytes
                 U X509_CRL_free
                 U X509_CRL_get0_by_cert
                 U X509_EXTENSION_free
                 U X509_free
                 U X509_get0_notAfter
                 U X509_get0_notBefore
                 U X509_get_ext_d2i
                 U X509_get_pubkey
                 U X509_get_signature_nid
                 U X509_new
                 U d2i_X509_bio
                 U sk_num
                 U sk_pop_free
                 U sk_value
                 T CfCreate
                 T CfFreeParamSet
                 T CfGetParam
                 T CfInitParamSet
```

### 4.2 符号可见性

```c
// cf_type.h
#define CF_API_EXPORT __attribute__ ((visibility("default")))
```

## 5. 安装路径

### 5.1 系统库路径

| 产物 | 目标路径 |
|------|---------|
| .so 库 | `/system/lib64/` |
| .a 静态库 | `/system/lib64/` |

### 5.2 头文件路径

| 头文件类型 | 导出路径 |
|-----------|---------|
| 内部 API | `/interfaces/inner_api/` (编译时使用) |
| SDK 头文件 | 通过 `inner_kits` 配置导出 |

## 6. 产物使用

### 6.1 JS/TS 应用

```javascript
import cert from '@ohos.security.cert';

// 直接使用，无需显式链接库
let x509Cert = await cert.createX509Cert(encodingBlob);
```

### 6.2 C/C++ 模块

```c
#include "cf_api.h"

// 链接: -lcertificate_framework_lib -lcrypto
CfEncodingBlob in = {
    .encodingFormat = PEM,
    .data = (uint8_t *)pemCert,
    .dataSize = strlen(pemCert)
};
CfObject *certObj = NULL;
int32_t ret = CfCreate(CF_OBJ_TYPE_CERT, &in, &certObj);
```

## 7. 产物验证

### 7.1 检查库存在

```bash
ls -la /system/lib64/libcertificate_framework_*.so
```

### 7.2 检查符号导出

```bash
nm -D /system/lib64/libcertificate_framework_lib.so | grep " T "
```

### 7.3 检查依赖

```bash
ldd /system/lib64/libcertificate_napi.so
```
