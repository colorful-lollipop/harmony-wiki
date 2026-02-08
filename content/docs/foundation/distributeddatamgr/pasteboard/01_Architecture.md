# 架构说明

## 目的

本文档详细说明 Pasteboard 剪贴板服务的系统架构，包括组件图、数据流、线程模型和关键时序。

## 适用范围

- 需要理解系统整体架构的开发者
- 进行模块设计或重构的架构师
- 进行性能优化或问题定位的工程师

## 系统架构图

### 整体架构

```mermaid
graph TB
    subgraph "Application Layer"
        JS[JS/TS Application]
        NAPP[Native Application]
    end
    
    subgraph "Interface Layer"
        NAPI[N-API Module<br/>libpasteboard_napi.z.so]
        NDK[NDK Library<br/>libpasteboard.so]
        ANI[ANI Module<br/>libpasteboard_ani.so]
    end
    
    subgraph "Framework Layer"
        Client[PasteboardClient<br/>InnerKit]
        Data[PasteData/Record<br/>Data Structure]
        Loader[ServiceLoader<br/>Proxy Management]
    end
    
    subgraph "IPC Layer"
        Proxy[Service Proxy]
        Stub[Service Stub]
        IPC[IPC Mechanism<br/>Binder]
    end
    
    subgraph "Service Layer"
        Service[PasteboardService<br/>SA ID: 3701]
        Core[Core Manager]
        DFX[DFX Reporter]
        Account[Account Manager]
    end
    
    JS --> NAPI
    NAPP --> NDK
    JS --> ANI
    
    NAPI --> Client
    NDK --> Client
    ANI --> Client
    
    Client --> Data
    Client --> Loader
    Loader --> Proxy
    
    Proxy --> IPC
    IPC --> Stub
    Stub --> Service
    
    Service --> Core
    Service --> DFX
    Service --> Account
```

### 模块依赖关系

```mermaid
graph LR
    subgraph "interfaces"
        NAPI[napi_init.cpp]
        NDK[oh_pasteboard.cpp]
    end
    
    subgraph "framework/innerkits"
        PBC[pasteboard_client.cpp]
        PBD[pasteboard_data.cpp]
        PSL[service_loader.cpp]
    end
    
    subgraph "services/zidl"
        PSTUB[pasteboard_*_stub.cpp]
        PPROXY[pasteboard_*_proxy.cpp]
    end
    
    subgraph "services/core"
        PBS[pasteboard_service.cpp]
        PDM[pasteboard_delay_manager.cpp]
        PAM[pasteboard_ability_manager.cpp]
    end
    
    NAPI --> PBC
    NDK --> PBC
    PBC --> PBD
    PBC --> PSL
    PSL --> PPROXY
    PPROXY --> IPC
    IPC --> PSTUB
    PSTUB --> PBS
    PBS --> PDM
    PBS --> PAM
```

## 数据流图

### SetPasteData 数据流（写入剪贴板）

```mermaid
sequenceDiagram
    participant App as Application
    participant NAPI as N-API Layer
    participant Client as PasteboardClient
    participant Proxy as Service Proxy
    participant IPC as IPC
    participant Service as PasteboardService
    participant Storage as Data Storage

    App->>NAPI: setData(pasteData)
    NAPI->>NAPI: Validate parameters
    NAPI->>NAPI: Create PasteDataNapi
    NAPI->>Client: SetPasteData()
    Client->>Client: Serialize to TLV
    Client->>Proxy: IPC: SET_PASTE_DATA
    Proxy->>IPC: MessageParcel
    IPC->>Service: OnRemoteRequest()
    Service->>Service: VerifyPermission()
    Service->>Service: Validate caller
    Service->>Storage: Store paste data
    Service->>Service: Notify observers
    Service-->>IPC: Return result
    IPC-->>Proxy: 
    Proxy-->>Client: 
    Client-->>NAPI: 
    NAPI-->>App: Promise resolved
```

### GetPasteData 数据流（读取剪贴板）

```mermaid
sequenceDiagram
    participant App as Application
    participant NAPI as N-API Layer
    participant Client as PasteboardClient
    participant Proxy as Service Proxy
    participant IPC as IPC
    participant Service as PasteboardService
    participant Storage as Data Storage

    App->>NAPI: getData()
    NAPI->>Client: GetPasteData()
    Client->>Proxy: IPC: GET_PASTE_DATA
    Proxy->>IPC: MessageParcel
    IPC->>Service: OnRemoteRequest()
    Service->>Service: VerifyPermission()
    Service->>Service: Check share option
    Service->>Storage: Retrieve paste data
    Service->>Service: Grant URI permissions
    Service->>Service: Marshal data
    Service-->>IPC: Return data (TLV/Ashmem)
    IPC-->>Proxy: 
    Proxy-->>Client: 
    Client->>Client: Deserialize TLV
    Client-->>NAPI: Return PasteData
    NAPI-->>App: Promise resolved with data
```

### 分布式数据同步流

```mermaid
sequenceDiagram
    participant DevA as Device A (Source)
    participant SA as Service A
    participant DM as Device Manager
    participant DevB as Device B (Target)
    participant SB as Service B

    DevA->>SA: Copy data to pasteboard
    SA->>SA: Mark as local data
    SA->>DM: Register device status
    
    DevB->>SB: Paste operation
    SB->>SB: Detect remote data
    SB->>DM: Query source device
    SB->>SA: P2P request data
    SA->>SA: Check share option
    SA->>SB: Transfer encrypted data
    SB->>SB: Decrypt and cache
    SB->>DevB: Return paste data
```

## 线程模型

### 服务线程架构

```mermaid
graph TB
    subgraph "Main Thread"
        OnStart[OnStart() - Service Init]
        Init[Init() - Publish SA]
        Handler[EventHandler<br/>Event Processing]
    end
    
    subgraph "IPC Thread Pool"
        IPC1[IPC Thread 1<br/>OnRemoteRequest]
        IPC2[IPC Thread 2<br/>OnRemoteRequest]
        IPC3[IPC Thread N<br/>...]
    end
    
    subgraph "FFRT Task Queue"
        FFRT1[Delay Getter Task]
        FFRT2[Data Sync Task]
        FFRT3[Entity Recognition]
    end
    
    OnStart --> Init
    Init --> Handler
    
    IPC1 --> Handler
    IPC2 --> Handler
    
    Handler --> FFRT1
    Handler --> FFRT2
    Handler --> FFRT3
```

### 线程配置

```cpp
// services/core/src/pasteboard_service.cpp:82
constexpr uint32_t MAX_IPC_THREAD_NUM = 32;  // IPC 线程池最大线程数

// OnStart() 中设置
IPCSkeleton::SetMaxWorkThreadNum(MAX_IPC_THREAD_NUM);
```

### 线程安全机制

| 资源 | 锁机制 | 代码位置 |
|------|--------|----------|
| pasteData_ | shared_mutex (pasteDataMutex_) | `pasteboard_service.cpp:122` |
| dataHistory_ | mutex (historyMutex_) | `pasteboard_service.cpp:121` |
| observerSet_ | mutex (observerSetMutex_) | `pasteboard_client.cpp:587` |
| clients_ | 内部封装 | `pasteboard_service.h:267` |

## 关键时序

### 服务启动时序

```mermaid
sequenceDiagram
    participant Init as Init Process
    participant SAMgr as SA Manager
    participant PBS as PasteboardService
    participant DPS as Distributed Module

    Init->>SAMgr: Start SA 3701
    SAMgr->>PBS: MakeAndRegisterAbility()
    PBS->>PBS: Constructor
    PBS->>SAMgr: Publish(this)
    PBS->>PBS: OnStart()
    PBS->>PBS: SetMaxWorkThreadNum(32)
    PBS->>PBS: InitServiceHandler()
    PBS->>DPS: moduleConfig_.Init()
    PBS->>DPS: Watch config changes
    PBS->>PBS: AddSysAbilityListener()
    PBS->>PBS: InitScreenStatus()
    PBS-->>SAMgr: Service running
```

### 数据变化通知时序

```mermaid
sequenceDiagram
    participant AppA as App A (Writer)
    participant Service as PasteboardService
    participant Observers as Registered Observers
    participant AppB as App B (Observer)

    AppA->>Service: SetPasteData()
    Service->>Service: Store data
    Service->>Service: changeCount_++
    Service->>Observers: NotifyObservers()
    
    loop For each observer
        Observers->>AppB: OnPasteboardChanged()
        AppB->>AppB: Handle callback
    end
```

### 延迟加载时序

```mermaid
sequenceDiagram
    participant App as Application
    participant Client as PasteboardClient
    participant Service as PasteboardService
    participant DelayGetter as Delay Getter

    App->>Client: SetPasteData with DelayGetter
    Client->>Service: SetPasteData (metadata only)
    Service->>Service: Store getter reference
    
    Note over App,Service: Later...
    
    App->>Client: GetPasteData
    Client->>Service: GET_PASTE_DATA
    Service->>Service: Detect delay data
    Service->>DelayGetter: GetDelayPasteData()
    DelayGetter->>DelayGetter: Load actual data
    DelayGetter-->>Service: Return full data
    Service-->>Client: Return complete PasteData
    Client-->>App: 
```

## 组件职责

### Interface Layer

| 组件 | 职责 | 关键文件 |
|------|------|----------|
| **N-API** | JS/TS 接口封装，参数转换 | `interfaces/kits/napi/src/napi_*.cpp` |
| **NDK** | C 接口封装，内存管理 | `interfaces/ndk/src/oh_pasteboard.cpp` |
| **CJ FFI** | Cangjie 语言桥接 | `interfaces/cj/src/pasteboard_ffi.cpp` |
| **ANI** | ArkTS Native 接口 | `interfaces/ani/src/pasteboard_ani.cpp` |

### Framework Layer

| 组件 | 职责 | 关键文件 |
|------|------|----------|
| **PasteboardClient** | 客户端 API，服务发现 | `framework/innerkits/src/pasteboard_client.cpp` |
| **PasteData** | 数据模型，序列化 | `framework/innerkits/src/paste_data.cpp` |
| **ServiceLoader** | SA 连接管理 | `framework/innerkits/src/pasteboard_service_loader.cpp` |
| **TLV** | 数据序列化/反序列化 | `framework/tlv/tlv_*.cpp` |

### Service Layer

| 组件 | 职责 | 关键文件 |
|------|------|----------|
| **PasteboardService** | 核心服务，IPC 处理 | `services/core/src/pasteboard_service.cpp` |
| **DelayManager** | 延迟数据管理 | `services/core/src/pasteboard_delay_manager.cpp` |
| **AbilityManager** | Ability 生命周期 | `services/core/src/pasteboard_ability_manager.cpp` |
| **DFX** | 诊断、日志、事件 | `services/dfx/src/*.cpp` |

## 关键结论

1. **分层架构**: 清晰的 Interface → Framework → Service 三层结构，职责分离明确。

2. **异步设计**: 主要 API 支持异步调用（Promise/Callback），避免阻塞主线程。

3. **IPC 优化**: 32 线程 IPC 池处理并发请求，FFRT 任务队列处理异步操作。

4. **数据安全**: 权限验证在服务入口统一处理，ShareOption 控制数据分享范围。

5. **分布式支持**: DeviceManager 集成，P2P 数据传输，支持延迟加载优化性能。

## 相关链接

- [目录结构 → 02_Directory_Structure.md](02_Directory_Structure.md)
- [内部 API → 04_Inner_API.md](04_Inner_API.md)
- [调用链图 → appendix/Callgraphs.md](appendix/Callgraphs.md)
