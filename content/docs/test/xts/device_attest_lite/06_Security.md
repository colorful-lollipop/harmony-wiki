# 安全风险评审

## 6.1 威胁模型

### 攻击面清单

| 攻击面 | 描述 | 风险等级 |
|--------|------|---------|
| **HAL Token 接口** | OEM 实现的 Token 读写接口 | HIGH |
| **网络通信** | TLS 连接与证书验证 | MEDIUM |
| **JSON 解析** | 服务器响应解析 | MEDIUM |
| **COAP 解析** | 二进制协议解析 | MEDIUM |
| **输入校验** | 参数与缓冲区检查 | MEDIUM |
| **密钥派生** | PSK 派生逻辑 | MEDIUM |
| **内存管理** | 敏感数据清除 | LOW |
| **UDID 生成** | 设备指纹生成 | LOW |

---

## 6.2 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      设备边界                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  HAL 层 (OEM 实现)                                        │  │
│  │  • Token 安全存储                                         │  │
│  │  • manuKey/productId 读取                                │  │
│  └───────────────────────────────────────────────────────────┘  │
│                            ↓                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Core 层 (本模块实现)                                      │  │
│  │  • 加密操作 (AES-128-CBC)                                 │  │
│  │  • 认证协议 (Challenge-Response)                          │  │
│  │  • Token 管理                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                            ↓                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  网络边界 (TLS 1.2+)                                       │  │
│  │  • 服务器证书验证                                          │  │
│  │  • TLS 加密通道                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                            ↓                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  云端服务器                                               │  │
│  │  • 挑战码生成                                             │  │
│  │  • 认证结果验证                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6.3 数据流与敏感操作

### 认证流程数据流

```
1. [本地] 读取 Token/manuKey/productId
   └── HAL 接口: HalReadToken(), HalGetManufactureKey(), HalGetProdId()
   └── 风险: Token 明文读取

2. [本地] 生成 HMAC
   └── HMAC-SHA256(challenge, token_value)
   └── 风险: Token 暴露于内存

3. [网络] TLS 发送认证请求
   └── mbedtls TLS 1.2 加密
   └── 风险: 证书验证绕过

4. [网络] 接收认证响应
   └── JSON/COAP 解析
   └── 风险: 解析器漏洞

5. [本地] 存储新 Token (如果认证成功)
   └── AES-128-CBC 加密
   └── 风险: 加密密钥派生
```

---

## 6.4 可利用点 (基于代码证据)

### 风险 1: Token 存储依赖 OEM 实现

**证据**: `services/core/adapter/attest_adapter_oem.c`

**触发条件**:
```
OEM 实现的 HalReadToken() / HalWriteToken()
    ↓
Token 存储在非安全分区
    ↓
攻击者可以提取 Token
```

**影响**:
- 设备身份伪造
- 认证结果欺骗

**修复建议**:
```
1. Token 必须存储在安全分区 (TEE/Secure Element)
2. 使用硬件绑定密钥加密 Token
3. Token 读取时进行完整性校验
```

---

### 风险 2: PSK 派生密钥硬编码

**证据**: `services/core/security/attest_security.c:32-40`

```c
// PSK 派生密钥 (硬编码)
static const uint8_t g_pskKey[16] = {...};        // Line 32
static const uint8_t g_encryptedPsk[16] = {...}; // Line 40

// 派生 AES 密钥
static int32_t GetAesKey(uint8_t *key, uint32_t keyLen)
{
    // XOR 组合两个密钥
    for (uint32_t i = 0; i < keyLen; i++) {
        key[i] = g_pskKey[i] ^ g_encryptedPsk[i];
    }
}
```

**触发条件**:
```
攻击者获取固件镜像
    ↓
提取 g_pskKey 和 g_encryptedPsk
    ↓
XOR 组合得到完整 PSK
    ↓
伪造认证消息
```

**影响**:
- 设备身份伪造
- 认证协议失效

**修复建议**:
```
1. PSK 应由 OEM 在生产时写入
2. 使用设备唯一密钥派生 PSK
3. PSK 存储在安全存储区
```

---

### 风险 3: TLS 证书验证可绕过

**证据**: `services/core/network/attest_channel.c:166-198`

```c
// TLS 配置 (Line 173)
mbedtls_ssl_config_init(&sslConf);
// ...
mbedtls_ssl_conf_authmode(&sslConf, MBEDTLS_SSL_VERIFY_REQUIRED);
// ...
// 自定义验证回调 (Line 180)
ret = mbedtls_ssl_set_verify_callback(&sslConf, LazyVerifyCert);
```

**问题**:
- `MBEDTLS_SSL_VERIFY_REQUIRED` 已设置
- 但 `LazyVerifyCert` 函数实现未知

**修复建议**:
```
1. 确保 LazyVerifyCert 执行完整证书链验证
2. 验证证书吊销状态 (CRL/OCSP)
3. 考虑证书固定 (Certificate Pinning)
```

---

### 风险 4: 内存中 Token证据**: `services 暴露

**/core/security/attest_security_token.c:681`

```c
// Token 使用后清除 (Line 681)
(void)memset_s(tokenValue, TOKEN_VALUE_LEN + 1, 0, TOKEN_VALUE_LEN + 1);
```

**触发条件**:
```
认证流程执行期间
    ↓
Token 值存在于进程内存中
    ↓
内存读取攻击可获取 Token
```

**影响**:
- Token 泄露
- 设备身份伪造

**修复建议**:
```
1. 使用 mbedtls_platform_zeroize() 清除敏感数据
2. 考虑使用 SGX 或 TEE 保护敏感操作
3. 减少 Token 在内存中的停留时间
```

---

### 风险 5: JSON 解析无 schema 验证

**证据**: `services/core/network/attest_network.c` (cJSON 使用)

**触发条件**:
```
服务器返回恶意构造的 JSON
    ↓
cJSON 解析器处理异常输入
    ↓
可能导致缓冲区溢出或拒绝服务
```

**影响**:
- 拒绝服务
- 潜在代码执行

**修复建议**:
```
1. 对所有 JSON 字段进行长度校验
2. 使用 cJSON 的边界检查函数
3. 添加 JSON schema 验证
```

---

### 风险 6: 调试日志泄露敏感信息

**证据**: `services/core/BUILD.gn:33`

```gn
if (enable_attest_log_debug) {
  defines += [ "__ATTEST_HILOG_LEVEL_DEBUG__" ]
}
```

**触发条件**:
```
启用调试日志
    ↓
HILOGE/HILOGI 输出敏感数据
    ↓
日志被读取或泄露
```

**影响**:
- Token 泄露
- 认证协议细节暴露

**修复建议**:
```
1. 禁止在生产版本中启用调试日志
2. 日志中不输出 Token、密钥等敏感数据
3. 使用编译时保护禁用日志
```

---

## 6.5 安全控制措施

### 已实现的安全措施

| 措施 | 实现位置 | 效果 |
|------|---------|------|
| AES-128-CBC 加密 | `attest_security.c` | Token 静态加密 |
| HKDF 密钥派生 | `attest_security.c:164-196` | 强密钥生成 |
| HMAC-SHA256 认证 | `attest_security_token.c` | 消息认证 |
| TLS 1.2 通信 | `attest_channel.c` | 传输加密 |
| 证书链验证 | `attest_channel.c:113-157` | 服务器身份验证 |
| 安全内存操作 | `attest_security_token.c:681` | 敏感数据清除 |

### 建议增加的安全措施

| 措施 | 优先级 | 难度 |
|------|--------|------|
| 硬件绑定密钥 | HIGH | HIGH |
| 证书固定 | MEDIUM | MEDIUM |
| 运行时完整性校验 | MEDIUM | HIGH |
| 防重放攻击时间戳 | LOW | LOW |
| 安全启动集成 | HIGH | HIGH |

---

## 6.6 检查范围与局限性

### 已检查范围

| 组件 | 检查深度 | 备注 |
|------|---------|------|
| JSI 接口 | 完整 | `native_device_attest.cpp` |
| Inner API | 完整 | `devattest_interface.h` |
| Core 业务逻辑 | 完整 | `services/core/attest/` |
| 网络模块 | 完整 | `services/core/network/` |
| 安全模块 | 完整 | `services/core/security/` |
| HAL 抽象层 | 基础 | `services/core/adapter/` |

### 未检查范围

| 组件 | 原因 |
|------|------|
| OEM HAL 实现 | 平台相关，未在此仓库 |
| 第三方库 (mbedtls) | 外部依赖 |
| 固件安全 | 系统级问题 |
| 网络基础设施 | 服务器端 |

---

## 相关跳转

- [02_Architecture](02_Architecture.md) - 架构概览
- [03_JSI_API](03_JSI_API.md) - JS 接口说明
- [05_Build](05_Build.md) - 构建配置
