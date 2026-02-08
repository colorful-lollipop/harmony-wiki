# N-API 接口参考

## 模块概览

| 模块 | 注册文件 | 导出名称 | 职责 |
|------|----------|----------|------|
| notification | `frameworks/js/napi/include/init.h:38-48` | `"notification"` | 核心通知API |
| notificationManager | `frameworks/js/napi/include/manager/init_module.h:38-47` | `"notificationManager"` | 通知管理API |
| notificationSubscribe | `frameworks/js/napi/include/subscribe/init_module.h:38-47` | `"notificationSubscribe"` | 订阅管理API |
| notificationExtensionSubscription | `frameworks/js/napi/include/extension_subscription/init_module.h:38-47` | `"notificationExtensionSubscription"` | 扩展订阅API |
| reminderAgent | `frameworks/js/napi/src/reminder/native_module.cpp:84-92` | `"reminderAgent"` | 提醒代理API |
| reminderAgentManager | `frameworks/js/napi/src/reminder/native_module_manager.cpp:104-112` | `"reminderAgentManager"` | 提醒管理API |

## N-API 注册模式

```cpp
// 模块定义结构
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "moduleName",
    .nm_priv = ((void *)0),
    .reserved = {0}
};

// 基于构造函数的注册
__attribute__((constructor)) void RegisterModule(void) {
    napi_module_register(&_module);
}
```

**证据**: `frameworks/js/napi/include/init.h:38-48`

---

## 核心通知 API (notification 模块)

**文件**: `frameworks/js/napi/src/init.cpp:40-122`

### 发布类

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `publish` | `Publish` | 发布通知 |
| `publishAsBundle` | `PublishAsBundle` | 作为代理发布 |

### 订阅类

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `subscribe` | `Subscribe` | 订阅通知 |
| `unsubscribe` | `Unsubscribe` | 取消订阅 |

### 取消类

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `cancel` | `Cancel` | 取消指定通知 |
| `cancelAll` | `CancelAll` | 取消所有通知 |
| `cancelGroup` | `CancelGroup` | 取消分组 |
| `cancelAsBundle` | `CancelAsBundle` | 作为代理取消 |

### 通道管理

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `addSlot` | `AddSlot` | 添加通道 |
| `addSlots` | `AddSlots` | 批量添加通道 |
| `getSlot` | `GetSlot` | 获取通道 |
| `getSlots` | `GetSlots` | 获取所有通道 |
| `removeSlot` | `RemoveSlot` | 删除通道 |
| `removeAllSlots` | `RemoveAllSlots` | 删除所有通道 |

### 查询类

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `getAllActiveNotifications` | `GetAllActiveNotifications` | 获取所有活动通知 |
| `getActiveNotifications` | `GetActiveNotifications` | 获取活动通知 |
| `getActiveNotificationCount` | `GetActiveNotificationCount` | 获取活动通知数量 |

---

## 通知管理 API (notificationManager 模块)

**文件**: `frameworks/js/napi/src/manager/init_module.cpp:78-248`

### 发布功能

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `publish` | `NapiPublish` | 发布通知 |
| `publishAsBundle` | `NapiPublishAsBundle` | 代理发布 |
| `show` | `NapiShowNotification` | 显示实时通知 |

### 取消功能

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `cancel` | `NapiCancel` | 取消通知 |
| `cancelAll` | `NapiCancelAll` | 取消所有 |
| `cancelGroup` | `NapiCancelGroup` | 取消分组 |
| `cancelAsBundle` | `NapiCancelAsBundle` | 代理取消 |

### 徽章管理

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `displayBadge` | `NapiDisplayBadge` | 显示徽章 |
| `isBadgeDisplayed` | `NapiIsBadgeDisplayed` | 徽章状态 |
| `getBadgeNumber` | `NapiGetBadgeNumber` | 获取徽章数 |
| `setBadgeNumber` | `NapiSetBadgeNumber` | 设置徽章数 |
| `setBadgeNumberByBundle` | `NapiSetBadgeNumberByBundle` | 设置应用徽章 |

### 免打扰

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `setDoNotDisturbDate` | `NapiSetDoNotDisturbDate` | 设置勿扰时间 |
| `getDoNotDisturbDate` | `NapiGetDoNotDisturbDate` | 获取勿扰时间 |
| `addDoNotDisturbProfile` | `NapiAddDoNotDisturbProfiles` | 添加勿扰配置 |
| `supportDoNotDisturbMode` | `NapiSupportDoNotDisturbMode` | 支持勿扰模式 |

### 分布式

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `isDistributedEnabled` | `NapiIsDistributedEnabled` | 分布式开关 |
| `setDistributedEnable` | `NapiEnableDistributed` | 设置分布式 |
| `setDistributedEnableByBundle` | `NapiEnableDistributedByBundle` | 按应用设置 |
| `getDistributedDeviceList` | `NapiGetDistributedDeviceList` | 获取设备列表 |

### 优先级

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `setPriorityEnabled` | `NapiSetPriorityEnabled` | 设置优先级 |
| `isPriorityEnabled` | `NapiIsPriorityEnabled` | 优先级状态 |
| `setBundlePriorityConfig` | `NapiSetBundlePriorityConfig` | 配置优先级 |

---

## 订阅管理 API (notificationSubscribe 模块)

**文件**: `frameworks/js/napi/src/subscribe/init_module.cpp:32-39`

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `subscribe` | `NapiSubscribe` | 订阅通知 |
| `unsubscribe` | `NapiUnsubscribe` | 取消订阅 |
| `remove` | `NapiRemove` | 移除通知 |
| `removeAll` | `NapiRemoveAll` | 移除所有 |
| `subscribeSelf` | `NapiSubscribeSelf` | 自订阅 |
| `distributeOperation` | `NapiDistributeOperation` | 分布式操作 |

---

## 扩展订阅 API (notificationExtensionSubscription 模块)

**文件**: `frameworks/js/napi/src/extension_subscription/init_module.cpp:30-41`

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `subscribe` | `NapiNotificationExtensionSubscribe` | 扩展订阅 |
| `unsubscribe` | `NapiNotificationExtensionUnsubscribe` | 取消扩展 |
| `getSubscribeInfo` | `NapiGetSubscribeInfo` | 获取订阅信息 |
| `isUserGranted` | `NapiIsUserGranted` | 用户授权状态 |
| `getUserGrantedState` | `NapiGetUserGrantedState` | 获取授权状态 |
| `setUserGrantedState` | `NapiSetUserGrantedState` | 设置授权状态 |

---

## 提醒代理 API (reminderAgent 模块)

**文件**: `frameworks/js/napi/src/reminder/native_module.cpp:26-37`

| JS方法 | C++绑定 | 说明 |
|--------|--------|------|
| `publishReminder` | `PublishReminder` | 发布提醒 |
| `cancelReminder` | `CancelReminder` | 取消提醒 |
| `cancelAllReminders` | `CancelAllReminders` | 取消所有 |
| `getValidReminders` | `GetValidReminders` | 获取有效提醒 |
| `addNotificationSlot` | `AddSlot` | 添加通道 |
| `removeNotificationSlot` | `NotificationNapi::RemoveSlot` | 删除通道 |

---

## 常量导出

**文件**: `frameworks/js/napi/src/constant.cpp`

| 枚举类型 | 用途 |
|----------|------|
| `RemoveReason` | 移除原因 |
| `SlotType` | 通道类型 |
| `SlotLevel` | 通道级别 |
| `SemanticActionButton` | 语义操作按钮 |
| `DoNotDisturbMode` | 勿扰模式 |
| `ContentType` | 内容类型 |
| `DeviceRemindType` | 设备提醒类型 |
| `PriorityNotificationType` | 优先级通知类型 |

---

## Inner API 接口

**主头文件**: `interfaces/inner_api/notification_helper.h`

### 核心功能分类

| 类别 | 关键方法 |
|------|----------|
| **通道管理** | `AddNotificationSlot`, `RemoveNotificationSlot`, `GetNotificationSlot` |
| **发布** | `PublishNotification`, `PublishAsBundle` |
| **取消** | `CancelNotification`, `CancelAllNotifications` |
| **订阅** | `SubscribeNotification`, `UnSubscribeNotification` |
| **设置** | `IsAllowedNotify`, `RequestEnableNotification` |
| **徽章** | `SetNotificationBadgeNum`, `GetShowBadgeEnabledForBundle` |
| **免打扰** | `SetDoNotDisturbDate`, `AddDoNotDisturbProfiles` |
| **分布式** | `EnableDistributed`, `SetDistributedEnabledBySlot` |
| **优先级** | `SetPriorityEnabled`, `SetBundlePriorityConfig` |

---

## 通知订阅扩展模式

### 扩展能力模块注册

```cpp
extern "C" __attribute__((constructor))
void NAPI_application_NotificationSubscriberExtensionAbility_AutoRegister()
{
    auto moduleManager = NativeModuleManager::GetInstance();
    NativeModule newModuleInfo = {
        .name = "application.NotificationSubscriberExtensionAbility",
        .fileName = "application/libnotificationsubscriberextensionability_napi.so",
    };
    moduleManager->Register(&newModuleInfo);
}
```

**证据**: `interfaces/kits/napi/notification_subscriber_extension/notification_subscriber_extension_ability_module.cpp`

### 生命周期方法

| 方法 | 绑定行号 | 说明 |
|------|----------|------|
| `OnStart` | 173 | 启动 |
| `OnStop` | 179 | 停止 |
| `OnConnect` | 186 | 连接 |
| `OnDisconnect` | 199 | 断开 |
| `OnDestroy` | 210 | 销毁 |
| `OnReceiveMessage` | 239 | 接收消息 |
| `OnCancelMessages` | 293 | 取消消息 |

**证据**: `frameworks/js/napi/src/extension/js_notification_subscriber_extension.cpp`

---

## API 调用链示例

### 发布通知流程

```
JS: notificationManager.publish(notification)
    ↓
C++: NapiPublish()
    ↓
Inner API: NotificationHelper::PublishNotification()
    ↓
IPC: IAnsManager::Publish()
    ↓
Service: AdvancedNotificationService::Publish()
```

### 订阅通知流程

```
JS: notificationSubscribe.subscribe(subscriber)
    ↓
C++: NapiSubscribe()
    ↓
IPC: IAnsManager::Subscribe()
    ↓
Service: NotificationSubscriberManager::AddSubscriber()
```
