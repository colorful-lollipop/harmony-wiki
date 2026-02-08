# 内部实现细节

> FFRT 核心类与内部机制详解
> 面向开发者的深度指南

---

## 1. 核心类职责

### 1.1 门面模式：FFRTFacade

**位置**: `src/util/ffrt_facade.h`

FFRT使用门面模式统一组件访问，是所有内部组件的入口点。

```cpp
class FFRTFacade {
public:
    static ExecuteUnit& GetEUInstance();      // 执行单元
    static DependenceManager& GetDMInstance(); // 依赖管理
    static Scheduler* GetSchedInstance();      // 调度器
    static IOPoller& GetPPInstance();          // IO轮询器
    static TimerManager& GetTMInstance();      // 定时器管理
    static DelayedWorker& GetDWInstance();     // 延迟任务
    static QueueMonitor& GetQMInstance();      // 队列监控
    static WorkerMonitor& GetWMInstance();     // Worker监控
};
```

**设计意图**:
- 解耦组件间的直接依赖
- 便于单元测试时Mock
- 统一管理组件生命周期

---

### 1.2 任务类层次

```
TaskBase (抽象基类)
├── CoTask (协程任务基类)
│   ├── CPUEUTask (CPU执行任务)
│   │   └── SCPUEUTask (支持依赖的CPU任务)
│   │       └── RootTask (根任务)
│   └── QueueTask (队列任务)
└── UVTask (libuv任务)
```

#### TaskBase - 任务基类

**位置**: `include/tm/task_base.h:80-145`

```cpp
class TaskBase {
public:
    // 任务类型
    ffrt_task_type_t type;           // normal_task / queue_task
    
    // 标识
    uint64_t gid;                    // 全局唯一ID
    std::string label;               // 任务名称
    int qos_;                        // QoS等级
    
    // 状态
    TaskStatus status;               // 当前状态
    BlockType blockType;             // 阻塞类型
    
    // 引用计数
    std::atomic_uint32_t deleteRef_;
    
    // 虚函数接口
    virtual void Execute() = 0;
    virtual void Ready() = 0;
    virtual void Wake() = 0;
    virtual void Cancel() = 0;
    virtual void SetQos(const QoS& newQos) = 0;
};
```

**状态机**:
```
PENDING → SUBMITTED → READY → POPPED → EXECUTING → FINISH
              ↓           ↓          ↓
         CANCELED   COROUTINE_BLOCK  THREAD_BLOCK
                        ↓              ↓
                   (CoYield)     (条件变量等待)
```

#### CPUEUTask - CPU任务

**位置**: `include/tm/cpu_task.h:51-139`

```cpp
class CPUEUTask : public CoTask {
public:
    // 任务关系
    CPUEUTask* parent = nullptr;     // 父任务
    std::atomic<uint64_t> childNum{0};  // 子任务数
    
    // 依赖
    std::vector<CPUEUTask*>* in_handles_ = nullptr;  // 输入依赖
    std::vector<VersionCtx*> ins_;   // 输入版本上下文
    std::vector<VersionCtx*> outs_;  // 输出版本上下文
    
    // 特性
    uint64_t delayTime = 0;          // 延迟时间
    TimeoutTask timeoutTask;         // 超时任务
    bool notifyWorker_ = true;       // 是否通知Worker
    
    #ifdef FFRT_TASK_LOCAL_ENABLE
    TaskLocalAttr* tlsAttr = nullptr;  // Task Local Storage
    #endif
    
    // 方法
    bool IsRoot() const { return parent == nullptr; }
    void SetInHandles(std::vector<CPUEUTask*>& in_handles);
};
```

---

### 1.3 依赖管理：VersionCtx

**位置**: `src/core/entity.h`

VersionCtx是FFRT依赖管理的核心数据结构，使用双向链表管理数据版本。

```cpp
class alignas(64) VersionCtx {
public:
    // 版本状态
    enum class VersionStatus : uint8_t {
        UNREADY = 0,      // 未就绪
        READY,            // 就绪（生产者完成）
        CONSUMED,         // 已消费
    };
    
    // 生产者-消费者链表
    VersionCtx* prev = nullptr;   // 前一个版本
    VersionCtx* next = nullptr;   // 后一个版本
    VersionCtx* child = nullptr;  // 子版本（嵌套任务）
    
    // 关联任务
    std::vector<SCPUEUTask*> consumers;  // 消费者列表
    SCPUEUTask* producer = nullptr;       // 生产者
    
    // 状态
    VersionStatus status;
    fast_mutex criticalMutex_;  // 保护临界区
    
    // 方法
    void AddConsumer(SCPUEUTask* task);
    void AddProducer(SCPUEUTask* task);
    void onProduced();
    void onConsumed();
};
```

**工作原理**:
```
数据D的版本链:
    V1(UNREADY) ← V2(UNREADY) ← V3(UNREADY)
     ↑producer      ↑producer      ↑producer
     T1             T2             T3
     
当T1完成:
    V1(READY) ← V2(UNREADY) ← V3(UNREADY)
     ↓consumers
     T2, T3
```

---

### 1.4 协程：CoRoutine

**位置**: `include/eu/co_routine.h:40-70`

```cpp
struct CoRoutine {
    // 上下文
    CoCtx ctx;                    // 协程上下文（寄存器状态）
    CoRoutineEnv* thEnv;          // 所属线程环境
    ffrt::CoTask* task;           // 关联任务
    
    // 栈内存
    StackMem stkMem;              // 栈内存描述
    size_t allocatedSize;         // 分配的内存大小
    
    // 状态
    std::atomic<int> status;      // 协程状态
    bool isTaskDone = false;      // 任务是否完成
    
    // 事务
    int cpuBoostCtxId = -1;       // CPU boost上下文
    
    #ifdef ASAN_MODE
    void* asanFakeStack;          // ASan影子栈
    void* asanFiberAddr;
    size_t asanFiberSize;
    #endif
};

// 协程环境（每个线程一个）
struct CoRoutineEnv {
    CoRoutine* runningCo = nullptr;   // 当前运行的协程
    CoCtx schCtx;                      // 调度器上下文
    std::function<bool(ffrt::CoTask*)>* pending = nullptr;
};
```

**栈内存布局**:
```
CoRoutine结构（栈底）
    ↓
    [CoRoutine对象: ~200字节]
    ↓
    [栈内存: 1MB - sizeof(CoRoutine)]
    ↓
栈顶 (运行时向下增长)
    ↓
    [保护页: 4KB, PROT_READ]
```

---

### 1.5 调度器：Scheduler

**位置**: `include/sched/scheduler.h:35-65`

```cpp
class Scheduler {
public:
    virtual ~Scheduler() = default;
    
    // 任务入队
    virtual void PushTaskGlobal(TaskBase* task) = 0;
    virtual void TaskEnqueue(TaskBase* task) = 0;
    
    // 任务出队
    virtual TaskBase* PickNextTask() = 0;
    virtual TaskBase* PopTask() = 0;
    
    // Worker管理
    virtual int IncWorker(int qos) = 0;
    virtual void WorkerRetired(int qos) = 0;
    
    // 查询
    virtual bool GetWorkerIdleState() const = 0;
    virtual int GetTaskCount() const = 0;
};
```

**调度策略**:
```cpp
// src/sched/stask_scheduler.cpp
class STaskScheduler : public TaskScheduler {
public:
    // 本地队列 + 全局队列
    SpmcQueue* GetLocalQueue();
    
    // 任务窃取
    int StealTask();
    
    // 优先级栈
    TaskBase* PopTaskLocalOrPriority();
};
```

---

## 2. 内部API契约

### 2.1 稳定接口

以下接口在多个版本间保持稳定：

| 接口 | 位置 | 稳定性 |
|------|------|--------|
| `ffrt_submit_base` | `interfaces/kits/c/task.h` | ✅ 稳定 |
| `ffrt_wait` | `interfaces/kits/c/task.h` | ✅ 稳定 |
| `ffrt_mutex_*` | `interfaces/kits/c/mutex.h` | ✅ 稳定 |
| `ffrt_queue_*` | `interfaces/kits/c/queue.h` | ✅ 稳定 |

### 2.2 内部实现细节（可能变更）

| 接口/类 | 位置 | 稳定性 | 说明 |
|---------|------|--------|------|
| `CoRoutine`结构 | `include/eu/co_routine.h` | ⚠️ 不稳定 | 内部实现 |
| `VersionCtx` | `src/core/entity.h` | ⚠️ 不稳定 | 依赖管理 |
| `FFRTFacade` | `src/util/ffrt_facade.h` | ⚠️ 不稳定 | 内部门面 |
| `TaskScheduler` | `include/sched/task_scheduler.h` | ⚠️ 不稳定 | 调度器 |

---

## 3. 资源生命周期

### 3.1 任务生命周期

```
创建 (Create)
    ↓ placement new in TaskFactory<SCPUEUTask>::Alloc()
初始化 (Init)
    ↓ SCPUEUTask构造函数
    • 设置parent、qos、gid
    • 初始化引用计数
就绪 (Ready)
    ↓ task->Ready()
    • 状态变为READY
    • 加入调度队列
执行 (Execute)
    ↓ task->Execute() in CoStartEntry
    • 状态变为EXECUTING
    • 运行用户函数
完成 (Finish)
    ↓ onTaskDone()
    • 通知消费者
    • 释放依赖
销毁 (Destroy)
    ↓ ~SCPUEUTask()
    • 引用计数归零
    • 回收内存到Slab
```

### 3.2 协程生命周期

```
分配 (Alloc)
    ↓ AllocNewCoRoutine()
    • mmap或Slab分配
    • 设置栈大小
    
初始化 (Init)
    ↓ ffrt_fiber_init()
    • 设置入口函数CoStartEntry
    • 初始化寄存器上下文
    
运行 (Running)
    ↓ CoStart()
    • 从线程切换到协程
    • 执行任务函数
    
让出 (Yield)
    ↓ CoYield()
    • 保存协程状态
    • 切回线程
    
恢复 (Resume)
    ↓ CoStart()循环
    • 从yield点恢复
    • 继续执行
    
完成 (Done)
    ↓ CoExit()
    • 标记isTaskDone
    • 切回线程不再返回
    
释放 (Free)
    ↓ CoMemFree()
    • munmap或Slab回收
```

### 3.3 Owner关系图

```
Thread (Owner)
    └── CoRoutineEnv
        └── CoRoutine (runningCo)
            └── CoTask
                ├── VersionCtx (ins/outs)
                │   ├── prev/next VersionCtx
                │   └── consumers (other tasks)
                └── parent/child tasks

ExecuteUnit (Owner)
    └── CPUWorkerGroup
        ├── mutex (保护Worker列表)
        └── CPUWorker
            └── thread

DependenceManager (Owner)
    └── VersionCtx (全局实体)
        └── linked list
```

---

## 4. 关键数据结构

### 4.1 任务属性

**位置**: `include/core/task_attr_private.h`

```cpp
class task_attr_private {
public:
    std::string name_;                    // 任务名称
    QoS qos_ = QoS();                     // QoS等级
    uint64_t delay_ = 0;                  // 延迟时间(us)
    uint64_t timeout_ = 0;                // 超时时间(us)
    ffrt_queue_priority_t prio_ = ffrt_queue_priority_immediate;
    uint64_t stackSize_ = 0;              // 栈大小
    bool notifyWorker_ = true;            // 是否通知Worker
    #ifdef FFRT_TASK_LOCAL_ENABLE
    bool taskLocal_ = false;              // Task Local标志
    #endif
};
```

### 4.2 依赖描述

**位置**: `interfaces/kits/c/type_def.h:50-56`

```cpp
typedef struct {
    uint32_t len;                         // 依赖数量
    ffrt_dependence_t* items;             // 依赖数组
} ffrt_deps_t;

typedef struct {
    ffrt_dependence_type_t type;          // DATA or TASK
    union {
        void* ptr;                        // 数据指针
        ffrt_task_handle_t task;          // 任务句柄
    };
} ffrt_dependence_t;
```

### 4.3 队列属性

**位置**: `interfaces/kits/c/type_def.h:58-68`

```cpp
typedef struct {
    uint64_t timeout_ = 0;                // 超时时间
    uint64_t monitorTimeout_ = 0;         // 监控超时
    uint64_t monitorId_ = 0;              // 监控ID
    int maxConcurrency_ = -1;             // 最大并发数(-1=无限制)
    char name_[MAX_NAME_LENGTH] = {0};    // 队列名称
} queue_attr_private;
```

---

## 5. 内存管理

### 5.1 Slab分配器

**位置**: `include/util/slab.h`

```cpp
template<typename T>
class SimpleAllocator {
public:
    static T* Alloc() {
        // 从freelist获取内存
        if (freeList != nullptr) {
            T* t = freeList;
            freeList = freeList->next;
            return t;
        }
        // 申请新内存块
        Expand();
        return Alloc();
    }
    
    static void Free(T* t) {
        t->~T();  // 析构
        t->next = freeList;
        freeList = t;
    }
    
private:
    static void Expand() {
        // mmap大块内存
        char* p = reinterpret_cast<char*>(
            mmap(nullptr, MmapSz, PROT_READ | PROT_WRITE, 
                 MAP_ANONYMOUS | MAP_PRIVATE, -1, 0));
        // 分割为小块加入freelist
    }
};
```

**使用方式**:
```cpp
// 定义分配器
using TaskFactory = SimpleAllocator<SCPUEUTask>;

// 分配任务
SCPUEUTask* task = TaskFactory::Alloc();
new(task) SCPUEUTask(attr, parent, gid);  // placement new构造

// 释放任务
task->~SCPUEUTask();  // 显式析构
TaskFactory::Free(task);
```

### 5.2 协程栈分配策略

| 栈大小 | 分配方式 | 释放方式 |
|--------|----------|----------|
| 默认(1MB) | Slab分配器 | Slab回收 |
| 自定义 | mmap | munmap |

```cpp
// 默认栈大小使用Slab
co = ffrt::CoRoutineAllocMem(stackSize);  // Slab

// 自定义栈大小使用mmap
co = static_cast<CoRoutine*>(mmap(nullptr, stackSize,
    PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_PRIVATE, -1, 0));
```

---

## 6. 扩展阅读

| 主题 | 文档 |
|------|------|
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 对外接口 | [04_Interface.md](04_Interface.md) |
| 代码地图 | [03_CodeMap.md](03_CodeMap.md) |
| 安全分析 | [06_SecurityReview.md](06_SecurityReview.md) |

