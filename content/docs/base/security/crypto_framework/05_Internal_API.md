# 内部 API (Internal API)

## 目的

本文档说明 crypto_framework 内部模块间的 API 接口，包括框架内部接口、SPI 接口、依赖方向等。

## 适用范围

- **目标读者**: 框架开发者、插件开发者、安全审计人员
- **阅读时长**: 30 分钟

## 命名规范

- **结构体前缀**: `Hcf` (Harmony Crypto Framework)
- **函数前缀**: `Hcf`
- **枚举前缀**: `Hcf`

## 核心接口

### 通用类型接口 (Common)

#### HcfResult - 结果码

**位置**: `interfaces/inner_api/common/result.h:19-38`

```c
typedef enum HcfResult {
    HCF_SUCCESS = 0,
    HCF_INVALID_PARAMS = -10001,
    HCF_NOT_SUPPORT = -10002,
    HCF_ERR_MALLOC = -20001,
    HCF_ERR_NAPI = -20002,
    HCF_ERR_ANI = -20002,
    HCF_ERR_PARAMETER_CHECK_FAILED = -20003,
    HCF_ERR_CRYPTO_OPERATION = -30001,
} HcfResult;
```

#### HcfBlob - 二进制数据块

**位置**: `interfaces/inner_api/common/blob.h`

```c
typedef struct HcfBlob {
    uint8_t *data;  // 数据指针
    size_t len;      // 数据长度
} HcfBlob;
```

#### HcfObjectBase - 对象基类

**位置**: `interfaces/inner_api/common/object_base.h`

```c
typedef struct HcfObjectBase {
    // 类类型标识
    int32_t type;
    
    // 销毁函数
    void (*destroy)(struct HcfObjectBase *self);
    
    // 获取类标识
    int32_t (*getClass)(const struct HcfObjectBase *self);
} HcfObjectBase;
```

## 密钥接口

### HcfKey - 密钥基类

**位置**: `interfaces/inner_api/key/key.h`

**主要方法**:
```c
typedef struct HcfKey {
    HcfObjectBase base;  // 继承自基类
    
    // 获取编码数据
    HcfResult (*getEncoded)(struct HcfKey *self, HcfBlob *blob);
    
    // 获取算法名称
    const char* (*getAlgName)(struct HcfKey *self);
    
    // 获取格式
    const char* (*getFormat)(struct HcfKey *self);
} HcfKey;
```

### HcfSymKey - 对称密钥

**位置**: `interfaces/inner_api/key/sym_key.h`

**主要方法**:
```c
typedef struct HcfSymKey {
    HcfKey base;  // 继承自密钥基类
    
    // 清除密钥内存
    HcfResult (*clearMem)(struct HcfSymKey *self);
} HcfSymKey;
```

### HcfPriKey - 私钥

**位置**: `interfaces/inner_api/key/pri_key.h`

**主要方法**:
```c
typedef struct HcfPriKey {
    HcfKey base;  // 继承自密钥基类
    
    // 获取公钥
    HcfResult (*getPubKey)(struct HcfPriKey *self, struct HcfPubKey **pubKey);
    
    // 获取密钥规格
    HcfResult (*getAsyKeySpec)(struct HcfPriKey *self, HcfAsyKeyParamsSpec **spec);
} HcfPriKey;
```

### HcfPubKey - 公钥

**位置**: `interfaces/inner_api/key/pub_key.h`

**主要方法**:
```c
typedef struct HcfPubKey {
    HcfKey base;  // 继承自密钥基类
    
    // 获取密钥规格
    HcfResult (*getAsyKeySpec)(struct HcfPubKey *self, HcfAsyKeyParamsSpec **spec);
} HcfPubKey;
```

### HcfAsyKeyGenerator - 非对称密钥生成器

**位置**: `interfaces/inner_api/key/asy_key_generator.h`

**主要方法**:
```c
typedef struct HcfAsyKeyGenerator {
    HcfObjectBase base;
    
    // 生成密钥对（异步）
    HcfResult (*generateKeyPair)(struct HcfAsyKeyGenerator *self, 
        HcfKeyPair **keyPair);
    
    // 转换密钥
    HcfResult (*convertKey)(struct HcfAsyKeyGenerator *self, 
        HcfBlob *blob, HcfKey *key);
    
    // 转换 PEM 密钥
    HcfResult (*convertPemKey)(struct HcfAsyKeyGenerator *self, 
        HcfBlob *pem, HcfKey *key);
} HcfAsyKeyGenerator;
```

### HcfSymKeyGenerator - 对称密钥生成器

**位置**: `interfaces/inner_api/key/sym_key_generator.h`

**主要方法**:
```c
typedef struct HcfSymKeyGenerator {
    HcfObjectBase base;
    
    // 生成对称密钥
    HcfResult (*generateSymKey)(struct HcfSymKeyGenerator *self, 
        HcfSymKey **symKey);
    
    // 转换密钥
    HcfResult (*convertKey)(struct HcfSymKeyGenerator *self, 
        HcfBlob *blob, HcfSymKey **symKey);
} HcfSymKeyGenerator;
```

## 密码操作接口

### HcfCipher - 加密/解密

**位置**: `interfaces/inner_api/crypto_operation/cipher.h`

**主要方法**:
```c
typedef struct HcfCipher {
    HcfObjectBase base;
    
    // 初始化
    HcfResult (*init)(struct HcfCipher *self, HcfCipherMode mode,
        HcfKey *key, HcfAlgorithmParameter *params);
    
    // 更新数据
    HcfResult (*update)(struct HcfCipher *self, HcfBlob *in,
        HcfBlob *out);
    
    // 完成操作
    HcfResult (*doFinal)(struct HcfCipher *self, HcfBlob *in,
        HcfBlob *out);
    
    // 设置加密规格
    HcfResult (*setCipherSpec)(struct HcfCipher *self,
        HcfAlgorithmParameter *spec);
    
    // 获取加密规格
    HcfResult (*getCipherSpec)(struct HcfCipher *self,
        HcfAlgorithmParameter **spec);
} HcfCipher;
```

### HcfMd - 消息摘要

**位置**: `interfaces/inner_api/crypto_operation/md.h`

**主要方法**:
```c
typedef struct HcfMd {
    HcfObjectBase base;
    
    // 更新数据
    HcfResult (*update)(struct HcfMd *self, HcfBlob *data);
    
    // 完成摘要
    HcfResult (*doFinal)(struct HcfMd *self, HcfBlob *result);
    
    // 获取摘要长度
    HcfResult (*getMdLength)(struct HcfMd *self, int32_t *length);
} HcfMd;
```

### HcfSign - 签名

**位置**: `interfaces/inner_api/crypto_operation/signature.h`

**主要方法**:
```c
typedef struct HcfSign {
    HcfObjectBase base;
    
    // 初始化签名
    HcfResult (*init)(struct HcfSign *self, HcfPriKey *priKey,
        HcfAlgorithmParameter *params);
    
    // 更新数据
    HcfResult (*update)(struct HcfSign *self, HcfBlob *data);
    
    // 生成签名
    HcfResult (*sign)(struct HcfSign *self, HcfBlob *signature);
    
    // 设置签名规格
    HcfResult (*setSignSpec)(struct HcfSign *self,
        HcfAlgorithmParameter *spec);
} HcfSign;
```

### HcfVerify - 验签

**位置**: `interfaces/inner_api/crypto_operation/signature.h`

**主要方法**:
```c
typedef struct HcfVerify {
    HcfObjectBase base;
    
    // 初始化验签
    HcfResult (*init)(struct HcfVerify *self, HcfPubKey *pubKey,
        HcfAlgorithmParameter *params);
    
    // 更新数据
    HcfResult (*update)(struct HcfVerify *self, HcfBlob *data);
    
    // 验证签名
    HcfResult (*verify)(struct HcfVerify *self, HcfBlob *signature,
        bool *result);
    
    // 恢复数据（部分签名算法）
    HcfResult (*recover)(struct HcfVerify *self, HcfBlob *signature,
        HcfBlob *data);
} HcfVerify;
```

### HcfMac - 消息认证码

**位置**: `interfaces/inner_api/crypto_operation/mac.h`

**主要方法**:
```c
typedef struct HcfMac {
    HcfObjectBase base;
    
    // 初始化 MAC
    HcfResult (*init)(struct HcfMac *self, HcfKey *key,
        HcfAlgorithmParameter *params);
    
    // 更新数据
    HcfResult (*update)(struct HcfMac *self, HcfBlob *data);
    
    // 完成 MAC
    HcfResult (*doFinal)(struct HcfMac *self, HcfBlob *mac);
    
    // 获取 MAC 长度
    HcfResult (*getMacLength)(struct HcfMac *self, int32_t *length);
} HcfMac;
```

### HcfKeyAgreement - 密钥协商

**位置**: `interfaces/inner_api/crypto_operation/key_agreement.h`

**主要方法**:
```c
typedef struct HcfKeyAgreement {
    HcfObjectBase base;
    
    // 生成共享密钥
    HcfResult (*generateSecret)(struct HcfKeyAgreement *self,
        HcfKey *priKey, HcfKey *pubKey, HcfBlob *secret);
    
    // 获取算法名称
    const char* (*getAlgName)(struct HcfKeyAgreement *self);
} HcfKeyAgreement;
```

### HcfKdf - 密钥派生函数

**位置**: `interfaces/inner_api/crypto_operation/kdf.h`

**主要方法**:
```c
typedef struct HcfKdf {
    HcfObjectBase base;
    
    // 派生密钥
    HcfResult (*generateSecret)(struct HcfKdf *self,
        HcfBlob *password, HcfBlob *salt, HcfBlob *secret);
    
    // 获取算法名称
    const char* (*getAlgName)(struct HcfKdf *self);
} HcfKdf;
```

### HcfRand - 随机数生成器

**位置**: `interfaces/inner_api/crypto_operation/rand.h`

**主要方法**:
```c
typedef struct HcfRand {
    HcfObjectBase base;
    
    // 生成随机数
    HcfResult (*generateRandom)(struct HcfRand *self,
        int32_t length, HcfBlob *random);
    
    // 设置种子
    HcfResult (*setSeed)(struct HcfRand *self, HcfBlob *seed);
    
    // 启用硬件熵
    HcfResult (*enableHardwareEntropy)(struct HcfRand *self);
} HcfRand;
```

## SPI 接口

### HcfMdSpi - 摘要 SPI

**位置**: `frameworks/spi/md_spi.h`

**主要方法**:
```c
typedef struct HcfMdSpi {
    HcfObjectBase base;
    
    // 更新
    HcfResult (*update)(struct HcfMdSpi *self, HcfBlob *data);
    
    // 完成
    HcfResult (*doFinal)(struct HcfMdSpi *self, HcfBlob *result);
    
    // 重置
    HcfResult (*reset)(struct HcfMdSpi *self);
    
    // 获取摘要长度
    int32_t (*getMdLength)(struct HcfMdSpi *self);
} HcfMdSpi;
```

### HcfSignSpi - 签名 SPI

**位置**: `frameworks/spi/signature_spi.h`

**主要方法**:
```c
typedef struct HcfSignSpi {
    HcfObjectBase base;
    
    // 初始化
    HcfResult (*init)(struct HcfSignSpi *self, HcfPriKey *priKey,
        HcfAlgorithmParameter *params);
    
    // 更新
    HcfResult (*update)(struct HcfSignSpi *self, HcfBlob *data);
    
    // 签名
    HcfResult (*sign)(struct HcfSignSpi *self, HcfBlob *signature);
} HcfSignSpi;

typedef struct HcfVerifySpi {
    HcfObjectBase base;
    
    // 初始化
    HcfResult (*init)(struct HcfVerifySpi *self, HcfPubKey *pubKey,
        HcfAlgorithmParameter *params);
    
    // 更新
    HcfResult (*update)(struct HcfVerifySpi *self, HcfBlob *data);
    
    // 验证
    HcfResult (*verify)(struct HcfVerifySpi *self, HcfBlob *signature,
        bool *result);
} HcfVerifySpi;
```

### HcfMacSpi - MAC SPI

**位置**: `frameworks/spi/mac_spi.h`

**主要方法**:
```c
typedef struct HcfMacSpi {
    HcfObjectBase base;
    
    // 初始化
    HcfResult (*init)(struct HcfMacSpi *self, HcfKey *key,
        HcfAlgorithmParameter *params);
    
    // 更新
    HcfResult (*update)(struct HcfMacSpi *self, HcfBlob *data);
    
    // 完成
    HcfResult (*doFinal)(struct HcfMacSpi *self, HcfBlob *mac);
    
    // 获取长度
    int32_t (*getMacLength)(struct HcfMacSpi *self);
} HcfMacSpi;
```

### HcfCipherSpi - 加密 SPI

**位置**: `frameworks/spi/cipher_factory_spi.h`

**主要方法**:
```c
typedef struct HcfCipherSpi {
    HcfObjectBase base;
    
    // 初始化
    HcfResult (*init)(struct HcfCipherSpi *self, HcfCipherMode mode,
        HcfKey *key, HcfAlgorithmParameter *params);
    
    // 更新
    HcfResult (*update)(struct HcfCipherSpi *self, HcfBlob *in,
        HcfBlob *out);
    
    // 完成
    HcfResult (*doFinal)(struct HcfCipherSpi *self, HcfBlob *in,
        HcfBlob *out);
} HcfCipherSpi;
```

### HcfKeyAgreementSpi - 密钥协商 SPI

**位置**: `frameworks/spi/key_agreement_spi.h`

**主要方法**:
```c
typedef struct HcfKeyAgreementSpi {
    HcfObjectBase base;
    
    // 生成共享密钥
    HcfResult (*generateSecret)(struct HcfKeyAgreementSpi *self,
        HcfPriKey *priKey, HcfPubKey *pubKey, HcfBlob *secret);
} HcfKeyAgreementSpi;
```

### HcfKdfSpi - 密钥派生 SPI

**位置**: `frameworks/spi/kdf_spi.h`

**主要方法**:
```c
typedef struct HcfKdfSpi {
    HcfObjectBase base;
    
    // 派生密钥
    HcfResult (*generateSecret)(struct HcfKdfSpi *self,
        HcfBlob *password, HcfBlob *salt, HcfBlob *secret);
} HcfKdfSpi;
```

### HcfRandSpi - 随机数 SPI

**位置**: `frameworks/spi/rand_spi.h`

**主要方法**:
```c
typedef struct HcfRandSpi {
    HcfObjectBase base;
    
    // 生成随机数
    HcfResult (*generateRandom)(struct HcfRandSpi *self,
        int32_t length, HcfBlob *random);
    
    // 设置种子
    HcfResult (*setSeed)(struct HcfRandSpi *self, HcfBlob *seed);
    
    // 启用硬件熵
    HcfResult (*enableHardwareEntropy)(struct HcfRandSpi *self);
} HcfRandSpi;
```

### HcfAsyKeyGeneratorSpi - 非对称密钥生成 SPI

**位置**: `frameworks/spi/asy_key_generator_spi.h`

**主要方法**:
```c
typedef struct HcfAsyKeyGeneratorSpi {
    HcfObjectBase base;
    
    // 生成密钥对
    HcfResult (*generateKeyPair)(struct HcfAsyKeyGeneratorSpi *self,
        HcfAsyKeyParamsSpec *params, HcfKeyPair **keyPair);
    
    // 转换密钥
    HcfResult (*convertKey)(struct HcfAsyKeyGeneratorSpi *self,
        HcfBlob *blob, HcfKey **key);
} HcfAsyKeyGeneratorSpi;
```

## 模块依赖关系

### 依赖方向

```
N-API 绑定层 (frameworks/js/napi/)
    ↓
框架核心层 (frameworks/crypto_operation/, frameworks/key/)
    ↓
SPI 接口层 (frameworks/spi/)
    ↓
插件实现层 (plugin/openssl_plugin/, plugin/mbedtls_plugin/)
    ↓
第三方库 (OpenSSL, MbedTLS)
```

### 稳定性标注

| 接口层次 | 稳定性 | 说明 |
|----------|----------|------|
| **对外 API** (interfaces/kits/) | 稳定 | 对外承诺接口，变更需谨慎 |
| **内部 API** (interfaces/inner_api/) | 半稳定 | 框架内部使用，可调整 |
| **SPI 接口** (frameworks/spi/) | 不稳定 | 插件接口，可随底层库变更 |
| **框架实现** (frameworks/*/) | 不稳定 | 内部实现，可重构 |

## 相关跳转

- **对外 API**: [04_External_API.md](04_External_API.md)
- **目录结构**: [02_Directory_Structure.md](02_Directory_Structure.md)
- **架构设计**: [03_Architecture.md](03_Architecture.md)

## 更新记录

- **2026-02-06**: 创建文档，基于内部 API 分析生成

## TODO

- [ ] 补充算法参数接口详细说明
- [ ] 补充 SPI 接口的生命周期管理
- [ ] 绘制完整的调用链图
