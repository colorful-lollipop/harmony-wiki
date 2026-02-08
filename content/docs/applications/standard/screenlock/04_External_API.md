# 04. 对外API

> 目的: 详细记录ScreenLock使用的所有系统API和权限  
> 适用范围: 开发者、安全审计人员

---

## 1. 权限清单

### 1.1 权限概览

| 类别 | 数量 | 说明 |
|------|------|------|
| system_grant | 17个 | 系统级权限，需系统签名 |
| normal | 5个 | 普通权限，用户授权 |
| **总计** | **22个** | 详见下文 |

### 1.2 完整权限列表

**来源**: `product/phone/src/main/module.json5:21-85`

| 序号 | 权限名称 | 级别 | 用途说明 |
|------|----------|------|----------|
| 1 | `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | system_grant | 管理本地用户账户 |
| 2 | `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS_EXTENSION` | system_grant | 跨本地账户扩展交互 |
| 3 | `ohos.permission.USE_USER_IDM` | system_grant | 使用用户身份管理 |
| 4 | `ohos.permission.ACCESS_USER_AUTH_INTERNAL` | system_grant | 内部用户认证访问 |
| 5 | `ohos.permission.ACCESS_PIN_AUTH` | system_grant | PIN码认证访问 |
| 6 | `ohos.permission.NOTIFICATION_CONTROLLER` | system_grant | 通知控制器权限 |
| 7 | `ohos.permission.GET_WALLPAPER` | system_grant | 获取锁屏壁纸 |
| 8 | `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | system_grant | 获取应用包信息（特权） |
| 9 | `ohos.permission.SET_WIFI_INFO` | system_grant | 设置WiFi信息 |
| 10 | `ohos.permission.MANAGE_WIFI_CONNECTION` | system_grant | 管理WiFi连接 |
| 11 | `ohos.permission.MANAGE_BLUETOOTH` | system_grant | 管理蓝牙 |
| 12 | `ohos.permission.CAPTURE_SCREEN` | system_grant | 屏幕截图 |
| 13 | `ohos.permission.MANAGE_SECURE_SETTINGS` | system_grant | 管理安全设置 |
| 14 | `ohos.permission.CONNECT_IME_ABILITY` | system_grant | 连接输入法Ability |
| 15 | `ohos.permission.ACCESS_SCREEN_LOCK_INNER` | system_grant | 内部锁屏访问 |
| 16 | `ohos.permission.ACCESS_SERVICE_DM` | system_grant | 服务分布式管理访问 |
| 17 | `ohos.permission.GET_WIFI_INFO` | normal | 获取WiFi信息 |
| 18 | `ohos.permission.GET_NETWORK_INFO` | normal | 获取网络信息 |
| 19 | `ohos.permission.USE_BLUETOOTH` | normal | 使用蓝牙 |
| 20 | `ohos.permission.DISCOVER_BLUETOOTH` | normal | 发现蓝牙设备 |
| 21 | `ohos.permission.SET_WALLPAPER` | normal | 设置壁纸 |
| 22 | `ohos.permission.REBOOT` | system_grant | 重启设备 (仅PC模块) |

**注**: PC模块比Phone模块多一个`REBOOT`权限，见`product/pc/src/main/module.json5:77`

### 1.3 权限使用详情

| 权限名 | 使用位置 | 代码用途 |
|--------|----------|----------|
| `MANAGE_LOCAL_ACCOUNTS` | `common/SwitchUserManager.ts:43`<br>`features/screenlock/model/accountsModel.ts:95` | 监听用户激活事件、获取账户管理器 |
| `USE_USER_IDM`<br>`ACCESS_USER_AUTH_INTERNAL` | `features/screenlock/model/accountsModel.ts:84,196` | 创建UserAuth实例、调用authUser() |
| `ACCESS_PIN_AUTH` | `features/screenlock/model/accountsModel.ts:85,243` | 创建PINAuth实例、注册密码输入器 |
| `ACCESS_SCREEN_LOCK_INNER` | `features/screenlock/model/screenLockModel.ts:17,29,49` | 调用ScreenLock服务API |
| `GET_WALLPAPER` | `features/wallpapercomponent/wallpaperViewModel.ts:17,41` | 获取锁屏壁纸PixelMap |
| `GET_WIFI_INFO`<br>`MANAGE_WIFI_CONNECTION` | `features/wificomponent/wifiModel.ts:16,122,143,148` | WiFi信息获取、启用/禁用WiFi |
| `REBOOT` | `features/shortcutcomponent/shortcutViewModel.ts:18,30` | PC端重启设备 (power.rebootDevice) |

---

## 2. 系统服务API清单

### 2.1 API使用统计

| 服务类别 | 服务数量 | 调用文件数 |
|----------|----------|------------|
| 系统服务 | 22个 | 48个文件 |
| 调用点总数 | 77+处 | - |

### 2.2 按类别分组的API

#### A. 锁屏与窗口服务

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.screenLock` | `features/screenlock/model/screenLockModel.ts` | `onSystemEvent()`, `sendScreenLockEvent()` | 锁屏事件监听与通知 |
| `@ohos.window` | `common/WindowManager.ts`, `phone/ServiceExtAbility.ts` | `create()`, `show()`, `hide()`, `loadContent()` | 窗口管理 |
| `@ohos.display` | `phone/ServiceExtAbility.ts` | `getDefaultDisplay()` | 获取屏幕信息 |

**关键调用示例**:

```typescript
// screenLockModel.ts - 监听系统事件
ScreenLockMar.onSystemEvent((event) => {
    callback(event.eventType);
});

// 发送锁屏事件
ScreenLockMar.sendScreenLockEvent(typeName, typeNo, (err, data) => {
    callback(err, data);
});

// ServiceExtAbility.ts - 创建锁屏窗口
windowManager.create(context, name, windowManager.WindowType.TYPE_KEYGUARD)
    .then((win) => {
        win.loadContent("pages/index");
        win.show();
    });
```

#### B. 账户与认证服务

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.account.osAccount` | `features/screenlock/model/accountsModel.ts`, `common/SwitchUserManager.ts` | `authUser()`, `getProperty()`, `registerInputer()` | 用户认证 |

**关键调用示例**:

```typescript
// accountsModel.ts - 用户认证
this.userAuthManager.authUser(userId, challenge, authType, authLevel, {
    onResult: (result, extraInfo) => {
        callback(result, extraInfo);
    },
    onAcquireInfo: (moduleId, acquire, extraInfo) => { }
});

// 注册PIN输入器
this.pinAuthManager.registerInputer({
    onGetData: (passType, inputData) => {
        inputData.onSetData(passType, uint8PW);
    }
});
```

#### C. 通知服务

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.notification` | `features/noticeitem/model/NotificationManager.ts`, `common/notificationManager.ts` | `subscribe()`, `unsubscribe()`, `getAllActiveNotifications()`, `remove()` | 通知管理 |

**关键调用示例**:

```typescript
// NotificationManager.ts
Notification.subscribe(subscriber, asyncCallback);
Notification.getAllActiveNotifications(callback);
Notification.remove(hashCode, removeReason, callback);
Notification.removeAll(callback);
```

#### D. 设备状态服务

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.batteryInfo` | `features/batterycomponent/batteryModel.ts` | `batterySOC`, `chargingStatus` | 电池状态 |
| `@ohos.telephony.radio` | `features/signalcomponent/signalModel.ts` | `getNetworkState()`, `getSignalInformation()` | 网络信号 |
| `@ohos.telephony.sim` | `features/signalcomponent/signalModel.ts` | `hasSimCard()` | SIM卡状态 |
| `@ohos.telephony.observer` | `features/signalcomponent/signalModel.ts` | `on('signalInfoChange')` | 信号监听 |
| `@ohos.wifi` | `features/wificomponent/wifiModel.ts` | `getLinkedInfo()`, `on('wifiStateChange')` | WiFi状态 |
| `@ohos.wallpaper` | `features/wallpapercomponent/wallpaperViewModel.ts` | `getPixelMap()` | 壁纸获取 |
| `@ohos.power` | `features/shortcutcomponent/shortcutViewModel.ts` | `shutdown()`, `rebootDevice()` | 电源控制 |

#### E. 系统事件与设置

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.commonEvent` | `common/ScreenLockManager.ts`, `common/commonEvent/CommonEventManager.ts` | `createSubscriber()`, `subscribe()`, `publish()` | 公共事件 |
| `@ohos.settings` | `common/TimeManager.ts` | `getValueSync()` | 设置读取 |
| `@ohos.data.dataShare` | `common/TimeManager.ts` | `createDataShareHelper()`, `on('dataChange')` | 数据共享 |
| `@ohos.systemparameter` | `features/screenlock/model/screenLockService.ts` | `set()`, `getSync()` | 系统参数 |

#### F. 分布式与设备

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.distributedDeviceManager` | `features/noticeitem/model/NotificationDistributionManager.ts` | `createDeviceManager()`, `getAvailableDeviceListSync()` | 分布式设备 |
| `@ohos.deviceInfo` | 多个UI组件 | 设备信息属性 | 设备信息 |

#### G. 多媒体与工具

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.multimedia.image` | `features/wallpapercomponent/wallpaperViewModel.ts` | 图像处理 | 壁纸处理 |
| `@ohos.multimedia.media` | `features/noticeitem/viewmodel/ViewModel.ts` | 音频播放 | 通知音效 |
| `@ohos.multimedia.audio` | `common/SingleInstanceHelper.ts` | 音频管理 | 音频控制 |
| `@ohos.wantAgent` | `features/noticeitem/view/item/notificationItem.ets` | Want代理 | 通知跳转 |
| `@ohos.bundle` | `common/abilitymanager/bundleManager.ts` | `getBundleInfo()` | 包信息 |
| `@ohos.util` | `features/screenlock/model/accountsModel.ts` | `TextEncoder` | 文本编码 |
| `@ohos.i18n` | `features/datetimecomponent/dateTime.ets` | 国际化 | 日期时间 |

#### H. 日志与调试

| 服务模块 | 导入位置 | 核心API | 用途 |
|----------|----------|---------|------|
| `@ohos.hilog` | `common/Log.ts` | `debug()`, `info()`, `error()` | 日志输出 |
| `@ohos.hiTraceMeter` | `common/Trace.ts` | `startTrace()`, `finishTrace()` | 性能追踪 |
| `@ohos.hidebug` | `features/screenlock/model/screenLockService.ts` | `getPss()` | 调试信息 |

#### I. Ability框架

| 服务模块 | 导入位置 | 用途 |
|----------|----------|------|
| `@ohos.app.ability.ServiceExtensionAbility` | `phone/ServiceExtAbility.ts` | 服务扩展基类 |
| `@ohos.app.ability.UIAbility` | `entry/MainAbility/MainAbility.ts` | UIAbility基类 |
| `@ohos.app.ability.AbilityStage` | 各模块AbilityStage.ts | AbilityStage基类 |
| `@ohos.arkui.UIContext` | `phone/ServiceExtAbility.ts` | UI上下文 |
| `@ohos.ability.featureAbility` | `common/abilitymanager/featureAbilityManager.ts` | FeatureAbility |
| `@ohos.app.ability.Want` | `features/noticeitem/view/item/notificationItem.ets` | Want对象 |
| `@ohos.base` | 多个ViewModel | Callback类型 |

---

## 3. API调用链分析

### 3.1 锁屏启动链

```
ServiceExtAbility.onCreate()
    ├── windowManager.create() ──► @ohos.window
    ├── win.loadContent()
    ├── win.show()
    ├── AbilityManager.setContext()
    └── TimeManager.init() ──► @ohos.data.dataShare
        └── @ohos.settings
```

### 3.2 解锁验证链

```
DigitalPSDViewModel.onKeyPress()
    └── ScreenLockService.authUser()
        └── AccountsModel.authUser()
            ├── pinAuthManager.registerInputer()
            └── userAuthManager.authUser() ──► @ohos.account.osAccount
                └── onResult callback
                    └── ScreenLockService.unlocking()
                        ├── ScreenLockModel.hiddenScreenLockWindow()
                        │   └── windowManager.find().hide() ──► @ohos.window
                        └── ScreenLockMar.sendScreenLockEvent() ──► @ohos.screenLock
```

### 3.3 通知显示链

```
NotificationManager.subscribe()
    └── Notification.subscribe() ──► @ohos.notification
        └── onConsume callback
            └── EventManager.publish()
                └── EventBus.emit()
                    └── NotificationList.refresh()
```

---

## 4. 权限与API对应关系

| 权限 | 使用的API | 文件位置 |
|------|-----------|----------|
| `MANAGE_LOCAL_ACCOUNTS` | `osAccount.activateOsAccount()` | `SwitchUserManager.ts` |
| `USE_USER_IDM` | `userAuthManager.authUser()` | `accountsModel.ts` |
| `ACCESS_PIN_AUTH` | `pinAuthManager.registerInputer()` | `accountsModel.ts` |
| `ACCESS_USER_AUTH_INTERNAL` | 内部认证流程 | `accountsModel.ts` |
| `NOTIFICATION_CONTROLLER` | `Notification.subscribe/remove` | `NotificationManager.ts` |
| `GET_WALLPAPER` | `WallpaperMar.getPixelMap()` | `wallpaperViewModel.ts` |
| `CAPTURE_SCREEN` | 窗口截图 | - |
| `ACCESS_SCREEN_LOCK_INNER` | `ScreenLockMar.*` | `screenLockModel.ts` |
| `MANAGE_WIFI_CONNECTION` | `wifi.enable/disableWifi()` | `wifiModel.ts` |
| `GET_WIFI_INFO` | `wifi.getLinkedInfo()` | `wifiModel.ts` |
| `MANAGE_BLUETOOTH` | 蓝牙管理 | - |
| `USE_BLUETOOTH` | 蓝牙使用 | - |

---

## 5. 敏感API说明

### 5.1 高风险API

| API | 风险等级 | 说明 | 防护措施 |
|-----|----------|------|----------|
| `userAuthManager.authUser()` | 高 | 用户认证，涉及敏感凭证 | 仅系统应用可调用 |
| `pinAuthManager.registerInputer()` | 高 | 密码输入器注册 | 需ACCESS_PIN_AUTH权限 |
| `power.shutdown/rebootDevice()` | 高 | 关机和重启 | 需系统权限 |
| `window.create(TYPE_KEYGUARD)` | 中 | 创建锁屏窗口 | 需系统签名 |
| `Notification.removeAll()` | 中 | 清除所有通知 | 需NOTIFICATION_CONTROLLER |

### 5.2 API调用频率

| API | 调用频率 | 优化建议 |
|-----|----------|----------|
| `BatteryInfo.batterySOC` | 低（状态变化时） | - |
| `wifi.getLinkedInfo()` | 中（状态变化时） | 使用监听而非轮询 |
| `Radio.getSignalInformation()` | 中（状态变化时） | 使用监听而非轮询 |
| `ScreenLockMar.onSystemEvent()` | 低（事件驱动） | - |
| `Notification.getAllActiveNotifications()` | 中（通知变化时） | - |

---

## 6. 遗留API

| API | 位置 | 说明 | 建议 |
|-----|------|------|------|
| `@system.router` | `phone/pages/index.ets` | 旧版路由API | 迁移至@ohos.router |

---

*关键结论: ScreenLock使用22个系统权限，调用22类系统服务，共77+处API调用。核心依赖@ohos.screenLock、@ohos.window、@ohos.account.osAccount、@ohos.notification四大服务。*
