# OpenHarmony NetStack 安全评审

## 1. 威胁模型概述

### 1.1 系统定位

**NetStack 是客户端网络库**，不是 SystemAbility 服务。

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层 (Applications)                       │
│  • JS/ETS 应用通过 N-API 调用                                    │
│  • C/C++ 应用通过 FFI 调用                                        │
│  信任边界: 运行时沙箱                                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ NetStack API 调用
┌─────────────────────────────────────────────────────────────────┐
│                      NetStack (当前代码库)                        │
│  • HTTP/HTTPS 客户端库                                            │
│  • Socket (TCP/UDP/TLS) 库                                      │
│  • WebSocket 客户端库                                              │
│  信任边界: 库加载边界                                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC (samgr_proxy)
                              ▼ 权限校验委托
┌─────────────────────────────────────────────────────────────────┐
│                   NetManager_Base (System Ability)                │
│  • 权限管理 (AccessToken)                                         │
│  • 连接管理                                                      │
│  信任边界: 系统服务边界                                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ 网络通信
┌─────────────────────────────────────────────────────────────────┐
│                         外部网络                                    │
│  • HTTP/HTTPS 服务器                                              │
│  • 其他 Socket 服务端                                              │
│  信任边界: 网络边界                                                │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面清单

| 攻击面 | 入口点 | 信任边界 | 风险等级 |
|--------|--------|----------|----------|
| **N-API 接口** | 所有网络操作入口 | JS ↔ Native | 高 |
| **URL 解析** | `httpRequest.request(url)` | 输入验证 | 高 |
| **HTTP 头解析** | `header` 参数 | 头部注入 | 中 |
| **Socket 输入** | `on('message')` 回调 | 数据解析 | 高 |
| **证书加载** | `clientCert` 参数 | 证书验证 | 高 |
| **代理配置** | `usingProxy` 参数 | 代理注入 | 中 |
| **文件路径** | `caPath`, `certPath` | 路径遍历 | 中 |
| **本地 Socket** | LocalSocket API | 权限检查 | 低 |

---

## 2. 安全机制分析

### 2.1 权限校验

#### 核心实现

**文件**: `utils/common_utils/src/netstack_common_utils.cpp:145-186`

```cpp
bool HasInternetPermission()
{
#ifdef OH_CORE_NETSTACK_PERMISSION_CHECK
    constexpr int inetGroup = 40002003;  // Internet 权限组 ID
    int groupNum = getgroups(0, nullptr);
    auto groups = (gid_t *)malloc(groupNum * sizeof(gid_t));
    groupNum = getgroups(groupNum, groups);
    for (int i = 0; i < groupNum; i++) {
        if (groups[i] == inetGroup) {
            free(groups);
            return true;  // 拥有 INTERNET 权限
        }
    }
    NETSTACK_LOGE("INTERNET permission denied by group");
    return false;
#else
    // 测试模式或特殊环境
    return true;
#endif
}
```

#### 权限错误处理

**文件**: `utils/napi_utils/include/base_context.h:34-35`

```cpp
static constexpr size_t PERMISSION_DENIED_CODE = 201;
static constexpr const char *PERMISSION_DENIED_MSG = "Permission denied";
```

**调用位置**:
- HTTP: `net_http_request_context.cpp:585-586`
- WebSocket: `websocket_client.cpp:639`
- Socket: `socket_exec.cpp` (多处)

### 2.2 证书验证

#### 证书验证核心

**文件**: `frameworks/native/net_ssl/net_ssl_verify_cert.cpp:20-80`

```cpp
uint32_t VerifyCert(const CertBlob *cert)
{
    X509 *certX509 = nullptr;
    X509_STORE *store = nullptr;
    X509_STORE_CTX *ctx = nullptr;
    
    // 加载系统 CA 证书
    if (X509_STORE_load_locations(store, nullptr, 
        SslConstant::SYSPRECAPATH) != VERIFY_RESULT_SUCCESS) {
        NETSTACK_LOGE("load SYSPRECAPATH store failed");
    }
    
    // 加载用户安装的 CA 证书
    std::string userInstalledCaPath = GetUserInstalledCaPath();
    if (X509_STORE_load_locations(store, nullptr, 
        userInstalledCaPath.c_str()) != VERIFY_RESULT_SUCCESS) {
        NETSTACK_LOGI("load userInstalledCaPath store failed");
    }
    
    // 执行验证
    verifyResult = static_cast<uint32_t>(X509_verify_cert(ctx));
}
```

#### 用户证书路径

**文件**: `frameworks/native/net_ssl/net_ssl_verify_cert.cpp:32-39`

```cpp
std::string GetUserInstalledCaPath()
{
    std::string userInstalledCaPath = SslConstant::USERINSTALLEDCAPATH;
    int32_t uid = OHOS::IPCSkeleton::GetCallingUid();
    uid /= SslConstant::UIDTRANSFORMDIVISOR;  // 除以 200000
    return userInstalledCaPath.append("/").append(std::to_string(uid).c_str());
}
```

#### TLS 验证模式

**文件**: `frameworks/native/tls_socket/src/tls_context.cpp:318-341`

```cpp
void TLSContext::SetVerify(TLSContext *tlsContext)
{
    if (!tlsContext->tlsConfiguration_.GetCertificate().data.Length() ||
        !tlsContext->tlsConfiguration_.GetPrivateKey().GetKeyData().Length()) {
        verifyMode_ = ONE_WAY_MODE;
        SSL_CTX_set_verify(tlsContext->ctx_, SSL_VERIFY_PEER, nullptr);
    } else {
        verifyMode_ = TWO_WAY_MODE;  // 双向认证
        SSL_CTX_set_verify(tlsContext->ctx_, 
            SSL_VERIFY_FAIL_IF_NO_PEER_CERT, nullptr);
    }
    
    // 跳过验证（危险！）
    if (tlsContext->tlsConfiguration_.GetSkipFlag()) {
        SSL_CTX_set_verify(tlsContext->ctx_, SSL_VERIFY_NONE, nullptr);
    }
}
```

### 2.3 证书锁定 (Certificate Pinning)

**文件**: `utils/common_utils/src/netstack_common_utils.cpp:564-593`

```cpp
bool IsCertPubKeyInPinned(const std::string &certPubKeyDigest, 
                          const std::string &pinnedPubkey)
{
    auto begin = pinnedPubkey.find("sha256//");
    if (begin != 0) {
        return false;
    }
    // 格式: sha256//base64hash1;base64hash2;base64hash3
    while (begin < pinnedPubkey.size()) {
        auto end = pinnedPubkey.find(";", begin);
        std::string candidate = pinnedPubkey.substr(
            begin + PINNED_PREFIX_LEN,  // "sha256//" 长度
            SHA256_BASE64_LEN);
        if (candidate == certPubKeyDigest) {
            return true;  // 匹配
        }
        begin = end + 1;
    }
    return false;
}
```

### 2.4 原子服务主机名限制

**文件**: `utils/common_utils/src/netstack_common_utils.cpp:197-213`

```cpp
bool IsAllowedHostname(const std::string &bundleName, 
                      const std::string &domainType, 
                      const std::string &url)
{
    if (bundleName.empty()) {
        NETSTACK_LOGE("isAllowedHostnameForAtomicService bundleName is empty");
        return true;
    }
    auto hostname = GetHostnameWithProtocolAndPortFromURL(url);
    return ApiPolicyUtils::IsAllowedHostname(bundleName, domainType, hostname);
}
```

**检查位置**:
- `websocket_client.cpp:639-646`
- `http_exec.cpp`

---

## 3. 可利用风险点

### ⚠️ 风险 1: 证书验证跳过风险

| 属性 | 值 |
|------|-----|
| **风险等级** | 🔴 高 |
| **CVE 编号** | 无 (潜在) |
| **证据位置** | `tls_context.cpp:341` |

**问题描述**:
当 `GetSkipFlag()` 返回 true 时，TLS 验证模式被设置为 `SSL_VERIFY_NONE`，完全跳过证书验证。

```cpp
// 危险代码
if (tlsContext->tlsConfiguration_.GetSkipFlag()) {
    SSL_CTX_set_verify(tlsContext->ctx_, SSL_VERIFY_NONE, nullptr);
}
```

**触发条件**:
```typescript
// WebSocket 可跳过服务器证书验证
let ws = webSocket.createWebSocket();
ws.connect(url, {
    skipServerCertVerification: true  // 危险！
});
```

**影响**:
- 中间人攻击 (MITM)
- 窃取敏感数据
- 注入恶意内容

**修复建议**:
1. 严格限制 `skipServerCertVerification` 的使用场景
2. 添加安全警告日志
3. 仅在调试模式下允许
4. 企业应用应强制不允许跳过

---

### ⚠️ 风险 2: 路径遍历风险

| 属性 | 值 |
|------|-----|
| **风险等级** | 🟠 中 |
| **CVE 编号** | 无 (潜在) |
| **证据位置** | `http_exec.cpp:1396-1399` |

**问题描述**:
证书路径直接使用用户传入的路径，未验证路径是否在允许范围内。

```cpp
// 潜在危险
certs.emplace_back(BASE_PATH + std::to_string(getuid() / UID_TRANSFORM_DIVISOR));
```

**触发条件**:
```typescript
httpRequest.request(url, {
    clientCert: {
        certPath: "../../../etc/passwd"  // 路径遍历尝试
    }
});
```

**影响**:
- 读取敏感文件
- 可能的代码执行

**修复建议**:
1. 使用 `realpath()` 验证规范化路径
2. 限制路径必须在 `/data/certificates/` 目录下
3. 检查路径穿越符号 (`..`)

---

### ⚠️ 风险 3: HTTP 头注入

| 属性 | 值 |
|------|-----|
| **风险等级** | 🟠 中 |
| **CVE 编号** | 无 (潜在) |
| **证据位置** | `http_request_options.cpp` |

**问题描述**:
HTTP 请求头直接接受用户传入的 Object，未验证头字段值。

```typescript
httpRequest.request(url, {
    header: {
        "Content-Type": "application/json",
        "Cookie": "session=xxx; admin=true"  // Cookie 注入
    }
});
```

**触发条件**:
```typescript
// 用户可控的 header 值
let userInput = getUserInput();
httpRequest.request(url, {
    header: {
        "X-Custom-Header": userInput  // 可能包含 CR LF 注入
    }
});
```

**影响**:
- HTTP 响应拆分攻击
- Cookie 注入
- 缓存投毒

**修复建议**:
1. 验证头字段值不包含 `\r\n`
2. 限制允许的头字段列表
3. 对敏感头进行过滤

---

### ⚠️ 风险 4: SOCKS5 代理认证凭据暴露

| 属性 | 值 |
|------|-----|
| **风险等级** | 🟠 中 |
| **证据位置** | `socks5_passwd_method.cpp` |

**问题描述**:
SOCKS5 代理认证凭据（用户名/密码）在内存中处理，可能在错误日志或转储中暴露。

```cpp
// 凭据处理
class Socks5PasswordAuthenticator {
    bool authenticate(const std::string& username, 
                     const std::string& password) {
        // 凭据传输到代理服务器
    }
};
```

**触发条件**:
```typescript
socket.connect({
    proxy: {
        type: ProxyTypes.SOCKS5,
        address: { host: "proxy.corp.com", port: 1080 },
        username: "domain\\admin",    // 可能包含敏感信息
        password: "SecretPassword123"
    }
});
```

**影响**:
- 凭据泄露
- 横向移动攻击

**修复建议**:
1. 使用后立即清除内存中的凭据
2. 不在日志中打印凭据
3. 考虑使用系统凭据管理器

---

### ⚠️ 风险 5: 整数溢出与缓冲区大小

| 属性 | 值 |
|------|-----|
| **风险等级** | 🟡 低-中 |
| **证据位置** | `socket_async_work.cpp` |

**问题描述**:
Socket 缓冲区大小从用户输入获取，可能导致整数溢出或过大分配。

```cpp
// socket_extra_options.cpp
void SetReceiveBufferSize(int size) {
    if (size > MAX_BUFFER_SIZE) {
        size = MAX_BUFFER_SIZE;  // 有限制
    }
    setsockopt(fd, SOL_SOCKET, SO_RCVBUF, &size, sizeof(size));
}
```

**触发条件**:
```typescript
udpSocket.setExtraOptions({
    receiveBufferSize: -1  // 整数溢出尝试
});
```

**影响**:
- 拒绝服务 (DoS)
- 内存耗尽

**修复建议**:
1. 对缓冲区大小进行严格范围检查
2. 使用无符号类型处理大小
3. 添加上限保护

---

## 4. 安全最佳实践

### 4.1 应用开发者建议

| 实践 | 说明 |
|------|------|
| ✅ 始终验证 SSL/TLS | 不跳过证书验证 |
| ✅ 使用证书锁定 | 对关键端点启用 certificatePinning |
| ✅ 限制超时时间 | 防止 Slowloris 攻击 |
| ✅ 清理敏感数据 | 使用后立即清除凭据 |
| ❌ 不跳过权限检查 | 确保声明 INTERNET 权限 |
| ❌ 不硬编码凭据 | 使用安全存储 |

### 4.2 配置建议

```typescript
// 安全配置示例
let request = http.createHttp();
request.request(url, {
    usingCache: false,           // 敏感数据禁用缓存
    connectTimeout: 10000,       // 10秒连接超时
    readTimeout: 30000,          // 30秒读取超时
    certificatePinning: {        // 证书锁定
        publicKeyHash: "sha256//...",
        hashAlgorithm: "SHA-256"
    },
    usingProxy: false           // 禁用代理（除非必要）
});
```

---

## 5. 安全相关文件索引

| 功能 | 文件路径 |
|------|----------|
| 权限检查 | `utils/common_utils/src/netstack_common_utils.cpp:145-186` |
| 上下文权限 | `utils/napi_utils/include/base_context.h` |
| 证书验证 | `frameworks/native/net_ssl/net_ssl_verify_cert.cpp` |
| TLS 上下文 | `frameworks/native/tls_socket/src/tls_context.cpp` |
| 证书锁定 | `utils/common_utils/src/netstack_common_utils.cpp:564-593` |
| 原子服务检查 | `utils/common_utils/src/netstack_common_utils.cpp:197-213` |
| SOCKS5 代理 | `frameworks/js/napi/proxy/src/socks5_passwd_method.cpp` |
| WebSocket 权限 | `frameworks/native/websocket_native/src/websocket_client.cpp:639` |

---

## 6. 审计范围说明

### 已检查范围
- ✅ N-API 接口入口点
- ✅ 权限校验机制
- ✅ 证书验证流程
- ✅ TLS/SSL 配置
- ✅ Socket 输入处理
- ✅ HTTP 请求构建
- ✅ 代理配置

### 未完全覆盖范围
- ⚠️ 第三方库 (curl, openssl) 内部漏洞
- ⚠️ 内核级 socket 实现
- ⚠️ 系统网络配置
- ⚠️ FFI/CJ 接口完整安全分析

---

## 相关文档
- [Architecture](01_Architecture.md)
- [N-API Reference](02_N-API_Reference.md)
- [Native API](03_Inner_API.md)
- [Build Targets](04_Build_Targets.md)
