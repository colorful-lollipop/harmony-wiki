# 附录 A: 调用链

## 目的与适用范围

本文档提供 MMS 应用中关键功能的完整调用链，帮助开发者理解代码执行路径。

**适用读者**: 开发工程师、安全审计人员  
**阅读时间**: 约 20 分钟

---

## 调用链图例

```
┌─────────────────────────────────────────────────────────────┐
│ 图形说明                                                     │
├─────────────────────────────────────────────────────────────┤
│  [FileName]      - 源文件                                    │
│  methodName()    - 函数调用                                  │
│  -->             - 调用方向                                  │
│  (line: XX)      - 代码行号                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. 短信发送调用链

### 完整调用链

```
[用户操作]
    ↓
[pages/conversation/conversation.ets]
    onSendMsg() (用户点击发送)
    ↓
[pages/conversation/conversationController.ets]
    sendMsg() (line: ~200)
    ↓
[service/SendMsgService.ets]
    sendMessage(params, callback) (line: 31)
    ↓
[@ohos.telephony.sms]
    sms.sendMessage({...}) (line: 33)
    ↓
[sendCallback] ← 系统回调
    ↓
[service/SendMsgService.ets]
    dealSendResult(value) (line: 61)
    ↓
[callback] → [pages/conversation/conversationController.ets]
    ↓
[service/ConversationService.ets]
    insertSessionAndDetail(actionData, callback, context) (line: 249)
    ↓
[service/ConversationService.ets]
    dealSendResults(sendResults) (line: 441)
    ↓
[service/ConversationListService.ets]
    querySessionByTelephone(telephone, callback, context) (line: ~100)
    ↓
[model/ConversationListModel.ets]
    querySessionByCondition(actionData, callback, context) (line: ~80)
    ↓
[@ohos.data.dataShare]
    createDataShareHelper(context, uri) (line: ~82)
    dataHelper.query(uri, condition, columns) (line: ~85)
    ↓
[callback] ← 数据库返回
    ↓
[service/ConversationService.ets]
    dealNoExistSession() / dealExistSession() (line: 268 / 305)
    ↓
[service/ConversationService.ets]
    dealInsertMessageDetail() (line: 340)
    ↓
[service/ConversationService.ets]
    insertMessageDetailByMaxGroupId() (line: 355)
    ↓
[model/ConversationModel.ets] (通过 Worker)
    insertSmsMmsInfo(valueBucket, callback, context) (line: 29)
    ↓
[@ohos.data.dataShare]
    createDataShareHelper() → dataHelper.insert() (line: 30-34)
    ↓
[短信数据库] com.ohos.smsmmsability
    ↓
[callback 返回]
    ↓
[UI 更新]
```

### 简化视图

```mermaid
graph LR
    User[用户] --> Conversation[conversation.ets]
    Conversation --> Controller[conversationController.ets]
    Controller --> SendService[SendMsgService.ets]
    SendService --> Telephony[@ohos.telephony.sms]
    Telephony -.回调.-> SendService
    SendService --> ConvService[ConversationService.ets]
    ConvService --> ConvListService[ConversationListService.ets]
    ConvListService --> ConvListModel[ConversationListModel.ets]
    ConvListModel --> DataShare[@ohos.data.dataShare]
    DataShare -.返回.-> ConvService
    ConvService --> ConvModel[ConversationModel.ets]
    ConvModel --> DataShare2[@ohos.data.dataShare]
    DataShare2 -.返回.-> UI[UI 更新]
```

---

## 2. 短信接收调用链

### 完整调用链

```
[系统服务]
    ↓
广播: usual.event.SMS_RECEIVE_COMPLETED
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    onReceiveEvent(data) (line: 35)
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    dealSmsReceiveData(data, context) (line: 44)
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    convertStrArray(pdu) (line: 49, 165) - PDU 解析
    ↓
[@ohos.telephony.sms]
    createMessage(pduArray, netType) (line: 49)
    ↓
[Promise.all] ← 等待所有 PDU 解析
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    拼接短信内容 (line: 57-60)
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    insertMessageDetailBy(param, callback, context) (line: 143)
    ↓
[service/ConversationService.ets]
    insertSessionAndDetail(actionData, callback, context) (line: 249)
    ↓
... (同发送流程的数据存储链)
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    callback ← 存储完成
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    sendNotification(telephone, msgId, content, context) (line: 212)
    ↓
[service/ContactsService.ets]
    queryContactDataByCondition(condition, callback, context) (line: ~50)
    ↓
[model/ContactsModel.ets]
    queryContactDataByCondition() (line: ~40)
    ↓
[@ohos.data.dataShare]
    查询联系人数据库
    ↓
[callback] ← 联系人信息
    ↓
[service/NotificationService.ets]
    sendNotify(actionData) (line: 46)
    ↓
[service/NotificationService.ets]
    buildNotificationRequest(actionData) (line: 107)
    buildWantAgentInfo(actionData) (line: 77)
    ↓
[@ohos.app.ability.wantAgent]
    getWantAgent(agentInfo) (line: 67)
    ↓
[@ohos.notificationManager]
    Notification.publish(notificationRequest) (line: 55)
    ↓
[系统通知服务]
    ↓
[StaticSubscriber/MmsStaticSubscriber.ts]
    publishData(telephone, content) (line: 197)
    ↓
[@ohos.commonEventManager]
    publish(RECEIVE_TRANSMIT_EVENT) (line: 203)
    ↓
[UI 层订阅者]
    刷新会话列表
```

### 简化视图

```mermaid
graph TD
    System[系统服务] -->|广播| Subscriber[MmsStaticSubscriber.ts]
    Subscriber -->|解析 PDU| Telephony[@ohos.telephony.sms]
    Telephony -.返回.-> Subscriber
    Subscriber -->|存储| ConvService[ConversationService.ets]
    ConvService -->|查询联系人| ContactService[ContactsService.ets]
    ContactService --> ContactModel[ContactsModel.ets]
    ContactModel --> DataShare[@ohos.data.dataShare]
    DataShare -.返回.-> Subscriber
    Subscriber -->|发送通知| NotifService[NotificationService.ets]
    NotifService --> WantAgent[@ohos.wantAgent]
    WantAgent -.返回.-> NotifService
    NotifService --> NotifMgr[@ohos.notificationManager]
    Subscriber -->|广播事件| CommonEvent[@ohos.commonEventManager]
    CommonEvent --> UI[UI 刷新]
```

---

## 3. 会话列表查询调用链

```
[pages/index.ets]
    aboutToAppear() (line: 61)
    ↓
[pages/conversationlist/conversationListController.ets]
    onInit() (line: ~50)
    ↓
[service/ConversationListService.ets]
    querySessionByCondition(actionData, callback, context) (line: ~100)
    ↓
[service/ConversationService.ets]
    querySmsMmsInfoByCondition(actionData, callback, context) (line: 89)
    ↓
[model/ConversationModel.ets] (通过 Worker)
    querySmsMmsInfoByCondition(actionData, callback, context) (line: 80)
    ↓
[workers/DataWorkerWrapper.ets]
    runInWorker(request, callBack, param) (line: 68)
    ↓
[model/ConversationModel.ets]
    querySmsMmsInfoByCondition() (Worker 线程执行)
    ↓
[@ohos.data.dataShare]
    createDataShareHelper() → dataHelper.query() (line: 81-85)
    ↓
[短信数据库]
    ↓
[ResultSet] ← 查询结果
    ↓
[model/ConversationModel.ets]
    buildSmsMmsInfoResult(resultSet) (line: 236)
    ↓
[callback] → [workers/DataWorkerWrapper.ets]
    postMessage 回主线程
    ↓
[service/ConversationService.ets]
    convertConversationList(mmsList) (line: 159)
    ↓
[service/ContactsService.ets]
    queryContactDataByCondition() (如果需要联系人信息)
    ↓
[callback] → [pages/conversationlist/conversationListController.ets]
    ↓
[pages/index.ets]
    更新 UI 列表
```

---

## 4. 通知发送调用链

```
[调用方: MmsStaticSubscriber 或 ConversationController]
    ↓
[service/NotificationService.ets]
    sendNotify(actionData) (line: 46)
    ↓
[service/NotificationService.ets]
    buildNotificationRequest(actionData) (line: 107)
    ↓
返回: notificationRequest = {
    content: { title, text },
    slotType: Notification.SlotType.OTHER_TYPES,
    groupName: 'MMS'
}
    ↓
[service/NotificationService.ets]
    buildWantAgentInfo(actionData) (line: 77)
    ↓
返回: wantAgentInfo = {
    wants: [{ bundleName, abilityName, parameters }],
    operationType: WantAgent.OperationType.START_ABILITY
}
    ↓
[service/NotificationService.ets]
    getWantAgent(agentInfo, callback) (line: 66)
    ↓
[@ohos.app.ability.wantAgent]
    getWantAgent(agentInfo).then(...) (line: 67)
    ↓
[callback] ← WantAgent 对象
    ↓
[service/NotificationService.ets]
    notificationRequest.wantAgent = data (line: 52)
    Notification.publish(notificationRequest) (line: 55)
    ↓
[@ohos.notificationManager]
    系统通知服务
    ↓
[系统通知栏]
    显示通知
```

---

## 5. SIM 卡状态监听调用链

```
[MainAbility.ts]
    onForeground() (line: 66)
    ↓
[service/SimCardService.ets]
    init() (line: ~30)
    ↓
[model/CardModel.ets]
    init() (line: 45)
    ↓
[model/CardModel.ets]
    setMaxSimCountToMap() (line: 48)
    [@ohos.telephony.sim] getMaxSimCount() (line: 74)
    ↓
[model/CardModel.ets]
    getSimState() (line: 49)
    ↓
循环每个卡槽:
    [@ohos.telephony.sim] getSimState(slotId, callback) (line: 79)
    ↓
[callback] ← SIM 状态
    ↓
[model/CardModel.ets]
    notifySimStateChange(slotId, simState) (line: 105)
    ↓
[model/CardModel.ets]
    setSimCardReadyFlagToMap() (line: 126)
    setDefaultSlotToMap() (line: 136)
    setSpnToMap() (line: 156)
    setSimTelephoneNumberToMap() (line: 210)
    ↓
[@ohos.telephony.sim]
    getSimSpn(), getSimTelephoneNumber(), ...
    ↓
[@ohos.events.emitter]
    emitter.emit(SIM_STATE_CHANGE_EVENT) (line: 118)
    ↓
[订阅者: SimCardService / UI Controllers]
    更新状态显示
```

---

## 6. Worker 线程数据操作调用链

```
[Service 层: ConversationService / ConversationListService / ContactsService]
    ↓
检查: if (globalThis.DataWorker != null)
    ↓
[workers/DataWorkerWrapper.ets]
    sendRequest(methodName, params, callback) (line: ~50)
    ↓
[workers/base/WorkerWrapper.ts]
    postMessage({request, param}) (line: ~80)
    ↓
[Worker 线程]
    onmessage 触发
    ↓
[workers/DataWorkerWrapper.ets]
    DataWorkerTask.runInWorker(request, callBack, param) (line: 68)
    ↓
Switch(methodName):
    Case "insertSmsMmsInfo":
        → [model/ConversationModel.ets] insertSmsMmsInfo()
    Case "querySmsMmsInfoByCondition":
        → [model/ConversationModel.ets] querySmsMmsInfoByCondition()
    Case "insertSession":
        → [model/ConversationListModel.ets] insertSession()
    Case "queryContactDataByCondition":
        → [model/ContactsModel.ets] queryContactDataByCondition()
    ...
    ↓
[Model 层]
    执行 DataShare 操作
    ↓
[callback]
    ↓
[workers/DataWorkerWrapper.ets]
    workerPort.postMessage({result}) (line: ~120)
    ↓
[主线程]
    onmessage 触发
    ↓
[workers/base/WorkerWrapper.ts]
    分发回调 (line: ~100)
    ↓
[Service 层 callback]
    处理结果
    ↓
[UI 更新]
```

---

## 关键文件索引

| 功能 | 主文件 | 辅助文件 |
|------|--------|----------|
| 短信发送 | SendMsgService.ets | ConversationService.ets, ConversationModel.ets |
| 短信接收 | MmsStaticSubscriber.ts | NotificationService.ets, ConversationService.ets |
| 会话列表 | ConversationListService.ets | ConversationListModel.ets, conversationListController.ets |
| 会话详情 | ConversationService.ets | ConversationModel.ets, conversationController.ets |
| 通知 | NotificationService.ets | MmsStaticSubscriber.ts |
| 联系人 | ContactsService.ets | ContactsModel.ets |
| SIM 卡 | SimCardService.ets | CardModel.ets |
| 数据操作 | DataWorkerWrapper.ets | WorkerWrapper.ts, WorkerTask.ts |

---

## 相关链接

- [架构设计](../01_Architecture.md) - 整体架构说明
- [数据流](../04_DataFlow.md) - 数据流转详细说明
- [系统 API](../03_SystemAPIs.md) - API 使用详情

---

*调用链基于代码: entry/src/main/ets/*
