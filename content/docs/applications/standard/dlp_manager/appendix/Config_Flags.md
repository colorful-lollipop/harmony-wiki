# 附录B: 关键配置与常量

## 目的

本文档汇总DLP Manager的关键常量、配置项和Feature Flags，便于开发者查阅。

## 文件位置

主要常量定义文件: `entry/src/main/ets/common/constant.ets`

---

## 1. 错误码定义

### 1.1 应用层错误码 (0-200)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| ERR_JS_APP_ACCOUNT_INFO | 0 | 账号信息正常 |
| ERR_JS_APP_INSIDE_ERROR | 1 | 应用内部错误 |
| ERR_JS_GET_ACCOUNT_ERROR | 2 | 获取账号失败 |
| ERR_JS_APP_NO_ACCOUNT_ERROR | 3 | 无账号错误 |
| ERR_JS_APP_PARAM_ERROR | 4 | 参数错误 |
| ERR_JS_APP_GET_FILE_ASSET_ERROR | 5 | 获取文件资源失败 |
| ERR_JS_APP_OPEN_REJECTED | 6 | 打开被拒绝 |
| ERR_JS_APP_ENCRYPTION_REJECTED | 7 | 加密被拒绝 |
| ERR_JS_APP_SYSTEM_IS_AUTHENTICATED | 8 | 系统已认证 |
| ERR_JS_APP_NETWORK_INVALID | 9 | 网络无效 |
| ERR_JS_APP_ENCRYPTING | 10 | 正在加密中 |
| ERR_JS_APP_CANNOT_OPEN | 11 | 无法打开 |
| ERR_JS_RELEASE_FILE_OPEN | 13 | 释放文件打开 |
| ERR_JS_APP_PERMISSION_DENY | 201 | 权限拒绝 |

**代码位置**: `entry/src/main/ets/common/constant.ets:134-147`

### 1.2 DLP服务错误码 (19100000+)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| ERR_JS_CREDENTIAL_TIMEOUT | 19100003 | 凭证超时 |
| ERR_JS_CREDENTIAL_SERVICE_ERROR | 19100004 | 凭证服务错误 |
| ERR_JS_CREDENTIAL_SERVER_ERROR | 19100005 | 凭证服务器错误 |
| ERR_JS_NOT_DLP_FILE | 19100008 | 不是DLP文件 |
| ERR_JS_DLP_FILE_READ_ONLY | 19100010 | DLP文件只读 |
| ERR_JS_USER_NO_PERMISSION | 19100013 | 用户无权限 |
| ERR_JS_ACCOUNT_NOT_LOGIN | 19100014 | 账号未登录 |
| ERR_JS_SYSTEM_NEED_TO_BE_UPGRADED | 19100015 | 系统需要升级 |
| ERR_JS_NOT_AUTHORIZED_APPLICATION | 19100018 | 未授权应用 |
| ERR_JS_FILE_EXPIRATION | 19100019 | 文件已过期 |
| ERR_JS_OFFLINE | 19100020 | 离线状态 |
| ERR_JS_NO_SPACE | 19100021 | 空间不足 |
| ERR_JS_DLP_ALLOWED_OPEN_COUNT_INVALID | 19100022 | 打开次数无效 |

**代码位置**: `entry/src/main/ets/common/constant.ets:148-160`

### 1.3 内部处理错误码 (-1 ~ -15)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| ERR_CODE_SUCCESS | 0 | 成功 |
| ERR_CODE_OPEN_FILE_ERROR | -1 | 打开文件错误 |
| ERR_CODE_PARAMS_CHECK_ERROR | -2 | 参数检查错误 |
| ERR_CODE_CREATE_DECRYPT_TASK_ERROR | -3 | 创建解密任务错误 |
| ERR_CODE_NETWORK_ERROR | -4 | 网络错误 |
| ERR_CODE_PARSE_DLP_FILE_ERROR | -5 | 解析DLP文件错误 |
| ERR_CODE_START_ABILITY_ERROR | -6 | 启动Ability错误 |
| ERR_CODE_GET_BUNDLE_INFO_ERROR | -7 | 获取包信息错误 |
| ERR_CODE_GET_LOCK_ASYNC_ERROR | -8 | 获取异步锁错误 |
| ERR_CODE_FILE_IS_DECRYPTING_ERROR | -9 | 文件正在解密错误 |
| ERR_CODE_USER_STOP_DIALOG | -10 | 用户停止对话框 |
| ERR_CODE_DECRYPT_TIME_OUT | -11 | 解密超时 |
| ERR_JS_USER_NO_PERMISSION_2B | -12 | 用户无权限(2B) |
| ERR_JS_USER_NO_PERMISSION_2C | -13 | 用户无权限(2C) |
| ERR_JS_DOMAIN_NO_ACCOUNT | -14 | 域无账号 |
| ERR_JS_INSTALL_SANDBOX_ERROR | -15 | 安装沙箱错误 |

**代码位置**: `entry/src/main/ets/common/constant.ets:278-294`

---

## 2. Bundle和Ability名称

### 2.1 Bundle名称

| 常量 | 值 | 说明 |
|------|-----|------|
| DLP_MANAGER_BUNDLE_NAME | 'com.ohos.dlpmanager' | DLP Manager本应用 |
| DLP_CREDMGR_BUNDLE_NAME | 'com.huawei.hmos.dlpcredmgr' | DLP凭证管理服务 |
| SHARE_BUNDLE | 'com.huawei.hmos.instantshare' | 分享应用 |

### 2.2 Ability名称

| 常量 | 值 | 说明 |
|------|-----|------|
| DLP_CREDMGR_DATA_ABILITY_NAME | 'DlpCredDataExtAbility' | 凭证数据Ability |
| DLP_CREDMGR_ACCOUNT_ABILITY_NAME | 'DlpCredAccountAbility' | 凭证账号Ability |

**代码位置**: `entry/src/main/ets/common/constant.ets:110-113`, `entry/src/main/ets/common/constant.ets:368`

---

## 3. RPC接口令牌

| 常量 | 值 | 用途 |
|------|-----|------|
| SA_INTERFACE_TOKEN | 'OHOS.Security.DlpCredentialAbility' | DLP凭证服务接口 |
| DLP_MGR_INTERFACE_TOKEN | 'OHOS.HapDlpMgrAbilityServiceStub' | DLP Manager接口 |
| DLP_PERMISSION_CLEAN_STUB | 'OHOS.HapDlpPermissionAbilityServiceStub' | DLP权限服务Stub |
| DLP_MGR_VIEW_ABILITY_TOKEN | 'OHOS.HapDlpMgrViewAbilityStub' | ViewAbility接口 |

**代码位置**: `entry/src/main/ets/common/constant.ets:114-119`

---

## 4. 命令码

### 4.1 RPC命令码

| 常量 | 值 | 功能 |
|------|-----|------|
| COMMAND_SEARCH_USER_INFO | 1 | 搜索用户信息 |
| COMMAND_GET_ACCOUNT_INFO | 2 | 获取账号信息 |
| COMMAND_GET_DOMAIN_ACCOUNT_INFO | 3 | 获取域账号信息 |
| COMMAND_BATCH_REFRESH | 4 | 批量刷新 |
| COMMAND_USER_CANCEL_DECRYPT | 5 | 用户取消解密 |

**代码位置**: `entry/src/main/ets/common/constant.ets:295-299`

### 4.2 清理缓存命令码

| 常量 | 值 | 功能 |
|------|-----|------|
| COMMAND_CLEAR_DLP_CACHE | 1 | 清理DLP缓存 |

**代码位置**: `entry/src/main/ets/common/constant.ets:301`

---

## 5. 文件格式常量

### 5.1 Magic数字

| 常量 | 值 | 说明 |
|------|-----|------|
| DLP_ZIP_MAGIC | 0x04034b50 | ZIP格式DLP文件Magic |
| DLP_RAW_MAGIC | 0x087f4922 | 原始格式DLP文件Magic |

### 5.2 文件头长度

| 常量 | 值 | 说明 |
|------|-----|------|
| HEAD_LENGTH_IN_BYTE | 80 | 文件头字节长度 |
| HEAD_LENGTH_IN_U32 | 20 | 文件头u32数组长度 |
| CERT_OFFSET | 16 | 证书偏移量索引 |
| CERT_SIZE | 6 | 证书大小索引 |

### 5.3 路径常量

| 常量 | 值 | 说明 |
|------|-----|------|
| FUSE_PATH | '/mnt/data/fuse/' | FUSE文件系统路径（禁止访问） |
| VALID_URI_PREFIX | 'file://' | 合法URI前缀 |
| DLP_FILE_SUFFIX | '.dlp' | DLP文件后缀 |

**代码位置**: `entry/src/main/ets/common/constant.ets:190-199`, `entry/src/main/ets/common/constant.ets:265`, `entry/src/main/ets/common/constant.ets:387`

---

## 6. 加密相关常量

### 6.1 HUKs配置

| 常量 | 值 | 说明 |
|------|-----|------|
| AES_NONCE_LENGTH | 12 | AES Nonce长度 |
| AE_TAG_SLICE_LENGTH | 16 | AE Tag切片长度 |
| MAX_DATA_LEN | 50000 | 最大数据长度 |

### 6.2 账号关联

| 常量 | 值 | 说明 |
|------|-----|------|
| ASSOCIATION_AAD | 'AssociationAAD' | 关联AAD |
| ASSOCIATION_KEY_ALIAS | 'ASSOCIATION_KEY_ALIAS' | 关联密钥别名 |
| ASSOCIATION_FILE_PREFIX | 'DlpAuthorizedAccounts_' | 关联文件前缀 |
| ASSOCIATION_MAX_SIZE | 20 | 最大关联账号数 |

**代码位置**: `entry/src/main/ets/common/constant.ets:318-326`

---

## 7. 超时时间配置

| 常量 | 值(毫秒) | 说明 |
|------|----------|------|
| DECRYPT_TIMEOUT_TIME | 60000 | 解密超时时间(60秒) |
| INSTALL_TIMEOUT_TIME | 5000 | 安装超时时间(5秒) |
| OPENING_DIALOG_TIMEOUT_TIME | 100000 | 打开对话框超时(100秒) |
| CLEAN_DLP_FILE_IN_CACHE_TIMEOUT | 3600000 | 清理缓存超时(1小时) |
| CONNECTION_TIMEOUT | 3000 | 连接超时(3秒) |
| SHARE_SET_TIMEOUT | 1500 | 分享提示超时(1.5秒) |

**代码位置**: `entry/src/main/ets/common/constant.ets:372-376`, `entry/src/main/ets/common/constant.ets:258`, `entry/src/main/ets/common/constant.ets:254`, `entry/src/main/ets/common/constant.ets:390`

---

## 8. Want参数Key

| 常量 | 值 | 用途 |
|------|-----|------|
| PARAMS_BUNDLE_NAME | 'ohos.dlp.params.bundleName' | Bundle名称参数 |
| PARAMS_CALLER_BUNDLE_NAME | 'ohos.aafwk.param.callerBundleName' | 调用者Bundle名 |
| PARAMS_STREAM | 'ability.params.stream' | 流参数 |
| PARAMS_CALLER_TOKEN | 'ohos.aafwk.param.callerToken' | 调用者Token |
| PARAMS_CALLER_APP_IDENTIFIER | 'ohos.aafwk.param.callerAppIdentifier' | 调用者App标识 |
| PARAMS_MODULE_NAME | 'ohos.dlp.params.moduleName' | 模块名参数 |
| PARAMS_ABILITY_NAME | 'ohos.dlp.params.abilityName' | Ability名参数 |

**代码位置**: `entry/src/main/ets/common/constant.ets:353-360`

---

## 9. 事件名称

| 常量 | 值 | 用途 |
|------|-----|------|
| SHOW_DIALOG_EVENT | 'SHOW_DIALOG_EVENT' | 显示对话框事件 |
| SCREEN_OFF_EVENT | 'SCREEN_OFF_EVENT' | 屏幕关闭事件 |
| CHECK_SHOW_DIALOG_STATE | 'CHECK_SHOW_DIALOG_STATE' | 检查对话框状态 |

**代码位置**: `entry/src/main/ets/common/constant.ets:363-365`

---

## 10. 大小限制

| 常量 | 值 | 说明 |
|------|-----|------|
| SHARE_MAX_SUPPORT_NUMBER | 9 | 最大支持分享文件数 |
| SHARE_MAX_SUPPORT_SIZE_IMAGE_VIDEO_MB | 4086 | 图片视频最大大小(MB) |
| SHARE_MAX_SUPPORT_SIZE_DOC_MB | 300 | 文档最大大小(MB) |
| DLP_FILE_LENGTH_LIMIT | 255 | DLP文件名长度限制 |
| ENCRYPTION_ADD_STAFF_LENGTH_MAX | 50 | 添加人员最大长度 |

**代码位置**: `entry/src/main/ets/common/constant.ets:259-264`, `entry/src/main/ets/common/constant.ets:96`

---

## 11. UI尺寸常量

| 常量 | 值 | 说明 |
|------|-----|------|
| FOOTER_ROW_WIDTH | '100%' | 底部行宽度 |
| HEADER_COLUMN_HEIGHT | 56 | 头部列高度 |
| ENCRYPTION_SUCCESS_CIRCLE | 64 | 成功图标圆圈大小 |
| ENCRYPTION_LOADING_HEIGHT | 340 | 加密加载框高度 |
| TRANSPARENT_BACKGROUND_COLOR | '#00000000' | 透明背景色 |

**代码位置**: `entry/src/main/ets/common/constant.ets:17-125`

---

## 12. Feature Flags

当前DLP Manager没有显式的Feature Flags配置，但以下功能可通过配置控制：

| 功能 | 控制方式 | 说明 |
|------|----------|------|
| ArkTSPartialUpdate | module.json metadata | ArkTS局部更新 |
| 签名配置 | BUILD.gn条件编译 | 开发/正式签名切换 |
| 调试日志 | 日志级别设置 | 控制日志输出详细程度 |

---

## 13. 系统参数

### 13.1 开发模式检测

```typescript
function isDevelopmentMode(): boolean {
  return systemParameterEnhance.getSync('const.security.developermode.state') === 'true';
}
```

**代码位置**: `entry/src/main/ets/common/FileUtils/utils.ets:909-916`

---

## 常量使用示例

### 检查错误码

```typescript
import Constants from '../common/constant';

if (errcode === Constants.ERR_CODE_SUCCESS) {
  // 处理成功
} else if (errcode === Constants.ERR_JS_USER_NO_PERMISSION) {
  // 处理无权限
}
```

### 验证文件路径

```typescript
import Constants from '../common/constant';

if (!uri.startsWith(Constants.VALID_URI_PREFIX)) {
  // 非法URI
}

if (uri.indexOf(Constants.FUSE_PATH) !== -1) {
  // 禁止访问FUSE路径
}
```

### 检查文件类型

```typescript
import Constants from '../common/constant';

if (buf[0] === Constants.DLP_ZIP_MAGIC) {
  // ZIP格式DLP文件
} else if (buf[0] === Constants.DLP_RAW_MAGIC) {
  // 原始格式DLP文件
}
```

---

*本文档为常量速查，详细定义请参考源码*
