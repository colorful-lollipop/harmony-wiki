# 03. 系统 API

## 目的与适用范围

本文档列出 MMS 应用使用的所有 OpenHarmony 系统 API，包括模块、接口和调用位置。

**适用读者**: 开发工程师、安全审计人员  
**阅读时间**: 约 20 分钟

---

## API 概览

MMS 应用使用了 **15+** 个 OpenHarmony 系统模块，按功能分类如下：

| 功能类别 | 模块 | 使用文件数 |
|----------|------|------------|
| 电话/短信 | @ohos.telephony.* | 6 |
| 数据存储 | @ohos.data.* | 5 |
| 系统能力 | @ohos.app.ability.* | 4 |
| 通知/事件 | @ohos.notificationManager, @ohos.commonEventManager | 3 |
| 网络 | @ohos.net.http | 2 |
| UI/交互 | @ohos.mediaquery, @ohos.promptAction | 4 |
| 其他 | @ohos.deviceInfo, @ohos.pasteboard | 2 |

---

## 电话/短信 API

### @ohos.telephony.sms

**用途**: 短信发送、接收、SMSC 地址获取

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `sendMessage()` | SendMsgService.ets | 33 | 发送短信 |
| `SendSmsResult.SEND_SMS_SUCCESS` | SendMsgService.ets | 63 | 发送结果判断 |
| `getSmscAddr()` | CardModel.ets | 253 | 获取短信中心号码 |
| `createMessage()` | MmsStaticSubscriber.ts | 49 | 解析接收的 PDU |

**关键代码示例**:
```typescript
// entry/src/main/ets/service/SendMsgService.ets:31
sendMessage(params, callback) {
    sms.sendMessage({
        slotId: Number(params.slotId),
        destinationHost: params.destinationHost,
        content: params.content,
        sendCallback: (err, value) => { ... },
        deliveryCallback: (err, value) => { ... }
    });
}
```

### @ohos.telephony.sim

**用途**: SIM 卡状态、信息获取

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `getMaxSimCount()` | CardModel.ets | 74, 94 | 获取最大 SIM 卡数 |
| `getSimState()` | CardModel.ets | 79 | 获取 SIM 卡状态 |
| `getSimSpn()` | CardModel.ets | 158 | 获取运营商名称 |
| `getDefaultVoiceSlotId()` | CardModel.ets | 145 | 获取默认语音卡槽 |
| `getSimTelephoneNumber()` | CardModel.ets | 211 | 获取 SIM 卡号码 |
| `getSimOperatorNumeric()` | CardModel.ets | 178 | 获取运营商编号 |
| `SimState.SIM_STATE_READY` | CardModel.ets | 38 | SIM 就绪状态判断 |

### @ohos.telephony.observer

**用途**: SIM 卡状态监听

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `on('simStateChange')` | CardModel.ets | 95 | 订阅 SIM 状态变化 |
| `off('simStateChange')` | CardModel.ets | 90 | 取消订阅 |

### @ohos.telephony.radio

**用途**: 运营商网络信息

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `getOperatorName()` | CardModel.ets | 168 | 获取网络运营商名 |

### @ohos.telephony.call

**用途**: 拨打电话

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `makeCall()` | CallService.ets | ~30 | 拨打号码 |

---

## 数据存储 API

### @ohos.data.dataShare

**用途**: 跨应用数据共享（短信/联系人数据库）

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `createDataShareHelper()` | ConversationModel.ets | 30 | 创建 DataShare 帮助类 |
| `insert()` | ConversationModel.ets | 34 | 插入数据 |
| `delete()` | ConversationModel.ets | 51 | 删除数据 |
| `update()` | ConversationModel.ets | 68 | 更新数据 |
| `query()` | ConversationModel.ets | 85 | 查询数据 |

**关键代码示例**:
```typescript
// entry/src/main/ets/model/ConversationModel.ets:29
public async insertSmsMmsInfo(valueBucket, callback, context): Promise<void> {
    let dataHelper = await dataShare.createDataShareHelper(context,
        common.string.URI_MESSAGE_LOG);  // "datashare:///com.ohos.smsmmsability"
    let managerUri = common.string.URI_MESSAGE_LOG + 
        common.string.URI_MESSAGE_INFO_TABLE;  // "/sms_mms/sms_mms_info"
    dataHelper.insert(managerUri, valueBucket).then(res => {
        callback(this.encapsulateReturnResult(common.int.SUCCESS, res));
    });
}
```

**数据 URI**:

| URI | 说明 |
|-----|------|
| `datashare:///com.ohos.smsmmsability` | 短信数据库 |
| `/sms_mms/sms_mms_info` | 短信详情表 |
| `/sms_mms/session` | 会话表 |
| `/sms_mms/sms_mms_info/unread_total` | 未读统计视图 |
| `datashare:///com.ohos.contactsdataability` | 联系人数据库 |
| `/contacts/contact_data` | 联系人数据表 |
| `/contacts/search_contact` | 联系人搜索视图 |

### @ohos.data.dataSharePredicates

**用途**: 构建数据查询条件

**调用位置**:

| 方法 | 文件 | 用途 |
|------|------|------|
| `DataSharePredicates()` | ConversationModel.ets | 创建查询条件 |
| `equalTo()` | ConversationModel.ets | 等于条件 |
| `and()` | ConversationModel.ets | 与条件 |
| `in()` | ConversationModel.ets | IN 条件 |
| `isNotNull()` | ConversationModel.ets | 非空条件 |

### @ohos.data.rdb

**用途**: 关系型数据库（MmsDatabaseHelper 中使用）

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `getRdbStore()` | MmsDatabaseHelper.ets | ~50 | 获取 RDB 实例 |
| `executeSql()` | MmsDatabaseHelper.ets | ~70 | 执行 SQL |

### @ohos.data.preferences

**用途**: 轻量级偏好设置存储

**调用位置**:

| 方法 | 文件 | 用途 |
|------|------|------|
| `getPreferences()` | MmsPreferences.ets | 获取 Preferences 实例 |
| `put()` | MmsPreferences.ets | 存储配置项 |
| `get()` | MmsPreferences.ets | 读取配置项 |
| `flush()` | MmsPreferences.ets | 持久化 |

---

## 通知/事件 API

### @ohos.notificationManager

**用途**: 发送/取消通知

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `publish()` | NotificationService.ets | 55 | 发送通知 |
| `cancel()` | NotificationService.ets | 146 | 取消通知 |
| `cancelAll()` | NotificationService.ets | 156 | 取消所有通知 |
| `setNotificationEnable()` | MyAbilityStage.ts | 25 | 启用通知 |

**关键代码示例**:
```typescript
// entry/src/main/ets/service/NotificationService.ets:46
sendNotify(actionData) {
    let notificationRequest = {
        content: {
            contentType: Notification.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
            normal: {
                title: message.title,
                text: message.text
            }
        },
        slotType: Notification.SlotType.OTHER_TYPES
    };
    Notification.publish(notificationRequest);
}
```

### @ohos.app.ability.wantAgent

**用途**: 创建通知点击跳转的 WantAgent

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `getWantAgent()` | NotificationService.ets | 67 | 创建 WantAgent |
| `OperationType.START_ABILITY` | NotificationService.ets | 94 | 启动 Ability 操作 |

### @ohos.commonEventManager

**用途**: 公共事件订阅与发布

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `createSubscriber()` | CardModel.ets | 234 | 创建订阅者 |
| `subscribe()` | CardModel.ets | 240 | 订阅 SPN 变化事件 |
| `unsubscribe()` | CardModel.ets | 225 | 取消订阅 |
| `publish()` | MmsStaticSubscriber.ts | 203 | 发布接收转发事件 |

### @ohos.events.emitter

**用途**: 应用内事件发射

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `emit()` | CardModel.ets | 118 | 发送 SIM 状态变化事件 |

---

## 系统能力 API

### @ohos.app.ability.UIAbility

**用途**: Ability 基类

**调用位置**:

| 文件 | 行号 | 用途 |
|------|------|------|
| MainAbility.ts | 15 | 继承 Ability |
| MyAbilityStage.ts | 15 | 继承 AbilityStage |

### @ohos.window

**用途**: 窗口管理

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `loadContent()` | MainAbility.ts | 52 | 加载页面内容 |

### @kit.AbilityKit

**用途**: Ability Kit 功能

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `errorManager.on()` | MainAbility.ts | 36 | 全局错误监听 |

### @ohos.worker

**用途**: 多线程 Worker

**调用位置**:

| 方法/类型 | 文件 | 用途 |
|-----------|------|------|
| `ThreadWorkerGlobalScope` | DataWorkerWrapper.ets | Worker 全局作用域 |
| `MessageEvents` | DataWorkerWrapper.ets | 消息事件 |
| `worker.spawn()` | base/Worker.ts | 创建 Worker |

---

## 网络 API

### @ohos.net.http

**用途**: HTTP 网络请求（MMS 发送/接收）

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `createHttp()` | SendMsgService.ets | 72 | 创建 HTTP 请求 |
| `request()` | SendMsgService.ets | 73 | 发送 MMS |
| `createHttp()` | MmsStaticSubscriber.ts | 98 | 下载 MMS 附件 |
| `request()` | MmsStaticSubscriber.ts | 99 | GET 请求下载 |

**关键代码示例**:
```typescript
// entry/src/main/ets/service/SendMsgService.ets:71
sendMmsMessage(params, callback) {
    let httpRequest = http.createHttp();
    httpRequest.request(common.string.MMS_URL, {  // http://mmsc.monternet.com
        method: http.RequestMethod.POST,
        header: {
            'Content-Type': 'application/vnd.wap.mms-message'
        },
        extraData: JSON.stringify(params),
        readTimeout: 60000,
        connectTimeout: 60000
    }, (err, data) => { ... });
}
```

---

## UI/交互 API

### @ohos.mediaquery

**用途**: 响应式布局媒体查询

**调用位置**:

| 方法 | 文件 | 行号 | 用途 |
|------|------|------|------|
| `matchMediaSync()` | index.ets | 64 | 监听屏幕尺寸变化 |

### @ohos.promptAction

**用途**: 提示框

**调用位置**:

| 方法 | 文件 | 用途 |
|------|------|------|
| `showToast()` | receiveController.ets | 显示提示 |

### @ohos.pasteboard

**用途**: 剪贴板操作

**调用位置**:

| 方法 | 文件 | 用途 |
|------|------|------|
| `getSystemPasteboard()` | Pasteboard.ets | 获取剪贴板 |
| `setData()` | Pasteboard.ets | 写入数据 |
| `getData()` | Pasteboard.ets | 读取数据 |

### @ohos.router / @system.router

**用途**: 页面路由导航

**调用位置**: 多个页面控制器文件

| 方法 | 用途 |
|------|------|
| `push()` | 页面跳转 |
| `back()` | 返回上一页 |

---

## 其他 API

### @ohos.deviceInfo

**用途**: 设备信息获取

**调用位置**:

| 方法 | 文件 | 用途 |
|------|------|------|
| `deviceType` | DeviceUtil.ets | 获取设备类型 |

### @ohos.hilog

**用途**: 日志输出

**调用位置**:

| 方法 | 文件 | 用途 |
|------|------|------|
| `info()` | HiLog.ets | 输出 info 日志 |
| `warn()` | HiLog.ets | 输出 warn 日志 |
| `error()` | HiLog.ets | 输出 error 日志 |

---

## API 使用统计

| 模块 | 调用次数 | 风险等级 |
|------|----------|----------|
| @ohos.telephony.sms | 5 | 🔴 高 |
| @ohos.data.dataShare | 20+ | 🟡 中 |
| @ohos.net.http | 2 | 🔴 高 |
| @ohos.notificationManager | 4 | 🟢 低 |
| @ohos.commonEventManager | 4 | 🟢 低 |
| @ohos.worker | 3 | 🟢 低 |

---

## 相关链接

- [安全评审](05_SecurityReview.md) - 查看 API 安全风险
- [数据流](04_DataFlow.md) - 了解 API 调用流程
- [目录结构](02_DirectoryStructure.md) - 查看代码组织

---

*API 清单基于代码扫描生成*
