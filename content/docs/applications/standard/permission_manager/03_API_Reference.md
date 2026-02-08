# 对外API参考

> **目的**: 说明PermissionManager使用的系统API和对外暴露的接口  
> **适用范围**: 应用开发者、系统集成人员  
> **最后更新**: 2026-02-05

---

## 重要说明

**PermissionManager是纯应用层项目**，使用ArkTS语言开发，**没有提供N-API接口**。本章节说明：

1. PermissionManager对外暴露的Ability（供系统调用）
2. PermissionManager使用的系统API
3. 应用如何触发PermissionManager功能

---

## 对外暴露的Ability

PermissionManager通过 `module.json` 注册了6个Ability，供系统或其他应用调用。

### 1. MainAbility - 权限管理设置主界面

| 属性 | 值 |
|------|-----|
| **注册名** | `com.ohos.permissionmanager.MainAbility` |
| **类型** | UIAbility |
| **启动模式** | singleton |
| **导出** | true |
| **所需权限** | `ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER` |

**调用方式**:

```typescript
// 通过Action调用
import Want from '@ohos.app.ability.Want';

let want: Want = {
    action: 'action.access.privacy.center',
    parameters: {
        bundleName: 'com.example.app'  // 可选：直接跳转到指定应用
    }
};
this.context.startAbility(want);
```

**入口**: Settings → Privacy → Permission Manager

**证据**: `permissionmanager/src/main/module.json:30-52`

---

### 2. GrantAbility (ServiceExtAbility) - 权限请求对话框

| 属性 | 值 |
|------|-----|
| **注册名** | `com.ohos.permissionmanager.GrantAbility` |
| **类型** | ServiceExtensionAbility |
| **导出** | true |

**调用方式**:

**注意**: 此Ability由系统权限服务（AT服务）在应用调用 `requestPermissionsFromUser()` 时自动启动，**不应由应用直接调用**。

**系统调用流程**:

```typescript
// 应用侧调用
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
let atManager = abilityAccessCtrl.createAtManager();
atManager.requestPermissionsFromUser(this.context, permissions, callback);

// 系统内部自动启动GrantAbility
// 传递的Want参数：
{
    'ohos.aafwk.param.callerBundleName': string,      // 调用方包名
    'ohos.aafwk.param.callerToken': number,           // 调用方Token
    'ohos.user.grant.permission': Permission[],       // 请求的权限列表
    'ohos.user.grant.permission.state': number[],     // 权限初始状态
    'ohos.ability.params.callback': rpc.RemoteObject, // IPC回调对象
    'ohos.ability.params.token': Property             // 窗口绑定Token
}
```

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:56-88`

---

### 3. GlobalExtAbility - 全局开关对话框

| 属性 | 值 |
|------|-----|
| **注册名** | `com.ohos.permissionmanager.GlobalExtAbility` |
| **类型** | ServiceExtensionAbility |
| **导出** | true |
| **所需权限** | `ohos.permission.GET_SENSITIVE_PERMISSIONS` |

**职责**: 当某个权限的全局开关关闭时，显示对话框请求用户开启。

**调用方式**: 由系统在需要时自动启动。

**Want参数**:
```typescript
{
    'ohos.sensitive.resource': string  // 权限资源标识
}
```

**证据**: `permissionmanager/src/main/module.json:62-69`

---

### 4. SecurityExtAbility - 安全组件对话框

| 属性 | 值 |
|------|-----|
| **注册名** | `com.ohos.permissionmanager.SecurityExtAbility` |
| **类型** | ServiceExtensionAbility |
| **导出** | true |
| **所需权限** | `ohos.permission.GET_SENSITIVE_PERMISSIONS` |

**职责**: 为安全组件（如CameraButton、LocationButton）提供授权对话框。

**Want参数**:
```typescript
{
    'ohos.display.width': number,           // 显示宽度
    'ohos.display.height': number,          // 显示高度
    'ohos.caller.uid': number,              // 调用方UID
    'ohos.ability.params.windowId': number, // 窗口ID
    'ohos.ability.params.token': Property   // 绑定Token
}
```

**证据**: `permissionmanager/src/main/ets/SecurityExtAbility/SecurityExtAbility.ets:39-53`

---

### 5. PermissionStateSheetAbility - 权限状态Sheet

| 属性 | 值 |
|------|-----|
| **注册名** | `com.ohos.permissionmanager.PermissionStateSheetAbility` |
| **类型** | UIExtensionAbility (sys/commonUI) |
| **导出** | true |

**职责**: 以Sheet形式展示权限状态，支持在其他应用上方弹出。

**证据**: `permissionmanager/src/main/module.json:78-84`

---

### 6. GlobalSwitchSheetAbility - 全局开关Sheet

| 属性 | 值 |
|------|-----|
| **注册名** | `com.ohos.permissionmanager.GlobalSwitchSheetAbility` |
| **类型** | UIExtensionAbility (sys/commonUI) |
| **导出** | true |

**职责**: 以Sheet形式展示全局开关状态。

**证据**: `permissionmanager/src/main/module.json:85-91`

---

## Ability注册清单

| Ability | 注册名 | 类型 | 权限要求 |
|---------|--------|------|----------|
| MainAbility | com.ohos.permissionmanager.MainAbility | UIAbility | ACCESS_SECURITY_PRIVACY_CENTER |
| GrantAbility | com.ohos.permissionmanager.GrantAbility | ServiceExtensionAbility | 无 |
| GlobalExtAbility | com.ohos.permissionmanager.GlobalExtAbility | ServiceExtensionAbility | GET_SENSITIVE_PERMISSIONS |
| SecurityExtAbility | com.ohos.permissionmanager.SecurityExtAbility | ServiceExtensionAbility | GET_SENSITIVE_PERMISSIONS |
| PermissionStateSheetAbility | com.ohos.permissionmanager.PermissionStateSheetAbility | UIExtensionAbility | 无 |
| GlobalSwitchSheetAbility | com.ohos.permissionmanager.GlobalSwitchSheetAbility | UIExtensionAbility | 无 |

**证据**: `permissionmanager/src/main/module.json:28-91`

---

## 使用的系统API

PermissionManager作为系统应用，使用了大量OpenHarmony系统API。

### 1. 权限管理 API (@ohos.abilityAccessCtrl)

| API | 用途 | 位置 |
|-----|------|------|
| `createAtManager()` | 创建权限管理器 | 多处 |
| `grantUserGrantedPermission(tokenId, permission, flag)` | 授予用户权限 | GrantDialogModel.ets:430 |
| `revokeUserGrantedPermission(tokenId, permission, flag)` | 撤销用户权限 | GrantDialogModel.ets:457 |
| `verifyAccessTokenSync(tokenId, permission)` | 验证访问令牌 | MainAbility.ets:147 |

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:430-469`

---

### 2. 应用包管理 API (@ohos.bundle.bundleManager)

| API | 用途 | 位置 |
|-----|------|------|
| `getBundleInfo(bundleName, flag)` | 获取应用信息 | MainAbility.ets:212 |
| `getBundleInfoSync(bundleName, flag, userId)` | 同步获取应用信息 | GrantDialogModel.ets:71 |
| `getAllBundleInfo(flag, userId)` | 获取所有应用 | MainAbility.ets:169 |
| `getBundleInfoForSelfSync(flag)` | 获取自身信息 | MainAbility.ets:144 |
| `queryAbilityInfo(want, flag)` | 查询Ability信息 | MainAbility.ets:180 |

**证据**: `permissionmanager/src/main/ets/MainAbility/MainAbility.ets:159-205`

---

### 3. 应用包监控 API (@ohos.bundle.bundleMonitor)

| API | 用途 | 位置 |
|-----|------|------|
| `on('add', callback)` | 监听应用安装 | MainAbility.ets:54 |
| `on('remove', callback)` | 监听应用卸载 | MainAbility.ets:61 |
| `on('update', callback)` | 监听应用更新 | MainAbility.ets:68 |
| `off('add')` | 取消监听 | MainAbility.ets:120 |

**证据**: `permissionmanager/src/main/ets/MainAbility/MainAbility.ets:54-77`

---

### 4. 窗口管理 API (@ohos.window)

| API | 用途 | 位置 |
|-----|------|------|
| `createWindow(config)` | 创建窗口 | GrantDialogModel.ets:234 |
| `moveWindowTo(x, y)` | 移动窗口 | GrantDialogModel.ets:241 |
| `resize(width, height)` | 调整窗口大小 | GrantDialogModel.ets:243 |
| `loadContent(path, storage)` | 加载页面内容 | GrantDialogModel.ets:245 |
| `showWindow()` | 显示窗口 | GrantDialogModel.ets:247 |
| `destroyWindow()` | 销毁窗口 | GrantDialogModel.ets:519 |
| `bindDialogTarget(token, callback)` | 绑定对话框目标 | GrantDialogModel.ets:269 |

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:225-287`

---

### 5. IPC通信 API (@ohos.rpc)

| API | 用途 | 位置 |
|-----|------|------|
| `MessageOption()` | 创建消息选项 | GrantDialogModel.ets:500 |
| `MessageSequence.create()` | 创建消息序列 | GrantDialogModel.ets:501-503 |
| `writeInterfaceToken(token)` | 写入接口令牌 | GrantDialogModel.ets:505 |
| `writeStringArray(arr)` | 写入字符串数组 | GrantDialogModel.ets:506 |
| `writeIntArray(arr)` | 写入整数数组 | GrantDialogModel.ets:507 |
| `sendMessageRequest(code, data, reply, option)` | 发送IPC请求 | GrantDialogModel.ets:509 |
| `reclaim()` | 回收资源 | GrantDialogModel.ets:514-516 |

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:496-522`

---

### 6. 其他系统API

| API | 模块 | 用途 | 位置 |
|-----|------|------|------|
| `getDefaultDisplaySync()` | @ohos.display | 获取显示信息 | ServiceExtAbility.ets:47 |
| `getSystemPasteboard()` | @ohos.pasteboard | 获取剪贴板 | GrantDialogModel.ets:296 |
| `getForegroundOsAccountLocalId()` | @ohos.account.osAccount | 获取用户ID | GrantDialogModel.ets:99 |
| `getBundleResourceInfo()` | @ohos.bundle.bundleResourceManager | 获取应用资源 | GrantDialogModel.ets:316 |

---

## 应用声明的权限

PermissionManager作为系统应用，在 `module.json` 中声明了12个系统权限：

| 权限 | 用途 | 级别 |
|------|------|------|
| `ohos.permission.GET_SENSITIVE_PERMISSIONS` | 获取敏感权限信息 | system_grant |
| `ohos.permission.GRANT_SENSITIVE_PERMISSIONS` | 授予敏感权限 | system_grant |
| `ohos.permission.REVOKE_SENSITIVE_PERMISSIONS` | 撤销敏感权限 | system_grant |
| `ohos.permission.GET_BUNDLE_INFO` | 获取应用包信息 | system_grant |
| `ohos.permission.GET_BUNDLE_RESOURCES` | 获取应用资源 | system_grant |
| `ohos.permission.PERMISSION_USED_STATS` | 访问权限使用统计 | system_grant |
| `ohos.permission.GET_INSTALLED_BUNDLE_LIST` | 获取已安装应用列表 | user_grant |
| `ohos.permission.LISTEN_BUNDLE_CHANGE` | 监听应用变更 | system_grant |
| `ohos.permission.ACCESS_BUNDLE_DIR` | 访问应用目录 | system_grant |
| `ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER` | 访问安全隐私中心 | system_grant |
| `ohos.permission.MICROPHONE_CONTROL` | 控制麦克风 | system_grant |
| `ohos.permission.CAMERA_CONTROL` | 控制相机 | system_grant |

**证据**: `permissionmanager/src/main/module.json:93-132`

---

## 支持的权限列表

PermissionManager支持处理48个用户授权权限，分为21个权限组。

### 权限枚举 (definition.ets)

```typescript
export enum Permission {
    // 位置权限
    LOCATION_IN_BACKGROUND = 'ohos.permission.LOCATION_IN_BACKGROUND',
    APPROXIMATELY_LOCATION = 'ohos.permission.APPROXIMATELY_LOCATION',
    LOCATION = 'ohos.permission.LOCATION',
    
    // 媒体权限
    CAMERA = 'ohos.permission.CAMERA',
    MICROPHONE = 'ohos.permission.MICROPHONE',
    READ_IMAGEVIDEO = 'ohos.permission.READ_IMAGEVIDEO',
    WRITE_IMAGEVIDEO = 'ohos.permission.WRITE_IMAGEVIDEO',
    MEDIA_LOCATION = 'ohos.permission.MEDIA_LOCATION',
    READ_AUDIO = 'ohos.permission.READ_AUDIO',
    WRITE_AUDIO = 'ohos.permission.WRITE_AUDIO',
    READ_DOCUMENT = 'ohos.permission.READ_DOCUMENT',
    WRITE_DOCUMENT = 'ohos.permission.WRITE_DOCUMENT',
    READ_MEDIA = 'ohos.permission.READ_MEDIA',
    WRITE_MEDIA = 'ohos.permission.WRITE_MEDIA',
    
    // 通讯录和日历
    READ_CONTACTS = 'ohos.permission.READ_CONTACTS',
    WRITE_CONTACTS = 'ohos.permission.WRITE_CONTACTS',
    READ_CALENDAR = 'ohos.permission.READ_CALENDAR',
    WRITE_CALENDAR = 'ohos.permission.WRITE_CALENDAR',
    READ_WHOLE_CALENDAR = 'ohos.permission.READ_WHOLE_CALENDAR',
    WRITE_WHOLE_CALENDAR = 'ohos.permission.WRITE_WHOLE_CALENDAR',
    
    // 健康和运动
    ACTIVITY_MOTION = 'ohos.permission.ACTIVITY_MOTION',
    READ_HEALTH_DATA = 'ohos.permission.READ_HEALTH_DATA',
    
    // 其他权限
    APP_TRACKING_CONSENT = 'ohos.permission.APP_TRACKING_CONSENT',
    GET_INSTALLED_BUNDLE_LIST = 'ohos.permission.GET_INSTALLED_BUNDLE_LIST',
    DISTRIBUTED_DATASYNC = 'ohos.permission.DISTRIBUTED_DATASYNC',
    ACCESS_BLUETOOTH = 'ohos.permission.ACCESS_BLUETOOTH',
    READ_PASTEBOARD = 'ohos.permission.READ_PASTEBOARD',
    READ_WRITE_DOWNLOAD_DIRECTORY = 'ohos.permission.READ_WRITE_DOWNLOAD_DIRECTORY',
    READ_WRITE_DESKTOP_DIRECTORY = 'ohos.permission.READ_WRITE_DESKTOP_DIRECTORY',
    READ_WRITE_DOCUMENTS_DIRECTORY = 'ohos.permission.READ_WRITE_DOCUMENTS_DIRECTORY',
    ACCESS_NEARLINK = 'ohos.permission.ACCESS_NEARLINK',
    CUSTOM_SCREEN_CAPTURE = 'ohos.permission.CUSTOM_SCREEN_CAPTURE'
}
```

**证据**: `permissionmanager/src/main/ets/common/model/definition.ets:16-49`

---

## 错误码

PermissionManager在运行过程中可能返回的错误码：

| 错误码 | 常量 | 含义 | 位置 |
|--------|------|------|------|
| 1 | ERR_MODAL_ALREADY_EXIST | 对话框已存在 | constant.ets |
| 0 | RESULT_SUCCESS | 操作成功 | constant.ets |
| -1 | RESULT_FAILURE | 操作失败 | constant.ets |

**注意**: 具体错误码定义在 `constant.ets` 文件中。

---

## 应用如何集成

### 1. 请求权限

应用通过 `@ohos.abilityAccessCtrl` 请求权限：

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import { BusinessError } from '@ohos.base';

async function requestPermissions(): Promise<void> {
    let atManager: abilityAccessCtrl.AtManager = abilityAccessCtrl.createAtManager();
    try {
        atManager.requestPermissionsFromUser(
            getContext(), 
            ['ohos.permission.CAMERA', 'ohos.permission.MICROPHONE']
        ).then((data) => {
            console.info('Request result: ' + JSON.stringify(data));
            // data.authResults: 授权结果数组 [0: granted, -1: denied]
            // data.permissions: 请求的权限列表
        }).catch((err: BusinessError) => {
            console.error('Request failed: ' + JSON.stringify(err));
        });
    } catch (err) {
        console.error('Request exception: ' + JSON.stringify(err));
    }
}
```

### 2. 跳转权限管理设置

```typescript
import common from '@ohos.app.ability.common';
import Want from '@ohos.app.ability.Want';

async function jumpToPermissionSettings(): Promise<void> {
    let context = getContext() as common.UIAbilityContext;
    let want: Want = {
        action: 'action.access.privacy.center',
        parameters: {
            bundleName: context.applicationInfo.name  // 可选：直接跳转到本应用
        }
    };
    try {
        await context.startAbility(want);
    } catch (err) {
        console.error('Jump failed: ' + JSON.stringify(err));
    }
}
```

---

## 与security_access_token的关系

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (应用开发者)                        │
│                  requestPermissionsFromUser()                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                security_access_token 服务层                  │
│  - 验证权限请求合法性                                        │
│  - 检查是否需要用户授权                                      │
│  - 启动 PermissionManager.GrantAbility                      │
│  - 接收授权结果并更新权限状态                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              PermissionManager (UI层)                        │
│  ServiceExtAbility → 显示权限请求对话框 → 用户交互 → IPC返回结果 │
└─────────────────────────────────────────────────────────────┘
```

PermissionManager负责UI展示和用户交互，**不直接处理权限策略决策**，权限状态的管理由底层服务负责。

---

*上一篇: [架构说明](02_Architecture.md) | 返回 [README](README.md) | 下一篇: [内部API说明](04_Inner_API.md)*
