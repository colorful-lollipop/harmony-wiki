# 内核 API - 04_Kernel_API

## 概述

LiteOS-M 内核提供 C 语言 API，所有 API 以 `LOS_` 为前缀。API 头文件位于 `kernel/include/` 目录。

## 错误码规范

### 错误码格式

```
0xAABBCDDD
│ │││└─── 子错误码 (8位)
│ ││└───── 模块错误码 (8位)
│ │└────── 保留 (8位)
│ └───────┘ 严重等级 (8位)
└────────── 保留 (8位)
```

### 错误码示例

| 错误码 | 定义位置 | 说明 |
|--------|----------|------|
| `LOS_ERRNO_TSK_NO_MEMORY` | los_task.h:60 | 任务创建内存不足 |
| `LOS_ERRNO_TSK_PTR_NULL` | los_task.h:70 | 空指针参数 |
| `LOS_ERRNO_TSK_PRIOR_ERROR` | los_task.h:90 | 优先级错误 |

> 参考: `kernel/include/los_task.h:60-100`

## API 分类

### 1. 任务管理 (los_task.h)

| API | 说明 | 返回值 |
|-----|------|--------|
| `LOS_TaskCreate()` | 创建任务 | `UINT32` (任务 ID) |
| `LOS_TaskDelete()` | 删除任务 | `UINT32` (错误码) |
| `LOS_TaskSuspend()` | 挂起任务 | `UINT32` (错误码) |
| `LOS_TaskResume()` | 恢复任务 | `UINT32` (错误码) |
| `LOS_TaskDelay()` | 任务延时 | `UINT32` (错误码) |
| `LOS_TaskPriSet()` | 设置优先级 | `UINT32` (错误码) |
| `LOS_TaskPriGet()` | 获取优先级 | `UINT32` (错误码) |
| `LOS_TaskYield()` | 让出 CPU | `UINT32` (错误码) |

### 2. 消息队列 (los_queue.h)

| API | 说明 | 返回值 |
|-----|------|--------|
| `LOS_QueueCreate()` | 创建队列 | `UINT32` (队列 ID) |
| `LOS_QueueWrite()` | 写入队列 | `UINT32` (错误码) |
| `LOS_QueueRead()` | 读取队列 | `UINT32` (错误码) |
| `LOS_QueueDelete()` | 删除队列 | `UINT32` (错误码) |
| `LOS_QueueInfoGet()` | 获取队列信息 | `UINT32` (错误码) |

### 3. 互斥锁 (los_mux.h)

| API | 说明 | 返回值 |
|-----|------|--------|
| `LOS_MuxCreate()` | 创建互斥锁 | `UINT32` (锁 ID) |
| `LOS_MuxLock()` | 获取锁 | `UINT32` (错误码) |
| `LOS_MuxUnlock()` | 释放锁 | `UINT32` (错误码) |
| `LOS_MuxDelete()` | 删除互斥锁 | `UINT32` (错误码) |

### 4. 信号量 (los_sem.h)

| API | 说明 | 返回值 |
|-----|------|--------|
| `LOS_SemCreate()` | 创建信号量 | `UINT32` (信号量 ID) |
| `LOS_SemPost()` | 释放信号量 | `UINT32` (错误码) |
| `LOS_SemPend()` | 获取信号量 | `UINT32` (错误码) |
| `LOS_SemDelete()` | 删除信号量 | `UINT32` (错误码) |

### 5. 事件 (los_event.h)

| API | 说明 | 返回值 |
|-----|------|--------|
| `LOS_EventInit()` | 初始化事件 | `UINT32` (错误码) |
| `LOS_EventWrite()` | 写事件 | `UINT32` (错误码) |
| `LOS_EventRead()` | 读事件 | `UINT32` (事件值) |
| `LOS_EventClear()` | 清除事件 | `UINT32` (错误码) |

### 6. 软件定时器 (los_swtmr.h)

| API | 说明 | 返回值 |
|-----|------|--------|
| `LOS_SwtmrCreate()` | 创建定时器 | `UINT32` (定时器 ID) |
| `LOS_SwtmrStart()` | 启动定时器 | `UINT32` (错误码) |
| `LOS_SwtmrStop()` | 停止定时器 | `UINT32` (错误码) |
| `LOS_SwtmrDelete()` | 删除定时器 | `UINT32` (错误码) |

## 任务管理示例

```c
#include "los_task.h"

UINT32 MyTaskEntry(UINT32 arg)
{
    (void)arg;
    // 任务主体
    LOS_TaskDelay(100);
    return LOS_OK;
}

void CreateMyTask(void)
{
    UINT32 ret;
    TSK_INIT_PARAM_S taskParam;

    taskParam.usTaskPrio = 10;  // 优先级 (0-31)
    taskParam.pfnTaskEntry = (TSK_ENTRY_FUNC)MyTaskEntry;
    taskParam.uwStackSize = 0x1000;  // 栈大小
    taskParam.pcName = "MyTask";

    ret = LOS_TaskCreate(NULL, &taskParam);
    if (ret != LOS_OK) {
        // 错误处理
    }
}
```

## API 调用链示例

### 任务创建到调度

```
LOS_TaskCreate()
    ↓ los_task.c:500+
    ↓ 参数校验
    ↓ 内存分配 (LOS_MemAlloc)
    ↓ 任务控制块 (TCB) 初始化
    ↓ 挂载到就绪链表 (OsTaskAddToReadyList)
    ↓ LosSchedReady()
    ↓ 触发调度 (可选)
```

## 稳定性标注

| 接口类型 | 位置 | 稳定性 | 说明 |
|----------|------|--------|------|
| 核心 API | `kernel/include/los_*.h` | **稳定** | 对外公开接口 |
| 内部 API | `kernel/src/` | 不稳定 | 实现细节，可能变化 |
| 扩展 API | `utils/` | 视情况 | 工具函数 |

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](03_Architecture.md)
- [内核抽象层](05_KAL.md)
