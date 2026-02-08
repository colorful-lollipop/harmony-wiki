# 架构说明

> 组件图、数据流、线程模型、关键时序

---

## 整体架构

```mermaid
graph TB
    subgraph "应用层"
        A[应用代码 JS/TS]
        B[应用代码 C/C++]
        C[应用代码 Cangjie]
    end

    subgraph "接口层 (Framework)"
        D[N-API
        @ohos.systemTime
        @ohos.systemTimer]
        E[C API
        time_service.h]
        F[Cangjie FFI]
    end

    subgraph "客户端 (Client)"
        G[TimeServiceClient
        单例模式
        死亡监听]
    end

    subgraph "IPC (Binder)"
        H[ITimeService
        IDL接口
        TimeServiceProxy
        TimeServiceStub]
    end

    subgraph "服务端 (Service)"
        I[TimeSystemAbility
        SA_ID: 3702
        OnStart/OnStop]
        J[TimerManager
        定时器核心
        独立线程]
        K[TimeZoneInfo
        时区管理]
        L[NtpTrustedTime
        NTP时间同步]
    end

    subgraph "内核"
        M[timerfd
        epoll
        alarm]
        N[RTC时钟
        /dev/rtc0]
    end

    A --> D
    B --> E
    C --> F
    D --> G
    E --> G
    F --> G
    G -->|GetProxy| H
    H -->|IPC调用| I
    I --> J
    I --> K
    I --> L
    J -->|epoll_wait| M
    K -->|settimeofday| N
```

---

## 组件职责

### 1. 接口层 (Framework)

| 组件 | 路径 | 职责 |
|------|------|------|
| **N-API 模块** | `framework/js/napi/` | JS/TS API 到 C++ 的绑定 |
| **C API** | `interfaces/kits/c/` | C/C++ 原生 API |
| **Inner API** | `interfaces/inner_api/` | C++ 内部客户端接口 |

### 2. 客户端 (Client)

**TimeServiceClient**
- 位置：`interfaces/inner_api/include/time_service_client.h`
- 模式：单例（线程安全）
- 职责：
  - 获取 `ITimeService` 代理
  - 监听服务端死亡
  - SA 状态变更监听
  - 定时器恢复信息管理

### 3. 服务端 (Service)

**TimeSystemAbility**
- 位置：`services/time_system_ability.cpp`
- SA_ID：3702（定义在 `services/profile/3702.json`）
- 继承：`SystemAbility` + `TimeServiceStub`
- 生命周期：`OnStart()` → `OnStop()`

**核心管理器**：

| 管理器 | 模式 | 职责 |
|--------|------|------|
| TimerManager | 单例 | 定时器生命周期管理 |
| TimerProxy | 单例 | 应用后台冻结时代理 |
| TimeZoneInfo | 单例 | 时区设置/查询 |
| NtpTrustedTime | 单例 | NTP 时间同步 |
| TimeServiceNotify | 单例 | 时间变更事件广播 |

---

## 数据流

### 场景 1: 设置系统时间

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant Client as TimeServiceClient
    participant IPC as IPC (Binder)
    participant Service as TimeSystemAbility
    participant Kernel as 内核

    App->>NAPI: systemTime.setTime(time)
    NAPI->>NAPI: 参数解析
    NAPI->>NAPI: 创建异步任务
    NAPI->>Client: TimeServiceClient::SetTime()
    Client->>Client: GetProxy()
    Client->>IPC: ITimeService::SetTime()
    IPC->>Service: 跨进程调用
    Service->>Service: 权限检查(CheckCallingPermission)
    Service->>Service: 校验时间有效性
    Service->>Kernel: settimeofday()
    Service->>Kernel: ioctl(RTC_SET_TIME)
    Service->>Service: 发布时间变更事件
    Service-->>IPC: 返回结果
    IPC-->>Client: 返回结果
    Client-->>NAPI: 返回错误码
    NAPI-->>App: Promise/Callback 回调
```

### 场景 2: 创建定时器

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant Client as TimeServiceClient
    participant IPC as IPC (Binder)
    participant Service as TimeSystemAbility
    participant TimerMgr as TimerManager
    participant Handler as TimerHandler

    App->>NAPI: systemTimer.createTimer(options)
    NAPI->>NAPI: 解析TimerOptions
    NAPI->>NAPI: 校验参数类型/范围
    NAPI->>Client: CreateTimerV9(timerOptions, timerId)
    Client->>IPC: ITimeService::CreateTimer()
    IPC->>Service: 跨进程调用
    Service->>Service: 权限检查
    Service->>Service: ParseTimerPara
    Service->>Service: CheckTimerPara
    Service->>TimerMgr: CreateTimer(paras, callback)
    TimerMgr->>TimerMgr: 生成 timerId
    TimerMgr->>TimerMgr: 保存定时器信息
    TimerMgr->>TimerMgr: 插入Batch
    TimerMgr->>Handler: SetTimer(wakeupTime)
    Handler->>Handler: timerfd_settime()
    TimerMgr-->>Service: 返回 timerId
    Service-->>IPC: 返回结果
    IPC-->>Client: 返回 timerId
    Client-->>NAPI: 返回结果
    NAPI-->>App: Promise resolve(timerId)
```

### 场景 3: 定时器触发回调

```mermaid
sequenceDiagram
    participant Kernel as 内核(timerfd)
    participant Handler as TimerHandler
    participant TimerMgr as TimerManager
    participant Service as TimeSystemAbility
    participant IPC as IPC (Binder)
    participant Client as TimerCallbackProxy
    participant App as 应用

    Kernel->>Handler: epoll_wait 唤醒
    Handler->>Handler: read(timerfd)
    Handler->>TimerMgr: 回调 OnTrigger
    TimerMgr->>TimerMgr: 查找定时器
    TimerMgr->>TimerMgr: 计算下次触发时间
    alt repeat=true
        TimerMgr->>Handler: 重新设置 timerfd
    else repeat=false
        TimerMgr->>TimerMgr: 移除定时器
    end
    TimerMgr->>Service: callbackFunc(timerId)
    Service->>IPC: ITimerCallback::NotifyTimer()
    IPC->>Client: 跨进程回调
    Client->>App: WantAgent 触发或 JS 回调
```

---

## 线程模型

### TimerManager 线程结构

```mermaid
graph TB
    subgraph "TimerManager 线程"
        A[alarmThread_] --> B[TimerLooper 主循环]
        B --> C[epoll_wait]
        C -- 唤醒 --> D[处理到期定时器]
        D --> E[回调触发]
        E --> F[计算下次时间]
        F --> G[重新设置 alarm]
        G --> C
    end

    subgraph "主线程"
        H[CreateTimer]
        I[StartTimer]
        J[StopTimer]
        K[DestroyTimer]
    end

    H -- 加锁 --> L[mutex_]
    I -- 加锁 --> L
    J -- 加锁 --> L
    K -- 加锁 --> L
    L -- 条件变量 --> C
```

**关键实现**（`services/timer/src/timer_manager.cpp`）：

```cpp
// TimerManager 使用独立的线程运行定时器主循环
std::unique_ptr<std::thread> alarmThread_;  // 在 Init() 中创建
std::atomic_bool runFlag_;                  // 控制线程退出
std::mutex mutex_;                          // 保护定时器数据结构
std::condition_variable cv_;                // 唤醒 epoll_wait

// 线程入口
void TimerManager::TimerLooper() {
    while (runFlag_) {
        // 1. 计算最近到期时间
        // 2. 设置 timerfd
        // 3. epoll_wait 等待
        // 4. 处理到期定时器
        // 5. 回调触发
    }
}
```

### 其他线程

| 线程 | 触发条件 | 职责 |
|------|----------|------|
| Init 重试线程 | OnStart 失败 | 延迟重试初始化 |
| 事件重试线程 | 订阅事件失败 | 延迟重试订阅 |
| SNTP 线程 | NTP 同步 | 网络时间查询（异步） |
| HIDumper 线程 | 命令执行 | 诊断信息输出 |

---

## IPC 接口定义

### IDL 文件

**ITimeService.idl** (`services/ITimeService.idl`):

```idl
interface ITimeService {
    // 时间设置
    int32 SetTime([in] int64_t time, [in] int8_t apiVersion);
    int32 SetTimeZone([in] String timeZoneId, [in] int8_t apiVersion);
    int32 GetTimeZone([out] String timeZoneId);

    // 定时器管理
    int32 CreateTimer([in] String name, [in] int type, ...);
    int32 StartTimer([in] uint64_t timerId, [in] uint64_t triggerTime);
    int32 StopTimer([in] uint64_t timerId);
    int32 DestroyTimer([in] uint64_t timerId);

    // 定时器代理
    int32 ProxyTimer([in] int32 uid, [in] List<int> pidList, ...);
    int32 AdjustTimer([in] bool isAdjust, [in] uint32 interval, ...);

    // 时间获取
    int32 GetNtpTimeMs([out] int64_t time);
    int32 GetRealTimeMs([out] int64_t time);
};
```

**ITimerCallback.idl** (`services/ITimerCallback.idl`):

```idl
[callback] interface ITimerCallback {
    [oneway] int32 NotifyTimer([in] uint64_t timerId);
};
```

### IPC 通信流程

```
客户端进程                          服务端进程 (timeservice)
------------                        ------------------------
TimeServiceClient                           |
      |                                     |
GetProxy()                                  |
      |                                     |
      v                                     v
SystemAbilityManagerClient  ---->  SystemAbilityManager
      |                                     |
      |  GetSystemAbility(3702)             |
      |                                     |
      <------------------------ sptr<IRemoteObject>
      |                                     |
      v                                     v
TimeServiceProxy                  TimeServiceStub
      |                                     |
      |-------- IPC (Binder) -------------->|
      |                                     |
      |                                     v
      |                            TimeSystemAbility
      |                                     |
      <-------- 返回结果 ------------------|
```

---

## 模块依赖关系

```mermaid
graph LR
    subgraph "客户端"
        A[TimeServiceClient]
    end

    subgraph "IPC"
        B[ITimeService]
        C[ITimerCallback]
    end

    subgraph "服务端"
        D[TimeSystemAbility]
        E[TimerManager]
        F[TimeZoneInfo]
        G[NtpTrustedTime]
        H[TimerProxy]
        I[TimerHandler]
    end

    subgraph "外部依赖"
        J[WantAgent]
        K[CommonEvent]
        L[PowerManager]
        M[AccountManager]
    end

    A -- 使用 --> B
    B -- 实现 --> D
    D -- 管理 --> E
    D -- 管理 --> F
    D -- 管理 --> G
    E -- 使用 --> H
    E -- 使用 --> I
    E -- 回调 --> C
    D -- 监听 --> J
    D -- 订阅 --> K
    D -- 监听 --> L
    D -- 监听 --> M
```

---

## 相关链接

- [目录结构](./01_Directory_Structure.md) - 源码组织
- [内部 API](./04_Inner_API.md) - C++ 接口详情
- [N-API 参考](./03_NAPI_Reference.md) - JS 接口详情
