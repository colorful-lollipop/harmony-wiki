# 目录结构与代码地图

本文档提供证书算法库框架的完整目录结构和代码导航，帮助开发者快速定位关键文件和功能模块。

## 1. 顶层目录结构

```
certificate_framework/                    # 项目根目录
├── bundle.json                           # 部件配置文件
├── BUILD.gn                              # GN 构建根配置
├── cf.gni                                # GN 构建变量定义
├── LICENSE                               # Apache 2.0 许可证
├── OAT.xml                               # OpenArkCompiler 兼容性配置
├── README.md                             # 项目说明文档
├── README-en.md                          # 英文版说明文档
├── config/                               # 构建配置目录
│   └── build/
│       └── BUILD.gn                      # 覆盖率配置
├── figures/                              # README 图片资源
├── frameworks/                           # 框架实现层（核心代码）
├── interfaces/                           # 对外接口目录
├── test/                                 # 测试代码（排除在文档外）
└── wiki/                                 # Wiki 文档
```

**证据来源**: `ls -la` 目录列表, `README.md:22-38`

---

## 2. Frameworks 目录详解

### 2.1 整体结构

```
frameworks/
├── ability/                              # 框架能力注册模块
│   ├── inc/
│   │   └── cf_ability.h                 # 能力注册接口定义
│   └── src/
│       └── cf_ability.cpp               # 能力注册实现
├── adapter/                              # 算法库适配层
│   ├── v1.0/                            # v1.0 SPI 适配模式
│   │   ├── inc/                         # 头文件目录
│   │   │   ├── certificate_openssl_class.h
│   │   │   ├── certificate_openssl_common.h
│   │   │   ├── x509_crl_entry_openssl.h
│   │   │   ├── x509_cert_chain_openssl.h
│   │   │   ├── x509_cert_chain_openssl_ex.h
│   │   │   ├── x509_distinguished_name_openssl.h
│   │   │   ├── x509_certificate_openssl.h
│   │   │   ├── x509_csr_openssl.h
│   │   │   ├── x509_cert_chain_validator_openssl.h
│   │   │   ├── x509_crl_openssl.h
│   │   │   ├── x509_cert_cms_generator_openssl.h
│   │   │   └── x509_certificate_create.h
│   │   └── src/                         # 实现文件目录
│   │       └── x509_certificate_openssl.c
│   │       └── x509_cert_chain_openssl.c
│   ├── v2.0/                            # v2.0 Ability 适配模式
│   │   ├── inc/
│   │   │   ├── cf_adapter_cert_openssl.h
│   │   │   └── cf_adapter_extension_openssl.h
│   │   └── src/
│   └── attestation/                      # 认证证书支持
│       ├── inc/
│       │   └── attestation_cert_verify.h
│       └── src/
│           └── attestation_cert_verify.c
├── common/                               # 公共工具模块
│   └── v1.0/
│       ├── inc/                         # 头文件目录
│       │   ├── cf_blob.h               # Blob 数据结构
│       │   ├── cf_check.h              # 输入验证
│       │   ├── cf_log.h                # 日志接口
│       │   └── cf_memory.h             # 内存管理
│       └── src/                         # 实现文件目录
│           ├── cf_blob.c
│           ├── cf_check.c
│           ├── cf_log.c
│           ├── cf_memory.c
│           └── utils.c
├── core/                                 # 框架核心模块
│   ├── v1.0/                            # v1.0 业务逻辑
│   │   ├── spi/                        # SPI 接口定义
│   │   │   ├── cert_chain_validator_spi.h
│   │   │   ├── cert_cms_generator_spi.h
│   │   │   ├── x509_cert_chain_spi.h
│   │   │   ├── x509_certificate_spi.h
│   │   │   ├── x509_crl_spi.h
│   │   │   └── x509_distinguished_name_spi.h
│   │   ├── certificate/                 # 证书操作模块
│   │   │   └── cert_chain_validator.c
│   │   └── BUILD.gn
│   ├── cert/                            # 证书对象管理
│   │   ├── inc/
│   │   │   ├── cf_cert_adapter_ability_define.h
│   │   │   └── cf_object_cert.h
│   │   └── src/
│   │       └── cf_object_cert.c
│   ├── extension/                        # 扩展对象管理
│   │   ├── inc/
│   │   │   ├── cf_adapter_extension_ability_define.h
│   │   │   └── cf_object_extension.h
│   │   └── src/
│   │       └── cf_object_extension.c
│   ├── life/                             # 对象生命周期
│   │   ├── inc/
│   │   │   └── cf_object_ability_define.h
│   │   └── src/
│   │       └── cf_object_life.cpp
│   ├── param/                             # 参数处理
│   │   ├── inc/
│   │   │   └── cf_param_parse.h
│   │   └── src/
│   │       └── cf_param_parse.cpp
│   ├── attestation/                       # 认证处理
│   │   ├── inc/
│   │   │   └── cf_attestation_ability_define.h
│   │   └── src/
│   │       └── cf_attestation.cpp
│   └── BUILD.gn
├── js/                                   # JavaScript 绑定层
│   ├── ani/                              # ANI 接口封装
│   │   ├── inc/                         # 头文件目录
│   │   │   ├── ani_cert_chain_validator.h
│   │   │   ├── ani_cert_chain_build_result.h
│   │   │   ├── ani_cert_cms_generator.h
│   │   │   ├── ani_cert_crl_collection.h
│   │   │   ├── ani_cert_extension.h
│   │   │   ├── ani_cert_extension_ability.h
│   │   │   ├── ani_common.h
│   │   │   ├── ani_constructor.h
│   │   │   ├── ani_parameters.h
│   │   │   ├── ani_pub_key.h
│   │   │   ├── ani_x500_distinguished_name.h
│   │   │   ├── ani_x509_cert.h
│   │   │   ├── ani_x509_cert_chain.h
│   │   │   ├── ani_x509_cert_chain_validate_result.h
│   │   │   ├── ani_x509_crl.h
│   │   │   ├── ani_x509_crl_entry.h
│   │   │   └── ani_object.h
│   │   └── src/                         # 实现文件目录
│   │       ├── ani_cert_chain_validator.cpp
│   │       └── ani_x509_cert.cpp
│   └── napi/                             # N-API 接口封装
│       └── certificate/                  # N-API 模块
│           ├── inc/                     # 头文件目录
│           │   ├── napi_cert_cms_generator.h
│           │   ├── napi_cert_chain_validator.h
│           │   ├── napi_cert_defines.h
│           │   ├── napi_cert_extension.h
│           │   ├── napi_cert_utils.h
│           │   ├── napi_common.h
│           │   ├── napi_key.h
│           │   ├── napi_object.h
│           │   ├── napi_pub_key.h
│           │   ├── napi_x509_cert_chain.h
│           │   ├── napi_x509_cert_chain_validate_params.h
│           │   ├── napi_x509_cert_chain_validate_result.h
│           │   ├── napi_x509_cert_match_parameters.h
│           │   ├── napi_x509_certificate.h
│           │   ├── napi_x509_crl.h
│           │   ├── napi_x509_crl_entry.h
│           │   └── napi_x509_distinguished_name.h
│           └── src/                     # 实现文件目录
│               ├── napi_certificate_init.cpp    # 模块注册入口
│               ├── napi_cert_extension.cpp
│               ├── napi_cert_utils.cpp
│               ├── napi_common.cpp
│               ├── napi_key.cpp
│               ├── napi_object.cpp
│               ├── napi_pub_key.cpp
│               ├── napi_x509_cert_chain.cpp
│               ├── napi_x509_certificate.cpp
│               ├── napi_x509_crl.cpp
│               ├── napi_x509_crl_entry.cpp
│               └── napi_x509_distinguished_name.cpp
├── cj/                                   # Cangjie FFI 绑定
│   └── BUILD.gn
└── BUILD.gn
```

**证据来源**: `glob` 搜索结果, `README.md:28-35`

### 2.2 目录职责速查表

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `ability/` | 能力注册中心 | `cf_ability.h`, `cf_ability.cpp` |
| `adapter/v1.0/` | OpenSSL v1.0 适配器 | `x509_certificate_openssl.h`, `x509_cert_chain_openssl.h` |
| `adapter/v2.0/` | OpenSSL v2.0 适配器 | `cf_adapter_cert_openssl.h` |
| `adapter/attestation/` | 认证证书支持 | `attestation_cert_verify.h` |
| `common/v1.0/` | 公共工具 | `cf_check.h`, `cf_memory.h`, `cf_blob.h` |
| `core/v1.0/spi/` | SPI 接口定义 | `x509_certificate_spi.h`, `x509_crl_spi.h` |
| `core/cert/` | 证书对象 | `cf_object_cert.h`, `cf_cert_adapter_ability_define.h` |
| `core/extension/` | 扩展对象 | `cf_object_extension.h` |
| `core/life/` | 生命周期 | `cf_object_ability_define.h` |
| `core/param/` | 参数解析 | `cf_param_parse.h` |
| `js/napi/certificate/` | N-API 实现 | `napi_certificate_init.cpp` |

---

## 3. Interfaces 目录详解

```
interfaces/
└── inner_api/                            # 内部 API 头文件
    ├── include/                          # 核心 API
    │   ├── cf_api.h                     # 主入口 CfCreate()
    │   ├── cf_type.h                    # 类型定义枚举
    │   ├── cf_param.h                   # 参数结构
    │   └── cf_result.h                  # 错误码定义
    ├── common/                           # 公共接口
    │   ├── cf_blob.h                    # Blob 数据结构
    │   ├── cf_object_base.h             # 对象基类
    │   ├── cf_result.h                  # 错误码
    │   └── cf_type.h                    # 类型定义
    └── certificate/                      # 证书相关接口
        ├── cert_chain_validator.h        # 链校验器接口
        ├── cert_crl_common.h            # CRL 公共接口
        ├── cert_crl_collection.h         # CRL 集合接口
        ├── certificate.h                 # 证书接口
        ├── x509_cert_chain.h            # X.509 证书链
        ├── x509_cert_chain_validate_params.h  # 校验参数
        ├── x509_cert_chain_validate_result.h   # 校验结果
        ├── x509_cert_match_parameters.h       # 匹配参数
        ├── x509_certificate.h            # X.509 证书
        ├── x509_crl.h                   # X.509 CRL
        ├── x509_crl_entry.h             # CRL 条目
        ├── x509_crl_match_parameters.h  # CRL 匹配参数
        ├── x509_distinguished_name.h    # 可分辨名称
        └── x509_trust_anchor.h         # 信任锚
```

**证据来源**: `bundle.json:42-63`

---

## 4. 核心功能→文件映射

### 4.1 证书创建与解析

| 功能 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| 证书创建入口 | `interfaces/inner_api/include/cf_api.h` | 32CfCreate()` 工厂函数 |
| | ` 证书 SPI 定义 | `frameworks/core/v1.0/spi/x509_certificate_spi.h` | 27-87 | SPI 接口方法清单 |
| OpenSSL 适配 | `frameworks/adapter/v1.0/inc/x509_certificate_openssl.h` | - | OpenSSL 证书对象 |
| N-API 注册 | `frameworks/js/napi/certificate/src/napi_certificate_init.cpp` | 443-455 | 模块注册入口 |
| N-API 类定义 | `frameworks/js/napi/certificate/inc/napi_x509_certificate.h` | - | JS 类接口 |

### 4.2 CRL 操作

| 功能 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| CRL SPI 定义 | `frameworks/core/v1.0/spi/x509_crl_spi.h` | - | CRL SPI 接口 |
| CRL OpenSSL | `frameworks/adapter/v1.0/inc/x509_crl_openssl.h` | - | OpenSSL CRL 适配 |
| CRL N-API | `frameworks/js/napi/certificate/inc/napi_x509_crl.h` | - | JS 类接口 |

### 4.3 证书链校验

| 功能 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| 链 SPI 定义 | `frameworks/core/v1.0/spi/x509_cert_chain_spi.h` | - | 链 SPI 接口 |
| 链校验器 SPI | `frameworks/core/v1.0/spi/cert_chain_validator_spi.h` | - | 校验器接口 |
| 校验实现 | `frameworks/core/v1.0/certificate/cert_chain_validator.c` | 128 | 校验逻辑 |
| OpenSSL 链 | `frameworks/adapter/v1.0/inc/x509_cert_chain_openssl.h` | - | OpenSSL 实现 |

### 4.4 输入验证

| 功能 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| Blob 验证 | `frameworks/common/v1.0/src/cf_check.c` | 21 | `CfCheckBlob()` |
| 编码验证 | `frameworks/common/v1.0/src/cf_check.c` | 30 | `CfCheckEncodingBlob()` |
| 字符串验证 | `frameworks/common/v1.0/src/utils.c` | 27 | `CfIsStrValid()` |
| URL 验证 | `frameworks/common/v1.0/src/utils.c` | 72 | `CfIsUrlValid()` |

### 4.5 内存管理

| 功能 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| 内存分配 | `frameworks/common/v1.0/inc/cf_memory.h` | 25 | `CfMalloc()`, `CfMallocEx()` |
| 内存释放 | `frameworks/common/v1.0/inc/cf_memory.h` | 27 | `CfFree()` |
| 安全擦除 | `frameworks/common/v1.0/src/cf_blob.c` | 53 | `CfBlobDataClearAndFree()` |

---

## 5. 构建配置→文件映射

### 5.1 GN 构建文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 根构建目标定义 |
| `cf.gni` | 构建变量定义 |
| `frameworks/BUILD.gn` | 框架库组 |
| `frameworks/core/BUILD.gn` | 核心共享库 |
| `frameworks/adapter/BUILD.gn` | 适配器静态库 |
| `frameworks/common/BUILD.gn` | 公共静态库 |
| `frameworks/ability/BUILD.gn` | 能力注册静态库 |
| `frameworks/js/napi/certificate/BUILD.gn` | N-API 共享库 |
| `frameworks/js/ani/BUILD.gn` | ANI 共享库 |
| `frameworks/cj/BUILD.gn` | Cangjie FFI 共享库 |
| `test/unittest/BUILD.gn` | 测试套件 |

### 5.2 部件配置

| 文件 | 用途 |
|------|------|
| `bundle.json` | 部件元数据、Inner Kits 定义、依赖声明 |

---

## 6. 代码导航快捷入口

### 6.1 常用搜索模式

```bash
# 查找证书相关实现
rg "X509Certificate" --type cpp

# 查找 N-API 注册代码
rg "RegisterCertModule" --type cpp

# 查找 SPI 接口定义
rg "struct Hcf.*Spi" --type cpp

# 查找输入验证代码
rg "CfCheck" --type cpp

# 查找错误码定义
rg "CF_" --type h
```

### 6.2 关键符号速查

| 符号类型 | 符号名 | 位置 |
|---------|--------|------|
| 工厂函数 | `CfCreate` | `cf_api.h:32` |
| 对象接口 | `CfObjectInner` | `cf_api.h:22-26` |
| Blob 类型 | `CfEncodingBlob` | `cf_type.h` |
| 参数集 | `CfParamSet` | `cf_type.h:187-191` |
| 错误码 | `CfResult` | `cf_result.h` |

### 6.3 类继承关系

```
CfObjectBase (基类)
├── HcfX509CertificateSpi (证书 SPI)
│   └── HcfOpensslX509Cert (OpenSSL 实现)
├── HcfX509CrlSpi (CRL SPI)
│   └── HcfOpensslX509Crl (OpenSSL 实现)
├── HcfX509CertChainSpi (证书链 SPI)
│   └── HcfOpensslX509CertChain (OpenSSL 实现)
└── CertChainValidatorSpi (链校验器 SPI)
    └── HcfOpensslCertChainValidator (OpenSSL 实现)
```

---

## 7. 命名约定

### 7.1 文件命名

| 前缀 | 用途 | 示例 |
|------|------|------|
| `cf_` | 核心框架文件 | `cf_api.h`, `cf_check.c` |
| `napi_` | N-API 实现 | `napi_certificate_init.cpp` |
| `ani_` | ANI 实现 | `ani_x509_cert.cpp` |
| `x509_` | X.509 相关 | `x509_certificate_openssl.h` |
| `openssl_` | OpenSSL 适配 | `openssl_cert_class.h` |

### 7.2 符号命名

| 前缀 | 用途 | 示例 |
|------|------|------|
| `Cf` | 框架核心类型 | `CfBlob`, `CfResult`, `CfCreate` |
| `Hcf` | SPI 实现类型 | `HcfX509CertificateSpi` |
| `Napi` | N-API 封装类 | `NapiX509Certificate` |
| `Ani` | ANI 封装类 | `AniX509Cert` |

---

## 8. 测试目录（供参考）

```
test/                                      # 测试代码（不纳入文档范围）
├── fuzztest/                              # Fuzz 测试
│   ├── cfcreate_fuzzer/
│   ├── cfgetandcheck_fuzzer/
│   ├── cfparam_fuzzer/
│   └── v1.0/
│       ├── x509certificate_fuzzer/
│       ├── x509certchain_fuzzer/
│       ├── x509crl_fuzzer/
│       └── x509distinguishedname_fuzzer/
└── unittest/                               # 单元测试
    ├── BUILD.gn
    ├── cf_core_test/
    ├── cf_adapter_test/
    ├── cf_sdk_test/
    └── v1.0/
```

**注意**: 测试代码不在本文档范围内，仅作目录结构完整性记录。

---

*最后更新: 2025-02-07*
*基于代码版本: certificate_framework v4.0*
