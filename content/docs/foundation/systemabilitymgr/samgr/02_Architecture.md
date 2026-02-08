# 架构设计

## 组件架构图

```mermaid
graph TB
    subgraph Client[客户端]
        App[应用程序]
        SA[系统能力]
    end
    
    subgraph Framework[框架层]
        Proxy[SystemAbilityManagerProxy]
        CallbackStub[Callback Stub]
    end
    
    subgraph IPC[IPC层]
        Binder[Binder IPC]
        DBinder[DBinder 分布式]
    end
    
    subgraph Service[服务层]
        Stub[SystemAbilityManagerStub]
        SAM[SystemAbilityManager]
        Scheduler[StateScheduler]
        Collector[DeviceStatusCollectManager]
    end
    
    subgraph Data[数据层]
        AbilityMap[(abilityMap_)]
        ListenerMap[(listenerMap_)]
        OnDemandMap[(onDemandAbilityMap_)]
        ProcessMap[(systemProcessMap_)]
    end
    
    App --> Proxy
    SA --> Proxy
    Proxy --> Binder
    Binder --> Stub
    Stub --> SAM
    SAM --> Scheduler
    SAM --> Collector
    SAM --> AbilityMap
    SAM --> ListenerMap
    SAM --> OnDemandMap
    SAM --> ProcessMap
```

## 类层次结构

```mermaid
classDiagram
    class IRemoteBroker {
        <<interface>>
    }
    
    class ISystemAbilityManager {
        <<interface>>
        +GetSystemAbility(saId)
        +AddSystemAbility(saId, ability)
        +RemoveSystemAbility(saId)
        +SubscribeSystemAbility(saId, listener)
        +LoadSystemAbility(saId, callback)
    }
    
    class SystemAbilityManagerStub {
        +OnRemoteRequest(code, data, reply)
    }
    
    class SystemAbilityManager {
        -abilityMap_: map~int,SAInfo~
        -listenerMap_: map~int,list~SAListener~~
        -onDemandAbilityMap_: map~int,u16string~
        -systemProcessMap_: map~u16string,IRemoteObject~
        +GetSystemAbility(saId)
        +AddSystemAbility(saId, ability, extraProp)
        +LoadSystemAbility(saId, callback)
        -DoLoadSystemAbility(...)
    }
    
    class SystemAbilityManagerProxy {
        -remote_: IRemoteObject
        +GetSystemAbility(saId)
        +AddSystemAbility(saId, ability)
    }
    
    class DynamicCache {
        +QueryResult(saId, code)
        +Recompute(saId, code)
    }
    
    class SystemAbilityStateScheduler {
        -stateContext_: map~int,SAContext~
        -processContext_: map~u16string,ProcessContext~
        +ScheduleAbilityState(saId, event)
        +ScheduleProcessState(procName, event)
    }
    
    IRemoteBroker <|-- ISystemAbilityManager
    ISystemAbilityManager <|.. SystemAbilityManagerStub
    SystemAbilityManagerStub <|-- SystemAbilityManager
    DynamicCache <|-- SystemAbilityManager
    SystemAbilityManager --> SystemAbilityStateScheduler
    ISystemAbilityManager <|.. SystemAbilityManagerProxy
```

## 数据流

### 1. 服务注册流程

```mermaid
sequenceDiagram
    participant SA as 系统能力
    participant Proxy as SamgrProxy
    participant Stub as SamgrStub
    participant SAM as SystemAbilityManager
    participant Map as abilityMap_
    
    SA->>Proxy: AddSystemAbility(saId, remoteObj)
    Proxy->>Stub: IPC Transaction
    Stub->>Stub: CheckInputSysAbilityId(saId)
    Stub->>Stub: SELinux CheckPermission
    Stub->>SAM: AddSystemAbility(saId, remoteObj, extraProp)
    SAM->>SAM: CheckInputSysAbilityId(saId)
    SAM->>SAM: Validate ability != nullptr
    SAM->>SAM: Check abilityMap_.size() < MAX_SERVICES
    SAM->>Map: abilityMap_[saId] = saInfo
    SAM->>SAM: AddDeathRecipient(abilityDeath_)
    SAM->>SAM: SendSystemAbilityAddedMsg(saId)
    SAM-->>Stub: ERR_OK
    Stub-->>Proxy: ERR_OK
    Proxy-->>SA: ERR_OK
```

### 2. 服务查询流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Proxy as SamgrProxy
    participant Cache as DynamicCache
    participant Stub as SamgrStub
    participant SAM as SystemAbilityManager
    participant Map as abilityMap_
    
    Client->>Proxy: GetSystemAbility(saId)
    Proxy->>Cache: QueryResult(saId)
    alt Cache Hit
        Cache-->>Proxy: cached remoteObj
        Proxy-->>Client: remoteObj
    else Cache Miss
        Proxy->>Stub: IPC Transaction
        Stub->>Stub: CheckInputSysAbilityId(saId)
        Stub->>SAM: GetSystemAbility(saId)
        SAM->>SAM: CheckInputSysAbilityId(saId)
        SAM->>Map: abilityMap_.find(saId)
        Map-->>SAM: SAInfo
        SAM-->>Stub: remoteObj
        Stub-->>Proxy: remoteObj
        Proxy->>Cache: UpdateCache(saId, remoteObj)
        Proxy-->>Client: remoteObj
    end
```

### 3. 动态加载流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Proxy as SamgrProxy
    participant Stub as SamgrStub
    participant SAM as SystemAbilityManager
    participant State as StateScheduler
    participant LSAM as LocalAbilityManager
    participant SA as 目标SA
    
    Client->>Proxy: LoadSystemAbility(saId, callback)
    Proxy->>Stub: IPC Transaction
    Stub->>SAM: LoadSystemAbility(saId, callback)
    SAM->>SAM: CheckInputSysAbilityId(saId)
    SAM->>SAM: Find procName from profile
    SAM->>State: ScheduleProcessState(procName, START)
    State->>LSAM: StartProcess(procName)
    LSAM->>SA: Fork & Exec
    SA->>SAM: AddSystemAbility(saId, remoteObj)
    SAM->>SAM: NotifySystemAbilityLoaded(saId)
    SAM->>Client: OnLoadSystemAbilitySuccess(saId, remoteObj)
```

## 线程模型

### 主服务线程

```
┌─────────────────────────────────────────────┐
│              Samgr Main Thread              │
│  ┌─────────────────────────────────────┐   │
│  │  IPC Thread Pool (Binder)           │   │
│  │  - OnRemoteRequest handlers         │   │
│  │  - Synchronous operations           │   │
│  └─────────────────────────────────────┘   │
│                    │                        │
│  ┌─────────────────▼───────────────────┐   │
│  │  FFRT Work Queue (async tasks)      │   │
│  │  - SendSystemAbilityAddedMsg        │   │
│  │  - RemoveCheckLoadedMsg             │   │
│  │  - Notify callbacks                 │   │
│  └─────────────────────────────────────┘   │
│                    │                        │
│  ┌─────────────────▼───────────────────┐   │
│  │  Timer Thread                       │   │
│  │  - reportEventTimer_                │   │
│  │  - Periodic tasks                   │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### 并发控制

| 数据 | 锁类型 | 用途 |
|------|--------|------|
| `abilityMap_` | `shared_mutex` | SA 注册表读写保护 |
| `listenerMap_` | `mutex` | 监听者列表保护 |
| `onDemandLock_` | `mutex` | 按需加载状态保护 |
| `saProfileMapLock_` | `mutex` | SA 配置文件保护 |
| `systemProcessMapLock_` | `mutex` | 进程映射保护 |

## 关键时序

### 启动时序

```mermaid
timeline
    title Samgr 启动流程
    section 初始化
        main() : 进程启动
                : 创建 SystemAbilityManager 单例
                : Init() 初始化
    section 配置加载
        InitSaProfile() : 加载 SA 配置文件
                        : 解析 XML/JSON 配置
                        : 填充 saProfileMap_
    section 状态机
        StateScheduler : 初始化状态调度器
                       : 创建状态机实例
    section 服务就绪
        AddSamgrToAbilityMap : 注册自身到 abilityMap_
                             : 启动 IPC 监听
                             : 服务就绪
```

### SA 生命周期状态机

```
                    ┌─────────────┐
                    │    INIT     │
                    └──────┬──────┘
                           │ Load request
                           ▼
                    ┌─────────────┐
         ┌─────────│  STARTING   │─────────┐
         │         └──────┬──────┘         │
         │ Timeout        │ Success        │ Fail
         ▼                ▼                ▼
   ┌───────────┐   ┌───────────┐    ┌───────────┐
   │ LOAD_FAIL │   │  STARTED  │    │ LOAD_FAIL │
   └───────────┘   └─────┬─────┘    └───────────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
        ┌─────────┐ ┌─────────┐ ┌─────────┐
        │  IDLE   │ │ ACTIVE  │ │ STOPPED │
        └─────────┘ └─────────┘ └─────────┘
```

## 分布式架构

```mermaid
graph TB
    subgraph DeviceA[设备 A]
        SAM_A[Samgr]
        SA_A1[SA #1]
        SA_A2[SA #2]
        DBinder_A[DBinder Service]
    end
    
    subgraph DeviceB[设备 B]
        SAM_B[Samgr]
        SA_B1[SA #10]
        DBinder_B[DBinder Service]
    end
    
    subgraph SoftBus[SoftBus 网络层]
        Discovery[设备发现]
        Networking[网络传输]
    end
    
    SAM_A <-->|本地| SA_A1
    SAM_A <-->|本地| SA_A2
    SAM_B <-->|本地| SA_B1
    
    SAM_A <-->|分布式发现| SAM_B
    SAM_A <-->|RPC| DBinder_A
    SAM_B <-->|RPC| DBinder_B
    DBinder_A <-->|SoftBus| SoftBus
    DBinder_B <-->|SoftBus| SoftBus
```

## 模块交互

```mermaid
flowchart LR
    subgraph Client[客户端]
        C1[GetSA]
        C2[Subscribe]
        C3[LoadSA]
    end
    
    subgraph Core[核心模块]
        SAM[SystemAbilityManager]
        Stub[Stub]
        Proxy[Proxy]
    end
    
    subgraph Extension[扩展模块]
        State[StateScheduler]
        Collect[CollectManager]
        DFX[HiSysEvent]
    end
    
    subgraph Storage[存储]
        AM[(abilityMap)]
        LM[(listenerMap)]
        OD[(onDemandMap)]
    end
    
    C1 --> Proxy
    C2 --> Proxy
    C3 --> Proxy
    Proxy <--> Stub
    Stub --> SAM
    SAM <--> State
    SAM <--> Collect
    SAM <--> DFX
    SAM <--> AM
    SAM <--> LM
    SAM <--> OD
```
