# 关键调用链

## 1. 设备发现调用链

### 1.1 发现流程

```mermaid
sequenceDiagram
    participant JS as JS 应用
    participant NAPI as N-API 层
    participant SDK as Inner SDK
    participant IPC as IPC 骨架
    participant DM as DeviceManager Service
    participant SB as DSoftBus

    JS->>NAPI: startDiscovering(discoverParam)
    NAPI->>SDK: StartDiscovery(subscribeInfo)
    SDK->>IPC: SendRequest(CMD_START_DISCOVERY)
    IPC->>DM: OnRemoteRequest()
    DM->>SB: StartDiscovery(subscribeId, info)
    
    Note over SB: 发现设备
    
    SB->>DM: OnDeviceFound(deviceId, info)
    DM->>IPC: SendResponse(result)
    IPC->>SDK: OnResult(result)
    SDK->>NAPI: OnDiscoverSuccess(deviceInfo)
    NAPI->>JS: discoverSuccess 事件
```

### 1.2 调用入口

| 阶段 | 函数 | 文件:行号 |
|-----|------|----------|
| JS | `DeviceManagerNapi::StartDiscovering()` | TODO |
| SDK | `DeviceManager::StartDiscovery()` | TODO |
| IPC | `IPCSkeleton::SendRequest()` | 系统 IPC |
| Service | `DeviceManagerServiceImpl::StartDiscovery()` | TODO |
| DSoftBus | `Discovery::StartDiscovery()` | dsoftbus |

## 2. 设备认证调用链

### 2.1 认证流程

```mermaid
sequenceDiagram
    participant JS as JS 应用
    participant NAPI as N-API 层
    participant SDK as Inner SDK
    participant IPC as IPC 骨架
    participant DM as DeviceManager Service
    participant HC as HiChain
    participant UI as PIN UI

    JS->>NAPI: bindTarget(deviceId, bindParam)
    NAPI->>SDK: AuthenticateDevice()
    SDK->>IPC: SendRequest(CMD_AUTH_DEVICE)
    IPC->>DM: OnRemoteRequest()
    
    DM->>HC: CreateGroup()
    HC->>HC: 群组协商
    
    alt 需要用户确认
        DM->>UI: 弹出 PIN 码确认
        UI->>DM: 用户确认结果
    end
    
    HC->>DM: 认证结果
    DM->>IPC: SendResponse()
    IPC->>SDK: OnAuthResult()
    SDK->>NAPI: OnAuthSuccess()
    NAPI->>JS: 认证成功回调
```

### 2.2 关键函数

| 阶段 | 函数 | 文件:行号 |
|-----|------|----------|
| JS | `DeviceManagerNapi::BindTarget()` | TODO |
| Service | `DmAuthManager::StartAuth()` | `services/implementation/src/authentication/dm_auth_manager.cpp` |
| HiChain | `HiChainConnector::AuthenticateDevice()` | TODO |
| UI | `DmDialogManager` | TODO |

## 3. 设备状态监听调用链

### 3.1 监听注册

```mermaid
sequenceDiagram
    participant JS as JS 应用
    participant NAPI as N-API 层
    participant SDK as Inner SDK
    participant IPC as IPC 骨架
    participant DM as DeviceManager Service
    participant SB as DSoftBus

    JS->>NAPI: on('deviceStateChange', callback)
    NAPI->>NAPI: 创建回调对象
    NAPI->>SDK: RegisterDeviceStatusCallback()
    SDK->>IPC: Subscribe()
    IPC->>DM: SubscribeDeviceState()
    DM->>DM: 注册状态变更回调
```

### 3.2 状态变更通知

```mermaid
sequenceDiagram
    participant SB as DSoftBus
    participant DM as DeviceManager Service
    participant IPC as IPC 骨架
    participant SDK as Inner SDK
    participant NAPI as N-API 层
    participant JS as JS 应用

    SB->>DM: OnDeviceOnline(deviceInfo)
    DM->>DM: 更新设备状态
    DM->>IPC: SendBroadcast()
    IPC->>SDK: OnDeviceStatusChanged()
    SDK->>NAPI: OnDeviceStateChange()
    NAPI->>JS: emit('deviceStateChange', data)
```

## 4. 设备发布调用链

### 4.1 发布流程

```mermaid
sequenceDiagram
    participant JS as JS 应用
    participant NAPI as N-API 层
    participant SDK as Inner SDK
    participant IPC as IPC 骨架
    participant DM as DeviceManager Service
    participant SB as DSoftBus

    JS->>NAPI: publishDeviceDiscovery(publishInfo)
    NAPI->>SDK: PublishDeviceDiscovery()
    SDK->>IPC: SendRequest(CMD_PUBLISH)
    IPC->>DM: OnRemoteRequest()
    DM->>SB: PublishDeviceDiscovery()
    
    Note over SB: 设备可被发现
    
    SB->>DM: 发布结果
    DM->>IPC: SendResponse()
    IPC->>SDK: OnPublishResult()
    SDK->>NAPI: OnPublishSuccess()
    NAPI->>JS: publishSuccess 事件
```

## 5. 初始化调用链

### 5.1 服务启动

```mermaid
flowchart TD
    A[SAMgr] --> B{OnStart}
    B --> C[DeviceManagerServiceImpl::Init]
    C --> D[初始化 IPC]
    C --> E[初始化认证模块]
    C --> F[初始化发现模块]
    C --> G[注册软总线回调]
    C --> H[订阅系统事件]
```

### 5.2 实例创建

```mermaid
flowchart TD
    A[createDeviceManager] --> B{N-API 检查}
    B -->|已创建| C[返回现有实例]
    B -->|未创建| D[创建 DeviceManagerNapi]
    D --> E[调用 SDK Init]
    E --> F[IPC 注册回调]
    F --> G[返回实例]
```

## 6. 释放调用链

### 6.1 实例释放

```mermaid
flowchart TD
    A[releaseDeviceManager] --> B{检查引用计数}
    B -->|还有引用| C[引用计数减一]
    B -->|无引用| D[注销回调]
    D --> E[释放 SDK 资源]
    E --> F[删除 N-API 实例]
    F --> G[释放成功]
```

## 7. 关键内部调用

### 7.1 HiChain 交互

```mermaid
sequenceDiagram
    participant DM as DeviceManager Service
    participant HC as HiChain
    participant Cred as 凭据存储

    DM->>HC: Initialize()
    HC->>DM: 返回会话 ID
    
    DM->>HC: CreateGroup(groupInfo)
    HC->>Cred: 创建凭据
    Cred->>HC: 凭据引用
    HC->>DM: 群组 ID
    
    Note over DM,HC: 设备认证协商
    
    DM->>HC: FinishAuth()
    HC->>DM: 认证结果
```

### 7.2 DSoftBus 交互

```mermaid
sequenceDiagram
    participant DM as DeviceManager Service
    participant SB as DSoftBus
    participant Cache as 设备缓存

    DM->>SB: GetLocalDeviceInfo()
    SB->>DM: 本地设备信息
    
    DM->>SB: GetTrustedDeviceList()
    SB->>DM: 可信设备列表
    
    loop 设备发现
        DM->>SB: StartDiscovery()
        SB->>DM: 设备发现结果
    end
    
    SB->>DM: 设备上下线通知
    DM->>Cache: 更新缓存
```

## 8. 逆向调用链

### 8.1 回调处理

```mermaid
sequenceDiagram
    participant SB as DSoftBus
    participant DM as DeviceManager Service
    participant IPC as IPC 骨架
    participant NAPI as N-API 层
    participant JS as JS 应用

    SB->>DM: DeviceOnline(deviceInfo)
    DM->>DM: 处理状态变更
    DM->>IPC: SendRequest/Reply
    IPC->>NAPI: DeviceStatusCallback
    NAPI->>JS: emit(event, data)
```

## 9. 错误处理调用链

### 9.1 错误传播

```mermaid
flowchart TD
    A[底层错误] --> B{错误分类}
    B -->|参数错误| C[返回 NAPI 错误码]
    B -->|权限错误| D[返回 PERMISSION_DENIED]
    B -->|系统错误| E[返回 SYSTEM_ERROR]
    
    C --> F[JS onerror 回调]
    D --> F
    E --> F
    E --> G[日志记录]
```

## 10. 调用链速查表

| 功能 | 入口 | 关键路径 | 终点 |
|-----|------|---------|------|
| 设备发现 | `startDiscovering` | SDK → IPC → Service → DSoftBus | `discoverSuccess` |
| 设备认证 | `bindTarget` | SDK → IPC → Service → HiChain | `bindSuccess` |
| 设备发布 | `publishDeviceDiscovery` | SDK → IPC → Service → DSoftBus | `publishSuccess` |
| 状态监听 | `on('deviceStateChange')` | SDK → IPC → Service → 回调注册 | 事件触发 |
| 实例创建 | `createDeviceManager` | N-API → SDK | 实例返回 |
| 实例释放 | `releaseDeviceManager` | SDK 清理 → 回调注销 | 完成 |
