# API/接口差异分析

## 概述

mbedtls 库在 OpenHarmony 中的 API **与上游版本保持高度一致**，主要差异集中在以下方面：

1. **新增 OH 特有接口**: Port 适配层提供的简化 API
2. **构建配置差异**: OH 特定编译选项
3. **PSA API 原生集成**: 与上游一致的 PSA 加密 API

## 新增 OH 特有接口

### TLS 客户端简化接口

Port 适配层提供了简化的 TLS 客户端接口，降低使用门槛：

| 接口 | 文件 | 功能描述 |
|------|------|----------|
| `MbedtlsClientInit()` | `port/src/tls_client.c` | 初始化 TLS 会话 |
| `MbedtlsClientClose()` | `port/src/tls_client.c` | 关闭 TLS 连接 |
| `MbedtlsClientContext()` | `port/src/tls_client.c` | 配置 TLS 上下文 |
| `MbedtlsClientConnect()` | `port/src/tls_client.c` | 建立 TLS 连接 |
| `MbedtlsClientRead()` | `port/src/tls_client.c` | 读取加密数据 |
| `MbedtlsClientWrite()` | `port/src/tls_client.c` | 写入加密数据 |

#### 接口定义

```c
// port/include/tls_client.h

int MbedtlsClientInit(MbedTLSSession *session, void *entropy, size_t entropyLen);
int MbedtlsClientClose(MbedTLSSession *session);
int MbedtlsClientContext(MbedTLSSession *session);
int MbedtlsClientConnect(MbedTLSSession *session);
int MbedtlsClientRead(MbedTLSSession *session, unsigned char *buf, size_t len);
int MbedtlsClientWrite(MbedTLSSession *session, const unsigned char *buf, size_t len);
```

#### 数据结构

```c
typedef struct MbedTLSSession {
    char *host;
    char *port;
    unsigned char *buffer;
    size_t buffer_len;
    mbedtls_ssl_context ssl;
    mbedtls_ssl_config conf;
    mbedtls_entropy_context entropy;
    mbedtls_ctr_drbg_context ctr_drbg;
    mbedtls_net_context server_fd;
    mbedtls_x509_crt cacert;
} MbedTLSSession;
```

### 日志接口

#### 接口定义

```c
// port/include/mbedtls_log.h

#define LOGD(fmt, ...)  // 调试日志
#define LOGI(fmt, ...)  // 信息日志
#define LOGW(fmt, ...)  // 警告日志
#define LOGE(fmt, ...)  // 错误日志
```

#### 使用示例

```c
#include "mbedtls_log.h"

// 调试日志（仅 OHOS_DEBUG 时启用）
LOGD("TLS handshake completed");
LOGD("Connection to %s:%s established", host, port);

// 错误日志
LOGE("TLS handshake failed: -0x%x", -ret);
```

## 与上游 API 的兼容性

### 100% 兼容的上游 API

以下 API 与上游 mbedtls 完全一致：

| 类别 | API 列表 |
|------|----------|
| **TLS/SSL** | `mbedtls_ssl_init()`, `mbedtls_ssl_setup()`, `mbedtls_ssl_handshake()`, `mbedtls_ssl_write()`, `mbedtls_ssl_read()`, `mbedtls_ssl_close_notify()` |
| **X.509 证书** | `mbedtls_x509_crt_init()`, `mbedtls_x509_crt_parse()`, `mbedtls_x509_crt_verify()` |
| **加密操作** | `mbedtls_cipher_init()`, `mbedtls_cipher_setup()`, `mbedtls_cipher_encrypt()`, `mbedtls_cipher_decrypt()` |
| **哈希** | `mbedtls_md_init()`, `mbedtls_md_setup()`, `mbedtls_md_update()`, `mbedtls_md_finish()` |
| **PSA API** | `psa_crypto_init()`, `psa_generate_key()`, `psa_sign_hash()`, `psa_verify_hash()` |

### 配置宏差异

#### OH 特有配置

```c
// 在 OH 构建中使用
#define MBEDTLS_CONFIG_FILE <../port/config/config_liteos_m.h>
// 或
#define MBEDTLS_CONFIG_FILE <../port/config/config_liteos_a.h>
```

#### 上游标准配置

```c
// 在上游标准构建中使用
#include "mbedtls/config.h"
```

### 功能启用差异

| 功能 | OH 配置 | 上游配置 |
|------|---------|----------|
| **TLS 1.2** | 默认启用 | 可配置 |
| **TLS 1.3** | 默认启用 | 可配置 |
| **SSL 服务器** | 可选 (`mbedtls_enable_ssl_srv`) | 可配置 |
| **PSA API** | 默认启用 | 可配置 |
| **特定算法** | 根据内核类型启用 | 完全可配置 |

## PSA 加密 API 支持

### OH 中的 PSA API

mbedtls v3.6.5 在 OH 中**原生支持 PSA 加密 API**，无需额外适配：

```c
#include "psa/crypto.h"

// 初始化 PSA
psa_crypto_init();

// 生成密钥
psa_key_attributes_t attr = PSA_KEY_ATTRIBUTES_INIT;
psa_set_key_usage_flags(&attr, PSA_KEY_USAGE_SIGN_HASH);
psa_set_key_algorithm(&attr, PSA_ALG_ECDSA_ANY);
psa_set_key_type(&attr, PSA_KEY_TYPE_ECC_KEY_PAIR(PSA_ECC_FAMILY_SECP_R1));
psa_set_key_bits(&attr, 256);

psa_key_id_t key_id;
psa_generate_key(&attr, &key_id);

// 签名操作
psa_sign_hash(key_id, PSA_ALG_ECDSA_ANY, hash, hash_len, signature, &sig_len);
```

### PSA API 优势

1. **统一接口**: 统一的加密操作接口
2. **密钥抽象**: 密钥句柄机制，隐藏密钥细节
3. **驱动支持**: 支持硬件加密加速器
4. **安全存储**: 支持密钥安全存储集成

## 行为变更说明

### 1. 默认配置差异

| 配置项 | OH 默认 | 上游默认 | 影响 |
|--------|---------|----------|------|
| `MBEDTLS_SSL_VERIFY_NONE` | 默认禁用证书验证 | 默认禁用 | OH 可配置启用 |
| TLS 版本 | 1.2+ | 1.2+ | 一致 |
| 加密套件 | 标准集 | 标准集 | 一致 |

### 2. 证书验证行为

```c
// OH Port 层的证书验证行为
// 文件: port/src/tls_client.c

// 默认禁用证书验证（便于测试）
mbedtls_ssl_conf_authmode(&session->conf, MBEDTLS_SSL_VERIFY_NONE);

// 生产环境应启用验证
// mbedtls_ssl_conf_authmode(&session->conf, MBEDTLS_SSL_VERIFY_REQUIRED);
```

### 3. 错误码处理

OH Port 层统一了错误码返回格式：

```c
// Port 层错误码
#define RET_ERROR -1
#define RET_EOK 0

// 使用示例
int ret = mbedtls_ssl_handshake(&session->ssl);
if (ret != 0) {
    return -RET_ERROR;  // 转换为 Port 层错误码
}
return RET_EOK;
```

## 废弃或禁用功能

### 未在 OH 中启用的功能

| 功能 | 上游支持 | OH 状态 | 原因 |
|------|----------|---------|------|
| **TLS 1.0** | 是 | 禁用 | 安全考虑 |
| **TLS 1.1** | 是 | 禁用 | 安全考虑 |
| **SSL v3** | 是 | 禁用 | 安全考虑 |
| **RC4** | 是 | 禁用 | 安全考虑 |
| **3DES** | 是 | 可配置 | 性能考虑 |

### 配置文件中的禁用项

```c
// 在 config_liteos_m.h 中
// #define MBEDTLS_SSL_PROTO_SSL3     // 已禁用
// #define MBEDTLS_SSL_PROTO_TLS1     // 已禁用
// #define MBEDTLS_SSL_PROTO_TLS1_1   // 已禁用
```

## 接口迁移指南

### 从上游 API 迁移到 OH Port 层

#### 场景：使用简化 TLS 客户端

**上游方式**:
```c
mbedtls_ssl_context ssl;
mbedtls_ssl_init(&ssl);
mbedtls_ssl_setup(&ssl, &conf);
// ... 复杂的手动配置
```

**OH Port 层方式**:
```c
MbedTLSSession session = {0};
session.host = "example.com";
session.port = "443";
MbedtlsClientInit(&session, entropy, entropy_len);
MbedtlsClientContext(&session);
MbedtlsClientConnect(&session);
// ... 简化使用
```

### 从 OH Port 层迁移到上游 API

如果需要更多控制，可以直接使用上游 API：

```c
// 直接使用 mbedtls API
mbedtls_ssl_context ssl;
mbedtls_ssl_init(&ssl);
mbedtls_ssl_setup(&ssl, &conf);
// ... 完全控制
```

## 总结

| 差异类型 | OH 特有 | 上游兼容 | 说明 |
|----------|---------|----------|------|
| **TLS 客户端接口** | 是 | - | 简化 API |
| **日志接口** | 是 | - | Hilog 集成 |
| **PSA API** | - | 是 | 完全兼容 |
| **标准加密 API** | - | 是 | 完全兼容 |
| **构建配置** | 是 | - | 内核特定配置 |
| **根证书** | 是 | - | OH 证书数据 |
