# 附录A: 完整API清单

## 1. IAnsManager IPC接口 (446+ 方法)

**定义文件**: `frameworks/ans/IAnsManager.idl`

### 通知发布

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| Publish | label, notification | - | 发布通知 |
| PublishWithMaxCapacity | label, notification | - | 大量数据发布 |
| PublishNotificationForIndirectProxy | notification | - | 间接代理发布 |
| PublishAsBundle | notification, representativeBundle | - | 作为代理发布 |
| PublishContinuousTaskNotification | request | - | 发布持续任务通知 |

### 通知取消

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| Cancel | notificationId, label, instanceKey | - | 取消通知 |
| CancelAll | instanceKey | - | 取消所有 |
| CancelAsBundle | notificationId, representativeBundle, userId | - | 作为代理取消 |
| CancelContinuousTaskNotification | label, notificationId | - | 取消持续任务 |

### 通道管理

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| AddSlotByType | slotTypeInt | - | 按类型添加 |
| AddSlots | slots | - | 批量添加 |
| RemoveSlotByType | slotTypeInt | - | 按类型删除 |
| RemoveAllSlots | - | - | 删除所有 |
| GetSlotByType | slotTypeInt | slot | 获取通道 |
| GetSlots | - | slots | 获取所有 |
| UpdateSlots | bundleOption, slots | - | 更新通道 |

### 订阅管理

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| Subscribe | subscriber, info, subscribedFlags | - | 订阅 |
| SubscribeSelf | subscriber, subscribedFlags | - | 自订阅 |
| Unsubscribe | subscriber, info | - | 取消订阅 |
| SubscribeLocalLiveView | subscriber, info, isNative | - | 实时订阅 |

### 权限设置

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| IsAllowedNotify | - | allowed | 检查权限 |
| RequestEnableNotification | deviceId, callback | - | 请求开启 |
| SetNotificationsEnabledForBundle | deviceId, enabled | - | 按应用设置 |
| SetNotificationsEnabledForAllBundles | deviceId, enabled | - | 全局设置 |

### 免打扰

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| SetDoNotDisturbDate | date | - | 设置勿扰 |
| GetDoNotDisturbDate | - | date | 获取勿扰 |
| AddDoNotDisturbProfiles | profiles | - | 添加配置 |
| DoesSupportDoNotDisturbMode | - | doesSupport | 检查支持 |

### 分布式

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| IsDistributedEnabled | - | enabled | 检查开关 |
| EnableDistributed | enabled | - | 设置开关 |
| EnableDistributedByBundle | bundleOption, enabled | - | 按应用设置 |
| GetDistributedDevicelist | - | deviceTypes | 获取设备 |

### 徽章

| 方法名 | 输入参数 | 输出参数 | 说明 |
|--------|----------|----------|------|
| SetNotificationBadgeNum | num | - | 设置徽章 |
| SetBadgeNumber | badgeNumber, instanceKey | - | 设置数字 |
| SetBadgeNumberByBundle | bundleOption, badgeNumber | - | 按应用设置 |
| GetBadgeNumber | - | badgeNumber | 获取数字 |

---

## 2. Inner API 核心方法

**定义文件**: `interfaces/inner_api/notification_helper.h`

### 静态方法分类

#### 通道管理
```cpp
static ErrCode AddNotificationSlot(const NotificationSlot &slot);
static ErrCode AddSlotByType(const NotificationConstant::SlotType &slotType);
static ErrCode RemoveNotificationSlot(const NotificationConstant::SlotType &slotType);
static ErrCode GetNotificationSlot(const NotificationConstant::SlotType &slotType, sptr<NotificationSlot> &slot);
static ErrCode GetNotificationSlots(std::vector<sptr<NotificationSlot>> &slots);
```

#### 发布与取消
```cpp
static ErrCode PublishNotification(const std::string &label, const NotificationRequest &request);
static ErrCode CancelNotification(int notificationId, const std::string &label);
static ErrCode CancelAllNotifications(const std::string &instanceKey);
```

#### 权限控制
```cpp
static ErrCode IsAllowedNotify(bool &allowed);
static ErrCode IsAllowedNotifySelf(bool &allowed);
static ErrCode RequestEnableNotification(const std::string &deviceId, const sptr<IRemoteObject> &callback);
```

---

## 3. 回调接口

### IAnsSubscriber
| 方法 | 说明 |
|------|------|
| OnConnected | 连接成功 |
| OnDisconnected | 断开连接 |
| OnConsumed | 通知被消费 |
| OnCanceled | 通知被取消 |
| OnBadgeChanged | 徽章变更 |

### IAnsDialogCallback
| 方法 | 说明 |
|------|------|
| OnDialogStatusChanged | 对话框状态变更 |

### IAnsOperationCallback
| 方法 | 说明 |
|------|------|
| OnOperationCallback | 操作结果回调 |
