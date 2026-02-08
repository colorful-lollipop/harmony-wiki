# 关键调用链

## 1. N-API 调用链

### 1.1 ContinuationManager 注册调用链

```
JS: continuation.register(options)
    |
    v
N-API: JsContinuationManager::Register (continuation_manager/js_continuation_manager.cpp:1048)
    |
    +---> 参数解析: UnWrapContinuationExtraParams
    |         |---> JsToContinuationExtraParams
    |         |---> napi_get_value_string_utf8
    |         |---> napi_get_value_int32
    |
    +---> 权限检查: IsSystemApp / VerifyPermission
    |
    +---> 调用 Inner API:
    |     DistributedAbilityManagerClient::Register
    |               |
    |               v
    |     GetContinuationMgrService -> IDistributedAbilityManager::Register
    |
    +---> 结果返回: Promise resolve/reject
```

### 1.2 AbilityConnection Connect 调用链

```
JS: session.connect(deviceId)
    |
    v
N-API: JsAbilityConnectionManager::Connect (js_ability_connection_manager.cpp:2209)
    |
    +---> 参数解析: JSToConnectOption
    |         |---> JsToPeerInfo
    |         |---> JsToServiceName
    |         |---> napi_get_value_string_utf8
    |
    +---> 权限检查: IsSystemApp
    |
    +---> 创建异步工作:
    |     napi_create_promise
    |     napi_create_async_work
    |     ExecuteConnect
    |               |
    |               v
    |     ConnectInner (threadsafefunction)
    |               |
    |               v
    |     AbilityConnectionManager::Connect
    |               |
    |               v
    |     ChannelManager::CreateSession
    |               |
    |               v
    |     SoftbusAdapter::Connect
    |
    +---> 结果返回: Promise resolve/reject
```

### 1.3 SendMessage 调用链

```
JS: session.sendMessage({type, data})
    |
    v
N-API: JsAbilityConnectionManager::SendMessage (js_ability_connection_manager.cpp:2220)
    |
    +---> 参数解析:
    |     napi_get_value_string_utf8
    |     message.length check
    |
    +---> 创建异步工作:
    |     napi_create_promise
    |     napi_create_async_work
    |     ExecuteSendMessage
    |               |
    |               v
    |     ChannelManager::SendData
    |               |
    |               v
    |     SoftbusAdapter::Send
    |
    +---> 结果返回: Promise resolve/reject
```

## 2. IPC 调用链

### 2.1 分布式调度 IPC 调用

```
DistributedSchedProxy (distributed_sched_proxy.cpp)
    |
    v
MessageParcel::WriteRemoteObject(target)
    |
    v
IPCSkeleton::SendRequest(code, data, reply, option)
    |
    v
Binder Driver (Kernel)
    |
    v
DistributedSchedStub (distributed_sched_stub.cpp)
    |
    v
DistributedSchedService::OnRemoteRequest
    |
    +---> 根据 code 分发:
    |     |---> START_ABILITY: StartAbilityInner
    |     |---> CONNECT_ABILITY: ConnectAbilityInner
    |     |---> CONTINUE_ABILITY: ContinueAbilityInner
    |     |---> SEND_DATA: SendDataInner
```

### 2.2 SA 服务注册调用链

```
DistributedSchedService (distributed_sched_service.cpp)
    |
    v
SystemAbility::MakeAndRegisterAbility
    |
    v
SAMgr::RegisterSystemAbility
    |
    v
SA Profile 加载 (sa_profile/1401.json)
    |
    v
服务实例化并添加到 SA 列表
```

## 3. 软总线通信调用链

### 3.1 Softbus Session 创建

```
AbilityConnectionManager::Connect
    |
    v
ChannelManager::CreateSession
    |
    v
SoftbusAdapter::CreateSessionWrapper
    |
    v
ISoftbusSession::OpenSession
    |
    v
DSoftbus SDK
    |
    v
设备间握手
    |
    v
Session 建立回调
```

### 3.2 数据发送

```
SendData(msg)
    |
    v
DataSender::Send
    |
    v
SoftbusAdapter::SendBytes/SendMessage
    |
    v
ISoftbusSession::SendBytes
    |
    v
DSoftbus SDK
```

### 3.3 数据接收

```
DSoftbus SDK (回调)
    |
    v
IDataListener::OnDataReceived
    |
    v
DataReceiver::OnDataReceived
    |
    v
ChannelManager::OnSessionDataReceived
    |
    v
N-API Callback (ThreadSafeFunction)
    |
    v
JS Handler
```

## 4. 接续管理调用链

### 4.1 设备选择流程

```
JS: continuation.on('deviceSelect', callback)
    |
    v
N-API: JsContinuationManager::RegisterDeviceSelectionCallback
    |
    v
DistributedAbilityManagerClient::RegisterDeviceSelectionCallback
    |
    v
DeviceSelectionNotifierStub
    |
    v
设备选择 UI 启动
    |
    v
用户选择设备
    |
    v
UpdateConnectStatus(deviceId, status)
    |
    v
回调通知 JS 层
```

### 4.2 接续执行流程

```
JS: continuation.startContinuation(options)
    |
    v
N-API: JsContinuationManager::StartContinuationDeviceManager
    |
    v
DistributedAbilityManagerClient::StartDeviceManager
    |
    v
DistributedSchedService::ContinueAbility
    |
    v
远端设备: AbilityScheduler::ContinueAbility
    |
    v
结果返回
```

## 5. 音视频流调用链

### 5.1 Stream 创建

```
JS: session.createStream(options)
    |
    v
N-API: JsAbilityConnectionManager::CreateStream
    |
    v
AVSenderEngine::CreateStream
    |
    v
SurfaceEncoderFilter::Init
    |
    v
AVReceiverEngine::Prepare
    |
    v
ChannelManager::CreateDataStream
    |
    v
Stream 建立
```

### 5.2 Surface 数据流

```
Surface (Producer)
    |
    v
SurfaceEncoderFilter::ProcessInput
    |
    v
编码器编码
    |
    v
ChannelManager::SendStreamData
    |
    v
SoftbusAdapter::SendStream
    |
    v
远端解码显示
```

## 6. 关键时序图

### 6.1 远程启动时序

```
participant JS as "JS 应用"
participant NAPI as "N-API 层"
participant Proxy as "分布式调度代理"
participant Service as "分布式调度服务"
participant Remote as "远端设备"

JS->>NAPI: startAbility(want)
NAPI->>Proxy: StartAbility(want)
Proxy->>Service: IPC(START_ABILITY)
Service->>Remote: IPC(调度请求)
Remote-->>Service: 结果
Service-->>Proxy: IPC回复
Proxy-->>NAPI: 结果
NAPI-->>JS: Promise resolve
```

### 6.2 能力连接时序

```
participant JS as "JS 应用"
participant NAPI as "N-API 层"
participant ACM as "能力连接管理器"
participant CM as "通道管理器"
participant Softbus as "软总线"
participant Remote as "远端设备"

JS->>NAPI: connect(deviceId)
NAPI->>ACM: Connect(options)
ACM->>CM: CreateSession()
CM->>Softbus: OpenSession()
Softbus->>Remote: 设备连接
Remote-->>Softbus: 连接确认
Softbus-->>CM: Session建立
CM-->>ACM: SessionId
ACM-->>NAPI: 连接成功
NAPI-->>JS: Promise resolve
```

## 7. 错误传播路径

```
Native Error
    |
    v
Service Layer (Error Code Mapping)
    |
    v
IPC Layer (ResultCode)
    |
    v
N-API Layer (ErrorCode -> JS Error)
    |
    v
JS Error (Promise reject)
```

## 8. 回调注册与触发

### 8.1 回调注册

```
JS: abilityConnectionManager.on('session', callback)
    |
    v
N-API: JsAbilityConnectionManager::RegisterAbilityConnectionSessionCallback
    |
    v
NativeReference 包装 JS Callback
    |
    v
napi_threadsafe_function_create
    |
    v
注册到 AbilityConnectionManager
```

### 8.2 回调触发

```
事件发生
    |
    v
AbilityConnectionManager::TriggerCallback
    |
    v
napi_threadsafe_function_call
    |
    v
JS Callback 执行
```

## 9. 资源生命周期

### 9.1 会话生命周期

```
CreateSession --> Session建立 --> 通信 --> Session关闭 --> 资源释放
     |                |            |           |          |
     v                v            v           v          v
  Alloc           Handshake    Transfer   Release    Cleanup
  SessionId
```

### 9.2 流生命周期

```
CreateStream --> Prepare --> Start --> 传输 --> Stop --> Destroy
     |            |          |         |        |        |
     v            v          v         v        v        v
  Alloc        Encoder     Encoder   Transfer  Release  Cleanup
  StreamId     Init        Start     Data     Buffer   All
```
