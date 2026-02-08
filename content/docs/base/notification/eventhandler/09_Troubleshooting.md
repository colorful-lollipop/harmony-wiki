# 常见问题与定位方法

## 目的

本文档提供 EventHandler 部件的常见构建、运行和调试问题的定位方法。

## 适用范围

- 构建系统：GN + Ninja
- 运行环境：OpenHarmonyOS
- 问题类型：构建失败、运行时崩溃、性能问题

## 构建问题

### 问题 1: 编译错误：找不到头文件

**症状**：
```
error: interfaces/inner_api/event_handler.h: No such file or directory
```

**定位步骤**：
1. 检查 `eventhandler.gni` 中的路径变量：
```bash
grep -n "frameworks_path\|inner_api_path" eventhandler.gni
```

2. 验证路径是否正确：
```bash
ls -la /base/notification/eventhandler/interfaces/inner_api/
```

3. 检查 BUILD.gn 中的 include_dirs：
```bash
grep -n "include_dirs" frameworks/eventhandler/BUILD.gn
```

**代码证据**：
- 路径定义：`eventhandler.gni:14-16`
- include 使用：`eventhandler/BUILD.gn:19-23`

### 问题 2: 链接错误：undefined reference

**症状**：
```
undefined reference to `EventRunner::Create`
```

**定位步骤**：
1. 检查 target 依赖：
```bash
grep -n "deps\|public_deps" frameworks/eventhandler/BUILD.gn
```

2. 检查源文件列表：
```bash
grep -n "sources = " frameworks/eventhandler/BUILD.gn:29
```

3. 检查 `inner_api_sources.gni`：
```bash
cat frameworks/eventhandler/inner_api_sources.gni
```

**代码证据**：
- 源文件列表：`eventhandler/inner_api_sources.gni:15-34`
- 依赖声明：`eventhandler/BUILD.gn:50-75`

### 问题 3: FFRT 未启用导致编译失败

**症状**：
```
error: 'ffrt_this_task_get_id' was not declared in this scope
```

**定位步骤**：
1. 检查 `eventhandler.gni` 中的 FFRT 配置：
```bash
grep -n "eventhandler_ffrt_usage" eventhandler.gni
```

2. 检查 `event_queue_ffrt.cpp` 是否在源列表中：
```bash
grep -n "event_queue_ffrt.cpp" frameworks/eventhandler/inner_api_sources.gni
```

3. 检查是否定义了 `FFRT_USAGE_ENABLE`：
```bash
grep -rn "FFRT_USAGE_ENABLE" frameworks/eventhandler/src/
```

**代码证据**：
- FFRT 使用标志：`eventhandler.gni:27`
- 条件编译：`eventhandler/inner_api_sources.gni:31-34`
- FFRT 检测：`event_runner.cpp:779`

**解决方案**：
- 确保编译时启用了 FFRT：`--args eventhandler_ffrt_usage=true`
- 或修改 `eventhandler.gni`：`eventhandler_ffrt_usage = false`

### 问题 4: PGO 优化配置错误

**症状**：
```
error: --fprofile-use=...: No such file or directory
```

**定位步骤**：
1. 检查 `eventhandler.gni` 中的 PGO 配置：
```bash
grep -n "eventhandler_feature_enable_pgo\|eventhandler_feature_pgo_path" eventhandler.gni
```

2. 验证 PGO profile 文件是否存在：
```bash
ls -la /path/to/profile/libeventhandler.profdata
```

3. 检查编译参数：
```bash
gn args --list
```

**代码证据**：
- PGO 配置：`eventhandler.gni:44-47`
- 编译选项：`eventhandler/BUILD.gn:97-105`

**解决方案**：
- 禁用 PGO：`--args eventhandler_feature_enable_pgo=false`
- 或提供正确的 profile 路径

## 运行时问题

### 问题 5: EventRunner 无法启动

**症状**：
- 调用 `EventRunner::Run()` 返回错误码
- 事件不处理
- 日志显示 "Event runner is not running"

**定位步骤**：
1. 检查日志输出：
```bash
hdc shell "hilog -T AppExecFwk/EH | grep EventRunner"
```

2. 检查 EventRunner 是否已运行：
```cpp
// 在应用代码中检查
if (!runner->IsRunning()) {
    HILOGE("Event runner is not running");
}
```

3. 检查是否有多个 Run() 调用：
```bash
hdc shell "hilog -T AppExecFwk/EH | grep 'Already running'"
```

**代码证据**：
- 运行状态检查：`event_runner.cpp:702-705`
- 错误码：`event_handler_errors.h:41`

**可能原因**：
- 重复调用 `Run()`
- Stop() 后未重新启动
- 线程已退出

**解决方案**：
```cpp
// 检查运行状态后再调用
auto runner = EventRunner::Create();
if (runner->Run() != ERR_OK) {
    auto errCode = runner->GetLastErrorCode();
    HILOGE("Failed to start runner: %{public}d", errCode);
    // 处理错误
}
```

### 问题 6: 事件丢失

**症状**：
- 发送的事件未被处理
- 回调未被调用
- 日志中有 "ProcessEvent has no valid callback"

**定位步骤**：
1. 启用详细日志：
```bash
hdc shell "param set debug.hilog.log.on true"
hdc shell "param set debug.hilog.tag on AppExecFwk/EH:V"
```

2. 检查事件队列状态：
```cpp
// 在应用代码中添加 Dump
runner->GetEventQueue()->Dump(dumper);
```

3. 检查回调是否被移除：
```cpp
// 检查 emitter 实例状态
auto listenerCount = emitter->getListenerCount("myEvent");
```

**代码证据**：
- 日志位置：`events_emitter.cpp:194`
- Dump 方法：`event_queue.h:133`

**可能原因**：
- EventHandler 被销毁
- 回调抛出异常
- 事件优先级过低，队列中等待超时

**解决方案**：
```cpp
// 添加异常处理
try {
    eventHandler->SendEvent(event);
} catch (const std::exception& e) {
    HILOGE("Exception in SendEvent: %{public}s", e.what());
}

// 添加错误回调
eventEmitter.on("error", (err) => {
    console.error("Event send failed:", err);
});
```

### 问题 7: 内存泄漏

**症状**：
- 应用内存持续增长
- 系统内存告警
- 性能下降

**定位步骤**：
1. 使用 HiTrace 追踪事件分配：
```bash
hdc shell "hdc shell 'hidumper --trace -s eventhandler'"
```

2. 使用 Valgrind 检测：
```bash
valgrind --leak-check=full --log-file=leak.log ./your_app
```

3. 检查对象池状态：
```cpp
// 在 InnerEvent 中添加统计
class InnerEvent {
    static std::atomic<size_t> poolSize_{0};
    static std::atomic<size_t> activeCount_{0};
};
```

**代码证据**：
- 对象池：`inner_event.h:104-112`
- 日志宏：`event_logger.h`

**可能原因**：
- 回调中未正确释放事件对象
- 异常路径中资源清理不完整
- 循环引用

**解决方案**：
```cpp
// 使用智能指针
auto event = InnerEvent::Get(eventId, param);
// 智能指针自动管理生命周期

// 添加 RAII 包装
class EventGuard {
    InnerEvent::Pointer event_;
public:
    EventGuard(InnerEvent::Pointer&& e) : event_(std::move(e)) {}
    ~EventGuard() {
        // 自动释放
    }
};
```

### 问题 8: FFRT 任务卡死

**症状**：
- FFRT 队列中的任务不执行
- 事件处理延迟
- 线程阻塞

**定位步骤**：
1. 检查 FFRT 任务状态：
```bash
# 需要内核支持
cat /sys/kernel/debug/sched_features
```

2. 检查事件队列状态：
```cpp
runner->GetEventQueue()->Dump(dumper);
```

3. 查看堆栈跟踪：
```bash
hdc shell "pidof your_app | xargs -I {} pmap -x | grep stack"
```

**代码证据**：
- FFRT 队列：`event_queue_ffrt.cpp:70-97`
- Dump 方法：`event_queue.h:133`

**可能原因**：
- FFRT 任务无超时设置
- 优先级设置不当
- FFRT 调度器过载

**解决方案**：
```cpp
// 设置 FFRT 任务超时
ffrt_queue_submit(..., timeoutMs, ...);

// 添加超时回调
if (event->GetTaskTime() + timeout < now) {
    HILOGW("Event timeout: %{public}s", event->GetEventId());
    // 超时处理
}
```

## 性能问题

### 问题 9: 事件处理延迟高

**症状**：
- 事件发送到处理延迟过大
- 用户体验卡顿
- 监控显示高延迟

**定位步骤**：
1. 使用 HiTrace 追踪延迟：
```cpp
#include "hitrace_meter_adapter.h"

auto begin = DistributeBegin_(eventName);
// ... 处理事件
auto end = DistributeEnd_(eventName, begin);
```

2. 添加性能日志：
```cpp
HILOGI("Event processed in %{public}lld us", duration.count());
```

3. 检查事件队列长度：
```cpp
size_t queueSize = queue_->GetEventSize();
HILOGW("Queue size: %{public}zu", queueSize);
```

**代码证据**：
- 追踪接口：`event_runner.h:47-52`
- 日志宏：`event_logger.h`
- 队列大小：`event_queue.h:141`

**可能原因**：
- 事件处理函数执行时间过长
- 事件优先级设置不当（大量 LOW 优先级）
- 线程阻塞（I/O 等待）

**解决方案**：
```cpp
// 优化事件处理函数
void FastEventHandler::ProcessEvent(const InnerEvent::Pointer &event) {
    auto start = std::chrono::steady_clock::now();

    // ... 处理逻辑

    auto end = std::chrono::steady_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

    if (duration.count() > 10000) {  // 超过 10ms
        HILOGW("Slow event processing: %{public}lld us", duration.count());
    }
}
```

### 问题 10: 优先级倒置

**症状**：
- 高优先级事件被低优先级事件阻塞
- 关键事件延迟处理

**定位步骤**：
1. 检查事件优先级分布：
```bash
hdc shell "hilog -T AppExecFwk/EH | grep Priority"
```

2. 使用 HiChecker 检测优先级反转：
```bash
hdc shell "hichecker -s eventhandler"
```

3. 添加优先级追踪：
```cpp
auto now = std::chrono::steady_clock::now();
HILOGI("Event priority=%{public}d, delay=%{public}lld ms",
    static_cast<int>(priority),
    delay.count());
```

**代码证据**：
- 优先级枚举：`event_queue.h:42-45`
- HiChecker 集成：`eventhandler.gni:64-67`

**可能原因**：
- 不正确使用 IMMEDIATE 优先级
- 优先级继承锁配置不当
- 事件队列实现缺陷

**解决方案**：
```cpp
// 仔细评估优先级使用
void SendCriticalEvent(const std::string& eventId) {
    // 关键事件使用 IMMEDIATE
    auto event = InnerEvent::Get(eventId, 0);
    eventHandler->SendEvent(event, 0, Priority::IMMEDIATE);
}

void SendNormalEvent(const std::string& eventId) {
    // 普通事件使用 HIGH 或 LOW
    auto event = InnerEvent::Get(eventId, 0);
    eventHandler->SendEvent(event, 0, Priority::HIGH);
}
```

## 调试技巧

### 启用详细日志

```bash
# 开启所有 HiLog 日志
hdc shell "param set debug.hilog.log.on true"
hdc shell "param set debug.hilog.tag on *"

# 重启后生效
hdc shell "killall -9 com.example.app"
```

### 使用 Dump 功能

```cpp
// 添加自定义 Dump 方法
void MyEventHandler::Dump(Dumper &dumper) {
    // 调用父类 Dump
    EventHandler::Dump(dumper);

    // 添加自定义信息
    dumper.Dump(dumper.GetTag() + " MyState: " + myState_);
}
```

### 添加性能监控

```cpp
// 在关键位置添加统计
class PerformanceMonitor {
    static std::atomic<uint64_t> eventCount_{0};
    static std::atomic<uint64_t> totalLatency_{0};

public:
    static void RecordEvent(uint64_t latency) {
        eventCount_++;
        totalLatency_ += latency;
    }

    static void PrintStats() {
        double avgLatency = static_cast<double>(totalLatency_) / eventCount_;
        HILOGI("Stats: count=%{public}lu, avg=%.2f us",
            eventCount_.load(), avgLatency);
    }
};
```

### 使用 GDB 调试

```bash
# 使用 GDB 调试
gdb --args `pidof your_app`
(gdb) break EventHandler::ProcessEvent
(gdb) run

# 查看堆栈
(gdb) bt
```

## 相关文档

- [安全风险评审](08_Security_Review.md) - 安全问题分析
- [GN 构建目标](06_GN_Targets.md) - 构建配置
- [架构说明](03_Architecture.md) - 架构设计
