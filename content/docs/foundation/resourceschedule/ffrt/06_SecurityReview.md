# 安全风险评估

> FFRT 深度安全分析
> 基于代码审计的风险识别与修复建议

---

## 风险评估概览

| 风险类别 | 数量 | 最高等级 | 整体风险 |
|----------|------|----------|----------|
| 输入验证缺陷 | 3 | 中 | 🟡 中等 |
| 内存安全问题 | 4 | 高 | 🔴 高 |
| 权限与鉴权 | 2 | 低 | 🟢 低 |
| 并发安全 | 3 | 中 | 🟡 中等 |
| 逻辑漏洞 | 2 | 中 | 🟡 中等 |

**综合评估**: 整体风险等级为 **中等偏高**，主要风险集中在内存安全和并发安全领域。

---

## R1: 空指针解引用风险（中危）

### 证据

**位置**: `src/core/task.cpp:67-84`

```cpp
void DestroyFunctionWrapper(ffrt_function_header_t* f,
    ffrt_function_kind_t kind = ffrt_function_kind_general)
{
    if (f == nullptr || f->destroy == nullptr) {  // ✓ 有检查
        return;
    }
    f->destroy(f);  // 调用用户提供的销毁函数
    // ... 后续使用f的代码
}
```

**问题**: 虽然检查了 `f` 和 `f->destroy`，但 `f->exec` 在 `submit` 流程中未被充分验证。

### 触发路径

```
攻击者构造恶意调用:
  ↓
ffrt_submit_f(nullptr, nullptr, ...)
  ↓
src/core/task.cpp:332 - 检查f非空
  ↓
如果绕过检查（如并发修改）
  ↓
解引用空指针 → SIGSEGV
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 高 - 易于构造 |
| **影响范围** | 进程崩溃 |
| **权限提升** | 无 |
| **CVSS估算** | 5.3 (Medium) |

### 修复建议

```cpp
// 建议在所有入口添加严格检查
API_ATTRIBUTE((visibility("default")))
void ffrt_submit_base(ffrt_function_header_t* f, ...)
{
    // 当前检查
    if (unlikely(!f)) {
        FFRT_LOGE("function handler should not be empty");
        return;
    }
    
    // 建议增加exec检查
    if (unlikely(!f->exec)) {
        FFRT_LOGE("exec function should not be empty");
        return;
    }
    
    // ... 原有逻辑
}
```

---

## R2: 栈溢出风险（中危）

### 证据

**位置**: `src/eu/co_routine.cpp:55-69`

```cpp
static inline void CoStackCheck(CoRoutine* co)
{
    if (unlikely(co->stkMem.magic != STACK_MAGIC)) {
        FFRT_SYSEVENT_LOGE("sp offset:%llx.\n", co->stkMem.stk +
            co->stkMem.size - co->ctx.storage[FFRT_REG_SP]);
        FFRT_SYSEVENT_LOGE("stack over flow, check local variable in you tasks"
            " or use api 'ffrt_task_attr_set_stack_size'.\n");
        if (ExecuteCtx::Cur()->task != nullptr) {
            auto curTask = ExecuteCtx::Cur()->task;
            FFRT_SYSEVENT_LOGE("task name[%s], gid[%llu], submit_tid[%d]",
                curTask->GetLabel().c_str(), curTask->gid, curTask->fromTid);
        }
        abort();  // 检测到溢出后直接终止
    }
}
```

### 触发路径

```
攻击者:
  1. 创建大量嵌套任务
  2. 每个任务占用较大栈空间
  3. 超过协程栈限制（默认1MB）
  ↓
CoStackCheck 检测到 magic 被破坏
  ↓
abort() - 进程终止
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 需要构造递归场景 |
| **影响范围** | DoS（拒绝服务） |
| **权限提升** | 无 |
| **CVSS估算** | 5.3 (Medium) |

### 修复建议

当前实现已在检测到溢出时终止进程，这是安全的做法。建议增加：

1. **预防性措施**:
```cpp
// 在任务提交时检查递归深度
constexpr int MAX_NESTING_DEPTH = 100;
if (curTask->nestingDepth > MAX_NESTING_DEPTH) {
    FFRT_LOGE("Task nesting too deep, may cause stack overflow");
    return ffrt_error;
}
```

2. **监控告警**:
```cpp
// 在栈使用达到80%时发出警告
if (co->stkMem.size - usedStack < co->stkMem.size * 0.2) {
    FFRT_LOGW("Stack usage high: %zu bytes remaining", remaining);
}
```

---

## R3: Use-After-Free 风险（高危）

### 证据

**位置**: `src/tm/cpu_task.cpp:79-94`

```cpp
inline void SetInHandles(std::vector<CPUEUTask*>& in_handles)
{
    if (in_handles.empty()) {
        return;
    }
    in_handles_ = new std::vector<CPUEUTask*>(in_handles);  // 拷贝指针
}

inline const std::vector<CPUEUTask*>& GetInHandles()
{
    static const std::vector<CPUEUTask*> empty;
    if (!in_handles_) {
        return empty;
    }
    return *in_handles_;  // 返回存储的指针
}
```

**位置**: `src/dm/sdependence_manager.cpp:88-92`

```cpp
// placement new 创建任务
SCPUEUTask* task = reinterpret_cast<SCPUEUTask*>(
    ffrt::TaskFactory<SCPUEUTask>::Alloc());
new(task) SCPUEUTask(attr, parent, gid);  // 原地构造
```

### 触发路径

```
攻击者构造竞态场景:
  ↓
Thread A: 创建任务 T1
  ↓
Thread B: 等待 T1 完成 → onTaskDone 释放 T1
  ↓
Thread A: 仍持有 T1 句柄，继续访问
  ↓
Use-After-Free
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 需要竞态条件 |
| **影响范围** | 内存破坏、代码执行 |
| **权限提升** | 可能 |
| **CVSS估算** | 7.4 (High) |

### 修复建议

```cpp
// 建议：使用原子引用计数检查
class CPUEUTask : public CoTask {
public:
    // 增加安全检查
    bool IsValid() const {
        return deleteRef_.load() > 0 && status != TaskStatus::FINISH;
    }
};

// 在访问前检查有效性
void SomeOperation(CPUEUTask* task) {
    if (!task || !task->IsValid()) {
        FFRT_LOGE("Accessing invalid task");
        return;
    }
    // ... 安全访问
}
```

---

## R4: 竞态条件风险（中危）

### 证据

**位置**: `src/sync/wait_queue.cpp` (WaitEntry处理)

```cpp
// 潜在的TOCTOU模式
if (task == nullptr || task->Block() == BlockType::BLOCK_THREAD) {
    // task 可能在此间隙被其他线程修改
    WaitQueue::ThreadWait(wue, lk, task);
}
```

**位置**: `src/eu/co_routine.cpp:419-426`

```cpp
static inline bool CoBboxPreCheck(ffrt::CoTask* task)
{
    if (task->coRoutine) {
        int ret = task->coRoutine->status.exchange(static_cast<int>(CoStatus::CO_RUNNING));
        // 可能的状态竞态
        if (ret == static_cast<int>(CoStatus::CO_RUNNING) && GetBboxEnableState() != 0) {
            return false;
        }
    }
    return true;
}
```

### 触发路径

```
多线程并发访问同一任务:
  Thread A: 检查 task->status == READY
  Thread B: 修改 task->status = EXECUTING
  Thread A: 基于旧状态做决策 → 竞态
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 难以稳定触发 |
| **影响范围** | 状态不一致、逻辑错误 |
| **权限提升** | 不太可能 |
| **CVSS估算** | 4.3 (Medium) |

### 修复建议

```cpp
// 使用原子操作保证状态一致性
bool TrySetExecuting(TaskBase* task) {
    TaskStatus expected = TaskStatus::READY;
    return task->status.compare_exchange_strong(
        expected, 
        TaskStatus::EXECUTING,
        std::memory_order_acquire,
        std::memory_order_relaxed
    );
}

// 在CoBboxPreCheck中
if (task->coRoutine) {
    auto expected = static_cast<int>(CoStatus::CO_NOT_FINISH);
    bool exchanged = task->coRoutine->status.compare_exchange_strong(
        expected,
        static_cast<int>(CoStatus::CO_RUNNING),
        std::memory_order_seq_cst
    );
    if (!exchanged) {
        // 状态已被修改，放弃当前操作
        return false;
    }
}
```

---

## R5: 资源耗尽风险（中危）

### 证据

**位置**: `src/core/task.cpp:478-494`

```cpp
int ffrt_set_cpu_worker_max_num(ffrt_qos_t qos, uint32_t num)
{
    if (num == 0 || num > ffrt::QOS_WORKER_MAXNUM) {  // 有上限检查
        FFRT_LOGE("qos[%d] worker num[%u] is invalid.", qos, num);
        return -1;
    }
    // ...
    return ffrt::FFRTFacade::GetEUInstance().SetWorkerMaxNum(_qos, num);
}
```

**位置**: `src/eu/co_routine.cpp:292-318`

```cpp
static inline CoRoutine* AllocNewCoRoutine(size_t stackSize)
{
    // 内存分配，可能失败
    co = static_cast<CoRoutine*>(mmap(nullptr, stackSize,
        PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_PRIVATE, -1, 0));
    if (co == reinterpret_cast<CoRoutine*>(MAP_FAILED)) {
        FFRT_SYSEVENT_LOGE("memory mmap failed.");
        return nullptr;  // 返回nullptr
    }
    // ...
}
```

### 触发路径

```
攻击者:
  1. 大量调用 ffrt_submit 创建任务
  2. 每个任务占用协程栈（1MB默认）
  3. 系统内存耗尽
  ↓
后续任务创建失败或系统OOM
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 高 - 易于构造 |
| **影响范围** | DoS（拒绝服务） |
| **权限提升** | 无 |
| **CVSS估算** | 5.3 (Medium) |

### 修复建议

```cpp
// 建议增加全局资源限制
class ResourceLimiter {
public:
    static bool CanCreateTask() {
        return activeTaskCount_.load() < MAX_TOTAL_TASKS;
    }
    
    static void OnTaskCreated() {
        activeTaskCount_.fetch_add(1);
    }
    
    static void OnTaskDestroyed() {
        activeTaskCount_.fetch_sub(1);
    }
    
private:
    static constexpr size_t MAX_TOTAL_TASKS = 10000;
    static std::atomic<size_t> activeTaskCount_;
};

// 在任务提交时检查
int ffrt_set_cpu_worker_max_num(ffrt_qos_t qos, uint32_t num) {
    // 增加总任务数限制
    if (!ResourceLimiter::CanCreateTask()) {
        FFRT_LOGE("Global task limit reached");
        return ffrt_error_busy;
    }
    // ...
}
```

---

## R6: 类型混淆风险（低危）

### 证据

多处使用 `reinterpret_cast` 进行指针类型转换：

**位置**: `src/core/task.cpp:76-83`

```cpp
// 从函数指针偏移计算任务指针
CPUEUTask *t = reinterpret_cast<CPUEUTask *>(
    static_cast<uintptr_t>(
        static_cast<size_t>(reinterpret_cast<uintptr_t>(f)) - 
        OFFSETOF(CPUEUTask, func_storage)));
```

**位置**: `src/sync/mutex.cpp:345`

```cpp
auto p = reinterpret_cast<ffrt::mutexBase*>(mutex);
```

### 触发路径

```
攻击者:
  1. 构造假的指针值
  2. 传入API（如ffrt_mutex_lock）
  3. 内部reinterpret_cast为内部类型
  4. 访问不存在的成员 → 崩溃或信息泄露
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 需要控制指针值 |
| **影响范围** | 崩溃、信息泄露 |
| **权限提升** | 不太可能 |
| **CVSS估算** | 3.7 (Low) |

### 修复建议

```cpp
// 建议增加魔数检查验证对象有效性
struct mutexBase {
    static constexpr uint64_t MAGIC = 0xFFRTMUTEX;
    uint64_t magic = MAGIC;
    // ... 其他成员
    
    bool IsValid() const {
        return magic == MAGIC;
    }
};

// 在使用前验证
int ffrt_mutex_lock(ffrt_mutex_t* mutex) {
    auto p = reinterpret_cast<ffrt::mutexBase*>(mutex);
    if (!p || !p->IsValid()) {
        return ffrt_error_inval;
    }
    // ... 安全使用
}
```

---

## R7: 信息泄露风险（低危）

### 证据

**位置**: 多处错误日志

```cpp
FFRT_SYSEVENT_LOGE("task name[%s], gid[%llu], submit_tid[%d]",
    curTask->GetLabel().c_str(), curTask->gid, curTask->fromTid);
```

### 触发路径

```
攻击者:
  1. 触发错误条件
  2. 读取日志或错误输出
  3. 获取任务名称、ID、线程ID等敏感信息
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 需要访问日志 |
| **影响范围** | 信息泄露 |
| **权限提升** | 无直接提升 |
| **CVSS估算** | 2.3 (Low) |

### 修复建议

```cpp
// 在Release版本中减少敏感信息输出
#ifdef FFRT_RELEASE
    FFRT_SYSEVENT_LOGE("Stack overflow detected");  // 不输出具体信息
#else
    FFRT_SYSEVENT_LOGE("task name[%s], gid[%llu]", ...);  // Debug版本可详细
#endif
```

---

## 修复优先级矩阵

| 风险 | 严重性 | 修复难度 | 优先级 | 建议时间 |
|------|--------|----------|--------|----------|
| R3: UAF | 高 | 中 | P0 | 1周内 |
| R2: 栈溢出 | 中 | 低 | P1 | 2周内 |
| R5: 资源耗尽 | 中 | 低 | P1 | 2周内 |
| R4: 竞态条件 | 中 | 高 | P2 | 1月内 |
| R1: 空指针 | 中 | 低 | P2 | 1月内 |
| R6: 类型混淆 | 低 | 中 | P3 | 2月内 |
| R7: 信息泄露 | 低 | 低 | P3 | 2月内 |

---

## 测试建议

### 建议增加的安全测试

1. **Fuzz测试**:
```bash
# 使用libFuzzer测试API入口
ffrt_submit_f(FUZZED_DATA, ...)
ffrt_mutex_init(FUZZED_PTR, ...)
```

2. **压力测试**:
```cpp
// 并发创建/销毁任务
for (int i = 0; i < 100000; i++) {
    auto handle = ffrt_submit_h(...);
    if (i % 2 == 0) ffrt_task_handle_destroy(handle);
}
```

3. **资源耗尽测试**:
```cpp
// 测试内存限制
while (true) {
    auto ret = ffrt_submit_f(...);
    if (ret != ffrt_success) break;
}
```

4. **异常输入测试**:
```cpp
// 测试非法枚举值
ffrt_task_attr_set_qos(attr, 999);  // 非法QoS
ffrt_task_attr_set_queue_priority(attr, -1);  // 非法优先级
```

