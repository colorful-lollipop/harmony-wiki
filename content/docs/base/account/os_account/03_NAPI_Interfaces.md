# 03_NAPI_Interfaces.md

# OpenHarmony os_account N-API 对外接口文档

> 本文档描述 os_account 子系统对外暴露的 JavaScript/TypeScript API。

---

## 1. 模块注册概览

### 1.1 N-API 模块清单

| 模块名 | 命名空间 | 入口文件 | 聚合子模块 |
|--------|----------|----------|-----------|
| **account.osAccount** | `account.osAccount` | `napi_init.cpp` | OsAccount, AccountIAM, DomainAccount, Authorization |
| **account.appAccount** | `account.appAccount` | `napi_app_account_module.cpp` | AppAccountManager, Constants, Authenticator |

### 1.2 注册模式

所有模块遵循标准 N-API 注册模式：

```cpp
// 模块定义
static napi_module _module = {
    .nm_version = 1,
    .nm_register_func = Init,
    .nm_modname = "account.xxx",
};

// 构造函数注册
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&_module);
}
```

---

## 2. account.osAccount 模块

**入口文件**: `interfaces/kits/napi/osaccount/src/napi_init.cpp`  
**命名空间**: `account.osAccount`

### 2.1 OsAccount 子模块

**实现文件**: `interfaces/kits/napi/osaccount/src/napi_os_account.cpp`

#### 导出函数清单

| JS 函数名 | C++ 实现 | 说明 |
|-----------|----------|------|
| `queryOsAccountById` | `QueryOsAccountById` | 根据 ID 查询 OS 账号 |
| `removeOsAccount` | `RemoveOsAccount` | 删除 OS 账号 |
| `setOsAccountName` | `SetOsAccountName` | 设置 OS 账号名称 |
| `setOsAccountConstraints` | `SetOsAccountConstraints` | 设置账号约束 |
| `activateOsAccount` | `ActivateOsAccount` | 激活 OS 账号 |
| `deactivateOsAccount` | `DeactivateOsAccount` | 停用 OS 账号 |
| `createOsAccount` | `CreateOsAccount` | 创建 OS 账号 |
| `createOsAccountForDomain` | `CreateOsAccountForDomain` | 为域账号创建 OS 账号 |
| `getCreatedOsAccountsCount` | `GetCreatedOsAccountsCount` | 获取已创建账号数量 |
| `getOsAccountCount` | `GetOsAccountCount` | 获取账号总数 |
| `getDistributedVirtualDeviceId` | `GetDistributedVirtualDeviceId` | 获取分布式虚拟设备 ID |
| `queryDistributedVirtualDeviceId` | `QueryDistributedVirtualDeviceId` | 查询分布式虚拟设备 ID |
| `getOsAccountAllConstraints` | `GetOsAccountAllConstraints` | 获取所有账号约束 |
| `getOsAccountConstraints` | `GetOsAccountConstraints` | 获取账号约束 |
| `getOsAccountLocalIdFromProcess` | `GetOsAccountLocalIdFromProcess` | 从进程获取本地 ID |
| `queryOsAccountLocalIdFromProcess` | `QueryOsAccountLocalIdFromProcess` | 查询进程本地 ID |
| `getOsAccountLocalId` | `QueryOsAccountLocalIdFromProcess` | 获取本地 ID（别名） |
| `queryAllCreatedOsAccounts` | `QueryAllCreatedOsAccounts` | 查询所有已创建账号 |
| `queryOsAccountConstraintSourceTypes` | `QueryOsAccountConstraintSourceTypes` | 查询约束来源类型 |
| `getOsAccountConstraintSourceTypes` | `QueryOsAccountConstraintSourceTypes` | 获取约束来源类型 |
| `queryActivatedOsAccountIds` | `QueryActivatedOsAccountIds` | 查询已激活账号 ID |
| `getActivatedOsAccountIds` | `GetActivatedOsAccountIds` | 获取已激活账号 ID |
| `getActivatedOsAccountLocalIds` | `GetActivatedOsAccountIds` | 获取已激活本地 ID |
| `getForegroundOsAccountLocalId` | `GetForegroundOsAccountLocalId` | 获取前台账号本地 ID |
| `getForegroundOsAccountDisplayId` | `GetForegroundOsAccountDisplayId` | 获取前台账号显示 ID |
| `getOsAccountProfilePhoto` | `GetOsAccountProfilePhoto` | 获取账号头像 |
| `getOsAccountName` | `GetOsAccountName` | 获取账号名称 |
| `queryCurrentOsAccount` | `QueryCurrentOsAccount` | 查询当前账号 |
| `getCurrentOsAccount` | `GetCurrentOsAccount` | 获取当前账号 |
| `getOsAccountLocalIdFromUid` | `GetOsAccountLocalIdFromUid` | 从 UID 获取本地 ID |
| `getOsAccountLocalIdForUid` | `GetOsAccountLocalIdForUid` | 为 UID 获取本地 ID |
| `getOsAccountLocalIdForUidSync` | `GetOsAccountLocalIdForUidSync` | 同步获取本地 ID |
| `getBundleIdFromUid` | `GetBundleIdFromUid` | 从 UID 获取包名 |
| `getBundleIdForUid` | `GetBundleIdFromUid` | 为 UID 获取包名 |
| `getBundleIdForUidSync` | `GetBundleIdForUidSync` | 同步获取包名 |
| `getOsAccountLocalIdFromDomain` | `GetOsAccountLocalIdFromDomain` | 从域获取本地 ID |
| `queryOsAccountLocalIdFromDomain` | `QueryOsAccountLocalIdFromDomain` | 查询域本地 ID |
| `getOsAccountLocalIdForDomain` | `QueryOsAccountLocalIdFromDomain` | 为域获取本地 ID |
| `setOsAccountProfilePhoto` | `SetOsAccountProfilePhoto` | 设置账号头像 |
| `queryMaxOsAccountNumber` | `QueryMaxOsAccountNumber` | 查询最大账号数 |
| `queryMaxLoggedInOsAccountNumber` | `QueryMaxLoggedInOsAccountNumber` | 查询最大登录数 |
| `isOsAccountActived` | `IsOsAccountActived` | 账号是否已激活 |
| `checkOsAccountActivated` | `CheckOsAccountActivated` | 检查账号激活状态 |
| `isOsAccountConstraintEnable` | `IsOsAccountConstraintEnable` | 约束是否启用 |
| `checkConstraintEnabled` | `CheckConstraintEnabled` | 检查约束启用 |
| `checkOsAccountConstraintEnabled` | `CheckConstraintEnabled` | 检查账号约束 |
| `getOsAccountTypeFromProcess` | `GetOsAccountTypeFromProcess` | 从进程获取账号类型 |
| `getOsAccountType` | `GetOsAccountType` | 获取账号类型 |
| `isMultiOsAccountEnable` | `IsMultiOsAccountEnable` | 是否启用多账号 |
| `checkMultiOsAccountEnabled` | `CheckMultiOsAccountEnabled` | 检查多账号启用 |
| `isOsAccountVerified` | `IsOsAccountVerified` | 账号是否已验证 |
| `isTestOsAccount` | `IsTestOsAccount` | 是否为测试账号 |

---

### 2.2 AccountIAM 子模块

**实现文件**: `interfaces/kits/napi/account_iam/src/napi_account_iam_*.cpp`

#### UserAuth 类

| JS 方法 | 说明 | 权限要求 |
|---------|------|----------|
| `getVersion()` | 获取版本 | - |
| `getAvailableStatus(authType, level)` | 获取可用状态 | `ohos.permission.USE_USER_IDM` |
| `getProperty({authType, credType})` | 获取属性 | `ohos.permission.USE_USER_IDM` |
| `getPropertyByCredentialId({credId, authType})` | 按凭据 ID 获取属性 | `ohos.permission.USE_USER_IDM` |
| `setProperty({authType, credType, newSecret})` | 设置属性 | `ohos.permission.MANAGE_USER_IDM` |
| `auth({authType, challenge, credType, authTrustLevel})` | 认证 | `ohos.permission.ACCESS_USER_AUTH_INTERNAL` |
| `authUser({userId, authType, authTrustLevel})` | 用户认证 | `ohos.permission.ACCESS_USER_AUTH_INTERNAL` |
| `cancelAuth(sessionId)` | 取消认证 | - |
| `prepareRemoteAuth()` | 准备远程认证 | - |

#### UserIdentityManager 类

| JS 方法 | 说明 | 权限要求 |
|---------|------|----------|
| `openSession()` | 打开会话 | - |
| `addCredential({credType, credSecret, authTrustLevel})` | 添加凭据 | `ohos.permission.MANAGE_USER_IDM` |
| `updateCredential({credType, oldSecret, newSecret, authTrustLevel})` | 更新凭据 | `ohos.permission.MANAGE_USER_IDM` |
| `closeSession()` | 关闭会话 | - |
| `cancel()` | 取消操作 | - |
| `delUser({userId, clearCred)` | 删除用户 | `ohos.permission.MANAGE_USER_IDM` |
| `delCred({credId, authTrustLevel})` | 删除凭据 | `ohos.permission.MANAGE_USER_IDM` |
| `getAuthInfo({credId, authType})` | 获取认证信息 | `ohos.permission.USE_USER_IDM` |
| `getEnrolledId(authType)` | 获取已注册 ID | `ohos.permission.USE_USER_IDM` |
| `onCredentialChanged(type, callback)` | 凭据变更订阅 | - |
| `offCredentialChanged(callback)` | 取消凭据变更订阅 | - |

#### PINAuth 类

| JS 方法 | 说明 |
|---------|------|
| `constructor(inputer)` | 构造函数 |
| `registerInputer(inputer)` | 注册输入器 |
| `unregisterInputer()` | 注销输入器 |

#### InputerManager 类

| JS 方法 | 说明 |
|---------|------|
| `registerInputer(inputer)` | 注册输入器 |
| `unregisterInputer(inputer)` | 注销输入器 |

---

### 2.3 DomainAccount 子模块

**实现文件**: `interfaces/kits/napi/domain_account/src/napi_domain_account_*.cpp`

#### DomainAccountManager 类

| JS 方法 | 说明 |
|---------|------|
| `registerPlugin(plugin)` | 注册插件 |
| `unregisterPlugin()` | 注销插件 |
| `auth({domain, account, callback})` | 认证 |
| `authWithPopup(domain, account)` | 弹窗认证 |
| `hasAccount(domain, account)` | 检查账号是否存在 |
| `updateAccountToken(domain, account, token)` | 更新账号令牌 |
| `isAuthenticationExpired(domain, account)` | 认证是否过期 |
| `getAccessToken(domain, account)` | 获取访问令牌 |
| `getAccountInfo(domain, account)` | 获取账号信息 |
| `updateAccountInfo(domain, account, accountInfo)` | 更新账号信息 |

---

### 2.4 Authorization 子模块

**实现文件**: `interfaces/kits/napi/authorization/src/napi_authorization_*.cpp`

#### AuthorizationManager 类

| JS 方法 | 说明 |
|---------|------|
| `acquireAuthorization(authType, callerCallback)` | 获取授权 |
| `getAuthorizationManager()` | 获取授权管理器 |

#### AuthorizationResultCode 枚举

| 值 | 说明 |
|-----|------|
| `AUTHORIZATION_SUCCESS` | 授权成功 |
| `AUTHORIZATION_CANCELED` | 授权取消 |
| `AUTHORIZATION_CANCELED_BY_USER` | 用户取消 |
| `AUTHORIZATION_MANAGER_ERROR` | 管理器错误 |

---

## 3. account.appAccount 模块

**入口文件**: `interfaces/kits/napi/appaccount/src/napi_app_account_module.cpp`  
**命名空间**: `account.appAccount`

### 3.1 AppAccountManager 类

**实现文件**: `interfaces/kits/napi/appaccount/src/napi_app_account.cpp`

| JS 方法 | 说明 | 权限要求 |
|---------|------|----------|
| `addAccount(name, extraInfo)` | 添加账号 | - |
| `addAccountImplicitly(owner, authType, options, callback)` | 隐式添加 | - |
| `deleteAccount(name)` | 删除账号 | - |
| `disableAppAccess(name, bundleName)` | 禁用应用访问 | - |
| `enableAppAccess(name, bundleName)` | 启用应用访问 | - |
| `checkAppAccountSyncEnable(name)` | 检查同步启用 | - |
| `setAccountCredential(name, credentialType, credential)` | 设置凭据 | - |
| `setAccountExtraInfo(name, extraInfo)` | 设置附加信息 | - |
| `setAppAccountSyncEnable(name, isEnable)` | 设置同步状态 | - |
| `setAssociatedData(name, key, value)` | 设置关联数据 | - |
| `authenticate(name, owner, authType, options, callback)` | 认证 | - |
| `getAllAccessibleAccounts()` | 获取所有可访问账号 | `ohos.permission.GET_ALL_APP_ACCOUNTS` |
| `getAllAccounts(owner)` | 获取指定所有账号 | `ohos.permission.GET_ALL_APP_ACCOUNTS` |
| `getAccountCredential(name, credentialType)` | 获取凭据 | - |
| `getAccountExtraInfo(name)` | 获取附加信息 | - |
| `getAssociatedData(name, key)` | 获取关联数据 | - |
| `getOAuthToken(name, owner, authType)` | 获取 OAuth 令牌 | - |
| `setOAuthToken(name, authType, token)` | 设置 OAuth 令牌 | - |
| `deleteOAuthToken(name, owner, authType, token)` | 删除 OAuth 令牌 | - |
| `getAuthenticatorInfo(owner)` | 获取认证器信息 | - |
| `getAllOAuthTokens(name, owner)` | 获取所有 OAuth 令牌 | - |
| `getOAuthList(name, authType)` | 获取 OAuth 列表 | - |
| `setOAuthTokenVisibility(name, authType, bundleName, isVisible)` | 设置令牌可见性 | - |
| `checkOAuthTokenVisibility(name, authType, bundleName)` | 检查令牌可见性 | - |
| `getAuthenticatorCallback(sessionId)` | 获取认证器回调 | - |
| `on(type, owners, callback)` | 订阅变更 | - |
| `off(type, callback)` | 取消订阅 | - |
| `checkAppAccess(name, bundleName)` | 检查应用访问 | - |
| `checkAccountLabels(name, owner, labels)` | 检查账号标签 | - |
| `setAuthenticatorProperties(owner, options, callback)` | 设置认证器属性 | - |
| `verifyCredential(name, owner, options, callback)` | 验证凭据 | - |
| `selectAccountsByOptions(options)` | 根据选项选择账号 | - |

### 3.2 AppAccountInfo 类

| 属性 | 说明 |
|------|------|
| `name` | 账号名称 |
| `owner` | 所属包名 |
| `extraInfo` | 附加信息 |

### 3.3 OAuthTokenInfo 类

| 属性 | 说明 |
|------|------|
| `token` | OAuth 令牌 |
| `authType` | 认证类型 |

### 3.4 AuthenticatorInfo 类

| 属性 | 说明 |
|------|------|
| `owner` | 所属包名 |
| `iconId` | 图标 ID |
| `labelId` | 标签 ID |

### 3.5 AuthenticatorCallback 类

| 回调方法 | 说明 |
|----------|------|
| `onResult(code, result)` | 认证结果 |
| `onRequestRedirected(request)` | 请求重定向 |
| `onRequestContinued()` | 请求继续 |

### 3.6 Authenticator 类

| JS 方法 | 说明 |
|---------|------|
| `addAccountImplicitly(authType, callerBundleName, options, callback)` | 隐式添加账号 |
| `authenticate(name, authType, callerBundleName, options, callback)` | 认证 |
| `verifyCredential(name, options, callback)` | 验证凭据 |
| `setProperties(options, callback)` | 设置属性 |
| `checkAccountLabels(name, labels, callback)` | 检查标签 |
| `isAccountRemovable(name, callback)` | 检查是否可删除 |
| `getRemoteObject()` | 获取远程对象 |

### 3.7 Constants 常量

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `Action` | - | 操作类型枚举 |
| `CredentialType` | - | 凭据类型枚举 |
| `AuthType` | - | 认证类型枚举 |
| `EnableMode` | - | 启用模式枚举 |

### 3.8 ResultCode 枚举

| 值 | 说明 |
|-----|------|
| `SUCCESS` | 成功 |
| `ERR_INVALID_NAME` | 无效名称 |
| `ERR_INVALID_PASSWORD` | 无效密码 |
| `ERR_ACCOUNT_NOT_EXIST` | 账号不存在 |
| `ERR_APP_ACCOUNT_SERVICE_EXCEPTION` | 服务异常 |

---

## 4. account.distributedAccount 模块

**实现文件**: `interfaces/kits/napi/distributedaccount/src/napi_distributed_account.cpp`  
**命名空间**: `account.distributedAccount`

### 4.1 导出函数

| JS 方法 | 说明 |
|---------|------|
| `getDistributedAccountAbility()` | 获取分布式账号能力（工厂函数） |

### 4.2 DistributedAccountAbility 类

| JS 方法 | 说明 |
|---------|------|
| `queryOsAccountDistributedInfo()` | 查询分布式信息 |
| `getOsAccountDistributedInfo()` | 获取分布式信息 |
| `updateOsAccountDistributedInfo(accountInfo)` | 更新分布式信息 |
| `setOsAccountDistributedInfo(accountInfo)` | 设置分布式信息 |
| `getOsAccountDistributedInfoByLocalId(localId)` | 根据本地 ID 获取 |
| `setOsAccountDistributedInfoByLocalId(localId, accountInfo)` | 根据本地 ID 设置 |
| `setCurrentOsAccountDistributedInfo(accountInfo)` | 设置当前分布式信息 |

### 4.3 DistributedInfo 类

| 属性 | 说明 |
|------|------|
| `name` | 账号名 |
| `id` | 账号 ID |
| `status` | 状态 |
| `nickName` | 昵称 |
| `avatar` | 头像 |
| `userInfo` | 用户信息 |

### 4.4 DistributedAccountStatus 枚举

| 值 | 说明 |
|-----|------|
| `NOT_LOGGED_IN` | 未登录 |
| `LOGGED_IN` | 已登录 |

---

## 5. 调用链示例

### 5.1 JS → N-API → Inner Kit → IPC → Service

```
┌─────────────────────────────────────────────────────────────────┐
│ JavaScript Layer                                                │
│ const accountManager = account.osAccount;                       │
│ accountManager.createOsAccount("testUser", type, callback);     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ N-API Layer (napi_os_account.cpp)                               │
│ CreateOsAccount(env, info)                                      │
│   ├── napi_get_cb_info()                                        │
│   ├── 参数解析与校验                                             │
│   └── OsAccountManager::CreateOsAccount()                       │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ Inner Kit (os_account_manager.h)                                │
│ OsAccountManager::CreateOsAccount(localName, type, info)       │
│   ├── 权限校验 (AccountPermissionManager)                       │
│   └── OsAccountManagerServiceProxy::CreateOsAccount()           │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ IPC Layer (Binder)                                              │
│ MessageParcel::WriteInterfaceToken()                            │
│ MessageParcel::WriteXXX()                                       │
│ IPCSkeleton::GetCallingTokenID()                                │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ Service Layer (os_account_manager_service.cpp)                   │
│ OsAccountManagerService::CreateOsAccount()                       │
│   ├── 调用者权限校验                                             │
│   ├── 创建账号逻辑                                               │
│   ├── 数据持久化                                                 │
│   └── 事件通知                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 权限要求汇总

| 权限 | 用途 |
|------|------|
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 管理本地账号 |
| `ohos.permission.GET_LOCAL_ACCOUNTS` | 获取本地账号信息 |
| `ohos.permission.MANAGE_DISTRIBUTED_ACCOUNTS` | 管理分布式账号 |
| `ohos.permission.GET_DISTRIBUTED_ACCOUNTS` | 获取分布式账号 |
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 |
| `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS` | 跨账号交互 |
| `ohos.permission.ACCESS_USER_AUTH_INTERNAL` | 内部用户认证 |
| `ohos.permission.MANAGE_USER_IDM` | 管理用户 IDM |
| `ohos.permission.USE_USER_IDM` | 使用用户 IDM |
| `ohos.permission.GET_ALL_APP_ACCOUNTS` | 获取所有应用账号 |

---

## 7. 相关文档

- [概述](./01_Overview.md)
- [Inner API](./04_Inner_API.md)
- [服务与 IPC](./05_Service_IPC.md)
- [安全机制](./07_Security.md)
