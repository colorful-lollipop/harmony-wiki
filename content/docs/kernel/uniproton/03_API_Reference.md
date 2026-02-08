# UniProton API 参考

## 概述

UniProton 内核提供 **C 原生 API**，所有 API 均以 `PRT_` 为前缀。无 N-API / JavaScript 绑定层。

> **重要**: 本内核为**裸机 RTOS**，无用户/内核态分离，无进程概念，仅有任务 (Task)。

## API 分类

### 1. 任务管理 (Task Management)

#### 核心函数

| 函数名 | 功能 | 头文件 | 同步/异步 |
|--------|------|--------|----------|
| `PRT_TaskCreate` | 创建任务 | `prt_task.h` | 同步 |
| `PRT_TaskDelete` | 删除任务 | `prt_task.h` | 同步 |
| `PRT_TaskSuspend` | 挂起任务 | `prt_task.h` | 同步 |
| `PRT_TaskResume` | 恢复任务 | `prt_task.h` | 同步 |
| `PRT_TaskDelay` | 任务延时 | `prt_task.h` | 同步 |
| `PRT_TaskLock` | 禁止调度 | `prt_task.h` | 同步 |
| `PRT_TaskUnlock` | 允许调度 | `prt_task.h` | 同步 |
| `PRT_TaskYield` | 让出 CPU | `prt_task.h` | 同步 |
| `PRT_TaskGetPriority` | 获取优先级 | `prt_task.h` | 同步 |
| `PRT_TaskSetPriority` | 设置优先级 | `prt_task.h` | 同步 |
| `PRT_TaskSelf` | 获取自身 PID | `prt_task.h` | 同步 |

#### 任务属性

```c
/* 任务名最大长度 */
#define OS_TSK_NAME_LEN 16

/* 优先级范围: 0-31 (0最高, 31为IDLE) */
#define OS_TSK_PRIORITY_00 0
/* ... */
#define OS_TSK_PRIORITY_31 31
```

**证据**: `src/include/uapi/prt_task.h` 第 40-100 行

---

### 2. 信号量 (Semaphore)

#### 核心函数

| 函数名 | 功能 | 头文件 | 说明 |
|--------|------|--------|------|
| `PRT_SemCreate` | 创建信号量 | `prt_sem.h` | 支持计数/二元 |
| `PRT_SemDelete` | 删除信号量 | `prt_sem.h` | - |
| `PRT_SemPend` | P 操作 | `prt_sem.h` | 等待信号量 |
| `PRT_SemPost` | V 操作 | `prt_sem.h` | 释放信号量 |
| `PRT_SemGetInfo` | 获取信息 | `prt_sem.h` | - |
| `PRT_SemGetCount` | 获取计数 | `prt_sem.h` | - |

#### 信号量模式

- **计数信号量**: 初始计数 0-0xFFFFFFFE
- **二元信号量**: 类似互斥锁，支持优先级继承

**证据**: `src/core/ipc/sem/prt_sem.c` 第 174 行 (PRT_SemPend), 第 273 行 (PRT_SemPost)

---

### 3. 消息队列 (Message Queue)

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_QueueCreate` | 创建队列 | `prt_queue.h` |
| `PRT_QueueDelete` | 删除队列 | `prt_queue.h` |
| `PRT_QueueWrite` | 写入消息 | `prt_queue.h` |
| `PRT_QueueRead` | 读取消息 | `prt_queue.h` |
| `PRT_QueuePeak` | peek 消息 | `prt_queue.h` |

#### 消息优先级

- `OS_QUEUE_PRI_NORMAL`: 普通优先级
- `OS_QUEUE_PRI_URGENT`: 紧急优先级

**证据**: `src/core/ipc/queue/prt_queue.c` 第 129 行 (PRT_QueueRead), 第 263 行 (PRT_QueueWrite)

---

### 4. 事件标志 (Event)

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_EventCreate` | 创建事件 | `prt_event.h` |
| `PRT_EventRead` | 读取事件 | `prt_event.h` |
| `PRT_EventWrite` | 写事件 | `prt_event.h` |

#### 等待模式

- `OS_EVENT_ANY`: 等待任意位
- `OS_EVENT_ALL`: 等待所有位

**证据**: `src/core/ipc/event/prt_event.c` 第 98 行 (PRT_EventRead), 第 162 行 (PRT_EventWrite)

---

### 5. 定时器 (Timer)

#### 软件定时器

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_SwtmrCreate` | 创建软件定时器 | `prt_swtmr.h` |
| `PRT_SwtmrStart` | 启动定时器 | `prt_swtmr.h` |
| `PRT_SwtmrStop` | 停止定时器 | `prt_swtmr.h` |
| `PRT_SwtmrDelete` | 删除定时器 | `prt_swtmr.h` |

#### 系统 Tick

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_TickGet` | 获取 tick 数 | `prt_tick.h` |
| `PRT_TickSet` | 设置 tick | `prt_tick.h` |

**证据**: `src/core/kernel/timer/swtmr/prt_swtmr.c` 及相关文件

---

### 6. 内存管理 (Memory)

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_MemAlloc` | 动态分配 | `prt_mem.h` |
| `PRT_MemFree` | 释放内存 | `prt_mem.h` |

#### 固定大小块分配

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_FscMemAlloc` | 分配固定块 | `prt_fscmem.h` |
| `PRT_FscMemFree` | 释放固定块 | `prt_fscmem.h` |

**证据**: `src/mem/prt_mem.c`, `src/mem/fsc/prt_fscmem.c`

---

### 7. 中断处理 (Hardware Interrupt)

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_HwiCreate` | 创建中断 | `prt_hwi.h` |
| `PRT_HwiDelete` | 删除中断 | `prt_hwi.h` |
| `PRT_HwiEnable` | 使能中断 | `prt_hwi.h` |
| `PRT_HwiDisable` | 禁能中断 | `prt_hwi.h` |
| `PRT_HwiTrigger` | 触发中断 | `prt_hwi.h` |

**证据**: `src/core/kernel/irq/prt_irq.c`

---

### 8. 异常处理 (Exception)

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_ExcInit` | 初始化异常 | `prt_exc.h` |
| `PRT_ExcHandler` | 异常处理 | `prt_exc.h` |

---

### 9. CPU 占用率 (CPU Usage)

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_CpupGet` | 获取 CPU 占用率 | `prt_cpup.h` |

**证据**: `src/om/cpup/prt_cpup.c`

---

### 10. Hook 机制

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_HookUserCreate` | 创建用户 Hook | `prt_hook.h` |

**证据**: `src/om/hook/prt_hook_init.c`

---

### 11. 系统控制

#### 核心函数

| 函数名 | 功能 | 头文件 |
|--------|------|--------|
| `PRT_SysInit` | 系统初始化 | `prt_sys.h` |
| `PRT_SysStart` | 启动系统 | `prt_sys.h` |
| `PRT_SysReboot` | 系统重启 | `prt_sys.h` |

---

### 12. 错误码

所有 API 返回 `U32` 类型错误码：

```c
#define OS_ERRNO_TSK_NO_MEMORY             0x02000100U
#define OS_ERRNO_TSK_PRIORITY_ERROR        0x02000200U
#define OS_ERRNO_SEM_INVALID_ARGS          0x02000300U
#define OS_ERRNO_QUEUE_INVALID_ARGS         0x02000400U
/* ... 更多错误码 ... */
```

**证据**: `src/include/uapi/prt_errno.h`

---

## POSIX 兼容层 (OSAL)

UniProton 提供 POSIX 兼容接口，路径: `src/osal/posix/`

| 接口 | 实现文件 |
|------|----------|
| `sem_init/sem_wait/sem_post` | `prt_posix_sem.c` |
| `pthread_mutex_*` | `prt_posix_mutex.c` |
| `pthread_rwlock_*` | `prt_pthread_rwlock.c` |

**证据**: `src/osal/posix/` 目录

---

## N-API 状态

**UniProton 内核中无 N-API 实现。**

N-API 绑定需要在上层框架中实现 (如 OpenHarmony 的 ace_engine 或 ability_runtime 层)。

---

## 相关文档

- [概览](./01_Overview.md) - 项目定位
- [目录结构](./02_Directory_Structure.md) - 代码组织
- [架构设计](./04_Architecture.md) - 调用关系
