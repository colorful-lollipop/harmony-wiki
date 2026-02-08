# 配置标志和常量

## 目的

本文档列出 FilePicker 应用中的关键宏定义、feature flags 和常量。

## 适用范围

本文档适用于：
- 需要了解配置选项的开发者
- 需要定制行为的维护人员

## 应用级常量

### 包信息

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| BUNDLE_NAME | "com.ohos.filepicker" | `FilePickerUtil.ets:44` | 应用包名 |
| versionCode | 10100300 | `app.json5:5` | 版本号 |
| versionName | "1.1.0.300" | `app.json5:6` | 版本名称 |

**证据**：
- `FilePickerUtil.ets:44`
- `AppScope/app.json5:3-8`

## 构建配置

### SDK 版本

| 配置项 | 值 | 证据 | 说明 |
|--------|-----|-------|------|
| compileSdkVersion | 23 | `build-profile.json5:7` | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | `build-profile.json5:8` | 兼容 SDK 版本 |
| runtimeOS | "OpenHarmony" | `build-profile.json5:9` | 运行时操作系统 |

**证据**：`build-profile.json5:7-9`

### 签名配置

| 配置项 | 值 | 证据 | 说明 |
|--------|-----|-------|------|
| signAlg | "SHA256withECDSA" | `build-profile.json5:21` | 签名算法 |
| keyAlias | "debugKey" | `build-profile.json5:18` | 密钥别名 |
| signingConfig.name | "default" | `build-profile.json5:14` | 签名配置名称 |

**证据**：`build-profile.json5:14-24`

## 运行时常量

### 文件系统常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| FILENAME_REGEXP | `/^[^\\/:*?<>\"|]+$/` | `Constant.ts:32` | 文件名正则表达式 |
| FILENAME_MAX_LENGTH | 225 | `Constant.ts:34` | 文件名最大长度 |
| DESKTOP_FOLDER | "Desktop" | `Constant.ts:43` | Desktop 文件夹名称 |
| DOCUMENTS_FOLDER | "Documents" | `Constant.ts:45` | Documents 文件夹名称 |
| INTERNAL_STORAGE_ROOT_URI | "file://media" | `Constant.ts:50` | 内部存储根 URI |
| DOWNLOAD_PATH | "/storage/Users/currentUser/Download" | `DownloadAuth.ets:31` | 下载目录路径 |
| SANDBOX_APPDATA_PATH | "/data/storage/el2/share/r/docs/storage/Users/currentUser/appdata" | `Constant.ts:93` | 沙箱应用数据路径 |
| RECENT_DB_ROOT_PATH | "/data/storage/el2/share/r/docs/storage/Users/currentUser/.Recent" | `Constant.ts:95-96` | 最近文件数据库路径 |
| TRASH_DB_ROOT_PATH | "/data/storage/el2/share/r/docs/storage/Users/currentUser/.Trash" | `Constant.ts:97` | 回收站数据库路径 |
| DESKTOP_ROOT_PATH | "/data/storage/el2/share/r/docs/storage/Users/currentUser/DeskTop" | `Constant.ts:98` | 桌面根路径 |

**证据**：
- `Constant.ts:32-100`
- `DownloadAuth.ets:31`

### 选择模式常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| SELECT_MODE.FILE | 0 | `Constant.ts:55-59` | 文件选择模式 |
| SELECT_MODE.FOLDER | 1 | `Constant.ts:56-59` | 文件夹选择模式 |
| SELECT_MODE.MIX | 2 | `Constant.ts:57-59` | 混合选择模式 |

**证据**：`Constant.ts:55-59`

### 文件后缀常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| FILE_SUFFIX.SUFFIX_SPLIT | "," | `Constant.ts:65` | 后缀分隔符 |
| FILE_SUFFIX.SUFFIX_START | "." | `Constant.ts:66` | 后缀起始符 |

**证据**：`Constant.ts:64-67`

### 目录层级常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| FOLDER_LEVEL.MIN_LEVEL | 1 | `Constant.ts:72` | 目录最小层级 |
| FOLDER_LEVEL.MAX_LEVEL | 21 | `Constant.ts:74` | 目录最大层级 |

**证据**：`Constant.ts:72-75`

### 页面类型常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| PAGE_TYPE.MY_PHONE | "myPhone" | `Constant.ts:80-82` | 我的手机页面类型 |

**证据**：`Constant.ts:80-82`

### 首选项常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| FILE_MANAGER_PREFERENCES.name | "FileManagerPreferences" | `Constant.ts:84-86` | 文件管理器首选项名称 |
| FILE_MANAGER_PREFERENCES.lastSelectPath.key | "lastSelectPath" | `Constant.ts:87-88` | 上次选择路径键 |
| FILE_MANAGER_PREFERENCES.lastSelectPath.defaultValue | "" | `Constant.ts:89` | 默认值为空 |

**证据**：`Constant.ts:84-90`

## 错误码常量

### 应用层错误码

| 错误码 | 名称 | 值 | 证据 | 说明 |
|--------|------|-----|-------|------|
| PICKER.NORMAL | 1000 | `ErrorCodeConst.ts:70` | 正常 |
| PICKER.FILE_NAME_EXIST | 1001 | `ErrorCodeConst.ts:54` | 文件名已存在 |
| PICKER.FILE_NAME_INVALID | 1002 | `ErrorCodeConst.ts:58` | 文件名非法 |
| PICKER.GRANT_URI_PERMISSION_FAIL | 2001 | `ErrorCodeConst.ts:62` | URI 授权失败 |
| PICKER.OTHER_ERROR | 9001 | `ErrorCodeConst.ts:66` | 其他错误 |

**证据**：`ErrorCodeConst.ts:50-71`

### 系统层错误码

| 错误码 | 名称 | 值 | 证据 | 说明 |
|--------|------|-----|-------|------|
| FILE_ACCESS.FILE_NAME_EXIST | 13900015 | `ErrorCodeConst.ts:32` | 文件名已存在 |
| FILE_ACCESS.FILE_NAME_INVALID | 14000001 | `ErrorCodeConst.ts:36` | 文件名非法 |
| FILE_ACCESS.IPC_ERROR | -102825984 | `ErrorCodeConst.ts:40` | IPC 异常 |
| FILE_ACCESS.GET_MEDIAFILE_NULL | "3" | `ErrorCodeConst.ts:44` | 媒体库查询为空 |

**证据**：`ErrorCodeConst.ts:28-45`

### 特殊错误码

| 错误码 | 名称 | 值 | 证据 | 说明 |
|--------|------|-----|-------|------|
| ACCOUNT_NOT_LOGIN | 2002 | `ErrorCodeConst.ts:23` | 账号未登录 |

**证据**：`ErrorCodeConst.ts:22-23`

## 权限标志常量

### Want 标志

| 标志 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| FLAG_AUTH_READ_URI_PERMISSION | 0x01 | `FilePickerUtil.ets:135` | URI 读权限 |
| FLAG_AUTH_WRITE_URI_PERMISSION | 0x02 | `FilePickerUtil.ets:136` | URI 写权限 |
| FLAG_AUTH_PERSISTABLE_URI_PERMISSION | 0x04 | `FilePickerUtil.ets:137` | URI 持久化权限 |

**证据**：`FilePickerUtil.ets:135-137`

### Ability 窗口模式

| 模式 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| windowMode 102 | { windowMode: 102 } | `README.md:32` | 窗口模式 102（弹窗模式） |

**证据**：`README.md:32`

## 任务池常量

### 任务池配置

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| maxRunningTask | 5 | `TaskManager.ts:36` | 最大并发任务数 |

**证据**：`TaskManager.ts:36`

### 任务池名称

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| TaskpoolName.SINGLE_DEFAULT | "SINGLE_DEFAULT" | `TaskManager.ts:83` | 默认任务池名称 |
| TaskpoolName.BATCH_AUTH_URI_PERMISSION_TASK | "BATCH_AUTH_URI_PERMISSION_TASK" | `BatchGrantPermissionTask.ets:23` | 批量授权任务池名称 |

**证据**：
- `TaskManager.ts:83`
- `BatchGrantPermissionTask.ets:23`

### 任务执行模式

| 模式 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| TaskConst.taskExecuteMode.reject | 0 | `TaskManager.ts:97` | 拒绝模式（任务队列满） |
| TaskConst.taskExecuteMode.waitToExecute | 1 | `TaskManager.ts:100` | 等待执行模式（排队） |
| TaskConst.taskExecuteMode.executeNow | 2 | `TaskManager.ts:103` | 立即执行模式 |

**证据**：`TaskManager.ts:96-109`

### 任务状态

| 状态 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| TaskStatus.END | 0 | `BatchGrantPermissionTask.ets:32` | 任务结束 |

**证据**：`BatchGrantPermissionTask.ets:17, 32`

## UI 常量

### UI 颜色

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| SystemBarColor.WHITE | "#FFFFFF" | `MyPhone.ets:162` | 白色系统栏 |
| SystemBarColor.BLACK | "#000000" | `MyPhone.ets:163` | 黑色系统栏 |
| SystemBarColor.LIGHT_GRAY | "#F2F2F2" | `PathPicker.ets:44` | 浅灰色背景 |

**证据**：
- `MyPhone.ets:162-163`
- `PathPicker.ets:44`

### UI 常量

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| TRANSPARENT_COLOR | "#00000000" | `FilePickerUIExtAbility.ets:28` | 透明颜色 |
| NORMAL_PICKER_SELECT_NUM | 50 | `FilePickerUtil.ets:207` | 正常选择器选择数量阈值 |
| FOLDER_LEVEL.MAX_LEVEL | 21 | `Constant.ts:74` | 文件夹最大层级 |

**证据**：
- `FilePickerUIExtAbility.ets:28`
- `FilePickerUtil.ets:207`
- `Constant.ts:74`

## 日志常量

### 日志级别

| 级别 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| LogVersion.Debug | 1 | `Logger.ts:28` | 调试级别 |
| LogVersion.Info | 2 | `Logger.ts:29` | 信息级别 |
| LogVersion.Warn | 3 | `Logger.ts:30` | 警告级别 |
| LogVersion.Error | 4 | `Logger.ts:31` | 错误级别 |
| LogVersion.Fatal | 5 | `Logger.ts:32` | 致命级别 |

**证据**：`Logger.ts:27-33`

### 日志配置

| 常量 | 值 | 证据 | 说明 |
|------|-----|-------|------|
| APP_TAG | "FilePicker" | `Logger.ts:43` | 应用标签 |
| LOG_DOMAIN | 0 | `Logger.ts:48` | 日志域 |
| LOG_LEVEL | LogVersion.Debug | `Logger.ts:38` | 当前日志级别（开发环境） |

**证据**：`Logger.ts:38-48`

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 查看常量定义位置
- [安全评审](07_Security_Review.md) - 了解常量相关的安全考虑
