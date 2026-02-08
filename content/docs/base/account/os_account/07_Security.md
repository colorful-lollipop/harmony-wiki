# 07_Security.md

# OpenHarmony os_account 安全风险评审

> 本文档基于代码分析，对 os_account 子系统进行安全风险评审。

---

## 1. 安全模型概述

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不可信区域                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  第三方应用 (HAP)                                         │   │
│  │  - 可能存在恶意应用                                        │   │
│  │  - 权限可能受限                                           │   │
│  │  - UID/Token 可能被伪造                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ IPC 边界                         │
│                              ▼                                   │
├─────────────────────────────────────────────────────────────────┤
│                      信任边界边界                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  框架层 (Frameworks)                                      │   │
│  │  - 权限校验                                              │   │
│  │  - 参数验证                                              │   │
│  │  - Token 验证                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ IPC 边界                         │
│                              ▼                                   │
├─────────────────────────────────────────────────────────────────┤
│                      信任边界内部                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  服务层 (Services)                                        │   │
│  │  - AccountMgrService (SA 200)                            │   │
│  │  - OsAccountManagerService                               │   │
│  │  - AppAccountManagerService                              │   │
│  │  - AccountIAMService                                     │   │
│  │  - DomainAccountManagerService                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  存储层                                                  │   │
│  │  - SQLite 数据库                                         │   │
│  │  - KV 存储                                               │   │
│  │  - 配置文件                                               │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| **权限校验** | `AccountPermissionManager` | AccessTokenKit 验证 |
| **系统应用校验** | `CheckSystemApp()` | Token 类型验证 |
| **SA 调用校验** | `CheckSaCall()` | Native Token 验证 |
| **Shell 调用校验** | `CheckShellCall()` | Shell Token 验证 |
| **UID 白名单** | `WHITE_LIST` | 特殊 UID 放行 |
| **Bundle 校验** | `BundleManagerAdapter` | 包名验证 |
| **Token 缓存** | `PrivilegeCacheManager` | 权限缓存验证 |

---

## 2. 攻击面清单

### 2.1 N-API 接口攻击面

| 接口 | 风险等级 | 攻击面描述 |
|------|----------|----------|
| `createOsAccount()` | 高 | 任意创建系统账号 |
| `removeOsAccount()` | 高 | 任意删除系统账号 |
| `setOsAccountName()` | 中 | 修改账号属性 |
| `setOsAccountConstraints()` | 高 | 修改账号约束 |
| `activateOsAccount()` | 高 | 切换活跃账号 |
| `addAccount()` (app) | 中 | 应用账号注入 |
| `setAssociatedData()` | 中 | 注入关联数据 |
| `authenticate()` | 中 | OAuth 认证劫持 |
| `setOAuthToken()` | 高 | OAuth 令牌注入 |

### 2.2 IPC 攻击面

| 接口 | 风险等级 | 攻击面描述 |
|------|----------|----------|
| `CreateOsAccount()` | 高 | 未授权的账号创建 |
| `RemoveOsAccount()` | 高 | 未授权的账号删除 |
| `SetOsAccountConstraints()` | 高 | 未授权的约束修改 |
| `AddCredential()` (IAM) | 高 | 未授权的凭据添加 |
| `DelUser()` (IAM) | 高 | 未授权的用户删除 |

### 2.3 文件/存储攻击面

| 路径 | 风险等级 | 攻击面描述 |
|------|----------|----------|
| `/data/service/el1/public/account/` | 高 | 账号数据存储目录 |
| `os_account_config.json` | 中 | 配置注入 |
| SQLite 数据库 | 高 | SQL 注入/数据篡改 |

### 2.4 权限提升攻击面

| 攻击路径 | 风险等级 | 描述 |
|----------|----------|------|
| 普通应用 → 系统应用 | 高 | 绕过 `CheckSystemApp()` |
| Shell 权限 → SA 权限 | 高 | 绕过 SA 调用校验 |
| Token 伪造 | 高 | AccessToken 伪造 |

---

## 3. 安全风险点

### 风险 1: 未授权的 OS 账号创建

**代码证据**:
```
文件: services/accountmgr/src/osaccount/os_account_manager_service.cpp
行号: 238
代码:
ErrCode result = AccountPermissionManager::CheckSystemApp();
```

**触发条件**:
1. 调用 `CreateOsAccount()` IPC 接口
2. 调用者不是系统应用
3. `CheckSystemApp()` 返回失败但错误码被忽略

**影响**:
- 任意应用可创建设备账号
- 可能导致账号数量耗尽
- 绕过账号数量限制（`maxOsAccountNum`）

**修复建议**:
```cpp
// 在 CheckSystemApp 失败时直接返回错误
ErrCode result = AccountPermissionManager::CheckSystemApp();
if (result != ERR_OK) {
    return result;  // 明确返回错误
}
```

---

### 风险 2: 未授权的 OS 账号删除

**代码证据**:
```
文件: services/accountmgr/src/osaccount/os_account_manager_service.cpp  
行号: 295
代码:
ErrCode result = AccountPermissionManager::CheckSystemApp();
if (result != ERR_OK) {
    return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
}
```

**触发条件**:
1. 调用 `RemoveOsAccount()` IPC 接口
2. 调用者没有 `MANAGE_LOCAL_ACCOUNTS` 权限
3. 边界条件：`saCreatedNum` 可能被绕过

**影响**:
- 任意应用可删除系统账号
- 可能导致设备无法正常登录
- 删除关键系统账号

**修复建议**:
```cpp
// 增强删除前的权限和约束检查
ErrCode RemoveOsAccount(int32_t localId) {
    // 1. 检查调用者权限
    ErrCode result = AccountPermissionManager::CheckSystemApp();
    if (result != ERR_OK) {
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
    
    // 2. 检查是否为系统创建的账号
    if (localId < saCreatedNum) {
        ACCOUNT_LOGE("Cannot remove system created account");
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
    
    return ERR_OK;
}
```

---

### 风险 3: 约束绕过

**代码证据**:
```
文件: services/accountmgr/src/osaccount/os_account_manager_service.cpp
行号: 2023
代码:
ErrCode checkResult = AccountPermissionManager::CheckSystemApp();
```

**触发条件**:
1. 调用 `SetOsAccountConstraints()` 接口
2. 约束检查在权限检查之后
3. 约束类型字符串未验证

**影响**:
- 注入非法约束类型
- 绕过系统约束机制
- 锁定用户账号

**修复建议**:
```cpp
// 约束类型白名单验证
const std::set<std::string> VALID_CONSTRAINTS = {
    "constraint.os.account.DISALLOW_MODIFY_ACCOUNTS",
    "constraint.os.account.DISALLOW_ADDING_ACCOUNTS",
    "constraint.os.account.DISALLOW_EXPANDED_METHODS"
};

ErrCode SetOsAccountConstraints(int32_t localId, 
    const std::vector<std::string>& constraints, bool enable) {
    // 验证约束类型
    for (const auto& constraint : constraints) {
        if (VALID_CONSTRAINTS.find(constraint) == VALID_CONSTRAINTS.end()) {
            ACCOUNT_LOGE("Invalid constraint type: %{public}s", constraint.c_str());
            return ERR_ACCOUNT_COMMON_INVALID_PARAM;
        }
    }
}
```

---

### 风险 4: OAuth 令牌注入

**代码证据**:
```
文件: services/accountmgr/src/appaccount/app_account_control_manager.cpp
行号: 1054
代码:
ErrCode result = AccountPermissionManager::VerifyPermission(GET_ALL_APP_ACCOUNTS);
```

**触发条件**:
1. 调用 `setOAuthToken()` 接口
2. 令牌内容未校验长度和格式
3. 跨账号令牌注入

**影响**:
- 注入超长令牌（DoS）
- 注入格式错误的令牌
- 冒充其他应用的令牌

**修复建议**:
```cpp
ErrCode SetOAuthToken(const std::string& name, 
    const std::string& authType, const std::string& token) {
    // 1. 令牌长度限制
    constexpr size_t MAX_TOKEN_LENGTH = 4096;
    if (token.size() > MAX_TOKEN_LENGTH) {
        ACCOUNT_LOGE("Token too long: %{public}zu", token.size());
        return ERR_ACCOUNT_COMMON_INVALID_PARAM;
    }
    
    // 2. 令牌格式验证（根据 authType）
    if (!ValidateTokenFormat(authType, token)) {
        ACCOUNT_LOGE("Invalid token format for authType: %{public}s", authType.c_str());
        return ERR_ACCOUNT_COMMON_INVALID_PARAM;
    }
}
```

---

### 风险 5: BundleName 伪造

**代码证据**:
```
文件: services/accountmgr/src/appaccount/app_account_manager_service.cpp
行号: 1159-1176
代码:
ErrCode AppAccountManagerService::GetBundleNameAndCallingUid(
    int32_t &callingUid, std::string &bundleName)
{
    ErrCode bundleRet = BundleManagerAdapter::GetInstance()
        ->GetNameForUid(callingUid, bundleName);
}
```

**触发条件**:
1. UID 验证依赖于 BundleManager
2. `BundleManagerAdapter::GetNameForUid()` 可能返回错误但被忽略
3. 恶意应用伪造调用 UID

**影响**:
- 绕过应用账号所有权检查
- 修改其他应用的账号数据
- 冒充合法应用进行 OAuth 授权

**修复建议**:
```cpp
ErrCode AppAccountManagerService::GetCallingInfo(
    int32_t &callingUid, std::string &bundleName, uint32_t &appIndex)
{
    callingUid = IPCSkeleton::GetCallingUid();
    
    // 1. UID 有效性检查
    constexpr int32_t MIN_APP_UID = 10000;
    constexpr int32_t MAX_APP_UID = 99999;
    if (callingUid < MIN_APP_UID || callingUid > MAX_APP_UID) {
        ACCOUNT_LOGE("Invalid calling UID: %{public}d", callingUid);
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
    
    // 2. BundleName 获取和验证
    ErrCode bundleRet = GetBundleNameAndCallingUid(callingUid, bundleName);
    if (bundleRet != ERR_OK || bundleName.empty()) {
        ACCOUNT_LOGE("Failed to get bundle name");
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
}
```

---

### 风险 6: PIN 输入器注入

**代码证据**:
```
文件: services/accountmgr/src/account_iam/account_iam_service.cpp
行号: 684
代码:
result = AccountPermissionManager::VerifyPermission(ACCESS_PIN_AUTH);
```

**触发条件**:
1. 调用 `RegisterPinInputer()` 接口
2. PIN 输入器回调未验证
3. 恶意 PIN 输入器可窃取用户 PIN 码

**影响**:
- 注入恶意 PIN 输入器
- 窃取用户认证凭据
- 绕过 PIN 验证

**修复建议**:
```cpp
ErrCode RegisterPinInputer(sptr<IInputer> inputer) {
    // 1. 权限检查
    ErrCode result = AccountPermissionManager::VerifyPermission(ACCESS_PIN_AUTH);
    if (result != ERR_OK) {
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
    
    // 2. 输入器验证
    if (inputer == nullptr) {
        ACCOUNT_LOGE("Invalid inputer");
        return ERR_ACCOUNT_COMMON_INVALID_PARAM;
    }
    
    // 3. 输入器白名单或签名验证（可选）
    // CheckInputerSignature(inputer);
    
    // 4. 只保留最后一个注册，移除旧的
    UnregisterInputer();
    pinInputer_ = inputer;
    
    return ERR_OK;
}
```

---

### 风险 7: 分布式账号信息泄露

**代码证据**:
```
文件: services/accountmgr/src/ohos_account_manager.cpp
行号: 91-102
代码:
uint64_t fullTokenId = IPCSkeleton::GetCallingFullTokenId();
Security::AccessToken::ATokenTypeEnum tokenType = 
    Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
isSystemApp = Security::AccessToken::TokenIdKit
    ::IsSystemAppByFullTokenID(fullTokenId);
```

**触发条件**:
1. 调用 `QueryOhosAccountInfo()` 接口
2. 非系统应用可能通过其他方式获取信息
3. Token 验证不完整

**影响**:
- 泄露分布式账号身份信息
- 跨设备追踪用户

**修复建议**:
```cpp
ErrCode QueryOhosAccountInfo(OhosAccountInfo& info) {
    // 1. 强制权限检查
    if (!HasAccountRequestPermission(GET_DISTRIBUTED_ACCOUNTS)) {
        ACCOUNT_LOGE("Caller does not have permission");
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
    
    // 2. 系统应用验证
    if (!isSystemApp) {
        ACCOUNT_LOGE("Caller is not system app");
        return ERR_ACCOUNT_COMMON_NOT_SYSTEM_APP_ERROR;
    }
    
    // 3. 敏感信息脱敏（非系统应用获取时）
    if (!isSystemApp) {
        info = SanitizeAccountInfo(info);
    }
}
```

---

## 4. 权限清单

### 4.1 系统权限

| 权限名 | 用途 | 保护的操作 |
|--------|------|-----------|
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 管理本地账号 | 创建、删除、修改账号 |
| `ohos.permission.GET_LOCAL_ACCOUNTS` | 获取本地账号 | 查询账号信息 |
| `ohos.permission.MANAGE_DISTRIBUTED_ACCOUNTS` | 管理分布式账号 | 设置分布式账号 |
| `ohos.permission.GET_DISTRIBUTED_ACCOUNTS` | 获取分布式账号 | 查询分布式账号 |
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 | 同步开关 |
| `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS` | 跨账号交互 | 订阅事件 |
| `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS_EXTENSION` | 跨账号扩展 | 扩展操作 |
| `ohos.permission.GET_DOMAIN_ACCOUNTS` | 获取域账号 | 查询域账号 |
| `ohos.permission.MANAGE_EDM_POLICY` | 管理 EDM 策略 | 设置策略 |

### 4.2 IAM 权限

| 权限名 | 用途 | 保护的操作 |
|--------|------|-----------|
| `ohos.permission.ACCESS_USER_AUTH_INTERNAL` | 内部用户认证 | 认证、获取属性 |
| `ohos.permission.MANAGE_USER_IDM` | 管理用户 IDM | 添加/删除凭据、用户 |
| `ohos.permission.USE_USER_IDM` | 使用用户 IDM | 查询凭据、已注册 ID |
| `ohos.permission.ACCESS_PIN_AUTH` | 访问 PIN 认证 | 注册 PIN 输入器 |

### 4.3 应用权限

| 权限名 | 用途 | 保护的操作 |
|--------|------|-----------|
| `ohos.permission.GET_ALL_APP_ACCOUNTS` | 获取所有应用账号 | 获取可访问账号列表 |
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 | 同步开关 |

---

## 5. 安全最佳实践

### 5.1 输入验证

```cpp
// 字符串参数长度限制
constexpr size_t MAX_NAME_LENGTH = 256;
constexpr size_t MAX_EXTRA_INFO_LENGTH = 4096;
constexpr size_t MAX_CONSTRAINT_LENGTH = 128;

// 参数非空检查
if (name.empty()) {
    return ERR_ACCOUNT_COMMON_INVALID_PARAM;
}

// 特殊字符过滤
if (ContainsInvalidChars(name)) {
    return ERR_ACCOUNT_COMMON_INVALID_PARAM;
}
```

### 5.2 权限校验模式

```cpp
// 标准权限检查模式
ErrCode ProtectedOperation(const Request& req) {
    // 1. 权限检查
    ErrCode result = AccountPermissionManager::VerifyPermission(req.permission);
    if (result != ERR_OK) {
        ACCOUNT_LOGE("Permission denied: %{public}s", req.permission.c_str());
        return ERR_ACCOUNT_COMMON_PERMISSION_DENIED;
    }
    
    // 2. 系统应用检查（如果需要）
    if (requireSystemApp) {
        result = AccountPermissionManager::CheckSystemApp();
        if (result != ERR_OK) {
            ACCOUNT_LOGE("Caller is not system app");
            return ERR_ACCOUNT_COMMON_NOT_SYSTEM_APP_ERROR;
        }
    }
    
    // 3. 参数验证
    if (!ValidateParams(req)) {
        return ERR_ACCOUNT_COMMON_INVALID_PARAM;
    }
    
    return ERR_OK;
}
```

### 5.3 审计日志

```cpp
// 操作审计日志
ACCOUNT_LOGI("CreateOsAccount: localId=%{public}d, name=%{public}s, "
    "callerUid=%{public}d, callerTokenId=%{public}llu",
    localId, name.c_str(), 
    IPCSkeleton::GetCallingUid(),
    IPCSkeleton::GetCallingFullTokenID());
```

---

## 6. 检查范围与局限性

### 6.1 已检查范围

| 类别 | 状态 | 说明 |
|------|------|------|
| N-API 参数校验 | ✅ 已检查 | 基础验证存在，部分可绕过 |
| IPC 权限校验 | ✅ 已检查 | 核心权限检查到位 |
| 权限定义 | ✅ 已检查 | privileges.json 完整 |
| Token 验证 | ✅ 已检查 | AccessTokenKit 使用正确 |
| BundleName 校验 | ⚠️ 部分 | 依赖 BundleManager |
| 数据存储安全 | ❌ 未检查 | SQLite/KV 存储加密 |

### 6.2 未检查范围

| 范围 | 原因 |
|------|------|
| TEE/SE 安全 | 需要专门的安全评估 |
| 密钥管理 | Huks 集成需要单独审计 |
| 分布式同步 | 跨设备安全需要端到端分析 |
| 测试代码 | 按要求忽略 |

---

## 7. 相关文档

- [概述](./01_Overview.md)
- [N-API 接口](./03_NAPI_Interfaces.md)
- [服务与 IPC](./05_Service_IPC.md)
- [常见问题与调试](./09_FAQ_Debug.md)
