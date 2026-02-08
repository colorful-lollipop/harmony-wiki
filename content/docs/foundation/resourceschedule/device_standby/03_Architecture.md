# 架构说明

## 整体架构图

```mermaid
graph TB
    subgraph "应用层"
        JS["JS 应用"]
        ArkTS["ArkTS 应用"]
    end

    subgraph "接口层"
        NAPI["N-API\n(interfaces/kits/napi)"]
        Taihe["Taihe/ArkTS API\n(interfaces/kits/ani)"]
        InnerKits["Inner Kits\n(interfaces/innerkits)"]
    end

    subgraph "服务层"
        SSC["StandbyServiceClient"]
        SS["StandbyService"]
        SSI["StandbyServiceImpl"]
    end

    subgraph "插件层"
        SM["State Manager\n状态管理"]
        LM["Listener Manager\n消息监听"]
        STM["Strategy Manager\n策略执行"]
    end

    subgraph "状态机"
        WS["Working State"]
        SSleep["Sleep State"]
        NS["Nap State"]
        DS["Dark State"]
        MS["Maintenance State"]
    end

    subgraph "系统服务"
        AppMgr["App Manager"]
        BundleMgr["Bundle Manager"]
        PowerMgr["Power Manager"]
        BatteryMgr["Battery Manager"]
    end

    JS --> NAPI
    ArkTS --> Taihe
    NAPI --> SSC
    Taihe --> SSC
    SSC --> SS
    SS --> SSI
    SSI --> SM
    SM --> WS
    SM --> SSleep
    SM --> NS
    SM --> DS
    SM --> MS
    SSI --> LM
    SSI --> STM
    SSI --> AppMgr
    SSI --> BundleMgr
    SSI --> PowerMgr
```

## 数据流

### 豁免申请数据流

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API层
    participant Client as StandbyServiceClient
    participant Service as StandbyServiceImpl
    participant Plugin as Plugin层
    participant Sys as 系统服务

    App->>NAPI: requestExemptionResource()
    NAPI->>NAPI: 参数校验
    NAPI->>Client: ApplyAllowResource()
    Client->>Service: IPC调用
    Service->>Service: CheckCallerPermission()
    alt 权限校验失败
        Service-->>App: ERR_PERMISSION_ERROR
    else 权限校验通过
        Service->>Service: 参数校验
        Service->>Service: UpdateRecord()
        Service->>Plugin: NotifyAllowListChanged()
        Service->>Sys: 注册资源限制
        Service-->>App: ERR_OK
    end
```

## 状态机

### 设备状态转换

```mermaid
stateDiagram-v2
    [*] --> Working: 设备启动

    Working --> Sleep: 长时间无操作
    Working --> Nap: 短时间休息
    Working --> Dark: 进入暗色模式

    Nap --> Working: 用户交互
    Nap --> Sleep: 休息超时

    Dark --> Working: 用户交互
    Dark --> Sleep: 暗色超时

    Sleep --> Maintenance: 系统维护
    Maintenance --> Working: 维护完成
    Maintenance --> Sleep: 维护超时

    Sleep --> Working: 任意用户交互
    Sleep --> Nap: 快速唤醒
```

### 状态管理职责

| 状态 | 进入条件 | 退出条件 | 限制策略 |
|------|----------|----------|----------|
| `working` | 设备启动/用户活跃 | 检测到空闲 | 无限制 |
| `sleep` | 空闲超时 | 用户交互 | 网络/Timer限制 |
| `nap` | 短时休息 | 用户交互/超时 | 宽松限制 |
| `dark` | 暗色模式 | 用户交互 | 低功耗模式 |
| `maintenance` | 系统维护 | 维护完成 | 严格限制 |

## 线程模型

### 核心线程

| 线程 | 职责 | 备注 |
|------|------|------|
| `Main Thread` | 服务初始化、IPC 响应 | SA 主线程 |
| `Event Handler` | 事件处理、状态机驱动 | 独立 EventRunner |
| `Plugin Thread` | 插件回调处理 | 根据配置 |

### 线程安全机制

| 保护对象 | 机制 | 文件 |
|----------|------|------|
| `allowRecordMutex_` | 互斥锁 | `standby_service_impl.h` |
| `backupRestoreMutex_` | 互斥锁 | `standby_service_impl.h` |
| `appStateObserverMutex_` | 互斥锁 | `standby_service_impl.h` |
| `eventObserverMutex_` | 互斥锁 | `standby_service_impl.h` |
| `timerObserverMutex_` | 递归互斥锁 | `standby_service_impl.h` |

## IPC 通信

### SA 注册信息

| 配置项 | 值 |
|--------|-----|
| SA ID | 1914 |
| 进程 | resource_schedule_service |
| 库路径 | libstandby_service.z.so |
| 启动方式 | 启动时加载 (run-on-create) |

### IPC 接口

主要通过 `IStandbyService` 接口进行通信，实现类为 `StandbyServiceImpl`。

```cpp
// 接口定义位置
frameworks/include/istandby_service.h  // TODO: 待确认实际路径

// 主要方法
ErrCode ApplyAllowResource(ResourceRequest& request);
ErrCode UnapplyAllowResource(ResourceRequest& request);
ErrCode GetAllowList(uint32_t allowType, std::vector<AllowInfo>& list, uint32_t reasonCode);
ErrCode IsDeviceInStandby(bool& isStandby);
ErrCode SubscribeStandbyCallback(const sptr<IStandbyServiceSubscriber>& subscriber);
ErrCode UnsubscribeStandbyCallback(const sptr<IStandbyServiceSubscriber>& subscriber);
```

## 关键时序图

### 初始化时序

```mermaid
sequenceDiagram
    participant SAMgr as SAMgr
    participant SS as StandbyService
    participant SSI as StandbyServiceImpl
    participant Plugin as Plugin Layer
    participant Observer as State Observer

    SAMgr->>SS: OnStart()
    SS->>SSI: Init()
    SSI->>Observer: RegisterAppStateObserver()
    SSI->>Observer: RegisterCommEventObserver()
    SSI->>Observer: RegisterTimeObserver()
    SSI->>Plugin: RegisterPlugin()
    SSI->>Plugin: InitReadyState()
    SS-->>SAMgr: OnReady()
```

### 状态切换时序

```mermaid
sequenceDiagram
    participant Event as 事件源
    participant SSI as StandbyServiceImpl
    participant StateMgr as State Manager
    participant NewState as 新状态
    participant OldState as 旧状态
    participant Subscriber as 订阅者

    Event->>SSI: HandleEvent()
    SSI->>StateMgr: CheckStateTransition()
    StateMgr->>OldState: OnExit()
    OldState-->>Subscriber: 状态变更通知
    StateMgr->>NewState: OnEnter()
    NewState-->>Subscriber: 状态变更通知
```

## 稳定性标注

| 层级 | 稳定性 | 说明 |
|------|--------|------|
| N-API | 稳定 | 对外公开接口 |
| Taihe | 稳定 | ArkTS 官方接口 |
| Inner Kits | 较稳定 | 系统内部接口 |
| Plugin | 内部 | 插件内部接口 |
| 服务实现 | 内部 | 核心实现细节 |
