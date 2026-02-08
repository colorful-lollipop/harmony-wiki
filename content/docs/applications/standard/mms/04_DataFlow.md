# 04. 数据流

## 目的与适用范围

本文档描述 MMS 应用中的核心数据流转过程，包括短信收发、数据存储和通知流程。

**适用读者**: 开发工程师、架构师  
**阅读时间**: 约 25 分钟

---

## 数据流概览

```mermaid
flowchart TB
    subgraph 输入源
        UI[用户输入]
        Network[网络接收]
        System[系统事件]
    end

    subgraph 处理层
        Service[Service 层]
        Model[Model 层]
        Worker[Worker 线程]
    end

    subgraph 存储层
        DataShare[DataShare API]
        Preferences[Preferences]
        SMS_DB[(短信数据库)]
        Contact_DB[(联系人数据库)]
    end

    subgraph 输出
        Notification[通知]
        UI_Update[UI 更新]
        Network_Out[网络发送]
    end

    UI --> Service
    Network --> Service
    System --> Service
    Service --> Model
    Model --> Worker
    Worker --> DataShare
    DataShare --> SMS_DB
    DataShare --> Contact_DB
    Model --> Preferences
    Service --> Notification
    Service --> UI_Update
    Service --> Network_Out
```

---

## 短信发送流程

### 流程图

```mermaid
sequenceDiagram
    autonumber
    participant U as 用户
    participant P as conversation.ets
    participant CC as conversationController
    participant SMS as SendMsgService
    participant TEL as @ohos.telephony.sms
    participant CS as ConversationService
    participant CM as ConversationModel
    participant DS as DataShare

    U->>P: 1. 输入短信内容
    U->>P: 2. 点击发送按钮
    P->>CC: 3. 调用 sendMsg()
    CC->>SMS: 4. sendMessage(params)
    
    activate SMS
    SMS->>TEL: 5. sms.sendMessage()
    TEL-->>SMS: 6. sendCallback(结果)
    SMS->>SMS: 7. dealSendResult()
    SMS-->>CC: 8. callback(发送状态)
    deactivate SMS
    
    CC->>CS: 9. insertSessionAndDetail()
    activate CS
    CS->>CS: 10. dealSendResults()
    CS->>CS: 11. querySessionByTelephone()
    
    alt 会话不存在
        CS->>CS: 12a. dealNoExistSession()
        CS->>CS: 13a. insertSession()
    else 会话存在
        CS->>CS: 12b. dealExistSession()
        CS->>CS: 13b. updateSessionByCondition()
    end
    
    CS->>CS: 14. dealInsertMessageDetail()
    CS->>CM: 15. insertSmsMmsInfo()
    activate CM
    CM->>DS: 16. createDataShareHelper()
    CM->>DS: 17. insert(uri, valueBucket)
    DS-->>CM: 18. 插入结果
    CM-->>CS: 19. callback(结果)
    deactivate CM
    
    CS-->>CC: 20. callback(完成)
    deactivate CS
    
    CC->>P: 21. 更新 UI 状态
    P-->>U: 22. 显示发送结果
```

### 关键数据转换

```
用户输入
    ↓
{
    slotId: 0,                    // SIM 卡槽
    destinationHost: "13800138000", // 目标号码
    content: "短信内容"            // 短信内容
}
    ↓ (SendMsgService.sendMessage)
telephony.sms.sendMessage 参数
    ↓
发送结果 (SendSmsResult)
    ↓
存储到数据库 (ConversationModel.insertSmsMmsInfo)
    ↓
{
    slot_id: 0,
    receiver_number: "13800138000",
    sender_number: "本机号码",
    msg_content: "短信内容",
    msg_state: 0/1/2,  // 成功/发送中/失败
    session_id: 123,   // 关联会话
    group_id: 456      // 群发分组
}
```

### 关键代码路径

1. **用户输入**: `entry/src/main/ets/pages/conversation/conversation.ets`
2. **发送调用**: `entry/src/main/ets/pages/conversation/conversationController.ets`
3. **短信发送**: `entry/src/main/ets/service/SendMsgService.ets:31`
4. **数据存储**: `entry/src/main/ets/service/ConversationService.ets:249`
5. **数据库插入**: `entry/src/main/ets/model/ConversationModel.ets:29`

---

## 短信接收流程

### 流程图

```mermaid
sequenceDiagram
    autonumber
    participant Sys as 系统服务
    participant SS as MmsStaticSubscriber
    participant SMS as @ohos.telephony.sms
    participant CS as ConversationService
    participant NS as NotificationService
    participant CEM as @ohos.commonEventManager
    participant UI as UI 层

    Sys->>SS: 1. 广播 SMS_RECEIVE_COMPLETED
    activate SS
    SS->>SS: 2. dealSmsReceiveData()
    
    loop 解析每个 PDU
        SS->>SMS: 3. createMessage(pdu, netType)
        SMS-->>SS: 4. ShortMessage
    end
    
    SS->>SS: 5. 拼接长短信内容
    SS->>SS: 6. insertMessageDetailBy()
    
    SS->>CS: 7. insertSessionAndDetail()
    activate CS
    CS->>CS: 8. 检查/创建会话
    CS->>CS: 9. 插入消息详情
    CS-->>SS: 10. callback(结果)
    deactivate CS
    
    SS->>SS: 11. sendNotification()
    SS->>NS: 12. sendNotify()
    activate NS
    NS->>NS: 13. buildNotificationRequest()
    NS->>NS: 14. buildWantAgentInfo()
    NS-->>SS: 15. 通知完成
    deactivate NS
    
    SS->>CEM: 16. publish(RECEIVE_TRANSMIT_EVENT)
    CEM->>UI: 17. 广播事件
    deactivate SS
    
    UI->>UI: 18. 刷新会话列表
```

### PDU 处理详解

```
系统广播 PDU (字符串数组)
    ↓
MmsStaticSubscriber.convertStrArray()
    ↓
十六进制字符串 → 数字数组
    ↓
telephony.sms.createMessage(pdu, netType)
    ↓
ShortMessage 对象
    {
        visibleRawAddress: "发送号码",
        visibleMessageBody: "短信内容片段"
    }
    ↓
拼接多条 PDU (长短信)
    ↓
完整的短信内容
```

### 关键代码路径

1. **接收入口**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:35`
2. **PDU 解析**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:165`
3. **数据存储**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:143`
4. **通知发送**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:212`
5. **事件广播**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:197`

---

## MMS 接收流程

### 流程图

```mermaid
sequenceDiagram
    autonumber
    participant Sys as 系统服务
    participant SS as MmsStaticSubscriber
    participant HTTP as @ohos.net.http
    participant CS as ConversationService
    participant NS as NotificationService

    Sys->>SS: 1. 广播 MMS_RECEIVE_COMPLETED
    SS->>SS: 2. dealMmsReceiveData()
    SS->>SS: 3. JSON.parse(data)
    
    SS->>SS: 4. saveAttachment()
    loop 下载每个附件
        SS->>HTTP: 5. http.request(MMS_URL)
        HTTP-->>SS: 6. 下载结果
    end
    
    SS->>SS: 7. getMmsContent()
    SS->>SS: 8. getNotificationContent()
    
    SS->>CS: 9. insertSessionAndDetail()
    CS-->>SS: 10. 存储完成
    
    SS->>NS: 11. sendNotification()
    NS-->>SS: 12. 通知完成
```

### MMS 数据处理

```javascript
// 接收的 MMS 数据格式
{
    telephone: "发送号码",
    content: "主题内容",
    mmsSource: [
        {
            msgType: 0/1/2/3,  // 主题/图片/视频/音频
            content: "内容",
            msgUriPath: "附件 URL"
        }
    ],
    slotId: 0
}
```

### 关键代码路径

1. **MMS 接收**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:78`
2. **附件下载**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:95`
3. **内容提取**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:115`

---

## 数据库存储流程

### 表结构关系

```mermaid
erDiagram
    SESSION ||--o{ SMS_MMS_INFO : contains
    
    SESSION {
        int id PK
        string telephone
        string content
        int contacts_num
        int sms_type
        int unread_count
        int sending_status
        int has_draft
        bigint time
        int message_count
        int has_mms
        int has_attachment
    }
    
    SMS_MMS_INFO {
        int msgId PK
        int session_id FK
        int slot_id
        string receiver_number
        string sender_number
        bigint start_time
        bigint end_time
        int msg_type
        int sms_type
        string msg_title
        string msg_content
        int msg_state
        string operator_service_number
        int msg_code
        int is_lock
        int is_read
        int is_collect
        int session_type
        int retry_number
        int is_subsection
        int group_id
        int is_sender
        int is_send_report
    }
```

### 存储流程

```
业务数据
    ↓
Service 层处理
    ↓
构建 valueBucket 对象
    ↓
Model 层封装
    ↓
DataShareHelper.createDataShareHelper()
    ↓
dataShare.insert/update/delete/query()
    ↓
DataShare 服务
    ↓
短信数据库 (com.ohos.smsmmsability)
```

### 关键代码示例

```typescript
// 插入会话数据 (ConversationListModel)
async insertSession(valueBucket, callback, context) {
    let dataHelper = await dataShare.createDataShareHelper(
        context, 
        'datashare:///com.ohos.smsmmsability'
    );
    let uri = 'datashare:///com.ohos.smsmmsability/sms_mms/session';
    dataHelper.insert(uri, valueBucket)
        .then(res => callback(success))
        .catch(error => callback(failure));
}

// 插入短信详情 (ConversationModel)
async insertSmsMmsInfo(valueBucket, callback, context) {
    let dataHelper = await dataShare.createDataShareHelper(
        context,
        'datashare:///com.ohos.smsmmsability'
    );
    let uri = 'datashare:///com.ohos.smsmmsability/sms_mms/sms_mms_info';
    dataHelper.insert(uri, valueBucket)
        .then(res => callback(success))
        .catch(error => callback(failure));
}
```

---

## Worker 数据流

### Worker 架构

```mermaid
graph TB
    subgraph 主线程
        UI[UI 组件]
        Controller[Controller]
        Service[Service]
    end

    subgraph Worker线程
        Port[workerPort]
        Task[DataWorkerTask]
        Models[Model 实例]
    end

    subgraph 数据存储
        DB[(数据库)]
    end

    UI --> Controller
    Controller --> Service
    Service -->|1. sendRequest| Port
    Port -->|2. postMessage| Task
    Task -->|3. 调用| Models
    Models -->|4. DataShare| DB
    DB -->|5. 返回数据| Models
    Models -->|6. callback| Task
    Task -->|7. postMessage| Port
    Port -->|8. 回调| Service
    Service -->|9. 更新| UI
```

### Worker 方法映射

```typescript
// DataWorkerTask.runInWorker() 方法分发
switch (request) {
    // 联系人相关
    case 'queryContactDataByCondition':
        this.mContactsModel.queryContactDataByCondition(...)
        break;
    
    // 短信详情相关
    case 'insertSmsMmsInfo':
        this.mConversationModel.insertSmsMmsInfo(...)
        break;
    case 'deleteSmsMmsInfoByCondition':
        this.mConversationModel.deleteSmsMmsInfoByCondition(...)
        break;
    case 'updateSmsMmsInfoByCondition':
        this.mConversationModel.updateSmsMmsInfoByCondition(...)
        break;
    case 'querySmsMmsInfoByCondition':
        this.mConversationModel.querySmsMmsInfoByCondition(...)
        break;
    
    // 会话列表相关
    case 'insertSession':
        this.mConversationListModel.insertSession(...)
        break;
    case 'updateSessionByCondition':
        this.mConversationListModel.updateSessionByCondition(...)
        break;
    // ...
}
```

---

## 通知数据流

### 通知发送流程

```mermaid
sequenceDiagram
    participant Caller as 调用方
    participant NS as NotificationService
    participant WA as @ohos.wantAgent
    participant NM as @ohos.notificationManager

    Caller->>NS: sendNotify(actionData)
    NS->>NS: buildNotificationRequest()
    Note over NS: 构建通知内容
    
    NS->>NS: buildWantAgentInfo()
    Note over NS: 构建点击跳转参数
    
    NS->>WA: getWantAgent(wantAgentInfo)
    WA-->>NS: WantAgent 对象
    
    NS->>NM: Notification.publish(request)
    NM-->>NS: 发布结果
    NS-->>Caller: 完成
```

### 通知数据结构

```typescript
// 通知请求结构
{
    id: msgId,                    // 通知 ID
    label: 'notification_' + msgId, // 标签
    content: {
        contentType: Notification.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
        normal: {
            title: '联系人名称/号码',
            text: '短信内容...'
        }
    },
    wantAgent: WantAgent,         // 点击跳转
    slotType: Notification.SlotType.OTHER_TYPES,
    deliveryTime: timestamp,
    groupName: 'MMS'
}

// WantAgent 信息
{
    wants: [{
        bundleName: 'com.ohos.mms',
        abilityName: 'com.ohos.mms.MainAbility',
        action: 'mms.event.notification',
        parameters: {
            pageFlag: 'conversation',
            contactObjects: [...]
        }
    }],
    operationType: WantAgent.OperationType.START_ABILITY
}
```

---

## 事件数据流

### 应用内事件 (Emitter)

```mermaid
graph LR
    A[CardModel] -->|SIM_STATE_CHANGE_EVENT| B[SimCardService]
    A -->|SLOTID_CHANGE_EVENT| C[ConversationListController]
    D[ConversationController] -->|刷新事件| E[conversation.ets]
```

### 系统事件 (CommonEvent)

```
系统 SMS_RECEIVE_COMPLETED
    ↓
MmsStaticSubscriber.onReceiveEvent()
    ↓
处理短信/MMS
    ↓
publish(RECEIVE_TRANSMIT_EVENT)
    ↓
UI 层监听并刷新
```

---

## 相关链接

- [架构设计](01_Architecture.md) - 了解整体架构
- [系统 API](03_SystemAPIs.md) - 查看 API 详情
- [附录 A: 调用链](appendix/Callgraphs.md) - 完整调用路径

---

*数据流基于代码: entry/src/main/ets/*
