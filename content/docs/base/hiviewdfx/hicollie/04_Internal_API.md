# HiCollie 内部 API 文档

> 模块接口、依赖方向、稳定性标识

---

## 目的与适用范围

### 文档目的
本文档描述 HiCollie 内部模块的 API 接口、依赖关系和稳定性分级，帮助模块开发者理解和扩展代码。

### 适用场景
- 🏗️ **模块开发** - 理解内部模块接口
- 🔧 **功能扩展** - 在正确位置添加新 API
- 🔍 **代码维护** - 理解模块依赖方向

---

## 模块接口清单

### 1. Watchdog 内部接口

#### 1.1 核心调度器 (WatchdogInner)

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `AddThread` | `int AddThread(name, handler, callback, interval, priority)` | 稳定 | 添加 Handler 监控线程 |
| `RunOneShotTask` | `void RunOneShotTask(name, task, delay)` | 稳定 | 执行一次性任务 |
| `RunPeriodicalTask` | `void RunPeriodicalTask(name, task, interval, delay)` | 稳定 | 执行周期性任务 |
| `RunXCollieTask` | `int64_t RunXCollieTask(name, timeout, func, arg, flag)` | 稳定 | 添加 XCollie 超时任务 |
| `RemoveXCollieTask` | `void RemoveXCollieTask(id)` | 稳定 | 移除 XCollie 任务 |
| `AddIpcFull` | `bool AddIpcFull(interval, flag, func, arg)` | 稳定 | 添加 IPC Full 检测 |
| `AsyncBinderSpaceFull` | `bool AsyncBinderSpaceFull(interval, count, flag, func, arg)` | 条件 | 添加异步 Binder 空间检测 |
| `SetTimerCountTask` | `int64_t SetTimerCountTask(name, timeLimit, countLimit)` | 稳定 | 设置计数器任务 |
| `TriggerTimerCountTask` | `void TriggerTimerCountTask(name, bTrigger, message)` | 稳定 | 触发计数器 |
| `StopWatchdog` | `void StopWatchdog()` | 稳定 | 停止看门狗线程 |
| `InitFfrtWatchdog` | `void InitFfrtWatchdog()` | 稳定 | 初始化 FFRT 看门狗 |
| `SetBundleInfo` | `void SetBundleInfo(bundleName, bundleVersion)` | 稳定 | 设置 Bundle 信息 |
| `SetSystemApp` | `void SetSystemApp(isSystemApp)` | 稳定 | 设置是否系统应用 |
| `SetForeground` | `void SetForeground(isForeground)` | 稳定 | 设置前后台状态 |
| `SetAppDebug` | `void SetAppDebug(isAppDebug)` | 稳定 | 设置是否调试模式 |
| `RemoveInnerTask` | `void RemoveInnerTask(name)` | 稳定 | 移除内部任务 |
| `InitMainLooperWatcher` | `void InitMainLooperWatcher(beginFunc, endFunc)` | 稳定 | 初始化主循环监听器 |
| `SetEventConfig` | `int SetEventConfig(paramsMap)` | 稳定 | 设置事件配置 |
| `ConfigEventPolicy` | `int ConfigEventPolicy(paramsMap)` | 稳定 | 配置事件策略 |
| `SetSpecifiedProcessName` | `void SetSpecifiedProcessName(name)` | 稳定 | 设置指定进程名 |
| `SetScrollState` | `void SetScrollState(isScroll)` | 稳定 | 设置滚动状态 |
| `StartSample` | `void StartSample(duration, interval)` | 稳定 | 开始采样 |
| `StopSample` | `std::string StopSample(sampleCount)` | 稳定 | 停止采样 |
| `GetSamplerResult` | `SamplerResult GetSamplerResult()` | 稳定 | 获取采样结果 |
| `GetReservedTimeForLogging` | `int32_t GetReservedTimeForLogging()` | 稳定 | 获取预留时间 |

**证据**: `watchdog_inner.h:44-95` - WatchdogInner 类声明

#### 1.2 辅助接口

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `WriteStringToFile` | `static bool WriteStringToFile(pid, str)` | 内部 | 写字符串到文件 |
| `FfrtCallback` | `static void FfrtCallback(taskId, taskInfo, delayedTaskCount)` | 内部 | FFRT 回调 |
| `SendFfrtEvent` | `static void SendFfrtEvent(msg, eventName, taskInfo, faultTimeStr, isDumpStack)` | 内部 | 发送 FFRT 事件 |
| `LeftTimeExitProcess` | `static void LeftTimeExitProcess(description)` | 内部 | 超时退出进程 |
| `KillPeerBinderProcess` | `static void KillPeerBinderProcess(description)` | 内部 | 杀死对端 Binder 进程 |

**证据**: `watchdog_inner.h:64-70` - 静态辅助函数声明

---

### 2. XCollie 接口

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `SetTimer` | `int SetTimer(name, timeout, func, arg, flag)` | 稳定 | 设置定时器 |
| `CancelTimer` | `void CancelTimer(id)` | 稳定 | 取消定时器 |
| `SetTimerCount` | `int SetTimerCount(name, timeLimit, countLimit)` | 稳定 | 设置计数器 |
| `TriggerTimerCount` | `void TriggerTimerCount(name, bTrigger, message)` | 稳定 | 触发计数器 |

**证据**: `xcollie.h:28-51` - XCollie 类声明

---

### 3. IpcFull 接口

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `AddIpcFull` | `bool AddIpcFull(interval, flag, func, arg)` | 稳定 | 添加 IPC Full 检测 |
| `AsyncBinderSpaceFull` | `bool AsyncBinderSpaceFull(interval, count, flag, func, arg)` | 条件 | 异步 Binder 空间满检测 |

**证据**: `ipc_full.h:28-33` - IpcFull 类声明

---

### 4. Watchdog 对外 C++ 接口

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `AddThread` | `int AddThread(name, handler, callback, interval)` | 稳定 | 添加线程（默认优先级）|
| `AddThread` | `int AddThread(name, handler, callback, interval, priority)` | 稳定 | 添加线程（指定优先级）|
| `RunOneShotTask` | `void RunOneShotTask(name, task, delay)` | 稳定 | 运行一次性任务 |
| `RunPeriodicalTask` | `void RunPeriodicalTask(name, task, interval, delay)` | 稳定 | 运行周期性任务 |
| `StopWatchdog` | `void StopWatchdog()` | 稳定 | 停止看门狗 |
| `InitFfrtWatchdog` | `void InitFfrtWatchdog()` | 稳定 | 初始化 FFRT 看门狗 |
| `SetBundleInfo` | `void SetBundleInfo(bundleName, bundleVersion)` | 稳定 | 设置 Bundle 信息 |
| `SetSystemApp` | `void SetSystemApp(isSystemApp)` | 稳定 | 设置系统应用标志 |
| `SetForeground` | `void SetForeground(isForeground)` | 稳定 | 设置前台状态 |
| `SetAppDebug` | `void SetAppDebug(isAppDebug)` | 稳定 | 设置调试模式 |
| `RemovePeriodicalTask` | `void RemovePeriodicalTask(name)` | 稳定 | 移除周期性任务 |
| `RemoveThread` | `void RemoveThread(name)` | 稳定 | 移除线程 |
| `InitMainLooperWatcher` | `void InitMainLooperWatcher(beginFunc, endFunc)` | 稳定 | 初始化主循环监听器 |
| `SetEventConfig` | `int SetEventConfig(paramsMap)` | 稳定 | 设置事件配置 |
| `ConfigEventPolicy` | `int ConfigEventPolicy(paramsMap)` | 稳定 | 配置事件策略 |
| `SetSpecifiedProcessName` | `void SetSpecifiedProcessName(name)` | 稳定 | 设置指定进程名 |
| `SetScrollState` | `void SetScrollState(isScroll)` | 稳定 | 设置滚动状态 |
| `StartSample` | `void StartSample(duration, interval)` | 稳定 | 开始采样 |
| `StopSample` | `std::string StopSample(sampleCount)` | 稳定 | 停止采样 |
| `GetSamplerResult` | `void GetSamplerResult(...)` | 稳定 | 获取采样结果 |

**证据**: `watchdog.h:40-209` - Watchdog 类声明

---

### 5. AppWatchdog 接口

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `GetReservedTimeForLogging` | `int32_t GetReservedTimeForLogging()` | 稳定 | 获取预留日志时间 |
| `SetBundleInfo` | `void SetBundleInfo(bundleName, bundleVersion)` | 稳定 | 设置 Bundle 信息 |
| `SetSystemApp` | `void SetSystemApp(isSystemApp)` | 稳定 | 设置系统应用标志 |
| `SetForeground` | `void SetForeground(isForeground)` | 稳定 | 设置前台状态 |
| `SetAppDebug` | `void SetAppDebug(isAppDebug)` | 稳定 | 设置调试模式 |
| `SetSpecifiedProcessName` | `void SetSpecifiedProcessName(name)` | 稳定 | 设置指定进程名 |
| `SetScrollState` | `void SetScrollState(isScroll)` | 稳定 | 设置滚动状态 |
| `GetSystemApp` | `bool GetSystemApp()` | 稳定 | 获取系统应用标志 |
| `GetBundleName` | `std::string GetBundleName()` | 稳定 | 获取 Bundle 名称 |
| `GetForeground` | `bool GetForeground()` | 稳定 | 获取前台状态 |
| `GetAppDebug` | `bool GetAppDebug()` | 稳定 | 获取调试模式 |
| `GetSpecifiedProcessName` | `std::string GetSpecifiedProcessName()` | 稳定 | 获取指定进程名 |
| `GetScrollState` | `bool GetScrollState()` | 稳定 | 获取滚动状态 |

**证据**: `app_watchdog.h:26-108` - AppWatchdog 类声明

---

### 6. ThreadSampler 接口

| 接口 | 签名 | 稳定性 | 说明 |
|-----|------|--------|------|
| `Init` | `bool Init(pid, handler)` | 稳定 | 初始化采样器 |
| `Start` | `void Start()` | 稳定 | 开始采样 |
| `Stop` | `void Stop()` | 稳定 | 停止采样 |
| `SetSampleConfig` | `void SetSampleConfig(sampleInterval, sampleCount)` | 稳定 | 设置采样配置 |

**证据**: `thread_sampler_api.h` - ThreadSampler API 声明

---

## 依赖关系

### 模块依赖图

```
Watchdog (对对外接口)
    ↓ 依赖
WatchdogInner (核心实现)
    ↓ 依赖
├── WatchdogTask (任务封装)
├── HandlerChecker (状态检查)
├── XCollie (超时检测)
│   ↓ 依赖
│   └── WatchdogInner (循环依赖，通过回调)
├── IpcFull (IPC 监控)
│   ↓ 依赖
│   └── WatchdogInner
└── ThreadSampler (线程采样)
    ↓ 动态加载
    libthread_sampler.z.so

AppWatchdog (应用层)
    ↓ 依赖
WatchdogInner (核心实现)
```

**证据**: `watchdog_inner.h:28-29` - 依赖头文件

### 外部依赖

| 依赖模块 | 用途 | BUILD.gn 位置 |
|---------|------|---------------|
| `c_utils:utils` | 工具函数 | `frameworks/native/BUILD.gn:40` |
| `hilog:libhilog` | 日志输出 | `frameworks/native/BUILD.gn:44` |
| `ipc:ipc_core` | IPC 调用 | `frameworks/native/BUILD.gn:47` |
| `ffrt:libffrt` | FFRT 框架 | `frameworks/native/BUILD.gn:43` |
| `faultloggerd:libbacktrace_local` | 堆栈回溯 | `frameworks/native/BUILD.gn:41` |
| `eventhandler:libeventhandler` | EventHandler 支持 | `frameworks/native/BUILD.gn:74` |

---

## 稳定性标识

### 标识定义

| 稳定性 | 说明 | 修改影响 | 使用建议 |
|--------|------|---------|----------|
| **STABLE** | 公开 API，向后兼容 | 不能修改签名 | 外部调用可放心使用 |
| **INTERNAL** | 内部使用，可能变更 | 可以修改 | 仅框架内部使用 |
| **CONDITIONAL** | 仅在特定配置下可用 | 需检查编译宏 | 需条件编译保护 |
| **PRIVATE** | 仅文件内使用 | 可以任意修改 | 不应被外部引用 |

### 稳定性分级详细说明

```cpp
// 文件: watchdog_inner.h

// STABLE - 稳定接口，公开 API
/**
 * @brief 添加线程监控
 * @param name 线程名称
 * @param handler EventHandler 指针
 * @param callback 超时回调
 * @param interval 检测间隔（秒）
 * @return 0 成功，-1 失败
 * @note 这是稳定接口，签名不会变更
 */
int AddThread(const std::string& name, const std::shared_ptr<EventHandler>& handler,
    const std::function<void(const std::string&)>& callback, uint64_t interval);

// INTERNAL - 内部接口
/**
 * @brief 内部任务插入（仅框架内部使用）
 * @note 此接口可能在未来版本中变更
 */
void InsertWatchdogTaskLocked(const std::shared_ptr<WatchdogTask>& task);

// CONDITIONAL - 条件编译接口
#ifdef HICOLLIE_JANK_ENABLE
/**
 * @brief 初始化主循环监听器
 * @note 仅在 hicollie_jank_detection_enable=true 时可用
 */
void InitMainLooperWatcher(const std::function<void(const std::string&)>& beginFunc,
    const std::function<void(const std::string&)>& endFunc);
#endif
```

### 稳定接口清单

| 模块 | 接口 | 稳定性 | 证据 |
|-----|------|--------|------|
| Watchdog | AddThread | STABLE | `watchdog.h:56-78` |
| Watchdog | RunOneShotTask | STABLE | `watchdog.h:80-95` |
| Watchdog | RunPeriodicalTask | STABLE | `watchdog.h:97-120` |
| Watchdog | StopWatchdog | STABLE | `watchdog.h:122-128` |
| XCollie | SetTimer | STABLE | `xcollie.h:32-52` |
| XCollie | CancelTimer | STABLE | `xcollie.h:54-64` |
| IpcFull | AddIpcFull | STABLE | `ipc_full.h:28-38` |
| ThreadSampler | Init | STABLE | `thread_sampler_api.h:25-32` |
| ThreadSampler | Start | STABLE | `thread_sampler_api.h:34-40` |
| ThreadSampler | Stop | STABLE | `thread_sampler_api.h:42-48` |

### 条件编译接口

| 模块 | 接口 | 稳定性 |
|-----|------|--------|
| Watchdog | AddThread | 稳定 |
| Watchdog | RunOneShotTask | 稳定 |
| Watchdog | RunPeriodicalTask | 稳定 |
| Watchdog | StopWatchdog | 稳定 |
| XCollie | SetTimer | 稳定 |
| XCollie | CancelTimer | 稳定 |
| IpcFull | AddIpcFull | 稳定 |
| ThreadSampler | Init | 稳定 |
| ThreadSampler | Start | 稳定 |
| ThreadSampler | Stop | 稳定 |

### 条件编译接口

| 接口 | 编译条件 | 说明 |
|-----|---------|------|
| `AsyncBinderSpaceFull` | `ASYNC_BINDER_SPACE_FULL` | 需启用 `hicollie_asyncbinderspacefull_enable` |
| `InitFfrtWatchdog` | `KICK_WATCHDOG_ENABLE` | 需启用 `hicollie_kick_watchdog_enable` |
| `SetEventConfig` | `HICOLLIE_JANK_ENABLE` | 需启用 `hicollie_jank_detection_enable` |

**证据**: `frameworks/native/BUILD.gn:56-70` - 条件编译宏

---

## 数据流

### 任务添加流程

```
应用代码
    ↓
Watchdog::AddThread()
    ↓
WatchdogInner::AddThread()
    ↓ [加锁 lock_]
InsertWatchdogTaskLocked()
    ↓
checkerQueue_.push(task)
    ↓ [解锁]
condition_.notify_one()
```

### 任务执行流程

```
看门狗线程
    ↓
condition_.wait_for()
    ↓
FetchNextTask()
    ↓
task.Run()
    ↓
[根据任务类型执行]
    ├── DoCallback() [XCollie]
    ├── RunHandlerCheckerTask() [Handler]
    ├── AsyncBinderSpace() [IPC]
    └── TimerCountTask() [计数器]
```

---

## 关键结论

### 接口特点
1. **单例模式** - 核心类使用单例确保全局唯一
2. **分层设计** - Watchdog 作为外观类封装 WatchdogInner
3. **回调驱动** - 大量使用回调函数实现通知机制
4. **条件编译** - 部分接口仅在特定宏定义时可用

### 依赖特点
1. **单向依赖** - 接口层 → 实现层 → 工具模块
2. **外部依赖** - 依赖 DFX 子系统其他组件
3. **动态加载** - ThreadSampler 通过 dlopen 动态加载

### 可扩展点
1. **新增任务类型** - 在 WatchdogTask 中添加新类型
2. **新增检测机制** - 在 WatchdogInner 中添加新检测方法
3. **扩展 Jank 检测** - 修改 WatchdogInner 的事件处理逻辑

---

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目定位
- [架构说明](02_Architecture.md) - 了解模块交互
- [NDK C API](03_NDK_API.md) - 对外 C API
- [安全评审](07_Security_Review.md) - 了解输入校验
