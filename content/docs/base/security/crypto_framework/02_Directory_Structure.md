# 目录结构与模块职责

## 目的

本文档详细说明 crypto_framework 的目录结构、各模块的职责和关键文件，帮助开发者快速定位代码和理解代码组织。

## 适用范围

- **目标读者**: 开发者、代码审查人员、集成工程师
- **阅读时长**: 20 分钟

## 完整目录结构

```
crypto_framework/
├── BUILD.gn                          # 根构建配置
├── bundle.json                        # OpenHarmony 组件配置
├── LICENSE
├── OAT.xml
├── README_zh.md                      # 中文 README
├── build/                             # 构建配置
│   └── config.gni                     # 构建配置文件
├── common/                            # 公共工具模块
│   ├── BUILD.gn
│   ├── common.gni                     # 公共模块源文件定义
│   ├── inc/                           # 公共头文件
│   │   ├── config.h
│   │   ├── hcf_parcel.h
│   │   ├── hcf_string.h
│   │   ├── log.h
│   │   ├── memory.h
│   │   ├── params_parser.h
│   │   └── utils.h
│   └── src/                           # 公共实现
│       ├── asy_key_params.c
│       ├── blob.c
│       ├── hcf_parcel.c
│       ├── hcf_string.c
│       ├── memory.c
│       ├── object_base.c
│       ├── params_parser.c
│       └── utils.c
├── figures/                           # 文档图片
│   └── zh-cn_crypto_framework_architecture.png
├── frameworks/                        # 框架实现层
│   ├── BUILD.gn                       # 框架层构建配置
│   ├── frameworks.gni                  # 框架层源文件定义
│   ├── spi/                           # SPI (Service Provider Interface)
│   │   ├── asy_key_generator_spi.h
│   │   ├── cipher_factory_spi.h
│   │   ├── dh_key_util_spi.h
│   │   ├── ecc_key_util_spi.h
│   │   ├── kdf_spi.h
│   │   ├── key_agreement_spi.h
│   │   ├── mac_spi.h
│   │   ├── md_spi.h
│   │   ├── rand_spi.h
│   │   ├── signature_spi.h
│   │   └── sym_key_factory_spi.h
│   ├── js/                            # JavaScript 接口封装
│   │   ├── ani/                        # ANI 封装实现
│   │   │   ├── BUILD.gn
│   │   │   ├── dts/
│   │   │   ├── idl/
│   │   │   ├── inc/
│   │   │   └── src/
│   │   ├── jsi/                        # JSI 封装实现
│   │   │   ├── BUILD.gn
│   │   │   ├── inc/
│   │   │   └── src/
│   │   └── napi/                       # N-API 封装实现
│   │       └── crypto/
│   │           ├── BUILD.gn
│   │           ├── inc/
│   │           └── src/
│   │               ├── napi_init.cpp      # N-API 模块注册入口
│   │               ├── napi_rand.cpp
│   │               ├── napi_md.cpp
│   │               ├── napi_mac.cpp
│   │               ├── napi_sign.cpp
│   │               ├── napi_verify.cpp
│   │               ├── napi_cipher.cpp
│   │               ├── napi_key_agreement.cpp
│   │               ├── napi_asy_key_generator.cpp
│   │               ├── napi_asy_key_spec_generator.cpp
│   │               ├── napi_sym_key_generator.cpp
│   │               ├── napi_kdf.cpp
│   │               ├── napi_pri_key.cpp
│   │               ├── napi_pub_key.cpp
│   │               ├── napi_sym_key.cpp
│   │               ├── napi_key_pair.cpp
│   │               ├── napi_key.cpp
│   │               ├── napi_ecc_key_util.cpp
│   │               ├── napi_dh_key_util.cpp
│   │               ├── napi_sm2_crypto_util.cpp
│   │               ├── napi_sm2_ec_signature.cpp
│   │               └── napi_utils.cpp      # N-API 工具函数
│   ├── cj/                            # Cangjie FFI 接口
│   │   ├── BUILD.gn
│   │   ├── include/
│   │   └── src/
│   ├── crypto_operation/                # 密码操作核心实现
│   │   ├── cipher.c
│   │   ├── kdf.c
│   │   ├── key_agreement.c
│   │   ├── mac.c
│   │   ├── md.c
│   │   ├── rand.c
│   │   ├── signature.c
│   │   ├── sm2_crypto_util.c
│   │   └── sm2_ec_signature_data.c
│   ├── key/                           # 密钥管理核心实现
│   │   ├── asy_key_generator.c
│   │   ├── dh_key_util.c
│   │   ├── ecc_key_util.c
│   │   ├── key_utils.c
│   │   └── sym_key_generator.c
│   ├── native/                        # Native C API 实现
│   │   ├── BUILD.gn
│   │   ├── include/
│   │   │   └── native_common.h
│   │   └── src/
│   │       ├── asym_key.c
│   │       ├── crypto_asym_cipher.c
│   │       ├── crypto_common.c
│   │       ├── crypto_kdf.c
│   │       ├── crypto_key_agreement.c
│   │       ├── crypto_mac.c
│   │       ├── crypto_rand.c
│   │       ├── crypto_sym_cipher.c
│   │       ├── digest.c
│   │       ├── native_common.c
│   │       ├── signature.c
│   │       └── sym_key.c
│   ├── algorithm_parameter/             # 算法参数定义
│   │   ├── algorithm_parameter.h
│   │   ├── asy_key_params.h
│   │   ├── detailed_*.h               # 各类详细参数
│   │   ├── kdf_params.h
│   │   ├── mac_params.h
│   │   └── sm2_crypto_params.h
│   └── rand/                          # 随机数
├── interfaces/                        # 对外接口层
│   ├── inner_api/                     # 内部 API
│   │   ├── algorithm_parameter/         # 算法参数接口
│   │   ├── common/                    # 通用定义
│   │   ├── crypto_operation/           # 密码操作接口
│   │   └── key/                      # 密钥接口
│   └── kits/                          # 应用层 API (Kits)
│       └── native/
│           └── include/                # 对外公开头文件
│               ├── crypto_common.h
│               ├── crypto_sym_cipher.h
│               ├── crypto_asym_cipher.h
│               ├── crypto_digest.h
│               ├── crypto_signature.h
│               ├── crypto_mac.h
│               ├── crypto_rand.h
│               ├── crypto_sym_key.h
│               ├── crypto_asym_key.h
│               ├── crypto_kdf.h
│               └── crypto_key_agreement.h
└── plugin/                           # 插件层（第三方库适配）
    ├── BUILD.gn
    ├── plugin.gni                       # 插件层源文件定义
    ├── openssl_plugin/                 # OpenSSL 插件
    │   ├── common/
    │   │   ├── inc/
    │   │   │   ├── aes_openssl_common.h
    │   │   │   ├── dh_openssl_common.h
    │   │   │   ├── ecc_openssl_common.h
    │   │   │   ├── openssl_adapter.h
    │   │   │   └── ...
    │   │   └── src/
    │   ├── crypto_operation/             # 加密操作实现
    │   │   ├── cipher/
    │   │   ├── hmac/
    │   │   ├── kdf/
    │   │   ├── key_agreement/
    │   │   ├── md/
    │   │   ├── rand/
    │   │   └── signature/
    │   └── key/                      # 密钥生成实现
    │       ├── asy_key_generator/
    │       └── sym_key_generator/
    └── mbedtls_plugin/                 # MbedTLS 插件
        ├── common/
        ├── md/
        └── rand/
```

## 模块职责说明

### 1. common/ - 公共工具模块

**职责**: 提供框架内部使用的公共工具函数和数据结构。

**关键功能**:
- **内存管理**: 安全内存分配和释放
- **日志**: 统一日志接口
- **参数解析**: 解析和验证算法参数
- **数据结构**: Blob、String、Parcel 等通用数据结构

**关键文件**:
| 文件 | 说明 |
|------|------|
| `memory.c` | 内存管理实现 |
| `params_parser.c` | 参数解析实现 |
| `blob.c` | 二进制数据块操作 |
| `object_base.c` | 对象基类实现 |

**证据位置**: `common/inc/` 目录

### 2. interfaces/ - 对外接口层

#### 2.1 kits/native/ - Native Kits API

**职责**: 对外公开的 Native C API，供应用程序直接调用。

**命名规范**:
- 所有函数以 `OH_Crypto_` 开头
- 所有结构体以 `Crypto_` 开头
- 所有枚举以 `Crypto_` 开头

**关键文件**:
- `crypto_common.h`: 通用定义和错误码
- `crypto_sym_cipher.h`: 对称加密 API
- `crypto_signature.h`: 签名验签 API
- `crypto_rand.h`: 随机数 API

**证据位置**: `interfaces/kits/native/include/`

#### 2.2 inner_api/ - 内部 API

**职责**: 框架内部模块间调用的接口定义，不对外暴露。

**命名规范**:
- 所有结构体以 `Hcf` 开头（Harmony Crypto Framework）
- 所有枚举以 `Hcf` 开头

**子模块**:
- `common/`: 通用类型（HcfBlob, HcfResult, HcfObjectBase）
- `key/`: 密钥相关接口
- `crypto_operation/`: 密码操作接口
- `algorithm_parameter/`: 算法参数接口

**证据位置**: `interfaces/inner_api/`

### 3. frameworks/ - 框架实现层

#### 3.1 spi/ - SPI 接口定义

**职责**: 定义 Service Provider Interface，插件需要实现的接口规范。

**关键接口**:
| SPI 接口 | 功能 | 实现者 |
|----------|------|----------|
| `HcfMdSpi` | 摘要算法 | OpenSSL Plugin / MbedTLS Plugin |
| `HcfSignSpi` | 签名 | OpenSSL Plugin / MbedTLS Plugin |
| `HcfMacSpi` | MAC | OpenSSL Plugin / MbedTLS Plugin |
| `HcfCipherSpi` | 加解密 | OpenSSL Plugin / MbedTLS Plugin |

**证据位置**: `frameworks/spi/`

#### 3.2 js/ - JavaScript 接口封装

**职责**: 将框架内部接口封装为 JavaScript 可调用的接口。

**子模块**:
| 子模块 | 封装方式 | 适用系统 | 输出产物 |
|--------|----------|----------|----------|
| `napi/` | N-API (Node.js API) | Standard | libcryptoframework_napi.so |
| `ani/` | ANI (Advanced Native Interface) | Standard | crypto_framework_ani |
| `jsi/` | JSI (JavaScript Interface) | Mini | libcryptoframework_jsi.a |

**关键文件**:
- `napi/crypto/src/napi_init.cpp`: N-API 模块注册入口
- `napi/crypto/src/napi_utils.cpp`: N-API 工具函数

**证据位置**: `frameworks/js/`

#### 3.3 cj/ - Cangjie FFI 接口

**职责**: 提供 Cangjie 语言的 Foreign Function Interface。

**输出产物**: `libcj_cryptoframework_ffi.so`

**证据位置**: `frameworks/cj/BUILD.gn`

#### 3.4 crypto_operation/ - 密码操作核心实现

**职责**: 实现密码操作的框架层逻辑，调用底层 SPI 接口。

**关键功能**:
- **加解密**: `cipher.c` - 封装加解密操作
- **签名验签**: `signature.c` - 封装签名操作
- **摘要**: `md.c` - 封装摘要操作
- **MAC**: `mac.c` - 封装 MAC 操作
- **KDF**: `kdf.c` - 封装密钥派生操作
- **密钥协商**: `key_agreement.c` - 封装密钥协商操作
- **随机数**: `rand.c` - 封装随机数生成

**证据位置**: `frameworks/crypto_operation/`

#### 3.5 key/ - 密钥管理核心实现

**职责**: 实现密钥生成、导入导出的框架层逻辑。

**关键功能**:
- **对称密钥生成**: `sym_key_generator.c`
- **非对称密钥生成**: `asy_key_generator.c`
- **ECC 工具**: `ecc_key_util.c` - ECC 曲线参数处理
- **DH 工具**: `dh_key_util.c` - DH 域参数处理

**证据位置**: `frameworks/key/`

#### 3.6 native/ - Native C API 实现

**职责**: 实现 Native Kits API，将内部接口转换为对外 API。

**映射关系**:
```
Native Kits API (OH_Crypto_*) → 内部 API (Hcf*) → 框架实现 → 插件实现
```

**关键文件**:
- `crypto_common.c`: 通用 API 实现
- `sym_cipher.c`: 对称加密 API 实现
- `signature.c`: 签名 API 实现
- `rand.c`: 随机数 API 实现

**证据位置**: `frameworks/native/`

### 4. plugin/ - 插件层

#### 4.1 openssl_plugin/ - OpenSSL 插件

**职责**: 实现基于 OpenSSL 的密码算法插件。

**关键模块**:
- `common/`: OpenSSL 通用工具和适配器
- `crypto_operation/`: 各类加密操作的 OpenSSL 实现
  - `cipher/`: AES/SM4 加密实现
  - `signature/`: RSA/ECDSA/SM2 签名实现
  - `md/`: SHA/SM3 摘要实现
  - `hmac/`: HMAC/CMAC 实现
  - `kdf/`: PBKDF2/HKDF/Scrypt 实现
  - `key_agreement/`: ECDH/DH 实现
  - `rand/`: 随机数生成实现
- `key/`: 密钥生成的 OpenSSL 实现
  - `asy_key_generator/`: RSA/ECC/DSA 密钥生成
  - `sym_key_generator/`: AES/SM4 密钥生成

**证据位置**: `plugin/openssl_plugin/`

#### 4.2 mbedtls_plugin/ - MbedTLS 插件

**职责**: 实现基于 MbedTLS 的密码算法插件（轻量系统）。

**支持的功能**:
- 基础摘要算法
- 随机数生成

**证据位置**: `plugin/mbedtls_plugin/`

### 5. build/ - 构建配置

**职责**: 构建系统的配置文件。

**关键文件**:
- `config.gni`: 全局构建配置

**证据位置**: `build/config.gni`

## 代码组织原则

### 1. 分层架构

```
API 层 (interfaces/)
  ↑
框架层 (frameworks/)
  ↑
插件层 (plugin/)
```

### 2. 接口与实现分离

- **接口定义**: `interfaces/inner_api/` 和 `frameworks/spi/`
- **框架实现**: `frameworks/crypto_operation/`, `frameworks/key/`
- **插件实现**: `plugin/`

### 3. 多语言接口

同一框架实现，通过不同的绑定层支持多种语言：
- N-API: `frameworks/js/napi/`
- ANI: `frameworks/js/ani/`
- JSI: `frameworks/js/jsi/`
- CJ: `frameworks/cj/`
- Native: `frameworks/native/`

## 快速定位指南

### 查找 N-API 接口

**目标**: 找到某个 JS 方法对应的 C++ 实现

**步骤**:
1. 确定 JS 类名（如 `Sign`）
2. 打开 `frameworks/js/napi/crypto/src/`
3. 查找对应文件（如 `napi_sign.cpp`）
4. 搜索方法名（如 `sign`）

**示例**: 查找 `Sign.update` 方法
- 文件: `frameworks/js/napi/crypto/src/napi_sign.cpp`
- 函数: `NapiSign::Update`

### 查找 Native API 实现

**目标**: 找到某个 Native C 函数的实现

**步骤**:
1. 确定函数名（如 `OH_Crypto_SymCipher_Create`）
2. 打开 `frameworks/native/src/`
3. 查找对应文件（如 `crypto_sym_cipher.c`）
4. 搜索函数名

### 查找 SPI 实现

**目标**: 找到某个算法的具体实现

**步骤**:
1. 确定算法类型（如 `AES-CBC`）
2. 根据系统类型选择插件目录：
   - Standard: `plugin/openssl_plugin/`
   - Mini: `plugin/mbedtls_plugin/`
3. 查找对应实现文件

**示例**: 查找 AES-CBC 加密实现
- 文件: `plugin/openssl_plugin/crypto_operation/cipher/src/cipher_aes_openssl.c`
- 函数: `AesCipherOpenSslSpiInit`

### 查找错误码定义

**内部错误码**:
- 文件: `interfaces/inner_api/common/result.h`

**对外错误码**:
- 文件: `interfaces/kits/native/include/crypto_common.h`

## 相关跳转

- **项目定位**: [01_Project_Positioning.md](01_Project_Positioning.md)
- **架构设计**: [03_Architecture.md](03_Architecture.md)
- **对外 API**: [04_External_API.md](04_External_API.md)

## 更新记录

- **2026-02-06**: 创建文档，基于代码结构分析生成
