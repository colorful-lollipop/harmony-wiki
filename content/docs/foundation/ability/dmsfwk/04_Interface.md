# 04_Interface - 对外接口文档

## 接口概览

dmsfwk 提供三类对外接口：

| 接口类型 | 层级 | 使用场景 | 目标用户 |
|---------|------|---------|---------|
| **N-API** | JS/ArkTS | 应用开发 | 应用开发者 |
| **Taihe** | ArkTS (新) | 应用开发 | 应用开发者 |
| **InnerKits** | C++ | 系统服务 | 平台开发者 |
| **IPC** | 系统服务间 | SA 通信 | 框架开发者 |

---

## 1. N-API 接口

### 1.1 ContinuationManager 模块

**模块名**: `continuation.continuationManager`  
**库文件**: `libcontinuationmanager_napi.z.so`

#### 方法清单

| JS API | 参数 | 返回值 | C++ 入口 | 权限 |
|-------|------|-------|---------|------|
| `register(options, callback)` | options: object, callback: function | token: number | `JsContinuationManager::Register` | `DISTRIBUTED_DATASYNC` |
| `unregister(token, callback)` | token: number | void | `JsContinuationManager::Unregister` | 同注册者 |
| `on(type, token, callback)` | type: string, token: number | void | `RegisterDeviceSelectionCallback` | - |
| `off(type, token)` | type: string, token: number | void | `UnregisterDeviceSelectionCallback` | - |
| `updateConnectStatus(token, deviceId, status)` | deviceId: string, status: string | void | `UpdateConnectStatus` | - |
| `startDeviceManager(token, options, callback)` | options: object | deviceList: array | `StartDeviceManager` | - |

#### 参数校验

**options 对象结构**: `interfaces/kits/napi/continuation_manager/js_continuation_manager.cpp:134-146`
```cpp
if (!UnWrapContinuationExtraParams(env, argv[0], continuationExtraParams)) {
    HILOGE("Parse continuationExtraParams failed");
    errCode = ERR_NOT_OK;
}
```

**错误码处理**: `interfaces/kits/napi/include/napi_error_code.h`
```cpp
constexpr int32_t PARAMETER_CHECK_FAILED = 401;
```

### 1.2 AbilityConnectionManager 模块

**模块名**: `@ohos.distributedsched.abilityConnectionManager`  
**库文件**: `libabilityconnectionmanager_napi.z.so`

#### 方法清单

| JS API | 参数 | 同步/异步 | C++ 入口 |
|-------|------|----------|---------|
| `createAbilityConnectionSession(bundleName, moduleName, abilityName, options)` | bundleName: string, ... | 同步 | `CreateAbilityConnectionSession` |
| `destroyAbilityConnectionSession(sessionId)` | sessionId: number | 同步 | `DestroyAbilityConnectionSession` |
| `connect(sessionId, serverId, serverInfo)` | sessionId: number | 异步 | `Connect` |
| `disconnect(sessionId)` | sessionId: number | 异步 | `DisConnect` |
| `sendMessage(sessionId, data)` | data: string/ArrayBuffer | 异步 | `SendMessage` |
| `sendData(sessionId, data)` | data: ArrayBuffer | 异步 | `SendData` |
| `createStream(sessionId)` | sessionId: number | 同步 | `CreateStream` |
| `startStream(sessionId, surfaceId)` | surfaceId: string | 异步 | `StartStream` |

#### 关键校验点

**会话 ID 校验**: `services/dtbcollabmgr/src/channel_manager/channel_manager.cpp:1088-1089`
```cpp
CHECK_SOCKET_ID(socketId);
CHECK_CHANNEL_ID(socketId, channelId);
```

### 1.3 ContinuationStateManager 模块

**模块名**: `app.ability.continueManager`  
**库文件**: `libcontinuemanager_napi.z.so`

| JS API | 参数 | C++ 入口 |
|-------|------|---------|
| `on(eventType, callback)` | eventType: "prepareContinue" | `ContinueStateCallbackOn` |
| `off(eventType)` | eventType: "prepareContinue" | `ContinueStateCallbackOff` |

---

## 2. Taihe/ANI 接口

### 2.1 AbilityConnectionManager (Taihe)

**IDL 文件**: `interfaces/taihe/idl/ohos.distributedsched.abilityConnectionManager.taihe`

#### Session 管理

```typescript
// 创建会话
function CreateAbilityConnectionSession(
    bundleName: string, 
    moduleName: string,
    abilityName: string,
    options?: AbilityConnectionSessionOptions
): number;

// 销毁会话  
function DestroyAbilityConnectionSession(sessionId: number): void;
```

#### 连接控制

```typescript
// 建立连接
function ConnectSync(sessionId: number, serverId: string, serverInfo: ServerInfo): void;
function AcceptConnectSync(sessionId: number, serverId: string): void;
function Reject(sessionId: number, serverId: string, rejectReason: string): void;
function Disconnect(sessionId: number, serverId: string): void;
```

#### 数据传输

```typescript
// 发送消息
function SendMessageSync(sessionId: number, serverId: string, msgType: string, msg: string): void;
function SendDataSync(sessionId: number, serverId: string, data: Uint8Array): void;
function SendImageSync(sessionId: number, serverId: string, image: ImageData): void;
```

#### 音视频流

```typescript
function CreateStream(sessionId: number): SurfaceWrapper;
function SetSurfaceId(sessionId: number, surfaceId: string): void;
function StartStream(sessionId: number): void;
function StopStream(sessionId: number): void;
function DestroyStream(sessionId: number): void;
```

---

## 3. IPC 接口

### 3.1 IDistributedSched (SA 1401)

**描述符**: `OHOS.DistributedSchedule.IDistributedSched`  
**头文件**: `services/dtbschedmgr/include/distributed_sched_interface.h`

#### 核心方法

| 方法 | Code | 参数 | 返回 | 位置 |
|-----|------|------|------|------|
| `StartRemoteAbility` | 1 | Want, callType, callerInfo, accountInfo | int32 | :70 |
| `StartAbilityFromRemote` | 4 | Want, requestCode, callerInfo | int32 | :72 |
| `ContinueMission` | 16 | srcMissionId, dstDeviceId, srcDeviceId, params | int32 | :77 |
| `StartContinuation` | 11 | Want, callerInfo, missionId | int32 | :84 |
| `ConnectRemoteAbility` | 6 | Want, callerInfo, connect | int32 | :93 |
| `DisconnectRemoteAbility` | 7 | callerInfo, connect | int32 | :96 |
| `StartRemoteAbilityByCall` | 150 | callerInfo, callback | int32 | :151 |
| `RegisterDSchedEventListener` | 81 | type, listener | int32 | :102 |

#### OnRemoteRequest 分发

**位置**: `services/dtbschedmgr/src/distributed_sched_stub.cpp:266`

```cpp
int32_t DistributedSchedStub::OnRemoteRequest(uint32_t code, MessageParcel& data,
    MessageParcel& reply, MessageOption& option)
{
    // 接口 Token 校验
    if (!CheckInterfaceToken(data)) {
        return ERR_INVALID_DATA;
    }
    
    // 分发到对应 Handler
    auto func = remoteFuncsMap_.find(code);
    if (func != remoteFuncsMap_.end()) {
        return (this->*func->second)(data, reply);
    }
    return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
}
```

### 3.2 IAbilityConnectionManager

**描述符**: `OHOS.DistributedCollab.IAbilityConnectionManager`  
**头文件**: `services/dtbcollabmgr/include/ability_connection_manager/ability_connection_manager_interface.h`

| 方法 | Code | 说明 |
|-----|------|------|
| `NotifyCollabResult` | 0 | 通知协作结果 |
| `NotifyDisconnect` | 1 | 通知断开连接 |
| `NotifyWifiOpen` | 2 | 通知 WiFi 开启 |
| `NotifyPeerVersion` | 3 | 通知对端版本 |

### 3.3 IDExtension

**描述符**: `OHOS.AppManagement.Distributed.IExtension`

| 方法 | Code | 说明 |
|-----|------|------|
| `TriggerOnCreate` | 1 | 触发创建 |
| `TriggerOnDestroy` | 2 | 触发销毁 |
| `TriggerOnCollaborate` | 3 | 触发协作 |

---

## 4. InnerKits (C++ SDK)

### 4.1 DistributedAbilityManagerClient

**头文件**: `interfaces/innerkits/common/include/distributed_ability_manager_client.h`

```cpp
class DistributedAbilityManagerClient {
public:
    static DistributedAbilityManagerClient& GetInstance();
    
    int32_t Register(const std::shared_ptr<ContinuationExtraParams>& extraParams, 
                     int32_t& token);
    int32_t Unregister(int32_t token);
    int32_t RegisterDeviceSelectionCallback(int32_t token, 
                     const sptr<IDeviceSelectionNotifier>& notifier);
    int32_t UpdateConnectStatus(int32_t token, const std::string& deviceId,
                     const DeviceConnectStatus& deviceConnectStatus);
    int32_t StartDeviceManager(int32_t token, 
                     const std::shared_ptr<ContinuationExtraParams>& extraParams);
};
```

### 4.2 权限要求

**位置**: `services/dtbcollabmgr/src/ability_connection_manager/ability_connection_manager.cpp:44-47`

```cpp
static const std::vector<std::string> REQUIRED_PERMISSIONS = {
    "ohos.permission.INTERNET",
    "ohos.permission.GET_NETWORK_INFO",
    "ohos.permission.SET_NETWORK_INFO",
    "ohos.permission.DISTRIBUTED_DATASYNC"
};
```

---

## 5. 配置接口

### 5.1 SystemAbility 配置

**SA 1401**: `sa_profile/1401.json`
```json
{
    "name": "DistributedSched",
    "saId": 1401,
    "libpath": "libdistributedschedsvr.z.so",
    "distributed": true,
    "auto-restart": true,
    "start-on-demand": {
        "deviceonline": "on"
    }
}
```

**SA 1404**: `sa_profile/1404.json`
```json
{
    "name": "DistributedAbilityManager",
    "saId": 1404,
    "libpath": "libdistributed_ability_manager_svr.z.so"
}
```

---

## 6. 接口调用链示例

### 远程启动 Ability

```
JS API
  └── NAPI: StartRemoteAbility()
      └── IPC: DistributedSchedProxy::StartRemoteAbility()
          └── IPC: DistributedSchedStub::OnRemoteRequest(START_REMOTE_ABILITY)
              └── DistributedSchedService::StartRemoteAbilityInner()
                  └── 权限检查: VerifyAccessToken()
                  └── 启动目标 Ability
```

### 创建连接会话

```
JS API
  └── NAPI: createAbilityConnectionSession()
      └── AbilityConnectionManager::ConnectSession()
          └── 权限检查: CheckSessionPermission()
              └── VerifyAccessToken(DISTRIBUTED_DATASYNC)
          └── 创建 Session
```

---

## 相关链接

- 上一章: [03_CodeMap.md](03_CodeMap.md) - 代码地图
- 下一章: [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析
