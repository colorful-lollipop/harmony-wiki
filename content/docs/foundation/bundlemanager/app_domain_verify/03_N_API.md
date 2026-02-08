# N-API 参考

> 本文档描述 app_domain_verify 部件对外暴露的 N-API，主要供应用开发者调用。

## 1. 模块概述

### 1.1 模块信息

| 属性 | 值 |
|-----|------|
| **模块名** | `bundle.appDomainVerify` |
| **实现路径** | `interfaces/kits/js/jsi/` |
| **N-API 版本** | 1 |
| **安装路径** | `system/lib/module/bundle/libappdomainverify_napi.so` |

### 1.2 头文件

```cpp
#include "app_domain_verify_manager_napi.h"
```

### 1.3 注册机制

**注册位置**: `interfaces/kits/js/jsi/src/native_module.cpp:21-44`

```cpp
static napi_module bundle_manager_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = AppDomainVerifyExport,
    .nm_modname = "bundle.appDomainVerify",
    .nm_priv = ((void*)0),
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void BundleManagerRegister(void)
{
    napi_module_register(&bundle_manager_module);
}
```

## 2. API 清单

### 2.1 queryAssociatedDomains

> 根据包名查询关联的域名列表

**签名**:
```typescript
function queryAssociatedDomains(bundleName: string): string[]
```

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|-------|------|-----|------|
| `bundleName` | string | 是 | 应用包名 |

**返回值**:
| 类型 | 描述 |
|-----|------|
| string[] | 与该包名关联的域名列表 |

**同步/异步**: 同步

**权限要求**: 系统应用（需 `ohos.permission.GET_APP_DOMAIN_BUNDLE_INFO`）

**C/C++ 实现位置**:
- 绑定入口: `interfaces/kits/js/jsi/src/app_domain_verify_manager_napi.cpp`
- 核心逻辑: `AppDomainVerifyMgrClient::QueryAssociatedDomains()`

**调用链**:
```
JS: queryAssociatedDomains(bundleName)
    │
    ▼
N-API: QueryAssociatedDomains (native_module.cpp:24)
    │
    ▼
AppDomainVerifyMgrClient::QueryAssociatedDomains()
    │
    ▼
IPC: IAppDomainVerifyMgrService::QueryAssociatedDomains()
    │
    ▼
Manager Service: AppDomainVerifyMgrService::QueryAssociatedDomains()
    │
    ▼
RDB: AppDomainVerifyRdbDataManager::Query()
```

**错误码**:
| 错误码 | 描述 |
|-------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 16300001 | 内部错误 |
| 16300002 | 权限不足 |

### 2.2 queryAssociatedBundleNames

> 根据域名查询关联的包名列表

**签名**:
```typescript
function queryAssociatedBundleNames(domain: string): string[]
```

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|-------|------|-----|------|
| `domain` | string | 是 | 域名（不含协议） |

**返回值**:
| 类型 | 描述 |
|-----|------|
| string[] | 与该域名关联的包名列表 |

**同步/异步**: 同步

**权限要求**: 系统应用（需 `ohos.permission.GET_APP_DOMAIN_BUNDLE_INFO`）

**C/C++ 实现位置**:
- 绑定入口: `interfaces/kits/js/jsi/src/app_domain_verify_manager_napi.cpp:25`
- 核心逻辑: `AppDomainVerifyMgrClient::QueryAssociatedBundleNames()`

**调用链**:
```
JS: queryAssociatedBundleNames(domain)
    │
    ▼
N-API: QueryAssociatedBundleNames (native_module.cpp:25)
    │
    ▼
AppDomainVerifyMgrClient::QueryAssociatedBundleNames()
    │
    ▼
IPC: IAppDomainVerifyMgrService::QueryAssociatedBundleNames()
    │
    ▼
Manager Service: AppDomainVerifyMgrService::QueryAssociatedBundleNames()
    │
    ▼
RDB: AppDomainVerifyRdbDataManager::Query()
```

**错误码**:
| 错误码 | 描述 |
|-------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 16300001 | 内部错误 |
| 16300002 | 权限不足 |

## 3. ANI 接口（ArkTS Native Interface）

### 3.1 概述

除了 N-API，app_domain_verify 还提供了 ANI 接口供 ArkTS 使用：

- **库文件**: `libapp_domain_verify_ani.so`
- **ABC 文件**: `app_domain_verify_ets.abc`
- **安装路径**: `system/framework/`

### 3.2 IDL 定义

**文件位置**: `interfaces/kits/js/ani/idl/ohos.bundle.appDomainVerify.taihe`

## 4. 使用示例

### 4.1 N-API 使用示例

```typescript
import bundle from '@ohos.bundle.appDomainVerify'

// 根据包名查询关联域名
let bundleName = "com.example.myapp";
let domains = bundle.queryAssociatedDomains(bundleName);
console.log(`Associated domains: ${domains.join(', ')}`);

// 根据域名查询关联包名
let domain = "www.example.com";
let bundleNames = bundle.queryAssociatedBundleNames(domain);
console.log(`Associated bundle names: ${bundleNames.join(', ')}`);
```

### 4.2 错误处理

```typescript
import bundle from '@ohos.bundle.appDomainVerify'

try {
    let domains = bundle.queryAssociatedDomains("com.example.myapp");
    console.log(`Domains: ${domains}`);
} catch (error) {
    console.error(`Error: ${error.code} - ${error.message}`);
}
```

## 5. 参数校验规则

### 5.1 bundleName 校验

| 规则 | 说明 |
|-----|------|
| 长度 | 1-256 字符 |
| 格式 | 符合包名规范（点分格式） |
| 空值 | 不允许 |

### 5.2 domain 校验

| 规则 | 说明 |
|-----|------|
| 长度 | 1-512 字符 |
| 格式 | 有效的域名格式 |
| 空值 | 不允许 |
| 协议 | 不应包含协议前缀（自动剥离） |

## 6. 线程安全

N-API 调用在 **JS 线程** 执行，内部通过 IPC 转到 Manager Service 处理。

## 7. 相关文档

| 文档 | 链接 |
|-----|------|
| Inner API 参考 | [02_Inner_API.md](./02_Inner_API.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 安全风险评审 | [06_Security.md](./06_Security.md) |
