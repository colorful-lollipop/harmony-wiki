# 攻击面分析

> FFRT 外部输入与敏感操作识别
> 面向安全研究员的入口点指南

---

## 1. 外部输入清单

### 1.1 API 参数输入

FFRT 所有外部输入均通过 C/C++ API 接口传入，主要入口：

#### 任务相关 API

| API 函数 | 文件 | 风险参数 | 说明 |
|----------|------|----------|------|
| `ffrt_submit_base` | `src/core/task.cpp:329` | `f` (函数指针) | 任务执行函数 |
| `ffrt_submit_h_base` | `src/core/task.cpp:366` | `f`, `attr` | 带句柄的任务提交 |
| `ffrt_submit_f` | `src/core/task.cpp:358` | `func`, `arg` | 简化任务提交 |
| `ffrt_task_attr_init` | `src/core/task.cpp:128` | `attr` (指针) | 任务属性初始化 |
| `ffrt_task_attr_set_name` | `src/core/task.cpp:153` | `name` (字符串) | 任务名称 |
| `ffrt_task_attr_set_qos` | `src/core/task.cpp:174` | `qos` (整型) | QoS等级 |
| `ffrt_task_attr_set_delay` | `src/core/task.cpp:199` | `delay_us` | 延迟时间 |
| `ffrt_task_attr_set_timeout` | `src/core/task.cpp:226` | `timeout_us` | 超时时间 |
| `ffrt_task_attr_set_stack_size` | `src/core/task.cpp:299` | `size` | 栈大小 |
| `ffrt_wait_deps` | `src/core/task.cpp:436` | `deps` (依赖数组) | 等待依赖 |

**证据**：
```cpp
// src/core/task.cpp:329
void ffrt_submit_base(ffrt_function_header_t *f, const ffrt_deps_t *in_deps, 
    const ffrt_deps_t *out_deps, const ffrt_task_attr_t *attr)
{
    if (unlikely(!f)) {  // ✓ 有检查
        FFRT_LOGE("function handler should not be empty");
        return;
    }
    // ...
}
```

#### 队列相关 API

| API 函数 | 文件 | 风险参数 | 说明 |
|----------|------|----------|------|
| `ffrt_queue_create` | `src/queue/queue_api.cpp` | `type`, `name` | 队列创建 |
| `ffrt_queue_submit` | `src/queue/queue_api.cpp:45` | `queue`, `f` | 队列任务提交 |
| `ffrt_queue_submit_h` | `src/queue/queue_api.cpp` | `queue`, `f` | 带句柄提交 |

**证据**：
```cpp
// src/queue/queue_api.cpp:45
void ffrt_queue_submit(ffrt_queue_t queue, ffrt_function_header_t* f, 
    const ffrt_task_attr_t* attr)
{
    FFRT_COND_DO_ERR((queue == nullptr), return, "input invalid, queue is nullptr");
    FFRT_COND_DO_ERR((f == nullptr), return, "input invalid, f is nullptr");
    // ...
}
```

#### 同步原语 API

| API 函数 | 文件 | 风险参数 | 说明 |
|----------|------|----------|------|
| `ffrt_mutex_init` | `src/sync/mutex.cpp:335` | `mutex` | 互斥锁初始化 |
| `ffrt_mutex_lock` | `src/sync/mutex.cpp:350` | `mutex` | 加锁 |
| `ffrt_mutex_unlock` | `src/sync/mutex.cpp:378` | `mutex` | 解锁 |
| `ffrt_mutex_destroy` | `src/sync/mutex.cpp:391` | `mutex` | 销毁 |
| `ffrt_cond_init` | `src/sync/condition_variable.cpp` | `cond` | 条件变量初始化 |
| `ffrt_cond_wait` | `src/sync/condition_variable.cpp` | `cond`, `mutex` | 等待 |
| `ffrt_rwlock_init` | `src/sync/shared_mutex.cpp:170` | `rwlock` | 读写锁初始化 |
| `ffrt_rwlock_rdlock` | `src/sync/shared_mutex.cpp:187` | `rwlock` | 读锁 |
| `ffrt_rwlock_wrlock` | `src/sync/shared_mutex.cpp:210` | `rwlock` | 写锁 |

**证据**：
```cpp
// src/sync/mutex.cpp:335
API_ATTRIBUTE((visibility("default")))
int ffrt_mutex_init(ffrt_mutex_t* mutex, const ffrt_mutexattr_t* attr)
{
    FFRT_COND_DO_ERR((mutex == nullptr), return ffrt_error_inval, "input invalid, mutex is nullptr");
    // ...
}
```

#### 事件循环 API

| API 函数 | 文件 | 风险参数 | 说明 |
|----------|------|----------|------|
| `ffrt_loop_create` | `src/core/loop_api.cpp:27` | `loop` | 循环创建 |
| `ffrt_loop_start` | `src/core/loop_api.cpp:62` | `loop` | 启动循环 |
| `ffrt_loop_stop` | `src/core/loop_api.cpp:78` | `loop` | 停止循环 |
| `ffrt_loop_register_fd` | `src/core/loop_api.cpp:89` | `fd` (文件描述符) | 注册FD |

---

### 1.2 函数闭包输入

任务函数和参数通过函数指针传递：

```cpp
// 结构定义: interfaces/kits/c/type_def.h:30-40
typedef struct {
    ffrt_function_kind_t kind;
    void (*exec)(void*);      // 执行函数指针
    void (*destroy)(void*);   // 销毁函数指针
} ffrt_function_header_t;
```

**风险**：
- `exec` 函数指针可能被恶意设置
- `destroy` 函数指针可能指向非法地址
- 闭包数据 `arg` 可能被构造为攻击载体

**证据**：
```cpp
// src/core/task.cpp:67-84
void DestroyFunctionWrapper(ffrt_function_header_t* f, ...)
{
    if (f == nullptr || f->destroy == nullptr) {  // ✓ 有检查
        return;
    }
    f->destroy(f);  // 调用用户提供的销毁函数
    // ...
}
```

---

### 1.3 内存指针输入

依赖数据通过指针数组传递：

```cpp
// interfaces/kits/c/type_def.h:42-48
typedef enum {
    ffrt_dependence_data,
    ffrt_dependence_task,
} ffrt_dependence_type_t;

typedef struct {
    ffrt_dependence_type_t type;
    void* ptr;              // 数据指针
    uint64_t len;           // 数据长度
} ffrt_dependence_t;
```

**风险**：
- `ptr` 可能指向无效内存
- `len` 可能被恶意设置导致越界
- 依赖解析时可能触发非法内存访问

---

### 1.4 配置参数输入

构建时配置参数：`ffrt.gni`

```
declare_args() {
  ffrt_support_enable = true
  ffrt_async_stack_enable = true
  ffrt_task_local_enable = false
  ffrt_allocator_mmap_size = "8 * 1024 * 1024"    // 8MB
  ffrt_stack_size = "1 << 20"                      // 1MB
}
```

**运行时影响**：
- 栈大小影响协程内存占用
- 分配器大小影响内存映射行为
- 特性开关影响代码路径

---

## 2. 敏感操作清单

### 2.1 系统调用

| 系统调用 | 位置 | 用途 | 风险 |
|----------|------|------|------|
| `ioctl` | `src/eu/rtg_perf_ctrl.cpp` | RTG性能控制 | 特权操作 |
| `ioctl` | `src/eu/rtg_ioctl.cpp` | RTG控制 | 特权操作 |
| `ioctl` | `src/eu/qos_interface.cpp` | QoS控制 | 特权操作 |
| `futex` | `src/sync/sync.cpp` | 用户态锁 | 阻塞操作 |
| `gettid` | `include/internal_inc/osal.h` | 获取线程ID | 信息泄露 |
| `mmap/munmap` | `include/util/slab.h` | 内存分配 | 内存管理 |
| `mprotect` | `src/eu/co_routine.cpp:286` | 栈保护 | 内存保护 |

**证据**：
```cpp
// src/eu/co_routine.cpp:286
static void CoSetStackProt(CoRoutine* co, int prot)
{
    size_t p_size = getpagesize();
    uint64_t mp = reinterpret_cast<uint64_t>(co->stkMem.stk);
    mp = (mp + p_size - 1) / p_size * p_size;
    int ret = mprotect(reinterpret_cast<void *>(mp), p_size, prot);
    FFRT_UNLIKELY_COND_DO_ABORT(ret < 0, "...");
}
```

---

### 2.2 内存管理操作

| 操作 | 位置 | 说明 | 风险 |
|------|------|------|------|
| `mmap` | `src/eu/co_routine.cpp:299-304` | 协程栈分配 | 内存耗尽 |
| `new/malloc` | `src/sync/wait_queue.cpp:48` | WaitEntry分配 | 内存泄漏 |
| `new` | `src/tm/cpu_task.cpp:84` | in_handles分配 | UAF风险 |
| placement new | `src/dm/sdependence_manager.cpp:88` | 任务创建 | 内存重叠 |
| `dlopen` | 多处 | 动态库加载 | 代码注入 |

**证据**：
```cpp
// src/eu/co_routine.cpp:299-304
if (stackSize != defaultStackSize) {
    co = static_cast<CoRoutine*>(mmap(nullptr, stackSize,
        PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_PRIVATE, -1, 0));
    if (co == reinterpret_cast<CoRoutine*>(MAP_FAILED)) {
        FFRT_SYSEVENT_LOGE("memory mmap failed.");
        return nullptr;
    }
}
```

---

### 2.3 线程管理操作

| 操作 | 位置 | 说明 | 风险 |
|------|------|------|------|
| `pthread_create` | `src/eu/worker_thread.cpp` | Worker线程创建 | 资源耗尽 |
| `pthread_key_create` | `src/eu/co_routine.cpp:84` | TLS创建 | 泄漏 |
| `pthread_setspecific` | `src/eu/co_routine.cpp:97` | TLS设置 | UAF |

---

### 2.4 特权接口

| 接口 | 位置 | 用途 | 访问控制 |
|------|------|------|----------|
| `ffrt_set_cpu_worker_max_num` | `src/core/task.cpp:478` | 设置Worker数量 | 需检查权限 |
| `ffrt_set_cgroup_attr` | `src/core/task.cpp:457` | 设置cgroup属性 | 需特权 |
| `ffrt_set_sched_mode` | `src/core/task.cpp:729` | 设置调度模式 | 需特权 |
| `ffrt_enable_worker_escape` | `src/core/task.cpp:715` | 启用Worker逃逸 | 需特权 |

**证据**：
```cpp
// src/core/task.cpp:478
int ffrt_set_cpu_worker_max_num(ffrt_qos_t qos, uint32_t num)
{
    if (num == 0 || num > ffrt::QOS_WORKER_MAXNUM) {
        FFRT_LOGE("qos[%d] worker num[%u] is invalid.", qos, num);
        return -1;
    }
    // ...
    return ffrt::FFRTFacade::GetEUInstance().SetWorkerMaxNum(_qos, num);
}
```

---

## 3. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           不可信区域 (Untrusted)                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     用户应用程序                                   │   │
│  │  • 恶意构造的API参数                                               │   │
│  │  • 非法函数指针 (exec/destroy)                                     │   │
│  │  • 无效内存指针 (in_deps/out_deps)                                 │   │
│  │  • 恶意闭包数据                                                    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│                              ▼ FFRT C/C++ API                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    FFRT 运行时 (libffrt.z.so)                     │   │
│  │  ┌─────────────────────────────────────────────────────────┐   │   │
│  │  │                   输入验证层                             │   │   │
│  │  │  • nullptr检查                                          │   │   │
│  │  │  • 范围检查 (delay_us, timeout_us)                      │   │   │
│  │  │  • 枚举值检查 (qos, priority)                           │   │   │
│  │  └─────────────────────────────────────────────────────────┘   │   │
│  │                              │                                  │   │
│  │                              ▼                                 │   │
│  │  ┌─────────────────────────────────────────────────────────┐   │   │
│  │  │                   核心业务逻辑                           │   │   │
│  │  │  • 任务调度                                             │   │   │
│  │  │  • 依赖管理                                             │   │   │
│  │  │  • 协程切换                                             │   │   │
│  │  │  • 内存分配                                             │   │   │
│  │  └─────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│                              ▼ 系统调用                                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      内核 / 系统服务                              │   │
│  │  • mmap/mprotect (内存管理)                                       │   │
│  │  • ioctl (RTG/QoS控制)                                           │   │
│  │  • futex (同步原语)                                              │   │
│  │  • pthread (线程管理)                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                           可信区域 (Trusted)                             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 攻击向量汇总

### 4.1 输入验证绕过

| 向量 | 描述 | 风险等级 |
|------|------|----------|
| **空指针** | 传递 nullptr 触发未检查路径 | 高 |
| **越界枚举** | 传入非法 QoS/优先级值 | 中 |
| **超大数值** | delay_us/timeout_us 溢出 | 中 |
| **无效句柄** | 使用已销毁的任务句柄 | 高 |

### 4.2 内存安全

| 向量 | 描述 | 风险等级 |
|------|------|----------|
| **UAF** | 操作已释放的任务/队列 | 高 |
| **Double Free** | 重复销毁资源 | 高 |
| **内存耗尽** | 大量创建任务/协程 | 中 |
| **栈溢出** | 递归任务导致栈溢出 | 中 |

### 4.3 并发安全

| 向量 | 描述 | 风险等级 |
|------|------|----------|
| **竞态条件** | 多线程操作同一任务 | 高 |
| **死锁** | 错误的锁顺序 | 中 |
| **TOCTOU** | 检查与使用的时间差 | 中 |

### 4.4 逻辑漏洞

| 向量 | 描述 | 风险等级 |
|------|------|----------|
| **资源耗尽** | 创建过多Worker线程 | 中 |
| **优先级反转** | 高优先级任务饿死 | 低 |
| **信息泄露** | 通过错误消息获取敏感信息 | 低 |

---

## 5. 关键检查点

### 5.1 输入验证检查

| 检查项 | 位置 | 状态 |
|--------|------|------|
| 指针非空检查 | 326+ 处使用 `FFRT_COND_DO_ERR` | ✅ 完善 |
| delay_us 范围 | `src/core/task.cpp:199-209` | ✅ 有上限 |
| timeout_us 范围 | `src/core/task.cpp:226-244` | ✅ 有下限和上限 |
| qos 枚举检查 | `src/core/task.cpp:180-184` | ✅ 通过映射表转换 |
| 栈大小检查 | `src/eu/co_routine.cpp:292-318` | ✅ 动态分配 |

### 5.2 权限检查

| 接口 | 权限检查 | 状态 |
|------|----------|------|
| `ffrt_set_cpu_worker_max_num` | 数值范围检查 | ⚠️ 无身份校验 |
| `ffrt_set_cgroup_attr` | attr非空检查 | ⚠️ 无特权检查 |
| `ffrt_enable_worker_escape` | 范围检查 | ⚠️ 无身份校验 |

---

## 6. 审计建议

### 6.1 重点关注区域

1. **任务提交路径**：`src/core/task.cpp` - 所有API入口
2. **协程切换**：`src/eu/co_routine.cpp` - 栈管理和切换
3. **依赖管理**：`src/dm/sdependence_manager.cpp` - 指针解引用
4. **同步原语**：`src/sync/` - 锁的实现
5. **内存分配**：`include/util/slab.h` - 自定义分配器

### 6.2 建议检查项

- [ ] 所有 `reinterpret_cast` 的合法性
- [ ] 引用计数增减的配对性
- [ ] 锁的获取/释放平衡
- [ ] 错误处理路径的资源释放
- [ ] 边界条件的完整性

