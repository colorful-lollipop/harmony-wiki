# 对外 API 清单

> 本文档列出用户证书管理部件使用的所有对外 API 接口

---

## 目的

本文档提供完整的对外 API 清单，包括接口名称、参数、返回值、调用位置。

## 适用范围

- OpenHarmony 应用开发者
- 需要了解外部依赖的技术人员
- API 文档编写者

## 关键结论

1. **无 N-API 模块**: 本项目为纯 ArkTS 应用，无自定义 N-API 模块
2. **主要依赖**: `@ohos.security.certManager` 证书管理服务
3. **API 数量**: 20+ 个外部 API 调用
4. **调用方式**: 全部为异步 API (async/await)

## 相关跳转

- [00_Overview.md](wiki/00_Overview.md) - 项目概览
- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计
- [05_Internal_API.md](wiki/05_Internal_API.md) - 内部接口

---

## 1. N-API 模块说明

### 1.1 确认结果

**结论**: 本项目无 N-API 模块

**证据**:
- 项目中无 C/C++ 源文件
- 无 `napi_` 前缀函数调用
- 无 `NAPI_MODULE` 或 `napi_module_register` 宏定义
- 全部为 ArkTS (.ets/.ts) 源文件

**证据位置**: 搜索结果确认无 N-API 相关代码

---

## 2. 证书管理服务 API

### 2.1 API 清单表

#### 系统 CA 证书管理

| API 名称 | 调用位置 | 异步 | 说明 |
|---------|----------|------|------|
| `getSystemTrustedCertificateList()` | `CertMangerModel.ets:323` | ✓ | 获取系统 CA 证书列表 |
| `getSystemTrustedCertificate(uri)` | `CertMangerModel.ets:393` | ✓ | 获取系统 CA 证书详情 |

**返回类型**: `CertManager.CMResult`

**错误码**:
- 通用错误 (BusinessError)
- 证书不存在
- 格式错误

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:319-421`

---

#### 用户 CA 证书管理

| API 名称 | 调用位置 | 异步 | 说明 |
|---------|----------|------|------|
| `getAllUserTrustedCertificates()` | `CertMangerModel.ets:427` | ✓ | 获取用户 CA 证书列表 |
| `getUserTrustedCertificate(uri)` | `CertMangerModel.ets:458` | ✓ | 获取用户 CA 证书详情 |
| `installUserTrustedCertificate({inData, alias, certFormat, certScope})` | `CertMangerModel.ets:709` | ✓ | 安装用户 CA 证书 |
| `uninstallUserTrustedCertificate(uri)` | `CertMangerModel.ets:493` | ✓ | 删除用户 CA 证书 |
| `setCertificateStatus(uri, store, status)` | `CertMangerModel.ets:507` | ✓ | 设置证书启用/禁用状态 |
| `uninstallAllUserTrustedCertificate()` | `CertMangerModel.ets:636` | ✓ | 删除所有用户 CA 证书 |

**参数类型**:
- `inData`: Uint8Array - 证书数据
- `alias`: string - 证书别名
- `certFormat`: CertFileFormat - 证书格式 (PEM_DER / P7B)
- `certScope`: CertScope - 证书范围 (CURRENT_USER / ALL_USERS)
- `uri`: string - 证书 URI
- `store`: number - 存储类型 (1=系统, 2=用户)
- `status`: boolean - 启用状态

**错误码**:
- `CM_ERROR_INCORRECT_FORMAT`: 格式错误
- `CM_ERROR_MAX_CERT_COUNT_REACHED`: 达到最大数量
- `CM_ERROR_ALIAS_LENGTH_REACHED_LIMIT`: 别名过长

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:423-515`, `certmanager/src/main/ets/model/CertMangerModel.ets:697-730`, `certmanager/src/main/ets/model/CertMangerModel.ets:633-644`

---

#### 用户凭据管理

| API 名称 | 调用位置 | 异步 | 说明 |
|---------|----------|------|------|
| `getAllPublicCertificates()` | `CertMangerModel.ets:520` | ✓ | 获取用户凭据列表 |
| `getPublicCertificate(uri)` | `CertMangerModel.ets:566` | ✓ | 获取用户凭据详情 |
| `installPublicCertificate(data, pwd, alias)` | `CertMangerModel.ets:740` | ✓ | 安装用户凭据 |
| `uninstallPublicCertificate(uri)` | `CertMangerModel.ets:610` | ✓ | 删除用户凭据 |
| `uninstallAllAppCertificate()` | `CertMangerModel.ets:649` | ✓ | 删除所有用户凭据 |
| `getAuthorizedAppList(uri)` | `CertMangerModel.ets:662` | ✓ | 获取授权应用列表 |
| `grantPublicCertificate(uri, appUid)` | `CertMangerModel.ets:681` | ✓ | 授予应用访问权限 |
| `removeGrantedPublicCertificate(uri, appUid)` | `CertMangerModel.ets:686` | ✓ | 移除应用访问权限 |

**参数类型**:
- `data`: Uint8Array - 凭据数据
- `pwd`: string - 凭据密码
- `alias`: string - 凭据别名
- `uri`: string - 凭据 URI
- `appUid`: string - 应用 UID

**错误码**:
- `CM_ERROR_INCORRECT_FORMAT`: 格式错误
- `CM_ERROR_MAX_CERT_COUNT_REACHED`: 达到最大数量
- `CM_ERROR_ALIAS_LENGTH_REACHED_LIMIT`: 别名过长
- `CM_ERROR_PASSWORD_IS_ERR`: 密码错误

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:517-695`

---

#### 系统凭据管理

| API 名称 | 调用位置 | 异步 | 说明 |
|---------|----------|------|------|
| `getAllSystemAppCertificates()` | `CertMangerModel.ets:543` | ✓ | 获取系统凭据列表 |
| `getSystemAppCertificate(uri)` | `CertMangerModel.ets:588` | ✓ | 获取系统凭据详情 |
| `installSystemAppCertificate(data, pwd, alias)` | `CertMangerModel.ets:768` | ✓ | 安装系统凭据 |
| `uninstallSystemAppCertificate(uri)` | `CertMangerModel.ets:623` | ✓ | 删除系统凭据 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:540-631`, `certmanager/src/main/ets/model/CertMangerModel.ets:760-786`

---

#### 扩展功能

| API 名称 | 调用位置 | 异步 | 说明 |
|---------|----------|------|------|
| `getAllAppPrivateCertificatesByUid(appUid)` | `CertMangerModel.ets:791` | ✓ | 获取应用私钥凭据 |
| `getUkeyCertificateList('', {certPurpose})` | `CertMangerModel.ets:816` | ✓ | 获取 UKey 证书列表 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:788-834`

---

## 3. 用户认证 API

### 3.1 API 清单表

| API 名称 | 导入路径 | 异步 | 说明 |
|---------|----------|------|------|
| `getAvailableStatus(authType, authTrustLevel)` | `@ohos.userIAM.userAuth` | ✗ | 检查认证类型是否支持 |
| `getUserAuthInstance(authParam, widgetParam)` | `@ohos.userIAM.userAuth` | ✗ | 获取认证实例 |
| `userAuthInstance.start()` | `@ohos.userIAM.userAuth` | ✗ | 启动认证 |
| `userAuthInstance.on('result', callback)` | `@ohos.userIAM.userAuth` | ✗ | 监听认证结果 |

### 3.2 认证流程

```typescript
// 1. 检查支持的认证类型
userAuth.getAvailableStatus(authType, authTrustLevel)

// 2. 生成随机数 challenge
const randomData: Uint8Array = getRandomData();

// 3. 创建认证实例
const authParam: userAuth.AuthParam = {
  challenge: randomData,
  authType: authTypeArray,
  authTrustLevel: userAuth.AuthTrustLevel.ATL1
}
const widgetParam: userAuth.WidgetParam = {
  title: titleStr
};
let userAuthInstance = userAuth.getUserAuthInstance(authParam, widgetParam);

// 4. 启动认证
userAuthInstance.start();

// 5. 监听认证结果
userAuthInstance.on('result', {
  onResult(result) {
    if (result.result === userAuth.UserAuthResultCode.SUCCESS) {
      // 认证成功
    } else {
      // 认证失败或取消
    }
  }
})
```

**证据位置**: `certmanager/src/main/ets/model/CheckUserAuthModel.ets:32-117`

---

## 4. 包管理 API

### 4.1 API 清单表

| API 名称 | 导入路径 | 异步 | 说明 |
|---------|----------|------|------|
| `getAppCloneIdentity(appUid)` | `@ohos.bundle.bundleManager` | ✓ | 获取应用克隆标识 |
| `getBundleResourceInfo(bundleName, bundleFlags, appIndex)` | `@ohos.bundle.bundleResourceManager` | ✗ | 获取应用资源信息 |

### 4.2 参数类型

**AppCloneIdentity**:
- `bundleName`: string - Bundle 名称
- `appIndex`: number - 应用索引

**BundleResourceInfo**:
- `label`: string - 应用名称
- `icon`: string - 应用图标

**证据位置**: `certmanager/src/main/ets/model/BundleModel.ets:33-38`

---

## 5. 文件 I/O API

### 5.1 API 清单表

| API 名称 | 导入路径 | 异步 | 说明 |
|---------|----------|------|------|
| `fs.openSync(uri, mode)` | `@ohos.file.fs` | ✗ | 同步打开文件 |
| `fs.statSync(fd)` | `@ohos.file.fs` | ✗ | 同步获取文件状态 |
| `fs.readSync(fd, buf)` | `@ohos.file.fs` | ✗ | 同步读取文件 |
| `fs.closeSync(fd)` | `@ohos.file.fs` | ✗ | 同步关闭文件 |

### 5.2 文件读取流程

```typescript
// 1. 打开文件
let file = fs.openSync(mediaUri, fs.OpenMode.READ_ONLY);

// 2. 获取文件大小
let stat = fs.statSync(file.fd);

// 3. 读取文件内容
let buf = new ArrayBuffer(Number(stat.size));
let num = fs.readSync(file.fd, buf);

// 4. 转换为 Uint8Array
callback(new Uint8Array(buf));

// 5. 关闭文件
fs.closeSync(file.fd);
```

**证据位置**: `certmanager/src/main/ets/model/FileIoModel.ets:21-44`

---

## 6. 窗口管理 API

### 6.1 API 清单表

| API 名称 | 导入路径 | 异步 | 说明 |
|---------|----------|------|------|
| `window.getLastWindow(context)` | `@ohos.window` | ✓ | 获取顶层窗口 |
| `windowClass.setWindowPrivacyMode(isPrivacyMode)` | `@ohos.window` | ✓ | 设置窗口隐私模式 |
| `session.setWindowPrivacyMode(isPrivacyMode)` | `@ohos.app.ability.UIExtensionContentSession` | ✗ | 设置会话窗口隐私模式 |

### 6.2 防截屏实现

```typescript
// 对于 UIExtensionAbility
session.setWindowPrivacyMode(isPrivacyMode);

// 对于普通 Ability
window.getLastWindow(context).then((window) => {
  window.setWindowPrivacyMode(isPrivacyMode);
})
```

**证据位置**: `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:34-57`

---

## 7. 路由 API

### 7.1 API 清单表

| API 名称 | 导入路径 | 异步 | 说明 |
|---------|----------|------|------|
| `router.pushUrl({url, params})` | `@ohos.router` | ✗ | 跳转页面 |

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:84-86`

---

## 8. 文件选择器 API

### 8.1 API 清单表

| API 名称 | 导入路径 | 异步 | 说明 |
|---------|----------|------|------|
| `picker.*` | `@ohos.file.picker` | ✓ | 文件选择器 |

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:16`

---

## 9. 错误码定义

### 9.1 模型层错误码

| 错误码 | 名称 | 说明 |
|--------|------|------|
| 0 | `CM_MODEL_ERROR_SUCCESS` | 成功 |
| -1 | `CM_MODEL_ERROR_FAILED` | 失败 |
| -2 | `CM_MODEL_ERROR_EXCEPTION` | 异常 |
| -3 | `CM_MODEL_ERROR_UNKNOWN_OPT` | 未知操作 |
| -4 | `CM_MODEL_ERROR_NOT_SUPPORT` | 不支持 |
| -5 | `CM_MODEL_ERROR_NOT_FOUND` | 未找到 |
| -6 | `CM_MODEL_ERROR_INCORRECT_FORMAT` | 格式错误 |
| -7 | `CM_MODEL_ERROR_MAX_QUANTITY_REACHED` | 达到最大数量 |
| -8 | `CM_MODEL_ERROR_ALIAS_LENGTH_REACHED_LIMIT` | 别名过长 |
| -9 | `CM_MODEL_ERROR_PASSWORD_ERR` | 密码错误 |
| -10 | `CM_MODEL_ERROR_ADVANCED_SECURITY` | 高级安全 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:26-38`

---

## 10. API 调用汇总表

| 模块 | API 数量 | 主要 API |
|------|---------|---------|
| `@ohos.security.certManager` | 20+ | 证书/凭据管理 |
| `@ohos.userIAM.userAuth` | 4 | 用户认证 |
| `@ohos.bundle.bundleManager` | 1 | 包管理 |
| `@ohos.bundle.bundleResourceManager` | 1 | 包资源 |
| `@ohos.file.fs` | 4 | 文件 I/O |
| `@ohos.window` | 2 | 窗口管理 |
| `@ohos.router` | 1 | 路由 |
| `@ohos.file.picker` | 1+ | 文件选择 |
| `@ohos.hilog` | 2 | 日志 |

---

**END OF 04_External_API.md**
