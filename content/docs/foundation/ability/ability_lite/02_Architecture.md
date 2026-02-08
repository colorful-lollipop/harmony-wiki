# 架构说明

## 目的

本文档描述 ability_lite 的架构设计，包括组件关系、数据流、线程模型和关键时序。

## 适用范围

- 系统架构师
- 需要理解内部实现的开发者

## 组件架构图

```mermaid
graph TB
    subgraph "Application Process"
        JS[JS Application]
        Native[Native Application]
        AKit[AbilityKit]
        AThread[AbilityThread]
        AScheduler[AbilityScheduler]
        AMClient[AbilityManager Client]
    end
    
    subgraph "Foundation Process"
        SAMGR[SAMGR Lite]
        AMS[AMS Service]
        AMFeature[AbilityMgrFeature]
        AMHandler[AbilityMgrHandler]
        AWorker[AbilityWorker]
        AppMgr[AppManager]
        StackMgr[AbilityStackManager]
    end
    
    subgraph "System Services"
        AppSpawn[AppSpawn Service]
        BundleMS[Bundle Manager]
        WMS[Window Manager]
        PMS[Permission Service]
    end
    
    JS -->|N-API| AKit
    Native -->|Ability| AKit
    AKit --> AThread
    AThread --> AScheduler
    AScheduler --> AMClient
    AMClient -->|IPC| SAMGR
    SAMGR --> AMFeature
    AMFeature --> AMHandler
    AMHandler --> AWorker
    AWorker -->|Task| StartTask[StartAbilityTask]
    AWorker -->|Task| StopTask[StopAbilityTask]
    StartTask --> AppMgr
    StartTask --> StackMgr
    AppMgr -->|Spawn| AppSpawn
    AppMgr -->|Query| BundleMS
    StackMgr -->|Window| WMS
    AppMgr -->|Permission| PMS
```

## 数据流图

### Ability 启动流程

```mermaid
sequenceDiagram
    participant App as Application
    participant AMClient as AbilityManager Client
    participant SAMGR as SAMGR Lite
    participant AMFeature as AbilityMgrFeature
    participant AMHandler as AbilityMgrHandler
    participant AWorker as AbilityWorker
    participant AppMgr as AppManager
    participant AppSpawn as AppSpawn
    participant Ability as New Ability
    
    App->>AMClient: StartAbility(want)
    AMClient->>SAMGR: GetFeatureApi(AMS)
    SAMGR-->>AMClient: IProxy
    AMClient->>SAMGR: SendRequest(START_ABILITY)
    SAMGR->>AMFeature: Invoke(START_ABILITY)
    AMFeature->>AMFeature: StartAbilityInvoke()
    AMFeature->>AMFeature: GetCallingUid()
    AMFeature->>AMHandler: ServiceMsgProcess()
    AMHandler->>AWorker: PostTask()
    AWorker->>AWorker: ability_start_task
    AWorker->>AppMgr: StartAbility()
    AppMgr->>AppSpawn: SpawnApplication()
    AppSpawn-->>AppMgr: pid
    AppMgr->>Ability: AttachBundle()
    Ability-->>AppMgr: ready
    AppMgr->>Ability: AbilityTransaction(ACTIVE)
    Ability-->>AppMgr: done
```

### Service Ability 连接流程

```mermaid
sequenceDiagram
    participant Client as Client App
    participant Server as Service Ability
    participant AMClient as AM Client
    participant AMS as AMS Service
    
    Client->>AMClient: ConnectAbility(want, conn)
    AMClient->>AMS: SendRequest(CONNECT_ABILITY)
    AMS->>AMS: ConnectAbilityInvoke()
    AMS->>Server: AttachBundle() [if not running]
    AMS->>Server: ConnectAbility()
    Server-->>AMS: SvcIdentity
    AMS->>Server: ConnectAbilityDone()
    Server-->>AMS: OnConnect(want)
    AMS-->>Client: conn->OnConnect()
```

## 线程模型

### Application Process 线程

```
┌─────────────────────────────────────────┐
│         Application Process             │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │      AbilityThread (Main)       │    │
│  │  - Looper/Handler message loop  │    │
│  │  - IPC message handling         │    │
│  │  - Lifecycle callback dispatch  │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │    AbilityScheduler (Handler)   │    │
│  │  - Receives AMS commands        │    │
│  │  - Dispatches to Ability        │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

### Foundation Process 线程

```
┌─────────────────────────────────────────┐
│        Foundation Process               │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │   SAMGR Lite (Service Thread)   │    │
│  │  - Service registration         │    │
│  │  - Feature API routing          │    │
│  │  - IPC request dispatch         │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │   AMS Task (Single Thread)      │    │
│  │  TaskConfig:                    │    │
│  │  - LEVEL_HIGH                   │    │
│  │  - PRI_NORMAL                   │    │
│  │  - Stack: AMS_TASK_STACK_SIZE   │    │
│  │  - Queue: 20 messages           │    │
│  │  - SINGLE_TASK mode             │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │   AbilityWorker (Task Queue)    │    │
│  │  - Sequential task execution    │    │
│  │  - Lifecycle task scheduling    │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

**AMS 任务配置** (`services/abilitymgr_lite/src/ability_mgr_service.cpp:82`):
```cpp
TaskConfig config = {
    LEVEL_HIGH,              // 高优先级
    PRI_NORMAL,              // 普通优先级
    AMS_TASK_STACK_SIZE,     // 可配置栈大小
    QUEUE_SIZE,              // 20 消息队列
    SINGLE_TASK              // 单线程模式
};
```

## 关键时序

### Page Ability 生命周期

```mermaid
stateDiagram-v2
    [*] --> UNINITIALIZED: Create
    UNINITIALIZED --> INITIAL: OnStart(want)
    INITIAL --> INACTIVE: OnInactive()
    INACTIVE --> ACTIVE: OnActive(want)
    ACTIVE --> INACTIVE: OnInactive()
    INACTIVE --> BACKGROUND: OnBackground()
    BACKGROUND --> ACTIVE: OnActive(want)
    BACKGROUND --> INITIAL: OnStop()
    INITIAL --> [*]: Destroy
```

**状态转换时序**:

```
时间 →

应用调用:
├─ StartAbility(want)
│   └─ AMS 处理
│       ├─ 创建 AppRecord (如需要)
│       ├─ 启动进程 (如需要)
│       ├─ AttachBundle
│       └─ 调度生命周期
│
Ability 回调:
├─ OnStart(want)        ← INITIAL 状态
├─ OnInactive()         ← INACTIVE 状态
├─ OnActive(want)       ← ACTIVE 状态 (前台)
├─ OnInactive()         ← INACTIVE 状态
├─ OnBackground()       ← BACKGROUND 状态
└─ OnStop()             ← INITIAL 状态
```

### Service Ability 生命周期

```
时间 →

连接阶段:
├─ ConnectAbility(want, conn)
│   └─ AMS 处理
│       ├─ 启动 Service (如需要)
│       ├─ Service->OnConnect(want)
│       └─ conn->OnConnect(SvcIdentity)
│
通信阶段:
├─ Client 通过 SvcIdentity 发送消息
├─ Service->MsgHandle(funcId, request, reply)
└─ Client 接收响应
│
断开阶段:
├─ DisconnectAbility(conn)
│   └─ AMS 处理
│       ├─ Service->OnDisconnect(want)
│       └─ conn->OnDisconnect()
```

## 核心类关系

### Ability 类层次

```mermaid
classDiagram
    class Ability {
        +OnStart(want)
        +OnActive(want)
        +OnInactive()
        +OnBackground()
        +OnStop()
        +OnConnect(want) SvcIdentity
        +OnDisconnect(want)
        +MsgHandle(funcId, request, reply)
    }
    
    class AbilityContext {
        +GetBundleName()
        +GetAbilityInfo()
    }
    
    class AbilitySlice {
        +OnStart(want)
        +OnActive(want)
        +OnInactive()
        +OnBackground()
        +OnStop()
    }
    
    class SliteAbility {
        +OnCreate()
        +OnDestroy()
    }
    
    AbilityContext <|-- Ability
    Ability "1" --> "*" AbilitySlice : manages
    AbilityContext <|-- SliteAbility
```

### AMS 服务类关系

```mermaid
classDiagram
    class AbilityMgrService {
        +GetInstance() AbilityMgrService
        -ServiceInitialize()
        -ServiceMessageHandle()
    }
    
    class AbilityMgrFeature {
        +StartAbility(want)
        +StopAbility(want)
        +ConnectAbility(want, svc, token)
        +DisconnectAbility(svc, token)
        +Invoke(funcId, origin, req)
    }
    
    class AbilityMgrHandler {
        +Init()
        +ServiceMsgProcess(request)
    }
    
    class AbilityWorker {
        +PostTask(task)
        +ProcessTask()
    }
    
    class AppManager {
        +StartAbility(want)
        +TerminateApp(bundleName)
    }
    
    class AbilityStackManager {
        +PushAbility(ability)
        +PopAbility()
        +GetTopAbility()
    }
    
    AbilityMgrService "1" --> "1" AbilityMgrFeature
    AbilityMgrFeature --> AbilityMgrHandler
    AbilityMgrHandler --> AbilityWorker
    AbilityWorker --> AppManager
    AbilityWorker --> AbilityStackManager
```

## IPC 机制

### SAMGR Lite IPC

ability_lite 使用 SAMGR (System Ability Manager) Lite 框架进行 IPC：

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │ ──▶  │   SAMGR     │ ──▶  │   Service   │
│             │      │             │      │             │
│ GetFeatureApi      │ Route to    │      │ Feature API │
│ SendRequest        │ Feature     │      │ Invoke()    │
└─────────────┘      └─────────────┘      └─────────────┘
```

**关键接口** (`interfaces/inner_api/abilitymgr_lite/ability_service_interface.h`):

```cpp
// 服务名和 Feature 名
const char AMS_SERVICE[] = "abilityms";
const char AMS_FEATURE[] = "AmsFeature";

// 命令枚举
enum AmsCommand {
    START_ABILITY = 0,
    TERMINATE_ABILITY,
    ATTACH_BUNDLE,
    CONNECT_ABILITY,
    DISCONNECT_ABILITY,
    STOP_ABILITY,
    // ...
};

// 服务接口
struct AmsInterface {
    INHERIT_SERVER_IPROXY;
    int32 (*StartAbility)(const Want *want);
    int32 (*TerminateAbility)(uint64_t token);
    int32 (*ConnectAbility)(const Want *want, SvcIdentity *svc, uint64_t token);
    int32 (*DisconnectAbility)(SvcIdentity *svc, uint64_t token);
    int32 (*StopAbility)(const Want *want);
};
```

### IPC 数据序列化

使用 `IpcIo` 进行数据序列化：

```cpp
// 发送请求
IpcIo io;
char ipcData[MAX_IO_SIZE];
IpcIoInit(&io, ipcData, MAX_IO_SIZE, 0);
WriteInt32(&io, command);
WriteString(&io, bundleName);
// ...
SendRequest(svc, command, &io, &reply, option, NULL);

// 接收请求
IpcIo *req = ...;
int32 command = ReadInt32(req);
const char *bundleName = ReadString(req);
```

## 信任边界

```
┌─────────────────────────────────────────────────────────┐
│  Trust Zone: System (Foundation Process)                │
│  - AMS Service                                          │
│  - SAMGR                                                │
│  - AppSpawn                                             │
│  高权限，可创建进程、管理权限                            │
└─────────────────────────────────────────────────────────┘
                           │ IPC (受控)
                           ▼
┌─────────────────────────────────────────────────────────┐
│  Trust Zone: Application                                │
│  - AbilityKit                                           │
│  - Application Code                                     │
│  沙箱环境，受限权限                                      │
└─────────────────────────────────────────────────────────┘
```

**边界检查点**:
1. `GetCallingUid()` - 验证调用者身份 (`ability_mgr_feature.cpp:149`)
2. `GetCallingPid()` - 验证进程身份 (`ability_mgr_feature.cpp:322`)
3. `LoadPermissions()` - 加载应用权限 (`app_record.cpp:60`)

## 相关链接

- [项目概览](00_Overview.md)
- [目录结构](01_Directory_Structure.md)
- [内部 API](05_Inner_API.md)
- [安全评估](08_Security_Assessment.md)
