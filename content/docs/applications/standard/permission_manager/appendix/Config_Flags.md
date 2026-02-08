# 附录：配置标志

> **目的**: 记录PermissionManager中关键常量、枚举和配置项  
> **适用范围**: 开发者、配置人员  
> **最后更新**: 2026-02-05

---

## 常量定义

### 1. 常量文件 (constant.ets)

**文件位置**: `permissionmanager/src/main/ets/common/utils/constant.ets`

| 常量名 | 值 | 用途 |
|--------|-----|------|
| `ACCESS_TOKEN` | `'ohos.permission.ACCESS_TOKEN'` | IPC接口令牌 |
| `RESULT_CODE` | `1` | IPC结果码 - 返回授权结果 |
| `RESULT_CODE_1` | `2` | IPC结果码 - 设置对话框数据 |
| `RESULT_SUCCESS` | `0` | 操作成功 |
| `RESULT_FAILURE` | `-1` | 操作失败 |
| `DYNAMIC_OPER` | `1` | 动态操作标志 |
| `PASS_OPER` | `2` | 通过操作标志 |
| `SETTING_OPER` | `3` | 设置操作标志 |
| `PERMISSION_FLAG` | `0` | 权限标志 |
| `PERMISSION_ALLOW_THIS_TIME` | `1` | 允许本次标志 |
| `CREATE_WINDOW_REPEATED` | `1001` | 窗口重复创建错误码 |
| `ERR_MODAL_ALREADY_EXIST` | `1002` | 对话框已存在错误码 |
| `BG_COLOR` | `'#00000000'` | 背景色（透明） |

---

### 2. 位置权限标志

**文件位置**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:32-36`

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `LOCATION_NONE` | `0` | 无位置权限 |
| `LOCATION_FUZZY` | `1` | 仅模糊位置 |
| `LOCATION_UPGRADE` | `2` | 模糊升级为精确 |
| `LOCATION_BOTH_PRECISE` | `3` | 模糊+精确，开启精确 |
| `LOCATION_BOTH_CLOSE` | `4` | 模糊+精确，关闭精确 |

---

## 枚举定义

### 1. Permission (权限枚举)

**文件位置**: `permissionmanager/src/main/ets/common/model/definition.ets:16-49`

**位置权限 (3个)**:
```typescript
LOCATION_IN_BACKGROUND = 'ohos.permission.LOCATION_IN_BACKGROUND'  // 后台定位
APPROXIMATELY_LOCATION = 'ohos.permission.APPROXIMATELY_LOCATION'  // 模糊位置
LOCATION = 'ohos.permission.LOCATION'                              // 精确位置
```

**媒体权限 (11个)**:
```typescript
CAMERA = 'ohos.permission.CAMERA'                                  // 相机
MICROPHONE = 'ohos.permission.MICROPHONE'                          // 麦克风
READ_IMAGEVIDEO = 'ohos.permission.READ_IMAGEVIDEO'               // 读取图片视频
WRITE_IMAGEVIDEO = 'ohos.permission.WRITE_IMAGEVIDEO'             // 写入图片视频
MEDIA_LOCATION = 'ohos.permission.MEDIA_LOCATION'                  // 媒体位置
READ_AUDIO = 'ohos.permission.READ_AUDIO'                          // 读取音频
WRITE_AUDIO = 'ohos.permission.WRITE_AUDIO'                        // 写入音频
READ_DOCUMENT = 'ohos.permission.READ_DOCUMENT'                    // 读取文档
WRITE_DOCUMENT = 'ohos.permission.WRITE_DOCUMENT'                  // 写入文档
READ_MEDIA = 'ohos.permission.READ_MEDIA'                          // 读取媒体
WRITE_MEDIA = 'ohos.permission.WRITE_MEDIA'                        // 写入媒体
```

**通讯录权限 (2个)**:
```typescript
READ_CONTACTS = 'ohos.permission.READ_CONTACTS'                    // 读取通讯录
WRITE_CONTACTS = 'ohos.permission.WRITE_CONTACTS'                  // 写入通讯录
```

**日历权限 (4个)**:
```typescript
READ_CALENDAR = 'ohos.permission.READ_CALENDAR'                    // 读取日历
WRITE_CALENDAR = 'ohos.permission.WRITE_CALENDAR'                  // 写入日历
READ_WHOLE_CALENDAR = 'ohos.permission.READ_WHOLE_CALENDAR'        // 读取所有日历
WRITE_WHOLE_CALENDAR = 'ohos.permission.WRITE_WHOLE_CALENDAR'      // 写入所有日历
```

**健康和运动 (2个)**:
```typescript
ACTIVITY_MOTION = 'ohos.permission.ACTIVITY_MOTION'                // 运动健身
READ_HEALTH_DATA = 'ohos.permission.READ_HEALTH_DATA'              // 健康数据
```

**其他权限 (12个)**:
```typescript
APP_TRACKING_CONSENT = 'ohos.permission.APP_TRACKING_CONSENT'      // 应用跟踪
GET_INSTALLED_BUNDLE_LIST = 'ohos.permission.GET_INSTALLED_BUNDLE_LIST'  // 获取已安装应用列表
DISTRIBUTED_DATASYNC = 'ohos.permission.DISTRIBUTED_DATASYNC'      // 分布式数据同步
ACCESS_BLUETOOTH = 'ohos.permission.ACCESS_BLUETOOTH'              // 蓝牙
READ_PASTEBOARD = 'ohos.permission.READ_PASTEBOARD'                // 读取剪贴板
READ_WRITE_DOWNLOAD_DIRECTORY = 'ohos.permission.READ_WRITE_DOWNLOAD_DIRECTORY'  // 下载目录
READ_WRITE_DESKTOP_DIRECTORY = 'ohos.permission.READ_WRITE_DESKTOP_DIRECTORY'    // 桌面目录
READ_WRITE_DOCUMENTS_DIRECTORY = 'ohos.permission.READ_WRITE_DOCUMENTS_DIRECTORY' // 文档目录
ACCESS_NEARLINK = 'ohos.permission.ACCESS_NEARLINK'                // 星闪
CUSTOM_SCREEN_CAPTURE = 'ohos.permission.CUSTOM_SCREEN_CAPTURE'    // 屏幕录制
```

**总计**: 48个权限

---

### 2. PermissionGroup (权限组枚举)

**文件位置**: `permissionmanager/src/main/ets/common/model/definition.ets:51-74`

| 枚举值 | 包含权限数 | 策略类 |
|--------|-----------|--------|
| `LOCATION` | 3 | LocationStrategy |
| `CAMERA` | 1 | CameraStrategy |
| `MICROPHONE` | 1 | MicrophoneStrategy |
| `CONTACTS` | 2 | ContactsStrategy |
| `CALENDAR` | 4 | CalendarStrategy |
| `SPORT` | 1 | SportStrategy |
| `HEALTH` | 1 | HealthStrategy |
| `IMAGE_AND_VIDEOS` | 3 | ImageAndVideosStrategy |
| `AUDIOS` | 2 | AudioStrategy |
| `DOCUMENTS` | 4 | DocumentsStrategy |
| `ADS` | 1 | AdsStrategy |
| `GET_INSTALLED_BUNDLE_LIST` | 1 | GetInstalledBundleListStrategy |
| `DISTRIBUTED_DATASYNC` | 1 | DistributedDatasyncStrategy |
| `BLUETOOTH` | 1 | BluetoothStrategy |
| `PASTEBOARD` | 1 | PasteboardStrategy |
| `FOLDER` | 0 | (父组) |
| `DOWNLOAD_DIRECTORY` | 1 | DownloadDirectoryStrategy |
| `DESKTOP_DIRECTORY` | 1 | DesktopDirectoryStrategy |
| `DOCUMENTS_DIRECTORY` | 1 | DocumentsDirectoryStrategy |
| `NEARLINK` | 1 | NearlinkStrategy |
| `CUSTOM_SCREEN_CAPTURE` | 1 | ScreenCaptureStrategy |
| `OTHER` | 0 | 默认策略 |

**总计**: 21个权限组 (+ 1个OTHER)

---

### 3. ButtonStatus (按钮状态)

**文件位置**: `permissionmanager/src/main/ets/common/model/definition.ets:76-83`

```typescript
enum ButtonStatus {
    ALLOW = 'ALLOW',                              // 允许
    DENY = 'DENY',                                // 拒绝
    CANCEL = 'CANCEL',                            // 取消
    THIS_TIME_ONLY = 'THIS_TIME_ONLY',            // 仅本次允许
    ALLOW_THIS_TIME = 'ALLOW_THIS_TIME',          // 本次允许
    ALLOW_ONLY_DURING_USE = 'ALLOW_ONLY_DURING_USE'  // 仅在使用时允许
}
```

**使用场景**:
- 默认: [DENY, ALLOW]
- 位置权限: [DENY, ALLOW_ONLY_DURING_USE, ALLOW]
- 剪贴板权限: [DENY, THIS_TIME_ONLY, ALLOW]

---

### 4. ClickOption (点击选项)

**文件位置**: `permissionmanager/src/main/ets/common/model/definition.ets:85-92`

```typescript
enum ClickOption {
    GRANT = 'GRANT',    // 允许（包括THIS_TIME_ONLY, ALLOW_THIS_TIME等）
    DENY = 'DENY',      // 拒绝
    CANCEL = 'CANCEL'   // 取消
}
```

**映射关系** (PermissionGroupManager.getClickSituation):
- ALLOW, THIS_TIME_ONLY, ALLOW_THIS_TIME, ALLOW_ONLY_DURING_USE → GRANT
- CANCEL → CANCEL
- DENY → DENY (default)

---

### 5. PermissionOption (权限操作选项)

**文件位置**: `permissionmanager/src/main/ets/common/model/definition.ets:94-101`

```typescript
enum PermissionOption {
    GRANT = 'GRANT',    // 授予权限
    REVOKE = 'REVOKE',  // 撤销权限
    SKIP = 'SKIP'       // 跳过（不操作）
}
```

---

### 6. PermissionType (权限结果类型)

**文件位置**: `permissionmanager/src/main/ets/common/model/definition.ets:103-110`

```typescript
enum PermissionType {
    GRANT = 0,      // 允许
    FAILED = 1,     // 弹窗失败
    CANCELED = 2    // 取消
}
```

---

## GN构建配置

### BUILD.gn关键配置

**文件位置**: `BUILD.gn`

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `hap_name` | `"permission_manager"` | HAP包文件名 |
| `part_name` | `"permission_manager"` | 部件名称 |
| `subsystem_name` | `"applications"` | 子系统名称 |
| `module_install_dir` | `"app/com.ohos.permissionmanager"` | 安装路径 |
| `js_build_mode` | `"release"` | 构建模式 |
| `build_level` | `"module"` | 构建级别 |
| `assemble_type` | `"assembleHap"` | 组装类型 |
| `compatible_version` | `"9"` | 兼容版本 |

---

## module.json关键配置

### Ability配置

**MainAbility**:
```json
{
    "name": "com.ohos.permissionmanager.MainAbility",
    "srcEntry": "./ets/MainAbility/MainAbility.ts",
    "launchType": "singleton",
    "exported": true,
    "permissions": ["ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER"]
}
```

**GrantAbility (ServiceExtAbility)**:
```json
{
    "name": "com.ohos.permissionmanager.GrantAbility",
    "srcEntry": "./ets/ServiceExtAbility/ServiceExtAbility.ets",
    "type": "service",
    "exported": true
}
```

**GlobalExtAbility**:
```json
{
    "name": "com.ohos.permissionmanager.GlobalExtAbility",
    "srcEntry": "./ets/GlobalExtAbility/GlobalExtAbility.ets",
    "type": "service",
    "exported": true,
    "permissions": ["ohos.permission.GET_SENSITIVE_PERMISSIONS"]
}
```

**SecurityExtAbility**:
```json
{
    "name": "com.ohos.permissionmanager.SecurityExtAbility",
    "srcEntry": "./ets/SecurityExtAbility/SecurityExtAbility.ets",
    "type": "service",
    "exported": true,
    "permissions": ["ohos.permission.GET_SENSITIVE_PERMISSIONS"]
}
```

**PermissionStateSheetAbility**:
```json
{
    "name": "com.ohos.permissionmanager.PermissionStateSheetAbility",
    "srcEntry": "./ets/PermissionSheet/PermissionStateSheetAbility.ets",
    "type": "sys/commonUI",
    "exported": true
}
```

### 权限声明

**文件位置**: `permissionmanager/src/main/module.json:93-132`

| 权限 | 类型 | 用途 |
|------|------|------|
| `ohos.permission.GET_SENSITIVE_PERMISSIONS` | system_grant | 获取敏感权限信息 |
| `ohos.permission.GRANT_SENSITIVE_PERMISSIONS` | system_grant | 授予敏感权限 |
| `ohos.permission.REVOKE_SENSITIVE_PERMISSIONS` | system_grant | 撤销敏感权限 |
| `ohos.permission.GET_BUNDLE_INFO` | system_grant | 获取应用包信息 |
| `ohos.permission.GET_BUNDLE_RESOURCES` | system_grant | 获取应用资源 |
| `ohos.permission.PERMISSION_USED_STATS` | system_grant | 访问权限使用统计 |
| `ohos.permission.GET_INSTALLED_BUNDLE_LIST` | user_grant | 获取已安装应用列表 |
| `ohos.permission.LISTEN_BUNDLE_CHANGE` | system_grant | 监听应用变更 |
| `ohos.permission.ACCESS_BUNDLE_DIR` | system_grant | 访问应用目录 |
| `ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER` | system_grant | 访问安全隐私中心 |
| `ohos.permission.MICROPHONE_CONTROL` | system_grant | 控制麦克风 |
| `ohos.permission.CAMERA_CONTROL` | system_grant | 控制相机 |

**总计**: 12个权限（11个system_grant + 1个user_grant）

---

## Want参数键值

### ServiceExtAbility接收的参数

**文件位置**: `GrantDialogModel.ets:61-67`

| 键名 | 类型 | 说明 |
|------|------|------|
| `ohos.aafwk.param.callerBundleName` | string | 调用方包名 |
| `ohos.aafwk.param.callerToken` | number | 调用方Token ID |
| `ohos.user.grant.permission` | Permission[] | 请求的权限列表 |
| `ohos.user.grant.permission.state` | number[] | 权限初始状态 |
| `ohos.ability.params.callback` | Property | IPC回调对象 |
| `ohos.ability.params.token` | Property | 窗口绑定Token |

### SecurityExtAbility接收的参数

**文件位置**: `SecurityExtAbility.ets:43-50`

| 键名 | 类型 | 说明 |
|------|------|------|
| `ohos.display.width` | number | 显示宽度 |
| `ohos.display.height` | number | 显示高度 |
| `ohos.caller.uid` | number | 调用方UID |
| `ohos.ability.params.windowId` | number | 窗口ID |
| `ohos.ability.params.token` | Property | 绑定Token |

### GlobalExtAbility接收的参数

**文件位置**: `GlobalExtAbility.ets:36`

| 键名 | 类型 | 说明 |
|------|------|------|
| `ohos.sensitive.resource` | string | 权限资源标识 |

---

## 资源文件配置

### 颜色资源

**文件位置**: `permissionmanager/src/main/resources/base/element/color.json`

| 资源名 | 默认用途 |
|--------|----------|
| `default_background_color` | 默认背景色 |

### 字符串资源

**文件位置**: `permissionmanager/src/main/resources/base/element/string.json`

| 资源名 | 用途 |
|--------|------|
| `permission_manager` | 应用名称 |
| `entry_desc` | 应用描述 |
| `MainAbility_desc` | MainAbility描述 |
| `MainAbility_label` | MainAbility标签 |

### 媒体资源

**文件位置**: `permissionmanager/src/main/resources/base/media/`

| 资源名 | 用途 |
|--------|------|
| `app_icon` | 应用图标 |
| `icon` | Ability图标 |

---

## 快速参考

### 权限字符串 → 枚举 → 策略

```
'ohos.permission.CAMERA'
    ↓
Permission.CAMERA
    ↓
PermissionGroup.CAMERA
    ↓
CameraStrategy
```

### 按钮状态 → 点击选项 → 操作

```
ButtonStatus.ALLOW
    ↓
ClickOption.GRANT
    ↓
PermissionOption.GRANT
    ↓
atManager.grantUserGrantedPermission()
```

---

*返回 [README](README.md) | [调用链](Callgraphs.md)*
