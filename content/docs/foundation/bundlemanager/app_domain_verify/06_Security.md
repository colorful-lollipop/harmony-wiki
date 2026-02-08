# 安全风险评审

> 本文档对 app_domain_verify 部件进行安全风险分析，包括攻击面、信任边界和可被利用点。

## 1. 威胁模型

### 1.1 外部输入

| 输入源 | 类型 | 处理方式 |
|-------|------|---------|
| **HTTPS 链接** | 用户点击/外部传入 | URL 解析、域名提取 |
| **assetlinks.json** | 域名服务器响应 | JSON 解析、签名验证 |
| **应用配置** | module.json5 | SkillUri 解析 |
| **用户操作** | 点击链接 | Want 构造 |

### 1.2 敏感操作

| 操作 | 风险等级 | 说明 |
|-----|---------|------|
| **网络请求** | 高 | 向外部域名服务器发起 HTTP 请求 |
| **签名验证** | 高 | 验证应用签名是否在白名单中 |
| **数据库读写** | 中 | 持久化校验状态 |
| **IPC 通信** | 中 | 跨进程数据传输 |
| **文件操作** | 低 | 配置读取 |

## 2. 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   受信任区域                         │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐      │   │
│  │  │ Manager   │  │  Agent    │  │   RDB     │      │   │
│  │  │ Service   │  │  Service  │  │  Database │      │   │
│  │  │ (SA 6200) │  │ (SA 6201)│  │           │      │   │
│  │  └─────┬─────┘  └─────┬─────┘  └───────────┘      │   │
│  │        │              │                            │   │
│  │        └──────────────┴────────────────────────────┘   │   │
│  │                       │                                │   │
│  │        ┌──────────────┴────────────────────────────┐   │   │
│  │        │            内部模块                       │   │   │
│  │        │  Verifier / Extension / Common           │   │   │
│  │        └──────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│              ──────────────┼──────────────                │
│                            │ 不受信任                       │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   不受信任区域                       │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐      │   │
│  │  │  用户     │  │  域名     │  │  其他     │      │   │
│  │  │  输入     │  │  服务器   │  │  应用     │      │   │
│  │  └───────────┘  └───────────┘  └───────────┘      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 3. 攻击面分析

### 3.1 网络攻击面

| 攻击面 | 描述 | 风险 |
|-------|------|-----|
| **HTTP 请求** | 向域名服务器请求 assetlinks.json | 中-高 |
| **DNS 解析** | 域名解析可能被劫持 | 高 |
| **中间人攻击** | HTTP 流量可能被篡改 | 高 |
| **恶意服务器** | 返回恶意 JSON 或签名 | 中 |

### 3.2 数据注入攻击面

| 攻击面 | 描述 | 风险 |
|-------|------|-----|
| **JSON 注入** | 解析恶意构造的 assetlinks.json | 中 |
| **URL 注入** | 恶意构造的 SkillUri 配置 | 中 |
| **数据库注入** | RDB 操作中的 SQL 注入 | 低 |

### 3.3 IPC 攻击面

| 攻击面 | 描述 | 风险 |
|-------|------|-----|
| **权限绕过** | 绕过权限检查调用敏感接口 | 中 |
| **数据篡改** | IPC 参数被篡改 | 低 |
| **拒绝服务** | 大量请求导致服务不可用 | 中 |

## 4. 可被利用点

### 4.1 🔴 高风险

#### R1: HTTP 响应大小未限制可能导致内存耗尽

**证据位置**: `frameworks/common/src/httpsession/i_http_task.cpp` (OnDataReceive 回调)

**问题描述**:
HTTP 响应回调中未对响应大小进行硬性限制，可能导致恶意服务器发送超大响应耗尽设备内存。

**触发条件**:
1. 攻击者控制域名服务器
2. 返回超大 assetlinks.json (>20KB)
3. 多次触发校验请求

**影响**:
- 设备内存耗尽
- 服务崩溃
- 拒绝服务 (DoS)

**修复建议**:
```cpp
// 在 i_http_task.cpp 中添加响应大小限制
constexpr size_t MAX_RESPONSE_SIZE = 20 * 1024; // 20KB

void OnDataReceive(const char* data, size_t len) {
    if (currentSize_ + len > MAX_RESPONSE_SIZE) {
        // 断开连接，返回错误
        Disconnect();
        OnFail("Response too large");
        return;
    }
    // 正常处理
}
```

---

#### R2: 域名白名单可被动态更新但缺乏验证

**证据位置**: `frameworks/common/src/config/white_list_config_mgr.cpp`

**问题描述**:
`UpdateWhiteListUrls()` 接口允许动态更新域名白名单，但更新请求来源验证不严格。

**触发条件**:
1. 攻击者获得系统权限
2. 调用 `UpdateWhiteListUrls()` 添加恶意域名

**影响**:
- 恶意域名被加入白名单
- 绕过域名校验
- 安全机制失效

**修复建议**:
```cpp
bool WhiteListConfigMgr::UpdateWhiteListUrls(const std::vector<std::string>& urls) {
    // 1. 验证调用者权限
    if (!PermissionManager::IsSystemAppCall()) {
        APP_DOMAIN_VERIFY_HILOGE("Not authorized to update whitelist");
        return false;
    }
    
    // 2. 验证 URL 格式
    for (const auto& url : urls) {
        if (!DomainUrlUtil::IsValidUrl(url)) {
            APP_DOMAIN_VERIFY_HILOGE("Invalid URL in whitelist: %{public}s", url.c_str());
            return false;
        }
    }
    
    // 3. 持久化前签名验证
    // ...
}
```

---

### 4.2 🟠 中风险

#### R3: 重试机制可能导致资源耗尽

**证据位置**: `frameworks/verifier/src/verify_task.cpp` (Exponential backoff)

**问题描述**:
校验失败后的指数退避重试机制，虽然有最大重试次数限制，但状态 `FORBIDDEN_FOREVER` 之前的失败可能持续影响设备性能。

**触发条件**:
1. 大量域名校验持续失败
2. 恶意服务器返回特定错误码

**影响**:
- FFRT 任务队列堆积
- 设备性能下降

**修复建议**:
```cpp
// 添加全局重试次数限制
constexpr int MAX_GLOBAL_RETRIES = 100;
static std::atomic<int> g_totalRetries = 0;

void VerifyTask::HandleFailure(int errorCode) {
    if (g_totalRetries >= MAX_GLOBAL_RETRIES) {
        // 停止所有重试，记录告警
        ReportAnomaly("Too many retry attempts");
        return;
    }
    // 正常重试逻辑
}
```

---

#### R4: Short URL 转换缺乏速率限制

**证据位置**: `interfaces/inner_api/client/src/app_domain_verify_mgr_client.cpp` (ConvertFromShortUrl)

**问题描述**:
Short URL 转换接口没有请求频率限制，可能被用于发起 DoS 攻击。

**触发条件**:
1. 短时间内大量 Short URL 转换请求
2. 恶意应用循环调用

**影响**:
- 服务性能下降
- 网络带宽消耗

**修复建议**:
```cpp
// 添加滑动窗口速率限制
class RateLimiter {
    static constexpr int MAX_REQUESTS_PER_MINUTE = 100;
    std::map<uint64_t, std::chrono::steady_clock::time_point> requestHistory_;
    
    bool TryAcquire(const std::string& caller) {
        auto now = std::chrono::steady_clock::now();
        // 清理过期记录
        // 检查当前请求数
        // 返回是否可以处理
    }
};
```

---

#### R5: 延迟链接 (Deferred Link) 可能泄露用户行为

**证据位置**: `services/src/manager/deferred_link/deferred_link_mgr.cpp`

**问题描述**:
延迟链接存储用户点击但未安装应用时的链接，可能被用于追踪用户行为。

**触发条件**:
1. 用户点击多个应用的延迟链接
2. 链接被持久化存储

**影响**:
- 用户隐私泄露
- 行为追踪

**修复建议**:
```cpp
// 添加链接过期机制
constexpr time_t DEFERRED_LINK_EXPIRY = 7 * 24 * 60 * 60; // 7天

void DeferredLinkMgr::StoreLink(const std::string& link, const std::string& bundleName) {
    // 1. 检查是否超出存储限制
    if (GetStoredLinkCount() >= MAX_DEFERRED_LINKS) {
        DeleteOldestLink();
    }
    
    // 2. 添加过期时间戳
    DeferredLink entry = {
        .link = link,
        .bundleName = bundleName,
        .timestamp = std::time(nullptr),
        .expiry = std::time(nullptr) + DEFERRED_LINK_EXPIRY
    };
    
    // 3. 加密存储
    EncryptAndStore(entry);
}
```

---

### 4.2 🟡 低风险

#### R6: 日志可能泄露敏感信息

**证据位置**: `frameworks/common/include/app_domain_verify_hilog.h`

**问题描述**:
调试日志可能输出敏感信息如 URL、签名等。

**触发条件**:
1. 日志级别设置为 DEBUG
2. 日志被导出到外部

**影响**:
- 敏感信息泄露

**修复建议**:
```cpp
// 使用脱敏日志宏
#define APP_DOMAIN_VERIFY_HILOG_D(domain, format, ...) \
    do { \
        if (IsDebugEnabled()) { \
            std::string masked = MaskStr(format); \
            APP_DOMAIN_VERIFY_HILOGI(domain, masked, ##__VA_ARGS__); \
        } \
    } while (0)
```

---

#### R7: 应用标识符验证不严格

**证据位置**: `frameworks/verifier/src/domain_verifier.cpp`

**问题描述**:
`appIdentifier` 字段验证不够严格，可能接受格式错误的值。

**触发条件**:
1. 应用配置中包含恶意构造的 appIdentifier
2. 服务端接受并处理

**影响**:
- JSON 解析错误
- 潜在注入攻击

**修复建议**:
```cpp
bool DomainVerifier::ValidateAppIdentifier(const std::string& appIdentifier) {
    // 1. 长度检查
    if (appIdentifier.empty() || appIdentifier.length() > 256) {
        return false;
    }
    
    // 2. 格式检查（允许字符）
    static const std::regex pattern("^[a-zA-Z0-9.-]+$");
    if (!std::regex_match(appIdentifier, pattern)) {
        return false;
    }
    
    return true;
}
```

---

## 5. 安全机制评估

### 5.1 已有安全机制

| 机制 | 实现位置 | 有效性 |
|-----|---------|-------|
| **HTTPS 传输** | netstack | ✅ 有效 |
| **签名验证** | DomainVerifier | ✅ 有效 |
| **权限校验** | PermissionManager | ✅ 有效 |
| **IPC 权限控制** | IRemoteBroker | ✅ 有效 |
| **数据库加密** | RDB | ✅ 有效 |
| **响应大小限制** | i_http_task.cpp | ⚠️ 部分实现 |

### 5.2 缺失安全机制

| 机制 | 建议 | 优先级 |
|-----|------|-------|
| **HTTP 响应大小限制** | 硬性限制 20KB | 🔴 高 |
| **速率限制** | 防止 DoS 攻击 | 🟠 中 |
| **URL 白名单签名** | 更新时验证来源 | 🟠 中 |
| **日志脱敏** | 敏感信息脱敏 | 🟡 低 |
| **输入格式验证** | 严格验证 appIdentifier | 🟡 低 |

## 6. 安全加固建议

### 6.1 网络安全

1. **强制 HTTPS**
   ```cpp
   // 确保只使用 HTTPS
   if (url.rfind("https://", 0) != 0) {
       return ERROR_INVALID_URL;
   }
   ```

2. **证书固定 (Certificate Pinning)**
   ```cpp
   // 对于关键域名，固定证书指纹
   constexpr const char* PINNED_CERT_FINGERPRINT = "...";
   ```

3. **DNS over HTTPS**
   ```cpp
   // 使用 DoH 防止 DNS 劫持
   ```

### 6.2 数据安全

1. **敏感数据加密存储**
   ```cpp
   // 数据库字段加密
   std::string EncryptSensitiveData(const std::string& data);
   std::string DecryptSensitiveData(const std::string& encrypted);
   ```

2. **数据脱敏**
   ```cpp
   // 日志输出时脱敏
   std::string MaskStr(const std::string& input);
   ```

### 6.3 访问控制

1. **最小权限原则**
   ```cpp
   // 限制接口访问权限
   if (!PermissionManager::IsSACall() && 
       !PermissionManager::IsSystemAppCall()) {
       return ERROR_PERMISSION_DENIED;
   }
   ```

2. **接口调用审计**
   ```cpp
   // 记录敏感操作
   ApiEventReporter::ReportOperation("VerifyDomain", callerInfo);
   ```

## 7. 相关文档

| 文档 | 链接 |
|-----|------|
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| Inner API | [02_Inner_API.md](./02_Inner_API.md) |
| N-API | [03_N_API.md](./03_N_API.md) |
| GN 构建 | [04_GN_Build.md](./04_GN_Build.md) |
