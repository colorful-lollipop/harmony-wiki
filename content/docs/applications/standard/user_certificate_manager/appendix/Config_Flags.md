# 关键配置标志

> 本文档列出用户证书管理部件的关键宏、常量和配置标志

---

## 目的

本文档帮助开发者理解项目中的关键配置项和常量定义。

## 适用范围

- 需要修改配置的开发者
- 需要了解常量的技术人员
- 代码审查人员

## 相关跳转

- [02_Directory_Structure.md](wiki/02_Directory_Structure.md) - 目录结构
- [05_Internal_API.md](wiki/05_Internal_API.md) - 内部接口

---

## 1. 操作类型枚举

### 1.1 CMModelOptType

**位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:40-48`

```typescript
export enum CMModelOptType {
  CM_MODEL_OPT_UNKNOWN = 0,       // 未知操作
  CM_MODEL_OPT_SYSTEM_CA = 1,       // 系统 CA 证书
  CM_MODEL_OPT_USER_CA = 2,         // 用户 CA 证书
  CM_MODEL_OPT_APP_CRED = 3,        // 应用凭据
  CM_MODEL_OPT_PRIVATE_CRED = 4,     // 私有凭据
  CM_MODEL_OPT_SYSTEM_CRED = 5,      // 系统 凭据
  CM_MODEL_OPT_USER_CA_P7B = 6,      // 用户 CA 证书 (P7B 格式)
}
```

**用途**: 区分不同类型的证书和凭据操作

---

## 2. 存储类型枚举

### 2.1 CertManagerStore

**位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:68-77`

```typescript
export enum CertManagerStore {
  /* credential certificate store for end entity certificates. */
  CERT_MANAGER_CREDENTIAL_STORE = 0,
  /* read only, updated by system only. */
  CERT_MANAGER_SYSTEM_TRUSTED_STORE = 1,
  /* modifiable by applications and user. */
  CERT_MANAGER_USER_TRUSTED_STORE = 2,
  /* application specific trusted certificate store; modifiable by the application only. */
  CERT_MANAGER_APPLICATION_TRUSTED_STORE = 3,
}
```

**用途**: 区分不同类型的证书存储

---

## 3. 错误码枚举

### 3.1 CMModelErrorCode

**位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:26-38`

```typescript
export enum CMModelErrorCode {
  CM_MODEL_ERROR_SUCCESS = 0,                      // 成功
  CM_MODEL_ERROR_FAILED = -1,                     // 失败
  CM_MODEL_ERROR_EXCEPTION = -2,                   // 异常
  CM_MODEL_ERROR_UNKNOWN_OPT = -3,                  // 未知操作
  CM_MODEL_ERROR_NOT_SUPPORT = -4,                   // 不支持
  CM_MODEL_ERROR_NOT_FOUND = -5,                     // 未找到
  CM_MODEL_ERROR_INCORRECT_FORMAT = -6,              // 格式错误
  CM_MODEL_ERROR_MAX_QUANTITY_REACHED = -7,          // 达到最大数量
  CM_MODEL_ERROR_ALIAS_LENGTH_REACHED_LIMIT = -8,      // 别名过长
  CM_MODEL_ERROR_PASSWORD_ERR = -9,                   // 密码错误
  CM_MODEL_ERROR_ADVANCED_SECURITY = -10,              // 高级安全
}
```

**用途**: 模型层错误码，用于错误处理和用户提示

### 3.2 DialogErrorCode

**位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:50-60`

```typescript
export enum DialogErrorCode {
  DIALOG_ERROR_INTERNAL = -1000,                 // 内部错误
  DIALOG_OPERATION_CANCELS = -1001,             // 操作取消
  DIALOG_ERROR_NOT_ENTERPRISE_DEVICE = -1003,      // 非企业设备
  DIALOG_ERROR_INCORRECT_FORMAT = -1005,          // 格式错误
  DIALOG_ERROR_MAX_QUANTITY_REACHED = -1006,      // 达到最大数量
  DIALOG_ERROR_SA_INTERNAL_ERROR = -1007,          // SA 内部错误
  DIALOG_ERROR_PARAM_INVALID = -1010,              // 参数无效
  DIALOG_ERROR_CAPACITY_NOT_SUPPORT = -1012,       // 容量不支持
  DIALOG_ERROR_NO_CERTS_AVAILIBLE = -1013,       // 无可用证书
}
```

**用途**: 对话框错误码，用于 UI 提示

### 3.3 CmDialogOperationType

**位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:62-66`

```typescript
export enum CmDialogOperationType {
  INSTALL = 1,     // 安装
  UNINSTALL = 2,   // 卸载
  SHOWDETAIL = 3,   // 显示详情
}
```

**用途**: 对话框操作类型

---

## 4. 常量定义

### 4.1 日志常量

**位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:79-84`

```typescript
const TAG = 'CertMangerModel';
const DOMAIN = 0x0000;
```

**位置**: `certmanager/src/main/ets/model/CheckUserAuthModel.ets:22-23`

```typescript
const RANDOM_DATA_LENGTH = 16;
const DOMAIN = 0x0000;
const TAG = 'CheckUserAuthModel';
```

**位置**: `certmanager/src/main/ets/model/BundleModel.ets:22`

```typescript
const TAG = 'certManager BUNDLE:';
```

**位置**: `certmanager/src/main/ets/model/FileIoModel.ets:22`

```typescript
const TAG = 'CertManager FA';
```

**位置**: `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:22`

```typescript
const TAG: string = 'PreventScreenshotsModel';
```

**位置**: `certmanager/src/main/ets/MainAbility/CertPickerUiExtAbility.ets:25`

```typescript
const TAG = 'CertPickerUiExtAbility';
```

**位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:27`

```typescript
const TAG = 'CMFaPresenter: ';
```

---

### 4.2 UI 常量

**位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:28`

```typescript
const gridCountNum: number = 4;
```

**用途**: 对话框网格数量

---

### 4.3 页面类型常量

**位置**: `certmanager/src/main/ets/MainAbility/CertPickerUiExtAbility.ets:22-24`

```typescript
const PAGE_CA_INSTALL = 5;
const PAGE_REQUEST_AUTHORIZE = 6;
const PAGE_UKEY_AUTH = 7;
```

**用途**: 区分不同的页面类型，用于 Extension Ability 的页面加载逻辑

---

## 5. 文件格式常量

### 5.1 CA 证书支持的格式

**位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:68-69`

```typescript
if ((suffix === 'cer') || (suffix === 'pem') || (suffix === 'crt') ||
    (suffix === 'der') || (suffix === 'p7b') || (suffix === 'spc')) {
  // 支持的格式
}
```

**支持的格式**:
- `.cer` - 证书文件
- `.pem` - PEM 编码证书
- `.crt` - 证书文件
- `.der` - DER 编码证书
- `.p7b` - PKCS#7 证书链
- `.spc` - 软件发布证书

---

### 5.2 凭据支持的格式

**位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:82`

```typescript
if ((suffix === 'p12') || (suffix === 'pfx')) {
  // 支持的格式
}
```

**支持的格式**:
- `.p12` - PKCS#12 凭据
- `.pfx` - PKCS#12 凭据

---

## 6. 构建配置

### 6.1 SDK 版本

**位置**: `build-profile.json5:21-22`

```json5
{
  "compileSdkVersion": 23,
  "compatibleSdkVersion": 23
}
```

**说明**:
- `compileSdkVersion`: 编译使用的 SDK 版本
- `compatibleSdkVersion`: 兼容的最低 SDK 版本

---

### 6.2 产品类型

**位置**: `bundle.json:19-21`

```json
{
  "adapted_system_type": ["standard"]
}
```

**说明**: 支持标准系统类型

---

### 6.3 资源占用

**位置**: `bundle.json:23-24`

```json
{
  "rom": "1MB",
  "ram": "1MB"
}
```

**说明**:
- `rom`: ROM 占用
- `ram`: RAM 占用

---

## 7. 运行时配置

### 7.1 系统能力

**位置**: `certmanager/src/main/ets/common/constants/CertManagerConstants.ets:16`

```typescript
export const SYSCAP_HUKS_CRYPTO_EXTENTION: string = 'SystemCapability.Security.Huks.CryptoExtension';
```

**说明**: 定义系统能力

---

### 7.2 Ability 配置

**位置**: `certmanager/src/main/module.json:10-19`

```json
{
  "deviceTypes": ["default"],
  "deliveryWithInstall": true,
  "installationFree": false
}
```

**说明**:
- `deviceTypes`: 支持的设备类型
- `deliveryWithInstall`: 随应用安装
- `installationFree`: 是否支持免安装

---

### 7.3 Ability 启动类型

**位置**: `certmanager/src/main/module.json:31`

```json
{
  "launchType": "singleton"
}
```

**说明**: 单例模式，同一时间只有一个实例

---

### 7.4 窗口方向

**位置**: `certmanager/src/main/module.json:34`

```json
{
  "orientation": "auto_rotation_restricted"
}
```

**说明**: 窗口方向设置

---

## 8. 权限配置

### 8.1 权限列表

**位置**: `certmanager/src/main/module.json:84-115`

| 权限 | 权限 ID |
|--------|----------|
| 获取应用信息 | `ohos.permission.GET_BUNDLE_INFO` |
| 访问证书管理服务内部接口 | `ohos.permission.ACCESS_CERT_MANAGER_INTERNAL` |
| 访问证书管理服务 | `ohos.permission.ACCESS_CERT_MANAGER` |
| 获取应用资源 | `ohos.permission.GET_BUNDLE_RESOURCES` |
| 访问安全隐私中心 | `ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER` |
| 生物识别 | `ohos.permission.ACCESS_BIOMETRIC` |
| 隐私窗口 | `ohos.permission.PRIVACY_WINDOW` |
| 访问系统应用证书 | `ohos.permission.ACCESS_SYSTEM_APP_CERT` |
| 访问用户信任证书 | `ohos.permission.ACCESS_USER_TRUSTED_CERT` |
| 获取应用信息 (特权) | `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` |

---

## 9. 字符串资源键

### 9.1 应用名称

**位置**: `AppScope/app.json:3-8`

```json
{
  "label": "$string:app_name"
}
```

### 9.2 Ability 名称

**位置**: `certmanager/src/main/module.json:29`

```json
{
  "label": "$string:entry_MainAbility"
}
```

---

## 10. 编译标志

### 10.1 ETS 转 ABC

**位置**: `BUILD.gn:40`

```gn
ohos_js_assets("cert_manager_js_assets") {
  ets2abc = true,
  source_dir = "certmanager/src/main/ets"
}
```

**说明**: 启用 ETS 到 ABC 字节码的转换

### 10.2 构建模式

**位置**: `BUILD.gn:25`

```gn
js_build_mode = "release"
```

**说明**: JS 构建模式为 release

---

## 11. 扩展配置

### 11.1 ArkTS 部分更新

**位置**: `certmanager/src/main/module.json:10-18`

```json
{
  "metadata": [
    {
      "name": "ArkTSPartialUpdate",
      "value": "true"
    },
    {
      "name": "partialUpdateStrictCheck",
      "value": "all"
    }
  ]
}
```

**说明**:
- `ArkTSPartialUpdate`: 启用 ArkTS 部分更新
- `partialUpdateStrictCheck`: 严格检查模式

---

### 11.2 隐私中心配置

**位置**: `certmanager/src/main/module.json:69-74`

```json
{
  "metadata": [
    {
      "name": "metadata.access.privacy.center",
      "value": "security_privacy.json"
    }
  ]
}
```

**说明**: 隐私中心元数据配置

---

**END OF appendix/Config_Flags.md**
