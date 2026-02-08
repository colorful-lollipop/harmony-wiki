# 04_API 参考

> **重要说明**: 本项目是 ArkTS HAP 应用，**不包含 N-API (Native API) 绑定**。所有系统能力调用均通过 OpenHarmony 提供的 ArkTS/JS API 实现。
>
> 本文档记录应用内部 ArkTS API 接口及调用的系统 API。

---

## 1. 调用系统 API 清单

### 1.1 企业设备管理 API

#### adminManager (企业管理员管理)

**模块**: `@ohos.enterprise.adminManager`

**用途**: 管理设备管理员（启用/禁用/查询状态）

**API 列表**:

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 证据文件 |
|-----|-----|-------|---------|-----|---------|
| `enableAdmin(want, enterpriseInfo, adminType)` | Want, EnterpriseInfo, AdminType | Promise<void> | 异步 | MANAGE_ENTERPRISE_DEVICE_ADMIN | `applicationInfo.ets:295` |
| `disableAdmin(want)` | Want | Promise<void> | 异步 | MANAGE_ENTERPRISE_DEVICE_ADMIN | `applicationInfo.ets:308` |
| `isAdminEnabled(want)` | Want | Promise<boolean> | 异步 | - | `applicationInfo.ets:267` |
| `isSuperAdmin(bundleName)` | string | Promise<boolean> | 异步 | - | `applicationInfo.ets:269` |
| `getEnterpriseInfo(want)` | Want | Promise<EnterpriseInfo> | 异步 | - | `applicationInfo.ets:401` |

**AdminType 枚举**:

| 值 | 说明 |
---|-----|
| `ADMIN_TYPE_SUPER` | 超级管理员 (SDA) |
| `ADMIN_TYPE_NORMAL` | 普通管理员 (DA) |

**EnterpriseInfo 接口**:

```typescript
interface EnterpriseInfo {
  name: string;          // 企业名称
  description: string;   // 企业描述
}
```

**调用示例** (`applicationInfo.ets:285-305`):

```typescript
async activateAdmin(adminType: adminManager.AdminType) {
  let wantTemp: Want = {
    bundleName: elementNameVal.bundleName,
    abilityName: elementNameVal.abilityName,
  };
  await edmEnterpriseDeviceManager.enableAdmin(wantTemp,
    { name: enterInfo.name, description: enterInfo.description },
    edmEnterpriseDeviceManager.AdminType.ADMIN_TYPE_NORMAL)
    .catch((error: BusinessError) => {
      ret = false;
      logger.info(TAG, 'errorCode : ' + error.code + 'errorMessage : ' + error.message);
    });
}
```

#### 错误码参考

| 错误码 | 说明 |
|-------|-----|
| 0 | 成功 |
| 401 | 参数错误 |
| 16777218 | 权限不足 |
| 16777233 | 管理员已激活 |

---

### 1.2 Bundle 管理 API

**模块**: `@ohos.bundle.bundleManager`

**用途**: 查询应用 Bundle 信息

**API 列表**:

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 证据文件 |
|-----|-----|-------|---------|-----|---------|
| `getBundleInfo(bundleName, bundleFlag, userId)` | string, BundleFlag, number | Promise<BundleInfo> | 异步 | GET_BUNDLE_INFO | `appDetailData.ets:81` |
| `queryExtensionAbilityInfo(want, type, flag, userId)` | Want, ExtensionAbilityType, ExtensionAbilityFlag, number | Promise<ExtensionAbilityInfo[]> | 异步 | - | `appDetailData.ets:56` |

**BundleFlag 枚举**:

| 值 | 说明 |
---|-----|
| `GET_BUNDLE_INFO_WITH_REQUESTED_PERMISSION` | 包含请求的权限 |
| `GET_BUNDLE_INFO_WITH_APPLICATION` | 包含应用信息 |

**ExtensionAbilityType 枚举**:

| 值 | 说明 |
---|-----|
| `ENTERPRISE_ADMIN` | 企业管理员 |

**调用示例** (`appDetailData.ets:72-89`):

```typescript
async getBundleInfoItem(bundleName: string, appInfo: MyApplicationInfo) {
  let userId: UserId = { localId: 0 };
  let data = await bundle.getBundleInfo(bundleName,
    bundle.BundleFlag.GET_BUNDLE_INFO_WITH_REQUESTED_PERMISSION |
    bundle.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION,
    userId.localId);
}
```

---

### 1.3 账户管理 API

**模块**: `@ohos.account.osAccount`

**用途**: 查询当前操作系统账户

**API 列表**:

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 证据文件 |
|-----|-----|-------|---------|-----|---------|
| `getAccountManager()` | - | AccountManager | 同步 | - | `accountManager.ets:28` |
| `queryCurrentOsAccount()` | - | OsAccountInfo | 异步 | - | `accountManager.ets:28` |

**调用示例** (`accountManager.ets:27-35`):

```typescript
async getAccountUserId(userId: UserId): Promise<boolean> {
  let accountInfo = await account_osAccount.getAccountManager().queryCurrentOsAccount();
  if (!utils.isValid(accountInfo)) {
    return false;
  }
  userId.localId = accountInfo.localId;
  return true;
}
```

---

### 1.4 系统更新 API

**模块**: `@ohos.update`

**用途**: 系统恢复出厂设置

**API 列表**:

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 证据文件 |
|-----|-----|-------|---------|-----|---------|
| `getRestorer()` | - | Restorer | 同步 | - | `resetFactory.ets:24` |
| `factoryReset()` | - | Promise<void> | 异步 | FACTORY_RESET | `resetFactory.ets:25` |

**调用示例** (`resetFactory.ets:23-30`):

```typescript
rebootAndCleanUserData() {
  let restorer = update.getRestorer();
  restorer.factoryReset().then(() => {
    logger.info(TAG, 'rebootAndCleanUserData factoryReset success')
  }).catch((err: BusinessError) => {
    logger.error(TAG, 'rebootAndCleanUserData err=' + JSON.stringify(err))
  })
}
```

---

### 1.5 配置策略 API

**模块**: `@ohos.configPolicy`

**用途**: 获取系统配置文件路径

**API 列表**:

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 证据文件 |
|-----|-----|-------|---------|-----|---------|
| `getOneCfgFile(relativePath)` | string | Promise<string> | 异步 | - | `UIExtensionAbility.ets:43` |

**调用示例** (`UIExtensionAbility.ets:42-62`):

```typescript
let realpath = 'etc/edm/edm_provision_config.json';
await configPolicy.getOneCfgFile(realpath).then((value: string) => {
  let configStr = fs.readTextSync(value);
  let jsonArray = JSON.parse(configStr);
  // 提取配置信息...
})
```

---

### 1.6 文件系统 API

**模块**: `@ohos.file.fs`

**用途**: 文件读写操作

**API 列表**:

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 证据文件 |
|-----|-----|-------|---------|-----|---------|
| `readTextSync(filePath)` | string | string | 同步 | - | `UIExtensionAbility.ets:45` |

---

## 2. 应用内部 API

### 2.1 日志工具

**模块**: `common/logger.ts`

**用途**: 统一日志输出

**类**: `Logger`

| 方法 | 参数 | 用途 | 证据文件 |
|-----|-----|-----|---------|
| `debug(...args)` | unknown[] | debug 级别日志 | `logger.ts:28` |
| `info(...args)` | unknown[] | info 级别日志 | `logger.ts:32` |
| `warn(...args)` | unknown[] | warn 级别日志 | `logger.ts:36` |
| `error(...args)` | unknown[] | error 级别日志 | `logger.ts:40` |

**日志域**: `0x6977`

**使用示例**:

```typescript
import logger from '../common/logger';

const TAG = 'MyComponent';

logger.info(TAG, 'operation success');
logger.error(TAG, 'operation failed: ' + JSON.stringify(err));
```

---

### 2.2 工具函数

**模块**: `common/utils.ts`

**用途**: 通用工具函数

**类**: `Utils`

| 方法 | 参数 | 返回值 | 用途 | 证据文件 |
|-----|-----|-------|-----|---------|
| `isValid(item)` | unknown | boolean | 检查对象有效性 | `utils.ts:21` |
| `checkObjPropertyValid(obj, tree)` | T, string | boolean | 检查嵌套属性 | `utils.ts:25` |
| `isLargeDevice()` | - | number | 判断大屏设备 | `utils.ts:42` |

**使用示例**:

```typescript
import utils from '../common/utils';

// 检查对象有效性
if (!utils.isValid(data)) {
  return;
}

// 检查嵌套属性
if (!utils.checkObjPropertyValid(data, 'parameters.elementName.abilityName')) {
  logger.warn(TAG, 'invalid parameters');
}
```

---

### 2.3 账户管理

**模块**: `common/accountManager.ets`

**用途**: 账户相关操作

**类**: `AccountManager`

| 方法 | 参数 | 返回值 | 用途 | 证据文件 |
|-----|-----|-------|-----|---------|
| `getAccountUserId(userId)` | UserId | Promise<boolean> | 获取当前账户 ID | `accountManager.ets:27` |

---

### 2.4 应用详情数据

**模块**: `common/appManagement/appDetailData.ets`

**用途**: 应用 Bundle 信息查询

**类**: `AppDetailData`

| 方法 | 参数 | 返回值 | 用途 | 证据文件 |
|-----|-----|-------|-----|---------|
| `checkAppItem(elementNameVal)` | Want | Promise<boolean> | 检查是否为管理员应用 | `appDetailData.ets:40` |
| `getBundleInfoItem(bundleName, appInfo)` | string, MyApplicationInfo | Promise<void> | 获取 Bundle 信息 | `appDetailData.ets:72` |
| `terminateAbilityPage()` | - | Promise<void> | 终止当前页面 | `appDetailData.ets:145` |
| `getPermissionList(data, appInfo)` | BundleInfo, MyApplicationInfo | Promise<void> | 获取权限列表 | `appDetailData.ets:159` |

---

## 3. 数据结构定义

### 3.1 MyApplicationInfo

**定义** (`myApplicationInfo.ets:16-21`):

```typescript
export interface MyApplicationInfo {
  appIcon: string;              // 应用图标 (Base64)
  appTitle: string;             // 应用标题
  appBundleName: string;        // Bundle 名称
  appPermissionList: AppPermission[]; // 权限列表
}
```

### 3.2 AppPermission

**定义** (`myApplicationInfo.ets:23-27`):

```typescript
export interface AppPermission {
  permissionName: string;       // 权限名称
  permissionLabel: Resource;    // 权限标签 (资源)
  permissionDescription: Resource; // 权限描述 (资源)
}
```

### 3.3 UserId

**定义** (`accountManager.ets:22-24`):

```typescript
export interface UserId {
  localId: number;  // 用户 ID
}
```

---

## 4. 权限清单

### 4.1 声明的权限

**来源**: `module.json5:63-79`

| 权限名称 | 用途 | 使用位置 |
|---------|-----|---------|
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 管理本地账户 | `accountManager` |
| `ohos.permission.GET_BUNDLE_INFO` | 获取 Bundle 信息 | `appDetailData` |
| `ohos.permission.MANAGE_ENTERPRISE_DEVICE_ADMIN` | 管理企业设备管理员 | `adminManager` |
| `ohos.permission.FACTORY_RESET` | 恢复出厂设置 | `resetFactory` |
| `ohos.permission.INTERNET` | 网络访问 | - |
| `ohos.permission.PROVISIONING_MESSAGE` | 发放消息 | `AutoManagerAbility` |

---

*文档版本: 1.0*
