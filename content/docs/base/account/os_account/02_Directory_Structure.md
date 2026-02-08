# 02_Directory_Structure.md

# OpenHarmony os_account 目录结构与模块职责

> 本文档详细描述 os_account 子系统的目录结构及各模块职责。

---

## 1. 根目录概览

```
/base/account/os_account/
├── frameworks/              # 框架层（客户端库）
├── interfaces/             # API 接口层
├── services/               # 服务层（服务端实现）
├── sa_profile/            # SA 配置文件
├── tools/                 # 工具
├── dfx/                   # 诊断模块
├── os_account.gni        # GN 构建配置
├── bundle.json           # 组件配置
└── hisysevent.yaml       # 系统事件配置
```

---

## 2. 框架层 (frameworks/)

框架层包含客户端库，应用通过链接这些库来访问账号服务。

### 2.1 osaccount - 系统账号框架

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/osaccount/core/` | 目录 | OS account IPC 核心代码 |
| `frameworks/osaccount/core/include/` | 目录 | 核心头文件 |
| `frameworks/osaccount/core/src/` | 目录 | 核心源文件 |
| `frameworks/osaccount/native/` | 目录 | Native 实现 |
| `frameworks/osaccount/native/src/` | 目录 | Native 源文件 |

**关键类**: `OsAccountManager`, `OsAccountInfo`

### 2.2 appaccount - 应用账号框架

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/appaccount/native/include/` | 目录 | Native 头文件 |
| `frameworks/appaccount/native/src/` | 目录 | Native 源文件 |
| `frameworks/appaccount/cj/` | 目录 | CJ FFI 绑定 |
| `frameworks/appaccount/cj/include/` | 目录 | CJ 头文件 |
| `frameworks/appaccount/cj/src/` | 目录 | CJ 源文件 |

**关键类**: `AppAccountManager`, `AppAccountInfo`, `Authenticator`

### 2.3 domain_account - 域账号框架

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/domain_account/include/` | 目录 | 域账号头文件 |
| `frameworks/domain_account/src/` | 目录 | 域账号源文件 |

**关键类**: `DomainAccountClient`, `DomainAccountInfo`

### 2.4 ohosaccount - 分布式账号框架

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/ohosaccount/native/` | 目录 | Native 实现 |
| `frameworks/ohosaccount/native/include/` | 目录 | Native 头文件 |
| `frameworks/ohosaccount/native/src/` | 目录 | Native 源文件 |

**关键类**: `OhosAccountManager`, `DistributedAccountAbility`

### 2.5 account_iam - 身份认证框架

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/account_iam/include/` | 目录 | IAM 头文件 |
| `frameworks/account_iam/src/` | 目录 | IAM 源文件 |

**关键类**: `UserAuth`, `UserIdentityManager`, `PINAuth`, `InputerManager`

### 2.6 authorization - 授权框架

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/authorization/src/` | 目录 | 授权框架源文件 |

**关键类**: `AuthorizationManager`

### 2.7 common - 共通模块

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/common/account_error/` | 目录 | 错误码定义 |
| `frameworks/common/json_utils/` | 目录 | JSON 工具 |
| `frameworks/common/utils/` | 目录 | 通用工具 |
| `frameworks/common/privileges/` | 目录 | 权限系统 |
| `frameworks/common/log/` | 目录 | 日志工具 |
| `frameworks/common/file_operator/` | 目录 | 文件操作 |
| `frameworks/common/perf_stat/` | 目录 | 性能统计 |

**关键文件**:
- `account_error/include/account_error_no.h` - 错误码定义
- `log/include/account_log_wrapper.h` - 日志包装器

### 2.8 ets/taihe - 静态 Native API

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/ets/taihe/os_account/` | 目录 | OS account ANI 绑定 |
| `frameworks/ets/taihe/distributed_account/` | 目录 | 分布式账号 ANI |
| `frameworks/ets/taihe/app_account/` | 目录 | 应用账号 ANI |
| `frameworks/ets/taihe/common/` | 目录 | 共通 ANI |

### 2.9 cj - CJ FFI 绑定

| 路径 | 类型 | 说明 |
|------|------|------|
| `frameworks/cj/distributed_account/` | 目录 | 分布式账号 CJ 绑定 |

---

## 3. 服务层 (services/)

服务层包含服务端实现代码。

### 3.1 accountmgr - 主服务

| 路径 | 类型 | 说明 |
|------|------|------|
| `services/accountmgr/include/` | 目录 | 服务头文件 |
| `services/accountmgr/src/` | 目录 | 服务源文件 |
| `services/accountmgr/authorization_manager/` | 目录 | 授权管理器配置 |
| `services/accountmgr/param/` | 目录 | 参数文件 |

**关键子目录**:
- `services/accountmgr/src/osaccount/` - OS account 服务实现
- `services/accountmgr/src/appaccount/` - App account 服务实现
- `services/accountmgr/src/domain_account/` - Domain account 服务实现
- `services/accountmgr/src/account_iam/` - IAM 服务实现
- `services/accountmgr/src/authorization/` - 授权服务实现
- `services/accountmgr/src/common/` - 共通服务代码
- `services/accountmgr/src/bundle_manager_adapter/` - Bundle 管理器适配器
- `services/accountmgr/src/ability_manager_adapter/` - Ability 管理器适配器

**关键类**:
- `AccountMgrService` - 主服务入口
- `OsAccountManagerService` - OS account 服务
- `AppAccountManagerService` - App account 服务
- `DomainAccountManagerService` - Domain account 服务
- `AccountIAMService` - IAM 服务
- `AuthorizationManagerService` - 授权服务

---

## 4. 接口层 (interfaces/)

接口层包含对外 API 定义。

### 4.1 innerkits - 内部 C++ API

| 路径 | 说明 |
|------|------|
| `interfaces/innerkits/osaccount/native/include/` | OS account Inner API |
| `interfaces/innerkits/appaccount/native/include/` | App account Inner API |
| `interfaces/innerkits/domain_account/native/include/` | Domain account Inner API |
| `interfaces/innerkits/account_iam/native/include/` | IAM Inner API |
| `interfaces/innerkits/ohosaccount/native/include/` | 分布式账号 Inner API |
| `interfaces/innerkits/authorization/native/include/` | 授权 Inner API |
| `interfaces/innerkits/common/` | 共通 Inner API |

**关键头文件**:
- `interfaces/innerkits/osaccount/native/include/os_account_manager.h`
- `interfaces/innerkits/appaccount/native/include/app_account_manager.h`
- `interfaces/innerkits/account_iam/native/include/account_iam_client.h`
- `interfaces/innerkits/ohosaccount/native/include/ohos_account_kits.h`

### 4.2 kits - 对外 API

#### N-API (JavaScript/TypeScript)

| 路径 | 说明 |
|------|------|
| `interfaces/kits/napi/osaccount/` | OS account N-API |
| `interfaces/kits/napi/appaccount/` | App account N-API |
| `interfaces/kits/napi/distributedaccount/` | 分布式账号 N-API |
| `interfaces/kits/napi/domain_account/` | Domain account N-API |
| `interfaces/kits/napi/account_iam/` | IAM N-API |
| `interfaces/kits/napi/authorization/` | 授权 N-API |
| `interfaces/kits/napi/common/` | 共通 N-API |

**关键文件**:
- `interfaces/kits/napi/osaccount/src/napi_init.cpp` - OS account N-API 入口
- `interfaces/kits/napi/osaccount/src/napi_os_account.cpp` - OS account N-API 实现
- `interfaces/kits/napi/appaccount/src/napi_app_account_module.cpp` - App account N-API 入口
- `interfaces/kits/napi/distributedaccount/src/napi_distributed_account.cpp` - 分布式账号 N-API

#### C API

| 路径 | 说明 |
|------|------|
| `interfaces/kits/capi/osaccount/` | OS account C API |

#### CJ FFI

| 路径 | 说明 |
|------|------|
| `interfaces/kits/cj/osaccount/` | OS account CJ 绑定 |

---

## 5. SA 配置文件 (sa_profile/)

| 文件 | 说明 |
|------|------|
| `sa_profile/accountmgr.json` | SA 200 配置 |

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

## 6. 工具 (tools/)

| 路径 | 说明 |
|------|------|
| `tools/acm/` | Account Command Manager - 命令行账号管理工具 |

---

## 7. 诊断模块 (dfx/)

| 路径 | 说明 |
|------|------|
| `dfx/hidumper_adapter/` | HiDumper 适配器 |
| `dfx/hisysevent_adapter/` | HiSysEvent 适配器 |
| `dfx/hitrace_adapter/` | HiTrace 适配器 |
| `dfx/data_dfx/` | 数据诊断 |

---

## 8. 配置文件

| 文件 | 说明 |
|------|------|
| `os_account.gni` | GN 构建配置定义 |
| `bundle.json` | 组件配置 |
| `hisysevent.yaml` | 系统事件配置 |
| `cfi_blocklist.txt` | CFI 跳过列表 |

---

## 9. 完整目录树（不含测试）

```
os_account/
├── dfx/
│   ├── hidumper_adapter/
│   ├── hisysevent_adapter/
│   ├── hitrace_adapter/
│   └── data_dfx/
├── figures/
├── frameworks/
│   ├── account_iam/
│   │   ├── include/
│   │   └── src/
│   ├── appaccount/
│   │   ├── cj/
│   │   │   ├── include/
│   │   │   └── src/
│   │   └── native/
│   │       ├── include/
│   │       └── src/
│   ├── authorization/
│   │   └── src/
│   ├── cj/
│   │   └── distributed_account/
│   │       ├── include/
│   │       └── src/
│   ├── common/
│   │   ├── account_error/
│   │   │   └── src/
│   │   ├── json_utils/
│   │   │   ├── include/
│   │   │   └── src/
│   │   ├── utils/
│   │   │   ├── include/
│   │   │   └── src/
│   │   ├── privileges/
│   │   │   ├── include/
│   │   │   └── src/
│   │   ├── log/
│   │   │   ├── include/
│   │   │   └── src/
│   │   ├── file_operator/
│   │   │   ├── include/
│   │   │   └── src/
│   │   └── perf_stat/
│   │       ├── include/
│   │       └── src/
│   ├── domain_account/
│   │   ├── include/
│   │   └── src/
│   ├── ets/taihe/
│   │   ├── os_account/
│   │   ├── distributed_account/
│   │   ├── app_account/
│   │   └── common/
│   ├── ohosaccount/
│   │   └── native/
│   │       ├── include/
│   │       └── src/
│   └── osaccount/
│       ├── core/
│       │   ├── include/
│       │   └── src/
│       └── native/
│           └── src/
├── interfaces/
│   ├── innerkits/
│   │   ├── account_iam/native/include/
│   │   ├── appaccount/native/include/
│   │   ├── authorization/native/include/
│   │   ├── common/
│   │   │   └── include/
│   │   ├── domain_account/native/include/
│   │   ├── ohosaccount/native/include/
│   │   └── osaccount/native/include/
│   └── kits/
│       ├── capi/
│       │   └── osaccount/
│       ├── cj/
│       │   └── osaccount/
│       └── napi/
│           ├── account_iam/
│           │   ├── include/
│           │   └── src/
│           ├── appaccount/
│           │   ├── include/
│           │   │   └── common/
│           │   └── src/
│           │       └── common/
│           ├── authorization/
│           │   ├── include/
│           │   └── src/
│           ├── common/
│           │   ├── include/
│           │   └── src/
│           ├── distributedaccount/
│           │   ├── include/
│           │   └── src/
│           ├── domain_account/
│           │   ├── include/
│           │   └── src/
│           └── osaccount/
│               ├── include/
│               └── src/
├── sa_profile/
├── services/
│   └── accountmgr/
│       ├── authorization_manager/
│       │   └── config/
│       ├── include/
│       │   ├── ability_manager_adapter/
│       │   ├── account_iam/
│       │   ├── appaccount/
│       │   ├── authorization/
│       │   ├── authorization_manager/
│       │   ├── bundle_manager_adapter/
│       │   ├── common/
│       │   │   ├── database/
│       │   │   │   ├── kvstore/
│       │   │   │   └── sqlite/
│       │   │   └── tee/
│       │   ├── domain_account/
│       │   └── osaccount/
│       └── src/
│           ├── ability_manager_adapter/
│           ├── account_iam/
│           ├── appaccount/
│           ├── authorization/
│           ├── authorization_manager/
│           ├── bundle_manager_adapter/
│           ├── common/
│           │   ├── database/
│           │   │   ├── kvstore/
│           │   │   └── sqlite/
│           │   └── tee/
│           ├── domain_account/
│           └── osaccount/
├── tools/
│   └── acm/
│       ├── include/
│       └── src/
├── BUILD.gn
├── bundle.json
├── cfi_blocklist.txt
├── hisysevent.yaml
├── os_account.gn
├── README.md
└── README_zh.md
```

---

## 10. 模块职责总结

| 模块 | 职责 | 客户端入口 | 服务端实现 |
|------|------|-----------|-----------|
| **osaccount** | 系统账号生命周期管理 | `frameworks/osaccount/` | `services/accountmgr/src/osaccount/` |
| **appaccount** | 应用账号与 OAuth | `frameworks/appaccount/` | `services/accountmgr/src/appaccount/` |
| **domain_account** | 企业域账号 | `frameworks/domain_account/` | `services/accountmgr/src/domain_account/` |
| **ohosaccount** | 分布式账号 | `frameworks/ohosaccount/` | `services/accountmgr/` |
| **account_iam** | 身份认证 | `frameworks/account_iam/` | `services/accountmgr/src/account_iam/` |
| **authorization** | 授权管理 | `frameworks/authorization/` | `services/accountmgr/src/authorization/` |

---

## 11. 相关文档

- [概述](./01_Overview.md)
- [N-API 接口](./03_NAPI_Interfaces.md)
- [Inner API](./04_Inner_API.md)
- [服务与 IPC](./05_Service_IPC.md)
