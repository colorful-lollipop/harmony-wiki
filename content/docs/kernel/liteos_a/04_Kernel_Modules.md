# 内核模块详解

## 进程管理

**代码位置**: `kernel/base/core/los_process.c`

### 进程控制块 (PCB)

```c
// 关键结构
typedef struct {
    UINT32      processID;        // 进程ID
    UINT32      status;           // 进程状态
    UINT32      priority;         // 优先级
    UINT32      uid;              // 用户ID
    UINT32      gid;              // 组ID
    VOID        *vmSpace;        // 虚拟地址空间
    LosTaskCB   *taskGroup;      // 任务组
    // ...
} LosProcessCB;
```

### 进程状态

| 状态 | 说明 |
|------|------|
| `OS_PROCESS_STATUS_READY` | 就绪 |
| `OS_PROCESS_STATUS_RUNNING` | 运行中 |
| `OS_PROCESS_STATUS_PENDING` | 等待 |
| `OS_PROCESS_STATUS_SUSPEND` | 挂起 |
| `OS_PROCESS_STATUS_ZOMBIES` | 僵尸 |

### 关键 API

| API | 用途 |
|-----|------|
| `LosCreateProcess()` | 创建进程 |
| `OsExitProcess()` | 退出进程 |
| `OsGetProcessID()` | 获取进程ID |
| `OsGetParentProcessID()` | 获取父进程ID |

## 任务/线程管理

**代码位置**: `kernel/base/core/los_task.c`, `kernel/include/los_task.h`

### 任务控制块 (TCB)

```c
typedef struct {
    UINT32      taskID;           // 任务ID
    CHAR        *taskName;        // 任务名
    UINT16      taskStatus;       // 任务状态
    UINT8       priority;         // 优先级
    UINT32      stackSize;        // 栈大小
    VOID        *stackPtr;        // 栈指针
    UINT64      startTime;        // 启动时间
    UINT64      endTime;          // 结束时间
    // ...
} LosTaskCB;
```

### 任务状态机

```
                    +-----------+
    创建 -------->   |  Ready    |
                    +-----------+
                     |     ↑
                     |     |
                     ↓     |
    退出 --------+   +-----------+   <------- 唤醒
    终止        |   |  Running  |
               +-->+-----------+<---------+
                    ↑                     |
                    |                     |
                    |      +--------+      |
                    |     | Pending| <----+
                    +------+--------+
                           |
                           | 超时/信号
                           ↓
                    +-----------+
                    | Suspend   |
                    +-----------+
```

### 调度策略

| 策略 | 说明 |
|------|------|
| **优先级抢占** | 高优先级任务抢占低优先级 |
| **时间片轮转** | 同优先级任务时间片轮转 |
| **FIFO** | 先来先服务 |

### 关键 API

| API | 用途 |
|-----|------|
| `LOS_TaskCreate()` | 创建任务 |
| `LOS_TaskDelete()` | 删除任务 |
| `LOS_TaskSuspend()` | 挂起任务 |
| `LOS_TaskResume()` | 恢复任务 |
| `LOS_TaskDelay()` | 任务延时 |
| `LOS_TaskYield()` | 让出 CPU |

## 内存管理

**代码位置**: `kernel/base/mem/`, `kernel/base/vm/`

### 内存分配器

| 类型 | 文件 | 用途 |
|------|------|------|
| **TLSF** | `kernel/base/mem/tlsf/los_memory.c` | 动态内存分配 |
| **MemBox** | `kernel/base/mem/membox/los_membox.c` | 固定大小内存池 |
| **Slab** | - | 内核对象缓存 |

### 关键 API

| API | 用途 |
|-----|------|
| `LOS_MemAlloc()` | 分配内存 |
| `LOS_MemFree()` | 释放内存 |
| `LOS_MemRealloc()` | 重新分配 |
| `LOS_MemAllocAlign()` | 对齐分配 |
| `LOS_MemInit()` | 初始化堆 |

### 虚拟内存

**代码位置**: `kernel/base/vm/`

```c
// 关键结构
typedef struct {
    LosRbTree     rbTree;        // 红黑树管理
    LosAVLTree    avlTree;       // AVL 树管理
    VADDR_T       startAddr;    // 起始地址
    UINT32        size;          // 大小
    // ...
} LosVmSpace;
```

## IPC 机制

**代码位置**: `kernel/base/ipc/`

### 互斥锁 (Mutex)

**代码位置**: `kernel/include/los_mux.h`

```c
typedef struct {
    UINT16      muxStat;         // 状态
    UINT16      priority;        // 优先级
    UINT32      holdCount;       // 持有计数
    UINT32      taskID;         // 持有任务ID
    // ...
} LosMuxCB;
```

| API | 用途 |
|-----|------|
| `LOS_MuxCreate()` | 创建互斥锁 |
| `LOS_MuxDelete()` | 删除互斥锁 |
| `LOS_MuxLock()` | 加锁 |
| `LOS_MuxUnlock()` | 解锁 |

### 信号量 (Semaphore)

**代码位置**: `kernel/include/los_sem.h`

| API | 用途 |
|-----|------|
| `LOS_SemCreate()` | 创建信号量 |
| `LOS_SemDelete()` | 删除信号量 |
| `LOS_SemPend()` | P 操作 |
| `LOS_SemPost()` | V 操作 |

### 事件 (Event)

**代码位置**: `kernel/include/los_event.h`

```c
typedef struct {
    UINT32      eventId;         // 事件ID
    UINT32      eventMask;      // 事件掩码
    UINT32      eventBits;      // 事件位
    // ...
} LosEventCB;
```

| API | 用途 |
|-----|------|
| `LOS_EventInit()` | 初始化事件 |
| `LOS_EventRead()` | 读取事件 |
| `LOS_EventWrite()` | 写入事件 |
| `LOS_EventClear()` | 清除事件 |

### 消息队列 (Queue)

**代码位置**: `kernel/include/los_queue.h`

| API | 用途 |
|-----|------|
| `LOS_QueueCreate()` | 创建队列 |
| `LOS_QueueDelete()` | 删除队列 |
| `LOS_QueueWrite()` | 写入队列 |
| `LOS_QueueRead()` | 读取队列 |

### 快速用户态锁 (Futex)

**代码位置**: `kernel/base/ipc/los_futex.c`

## 定时器

**代码位置**: `kernel/base/core/los_swtmr.c`, `kernel/include/los_swtmr.h`

### 软件定时器

```c
typedef struct {
    UINT8       timerId;        // 定时器ID
    UINT8       timerType;      // 定时器类型
    UINT16      mode;           // 模式
    UINT32      interval;       // 间隔
    UINT32      arg;            // 参数
    TimerCB     callback;       // 回调函数
    // ...
} SwtmrCB;
```

| 定时器类型 | 说明 |
|-----------|------|
| `OS_SWTMR_MODE_ONCE` | 单次触发 |
| `OS_SWTMR_MODE_PERIOD` | 周期触发 |
| `OS_SWTMR_MODE_NO_SELF_DELETE` | 不自删除 |

## SMP 支持

**代码位置**: `kernel/include/los_smp.h`, `kernel/base/core/los_smp.c`

| 功能 | 说明 |
|------|------|
| **多核启动** | `_secondaryStartUp()` |
| **负载均衡** | `OsSmpSchedTask()` |
| **核间中断** | `LOS_SmpSendIpi()` |
| **亲和性** | `LOS_CpupriSet()` |

## 扩展模块

### 动态加载 (dynload)
**代码位置**: `kernel/extended/dynload/`

- ELF 文件解析
- 动态链接
- `dlopen()` / `dlsym()` 支持

### 容器 (container)
**代码位置**: `kernel/extended/container/`

- 进程命名空间
- 用户命名空间
- 网络命名空间
- 挂载命名空间

### vDSO
**代码位置**: `kernel/extended/vdso/`

- 虚拟动态共享对象
- 快速系统调用

### 性能追踪 (trace/perf)
**代码位置**: `kernel/extended/trace/`, `kernel/extended/perf/`

- ftrace 兼容
- CPU 使用率统计
- 性能事件采样

## 相关文档

- [架构说明](/02_Architecture.md)
- [系统调用接口](/05_Syscall_API.md)
- [安全评审](/06_Security_Review.md)
