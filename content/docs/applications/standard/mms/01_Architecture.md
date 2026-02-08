# 01. 架构设计

## 目的与适用范围

本文档描述 MMS 应用的技术架构，包括模块划分、组件关系、数据流和线程模型。

**适用读者**: 开发工程师、架构师  
**阅读时间**: 约 30 分钟

---

## 整体架构

### 分层架构

```mermaid
graph TB
    subgraph UI层
        P1[pages/index]
        P2[pages/conversation]
        P3[pages/settings]
        V1[views/MmsListItem]
        V2[views/MmsDialogs]
    end

    subgraph 业务逻辑层
        S1[SendMsgService]
        S2[ConversationService]
        S3[ConversationListService]
        S4[NotificationService]
        S5[ContactsService]
    end

    subgraph 数据模型层
        M1[ConversationModel]
        M2[ConversationListModel]
        M3[ContactsModel]
        M4[CardModel]
    end

    subgraph 系统服务层
        API1[@ohos.telephony.sms]
        API2[@ohos.data.dataShare]
        API3[@ohos.notificationManager]
        API4[@ohos.commonEventManager]
    end

    UI层 --> 业务逻辑层
    业务逻辑层 --> 数据模型层
    数据模型层 --> 系统服务层
```

### 模块职责

| 模块 | 职责 | 主要文件 |
|------|------|----------|
| **Application** | AbilityStage 生命周期 | MyAbilityStage.ts |
| **MainAbility** | UIAbility 生命周期管理 | MainAbility.ts |
| **StaticSubscriber** | 短信接收事件处理 | MmsStaticSubscriber.ts |
| **pages** | UI 页面实现 | 8 个页面目录 |
| **views** | 可复用 UI 组件 | 8 个视图组件 |
| **service** | 业务逻辑封装 | 10 个服务类 |
| **model** | 数据访问层 | 8 个模型类 |
| **data** | 数据类型定义 | 4 个数据文件 |
| **utils** | 工具类 | 8 个工具类 |
| **workers** | 后台线程处理 | Worker 相关 4 个文件 |

---

## 组件关系

### 核心组件交互图

```mermaid
sequenceDiagram
    participant User
    participant Index as pages/index
    participant CLC as ConversationListController
    participant CS as ConversationService
    participant CM as ConversationModel
    participant DS as DataShare

    User->>Index: 打开应用
    Index->>CLC: onInit()
    CLC->>CS: querySessionByCondition()
    CS->>CM: querySessionByCondition()
    CM->>DS: dataShare.query()
    DS-->>CM: ResultSet
    CM-->>CS: 会话列表数据
    CS-->>CLC: 处理后的数据
    CLC-->>Index: 更新 UI

    User->>Index: 点击会话
    Index->>CLC: 导航到 conversation
    CLC->>CS: queryMessageDetail()
    CS->>CM: querySmsMmsInfoByCondition()
    CM->>DS: dataShare.query()
    DS-->>CM: 短信详情
    CM-->>CS: 详情数据
    CS-->>Index: 渲染对话界面
```

### 服务层依赖关系

```mermaid
graph LR
    subgraph 服务层
        direction TB
        SendMsg[SendMsgService]
        ConServ[ConversationService]
        ConListServ[ConversationListService]
        NotifServ[NotificationService]
        ContServ[ContactsService]
        SimServ[SimCardService]
    end

    SendMsg --> TelephonySMS[@ohos.telephony.sms]
    ConServ --> ConListServ
    ConServ --> ContServ
    ConListServ --> ConListModel[ConversationListModel]
    NotifServ --> Notification[@ohos.notificationManager]
    ContServ --> ContactsModel[ContactsModel]
    SimServ --> Emitter[@ohos.events.emitter]
```

---

## 线程模型

### Worker 线程架构

MMS 应用使用 Worker 线程处理耗时数据库操作，避免阻塞 UI 线程。

```mermaid
graph TB
    subgraph 主线程
        UI[UI 组件]
        Controller[页面 Controller]
        Service[Service 层]
    end

    subgraph Worker线程
        DataWorker[DataWorkerWrapper]
        DataTask[DataWorkerTask]
        Models[Model 实例]
    end

    subgraph 系统服务
        DataShare[@ohos.data.dataShare]
    end

    UI --> Controller
    Controller --> Service
    Service -->|sendRequest| DataWorker
    DataWorker --> DataTask
    DataTask --> Models
    Models --> DataShare
```

### Worker 方法映射

定义在 `commonData.ets` 中的 Worker 方法:

| 方法名 | 所属模型 | 操作类型 |
|--------|----------|----------|
| queryContactDataByCondition | ContactsModel | 查询 |
| insertSmsMmsInfo | ConversationModel | 插入 |
| deleteSmsMmsInfoByCondition | ConversationModel | 删除 |
| updateSmsMmsInfoByCondition | ConversationModel | 更新 |
| querySmsMmsInfoByCondition | ConversationModel | 查询 |
| insertSession | ConversationListModel | 插入 |
| deleteSessionByCondition | ConversationListModel | 删除 |
| updateSessionByCondition | ConversationListModel | 更新 |

### 线程切换流程

```typescript
// entry/src/main/ets/service/ConversationService.ets:40
public insertSmsMmsInfo(valueBucket, callback, context): void {
    if (globalThis.DataWorker != null) {
        // 切换到 Worker 线程
        globalThis.DataWorker.sendRequest(
            common.RUN_IN_WORKER_METHOD.insertSmsMmsInfo,
            { valueBucket, context },
            res => { callback(res); }  // 回调回主线程
        );
    } else {
        // 主线程直接执行
        this.conversationModel.insertSmsMmsInfo(valueBucket, callback, mmsContext);
    }
}
```

---

## 关键时序

### 1. 短信发送时序

```mermaid
sequenceDiagram
    participant User
    participant Page as conversation.ets
    participant SendSvc as SendMsgService
    participant Telephony as @ohos.telephony.sms
    participant ConvSvc as ConversationService
    participant Model as ConversationModel

    User->>Page: 输入内容并发送
    Page->>SendSvc: sendMessage(params)
    SendSvc->>Telephony: sms.sendMessage()
    Telephony-->>SendSvc: sendCallback
    SendSvc->>ConvSvc: insertSessionAndDetail()
    ConvSvc->>Model: insertSmsMmsInfo()
    Model-->>ConvSvc: 插入结果
    ConvSvc-->>SendSvc: 完成
    SendSvc-->>Page: 更新发送状态
```

### 2. 短信接收时序

```mermaid
sequenceDiagram
    participant System as 系统服务
    participant Subscriber as MmsStaticSubscriber
    participant Telephony as @ohos.telephony.sms
    participant ConvSvc as ConversationService
    participant NotifSvc as NotificationService
    participant UI as 页面

    System->>Subscriber: 广播 SMS_RECEIVE_COMPLETED
    Subscriber->>Telephony: createMessage(pdus)
    Telephony-->>Subscriber: ShortMessage
    Subscriber->>ConvSvc: insertSessionAndDetail()
    ConvSvc-->>Subscriber: 存储完成
    Subscriber->>NotifSvc: sendNotify()
    NotifSvc-->>Subscriber: 通知发送
    Subscriber->>UI: 广播 RECEIVE_TRANSMIT_EVENT
```

### 3. 通知发送时序

```mermaid
sequenceDiagram
    participant Caller as 调用方
    participant NotifSvc as NotificationService
    participant WantAgent as @ohos.app.ability.wantAgent
    participant NotifMgr as @ohos.notificationManager

    Caller->>NotifSvc: sendNotify(actionData)
    NotifSvc->>NotifSvc: buildNotificationRequest()
    NotifSvc->>NotifSvc: buildWantAgentInfo()
    NotifSvc->>WantAgent: getWantAgent()
    WantAgent-->>NotifSvc: WantAgent 对象
    NotifSvc->>NotifMgr: Notification.publish()
    NotifMgr-->>NotifSvc: 发布完成
```

---

## 数据存储架构

### 数据库关系

```mermaid
erDiagram
    SESSION ||--o{ SMS_MMS_INFO : contains
    
    SESSION {
        int id PK
        string telephone
        string content
        int unread_count
        int message_count
        int sending_status
        bigint time
    }
    
    SMS_MMS_INFO {
        int msgId PK
        int session_id FK
        int slot_id
        string sender_number
        string receiver_number
        string msg_content
        int msg_state
        int group_id
        bigint start_time
    }
```

### 数据访问模式

```
┌─────────────────────────────────────────┐
│           Service 层                     │
│   - 业务逻辑处理                         │
│   - 数据格式转换                         │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│           Model 层                       │
│   - DataShareHelper 创建                │
│   - Predicates 构建                     │
│   - CRUD 操作封装                        │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│        @ohos.data.dataShare             │
│   - 跨进程数据访问                        │
│   - URI 路由到对应 Ability               │
└─────────────────────────────────────────┘
```

---

## 事件机制

### 事件总线

使用 `@ohos.events.emitter` 实现模块间通信:

```mermaid
graph LR
    CardModel -->|SIM_STATE_CHANGE_EVENT| SimCardService
    CardModel -->|SLOTID_CHANGE_EVENT| ConversationListController
    MmsStaticSubscriber -->|RECEIVE_TRANSMIT_EVENT| ConversationController
```

### 关键事件定义

| 事件 ID | 名称 | 发送方 | 接收方 |
|---------|------|--------|--------|
| 1 | SIM_STATE_CHANGE_EVENT | CardModel | SimCardService |
| 2 | SLOTID_CHANGE_EVENT | CardModel | UI Controllers |
| usual.event.SMS_RECEIVE_COMPLETED | 短信接收 | 系统 | MmsStaticSubscriber |
| usual.event.RECEIVE_COMPLETED_TRANSMIT | 接收转发 | MmsStaticSubscriber | UI 层 |

---

## 相关链接

- [目录结构](02_DirectoryStructure.md) - 查看代码组织
- [数据流](04_DataFlow.md) - 详细数据流转
- [附录A: 调用链](appendix/Callgraphs.md) - 完整调用关系

---

*架构图基于代码: entry/src/main/ets/*
