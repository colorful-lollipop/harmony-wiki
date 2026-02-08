# 08_Config_Flags.md

# OpenHarmony os_account 配置与 Feature Flags

> 本文档描述 os_account 子系统的编译时配置开关和运行时配置。

---

## 1. Feature Flags 总览

所有 Feature Flags 定义在 `os_account.gni` 文件中，通过 `declare_args()` 声明。

### 1.1 账号管理类开关

| 开关 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `os_account_multiple_active_accounts` | true | bool | 允许多个活跃账号 |
| `os_account_enable_multiple_os_accounts` | true | bool | 启用多账号支持 |
| `os_account_enable_multiple_foreground_os_accounts` | false | bool | 允许多个前台账号 |
| `os_account_enable_default_admin_name` | true | bool | 使用默认管理员名 |
| `os_account_enable_account_short_name` | false | bool | 启用短账号名 |
| `os_account_activate_last_logged_in_account` | false | bool | 自动激活最后登录账号 |
| `os_account_enable_account_1` | false | bool | 账号索引从 1 开始 |
| `os_account_support_deactivate_main_os_account` | false | bool | 允许停用主账号 |
| `os_account_support_lock_os_account` | false | bool | 支持锁定账号 |

### 1.2 分布式与域账号类开关

| 开关 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `os_account_distributed_feature` | true | bool | 启用分布式特性 |
| `os_account_support_domain_accounts` | true | bool | 支持域账号 |

### 1.3 授权类开关

| 开关 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `os_account_support_authorization` | false | bool | 支持授权框架 |

---

## 2. 条件依赖开关

### 2.1 用户认证相关

| 开关 | 条件依赖 | 默认值 | 说明 |
|------|----------|--------|------|
| `has_user_auth_part` | `useriam_user_auth_framework` | true | 用户认证框架 |
| `has_user_idm_part` | `useriam_user_auth_framework` | true | 用户 IDM |
| `has_pin_auth_part` | `useriam_pin_auth` 或 `useriam_user_auth_framework` | true | PIN 码认证 |

### 2.2 系统服务相关

| 开关 | 条件依赖 | 默认值 | 说明 |
|------|----------|--------|------|
| `has_ces_part` | `notification_common_event_service` | true | 公共事件服务 |
| `has_hiviewdfx_hisysevent_part` | `hiviewdfx_hisysevent` | true | 系统事件 |
| `has_hiviewdfx_hitrace_part` | `hiviewdfx_hitrace` | true | 性能追踪 |
| `has_hiviewdfx_hicollie_part` | `hiviewdfx_hicollie` | true | 休眠追踪 |

### 2.3 存储相关

| 开关 | 条件依赖 | 默认值 | 说明 |
|------|----------|--------|------|
| `has_kv_store_part` | `filemanagement_storage_service` | true | KV 存储 |
| `has_app_account_part` | `distributeddatamgr_kv_store` | true | 应用账号（需 KV） |
| `has_storage_service_part` | `filemanagement_storage_service` | true | 存储服务 |

### 2.4 安全相关

| 开关 | 条件依赖 | 默认值 | 说明 |
|------|----------|--------|------|
| `has_huks_part` | `security_huks` | true | 用户密钥库 |
| `has_asset_part` | `security_asset` | true | 资产保护 |
| `security_guard_enabled` | `security_security_guard` | true | 安全守护 |

### 2.5 其他

| 开关 | 条件依赖 | 默认值 | 说明 |
|------|----------|--------|------|
| `has_config_policy_part` | `customization_config_policy` | true | 配置策略 |

---

## 3. 运行时配置

### 3.1 账号配置文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `os_account_config.json` | `/data/service/el1/public/account/` | 账号限制配置 |
| `os_account_constraint_config.json` | `/data/service/el1/public/account/` | 约束配置 |
| `os_account_constraint_definition.json` | `/data/service/el1/public/account/` | 约束定义 |

### 3.2 配置项说明

#### os_account_config.json

```json
{
  "maxOsAccountNum": 3,
  "maxLoggedInOsAccountNum": 1,
  "domainAccountEnabled": true,
  "distributedAccountEnabled": true
}
```

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `maxOsAccountNum` | 最大 OS 账号数 | 3 |
| `maxLoggedInOsAccountNum` | 最大同时登录数 | 1 |
| `domainAccountEnabled` | 是否启用域账号 | true |
| `distributedAccountEnabled` | 是否启用分布式账号 | true |

#### os_account_constraint_definition.json

```json
{
  "constraints": [
    "constraint.os.account.DISALLOW_MODIFY_ACCOUNTS",
    "constraint.os.account.DISALLOW_ADDING_ACCOUNTS",
    "constraint.os.account.DISALLOW_REMOVING_ACCOUNTS",
    "constraint.os.account.DISALLOW_EXPANDED_METHODS"
  ]
}
```

### 3.3 账号约束类型

| 约束名 | 说明 |
|--------|------|
| `constraint.os.account.DISALLOW_MODIFY_ACCOUNTS` | 禁止修改账号 |
| `constraint.os.account.DISALLOW_ADDING_ACCOUNTS` | 禁止添加账号 |
| `constraint.os.account.DISALLOW_REMOVING_ACCOUNTS` | 禁止删除账号 |
| `constraint.os.account.DISALLOW_EXPANDED_METHODS` | 禁止扩展方法 |

---

## 4. Feature Flags 使用示例

### 4.1 条件编译

```cpp
#if defined(os_account_support_domain_accounts)
#include "domain_account_manager.h"
#endif

#if defined(has_user_auth_part)
#include "user_auth_framework.h"
#endif
```

### 4.2 条件实例化

```cpp
#if defined(os_account_distributed_feature)
OhosAccountManager* ohosAccountManager = new OhosAccountManager();
#else
OhosAccountManager* ohosAccountManager = nullptr;
#endif
```

### 4.3 条件注册

```cpp
#if defined(os_account_support_domain_accounts)
RegisterDomainAccountPlugin();
#endif
```

---

## 5. 权限配置

### 5.1 权限定义文件

**文件**: `services/accountmgr/authorization_manager/config/privileges.json`

```json
{
  "privileges": [
    {
      "name": "ohos.privilege.manage_local_accounts",
      "description": "Manage local accounts"
    }
  ]
}
```

### 5.2 权限列表

| 权限名 | 用途 | 保护范围 |
|--------|------|----------|
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 管理本地账号 | Create, Remove, Set |
| `ohos.permission.GET_LOCAL_ACCOUNTS` | 获取本地账号 | Query, Get |
| `ohos.permission.MANAGE_DISTRIBUTED_ACCOUNTS` | 管理分布式账号 | Set, Update |
| `ohos.permission.GET_DISTRIBUTED_ACCOUNTS` | 获取分布式账号 | Query, Get |
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式同步 | Sync Enable/Disable |
| `ohos.permission.ACCESS_USER_AUTH_INTERNAL` | 内部认证 | Auth, GetProperty |

---

## 6. 相关文档

- [概述](./01_Overview.md)
- [GN 构建系统](./06_Build_System.md)
- [安全机制](./07_Security.md)
