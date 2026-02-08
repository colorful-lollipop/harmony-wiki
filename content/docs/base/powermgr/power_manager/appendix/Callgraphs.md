# 关键调用链

本文档描述 Power Manager 模块的关键调用链路，包括从应用到底层的完整流程。

## 1. 电源控制调用链

### 1.1 关机流程

```mermaid
sequenceDiagram
    participant App as 应用 (JS)
    participant NAPI as N-API Layer
    participant IPC as ZIDL IPC
    participant Svc as PowerMgrService
    participant SM as StateMachine
    participant Ctrl as ShutdownController
    participant HDI as Power HDI
    participant Kernel as Kernel

    App->>NAPI: power.shutdown('user')
    NAPI->>NAPI: 参数校验 (PowerNapi::Shutdown)
    NAPI->>IPC: ShutdownDevice(reason)
    IPC->>IPC: 序列化数据
    IPC->>Svc: OnRemoteRequest(Shutdown)
    Svc->>Svc: 权限检查 (Permission::Check)
    Svc->>SM: RequestState(SHUTTING_DOWN)
    SM->>SM: 状态机转换
    SM->>Ctrl: OnShutdown()
    Ctrl->>Ctrl: 显示关机对话框
    Ctrl->>Ctrl: 确认关机
    Ctrl->>HDI: Shutdown()
    HDI->>Kernel: ioctl(POWER_SHUTDOWN)
    Kernel-->>HDI: 执行完成
    HDI-->>Ctrl: 完成
    Ctrl-->>Svc: 完成
    Svc-->>IPC: 完成
    IPC-->>NAPI: Promise resolve
    NAPI-->>App: 完成
```

### 1.2 唤醒流程

```mermaid
sequenceDiagram
    participant HW as 硬件事件
    participant HDI as Power HDI
    participant Svc as PowerMgrService
    participant SM as StateMachine
    participant Disp as DisplayManager
    participant App as 应用

    HW->>HDI: 唤醒中断
    HDI->>Svc: OnWakeup(wakeupReason)
    Svc->>Svc: 广播 WAKEUP 事件
    Svc->>SM: RequestState(AWAKE)
    SM->>SM: 状态机转换
    SM->>Disp: DisplayOn()
    Disp-->>SM: 完成
    SM-->>Svc: 完成
    Svc->>App: PowerStateCallback(ACTIVE)
```

---

## 2. 运行锁调用链

### 2.1 创建运行锁

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant IPC as ZIDL IPC
    participant Svc as PowerMgrService
    participant LockMgr as RunningLockMgr
    participant Lock as RunningLockEntity
    participant SM as StateMachine

    App->>NAPI: runningLock.create('task', BACKGROUND)
    NAPI->>NAPI: 参数校验
    NAPI->>NAPI: 获取调用方 UID/PID
    NAPI->>IPC: CreateRunningLock(name, type)
    IPC->>Svc: OnCreateRunningLock()
    Svc->>LockMgr: CreateRunningLock()
    LockMgr->>LockMgr: 检查类型支持 (IsSupported)
    LockMgr->>LockMgr: 检查数量限制
    LockMgr->>Lock: 创建 RunningLockEntity
    Lock->>Lock: 设置名称、类型、UID
    LockMgr->>Svc: 返回 Lock
    Svc->>IPC: 返回 Lock token
    IPC->>NAPI: 返回 RunningLock 实例
    NAPI-->>App: RunningLock 对象
```

### 2.2 加锁/解锁

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant IPC as ZIDL IPC
    participant Svc as PowerMgrService
    participant Lock as RunningLockEntity
    participant SM as StateMachine

    App->>NAPI: lock.hold(timeout)
    NAPI->>IPC: HoldLock(lockId, timeout)
    IPC->>Svc: OnHoldLock()
    Svc->>Lock: Hold(timeout)
    Lock->>Lock: 增加引用计数
    Lock->>Svc: 返回
    Svc->>SM: AcquireWakeLock()
    SM->>SM: 阻止休眠
    SM-->>Svc: 完成
    Svc-->>IPC: 完成
    IPC-->>NAPI: true
    NAPI-->>App: 加锁成功

    Note over App,SM: 业务逻辑执行

    App->>NAPI: lock.unhold()
    NAPI->>IPC: UnholdLock(lockId)
    IPC->>Svc: OnUnholdLock()
    Svc->>Lock: Unhold()
    Lock->>Lock: 减少引用计数
    Lock->>SM: ReleaseWakeLock()
    SM->>SM: 允许休眠
```

---

## 3. 状态机调用链

### 3.1 状态转换流程

```mermaid
stateDiagram-v2
    [*] --> AWAKE: 开机

    AWAKE --> INACTIVE: 屏幕超时
    AWAKE --> DOZING: 低功耗显示
    AWAKE --> SHUTTING_DOWN: 关机请求
    AWAKE --> SUSPENDING: 手动挂起

    INACTIVE --> AWAKE: 用户活动
    INACTIVE --> SUSPENDING: 挂起超时
    INACTIVE --> SHUTTING_DOWN: 关机请求

    DOZING --> SUSPEND: 灭屏
    DOZING --> AWAKE: 用户活动

    SUSPENDING --> SUSPEND: 准备完成
    SUSPENDING --> AWAKE: 中断

    SUSPEND --> AWAKE: 唤醒事件
    SUSPEND --> SLEEPING: 深度休眠

    SLEEPING --> AWAKE: 唤醒事件

    SHUTTING_DOWN --> SHUTDOWN: 关机完成
    SHUTTING_DOWN --> AWAKE: 取消关机
```

### 3.2 状态机事件处理

```cpp
// services/native/src/power_state_machine.cpp

class PowerStateMachine {
    void HandleEvent(PowerStateEvent event) {
        switch (currentState_) {
            case PowerState::AWAKE:
                HandleAwakeEvent(event);
                break;
            case PowerState::INACTIVE:
                HandleInactiveEvent(event);
                break;
            case PowerState::SUSPENDING:
                HandleSuspendingEvent(event);
                break;
            case PowerState::SUSPEND:
                HandleSuspendEvent(event);
                break;
            case PowerState::SHUTTING_DOWN:
                HandleShuttingDownEvent(event);
                break;
        }
    }
};
```

---

## 4. 回调通知调用链

### 4.1 状态变化回调

```mermaid
sequenceDiagram
    participant Svc as PowerMgrService
    participant CB as CallbackRegistry
    participant IPC as ZIDL IPC
    participant NAPI as N-API
    participant App as 应用

    Svc->>Svc: 状态变化 (AWAKE -> SUSPEND)
    Svc->>CB: NotifyStateChange(newState)
    CB->>CB: 遍历注册回调
    loop 每个->>IPC: SendCallback(callbackId, state)
        IPC->>IPC: 序列化
        IPC->>NAPI: on回调
        CBStateChanged(state)
        NAPI->>App: JS 回调
    end
```

### 4.2 关机回调

```mermaid
sequenceDiagram
    participant HW as 硬件
    participant Svc as PowerMgrService
    participant CB as ShutdownCallback
    participant IPC as ZIDL IPC
    participant App as 应用

    HW->>Svc: 关机中断
    Svc->>Svc: 开始关机流程
    Svc->>CB: NotifyShutdown(reason)
    CB->>CB: 遍历回调
    loop 每个回调
        CB->>IPC: OnShutdownCallback(type, reason)
        IPC->>App: 调用 JS 回调
    end
    Svc->>HDI: 执行关机
```

---

## 5. IPC 调用时序

### 5.1 同步 IPC

```mermaid
sequenceDiagram
    participant C as Client
    participant P as Proxy
    participant B as Binder
    participant S as Stub
    participant Svc as Service

    C->>P: SyncMethod()
    P->>P: Write parameters
    P->>B: Transact(WRITE)
    B->>S: Dispatch
    S->>Svc: ActualMethod()
    Svc-->>S: Result
    S->>S: Write reply
    S-->>B: Reply
    B-->>P: Reply
    P->>P: Read result
    P-->>C: Return
```

### 5.2 异步 IPC

```mermaid
sequenceDiagram
    participant C as Client
    participant P as Proxy
    participant B as Binder
    participant S as Stub
    participant Svc as Service
    participant CB as Callback

    C->>P: AsyncMethod(params, callback)
    P->>P: Register callback
    P->>B: Transact(ASYNC)
    B->>S: Dispatch
    S->>Svc: ActualMethod()
    Svc-->>S: Result
    S->>S: Write reply
    S-->>B: Reply
    B-->>P: Reply
    P->>B: Post callback
    B->>CB: Invoke callback
    CB-->>C: onResult(result)
```

---

## 6. 关键代码路径

### 6.1 关机路径

| 层级 | 文件 | 函数 |
|------|------|------|
| N-API | `frameworks/napi/power/power_napi.cpp` | `PowerNapi::Shutdown()` |
| IPC | `services/zidl/src/...` | `PowerMgrProxyAsync::Shutdown()` |
| Service | `services/native/src/shutdown/shutdown_controller.cpp` | `ShutdownController::Shutdown()` |
| HDI | `drivers_interface_power` | `PowerHdi::Shutdown()` |

### 6.2 运行锁路径

| 层级 | 文件 | 函数 |
|------|------|------|
| N-API | `frameworks/napi/runninglock/runninglock_napi.cpp` | `RunningLockNapi::Create()` |
| Service | `services/native/src/runninglock/running_lock_mgr.cpp` | `RunningLockMgr::CreateRunningLock()` |
| State | `services/native/src/power_state_machine.cpp` | `PowerStateMachine::AcquireWakeLock()` |

### 6.3 状态机路径

| 层级 | 文件 | 函数 |
|------|------|------|
| Core | `services/native/src/power_state_machine.cpp` | `PowerStateMachine::HandleEvent()` |
| State | `services/native/src/power_state_machine.cpp` | `PowerStateMachine::TransitionTo()` |
| Action | `services/native/src/actions/...` | `DeviceStateAction::...()` |

---

## 7. 线程模型

### 7.1 线程切换

```
JS 线程
   │
   ▼ (AsyncWork)
FFRT 线程池
   │
   ▼ (IPC)
Binder 线程
   │
   ▼ (OnRemoteRequest)
主线程 (Service)
   │
   ▼ (Dispatch)
工作线程
   │
   ▼ (IOCTL)
HDI 线程
```

### 7.2 同步点

| 场景 | 同步机制 |
|------|----------|
| 状态机转换 | `std::mutex` |
| RunningLock 计数 | `std::recursive_mutex` |
| 回调注册 | `std::shared_mutex` |
| 配置读取 | `std::once_flag` |
