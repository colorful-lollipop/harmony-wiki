# 目录结构与代码地图

> FFRT 项目目录结构导航
> 帮助快速定位核心代码

---

## 1. 顶层目录结构

```
/Volumes/lexar/code/d/work/oh/foundation/resourceschedule/ffrt/
├── benchmarks/           # 性能测试用例
├── docs/                 # 用户指南文档
├── examples/             # 使用案例
├── include/              # 对外头文件（41个头文件）
├── interfaces/           # 对外接口目录
├── scripts/              # 脚本工具
├── src/                  # 源码实现（85+源文件）
├── test/                 # 测试代码（**本文档忽略**）
├── tools/                # 工具（ffrt_trace_process）
└── wiki/                 # Wiki文档（本目录）
```

---

## 2. 核心目录详解

### 2.1 interfaces/ - 对外接口

```
interfaces/
├── inner_api/           # 内部接口（系统组件使用）
│   ├── c/              # C内部API
│   │   ├── deadline.h
│   │   ├── executor_task.h
│   │   ├── ffrt_dump.h
│   │   ├── ffrt_ipc.h
│   │   ├── init.h
│   │   ├── queue_ext.h
│   │   ├── task_ext.h
│   │   └── type_def_ext.h
│   └── cpp/            # C++内部API
│       ├── deadline.h
│       ├── future.h
│       └── task_ext.h
└── kits/               # 对外公开API
    ├── c/              # C API
    │   ├── condition_variable.h
    │   ├── loop.h
    │   ├── mutex.h
    │   ├── queue.h
    │   ├── shared_mutex.h
    │   ├── sleep.h
    │   ├── task.h          # 核心任务API
    │   ├── timer.h
    │   └── type_def.h      # 基础类型定义
    ├── cpp/            # C++ API
    │   ├── condition_variable.h
    │   ├── mutex.h
    │   ├── queue.h
    │   ├── shared_mutex.h
    │   ├── sleep.h
    │   ├── task.h          # C++任务封装
    │   └── pattern/        # 高级模式
    │       ├── job_partner.h
    │       ├── job_ring.h
    │       └── job_utils.h
    └── ffrt.h              # 统一入口头文件
```

**关键文件**：
| 文件 | 职责 |
|------|------|
| `interfaces/kits/ffrt.h` | 主入口头文件，自动选择C/C++ |
| `interfaces/kits/c/task.h` | C任务API（submit/wait/attr） |
| `interfaces/kits/c/type_def.h` | QoS、错误码、存储大小定义 |

---

### 2.2 include/ - 内部头文件

```
include/
├── core/               # 核心类型
│   └── task_attr_private.h
├── dm/                 # 依赖管理
│   └── dependence_manager.h
├── dfx/                # 维测功能
│   ├── async_stack/
│   ├── bbox/
│   ├── log/
│   ├── trace/
│   ├── trace_record/
│   └── watchdog/
├── eu/                 # 执行单元
│   ├── blockaware.h
│   ├── co_routine.h
│   ├── cpu_worker.h
│   ├── execute_unit.h
│   └── thread_group.h
├── internal_inc/       # 内部公共头文件
│   ├── non_copyable.h
│   ├── osal.h
│   └── types.h
├── sched/              # 调度器
│   ├── execute_ctx.h
│   ├── qos.h
│   ├── scheduler.h
│   └── task_runqueue.h
├── sync/               # 同步原语
│   ├── delayed_worker.h
│   └── sync.h
├── tm/                 # 任务管理
│   ├── cpu_task.h
│   ├── task_base.h
│   ├── task_factory.h
│   └── uv_task.h
└── util/               # 工具类
    ├── capability.h
    ├── ffrt_ring_buffer.h
    ├── linked_list.h
    ├── slab.h
    └── spmc_queue.h
```

**关键类定位**：
| 类名 | 文件 | 职责 |
|------|------|------|
| `DependenceManager` | `include/dm/dependence_manager.h` | 依赖管理 |
| `ExecuteUnit` | `include/eu/execute_unit.h` | 执行单元 |
| `CoRoutine` | `include/eu/co_routine.h` | 协程实现 |
| `Scheduler` | `include/sched/scheduler.h` | 任务调度 |
| `TaskBase` | `include/tm/task_base.h` | 任务基类 |
| `CPUEUTask` | `include/tm/cpu_task.h` | CPU任务 |

---

### 2.3 src/ - 源码实现

```
src/
├── core/               # 核心API实现
│   ├── entity.cpp
│   ├── loop_api.cpp
│   ├── poller_api.cpp
│   ├── task.cpp          # 任务提交核心（~740行）
│   ├── task_io.cpp
│   ├── timer_api.cpp
│   └── version_ctx.cpp
├── dfx/                # 维测功能实现
│   ├── async_stack/
│   ├── bbox/
│   ├── dump/
│   ├── log/
│   ├── sysevent/
│   ├── trace/
│   ├── trace_record/
│   └── watchdog/
├── dm/                 # 依赖管理实现
│   ├── dependence_manager.cpp
│   └── sdependence_manager.cpp
├── eu/                 # 执行单元实现（~29个文件）
│   ├── base_poller.cpp
│   ├── co2_context.c     # 协程上下文（汇编）
│   ├── co_routine.cpp    # 协程实现（~584行）
│   ├── co_routine_factory.cpp
│   ├── cpu_worker.cpp    # CPU Worker
│   ├── execute_unit.cpp
│   ├── io_poller.cpp
│   ├── loop.cpp
│   ├── loop_poller.cpp
│   ├── qos_convert.cpp
│   ├── qos_interface.cpp
│   ├── rtg_ioctl.cpp
│   ├── rtg_perf_ctrl.cpp
│   ├── sexecute_unit.cpp
│   └── worker_thread.cpp
├── ipc/                # IPC实现
│   └── ipc.cpp           # 极简实现（57行）
├── queue/              # 队列实现（~21个文件）
│   ├── base_queue.cpp
│   ├── concurrent_queue.cpp
│   ├── eventhandler_adapter_queue.cpp
│   ├── queue_api.cpp
│   ├── queue_handler.cpp   # 队列处理器（~800行）
│   ├── queue_monitor.cpp
│   ├── serial_queue.cpp
│   └── traffic_record.cpp
├── sched/              # 调度器实现
│   ├── deadline.cpp
│   ├── execute_ctx.cpp
│   ├── frame_interval.cpp
│   ├── interval.cpp
│   ├── load_tracking.cpp
│   ├── multi_workgroup.cpp
│   ├── qos.cpp
│   ├── sched_deadline.cpp
│   ├── scheduler.cpp
│   ├── stask_scheduler.cpp
│   └── task_scheduler.cpp
├── sync/               # 同步原语实现（~20个文件）
│   ├── condition_variable.cpp
│   ├── delayed_worker.cpp
│   ├── mutex.cpp
│   ├── mutex_private.h
│   ├── perf_counter.cpp
│   ├── record_mutex.cpp
│   ├── shared_mutex.cpp
│   ├── sleep.cpp
│   ├── sync.cpp
│   ├── thread.cpp
│   ├── timer_manager.cpp
│   └── wait_queue.cpp
├── tm/                 # 任务管理实现
│   ├── cpu_task.cpp
│   ├── io_task.cpp
│   ├── queue_task.cpp
│   ├── scpu_task.cpp
│   ├── task_base.cpp
│   ├── task_factory.cpp
│   └── uv_task.cpp
└── util/               # 工具实现（~24个文件）
    ├── capability.cpp
    ├── cpu_boost_wrapper.cpp
    ├── ffrt_cpu_boost.cpp
    ├── ffrt_facade.cpp     # 门面模式实现
    ├── graph_check.cpp
    ├── init.cpp
    ├── slab.cpp
    ├── spmc_queue.cpp
    ├── time_format.cpp
    ├── white_list.cpp
    └── worker_monitor.cpp
```

---

## 3. 代码导航图

### 3.1 功能 → 文件映射

| 功能 | 入口文件 | 实现文件 |
|------|----------|----------|
| **任务提交** | `interfaces/kits/c/task.h` | `src/core/task.cpp:329` |
| **任务等待** | `interfaces/kits/c/task.h` | `src/core/task.cpp:436` |
| **QoS设置** | `interfaces/kits/c/task.h` | `src/core/task.cpp:174` |
| **互斥锁** | `interfaces/kits/c/mutex.h` | `src/sync/mutex.cpp` |
| **条件变量** | `interfaces/kits/c/condition_variable.h` | `src/sync/condition_variable.cpp` |
| **串行队列** | `interfaces/kits/c/queue.h` | `src/queue/serial_queue.cpp` |
| **并发队列** | `interfaces/kits/c/queue.h` | `src/queue/concurrent_queue.cpp` |
| **定时器** | `interfaces/kits/c/timer.h` | `src/core/timer_api.cpp` |
| **事件循环** | `interfaces/kits/c/loop.h` | `src/core/loop_api.cpp` |

### 3.2 核心类 → 文件映射

| 类名 | 头文件 | 实现文件 | 职责 |
|------|--------|----------|------|
| `DependenceManager` | `include/dm/dependence_manager.h` | `src/dm/sdependence_manager.cpp` | 依赖管理 |
| `ExecuteUnit` | `include/eu/execute_unit.h` | `src/eu/sexecute_unit.cpp` | 执行单元 |
| `CoRoutine` | `include/eu/co_routine.h` | `src/eu/co_routine.cpp` | 协程实现 |
| `Scheduler` | `include/sched/scheduler.h` | `src/sched/stask_scheduler.cpp` | 任务调度 |
| `TaskBase` | `include/tm/task_base.h` | `src/tm/task_base.cpp` | 任务基类 |
| `CPUEUTask` | `include/tm/cpu_task.h` | `src/tm/cpu_task.cpp` | CPU任务 |
| `SCPUEUTask` | `src/tm/scpu_task.h` | `src/tm/scpu_task.cpp` | 支持依赖的CPU任务 |
| `QueueTask` | `include/tm/queue_task.h` | `src/tm/queue_task.cpp` | 队列任务 |
| `VersionCtx` | `src/core/version_ctx.h` | `src/core/version_ctx.cpp` | 版本上下文 |
| `Entity` | `src/core/entity.h` | `src/core/entity.cpp` | 实体管理 |

### 3.3 关键符号 → 文件映射

| 符号 | 文件 | 行号 | 说明 |
|------|------|------|------|
| `ffrt_submit_base` | `src/core/task.cpp` | 329 | 任务提交C API |
| `ffrt_submit_h_base` | `src/core/task.cpp` | 366 | 带句柄的任务提交 |
| `ffrt_wait` | `src/core/task.cpp` | 451 | 等待所有任务 |
| `ffrt_wait_deps` | `src/core/task.cpp` | 436 | 等待依赖任务 |
| `CoStart` | `src/eu/co_routine.cpp` | 430 | 协程启动 |
| `CoYield` | `src/eu/co_routine.cpp` | 510 | 协程让出 |
| `onSubmit` | `src/dm/sdependence_manager.cpp` | 69 | 依赖管理提交处理 |
| `onTaskDone` | `src/dm/sdependence_manager.cpp` | 253 | 任务完成处理 |

---

## 4. 构建相关文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | GN构建入口，定义libffrt目标 |
| `ffrt.gni` | GN构建参数配置 |
| `bundle.json` | 组件配置，声明系统能力 |
| `CMakeLists.txt` | CMake构建入口 |
| `hisysevent.yaml` | 系统事件定义 |
| `ffrt_whitelist.conf` | 白名单配置 |

---

## 5. 快速定位指南

### 5.1 我想了解...

| 想了解 | 查看文件 |
|--------|----------|
| **API怎么用** | `interfaces/kits/c/task.h` + `examples/` |
| **任务提交实现** | `src/core/task.cpp` |
| **调度器如何工作** | `src/sched/stask_scheduler.cpp` + `src/sched/scheduler.h` |
| **协程切换原理** | `src/eu/co_routine.cpp` + `src/eu/co2_context.c` |
| **依赖管理** | `src/dm/sdependence_manager.cpp` + `src/core/entity.cpp` |
| **内存分配** | `include/util/slab.h` + `src/eu/co_routine.cpp:292` |
| **同步原语** | `src/sync/mutex.cpp` + `src/sync/condition_variable.cpp` |
| **队列实现** | `src/queue/queue_handler.cpp` + `src/queue/serial_queue.cpp` |
| **QoS管理** | `src/sched/qos.cpp` + `src/eu/qos_interface.cpp` |
| **日志输出** | `src/dfx/log/ffrt_log.cpp` + `include/dfx/log/ffrt_log_api.h` |
| **Trace追踪** | `src/dfx/trace/ffrt_trace.cpp` |

### 5.2 调试线索

| 问题 | 查看位置 |
|------|----------|
| 任务未执行 | `src/sched/` - 检查调度队列 |
| 协程崩溃 | `src/eu/co_routine.cpp:55` - 栈溢出检查 |
| 内存泄漏 | `include/util/slab.h` - 自定义分配器 |
| 死锁 | `src/sync/wait_queue.cpp` - 等待队列 |
| 性能问题 | `src/sched/load_tracking.cpp` - 负载追踪 |

---

## 6. 代码统计

| 类别 | 数量 |
|------|------|
| 源码文件(.cpp/.c) | ~85个 |
| 头文件(.h) | ~41个 |
| 对外API文件 | 19个（C:11 + C++:8） |
| 核心模块 | 10个 |

