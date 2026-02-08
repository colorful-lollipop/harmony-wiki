# 安全风险评审

## 目的

本文档基于代码证据，分析 EventHandler 部件的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- 检查范围：核心实现代码（`frameworks/eventhandler/`, `frameworks/napi/`, `frameworks/emitter/`）
- 排除：测试代码（`test/`, `fuzztest/` 等）
- 分析方法：静态代码审查 + 威胁建模

## 威胁模型

### 外部输入 → 敏感操作

```
JavaScript 应用
    ↓ [N-API]
EventHandler 事件处理
    ↓ [事件队列]
I/O 操作（文件描述符）
    ↓ [系统调用]
内核资源
```

## 攻击面清单

### 1. N-API 输入验证

**攻击面**：JavaScript 应用通过 N-API 传递恶意参数

**代码证据**：
- 参数解析：`frameworks/napi/src/events_emitter.cpp:849-901`

```cpp
// 事件 ID 类型校验（events_emitter.cpp:861-864）
if (eventValueType != napi_object && eventValueType != napi_string) {
    HILOGD("type mismatch for parameter 1");
    return nullptr;  // 静默返回 null，无错误提示
}

// 回调函数校验（events_emitter.cpp:865-868）
if (GetNapiType(env, argv[1]) != napi_function) {
    HILOGD("OnOrOnce type mismatch for parameter 2");
    return nullptr;  // 静默返回 null
}
```

**风险**：
- ❌ 缺少参数类型验证的详细错误信息
- ❌ 无参数长度限制（事件 ID 字符串长度无限制）
- ❌ 无递归深度检查（事件数据对象可能包含嵌套对象）

**修复建议**：
```cpp
// 添加详细错误信息
if (eventValueType != napi_object && eventValueType != napi_string) {
    napi_throw_type_error(env, "event must be a string or object");
    return nullptr;
}

// 添加长度限制
if (eventValueType == napi_string) {
    size_t length = 0;
    napi_get_value_string_utf8(env, argv[0], nullptr, 0, &length);
    if (length > MAX_EVENT_ID_LENGTH) {
        napi_throw_range_error(env, "event ID too long");
        return nullptr;
    }
}
```

### 2. 事件数据反序列化

**攻击面**：恶意的序列化数据导致内存破坏或代码执行

**代码证据**：
- N-API 反序列化：`frameworks/napi/src/events_emitter.cpp:94-104`

```cpp
// 无边界检查的反序列化（events_emitter.cpp:100）
if (napi_deserialize_hybrid(callbackInner->env, *(eventDataInner->data), &resultData) != napi_ok ||
    resultData == nullptr) {
    HILOGE("Deserialize fail.");
    return;  // 仅仅记录错误，没有清理数据
}
```

**风险**：
- ⚠️ 恶意构造的序列化数据可能导致堆溢出
- ⚠️ 无数据大小限制
- ⚠️ 反序列化失败后状态不一致

**修复建议**：
```cpp
// 添加数据大小检查
if (eventDataInner->size > MAX_EVENT_DATA_SIZE) {
    HILOGE("Event data too large: %{public}zu bytes", eventDataInner->size);
    return;
}

// 添加清理逻辑
if (napi_deserialize_hybrid(...) != napi_ok || resultData == nullptr) {
    HILOGE("Deserialize failed, cleaning up");
    // 清理资源并返回错误
    return;
}
```

### 3. 文件描述符处理

**攻击面**：通过文件描述符监听实现权限提升或信息泄露

**代码证据**：
- EpollIoWaiter：`frameworks/eventhandler/src/epoll_io_waiter.cpp:61-100`

```cpp
// Epoll 初始化无权限检查（epoll_io_waiter.cpp:61）
bool EpollIoWaiter::Init()
{
    int32_t epollFd = epoll_create(MAX_EPOLL_EVENTS_SIZE);
    // 无对 epollFd 的权限或所有权验证
    awakenFd = eventfd(0, EFD_CLOEXEC | EFD_NONBLOCK);
    // 直接使用 awakenFd，未验证来源
    ...
}
```

**风险**：
- 🔴 无文件描述符权限验证（任意应用可监听任意 fd）
- 🔴 无 fd 来源验证（可能监听其他应用的 fd）
- 🔴 eventfd 无访问控制（任何知道 fd 值的应用可唤醒）

**修复建议**：
```cpp
// 添加文件描述符来源验证
bool EpollIoWaiter::Init()
{
    int32_t epollFd = epoll_create(MAX_EPOLL_EVENTS_SIZE);
    if (epollFd < 0) {
        HILOGE("Failed to create epoll");
        return false;
    }

    // 验证调用者权限
    if (!HasFdCreationPermission()) {
        HILOGE("No permission to create file descriptor");
        return false;
    }

    // 使用受控的 eventfd
    // 不接受外部传入的 fd，只使用内部创建的
    ...
}
```

**相关证据**：
- `interfaces/inner_api/file_descriptor_listener.h:18` - 文件描述符监听器接口
- `interfaces/kits/native/native_interface_eventhandler.h:44` - Native API 暴露 fd 操作

### 4. 优先级操纵

**攻击面**：应用通过设置 IMMEDIATE 优先级垄断事件队列

**代码证据**：
- 优先级枚举：`interfaces/inner_api/event_queue.h:42-45`

```cpp
enum class Priority {
    IMMEDIATE = 0,  // 立即处理，最高优先级
    HIGH = 1,
    LOW = 2,
    IDLE = 3,
};
```

**风险**：
- ⚠️ 恶意应用可大量发送 IMMEDIATE 优先级事件
- ⚠️ 可能导致饥饿攻击，阻止其他事件被处理
- ⚠️ 无优先级使用频率限制

**修复建议**：
```cpp
// 添加优先级配额
class EventQueue {
private:
    struct PriorityQuota {
        int immediateCount = 0;
        int64_t immediateWindowStart = 0;
    };
    std::map<uint32_t, PriorityQuota> priorityQuotas_;

public:
    bool Insert(InnerEvent::Pointer &event, Priority priority) {
        if (priority == Priority::IMMEDIATE) {
            // 检查配额
            auto& quota = priorityQuotas_[eventId];
            int64_t now = std::chrono::steady_clock::now();
            int64_t windowSize = 1000; // 1 秒窗口

            if (now - quota.immediateWindowStart < windowSize) {
                if (quota.immediateCount >= MAX_IMMEDIATE_PER_SECOND) {
                    HILOGW("IMMEDIATE priority quota exceeded");
                    return false;
                }
            } else {
                quota.immediateWindowStart = now;
                quota.immediateCount = 1;
            }
        }
        // ... 正常插入
    }
};
```

### 5. 线程安全

**攻击面**：竞态条件导致事件丢失或状态不一致

**代码证据**：
- 原子操作：`event_queue.cpp:20`

```cpp
std::mutex queueLock_;  // 使用普通互斥锁
```

**风险**：
- ⚠️ 优先级继承锁可能存在死锁风险（`PriorityInheritanceLock`）
- ⚠️ 无超时机制的锁等待
- ⚠️ 事件队列操作可能阻塞过长时间

**修复建议**：
```cpp
// 使用带超时的锁
class SafeEventQueue {
private:
    std::timed_mutex queueLock_;

public:
    bool Insert(InnerEvent::Pointer &event, Priority priority) {
        std::unique_lock<std::timed_mutex> lock(queueLock_, std::chrono::milliseconds(100));
        if (!lock.owns_lock()) {
            HILOGE("Failed to acquire lock within timeout");
            return false;
        }
        // ... 插入事件
        return true;
    }
};
```

### 6. 内存管理

**攻击面**：事件对象池的内存耗尽或释放后使用

**代码证据**：
- InnerEvent 对象池：`interfaces/inner_api/inner_event.h:104-112`

```cpp
static Pointer Get(uint32_t innerEventId, int64_t param = 0, const Caller &caller = {})
{
    // 从池中获取，无数量限制
    return Pointer(new (std::nothrow) InnerEvent(...), [](InnerEvent* ptr) {
        delete ptr;
    });
}
```

**风险**：
- 🔴 无对象池大小限制
- 🔴 对象池可能被耗尽（拒绝服务）
- 🔴 释放后使用指针的风险

**修复建议**：
```cpp
class BoundedEventPool {
private:
    static constexpr size_t MAX_POOL_SIZE = 1000;
    std::array<std::unique_ptr<InnerEvent>, MAX_POOL_SIZE> pool_;
    std::atomic<size_t> poolSize_{0};

public:
    static Pointer Get(uint32_t innerEventId, int64_t param = 0) {
        if (poolSize_.load() >= MAX_POOL_SIZE) {
            HILOGE("Event pool exhausted");
            return Pointer(new InnerEvent(...), [](InnerEvent* ptr) {
                delete ptr;
            });
        }
        // ... 从池中获取
        return Pointer(pool_[index++].release(), ...);
    }
};
```

### 7. 回调函数安全性

**攻击面**：线程安全回调中的数据竞争或释放后使用

**代码证据**：
- ThreadSafeCallback：`frameworks/napi/src/events_emitter.cpp:140-158`

```cpp
void ThreadSafeCallback(napi_env env, napi_value jsCallback, void* context, void* data)
{
    EventDataWorker* eventDataInner = static_cast<EventDataWorker*>(data);
    auto callbackInfoInner = eventDataInner->callbackInfo;

    // 无有效性检查直接访问
    if (callbackInfoInner && !(callbackInfoInner->isDeleted)) {
        napi_open_handle_scope(callbackInfoInner->env, &scope);
        ProcessCallback(eventDataInner);
        napi_close_handle_scope(callbackInfoInner->env, scope);
    }

    delete eventDataInner;
    // 资源清理时机不当
}
```

**风险**：
- 🔴 无 `callbackInfoInner` 的原子检查
- 🔴 释放后使用 `env` 指针
- 🔴 无引用计数机制

**修复建议**：
```cpp
void ThreadSafeCallback(napi_env env, napi_value jsCallback, void* context, void* data)
{
    EventDataWorker* eventDataInner = static_cast<EventDataWorker*>(data);

    // 使用原子操作获取共享指针
    std::shared_ptr<AsyncCallbackInfo> callbackInfo =
        callbackInfoInner->lock();  // 原子获取

    if (callbackInfo && !callbackInfo->isDeleted.load(std::memory_order_acquire)) {
        if (callbackInfo->env.load(std::memory_order_acquire) != nullptr) {
            napi_open_handle_scope(callbackInfo->env, &scope);
            ProcessCallback(eventDataInner);
            napi_close_handle_scope(callbackInfo->env, scope);
        } else {
            HILOGE("Callback environment already cleaned");
        }
    }

    delete eventDataInner;
}
```

### 8. FFRT 集成安全

**攻击面**：通过 FFRT 队列注入恶意任务

**代码证据**：
- FFRT 队列适配：`frameworks/eventhandler/src/event_queue_ffrt.cpp:106-144`

```cpp
bool EventQueueFFRT::Insert(InnerEvent::Pointer &event, Priority priority, EventInsertType insertType)
{
    ffrt_queue_t* queue = TransferQueuePtr(ffrtQueue_);
    // 无对事件内容的验证
    ffrt_queue_submit(*queue, reinterpret_cast<ffrt_task_t*>(task),
        TransferInnerPriority(priority),
        0,  // 无超时
        nullptr);
    ...
}
```

**风险**：
- ⚠️ 无 FFRT 任务内容验证
- ⚠️ 无超时设置（任务可能永久阻塞）
- ⚠️ FFRT 任务的权限继承来自调用者

**修复建议**：
```cpp
bool EventQueueFFRT::Insert(InnerEvent::Pointer &event, Priority priority, EventInsertType insertType)
{
    // 验证事件内容
    if (!ValidateEventContent(event)) {
        HILOGE("Invalid event content");
        return false;
    }

    ffrt_queue_t* queue = TransferQueuePtr(ffrtQueue_);
    ffrt_task_t* task = reinterpret_cast<ffrt_task_t*>(event.get());

    // 设置超时防止永久阻塞
    uint64_t timeoutMs = CalculateTimeout(event);
    ffrt_queue_submit(*queue, task,
        TransferInnerPriority(priority),
        timeoutMs,  // 添加超时
        nullptr);
    ...
}
```

## 信任边界

### 内部 vs 外部

| 边界 | 信任等级 | 说明 |
|-------|---------|------|
| 内部 API 调用 | 高信任 | 同一进程内的模块调用 |
| N-API 调用 | 中等信任 | 来自沙箱化的 JS 应用 |
| Native C API 调用 | 低信任 | 任意 Native 应用 |
| FFRT 任务 | 低信任 | FFRT 调度器 |

### 数据流边界

```
[JavaScript 应用] --(N-API 边界)--> [EventHandler]
    --(FFRT 边界)--> [FFRT Runtime] --(系统调用边界)--> [内核]
```

## 风险缓解措施

### 现有保护

| 保护机制 | 位置 | 效果 |
|----------|-------|------|
| 参数类型校验 | N-API 层 | 部分（缺少详细错误） |
| 互斥锁 | EventQueue | 竞态保护 |
| 原子操作 | 状态标记 | 部分并发控制 |
| HiTrace 追踪 | 全局 | 审计日志 |
| HiChecker 检测 | 全局 | 性能异常检测 |
| Sanitizers | 编译时 | 内存安全（integer_overflow, ubsan, cfi） |

**代码证据**：
- Sanitizer 配置：`eventhandler.gni:30-37`

```gni
sanitize = {
  integer_overflow = true,
  ubsan = true,
  boundary_sanitize = true,
  cfi = true,
  cfi_cross_dso = true,
  debug = false,
}
branch_protector_ret = "pac_ret"
```

## 安全检查范围

### 已检查

- ✅ N-API 输入验证
- ✅ 文件描述符处理
- ✅ 线程安全机制
- ✅ 内存管理
- ✅ FFRT 集成

### 未检查（局限性）

- ❌ 权限检查（无 ACE 权限验证）
- ❌ 输入大小限制（无全局常量）
- ❌ 速率限制（无 API 调用频率限制）
- ❌ 审计日志（仅错误日志，无安全事件日志）
- ❌ 恶意事件检测（无异常模式识别）

**代码证据**：
- 搜索关键字：`permission`, `access`, `auth`, `audit`, `rate_limit`
- 结果：未找到相关实现

## 安全加固建议

### 短期改进（高优先级）

1. **添加 N-API 参数验证详细错误**
   - 使用 `napi_throw_type_error()` 提供明确的错误信息
   - 实现 N-API 层的参数验证

2. **添加事件数据大小限制**
   - 定义 `MAX_EVENT_DATA_SIZE` 常量
   - 在反序列化前验证大小

3. **添加优先级配额机制**
   - 限制 IMMEDIATE 优先级事件频率
   - 防止队列饥饿攻击

4. **增强线程安全**
   - 使用带超时的锁
   - 添加引用计数到回调管理

### 中期改进（中优先级）

1. **实现文件描述符权限验证**
   - 限制哪些应用可以监听文件描述符
   - 验证 fd 来源

2. **添加事件对象池大小限制**
   - 防止内存耗尽
   - 实现池满时的优雅降级

3. **添加安全审计日志**
   - 记录敏感操作（fd 创建、事件发送）
   - 支持事后审计

### 长期改进（低优先级）

1. **实现权限检查集成**
   - 集成 ACE 权限系统
   - 验证调用者权限

2. **添加速率限制**
   - 限制 API 调用频率
   - 防止滥用

3. **实现威胁检测**
   - 检测异常模式（大量 IMMEDIATE 事件、频繁失败）
   - 自动缓解或告警

## 相关跳转

- [内部 API](05_Inner_API.md) - 接口安全细节
- [架构说明](03_Architecture.md) - 组件设计
- [N-API 接口](04_NAPI_API.md) - API 安全细节
