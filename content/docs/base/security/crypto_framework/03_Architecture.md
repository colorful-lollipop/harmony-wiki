# 架构说明

## 目的

本文档详细说明 crypto_framework 的架构设计，包括分层架构、组件关系、数据流转、线程模型和关键时序。

## 适用范围

- **目标读者**: 架构师、高级开发者、安全审计人员
- **阅读时长**: 30 分钟

## 分层架构

crypto_framework 采用经典的分层架构设计，分为四层：

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (Application Layer)               │
│  - JS/ArkTS 应用 (通过 N-API/ANI)                      │
│  - Native C 应用 (通过 OH_Crypto_* API)               │
│  - Cangjie 应用 (通过 CJ FFI)                          │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│                   API 绑定层 (Binding Layer)             │
│  - N-API 绑定 (frameworks/js/napi/)                   │
│  - ANI 绑定 (frameworks/js/ani/)                     │
│  - JSI 绑定 (frameworks/js/jsi/)                      │
│  - CJ FFI 绑定 (frameworks/cj/)                      │
│  - Native API (frameworks/native/)                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│                  框架核心层 (Framework Layer)             │
│  - 密码操作封装 (frameworks/crypto_operation/)            │
│  - 密钥材料管理 (frameworks/key/)                      │
│  - SPI 接口定义 (frameworks/spi/)                     │
│  - 算法参数 (frameworks/algorithm_parameter/)            │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│                   插件层 (Plugin Layer)                  │
│  - OpenSSL Plugin (plugin/openssl_plugin/)                 │
│  - MbedTLS Plugin (plugin/mbedtls_plugin/)                │
└──────────────────────────────────────────────────────────────┘
```

### 各层职责

| 层次 | 职责 | 关键文件 |
|------|------|----------|
| **应用层** | 使用加密功能，处理业务逻辑 | 应用代码 |
| **API 绑定层** | 将框架接口封装为各语言可调用的接口 | `frameworks/js/`, `frameworks/native/`, `frameworks/cj/` |
| **框架核心层** | 实现密码操作的统一逻辑，管理密钥材料 | `frameworks/crypto_operation/`, `frameworks/key/` |
| **插件层** | 实现具体的密码算法，适配第三方库 | `plugin/openssl_plugin/`, `plugin/mbedtls_plugin/` |

## 组件图

### 核心组件关系

```mermaid
graph TB
    subgraph "应用层"
        JSApp[JS/ArkTS 应用]
        NativeApp[Native C 应用]
        CJApp[Cangjie 应用]
    end

    subgraph "API 绑定层"
        NAPI[N-API 绑定<br/>libcryptoframework_napi.so]
        ANI[ANI 绑定<br/>crypto_framework_ani]
        JSI[JSI 绑定<br/>libcryptoframework_jsi.a]
        NativeAPI[Native API<br/>libohcrypto.so]
        CJ[CJ FFI<br/>libcj_cryptoframework_ffi.so]
    end

    subgraph "框架核心层"
        FrameworkLib[框架库<br/>libcrypto_framework_lib.so]
        CipherSp[Cipher SPI]
        SignSp[Sign SPI]
        MDSp[Md SPI]
        RandSp[Rand SPI]
    end

    subgraph "插件层"
        OpenSSLPlugin[OpenSSL Plugin<br/>libcrypto_openssl_plugin_lib.so]
        MbedTLSPlugin[MbedTLS Plugin<br/>libcrypto_mbedtls_plugin_lib.a]
    end

    subgraph "第三方库"
        OpenSSL[OpenSSL<br/>libcrypto.so]
        MbedTLS[MbedTLS<br/>libmbedtls.so]
    end

    JSApp --> NAPI
    NativeApp --> NativeAPI
    CJApp --> CJ

    NAPI --> FrameworkLib
    NativeAPI --> FrameworkLib
    CJ --> FrameworkLib

    FrameworkLib --> CipherSp
    FrameworkLib --> SignSp
    FrameworkLib --> MDSp
    FrameworkLib --> RandSp

    CipherSp --> OpenSSLPlugin
    SignSp --> OpenSSLPlugin
    MDSp --> OpenSSLPlugin
    RandSp --> OpenSSLPlugin

    OpenSSLPlugin --> OpenSSL
    MbedTLSPlugin --> MbedTLS
```

### 模块依赖关系

```
crypto_framework_component (group)
├── crypto_framework_lib (shared_library)
│   ├── crypto_plugin_common (static_library)
│   └── crypto_openssl_plugin_lib (shared_library)
│       └── crypto_plugin_common
├── ohcrypto (shared_library)
│   └── crypto_framework_lib
├── cryptoframework_napi (shared_library)
│   └── crypto_framework_lib
├── cj_cryptoframework_ffi (shared_library)
│   └── crypto_framework_lib
└── crypto_openssl_plugin_lib (shared_library)
    └── crypto_plugin_common
```

## 数据流

### 对称加密流程

```mermaid
sequenceDiagram
    participant App as 应用程序
    participant NAPI as N-API 绑定
    participant Framework as 框架层
    participant Plugin as OpenSSL 插件
    participant OpenSSL as OpenSSL 库

    App->>NAPI: 1. 创建对称密钥生成器<br/>createSymKeyGenerator("AES256")
    NAPI->>Framework: 2. 创建 HcfSymKeyGenerator
    Framework->>Plugin: 3. 调用 HcfSymKeyGeneratorSpi
    Plugin->>OpenSSL: 4. OpenSSL 生成密钥
    OpenSSL-->>Plugin: 5. 返回密钥
    Plugin-->>Framework: 6. 返回 HcfSymKey
    Framework-->>NAPI: 7. 返回 SymKey 对象
    NAPI-->>App: 8. 返回 JS SymKey 实例

    App->>NAPI: 9. 创建 Cipher<br/>createCipher("AES256", "GCM")
    NAPI->>Framework: 10. 创建 HcfCipher
    Framework->>Plugin: 11. 调用 HcfCipherSpiCreate
    Plugin->>OpenSSL: 12. OpenSSL 创建 Cipher 上下文
    OpenSSL-->>Plugin: 13. 返回上下文
    Plugin-->>Framework: 14. 返回 HcfCipher
    Framework-->>NAPI: 15. 返回 Cipher 对象
    NAPI-->>App: 16. 返回 JS Cipher 实例

    App->>NAPI: 17. 初始化 Cipher<br/>cipher.init(encryptMode, key, params)
    NAPI->>Framework: 18. 调用 HcfCipherInit
    Framework->>Plugin: 19. 调用 HcfCipherSpiInit
    Plugin->>OpenSSL: 20. OpenSSL 初始化加密
    OpenSSL-->>Plugin: 21. 返回成功
    Plugin-->>Framework: 22. 返回 HCF_SUCCESS
    Framework-->>NAPI: 23. 返回成功
    NAPI-->>App: 24. 返回 Promise<undefined>

    App->>NAPI: 25. 更新数据<br/>cipher.update(data)
    NAPI->>Framework: 26. 调用 HcfCipherUpdate
    Framework->>Plugin: 27. 调用 HcfCipherSpiUpdate
    Plugin->>OpenSSL: 28. OpenSSL 加密数据
    OpenSSL-->>Plugin: 29. 返回加密数据
    Plugin-->>Framework: 30. 返回加密结果
    Framework-->>NAPI: 31. 返回 HcfBlob
    NAPI-->>App: 32. 返回 Promise<ArrayBuffer>

    App->>NAPI: 33. 完成加密<br/>cipher.doFinal()
    NAPI->>Framework: 34. 调用 HcfCipherDoFinal
    Framework->>Plugin: 35. 调用 HcfCipherSpiDoFinal
    Plugin->>OpenSSL: 36. OpenSSL 完成加密
    OpenSSL-->>Plugin: 37. 返回最终数据和认证标签
    Plugin-->>Framework: 38. 返回最终结果
    Framework-->>NAPI: 39. 返回 HcfBlob
    NAPI-->>App: 40. 返回 Promise<ArrayBuffer>
```

### 签名验签流程

```mermaid
sequenceDiagram
    participant App as 应用程序
    participant NAPI as N-API 绑定
    participant Framework as 框架层
    participant Plugin as OpenSSL 插件
    participant OpenSSL as OpenSSL 库

    Note over App, OpenSSL: 签名阶段

    App->>NAPI: 1. 创建密钥生成器<br/>createAsyKeyGenerator("ECC256")
    NAPI->>Framework: 2. 创建 HcfAsyKeyGenerator
    Framework->>Plugin: 3. 生成 ECC 密钥对
    Plugin->>OpenSSL: 4. OpenSSL 生成密钥对
    OpenSSL-->>Plugin: 5. 返回公钥和私钥
    Plugin-->>Framework: 6. 返回 HcfKeyPair
    Framework-->>NAPI: 7. 返回 KeyPair 对象
    NAPI-->>App: 8. 返回 JS KeyPair 实例

    App->>NAPI: 9. 创建 Sign<br/>createSign("ECC256")
    NAPI->>Framework: 10. 创建 HcfSign
    Framework->>Plugin: 11. 调用 HcfSignSpiCreate
    Plugin->>OpenSSL: 12. OpenSSL 创建签名上下文
    OpenSSL-->>Plugin: 13. 返回上下文
    Plugin-->>Framework: 14. 返回 HcfSign
    Framework-->>NAPI: 15. 返回 Sign 对象
    NAPI-->>App: 16. 返回 JS Sign 实例

    App->>NAPI: 17. 初始化签名<br/>sign.init(priKey)
    NAPI->>Framework: 18. 调用 HcfSignInit
    Framework->>Plugin: 19. 调用 HcfSignSpiInit
    Plugin->>OpenSSL: 20. OpenSSL 初始化签名
    OpenSSL-->>Plugin: 21. 返回成功
    Plugin-->>Framework: 22. 返回成功
    Framework-->>NAPI: 23. 返回成功
    NAPI-->>App: 24. 返回 Promise<undefined>

    App->>NAPI: 25. 更新数据<br/>sign.update(data)
    NAPI->>Framework: 26. 调用 HcfSignUpdate
    Framework->>Plugin: 27. 调用 HcfSignSpiUpdate
    Plugin->>OpenSSL: 28. OpenSSL 更新签名
    OpenSSL-->>Plugin: 29. 返回成功
    Plugin-->>Framework: 30. 返回成功
    Framework-->>NAPI: 31. 返回成功
    NAPI-->>App: 32. 返回 Promise<undefined>

    App->>NAPI: 33. 生成签名<br/>sign.sign()
    NAPI->>Framework: 34. 调用 HcfSignSign
    Framework->>Plugin: 35. 调用 HcfSignSpiSign
    Plugin->>OpenSSL: 36. OpenSSL 生成签名
    OpenSSL-->>Plugin: 37. 返回签名数据
    Plugin-->>Framework: 38. 返回签名 HcfBlob
    Framework-->>NAPI: 39. 返回签名结果
    NAPI-->>App: 40. 返回 Promise<ArrayBuffer>

    Note over App, OpenSSL: 验签阶段

    App->>NAPI: 41. 创建 Verify<br/>createVerify("ECC256")
    NAPI->>Framework: 42. 创建 HcfVerify
    Framework->>Plugin: 43. 调用 HcfVerifySpiCreate
    Plugin->>OpenSSL: 44. OpenSSL 创建验签上下文
    OpenSSL-->>Plugin: 45. 返回上下文
    Plugin-->>Framework: 46. 返回 HcfVerify
    Framework-->>NAPI: 47. 返回 Verify 对象
    NAPI-->>App: 48. 返回 JS Verify 实例

    App->>NAPI: 49. 初始化验签<br/>verify.init(pubKey)
    NAPI->>Framework: 50. 调用 HcfVerifyInit
    Framework->>Plugin: 51. 调用 HcfVerifySpiInit
    Plugin->>OpenSSL: 52. OpenSSL 初始化验签
    OpenSSL-->>Plugin: 53. 返回成功
    Plugin-->>Framework: 54. 返回成功
    Framework-->>NAPI: 55. 返回成功
    NAPI-->>App: 56. 返回 Promise<undefined>

    App->>NAPI: 57. 更新数据<br/>verify.update(data)
    NAPI->>Framework: 58. 调用 HcfVerifyUpdate
    Framework->>Plugin: 59. 调用 HcfVerifySpiUpdate
    Plugin->>OpenSSL: 60. OpenSSL 更新验签
    OpenSSL-->>Plugin: 61. 返回成功
    Plugin-->>Framework: 62. 返回成功
    Framework-->>NAPI: 63. 返回成功
    NAPI-->>App: 64. 返回 Promise<undefined>

    App->>NAPI: 65. 验证签名<br/>verify.verify(signature)
    NAPI->>Framework: 66. 调用 HcfVerify
    Framework->>Plugin: 67. 调用 HcfVerifySpiVerify
    Plugin->>OpenSSL: 68. OpenSSL 验证签名
    OpenSSL-->>Plugin: 69. 返回验证结果
    Plugin-->>Framework: 70. 返回结果
    Framework-->>NAPI: 71. 返回布尔值
    NAPI-->>App: 72. 返回 Promise<boolean>
```

### 密钥协商流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Server as 服务端
    participant NAPI as N-API 绑定
    participant Framework as 框架层
    participant Plugin as OpenSSL 插件
    participant OpenSSL as OpenSSL 库

    Note over Client, Server: 阶段 1: 双方生成密钥对

    Client->>NAPI: 1. 创建密钥生成器<br/>createAsyKeyGenerator("ECDH")
    NAPI->>Framework: 2. 创建 HcfAsyKeyGenerator
    Framework->>Plugin: 3. 生成 ECDH 密钥对
    Plugin->>OpenSSL: 4. OpenSSL 生成密钥对
    OpenSSL-->>Plugin: 5. 返回公钥和私钥
    Plugin-->>Framework: 6. 返回 HcfKeyPair
    Framework-->>NAPI: 7. 返回 KeyPair 对象
    NAPI-->>Client: 8. 返回 JS KeyPair 实例

    Server->>NAPI: 9. 创建密钥生成器<br/>createAsyKeyGenerator("ECDH")
    NAPI->>Framework: 10. 创建 HcfAsyKeyGenerator
    Framework->>Plugin: 11. 生成 ECDH 密钥对
    Plugin->>OpenSSL: 12. OpenSSL 生成密钥对
    OpenSSL-->>Plugin: 13. 返回公钥和私钥
    Plugin-->>Framework: 14. 返回 HcfKeyPair
    Framework-->>NAPI: 15. 返回 KeyPair 对象
    NAPI-->>Server: 16. 返回 JS KeyPair 实例

    Note over Client, Server: 阶段 2: 交换公钥

    Client->>Server: 17. 发送客户端公钥
    Server-->>Client: 18. 发送服务端公钥

    Note over Client, Server: 阶段 3: 计算共享密钥

    Client->>NAPI: 19. 创建密钥协商<br/>createKeyAgreement("ECDH")
    NAPI->>Framework: 20. 创建 HcfKeyAgreement
    Framework->>Plugin: 21. 调用 HcfKeyAgreementSpiCreate
    Plugin->>OpenSSL: 22. OpenSSL 创建密钥协商上下文
    OpenSSL-->>Plugin: 23. 返回上下文
    Plugin-->>Framework: 24. 返回 HcfKeyAgreement
    Framework-->>NAPI: 25. 返回 KeyAgreement 对象
    NAPI-->>Client: 26. 返回 JS KeyAgreement 实例

    Client->>NAPI: 27. 计算共享密钥<br/>ka.generateSecret(priKey, pubKey)
    NAPI->>Framework: 28. 调用 HcfKeyAgreementGenerateSecret
    Framework->>Plugin: 29. 调用 HcfKeyAgreementSpiGenerateSecret
    Plugin->>OpenSSL: 30. OpenSSL 计算共享密钥
    OpenSSL-->>Plugin: 31. 返回共享密钥
    Plugin-->>Framework: 32. 返回 HcfBlob
    Framework-->>NAPI: 33. 返回共享密钥
    NAPI-->>Client: 34. 返回 Promise<ArrayBuffer>

    Server->>NAPI: 35. 创建密钥协商<br/>createKeyAgreement("ECDH")
    NAPI->>Framework: 36. 创建 HcfKeyAgreement
    Framework->>Plugin: 37. 调用 HcfKeyAgreementSpiCreate
    Plugin->>OpenSSL: 38. OpenSSL 创建密钥协商上下文
    OpenSSL-->>Plugin: 39. 返回上下文
    Plugin-->>Framework: 40. 返回 HcfKeyAgreement
    Framework-->>NAPI: 41. 返回 KeyAgreement 对象
    NAPI-->>Server: 42. 返回 JS KeyAgreement 实例

    Server->>NAPI: 43. 计算共享密钥<br/>ka.generateSecret(priKey, pubKey)
    NAPI->>Framework: 44. 调用 HcfKeyAgreementGenerateSecret
    Framework->>Plugin: 45. 调用 HcfKeyAgreementSpiGenerateSecret
    Plugin->>OpenSSL: 46. OpenSSL 计算共享密钥
    OpenSSL-->>Plugin: 47. 返回共享密钥
    Plugin-->>Framework: 48. 返回 HcfBlob
    Framework-->>NAPI: 49. 返回共享密钥
    NAPI-->>Server: 50. 返回 Promise<ArrayBuffer>

    Note over Client, Server: 阶段 4: 使用共享密钥

    Client->>Client: 51. 使用共享密钥加密通信
    Server->>Server: 52. 使用共享密钥解密通信
```

## SPI 机制

### SPI 架构

SPI (Service Provider Interface) 是 crypto_framework 的核心扩展机制，允许插入不同的密码算法库实现。

### SPI 接口层次

```
框架层定义 SPI 接口 (frameworks/spi/)
    ↑
插件层实现 SPI 接口 (plugin/)
    ↑
第三方库提供算法实现 (OpenSSL, MbedTLS)
```

### 关键 SPI 接口

| SPI 接口 | 方法 | 说明 | 插件实现位置 |
|----------|------|--------------|
| **HcfMdSpi** | Update, DoFinal, GetMdLength | `plugin/openssl_plugin/crypto_operation/md/` |
| **HcfSignSpi** | Init, Update, Sign | `plugin/openssl_plugin/crypto_operation/signature/` |
| **HcfMacSpi** | Init, Update, DoFinal | `plugin/openssl_plugin/crypto_operation/hmac/` |
| **HcfCipherSpi** | Init, Update, DoFinal | `plugin/openssl_plugin/crypto_operation/cipher/` |
| **HcfKeyAgreementSpi** | GenerateSecret | `plugin/openssl_plugin/crypto_operation/key_agreement/` |
| **HcfKdfSpi** | GenerateSecret | `plugin/openssl_plugin/crypto_operation/kdf/` |
| **HcfRandSpi** | GenerateRandom | `plugin/openssl_plugin/crypto_operation/rand/` |
| **HcfAsyKeyGeneratorSpi** | GenerateKeyPair, ConvertKey | `plugin/openssl_plugin/key/asy_key_generator/` |
| **HcfSymKeyFactorySpi** | GenerateSymKey | `plugin/openssl_plugin/key/sym_key_generator/` |

**证据位置**: `frameworks/spi/` 目录

### 插件加载流程

```mermaid
flowchart TD
    Start[系统启动] --> CheckSystem[检查系统类型<br/>os_level]
    CheckSystem --> Standard{标准系统?}
    Standard -->|Yes| LoadOpenSSL[加载 OpenSSL Plugin]
    Standard -->|No| LoadMbedTLS[加载 MbedTLS Plugin]
    LoadOpenSSL --> RegisterSPI[注册 SPI 实现]
    LoadMbedTLS --> RegisterSPI
    RegisterSPI --> Ready[框架就绪]

    Ready --> AppCall[应用调用 API]
    AppCall --> GetSPI[获取 SPI 实现]
    GetSPI --> CallPlugin[调用插件方法]
    CallPlugin --> OpenSSL[OpenSSL 执行<br/>或 MbedTLS 执行]
    OpenSSL --> ReturnResult[返回结果]
    ReturnResult --> AppCall
```

## 插件系统

### OpenSSL Plugin 结构

```
openssl_plugin/
├── common/                 # 通用工具和适配器
│   ├── inc/
│   │   ├── openssl_adapter.h      # OpenSSL 适配器
│   │   ├── ecc_openssl_common.h   # ECC 通用处理
│   │   └── ...
│   └── src/
├── crypto_operation/        # 密码操作实现
│   ├── cipher/            # 加密/解密
│   │   ├── inc/
│   │   │   ├── cipher_aes_openssl.h
│   │   │   └── cipher_sm4_openssl.h
│   │   └── src/
│   │       ├── cipher_aes_openssl.c
│   │       └── cipher_sm4_openssl.c
│   ├── hmac/              # HMAC/CMAC
│   ├── md/                # 摘要
│   ├── kdf/               # 密钥派生
│   ├── key_agreement/     # 密钥协商
│   ├── signature/          # 签名验签
│   └── rand/              # 随机数
└── key/                  # 密钥生成
    ├── asy_key_generator/ # 非对称密钥
    └── sym_key_generator/ # 对称密钥
```

### OpenSSL Adapter 模式

crypto_framework 通过 OpenSSL Adapter 层屏蔽底层 API 差异：

```c
// 伪代码示例
struct OpensslAdapter {
    // 将框架参数转换为 OpenSSL 参数
    HcfResult (*ConvertParams)(HcfAsyKeyParamsSpec*, EVP_PKEY_CTX*);
    
    // 调用 OpenSSL API
    HcfResult (*PerformOperation)(EVP_PKEY_CTX*, HcfBlob*, HcfBlob*);
    
    // 转换结果为框架格式
    HcfResult (*ConvertResult)(HcfBlob*, HcfResult*);
};
```

**证据位置**: `plugin/openssl_plugin/common/inc/openssl_adapter.h`

## 线程模型

### 同步操作

- **特点**: 阻塞当前线程，直接返回结果
- **适用**: 简单操作、小数据量
- **方法名**: `methodNameSync` (如 `digestSync`, `signSync`)

### 异步操作

- **特点**: 不阻塞当前线程，通过 Promise 或 Callback 返回结果
- **适用**: 耗时操作、大数据量
- **方法名**: `methodName` (如 `digest`, `sign`)

### N-API 异步模式

crypto_framework 使用 N-API 的异步工作队列实现异步操作：

```mermaid
flowchart LR
    App[JS 应用] -->|调用异步方法| NAPI[N-API 绑定]
    NAPI -->|创建 napi_async_work| AS[Async Resource]
    ASY -->|napi_queue_async_work| WL[工作线程池]
    WL -->|执行 Execute| EXE[Execute 函数]
    EXE -->|返回结果| NAPI
    NAPI -->|napi_async_complete| CB[Complete 回调]
    CB -->|调用 JS 回调| RES[Promise resolve/reject]
    RES --> App
```

**关键函数**:
- `napi_create_async_work`: 创建异步工作
- `napi_queue_async_work`: 加入工作队列
- `Execute`: 执行函数（在工作线程）
- `Complete`: 完成回调（在主线程）

**证据位置**:
- 各 napi 源文件中的 `New*AsyncWork` 函数
- `frameworks/js/napi/crypto/src/napi_utils.cpp` (工具函数)

## 资源生命周期

### 对象生命周期

```mermaid
stateDiagram-v2
    [*] --> Created: 创建对象<br/>(N-API create)
    Created --> Initialized: 初始化<br/>(init 方法)
    Initialized --> Ready: 就绪<br/>(可执行操作)
    Ready --> Processing: 处理中<br/>(update/doFinal 等方法)
    Processing --> Ready: 操作完成
    Ready --> Destroyed: 销毁对象<br/>(GC 或手动 destroy)
    Initialized --> Destroyed: 未使用即销毁
    Destroyed --> [*]
```

### 内存管理

**框架层内存管理**:
- 对象分配：使用 `HcfMalloc` 包装
- 对象释放：通过 `HcfDestroy` 接口
- 敏感数据清除：`clearMem` 方法

**N-API 内存管理**:
- ArrayBuffer：由 V8 引擎管理
- 外部内存：使用 `napi_create_external_arraybuffer`

**证据位置**:
- `interfaces/inner_api/common/object_base.h` (HcfObjectBase)
- `common/inc/memory.h` (HcfMalloc/HcfFree)

## 错误传播机制

### 错误码传播

```mermaid
flowchart LR
    Plugin[插件层<br/>OpenSSL 错误] -->|HCF_ERR_CRYPTO_OPERATION| Framework[框架层<br/>HcfResult]
    Framework -->|转换错误码| NAPI[N-API 绑定层<br/>JS_ERR_*]
    NAPI -->|生成 Error 对象| JS[JS 应用层<br/>Error 异常]
```

### 错误码映射

| 插件错误 | 框架错误 | N-API 错误 | JS 错误 |
|----------|----------|-------------|----------|
| OpenSSL `ERR_*` | `HCF_ERR_CRYPTO_OPERATION` | `JS_ERR_CRYPTO_OPERATION` | `Error(17630001)` |
| 参数无效 | `HCF_INVALID_PARAMS` | `JS_ERR_INVALID_PARAMS` | `Error(401)` |
| 内存不足 | `HCF_ERR_MALLOC` | `JS_ERR_OUT_OF_MEMORY` | `Error(17620001)` |

**证据位置**:
- `interfaces/inner_api/common/result.h` (HcfResult)
- `frameworks/js/napi/crypto/inc/napi_utils.h` (JS_ERR_*)

## 相关跳转

- **目录结构**: [02_Directory_Structure.md](02_Directory_Structure.md)
- **对外 API**: [04_External_API.md](04_External_API.md)
- **内部 API**: [05_Internal_API.md](05_Internal_API.md)

## 更新记录

- **2026-02-06**: 创建文档，基于架构分析生成

## TODO

- [ ] 补充异步操作的详细流程图
- [ ] 添加插件动态加载机制的说明
- [ ] 补充对象引用计数机制
