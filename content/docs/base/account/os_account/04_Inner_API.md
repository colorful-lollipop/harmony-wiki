# 04_Inner_API.md

# OpenHarmony os_account Inner API 内部接口

> 本文档描述 os_account 子系统的内部 C++ API。

---

## 1. Inner API 概览

Inner API 是供系统组件内部使用的 C++ 接口，定义了账号子系统的核心功能。

### 1.1 Inner Kit 清单

| 头文件 | 命名空间 | 说明 |
|--------|----------|------|
| `os_account_manager.h` | `OHOS::AccountSA` | OS 账号管理 |
| `os_account_info.h` | `OHOS::AccountSA` | OS 账号信息 |
| `app_account_manager.h` | `OHOS::AccountJsKit` | 应用账号管理 |
| `ohos_account_kits.h` | `OHOS::AccountSA` | 分布式账号 |
| `account_iam_client.h` | `OHOS::AccountIAM` | IAM 客户端 |
| `account_iam_info.h` | `OHOS::AccountIAM` | IAM 信息 |
| `domain_account_client.h` | `OHOS::AccountSA` | 域账号客户端 |
| `authorization_client.h` | `OHOS::AccountSA` | 授权客户端 |

---

## 2. OS Account Inner API

### 2.1 OsAccountManager 类

**头文件**: `interfaces/innerkits/osaccount/native/include/os_account_manager.h`

#### 创建与删除

| 方法 | 说明 |
|------|------|
| `CreateOsAccount(const std::string& name, OsAccountType type, OsAccountInfo& info)` | 创建 OS 账号 |
| `CreateOsAccountForDomain(OsAccountType type, const DomainAccountInfo& domainInfo, OsAccountInfo& info)` | 为域创建 OS 账号 |
| `RemoveOsAccount(int32_t localId)` | 删除 OS 账号 |

#### 激活与状态

| 方法 | 说明 |
|------|------|
| `ActivateOsAccount(int32_t localId)` | 激活 OS 账号 |
| `DeactivateOsAccount(int32_t localId)` | 停用 OS 账号 |
| `IsOsAccountActived(int32_t localId, bool& isActived)` | 检查是否激活 |
| `IsOsAccountVerified(int32_t localId, bool& isVerified)` | 检查是否验证 |
| `IsMultiOsAccountEnable(bool& enable)` | 检查是否启用多账号 |

#### 查询方法

| 方法 | 说明 |
|------|------|
| `QueryOsAccountById(int32_t localId, OsAccountInfo& info)` | 根据 ID 查询 |
| `QueryAllCreatedOsAccounts(std::vector<OsAccountInfo>& infos)` | 查询所有账号 |
| `GetCreatedOsAccountsCount(int32_t& count)` | 获取已创建数量 |
| `GetCurrentOsAccount(OsAccountInfo& info)` | 获取当前账号 |
| `GetOsAccountLocalIdFromUid(int32_t uid, int32_t& localId)` | 从 UID 获取本地 ID |
| `GetOsAccountLocalIdFromDomain(const DomainAccountInfo& domainInfo, int32_t& localId)` | 从域获取本地 ID |

#### 约束管理

| 方法 | 说明 |
|------|------|
| `SetOsAccountConstraints(int32_t localId, const std::vector<std::string>& constraints, bool enable)` | 设置约束 |
| `GetOsAccountAllConstraints(int32_t localId, std::vector<std::string>& constraints)` | 获取所有约束 |

#### 属性设置

| 方法 | 说明 |
|------|------|
| `SetOsAccountName(int32_t localId, const std::string& name)` | 设置名称 |
| `SetOsAccountProfilePhoto(int32_t localId, const std::string& photo)` | 设置头像 |

### 2.2 OsAccountInfo 类

**头文件**: `interfaces/innerkits/osaccount/native/include/os_account_info.h`

| 属性 | 类型 | 说明 |
|------|------|------|
| `localId_` | int32_t | 账号本地 ID |
| `localName_` | std::string | 账号名称 |
| `type_` | OsAccountType | 账号类型 |
| `isActived_` | bool | 是否激活 |
| `isVerified_` | bool | 是否验证 |
| `photo_` | std::string | 头像路径 |
| `constraints_` | std::vector<std::string> | 约束列表 |

### 2.3 账号类型枚举

```cpp
enum OsAccountType {
    ADMIN = 0,      // 管理员
    NORMAL = 1,     // 普通用户
    GUEST = 2,      // 访客
    PRIVATE = 1024  // 私有
};
```

---

## 3. App Account Inner API

### 3.1 AppAccountManager 类

**头文件**: `interfaces/innerkits/appaccount/native/include/app_account_manager.h`

#### 账号管理

| 方法 | 说明 |
|------|------|
| `AddAccount(const std::string& name, const std::string& extraInfo)` | 添加账号 |
| `DeleteAccount(const std::string& name)` | 删除账号 |
| `GetAccountExtraInfo(const std::string& name, std::string& extraInfo)` | 获取附加信息 |
| `SetAccountExtraInfo(const std::string& name, const std::string& extraInfo)` | 设置附加信息 |

#### 访问控制

| 方法 | 说明 |
|------|------|
| `EnableAppAccess(const std::string& name, const std::string& authorizedApp)` | 启用访问 |
| `DisableAppAccess(const std::string& name, const std::string& authorizedApp)` | 禁用访问 |
| `CheckAppAccountSyncEnable(const std::string& name, bool& enable)` | 检查同步启用 |
| `SetAppAccountSyncEnable(const std::string& name, bool enable)` | 设置同步 |

#### 关联数据

| 方法 | 说明 |
|------|------|
| `GetAssociatedData(const std::string& name, const std::string& key, std::string& value)` | 获取关联数据 |
| `SetAssociatedData(const std::string& name, const std::string& key, const std::string& value)` | 设置关联数据 |

#### 凭据管理

| 方法 | 说明 |
|------|------|
| `SetAccountCredential(const std::string& name, const std::string& credentialType, const std::string& credential)` | 设置凭据 |
| `GetAccountCredential(const std::string& name, const std::string& credentialType, std::string& credential)` | 获取凭据 |

#### OAuth

| 方法 | 说明 |
|------|------|
| `Authenticate(const std::string& name, const std::string& owner, const std::string& authType, const Aaf Winf& options, AuthenticatorCallback& callback)` | 认证 |
| `GetOAuthToken(const std::string& name, const std::string& owner, const std::string& authType, std::string& token)` | 获取令牌 |
| `SetOAuthToken(const std::string& name, const std::string& authType, const std::string& token)` | 设置令牌 |
| `DeleteOAuthToken(const std::string& name, const std::string& owner, const std::string& authType, const std::string& token)` | 删除令牌 |

---

## 4. Account IAM Inner API

### 4.1 AccountIAMClient 类

**头文件**: `interfaces/innerkits/account_iam/native/include/account_iam_client.h`

#### 会话管理

| 方法 | 说明 |
|------|------|
| `OpenSession(std::vector<uint8_t>& challenge)` | 打开会话 |
| `CloseSession()` | 关闭会话 |

#### 凭据管理

| 方法 | 说明 |
|------|------|
| `AddCredential(int32_t userId, const CredentialParam& credParam, const sptr<IDMCallback>& callback)` | 添加凭据 |
| `UpdateCredential(int32_t userId, const CredentialParam& credParam, const sptr<IDMCallback>& callback)` | 更新凭据 |
| `DelCredential(int32_t userId, uint64_t credentialId, const std::vector<uint8_t>& authToken, const sptr<IDMCallback>& callback)` | 删除凭据 |
| `DelUser(int32_t userId, const std::vector<uint8_t>& authToken, const sptr<IDMCallback>& callback)` | 删除用户 |

#### 认证

| 方法 | 说明 |
|------|------|
| `AuthUser(int32_t userId, const UserAuthParam& param, const sptr<IDMCallback>& callback)` | 用户认证 |
| `CancelAuth(int32_t channelId)` | 取消认证 |
| `GetAuthInfo(int32_t userId, int32_t authType, std::vector<CredentialInfo>& credInfos)` | 获取认证信息 |
| `GetEnrolledId(int32_t authType, int32_t& enrolledId)` | 获取已注册 ID |

#### 属性

| 方法 | 说明 |
|------|------|
| `GetAvailableStatus(int32_t authType, int32_t authTrustLevel, int32_t& status)` | 获取可用状态 |
| `GetProperty(int32_t userId, int32_t authType, int32_t propertyType, std::vector<uint8_t>& property)` | 获取属性 |
| `SetProperty(int32_t userId, int32_t authType, int32_t propertyType, const std::vector<uint8_t>& property, const sptr<IDMCallback>& callback)` | 设置属性 |

### 4.2 凭据类型枚举

```cpp
enum CredentialType {
    PIN = 1,
    FACE = 2,
    FINGERPRINT = 3,
    // ...
};
```

---

## 5. Domain Account Inner API

### 5.1 DomainAccountClient 类

**头文件**: `interfaces/innerkits/domain_account/native/include/domain_account_client.h`

| 方法 | 说明 |
|------|------|
| `HasDomainAccount(const std::string& domainInfo, bool& isExist)` | 检查账号存在 |
| `GetAccountStatus(const std::string& domainInfo, int32_t& status)` | 获取账号状态 |
| `Auth(const std::string& domainInfo, const std::string& authType, const std::string& param, const sptr<IDomainAccountCallback>& callback)` | 认证 |
| `UpdateAccountToken(const std::string& domainInfo, const std::string& token, bool& result)` | 更新令牌 |
| `GetAccessToken(const std::string& domainInfo, std::string& accessToken)` | 获取访问令牌 |

---

## 6. Ohos Account Inner API

### 6.1 OhosAccountKits 类

**头文件**: `interfaces/innerkits/ohosaccount/native/include/ohos_account_kits.h`

| 方法 | 说明 |
|------|------|
| `GetOhosAccountInfo(OhosAccountInfo& info)` | 获取账号信息 |
| `SetOhosAccountInfo(const OhosAccountInfo& info)` | 设置账号信息 |
| `GetOhosAccountName(std::string& name)` | 获取账号名称 |
| `GetOhosAccountUid(std::string& uid)` | 获取账号 UID |
| `GetDistributedVirtualDeviceId(std::string& deviceId)` | 获取分布式虚拟设备 ID |

---

## 7. 错误码定义

**头文件**: `frameworks/common/account_error/include/account_error_no.h`

| 错误码 | 定义 | 说明 |
|--------|------|------|
| `ERR_OK` | 0 | 成功 |
| `ERR_ACCOUNT_COMMON_PERMISSION_DENIED` | 401 | 权限拒绝 |
| `ERR_ACCOUNT_COMMON_INVALID_PARAM` | 401 | 无效参数 |
| `ERR_ACCOUNT_COMMON_NOT_SYSTEM_APP_ERROR` | 401 | 非系统应用 |
| `ERR_ACCOUNT_COMMON_ACCOUNT_NOT_EXIST` | 401 | 账号不存在 |
| `ERR_ACCOUNT_COMMON_ACCOUNT_EXIST` | 401 | 账号已存在 |
| `ERR_ACCOUNT_COMMON_SERVICE_EXCEPTION` | 460 | 服务异常 |

---

## 8. 相关文档

- [概述](./01_Overview.md)
- [目录结构](./02_Directory_Structure.md)
- [N-API 接口](./03_NAPI_Interfaces.md)
- [服务与 IPC](./05_Service_IPC.md)
