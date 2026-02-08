# 对外接口 (Public API)

## 目的

本文档描述DLP Manager对外提供的接口，包括Ability调用方式、参数说明和错误码定义。第三方应用可通过这些接口调用DLP功能。

## 适用范围

- 需要集成DLP能力的第三方开发者
- 需要调用DLP功能的系统应用开发者
- 进行接口评审的架构师

---

## 接口概览

DLP Manager作为系统应用，对外接口通过 **Ability调用** 方式提供（非N-API）。

### 接口分类

| 接口类型 | Ability | 功能 |
|---------|---------|------|
| DLP设置/修改 | MainAbilityEx | 生成DLP文件、修改DLP权限 |
| DLP打开 | ViewAbility | 打开DLP文件 |
| 加密分享 | EncryptedSharingAbility | DLP文件加密分享 |

---

## 1. MainAbilityEx - DLP权限设置

### 接口说明

**Ability名称**: `MainAbilityEx`  
**Ability类型**: UIExtensionAbility（`sys/commonUI`）  
**BundleName**: `com.ohos.dlpmanager`  
**是否导出**: 是（`exported: true`）

**配置位置**: `entry/src/main/module.json:31-43`

### 调用方式

```typescript
import common from '@ohos.app.ability.common';
import Want from '@ohos.app.ability.Want';

// 获取上下文
const context = getContext(this) as common.UIAbilityContext;

// 构造Want
const want: Want = {
  bundleName: 'com.ohos.dlpmanager',
  abilityName: 'MainAbilityEx',
  uri: 'file://docs/storage/Users/currentUser/Documents/test.doc',  // 原始文件URI
  parameters: {
    fileName: {
      name: 'test.doc'  // 文件名
    },
    callerToken: 123456,        // 调用者tokenId
    callerBundleName: 'com.example.app'  // 调用者bundleName
  }
};

// 启动Ability
context.startAbility(want)
  .then(() => {
    console.info('Start DLP Manager success');
  })
  .catch((err) => {
    console.error('Start DLP Manager failed', err);
  });
```

### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| bundleName | string | 是 | 固定值：`com.ohos.dlpmanager` |
| abilityName | string | 是 | 固定值：`MainAbilityEx` |
| uri | string | 是 | 原始文件URI（`file://`开头） |
| parameters.fileName.name | string | 是 | 文件名 |
| parameters.callerToken | number | 是 | 调用者tokenId |
| parameters.callerBundleName | string | 是 | 调用者bundleName |
| parameters.linkFileName | object | 否 | 链接文件名（沙箱内打开时） |

### 参数校验

**代码位置**: `entry/src/main/ets/Ability/MainAbilityEx.ets:197-246`

校验逻辑：
1. `parameters` 必须存在
2. `fileName.name` 必须存在
3. `uri` 必须存在且以 `file://` 开头
4. `callerToken` 和 `callerBundleName` 必须存在
5. URI不能包含 `/mnt/data/fuse/` 路径
6. URI不能是已打开的链接文件

### 返回结果

DLP Manager通过 `terminateSelfWithResult` 返回结果：

```typescript
// 成功
{
  resultCode: 0,
  want: {
    parameters: {
      result: 'success'
    }
  }
}

// 失败
{
  resultCode: 1,
  want: {
    parameters: {
      result: 'error'
    }
  }
}
```

---

## 2. ViewAbility - DLP文件打开

### 接口说明

**Ability名称**: `ViewAbility`  
**Ability类型**: ServiceExtensionAbility  
**BundleName**: `com.ohos.dlpmanager`  
**是否导出**: 否（`exported: false`）

**配置位置**: `entry/src/main/module.json:45-51`

**注意**: 此Ability不直接对外导出，通常由文件管理应用或系统在用户点击DLP文件时调用。

### 调用方式

```typescript
import common from '@ohos.app.ability.common';
import Want from '@ohos.app.ability.Want';

const context = getContext(this) as common.UIAbilityContext;

const want: Want = {
  bundleName: 'com.ohos.dlpmanager',
  abilityName: 'ViewAbility',
  uri: 'file://docs/storage/Users/currentUser/Documents/test.doc.dlp',  // DLP文件URI
  parameters: {
    fileName: {
      name: 'test.doc.dlp'
    },
    callerToken: 123456,
    callerBundleName: 'com.example.app',
    // 沙箱相关参数
    bundleName: 'com.example.targetapp',    // 用于打开文件的应用
    moduleName: 'entry',                     // 模块名
    abilityName: 'MainAbility'               // Ability名
  }
};

context.startAbility(want)
  .then(() => {
    console.info('Open DLP file success');
  })
  .catch((err) => {
    console.error('Open DLP file failed', err);
  });
```

### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| bundleName | string | 是 | 固定值：`com.ohos.dlpmanager` |
| abilityName | string | 是 | 固定值：`ViewAbility` |
| uri | string | 是 | DLP文件URI（`.dlp`后缀） |
| parameters.fileName.name | string | 是 | DLP文件名 |
| parameters.callerToken | number | 是 | 调用者tokenId |
| parameters.callerBundleName | string | 是 | 调用者bundleName |
| parameters.bundleName | string | 否 | 目标应用bundleName |
| parameters.moduleName | string | 否 | 目标应用模块名 |
| parameters.abilityName | string | 否 | 目标应用Ability名 |

### 处理流程

**代码位置**: `entry/src/main/ets/Ability/ViewAbility.ets:48`

1. 接收打开请求
2. 检查是否正在解密中
3. 调用 `OpenDlpFileProcessor.process()` 处理
4. 解析DLP文件
5. 验证用户权限
6. 创建沙箱并安装应用
7. 启动目标应用打开解密后的文件

---

## 3. EncryptedSharingAbility - 加密分享

### 接口说明

**Ability名称**: `EncryptedSharingAbility`  
**Ability类型**: ServiceExtensionAbility（`action`类型）  
**BundleName**: `com.ohos.dlpmanager`  
**是否导出**: 是（`exported: true`）  
**ExtensionProcessMode**: `instance`（每个调用独立实例）

**配置位置**: `entry/src/main/module.json:69-77`

### 调用方式

```typescript
import common from '@ohos.app.ability.common';
import Want from '@ohos.app.ability.Want';

const context = getContext(this) as common.UIAbilityContext;

const want: Want = {
  bundleName: 'com.ohos.dlpmanager',
  abilityName: 'EncryptedSharingAbility',
  action: 'ohos.want.action.send',  // 分享动作
  parameters: {
    fileName: {
      name: 'test.doc.dlp'
    },
    // 分享相关参数
  }
};

context.startAbility(want)
  .then(() => {
    console.info('Start encrypted sharing success');
  })
  .catch((err) => {
    console.error('Start encrypted sharing failed', err);
  });
```

---

## 4. 系统接口使用

DLP Manager内部使用以下系统接口：

### 4.1 @ohos.dlpPermission

**用途**: DLP核心功能

| 接口 | 说明 |
|------|------|
| `generateDlpFile()` | 生成DLP文件 |
| `openDlpFile()` | 打开DLP文件 |
| `closeDlpFile()` | 关闭DLP文件 |
| `installDlpSandbox()` | 安装DLP沙箱 |
| `uninstallDlpSandbox()` | 卸载DLP沙箱 |

### 4.2 @ohos.file.fs

**用途**: 文件操作

| 接口 | 说明 |
|------|------|
| `open()` / `openSync()` | 打开文件 |
| `read()` / `readSync()` | 读取文件 |
| `copyFile()` | 复制文件 |
| `mkdir()` | 创建目录 |

---

## 5. 错误码定义

### 5.1 应用层错误码 (0-200)

**定义位置**: `entry/src/main/ets/common/constant.ets:134-170`

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | ERR_JS_APP_ACCOUNT_INFO | 账号信息正常 |
| 1 | ERR_JS_APP_INSIDE_ERROR | 应用内部错误 |
| 2 | ERR_JS_GET_ACCOUNT_ERROR | 获取账号失败 |
| 3 | ERR_JS_APP_NO_ACCOUNT_ERROR | 无账号错误 |
| 4 | ERR_JS_APP_PARAM_ERROR | 参数错误 |
| 5 | ERR_JS_APP_GET_FILE_ASSET_ERROR | 获取文件资源失败 |
| 6 | ERR_JS_APP_OPEN_REJECTED | 打开被拒绝 |
| 7 | ERR_JS_APP_ENCRYPTION_REJECTED | 加密被拒绝 |
| 8 | ERR_JS_APP_SYSTEM_IS_AUTHENTICATED | 系统已认证 |
| 9 | ERR_JS_APP_NETWORK_INVALID | 网络无效 |
| 10 | ERR_JS_APP_ENCRYPTING | 正在加密中 |
| 11 | ERR_JS_APP_CANNOT_OPEN | 无法打开 |
| 13 | ERR_JS_RELEASE_FILE_OPEN | 释放文件打开 |
| 201 | ERR_JS_APP_PERMISSION_DENY | 权限拒绝 |

### 5.2 DLP服务错误码 (19100000+)

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 19100003 | ERR_JS_CREDENTIAL_TIMEOUT | 凭证超时 |
| 19100004 | ERR_JS_CREDENTIAL_SERVICE_ERROR | 凭证服务错误 |
| 19100005 | ERR_JS_CREDENTIAL_SERVER_ERROR | 凭证服务器错误 |
| 19100008 | ERR_JS_NOT_DLP_FILE | 不是DLP文件 |
| 19100010 | ERR_JS_DLP_FILE_READ_ONLY | DLP文件只读 |
| 19100013 | ERR_JS_USER_NO_PERMISSION | 用户无权限 |
| 19100014 | ERR_JS_ACCOUNT_NOT_LOGIN | 账号未登录 |
| 19100015 | ERR_JS_SYSTEM_NEED_TO_BE_UPGRADED | 系统需要升级 |
| 19100018 | ERR_JS_NOT_AUTHORIZED_APPLICATION | 未授权应用 |
| 19100019 | ERR_JS_FILE_EXPIRATION | 文件已过期 |
| 19100020 | ERR_JS_OFFLINE | 离线状态 |
| 19100021 | ERR_JS_NO_SPACE | 空间不足 |
| 19100022 | ERR_JS_DLP_ALLOWED_OPEN_COUNT_INVALID | 打开次数无效 |

### 5.3 内部处理错误码 (-1 ~ -15)

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | ERR_CODE_SUCCESS | 成功 |
| -1 | ERR_CODE_OPEN_FILE_ERROR | 打开文件错误 |
| -2 | ERR_CODE_PARAMS_CHECK_ERROR | 参数检查错误 |
| -3 | ERR_CODE_CREATE_DECRYPT_TASK_ERROR | 创建解密任务错误 |
| -4 | ERR_CODE_NETWORK_ERROR | 网络错误 |
| -5 | ERR_CODE_PARSE_DLP_FILE_ERROR | 解析DLP文件错误 |
| -6 | ERR_CODE_START_ABILITY_ERROR | 启动Ability错误 |
| -7 | ERR_CODE_GET_BUNDLE_INFO_ERROR | 获取包信息错误 |
| -8 | ERR_CODE_GET_LOCK_ASYNC_ERROR | 获取异步锁错误 |
| -9 | ERR_CODE_FILE_IS_DECRYPTING_ERROR | 文件正在解密错误 |
| -10 | ERR_CODE_USER_STOP_DIALOG | 用户停止对话框 |
| -11 | ERR_CODE_DECRYPT_TIME_OUT | 解密超时 |
| -12 | ERR_JS_USER_NO_PERMISSION_2B | 用户无权限(2B) |
| -13 | ERR_JS_USER_NO_PERMISSION_2C | 用户无权限(2C) |
| -14 | ERR_JS_DOMAIN_NO_ACCOUNT | 域无账号 |
| -15 | ERR_JS_INSTALL_SANDBOX_ERROR | 安装沙箱错误 |

---

## 6. 权限要求

调用DLP Manager的Ability需要以下权限：

| 权限 | 用途 | 级别 |
|------|------|------|
| ohos.permission.ACCESS_DLP_FILE | 访问DLP文件 | system_grant |
| ohos.permission.GET_BUNDLE_INFO | 获取包信息 | normal |
| ohos.permission.START_ABILITIES_FROM_BACKGROUND | 后台启动Ability | system_grant |

**声明位置**: `entry/src/main/module.json:95-126`

---

## 7. 调用示例

### 7.1 完整调用示例（生成DLP文件）

```typescript
import common from '@ohos.app.ability.common';
import Want from '@ohos.app.ability.Want';
import bundleManager from '@ohos.bundle.bundleManager';

async function generateDlpFile(fileUri: string, fileName: string): Promise<void> {
  const context = getContext(this) as common.UIAbilityContext;
  
  // 获取当前应用信息
  const bundleInfo = await bundleManager.getBundleInfoForSelf(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_SIGNATURE_INFO
  );
  
  const want: Want = {
    bundleName: 'com.ohos.dlpmanager',
    abilityName: 'MainAbilityEx',
    uri: fileUri,
    parameters: {
      fileName: { name: fileName },
      callerToken: bundleInfo.appInfo.accessTokenId,
      callerBundleName: bundleInfo.name
    }
  };
  
  try {
    await context.startAbilityForResult(want);
    console.info('DLP file generated successfully');
  } catch (err) {
    console.error('Failed to generate DLP file:', err);
    throw err;
  }
}
```

---

## 关键结论

1. **Ability接口**: DLP Manager通过Ability调用方式对外提供服务（非N-API）
2. **两个入口**: MainAbilityEx（设置权限）和 ViewAbility（打开文件）
3. **参数严格校验**: 所有入口都有严格的参数校验，防止非法调用
4. **权限控制**: 需要特定系统权限才能调用
5. **错误码分层**: 应用层错误码、服务层错误码、内部错误码三层体系

---

## 相关链接

- [架构设计](10_Architecture.md) - 了解接口背后的架构设计
- [安全风险](60_Security_Analysis.md) - 了解接口安全分析
- [调试指南](70_Debugging.md) - 了解接口调用调试
