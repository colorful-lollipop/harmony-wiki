# 05_Service_IPC.md

# OpenHarmony os_account 服务与 IPC 通信

> 本文档描述 os_account 子系统的系统服务架构与 IPC 通信机制。

---

## 1. 系统能力配置

### 1.1 SA 200 配置

**配置文件**: `sa_profile/accountmgr.json`

```json
{
    "process": "accountmgr",
    "systemability": [
        {
            "name": 200,
            "libpath": "libaccountmgr.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| **SA ID** | 200 | 系统能力标识符 |
| **进程** | accountmgr | 运行进程名 |
| **库路径** | libaccountmgr.z.so | 服务库文件名 |
| **启动方式** | run-on-create | 设备启动时自动拉起 |
| **分布式** | false | 不支持跨设备调用 |
| **Dump 级别** | 1 | 支持基础信息导出 |

### 1.2 服务进程

服务运行在独立的 `accountmgr` 进程中，通过 OpenHarmony 的 **SAMgr**（System Ability Manager）进行管理。

```
accountmgr 进程
├── AccountMgrService (主服务入口)
├── OsAccountManagerService (OS 账号服务)
├── AppAccountManagerService (应用账号服务)
├── DomainAccountManagerService (域账号服务)
├── AccountIAMService (身份认证服务)
└── AuthorizationManagerService (授权服务)
```

---

## 2. IPC 接口清单

### 2.1 主服务接口: IAccount

**文件**: `services/accountmgr/include/account_mgr_service.h`

| 方法 | 说明 |
|------|------|
| `UpdateOhosAccountInfo()` | 更新设备账号信息 |
| `SetOhosAccountInfo()` | 设置设备账号信息 |
| `QueryOhosAccountInfo()` | 查询设备账号信息 |
| `QueryDistributedVirtualDeviceId()` | 查询分布式虚拟设备 ID |
| `QueryDeviceAccountIdByUid()` | 根据 UID 查询设备账号 ID |
| `SubscribeDistributedAccountEvent()` | 订阅分布式账号事件 |
| `UnsubscribeDistributedAccountEvent()` | 取消订阅 |
| `GetAppAccountService()` | 获取应用账号服务 |
| `GetOsAccountService()` | 获取 OS 账号服务 |
| `GetAccountIAMService()` | 获取 IAM 服务 |
| `GetDomainAccountService()` | 获取域账号服务 |
| `GetAuthorizationService()` | 获取授权服务 |

### 2.2 OS 账号服务接口: IOsAccount

**文件**: `services/accountmgr/include/osaccount/os_account_manager_service.h`

| 方法 | 说明 |
|------|------|
| `CreateOsAccount()` | 创建 OS 账号 |
| `CreateOsAccountForDomain()` | 为域账号创建 OS 账号 |
| `RemoveOsAccount()` | 删除 OS 账号 |
| `IsOsAccountExists()` | 检查账号是否存在 |
| `IsOsAccountActived()` | 检查账号是否已激活 |
| `IsOsAccountVerified()` | 检查账号是否已验证 |
| `IsOsAccountDeactivating()` | 检查账号是否正在停用 |
| `GetCreatedOsAccountsCount()` | 获取已创建账号数量 |
| `QueryMaxOsAccountNumber()` | 查询最大账号数 |
| `QueryMaxLoggedInOsAccountNumber()` | 查询最大登录数 |
| `ActivateOsAccount()` | 激活 OS 账号 |
| `DeactivateOsAccount()` | 停用 OS 账号 |
| `StartOsAccount()` | 启动 OS 账号 |
| `DeactivateAllOsAccounts()` | 停用所有 OS 账号 |
| `SubscribeOsAccount()` | 订阅 OS 账号事件 |
| `UnsubscribeOsAccount()` | 取消订阅 |
| `SetOsAccountConstraints()` | 设置账号约束 |
| `GetOsAccountLocalIdFromDomain()` | 从域获取本地 ID |
| `GetOsAccountLocalIdBySerialNumber()` | 根据序列号获取本地 ID |
| `SetOsAccountName()` | 设置账号名称 |
| `SetOsAccountProfilePhoto()` | 设置账号头像 |
| `QueryOsAccountById()` | 根据 ID 查询账号 |
| `SetGlobalOsAccountConstraints()` | 设置全局账号约束 |
| `SetSpecificOsAccountConstraints()` | 设置特定账号约束 |
| `SubscribeOsAccountConstraints()` | 订阅账号约束变更 |
| `UnsubscribeOsAccountConstraints()` | 取消订阅 |
| `IsOsAccountForeground()` | 检查账号是否在前台 |
| `GetForegroundOsAccountLocalId()` | 获取前台账号本地 ID |
| `GetForegroundOsAccountDisplayId()` | 获取前台账号显示 ID |
| `BindDomainAccount()` | 绑定域账号 |
| `LockOsAccount()` | 锁定 OS 账号 |
| `PublishOsAccountLockEvent()` | 发布账号锁定事件 |

### 2.3 应用账号服务接口: IAppAccount

**文件**: `services/accountmgr/include/appaccount/app_account_manager_service.h`

| 方法 | 说明 |
|------|------|
| `AddAccount()` | 添加账号 |
| `AddAccountImplicitly()` | 隐式添加账号 |
| `CreateAccount()` | 创建账号 |
| `CreateAccountImplicitly()` | 隐式创建账号 |
| `DeleteAccount()` | 删除账号 |
| `GetAccountExtraInfo()` | 获取附加信息 |
| `SetAccountExtraInfo()` | 设置附加信息 |
| `EnableAppAccess()` | 启用应用访问 |
| `DisableAppAccess()` | 禁用应用访问 |
| `SetAppAccess()` | 设置应用访问 |
| `CheckAppAccess()` | 检查应用访问 |
| `CheckAppAccountSyncEnable()` | 检查同步启用 |
| `SetAppAccountSyncEnable()` | 设置同步启用 |
| `GetAssociatedData()` | 获取关联数据 |
| `SetAssociatedData()` | 设置关联数据 |
| `GetAccountCredential()` | 获取凭据 |
| `SetAccountCredential()` | 设置凭据 |
| `DeleteAccountCredential()` | 删除凭据 |
| `Authenticate()` | 认证 |
| `VerifyCredential()` | 验证凭据 |
| `CheckAccountLabels()` | 检查账号标签 |
| `GetOAuthToken()` | 获取 OAuth 令牌 |
| `GetAuthToken()` | 获取认证令牌 |
| `SetOAuthToken()` | 设置 OAuth 令牌 |
| `SetAuthToken()` | 设置认证令牌 |
| `DeleteOAuthToken()` | 删除 OAuth 令牌 |
| `DeleteAuthToken()` | 删除认证令牌 |
| `SetOAuthTokenVisibility()` | 设置令牌可见性 |
| `SetAuthTokenVisibility()` | 设置认证令牌可见性 |
| `CheckOAuthTokenVisibility()` | 检查令牌可见性 |
| `CheckAuthTokenVisibility()` | 检查认证令牌可见性 |
| `GetAuthenticatorInfo()` | 获取认证器信息 |
| `GetAllOAuthTokens()` | 获取所有 OAuth 令牌 |
| `GetOAuthList()` | 获取 OAuth 列表 |
| `GetAuthList()` | 获取认证列表 |
| `GetAuthenticatorCallback()` | 获取认证器回调 |
| `GetAllAccounts()` | 获取所有账号 |
| `GetAllAccessibleAccounts()` | 获取所有可访问账号 |
| `QueryAllAccessibleAccounts()` | 查询所有可访问账号 |
| `SelectAccountsByOptions()` | 根据选项选择账号 |
| `SetAuthenticatorProperties()` | 设置认证器属性 |
| `SubscribeAppAccount()` | 订阅应用账号事件 |
| `UnsubscribeAppAccount()` | 取消订阅 |

### 2.4 IAM 服务接口: IAccountIAM

**文件**: `services/accountmgr/include/account_iam/account_iam_service.h`

| 方法 | 说明 |
|------|------|
| `OpenSession()` | 打开会话 |
| `CloseSession()` | 关闭会话 |
| `AddCredential()` | 添加凭据 |
| `UpdateCredential()` | 更新凭据 |
| `Cancel()` | 取消操作 |
| `DelCred()` | 删除凭据 |
| `DelUser()` | 删除用户 |
| `GetCredentialInfo()` | 获取凭据信息 |
| `PrepareRemoteAuth()` | 准备远程认证 |
| `AuthUser()` | 用户认证 |
| `CancelAuth()` | 取消认证 |
| `GetAvailableStatus()` | 获取可用状态 |
| `GetProperty()` | 获取属性 |
| `GetPropertyByCredentialId()` | 根据凭据 ID 获取属性 |
| `SetProperty()` | 设置属性 |
| `GetEnrolledId()` | 获取已注册 ID |
| `GetAccountState()` | 获取账号状态 |

### 2.5 域账号服务接口: IDomainAccount

**文件**: `services/accountmgr/include/domain_account/domain_account_manager_service.h`

| 方法 | 说明 |
|------|------|
| `RegisterPlugin()` | 注册插件 |
| `UnregisterPlugin()` | 注销插件 |
| `HasDomainAccount()` | 检查域账号是否存在 |
| `GetAccountStatus()` | 获取账号状态 |
| `RegisterAccountStatusListener()` | 注册状态监听器 |
| `UnregisterAccountStatusListener()` | 注销状态监听器 |
| `Auth()` | 认证 |
| `AuthUser()` | 用户认证 |
| `AuthWithPopup()` | 弹窗认证 |
| `CancelAuth()` | 取消认证 |
| `UpdateAccountToken()` | 更新账号令牌 |
| `IsAuthenticationExpired()` | 认证是否过期 |
| `SetAccountPolicy()` | 设置账号策略 |
| `GetAccountPolicy()` | 获取账号策略 |
| `GetAccessToken()` | 获取访问令牌 |
| `GetDomainAccountInfo()` | 获取域账号信息 |
| `UpdateAccountInfo()` | 更新账号信息 |
| `AddServerConfig()` | 添加服务器配置 |
| `RemoveServerConfig()` | 移除服务器配置 |
| `UpdateServerConfig()` | 更新服务器配置 |
| `GetAccountServerConfig()` | 获取账号服务器配置 |
| `GetServerConfig()` | 获取服务器配置 |
| `GetAllServerConfigs()` | 获取所有服务器配置 |

### 2.6 授权服务接口: IAuthorization

**文件**: `services/accountmgr/include/authorization/authorization_manager_service.h`

| 方法 | 说明 |
|------|------|
| `AcquireAuthorization()` | 获取授权 |

---

## 3. IPC 模式

### 3.1 Stub-Proxy 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     服务端 (accountmgr 进程)                      │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ AccountMgrService : AccountStub : IRemoteStub<IAccount> │   │
│  │  └── OnRemoteRequest(command, data, reply)               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │ IPC (Binder)
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     客户端 (应用进程)                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ AccountMgrServiceProxy : IRemoteProxy<IAccount>          │   │
│  │  └── SendRequest(command, data, reply, option)           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 IPC 调用流程

```cpp
// 客户端调用示例
auto proxy = GetRemoteProxy<IAccount>();
MessageParcel data;
MessageParcel reply;
MessageOption option;

data.WriteInterfaceToken(IAccount::GetDescriptor());
data.WriteXXX(args...);  // 写入参数

int32_t err = proxy->SendRequest(
    IAccount::CMD_XXX,  // 命令码
    data,
    reply,
    option
);

// 读取返回结果
XXX result = reply.ReadXXX();
```

### 3.3 OnRemoteRequest 处理

```cpp
// 服务端 Stub 实现
int32_t AccountStub::OnRemoteRequest(
    uint32_t code,
    MessageParcel &data,
    MessageParcel &reply,
    MessageOption &option)
{
    switch (code) {
        case IAccount::CMD_XXX:
            return CmdXXX(data, reply);
        case IAccount::CMD_YYY:
            return CmdYYY(data, reply);
        default:
            return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
    }
}
```

---

## 4. 消息码枚举

各接口使用独立的 IPC 消息码枚举：

| 枚举 | 用途 |
|------|------|
| `IAccountIpcCode` | 主服务命令码 |
| `IOsAccountIpcCode` | OS 账号命令码 |
| `IAppAccountIpcCode` | 应用账号命令码 |
| `IAccountIAMIpcCode` | IAM 命令码 |
| `IDomainAccountIpcCode` | 域账号命令码 |

---

## 5. 服务发现

### 5.1 获取子服务

主服务 `AccountMgrService` 提供子服务获取接口：

```cpp
// 获取应用账号服务
sptr<IAppAccount> appAccountService = 
    accountMgrService->GetAppAccountService();

// 获取 OS 账号服务
sptr<IOsAccount> osAccountService = 
    accountMgrService->GetOsAccountService();

// 获取 IAM 服务
sptr<IAccountIAM> iamService = 
    accountMgrService->GetAccountIAMService();

// 获取域账号服务
sptr<IDomainAccount> domainAccountService = 
    accountMgrService->GetDomainAccountService();

// 获取授权服务
sptr<IAuthorization> authService = 
    accountMgrService->GetAuthorizationService();
```

### 5.2 SAMgr 获取

也可以通过 SAMgr 直接获取 SA 200：

```cpp
auto samgr = SystemAbilityManagerClient::GetSystemAbilityManager();
auto accountSa = samgr->GetSystemAbility(200);
sptr<IAccount> accountService = iface_cast<IAccount>(accountSa);
```

---

## 6. 回调与事件

### 6.1 远程回调接口

| 接口 | 说明 |
|------|------|
| `IAppAccountEvent` | 应用账号事件回调 |
| `IDomainAccountEvent` | 域账号事件回调 |
| `IUserCallback` | 用户操作回调 |

### 6.2 订阅模式

```cpp
// 订阅 OS 账号事件
sptr<OsAccountUserCallback> callback = new OsAccountUserCallback();
accountService->SubscribeOsAccount(callback);

// 订阅应用账号变更
sptr<IAppAccountEvent> eventProxy = new AppAccountEventProxy();
appAccountService->SubscribeAppAccount(eventProxy);
```

---

## 7. 相关文档

- [概述](./01_Overview.md)
- [目录结构](./02_Directory_Structure.md)
- [N-API 接口](./03_NAPI_Interfaces.md)
- [Inner API](./04_Inner_API.md)
- [安全机制](./07_Security.md)
