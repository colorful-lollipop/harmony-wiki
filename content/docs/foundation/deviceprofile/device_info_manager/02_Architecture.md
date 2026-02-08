# 02 架构与数据流

**文档目的**: 帮助读者理解 DeviceProfile 的组件架构、模块关系和数据流转  
**适用范围**: 所有读者（新人理解架构、安全研究员识别信任边界）  

---

## 2.1 整体架构

```mermaid
graph TB
    subgraph 应用层["应用层/系统服务"]
        Caller1[device_manager]
        Caller2[softbus_server]
        Caller3[其他系统服务]
    end
    
    subgraph IPC层["IPC层"]
        Client[DistributedDeviceProfileClient<br/>IPC客户端]
        Proxy[DistributedDeviceProfileProxy]
        Stub[DistributedDeviceProfileStubNew]
    end
    
    subgraph 服务层["服务层 (SA 6001)"]
        Service[DistributedDeviceProfileServiceNew<br/>主服务入口]
        
        subgraph 管理器层["核心管理器"]
            DPM[DeviceProfileManager<br/>Profile管理]
            TPM[TrustProfileManager<br/>可信设备管理]
            SPM[SubscribeProfileManager<br/>订阅管理]
            PDM[ProfileDataManager<br/>数据持久化]
            CSM[ContentSensorManager<br/>内容采集]
            MUM[MultiUserManager<br/>多用户管理]
        end
        
        PM[PermissionManager<br/>权限管理]
    end
    
    subgraph 数据层["数据层"]
        KV[KV Store<br/>Service/Characteristic]
        RDB[Relational DB<br/>Device/ACL/Trust]
    end
    
    subgraph 依赖层["依赖子系统"]
        DM[DeviceManager]
        SB[Softbus]
        AT[AccessToken]
        OA[OSAccount]
    end
    
    Caller1 --> Client
    Caller2 --> Client
    Caller3 --> Client
    Client --> Proxy
    Proxy --IPC--> Stub
    Stub --> Service
    Service --> PM
    Service --> DPM
    Service --> TPM
    Service --> SPM
    Service --> CSM
    Service --> MUM
    
    DPM --> PDM
    TPM --> PDM
    SPM --> PDM
    CSM --> DPM
    
    PDM --> KV
    PDM --> RDB
    
    Service --> DM
    Service --> SB
    PM --> AT
    MUM --> OA
```

---

## 2.2 核心组件

### 2.2.1 服务端组件 (SA 6001)

| 组件 | 文件位置 | 职责 |
|------|----------|------|
| **DistributedDeviceProfileServiceNew** | `services/core/include/distributed_device_profile_service_new.h:42` | SA 主入口，处理 IPC 请求 |
| **DeviceProfileManager** | `services/core/include/deviceprofilemanager/device_profile_manager.h` | Device/Service/Characteristic Profile 管理 |
| **TrustProfileManager** | `services/core/include/trustprofilemanager/trust_profile_manager.h` | 可信设备和 ACL 管理 |
| **SubscribeProfileManager** | `services/core/include/subscribeprofilemanager/subscribe_profile_manager.h` | 订阅和通知管理 |
| **ProfileDataManager** | `services/core/include/profiledatamanager/profile_data_manager.h` | 数据持久化抽象 |
| **ContentSensorManager** | `services/core/include/contentsensormanager/content_sensor_manager.h` | 内容传感器数据采集 |
| **PermissionManager** | `services/core/include/permissionmanager/permission_manager.h` | 权限校验和访问控制 |
| **MultiUserManager** | `services/core/include/multiusermanager/multi_user_manager.h` | 多用户支持 |

### 2.2.2 客户端组件

| 组件 | 文件位置 | 职责 |
|------|----------|------|
| **DistributedDeviceProfileClient** | `interfaces/innerkits/core/include/distributed_device_profile_client.h:46` | IPC 客户端单例，提供服务调用接口 |
| **DistributedDeviceProfileProxy** | `interfaces/innerkits/core/include/distributed_device_profile_proxy.h` | IPC 代理，封装 IPC 调用 |

### 2.2.3 通信组件

| 组件 | 文件位置 | 职责 |
|------|----------|------|
| **DistributedDeviceProfileStubNew** | `services/core/include/distributed_device_profile_stub_new.h` | IPC Stub，接收并分发 IPC 请求 |
| **IDistributedDeviceProfile** | `common/include/interfaces/i_distributed_device_profile.h` | IPC 接口定义 |

---

## 2.3 信任边界

```mermaid
graph TB
    subgraph 不信任域["不信任域"]
        Ext[外部输入]
    end
    
    subgraph 半信任域["半信任域<br/>(其他系统服务)"]
        Caller[调用者进程<br/>需权限校验]
    end
    
    subgraph 信任域["信任域<br/>(DeviceProfile SA)"]
        Service[SA 6001 服务]
        
        subgraph 内部边界["内部数据边界"]
            KV[KV Store]
            RDB[RDB]
        end
    end
    
    Ext --IPC输入--> Caller
    Caller --IPC请求--> Service
    Service --权限校验--> Service
    Service --数据操作--> KV
    Service --数据操作--> RDB
```

**信任边界说明**:

| 边界 | 说明 | 防护机制 |
|------|------|----------|
| **IPC 边界** | 进程间通信边界 | 权限检查 + 接口白名单 |
| **数据边界** | 数据存储边界 | 文件权限 + 加密 |
| **网络边界** | 跨设备同步边界 | Softbus 安全通道 + 设备绑定 |

---

## 2.4 数据流分析

### 2.4.1 Profile 查询流程

```mermaid
sequenceDiagram
    participant Caller as 调用者
    participant Client as DDPClient
    participant Proxy as Proxy
    participant Stub as Stub
    participant Service as Service
    participant PM as PermissionManager
    participant PDM as ProfileDataManager
    participant DB as KVStore/RDB
    
    Caller->>+Client: GetDeviceProfile(deviceId, serviceId)
    Client->>+Proxy: SendRequest(GET_DEVICE_PROFILE)
    Proxy->>Stub: IPC 调用
    Stub->>+Service: OnRemoteRequest
    Service->>+PM: IsCallerTrust("GetDeviceProfile")
    PM-->>-Service: 返回权限结果
    alt 权限校验通过
        Service->>+PDM: 查询数据
        PDM->>DB: Get
        DB-->>PDM: 返回数据
        PDM-->>-Service: Profile 数据
        Service-->>Stub: WriteToParcel
        Stub-->>Proxy: IPC 返回
        Proxy-->>Client: 反序列化
        Client-->>Caller: 返回 Profile
    else 权限校验失败
        Service-->>Stub: DP_PERMISSION_DENIED
        Stub-->>Proxy: IPC 返回
        Proxy-->>Client: 错误码
        Client-->>Caller: 返回错误
    end
```

**证据**: 
- `services/core/src/distributed_device_profile_stub_new.cpp`
- `services/core/src/distributed_device_profile_service_new.cpp:728-785`

### 2.4.2 Profile 同步流程

```mermaid
sequenceDiagram
    participant AppA as 设备A应用
    participant ClientA as DDPClient
    participant ServiceA as Service(设备A)
    participant SB as Softbus
    participant ServiceB as Service(设备B)
    participant ClientB as DDPClient
    participant AppB as 设备B应用
    
    AppA->>+ClientA: SyncDeviceProfile(syncOptions)
    ClientA->>ServiceA: IPC 请求
    ServiceA->>ServiceA: 权限校验
    
    loop 对每个目标设备
        ServiceA->>SB: 发送同步请求
        SB->>ServiceB: 跨设备传输
        ServiceB->>ServiceB: 验证来源
        ServiceB->>ServiceB: 存储数据
        ServiceB->>SB: 返回结果
        SB->>ServiceA: 返回结果
    end
    
    ServiceA->>ClientA: SyncCompleted 回调
    ClientA->>AppA: OnSyncCompleted
    
    ServiceB->>ClientB: ProfileChanged 通知
    ClientB->>AppB: OnProfileChanged
```

**证据**: `services/core/src/deviceprofilemanager/listener/`

### 2.4.3 Profile 变更订阅流程

```mermaid
sequenceDiagram
    participant App as 订阅者
    participant Client as DDPClient
    participant Service as SubscribeProfileManager
    participant KV as KV Store
    participant Remote as 远端设备
    
    App->>+Client: SubscribeDeviceProfile(subscribeInfo, callback)
    Client->>Service: IPC 注册订阅
    Service->>Service: 保存回调对象
    
    Note over Service,KV: 监听数据变化
    KV->>Service: OnDataChanged
    
    alt 本地变更
        Service->>Client: OnProfileChanged
        Client->>App: 回调通知
    else 远端变更
        Remote->>KV: 同步数据
        KV->>Service: OnDataChanged
        Service->>Client: OnProfileChanged
        Client->>App: 回调通知
    end
    
    App->>Client: UnSubscribeDeviceProfile
    Client->>Service: 取消订阅
```

---

## 2.5 线程模型

```mermaid
graph TB
    subgraph 主线程["主线程 (Main Thread)"]
        IPC[IPC 请求处理]
        Lifecycle[SA 生命周期]
    end
    
    subgraph Handler线程["EventHandler 线程"]
        Event[事件分发]
        Notify[变更通知]
        Callback[回调执行]
    end
    
    subgraph 线程池["线程池 (FFRT)"]
        AsyncDB[异步数据库操作]
        AsyncSync[异步同步操作]
    end
    
    IPC --> Event
    Lifecycle --> Event
    Event --> Notify
    Event --> Callback
    Event --> AsyncDB
    Event --> AsyncSync
```

| 线程类型 | 用途 | 证据 |
|----------|------|------|
| **主线程** | IPC 请求分发、SA 生命周期 | `services/core/src/distributed_device_profile_service_new.cpp` |
| **EventHandler 线程** | 异步事件处理、回调通知 | `services/core/include/common/event_handler_factory.h` |
| **FFRT 线程池** | 数据库操作、跨设备同步 | `bundle.json:31` 依赖 `ffrt` |

---

## 2.6 模块依赖关系

```mermaid
graph TD
    Service[DistributedDeviceProfileServiceNew]
    
    Service --> PM[PermissionManager]
    Service --> DPM[DeviceProfileManager]
    Service --> TPM[TrustProfileManager]
    Service --> SPM[SubscribeProfileManager]
    Service --> CSM[ContentSensorManager]
    Service --> MUM[MultiUserManager]
    
    DPM --> PDM[ProfileDataManager]
    TPM --> PDM
    SPM --> PDM
    
    PDM --> KV[KV Store Adapter]
    PDM --> RDB[RDB Adapter]
    
    CSM --> DPM
    
    PM --> AT[AccessTokenKit]
    MUM --> OA[OSAccountKit]
    
    Service --> DM[DeviceManager]
    Service --> SB[Softbus]
```

**依赖说明**:
- **PermissionManager**: 所有管理器都需通过它进行权限校验
- **ProfileDataManager**: 数据持久化统一入口
- **ContentSensorManager**: 采集数据写入 DeviceProfileManager
- **MultiUserManager**: 用户切换时清理/恢复数据

---

## 2.7 IPC 通信机制

### 2.7.1 IPC 接口定义

```cpp
// common/include/interfaces/i_distributed_device_profile.h
class IDistributedDeviceProfile : public IRemoteBroker {
public:
    enum Code {
        PUT_DEVICE_PROFILE = 0,
        GET_DEVICE_PROFILE,
        DELETE_DEVICE_PROFILE,
        SUBSCRIBE_PROFILE_EVENT,
        UNSUBSCRIBE_PROFILE_EVENT,
        SYNC_DEVICE_PROFILE,
        // ... 40+ 个命令码
    };
    
    virtual int32_t PutDeviceProfile(const ServiceCharacteristicProfile& profile) = 0;
    virtual int32_t GetDeviceProfile(const std::string& deviceId, 
                                      const std::string& serviceId,
                                      ServiceCharacteristicProfile& profile) = 0;
    // ... 更多纯虚函数
};
```

### 2.7.2 IPC 调用流程

```mermaid
graph LR
    A[Client] -->|SendRequest| B[Proxy]
    B -->|Binder| C[Stub]
    C -->|OnRemoteRequest| D[Service]
    D -->|处理| E[返回结果]
    E -->|WriteToParcel| C
    C -->|Binder| B
    B -->|ReadFromParcel| A
```

**证据**: 
- `common/include/interfaces/i_distributed_device_profile.h`
- `interfaces/innerkits/core/include/distributed_device_profile_proxy.h`
- `services/core/include/distributed_device_profile_stub_new.h`

---

## 2.8 关键结论

1. **分层架构**: Client-Proxy-Stub-Service-Manager-Data 六层架构，职责清晰
2. **权限控制集中**: PermissionManager 统一管理所有权限检查
3. **数据持久化抽象**: ProfileDataManager 屏蔽 KV Store 和 RDB 的差异
4. **异步处理**: 数据库和同步操作使用线程池，避免阻塞主线程
5. **跨设备信任**: 依赖 Softbus 的安全通道和设备绑定机制

---

## 2.9 相关章节

| 目标 | 推荐阅读 |
|------|----------|
| 代码位置 | [03_CodeMap.md](03_CodeMap.md) |
| API 详情 | [04_Interface.md](04_Interface.md) |
| 安全风险 | [05_AttackSurface.md](05_AttackSurface.md), [06_SecurityReview.md](06_SecurityReview.md) |
| 实现细节 | [08_Internals.md](08_Internals.md) |
