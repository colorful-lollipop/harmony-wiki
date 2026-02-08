# OpenHarmony 适配层详细分析

## 概述

mbedtls 库在 OpenHarmony 中的适配**未采用传统的 `.patch` 文件方式**，而是通过**完整的 Port 适配层**实现。这种方式更加清晰，便于维护和升级。

适配层位于 `port/` 目录，包含以下核心组件：

```
port/
├── include/
│   ├── tls_client.h       # TLS 客户端接口定义
│   ├── tls_certificate.h  # 证书接口定义
│   └── mbedtls_log.h      # 日志系统适配
├── src/
│   ├── tls_client.c       # TLS 客户端实现
│   └── tls_certificate.c  # 根证书实现
└── config/
    ├── config_liteos_m.h  # LiteOS-M 内核配置
    ├── config_liteos_a.h  # LiteOS-A 内核配置
    ├── compat_posix/      # POSIX 兼容层
    └── compat_lwip/       # LwIP 兼容层
```

## 适配层组件详解

### 1. TLS 客户端适配 (tls_client.h/tls_client.c)

#### 文件信息

| 属性 | 值 |
|------|-----|
| **路径** | `port/src/tls_client.c` |
| **代码行数** | 约 192 行 |
| **功能** | 提供简化的 TLS 客户端操作接口 |

#### 核心数据结构

```c
typedef struct MbedTLSSession {
    char *host;              // 服务器地址
    char *port;              // 服务器端口
    unsigned char *buffer;   // 数据缓冲区
    size_t buffer_len;       // 缓冲区长度
    
    mbedtls_ssl_context ssl;      // TLS 上下文
    mbedtls_ssl_config conf;      // TLS 配置
    mbedtls_entropy_context entropy;  // 熵源
    mbedtls_ctr_drbg_context ctr_drbg;  // 随机数生成器
    mbedtls_net_context server_fd; // 网络连接上下文
    mbedtls_x509_crt cacert;      // CA 证书
} MbedTLSSession;
```

#### API 接口

| 函数 | 功能 |
|------|------|
| `MbedtlsClientInit()` | 初始化 TLS 客户端会话 |
| `MbedtlsClientClose()` | 关闭 TLS 连接 |
| `MbedtlsClientContext()` | 配置 TLS 上下文 |
| `MbedtlsClientConnect()` | 建立 TLS 连接 |
| `MbedtlsClientRead()` | 读取加密数据 |
| `MbedtlsClientWrite()` | 写入加密数据 |

#### 使用示例

```c
// 1. 初始化会话
MbedTLSSession session = {0};
session.host = "example.com";
session.port = "443";
MbedtlsClientInit(&session, entropy, entropy_len);

// 2. 配置上下文
MbedtlsClientContext(&session);

// 3. 建立连接
MbedtlsClientConnect(&session);

// 4. 安全通信
MbedtlsClientWrite(&session, data, data_len);
MbedtlsClientRead(&session, buffer, buffer_size);

// 5. 关闭连接
MbedtlsClientClose(&session);
```

#### 适配特点

1. **简化接口**: 将复杂的 mbedtls TLS API 封装为易用的接口
2. **错误处理**: 统一的错误码返回 (-1: 错误, 0: 成功)
3. **证书验证**: 支持证书验证（默认禁用，可配置）
4. **调试支持**: 集成调试日志功能

### 2. 日志系统适配 (mbedtls_log.h)

#### 文件信息

| 属性 | 值 |
|------|-----|
| **路径** | `port/include/mbedtls_log.h` |
| **功能** | OH 日志系统集成 |

#### 日志宏定义

```c
// OH 日志域和标签
#define LOG_DOMAIN 0xD002B00
#define LOG_TAG "Mbedtls"

// 日志级别宏
#define LOGD(fmt, ...)  // 调试日志
#define LOGI(fmt, ...)  // 信息日志
#define LOGW(fmt, ...)  // 警告日志
#define LOGE(fmt, ...)  // 错误日志
```

#### 适配说明

- 使用 OH 日志系统 (Hilog) 输出日志
- 日志域: `0xD002B00` (安全子系统)
- 调试日志仅在 `OHOS_DEBUG` 定义时启用
- 格式: `{%s()-%s:%d} message` (函数名-文件名:行号)

### 3. 证书管理适配 (tls_certificate.h/tls_certificate.c)

#### 文件信息

| 属性 | 值 |
|------|-----|
| **路径** | `port/src/tls_certificate.c` |
| **功能** | 内置根证书管理 |

#### 证书数据

```c
// 华为根证书
extern const char G_MBEDTLS_ROOT_CERTIFICATE[];
extern const size_t G_MBEDTLS_ROOT_CERTIFICATE_LEN;
```

#### 证书用途

1. **TLS 连接验证**: 用于验证服务器证书
2. **证书链验证**: 提供 CA 证书进行链式验证
3. **安全连接**: 确保与服务器的加密通信

#### 注意事项

- 当前证书为测试用途
- 生产环境需要替换为正式的根证书
- 证书更新策略: 定期更新，跟随系统证书

## 配置文件适配

### LiteOS-M 内核配置 (config_liteos_m.h)

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **目标内核** | LiteOS-M | 轻量级物联网内核 |
| **配置文件路径** | `port/config/config_liteos_m.h` |
| **代码体积优化** | 是 | 针对资源受限设备 |

#### 主要配置特点

```c
// 在 BUILD.gn 中设置
if (ohos_kernel_type == "liteos_m") {
  defines += [
    "__unix__",
    "MBEDTLS_CONFIG_FILE=<../port/config/config_liteos_m.h>",
  ]
}
```

### LiteOS-A 内核配置 (config_liteos_a.h)

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **目标内核** | LiteOS-A | 支持 MMU 的轻量级内核 |
| **配置文件路径** | `port/config/config_liteos_a.h` |
| **功能完整性** | 是 | 完整功能集 |

#### 主要配置特点

```c
// 在 BUILD.gn 中设置
if (ohos_kernel_type == "liteos_a") {
  defines += [
    "__unix__",
    "MBEDTLS_CONFIG_FILE=<../port/config/config_liteos_a.h>",
  ]
}
```

### 配置差异对比

| 配置项 | LiteOS-M | LiteOS-A | 说明 |
|--------|----------|----------|------|
| **代码体积** | 优化 | 完整 | M 侧重资源节省 |
| **TLS 服务器** | 可选 | 可选 | 可通过 GN 配置控制 |
| **加密算法** | 基础集 | 完整集 | A 支持更多算法 |
| **PSA 支持** | 是 | 是 | 统一加密 API |

## 兼容性适配层

### POSIX 兼容层 (compat_posix)

| 属性 | 值 |
|------|-----|
| **路径** | `port/config/compat_posix/socket_compat.h` |
| **用途** | POSIX 系统 socket 兼容 |

### LwIP 兼容层 (compat_lwip)

| 属性 | 值 |
|------|-----|
| **路径** | `port/config/compat_lwip/socket_compat.h` |
| **用途** | LwIP TCP/IP 栈兼容 |

#### 使用场景

```python
# 根据产品配置选择兼容层
if product_name in ["generic_m55_arm_32_bes_aurora_wear_mini_application",
                    "generic_m55_arm_32_bes_phoinix_wear_mini_application"]:
    MBEDTLS_INLCUDE_DIRS += [ "$MBEDTLSDIR/port/config/compat_lwip" ]
else:
    MBEDTLS_INLCUDE_DIRS += [ "$MBEDTLSDIR/port/config/compat_posix" ]
```

## 适配层维护建议

### 升级上游版本注意事项

1. **Port 层独立**: Port 适配层与上游代码分离，升级影响小
2. **配置同步**: 需要同步更新 `config_liteos_*.h` 配置文件
3. **API 兼容性**: 检查上游 API 变更，确保 Port 层兼容
4. **测试验证**: 重点测试 TLS 客户端功能和证书验证

### 自定义适配建议

如需针对特定硬件平台进行优化：

1. **复制适配层**: 复制 `port/` 目录进行定制
2. **配置 GN 变量**: 使用 `mbedtls_porting_path` 指定自定义路径
3. **保持接口兼容**: 确保新适配层接口与原有接口一致

## 适配层文件清单

| 文件 | 类型 | 用途 | OH 特有 |
|------|------|------|--------|
| `port/include/tls_client.h` | 头文件 | TLS 客户端接口 | 是 |
| `port/include/tls_certificate.h` | 头文件 | 证书接口 | 是 |
| `port/include/mbedtls_log.h` | 头文件 | 日志适配 | 是 |
| `port/src/tls_client.c` | 源文件 | TLS 客户端实现 | 是 |
| `port/src/tls_certificate.c` | 源文件 | 证书实现 | 是 |
| `port/config/config_liteos_m.h` | 配置文件 | LiteOS-M 配置 | 是 |
| `port/config/config_liteos_a.h` | 配置文件 | LiteOS-A 配置 | 是 |
| `port/config/compat_posix/*` | 兼容层 | POSIX 兼容 | 是 |
| `port/config/compat_lwip/*` | 兼容层 | LwIP 兼容 | 是 |
