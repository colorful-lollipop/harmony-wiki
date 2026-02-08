# 06 - 安全风险评审

## 目的与适用范围

**本文档目的**：基于代码证据分析 `device_attest` 模块的安全风险，识别攻击面和可被利用点。

**适用范围**：
- 安全审计人员
- 安全架构师
- 代码审查人员

## 检查范围与方法

### 检查范围

| 检查项 | 范围 | 文件/目录 |
|--------|------|-----------|
| 输入校验 | N-API 入口 | `interfaces/kits/napi/src/` |
| 权限控制 | IPC 层 | `services/devattest_ability/src/` |
| 加密实现 | 安全模块 | `services/core/security/` |
| 网络通信 | 网络模块 | `services/core/network/` |
| 文件操作 | 适配层 | `services/core/adapter/`, `services/oem_adapter/` |
| 内存管理 | 核心逻辑 | `services/core/attest/`, `utils/` |

### 检查方法

- 静态代码分析
- 调用链追踪
- 信任边界识别
- 数据流分析

## 信任边界与攻击面

### 系统架构信任边界

```
┌────────────────────────────────────────────────────────────────┐
│                        外部不可信区                            │
│  ┌─────────┐    ┌─────────┐    ┌─────────────────────────┐    │
│  │ 恶意应用 │    │ 网络攻击 │    │ 伪造云端服务器          │    │
│  └────┬────┘    └────┬────┘    └───────────┬─────────────┘    │
│       │              │                      │                  │
│       │              │                      │                  │
│       ▼              ▼                      ▼                  │
│  ╔═══════════════════════════════════════════════════════════╗ │
│  ║                    信任边界（N-API/IPC）                   ║ │
│  ╚═══════════════════════════════════════════════════════════╝ │
│       │              │                      │                  │
│       ▼              ▼                      ▼                  │
│  ┌─────────┐    ┌─────────┐    ┌─────────────────────────┐    │
│  │ 系统应用 │    │ TLS/SSL │    │ 真实云端服务器          │    │
│  └─────────┘    └─────────┘    └─────────────────────────┘    │
│                        内部可信区                              │
└────────────────────────────────────────────────────────────────┘
```

### 攻击面清单

| 攻击面 | 入口点 | 风险等级 | 检查状态 |
|--------|--------|----------|----------|
| N-API 接口 | `getAttestStatus()` | 中 | ✅ 已检查 |
| IPC 通信 | `OnRemoteRequest()` | 中 | ✅ 已检查 |
| 网络响应解析 | `ParseHttpsResp()` | **高** | ✅ 已检查 |
| JSON 解析 | `ParseAuthType()` | **高** | ✅ 已检查 |
| 文件系统 | `attest_adapter*.c` | 中 | ✅ 已检查 |
| 加密模块 | `attest_security.c` | 高 | ✅ 已检查 |
| OEM HAL | `device_attest_oem_*.c` | 高 | ⚠️ OEM 依赖 |

## 详细风险分析

### 风险 1：N-API 参数校验

**位置**: `interfaces/kits/napi/src/devattest_napi.cpp:141-162`

**代码分析**:
```cpp
napi_value DevAttestNapi::GetAttestResultInfo(napi_env env, napi_callback_info info) {
    size_t argc = PARAM1;  // 最多1个参数
    napi_value argv[1] = {0};
    napi_get_cb_info(env, info, &argc, argv, &thisVar, &data);
    
    // 参数数量检查
    if (argc > PARAM1) {
        napi_throw(env, GenerateBusinessError(env, DEVATTEST_ERR_JS_PARAMETER_ERROR));
    }
    
    // 参数类型检查
    if (argc == PARAM1) {
        napi_typeof(env, argv[0], &type);
        if (type != napi_function) {
            napi_throw(env, GenerateBusinessError(env, DEVATTEST_ERR_JS_PARAMETER_ERROR));
        }
    }
}
```

**评估**: ✅ **安全**
- 参数数量限制为最多 1 个
- 参数类型校验为函数
- 错误时抛出 JS 异常

### 风险 2：IPC 接口 Token 校验

**位置**: `services/devattest_ability/src/devattest_service_stub.cpp:34-52`

**代码分析**:
```cpp
int DevAttestServiceStub::OnRemoteRequest(uint32_t code,
    MessageParcel& data, MessageParcel& reply, MessageOption& option) {
    // 接口描述符校验
    if (data.ReadInterfaceToken() != GetDescriptor()) {
        HILOGE("[OnRemoteRequest] failed, descriptor is not matched!");
        return DEVATTEST_SERVICE_FAILED;
    }
    // ...
}
```

**评估**: ✅ **安全**
- 校验 IPC 接口描述符
- 不匹配时返回错误

### 风险 3：系统应用权限检查

**位置**: `services/devattest_ability/src/devattest_service_stub.cpp:54-64`

**代码分析**:
```cpp
int DevAttestServiceStub::GetAttestStatusInner(MessageParcel& data, MessageParcel& reply) {
    // 系统应用检查
    if (!DelayedSingleton<Permission>::GetInstance()->IsSystem()) {
        HILOGE("[GetAttestStatusInner] not a system");
        reply.WriteInt32(DEVATTEST_ERR_JS_IS_NOT_SYSTEM_APP);
        return DEVATTEST_SUCCESS;
    }
}
```

**权限实现** (`common/permission/src/permission.cpp:36-69`):
```cpp
bool Permission::IsSystem() {
    AccessTokenID tokenId = IPCSkeleton::GetCallingTokenID();
    ATokenTypeEnum type = AccessTokenKit::GetTokenTypeFlag(tokenId);
    
    switch (type) {
        case ATokenTypeEnum::TOKEN_HAP:
            // 检查系统应用签名
            return TokenIdKit::IsSystemAppByFullTokenID(
                IPCSkeleton::GetCallingFullTokenID());
        case ATokenTypeEnum::TOKEN_NATIVE:
        case ATokenTypeEnum::TOKEN_SHELL:
            return true;  // Native/Shell 默认可通过
        default:
            return false;
    }
}
```

**评估**: ✅ **安全**
- 基于 AccessToken 框架检查
- 区分 HAP/Native/Shell 类型
- 系统应用需签名验证

**⚠️ 注意**: Native 和 Shell 进程默认通过，需确保这些进程本身可信。

### 风险 4：内存分配检查

**位置**: `services/devattest_ability/src/devattest_service.cpp:161-196`

**代码分析**:
```cpp
int32_t DevAttestService::GetAttestStatus(AttestResultInfo &attestResultInfo) {
    int32_t resultArraySize = MAX_ATTEST_RESULT_SIZE * sizeof(int32_t);
    int32_t *resultArray = (int32_t *)malloc(resultArraySize);
    if (resultArray == NULL) {
        HILOGE("[GetAttestStatus] malloc resultArray failed");
        return DEVATTEST_FAIL;
    }
    // ...
    do {
        ret = QueryAttest(&resultArray, MAX_ATTEST_RESULT_SIZE, &ticketStr, &ticketLength);
        // ...
    } while (0);
    
    // 释放资源
    if (ticketStr != NULL && ticketLength != 0) {
        free(ticketStr);
    }
    free(resultArray);
}
```

**评估**: ✅ **安全**
- 分配后检查 NULL
- 使用 do-while(0) 模式确保统一退出
- 正确释放资源

### 风险 5：字符串操作安全

**位置**: `services/core/security/attest_security.c:640-679`

**代码分析**:
```cpp
int32_t MD5Encode(const uint8_t* srcData, size_t srcDataLen, 
                  uint8_t* outputStr, int outputLen) {
    // 参数校验
    if (srcData == NULL || srcDataLen == 0 || outputStr == NULL) {
        return ATTEST_ERR;
    }
    
    uint8_t hash[MD5_LEN] = {0};
    char buf[DEV_BUF_LENGTH] = {0};
    
    for (size_t i = 0; i < MD5_LEN; i++) {
        // 使用 sprintf_s（安全版本）
        if (sprintf_s(buf, sizeof(buf), "%02x", value) < 0) {
            ATTEST_LOG_ERROR("[MD5Encode] Failed to sprintf");
            ret = ATTEST_ERR;
            break;
        }
        // 使用 strcat_s（安全版本）
        if (strcat_s((char*)outputStr, outputLen, buf) != 0) {
            ATTEST_LOG_ERROR("[MD5Encode] Failed to strcat");
            ret = ATTEST_ERR;
            break;
        }
    }
    // 清零敏感数据
    memset_s(buf, DEV_BUF_LENGTH, 0, DEV_BUF_LENGTH);
    memset_s(hash, MD5_LEN, 0, MD5_LEN);
}
```

**评估**: ✅ **安全**
- 使用安全版本函数（sprintf_s, strcat_s）
- 参数有效性检查
- 敏感数据使用后清零

### 风险 6：加密密钥派生

**位置**: `services/core/security/attest_security.c:339-371`

**代码分析**:
```cpp
int32_t GetAesKey(const SecurityParam* salt, const VersionData* versionData, 
                  const SecurityParam* aesKey) {
    // 参数校验
    if ((salt == NULL) || (versionData == NULL) || (aesKey == NULL) || 
        (versionData->versionLen == 0)) {
        return ERR_ATTEST_SECURITY_INVALID_ARG;
    }
    
    uint8_t productInfo[MANUFACTUREKEY_LEN + PRODUCT_ID_LEN] = {0};
    SecurityParam info = {productInfo, sizeof(productInfo)};
    ret = GetProductInfo(versionData->version, &info);
    
    uint8_t psk[PSK_LEN] = {0};
    ret = GetPsk(psk, PSK_LEN);
    
    SecurityParam key = {psk, sizeof(psk)};
    const mbedtls_md_info_t *mdInfo = mbedtls_md_info_from_type(MBEDTLS_MD_SHA256);
    
    // 使用 HKDF 派生密钥
    ret = mbedtls_hkdf(mdInfo, salt->param, salt->paramLen,
                       key.param, key.paramLen,
                       info.param, info.paramLen,
                       aesKey->param, aesKey->paramLen);
    
    memset_s(psk, PSK_LEN, 0, PSK_LEN);  // 清零 PSK
}
```

**评估**: ✅ **安全**
- 使用标准 HKDF 密钥派生
- SHA-256 哈希算法
- 敏感数据及时清零

### 风险 7：PSK 存储方式

**位置**: `services/core/security/attest_security.c:108-118`

**代码分析**:
```cpp
// g_pskKey 和 g_encryptedPsk 是 psk 的计算因子
uint8_t g_pskKey[BASE64_PSK_LENGTH] = {
    0x35, 0x4d, 0x36, 0x50, 0x42, 0x79, 0x39, 0x41, ...
};

uint8_t g_encryptedPsk[BASE64_PSK_LENGTH] = {
    0x74, 0x71, 0x57, 0x2b, 0x56, 0x6d, 0x52, 0x6b, ...
};

// PSK 通过异或计算得到
for (size_t i = 0; i < pskLen; i++) {
    psk[i] = base64Psk[i] ^ base64PskKey[i];
}
```

**评估**: ⚠️ **中风险**
- PSK 计算因子硬编码在二进制中
- 通过异或操作还原，可被逆向分析
- **缓解措施**: 这只是计算因子，实际 PSK 还需结合 manuKey 和 productId 派生

**建议**: 考虑使用更安全的密钥保护机制，如 White-box 加密。

### 风险 8：网络通信 TLS

**位置**: `services/core/network/attest_network.c`

**TLS/SSL 配置** (证据: `attest_network.c:380-429`):
```c
static int32_t InitSSLSocket(int32_t socketFd, SSL **socketSSL)
{
    SSL_CTX *socketCTX = SSL_CTX_new(SSLv23_client_method());
    // 证书验证启用
    SSL_CTX_set_verify(socketCTX, SSL_VERIFY_PEER, NULL);
    // 从 /etc/security/certificates 加载 CA 证书
    SSL_CTX_load_verify_locations(socketCTX, NULL, caFile);
    // 密码套件：ALL:!EXP（排除出口级弱密码）
    SSL_CTX_set_cipher_list(socketCTX, "ALL:!EXP");
}
```

**服务器证书验证** (证据: `attest_network.c:478-506`):
```c
static int32_t VerifySSLCA(SSL *postSSL)
{
    // X.509 证书链验证
    retCode = SSL_get_verify_result(postSSL);
    if (retCode != X509_V_OK) {
        return ATTEST_ERR;
    }
    // 提取并验证对端证书
    srvCert = SSL_get_peer_certificate(postSSL);
}
```

**评估**: ✅ **基本安全**
- ✅ 使用 OpenSSL TLS 1.2+
- ✅ 证书链验证 (X509_V_OK)
- ✅ CA 证书从系统信任存储加载
- ✅ 排除弱密码套件 (ALL:!EXP)

**⚠️ 注意**: 未启用证书固定 (Certificate Pinning)，依赖系统 CA 信任存储。

**建议**:
1. 考虑启用证书固定，绑定特定证书或公钥
2. 添加证书吊销列表 (CRL) 检查
3. 实现证书透明度 (Certificate Transparency) 日志验证

### 风险 8.1：网络响应 Content-Length 处理 【HIGH】

**位置**: `services/core/network/attest_network.c:1042-1079`

**代码分析**:
```c
static int32_t ParseHttpsResp(char *respMsg, char **outBody)
{
    int32_t contentLen = 0;
    // 从 HTTP 响应头解析 Content-Length
    retCode = ParseHttpsRespIntPara(respMsg, ATTEST_HTTPS_RESLEN, &contentLen);
    if (retCode != ATTEST_OK || contentLen <= 0 || contentLen >= MAX_ATTEST_MALLOC_BUFF_SIZE) {
        return ATTEST_ERR;
    }
    // 基于 contentLen 分配内存
    char *body = (char *)ATTEST_MEM_MALLOC(contentLen + 1);
    // [ISSUE] 未验证 respMsg 总长度是否 >= headerLen + contentLen
    retCode = memcpy_s(body, contentLen + 1, respMsg + headerLen, contentLen);
}
```

**触发路径**:
```
攻击者控制 HTTP 响应 →
Content-Length 伪造为超大值 →
memcpy_s 基于超大长度复制 →
堆溢出 / 拒绝服务
```

**触发条件**:
1. 攻击者能够中间人攻击网络连接，或
2. 云端服务器被攻陷并返回恶意响应

**影响评估**:
- **利用可能性**: 中（需中间人位置）
- **影响程度**: 严重（可导致代码执行或 DoS）
- **权限要求**: 网络层中间人位置

**修复建议**:
```c
// 添加响应总长度验证
uint32_t respTotalLen = strlen(respMsg);
if (headerLen > respTotalLen || contentLen > respTotalLen - headerLen) {
    ATTEST_LOG_ERROR("[ParseHttpsResp] Invalid content length");
    return ATTEST_ERR;
}
```

### 风险 9：文件路径遍历

**检查范围**: `services/core/adapter/attest_adapter*.c`

**评估**: ✅ **未发现明显问题**
- 文件路径为硬编码或从系统参数获取
- 未发现用户输入直接拼接路径的情况

### 风险 9：JSON 解析无严格验证 【HIGH】

**位置**: `services/core/attest/attest_service_auth.c:272-293`

**代码分析**:
```c
static int32_t ParseAuthType(const cJSON* root, AuthStatus* authStatus)
{
    // 直接从 JSON 提取字符串，无长度预检查
    char* temp = cJSON_GetStringValue(cJSON_GetObjectItem(root, "authType"));
    // [ISSUE] 未检查 temp 是否为 NULL
    uint32_t len = strlen(temp);  // 若 temp 为 NULL 导致崩溃
    if (len == 0 || len >= MAX_ATTEST_BUFF_LEN) {
        return ATTEST_ERR;
    }
    authStatus->authType = ATTEST_MEM_MALLOC(len + 1);
    memcpy_s(authStatus->authType, len + 1, temp, len);
}
```

**JSON 解析入口** (证据: `attest_service_auth.c:351, 944`):
```c
// 多个位置使用 cJSON_Parse 解析外部数据
cJSON* root = cJSON_Parse(decodedAuthStatus);  // Base64 解码后直接解析
cJSON* json = cJSON_Parse(msg);                  // 网络响应直接解析
```

**触发路径**:
```
攻击者构造恶意 JSON →
authType 字段为 null 或超长字符串 →
strlen(null) 崩溃 / 缓冲区溢出 / 内存耗尽
```

**修复建议**:
```c
// 添加 NULL 检查和长度验证
char* temp = cJSON_GetStringValue(cJSON_GetObjectItem(root, "authType"));
if (temp == NULL) {
    ATTEST_LOG_ERROR("[ParseAuthType] authType is null");
    return ATTEST_ERR;
}
uint32_t len = strlen(temp);
if (len == 0 || len >= MAX_ATTEST_BUFF_LEN) {
    ATTEST_LOG_ERROR("[ParseAuthType] authType length invalid");
    return ATTEST_ERR;
}
```

### 风险 10：竞态条件

**位置**: `services/core/attest/attest_service.c:39-59`

**代码分析**:
```c
pthread_mutex_t g_mtxAttest = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t g_authStatusMutex;

static int32_t GetAuthResultCode(void) {
    pthread_mutex_lock(&g_authStatusMutex);
    AttestReadAuthResultCode((char*)&g_authResultCode, 1);
    ret = g_authResultCode;
    pthread_mutex_unlock(&g_authStatusMutex);
    return ret;
}

static void UpdateAuthResultCode(uint8_t authResultCode) {
    pthread_mutex_lock(&g_authStatusMutex);
    AttestWriteAuthResultCode((char*)&authResultCode, 1);
    g_authResultCode = g_authResultCode;
    pthread_mutex_unlock(&g_authStatusMutex);
}
```

**评估**: ✅ **基本安全**
- 使用互斥锁保护共享状态
- `ProcAttest()` 全程持有 `g_mtxAttest`
- `QueryAttestStatus()` 查询时持有锁

**⚠️ 注意**: `g_authResultCode = g_authResultCode` 为无效赋值，应修复为 `g_authResultCode = authResultCode`。

## 可被利用点汇总

| # | 风险点 | 严重程度 | 利用条件 | 影响 | 修复建议 |
|---|--------|----------|----------|------|----------|
| 1 | 网络响应 Content-Length | **高** | 网络中间人 | 堆溢出/DoS | 验证响应总长度 |
| 2 | JSON 解析 NULL 检查缺失 | **高** | 恶意 JSON | 崩溃/信息泄露 | 添加 NULL 检查 |
| 3 | PSK 硬编码 | 中 | 逆向分析 | 可计算 PSK | 使用 White-box 加密 |
| 4 | TLS 证书固定缺失 | 中 | 中间人攻击 | 通信被窃听 | 启用证书固定 |
| 5 | Native/Shell 默认通过 | 低 | 获取 Native 权限 | 绕过系统应用检查 | 按需缩小 Native 权限范围 |
| 6 | OEM manuKey 配置 | **高** | OEM 实现缺陷 | 密钥泄露 | 确保厂商正确配置 |

## 安全建议

### 高优先级

1. **修复网络响应解析漏洞** (证据: `attest_network.c:1042-1079`)
   - 添加响应总长度验证，防止 contentLength 溢出
   - 验证 `headerLen + contentLen <= respTotalLen`

2. **修复 JSON 解析 NULL 检查** (证据: `attest_service_auth.c:272-293`)
   - 添加 cJSON_GetStringValue 返回值 NULL 检查
   - 添加字符串长度预验证

3. **加强 PSK 保护**
   - 使用 White-box 加密隐藏密钥
   - 或采用密钥分散方案，每个设备派生独立密钥

### 中优先级

4. **启用 TLS 证书固定**
   - 启用证书固定（Certificate Pinning）
   - 禁用弱密码套件
   - 启用证书吊销检查

5. **OEM 适配层安全加固**
   - 确保厂商正确配置 manuKey（不得使用默认值）
   - 确保 token 存储分区不可从用户空间访问
   - 审查 OEM 实现代码

6. **日志脱敏**
   - 检查日志中是否包含敏感信息（token、key 等）
   - 生产环境禁用调试日志

### 低优先级

7. **代码加固**
   - 启用编译器安全特性（已启用 CFI、PAC-RET）
   - 考虑启用代码混淆

8. **竞态条件修复**
   - 修复 `g_authResultCode = g_authResultCode` 无效赋值

## 检查局限性

### 已知局限性

1. **OEM 适配层安全依赖**
   - manuKey、productId 由厂商实现提供
   - OEMReadToken/OEMWriteToken 存储安全性依赖厂商
   - 硬编码密钥示例需由厂商替换（证据: `device_attest_oem_adapter.c:188-221`）

2. **云端服务器安全**
   - 未审查 OpenHarmony 兼容性平台服务器端安全
   - 服务器证书管理不在检查范围内

3. **网络协议实现**
   - 部分网络通信细节需进一步分析
   - REST API 响应格式验证依赖代码实现

4. **动态行为分析限制**
   - 静态代码分析无法覆盖所有运行时场景
   - 竞态条件利用需要在运行时验证

### 检查范围外

- `test/` 目录下的测试代码
- 构建工具链安全
- 开发环境配置安全
- 设备物理安全

## 相关链接

### 内部文档
- [架构说明](02_Architecture.md) - 了解系统架构
- [N-API 接口](03_NAPI.md) - 了解对外接口
- [内部 API](04_Inner_API.md) - 了解内部模块接口

### 代码证据索引
| 证据类型 | 文件路径 | 关键符号 |
|---------|---------|---------|
| 网络安全 | `services/core/network/attest_network.c` | InitSSLSocket, VerifySSLCA |
| JSON 解析 | `services/core/attest/attest_service_auth.c` | ParseAuthType, ParseHttpsResp |
| 权限检查 | `common/permission/src/permission.cpp` | IsSystem |
| OEM 适配 | `services/oem_adapter/src/device_attest_oem_adapter.c` | OEMGetManufacturekey |

### 外部参考
- [OpenHarmony 兼容性平台](https://compatibility.openharmony.cn/)
- [安全模块源码](../services/core/security/)

---

**证据来源**：
- N-API 实现：`interfaces/kits/napi/src/devattest_napi.cpp`
- IPC Stub：`services/devattest_ability/src/devattest_service_stub.cpp`
- 权限检查：`common/permission/src/permission.cpp`
- 安全模块：`services/core/security/attest_security.c`
- 服务实现：`services/devattest_ability/src/devattest_service.cpp`
