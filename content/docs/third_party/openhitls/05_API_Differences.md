# API/接口差异

## 核心结论

> **openHiTLS 在 OpenHarmony 中无 API 差异。**

openHiTLS 以**原生形式**集成到 OpenHarmony，未添加、修改或删除任何 API。所有接口与上游版本完全一致。

---

## 1. API 状态

### 1.1 无 OH 特定 API

| 检查项 | 结果 |
|-------|------|
| 新增 OH 专用 API | 无 |
| 修改现有 API 行为 | 无 |
| 废弃 API | 无 |
| API 签名变更 | 无 |

### 1.2 公共头文件

所有公共头文件与上游完全一致：

```
include/
├── auth/
│   └── *.h           # 认证相关 API
├── bsl/
│   ├── bsl_err.h     # 错误处理
│   ├── bsl_sal.h     # OS 适配层
│   └── ...           # 其他 BSL API
├── crypto/
│   ├── crypt_eal_*.h # 加密抽象层
│   ├── crypt_types.h # 类型定义
│   └── ...           # 其他 Crypto API
├── pki/
│   ├── hitls_pki_*.h # PKI API
│   └── ...
└── tls/
    ├── hitls.h       # TLS 核心 API
    ├── hitls_*.h     # 其他 TLS API
    └── ...
```

---

## 2. 行为一致性

### 2.1 初始化行为

```c
// 标准初始化流程（与上游一致）
BSL_GLOBAL_Init();
CRYPT_EAL_Init(CRYPT_EAL_INIT_CPU | CRYPT_EAL_INIT_PROVIDER);
```

### 2.2 算法支持

OpenHarmony 构建配置启用的算法与上游默认配置一致：

| 算法类别 | 上游默认 | OH 构建 |
|---------|---------|---------|
| AES | ✅ | ✅ |
| SM4 | ✅ | ✅ |
| SM3 | ✅ | ✅ |
| SM2 | ✅ | ✅ |
| RSA | ✅ | ✅ |
| ECC | ✅ | ✅ |
| SHA1/2/3 | ✅ | ✅ |
| ML-DSA | ✅ | ✅ |
| ML-KEM | ✅ | ✅ |
| TLS 1.3 | ✅ | ✅ |
| TLCP | ✅ | ✅ |

### 2.3 错误码

所有错误码与上游一致：

```c
// 来自 bsl_err.h
#define BSL_SUCCESS 0
#define BSL_INTERNAL_ERROR (-0x0001)
#define BSL_INVALID_ARG (-0x0002)
// ... 更多

// 来自 crypt_errno.h
#define CRYPT_SUCCESS 0
#define CRYPT_INVALID_ARG (-0x0100)
#define CRYPT_MEM_ALLOC_FAIL (-0x0101)
// ... 更多
```

---

## 3. OH 特有的行为差异

虽然 API 完全一致，但以下行为由构建配置决定，可能与上游默认不同：

### 3.1 平台优化选择

| 平台 | 上游默认 | OH 构建 |
|-----|---------|---------|
| ARM64 | 自动检测 | 强制 ARMv8 汇编优化 |
| x86_64 | 自动检测 | 强制 x86_64 汇编优化 |
| 其他 | 纯 C | 纯 C |

**说明**：这是性能优化选择，非 API 变更。

### 3.2 库类型

| 项目 | 上游 | OH |
|-----|------|-----|
| 默认库类型 | 静态/动态可选 | 动态库 |
| NDK 暴露 | 取决于配置 | 明确标记 llndk/ndk |

**说明**：库打包方式差异，API 调用方式不变。

---

## 4. 与 OpenSSL 的 API 差异

开发者可能关心 openHiTLS 与 OpenSSL 的 API 差异：

### 4.1 不兼容说明

openHiTLS 是**独立实现**，与 OpenSSL **不 API 兼容**：

| 方面 | OpenSSL | openHiTLS |
|-----|---------|-----------|
| 前缀 | `OPENSSL_`, `SSL_`, `EVP_` | `HITLS_`, `CRYPT_`, `BSL_` |
| 初始化 | `OPENSSL_init_ssl()` | `BSL_GLOBAL_Init()` |
| 错误处理 | `ERR_get_error()` | `BSL_ERR_GetLastError()` |
| TLS 连接 | `SSL_new()`, `SSL_connect()` | `HITLS_New()`, `HITLS_Connect()` |

### 4.2 迁移指南

从 OpenSSL 迁移到 openHiTLS 需要重写代码，而非简单替换。

**示例对比**：

```c
// OpenSSL
SSL_CTX *ctx = SSL_CTX_new(TLS_client_method());
SSL *ssl = SSL_new(ctx);
SSL_connect(ssl);

// openHiTLS
HITLS_Config *config = HITLS_CFG_NewTLSConfig();
HITLS_Ctx *ctx = HITLS_New(config);
HITLS_Connect(ctx);
```

---

## 5. API 版本兼容性

### 5.1 当前版本

- **上游版本**: 0.2.1
- **OH 组件版本**: 4.0
- **API 版本**: 与上游 0.2.1 一致

### 5.2 NDK 兼容性

由于标记为 `llndk/ndk`，API 承诺向后兼容：

| 保证 | 说明 |
|-----|------|
| ABI 兼容 | 版本升级保持二进制兼容 |
| API 兼容 | 不删除或修改现有 API |
| 行为兼容 | 核心行为保持一致 |

---

## 6. 使用参考

### 6.1 官方文档

- openHiTLS 官方文档: https://openhitls.net
- 文档目录: `docs/` (英文/中文)

### 6.2 示例代码

**curl 适配层** (`third_party/curl/lib/vtls/openhitls.c`) 是完整的 API 使用示例。

**简单示例**：

```c
#include "hitls.h"
#include "hitls_cert.h"
#include "bsl_sal.h"
#include "crypt_eal_init.h"

// 1. 初始化
void init() {
    BSL_GLOBAL_Init();
    CRYPT_EAL_Init(CRYPT_EAL_INIT_CPU | CRYPT_EAL_INIT_PROVIDER);
    CRYPT_EAL_RandInit(CRYPT_RAND_SHA256, NULL, NULL, NULL, 0);
    HITLS_CertMethodInit();
    HITLS_CryptMethodInit();
}

// 2. 创建 TLS 配置
HITLS_Config *create_config() {
    HITLS_Config *config = HITLS_CFG_NewTLSConfig();
    HITLS_CFG_SetCipherSuites(config, "TLS_AES_256_GCM_SHA384:");
    return config;
}

// 3. 建立连接
int connect_tls(const char *host) {
    HITLS_Config *config = create_config();
    HITLS_Ctx *ctx = HITLS_New(config);
    
    // 设置 UIO (底层 IO)
    BSL_UIO *uio = BSL_UIO_New(BSL_UIO_TcpMethod());
    BSL_UIO_Ctrl(uio, BSL_UIO_SET_FD, sizeof(int), &socket_fd);
    HITLS_SetUio(ctx, uio);
    
    // 连接
    int ret = HITLS_Connect(ctx);
    return ret;
}
```

---

## 7. 总结

| 问题 | 答案 |
|-----|------|
| 有 OH 专用 API 吗？ | 无 |
| API 与上游一致吗？ | 完全一致 |
| 与 OpenSSL 兼容吗？ | 不兼容，独立实现 |
| NDK 可用吗？ | 是，标记 llndk/ndk |
| API 稳定吗？ | 是，承诺向后兼容 |

**结论**：开发者可参考 openHiTLS 官方文档，无需关注 OH 特定差异。
