# 06_Build_System.md

# OpenHarmony os_account GN 构建系统

> 本文档描述 os_account 子系统的 GN 构建配置与编译产物。

---

## 1. 构建配置概览

### 1.1 根配置文件

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根目录构建入口 |
| `os_account.gni` | GN 配置定义与路径变量 |
| `bundle.json` | 组件配置与依赖声明 |

### 1.2 os_account.gni 主要变量

**文件**: `os_account.gni`

```gn
# 基础路径
os_account_path = "//base/account/os_account"
common_path = "${os_account_path}/frameworks/common"
services_path = "${os_account_path}/services"
tools_path = "${os_account_path}/tools"

# 模块路径
app_account_core_path = "${os_account_path}/frameworks/appaccount/core"
app_account_interfaces_native_path = "${os_account_path}/interfaces/innerkits/appaccount/native"
app_account_innerkits_native_path = "${os_account_path}/frameworks/appaccount/native"
app_account_kits_path = "${os_account_path}/interfaces/kits/napi/appaccount"

os_account_interfaces_native_path = "${os_account_path}/interfaces/innerkits/osaccount/native"
os_account_innerkits_native_path = "${os_account_path}/frameworks/osaccount/native"
os_account_core_path = "${os_account_path}/frameworks/osaccount/core"
os_account_kits_path = "${os_account_path}/interfaces/kits/napi/osaccount"

account_iam_kits_path = "${os_account_path}/interfaces/kits/napi/account_iam"
account_iam_interfaces_native_path = "${os_account_path}/interfaces/innerkits/account_iam/native"

domain_account_napi_path = "${os_account_path}/interfaces/kits/napi/domain_account"
domain_account_interfaces_native_path = "${os_account_path}/interfaces/innerkits/domain_account/native"

authorization_kits_path = "${os_account_path}/interfaces/kits/napi/authorization"
authorization_interfaces_native_path = "${os_account_path}/interfaces/innerkits/authorization/native"
```

---

## 2. Feature Flags

### 2.1 账号管理类开关

```gn
declare_args() {
  os_account_multiple_active_accounts = true
  os_account_support_deactivate_main_os_account = false
  os_account_enable_multiple_os_accounts = true
  os_account_enable_default_admin_name = true
  os_account_enable_account_short_name = false
  os_account_support_lock_os_account = false
  os_account_activate_last_logged_in_account = false
  os_account_enable_account_1 = false
  os_account_support_authorization = false
  os_account_enable_multiple_foreground_os_accounts = false
  os_account_distributed_feature = true
  os_account_support_domain_accounts = true
}
```

### 2.2 可选依赖开关

```gn
# 条件编译开关
if (!defined(global_parts_info) ||
    defined(global_parts_info.useriam_user_auth_framework)) {
  has_user_auth_part = true
  has_user_idm_part = true
} else {
  has_user_auth_part = false
  has_user_idm_part = false
}

if (!defined(global_parts_info) ||
    defined(global_parts_info.useriam_pin_auth) ||
    defined(global_parts_info.useriam_user_auth_framework)) {
  has_pin_auth_part = true
} else {
  has_pin_auth_part = false
}

if (!defined(global_parts_info) ||
    defined(global_parts_info.distributeddatamgr_kv_store)) {
  has_kv_store_part = true
  has_app_account_part = true
} else {
  has_kv_store_part = false
  has_app_account_part = false
}
```

---

## 3. bundle.json 配置

### 3.1 组件信息

```json
{
  "name": "@ohos/os_account",
  "version": "3.0",
  "subsystem": "account",
  "syscap": [
    "SystemCapability.Account.AppAccount",
    "SystemCapability.Account.OsAccount"
  ],
  "features": [
    "os_account_multiple_active_accounts",
    "os_account_support_deactivate_main_os_account",
    "os_account_distributed_feature",
    "os_account_enable_multiple_foreground_os_accounts",
    "os_account_enable_multiple_os_accounts",
    "os_account_enable_default_admin_name",
    "os_account_enable_account_short_name",
    "os_account_activate_last_logged_in_account",
    "os_account_support_domain_accounts",
    "os_account_enable_account_1",
    "os_account_support_lock_os_account",
    "os_account_support_authorization"
  ]
}
```

### 3.2 依赖组件

```json
"deps": {
  "components": [
    "ability_base",
    "ability_runtime",
    "access_token",
    "ace_engine",
    "asset",
    "cJSON",
    "bundle_framework",
    "common_event_service",
    "c_utils",
    "eventhandler",
    "kv_store",
    "hicollie",
    "hilog",
    "hisysevent",
    "hitrace",
    "huks",
    "init",
    "ipc",
    "napi",
    "pin_auth",
    "runtime_core",
    "safwk",
    "samgr",
    "security_guard",
    "selinux_adapter",
    "sqlite",
    "storage_service",
    "time_service",
    "user_auth_framework",
    "window_manager",
    "openssl",
    "config_policy",
    "icu",
    "tee_client"
  ]
}
```

---

## 4. 关键 Targets

### 4.1 基础组 (base_group)

| Target | 路径 | 输出 |
|--------|------|------|
| `capi_packages` | `interfaces/kits/capi` | C API 包 |
| `napi_packages` | `interfaces/kits/napi` | N-API 包 |
| `account_taihe` | `frameworks/ets/taihe` | 静态 Native API |
| `account_sa_profile` | `sa_profile` | SA 配置文件 |
| `tools_acm` | `tools` | ACM 工具 |

### 4.2 框架组 (fwk_group)

| Target | 路径 | 输出 |
|--------|------|------|
| `app_account_innerkits` | `frameworks/appaccount/native` | 应用账号框架库 |
| `authorization_innerkits` | `frameworks/authorization` | 授权框架库 |
| `common_target` | `frameworks/common` | 共通工具库 |
| `domain_account_innerkits` | `frameworks/domain_account` | 域账号框架库 |
| `libaccountkits` | `frameworks/ohosaccount/native` | 分布式账号库 |
| `os_account_innerkits` | `frameworks/osaccount/native` | OS 账号框架库 |

### 4.3 服务组 (service_group)

| Target | 路径 | 输出 |
|--------|------|------|
| `services_target` | `services` | 主服务库 |
| `app_account_service_core` | `services/accountmgr/src/appaccount` | 应用账号服务 |
| `param_files` | `services/accountmgr/param` | 参数文件 |

---

## 5. Inner Kits 清单

### 5.1 Inner Kit 头文件

| 头文件 | 路径 | 说明 |
|--------|------|------|
| `os_account_manager.h` | `interfaces/innerkits/osaccount/native/include/` | OS 账号管理 |
| `os_account_info.h` | `interfaces/innerkits/osaccount/native/include/` | OS 账号信息 |
| `app_account_manager.h` | `interfaces/innerkits/appaccount/native/include/` | 应用账号管理 |
| `ohos_account_kits.h` | `interfaces/innerkits/ohosaccount/native/include/` | 分布式账号 |
| `account_iam_client.h` | `interfaces/innerkits/account_iam/native/include/` | IAM 客户端 |
| `domain_account_client.h` | `interfaces/innerkits/domain_account/native/include/` | 域账号客户端 |
| `authorization_client.h` | `interfaces/innerkits/authorization/native/include/` | 授权客户端 |

---

## 6. 编译产物

### 6.1 主要库文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `libaccountmgr.z.so` | `out/${product}/accountmgr/` | 主服务库 |
| `libaccountkits.z.so` | `out/${product}/accountmgr/` | 分布式账号库 |
| `libos_account.z.so` | `out/${product}/accountmgr/` | OS 账号库 |
| `libapp_account.z.so` | `out/${product}/accountmgr/` | 应用账号库 |
| `libaccount_iam.z.so` | `out/${product}/accountmgr/` | IAM 库 |
| `libaccount_auth.z.so` | `out/${product}/accountmgr/` | 授权库 |
| `libaccount_common.z.so` | `out/${product}/accountmgr/` | 共通库 |

### 6.2 N-API 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `@ohos.account.osAccount` | `interfaces/kits/napi/` | OS 账号 JS API |
| `@ohos.account.appAccount` | `interfaces/kits/napi/` | 应用账号 JS API |
| `@ohos.account.distributedAccount` | `interfaces/kits/napi/` | 分布式账号 JS API |

### 6.3 工具产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `acm` | `tools/acm/` | 账号管理命令行工具 |

---

## 7. 构建命令

### 7.1 完整构建

```bash
./build.sh --product-name <product> --build-target os_account \
  account_build_unittest account_build_moduletest
```

### 7.2 分别构建

```bash
# 构建主服务
./build.sh --product-name <product> --build-target accountmgr

# 构建单元测试
./build.sh --product-name <product> --build-target account_build_unittest

# 构建模块测试
./build.sh --product-name <product> --build-target account_build_moduletest

# 构建模糊测试
./build.sh --product-name <product> --build-target account_build_fuzztest \
  --gn-args use_thin_lto=false
```

---

## 8. 输出目录结构

```
out/${product}/
├── accountmgr/
│   ├── libaccountmgr.z.so          # 主服务
│   ├── libaccountkits.z.so         # 分布式账号
│   ├── libos_account.z.so           # OS 账号
│   ├── libapp_account.z.so        # 应用账号
│   ├── libaccount_iam.z.so         # IAM
│   ├── libaccount_auth.z.so        # 授权
│   ├── libaccount_common.z.so     # 共通
│   └── ...
├── accountmgr_test/
│   ├── xxx_test                   # 测试可执行文件
│   └── ...
└── ...
```

---

## 9. 相关文档

- [概述](./01_Overview.md)
- [目录结构](./02_Directory_Structure.md)
- [配置与 Feature Flags](./08_Config_Flags.md)
