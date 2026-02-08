# 01_Overview.md

# OpenHarmony os_account 子系统概述

> 本文档描述 os_account 子系统的核心功能、模块划分与 Feature Flags。

## 1. 子系统定位

### 1.1 核心职责

os_account 是 OpenHarmony 的**账号管理基础设施**，负责：

1. **系统账号（OS Account）管理**
   - 账号生命周期（创建/删除/激活/停用）
   - 账号约束（Constraint）管理
   - 设备账号（Device Account）管理

2. **分布式账号（Distributed Account）管理**
   - 分布式身份同步
   - 跨设备账号状态管理

3. **应用账号（App Account）管理**
   - 应用级别账号存储
   - OAuth 授权框架
   - 凭据管理

4. **域账号（Domain Account）管理**
   - 企业域账号绑定
   - 统一身份认证

5. **身份认证与访问控制（IAM）**
   - PIN 码认证
   - 用户身份管理（User IDM）
   - 生物特征认证集成

### 1.2 在系统中的位置

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                      │
│  System Apps │ Third-party Apps │ System Services               │
├─────────────────────────────────────────────────────────────────┤
│                     os_account 子系统                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  N-API Layer (JS/TS)                                    │   │
│  │  @ohos.account.osAccount                                │   │
│  │  @ohos.account.appAccount                               │   │
│  │  @ohos.account.distributedAccount                       │   │
│  │  @ohos.account.domainAccount                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Framework Layer (C++)                                  │   │
│  │  osaccount │ appaccount │ domain_account               │   │
│  │  ohosaccount │ account_iam │ authorization             │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                     AccountMgrService (SA 200)                  │
│  系统启动时自动拉起的系统服务                                     │
├─────────────────────────────────────────────────────────────────┤
│                   OpenHarmony Core                              │
│  IPC/Binder │ SAMgr │ AbilityMgr │ BundleMgr                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 模块划分

### 2.1 五大核心模块

| 模块名称 | 命名空间 | 位置 | 主要功能 |
|----------|----------|------|----------|
| **osaccount** | `OHOS::AccountSA` | `frameworks/osaccount/` | 系统账号生命周期管理 |
| **appaccount** | `OHOS::AccountJsKit` | `frameworks/appaccount/` | 应用账号与 OAuth 授权 |
| **domain_account** | `OHOS::AccountSA` | `frameworks/domain_account/` | 企业域账号支持 |
| **ohosaccount** | `OHOS::AccountSA` | `frameworks/ohosaccount/` | 分布式账号同步 |
| **account_iam** | `OHOS::AccountIAM` | `frameworks/account_iam/` | 身份认证与访问控制 |

### 2.2 支撑模块

| 模块 | 功能 |
|------|------|
| **common** | 共通工具：错误码、日志、JSON 解析、文件操作 |
| **authorization** | 授权管理框架 |
| **dfx** | 诊断能力：HiLog、HiDumper、HiSysEvent、HiTrace |
| **tools/acm** | 命令行账号管理工具 |

---

## 3. Feature Flags 详解

> 所有 Feature Flags 定义在 `os_account.gni` 文件中。

### 3.1 账号管理类

| 开关名称 | 默认值 | 说明 |
|----------|--------|------|
| `os_account_enable_multiple_os_accounts` | true | 启用多账号支持 |
| `os_account_multiple_active_accounts` | true | 允许多个活跃账号 |
| `os_account_enable_multiple_foreground_os_accounts` | false | 允许多个前台账号 |
| `os_account_enable_default_admin_name` | true | 使用默认管理员名 |
| `os_account_enable_account_short_name` | false | 启用短账号名 |
| `os_account_activate_last_logged_in_account` | false | 自动激活最后登录账号 |
| `os_account_enable_account_1` | false | 启用账号索引从 1 开始 |
| `os_account_support_deactivate_main_os_account` | false | 允许停用主账号 |
| `os_account_support_lock_os_account` | false | 支持锁定账号 |

### 3.2 分布式与域账号类

| 开关名称 | 默认值 | 说明 |
|----------|--------|------|
| `os_account_distributed_feature` | true | 启用分布式特性 |
| `os_account_support_domain_accounts` | true | 支持域账号 |

### 3.3 授权与认证类

| 开关名称 | 默认值 | 说明 |
|----------|--------|------|
| `os_account_support_authorization` | false | 支持授权框架 |

### 3.4 可选依赖类

| 开关名称 | 条件依赖 | 说明 |
|----------|----------|------|
| `has_user_auth_part` | useriam_user_auth_framework | 用户认证框架 |
| `has_user_idm_part` | useriam_user_auth_framework | 用户 IDM |
| `has_pin_auth_part` | useriam_pin_auth | PIN 码认证 |
| `has_app_account_part` | distributeddatamgr_kv_store | 应用账号（需 KV 存储） |
| `has_hiviewdfx_hisysevent_part` | hiviewdfx_hisysevent | 系统事件 |
| `has_hiviewdfx_hitrace_part` | hiviewdfx_hitrace | 性能追踪 |
| `has_hiviewdfx_hicollie_part` | hiviewdfx_hicollie | 休眠追踪 |
| `has_ces_part` | notification_common_event_service | 公共事件服务 |

---

## 4. 系统能力配置

### 4.1 SA 200 配置

| 属性 | 值 |
|------|-----|
| **SA ID** | 200 |
| **进程名** | accountmgr |
| **库路径** | libaccountmgr.z.so |
| **启动方式** | run-on-create（设备启动时自动拉起） |
| **分布式** | false |
| **Dump 级别** | 1 |

配置文件：`sa_profile/accountmgr.json`

```json
{
    "process": "accountmgr",
    "systemability": [{
        "name": 200,
        "libpath": "libaccountmgr.z.so",
        "run-on-create": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

---

## 5. 账号类型详解

### 5.1 OS Account（系统账号）

| 类型 | 值 | 说明 |
|------|-----|------|
| ADMIN | 0 | 管理员账号 |
| NORMAL | 1 | 普通用户账号 |
| GUEST | 2 | 访客账号 |
| PRIVATE | 1024 | 私有账号 |

**约束类型（Constraint）**：
- `constraint.os.account.DISALLOW_MODIFY_ACCOUNTS` - 禁止修改账号
- `constraint.os.account.DISALLOW_ADDING_ACCOUNTS` - 禁止添加账号
- `constraint.os.account.DISALLOW_REMOVE_ACCOUNTS` - 禁止删除账号
- `constraint.os.account.DISALLOW_EXPANDED_METHODS` - 禁止扩展方法

### 5.2 Domain Account（域账号）

**账号状态**：
| 状态 | 值 | 说明 |
|------|-----|------|
| NOT_LOGGED_IN | 0 | 未登录 |
| LOGGED_IN | 1 | 已登录 |

---

## 6. 依赖关系

### 6.1 系统依赖

| 依赖组件 | 用途 |
|----------|------|
| ability_base | 基础 Ability 能力 |
| ability_runtime | 运行时 Ability |
| access_token | 权限 token 管理 |
| bundle_framework | Bundle 框架 |
| ipc | IPC 通信 |
| samgr | 系统能力管理 |
| safwk | 系统能力框架 |
| napi | Node.js API |
| hilog | 日志 |
| hisysevent | 系统事件 |
| hitrace | 性能追踪 |
| sqlite | 本地数据库 |
| huks | 用户密钥库 |
| kv_store | 分布式 KV 存储 |

### 6.2 模块间依赖

```
                    ┌─────────────────┐
                    │   acm (工具)    │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   osaccount   │   │   appaccount │   │ ohosaccount  │
│  framework    │   │  framework   │   │  framework   │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                    │
        └───────────────────┼────────────────────┘
                            │
                            ▼
              ┌─────────────────────────┐
              │   AccountMgrService     │
              │      (SA 200)           │
              └───────────┬─────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
   ┌────────────┐  ┌────────────┐  ┌────────────┐
   │ ability_  │  │  bundle_   │  │   other    │
   │ manager   │  │  manager   │  │   system   │
   │ adapter   │  │  adapter   │  │   services │
   └────────────┘  └────────────┘  └────────────┘
```

---

## 7. 日志与诊断

### 7.1 日志配置

| 属性 | 值 |
|------|-----|
| **日志 TAG** | ACCOUNT |
| **日志域** | 0xD001B00 |

### 7.2 系统事件

配置文件：`hisysevent.yaml`

支持的事件类型：
- 账号创建/删除事件
- 登录/登出事件
- 认证结果事件
- 授权变更事件

---

## 8. 数据存储

### 8.1 存储位置

| 数据类型 | 存储路径 |
|----------|----------|
| 运行时数据 | `/data/service/el1/public/account/` |
| 配置数据 | JSON 配置文件 |

### 8.2 配置文件

| 文件 | 用途 |
|------|------|
| `os_account_config.json` | 账号限制配置 |
| `os_account_constraint_config.json` | 约束配置 |
| `os_account_constraint_definition.json` | 约束定义 |

---

## 9. 相关文档

- [N-API 接口文档](./03_NAPI_Interfaces.md)
- [Inner API 文档](./04_Inner_API.md)
- [服务与 IPC](./05_Service_IPC.md)
- [安全机制](./07_Security.md)
- [构建系统](./06_Build_System.md)
