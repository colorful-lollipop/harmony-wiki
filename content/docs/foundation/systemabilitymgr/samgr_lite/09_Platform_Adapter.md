# 平台适配

## 概述

samgr_lite 支持 M-core 和 A-core 两种硬件平台，通过适配层屏蔽底层差异。

## 平台差异对比

| 特性 | M-core | A-core |
|------|--------|--------|
| 处理器 | Cortex-M / RISC-V | Cortex-A / RISC-V |
| 内存 | < 512KB | > 512KB |
| 文件系统 | 无/轻量级 | 完整 |
| API 标准 | CMSIS | POSIX |
| IPC 支持 | 可选 | 完整 |
| 库类型 | 静态库 (.a) | 动态库 (.so) |

## 适配层结构

```
services/samgr_lite/samgr/adapter/
├── posix/                    # A-core 适配
│   ├── lock_free_queue.h     # 无锁队列
│   ├── lock_free_queue.c
│   ├── memory_adapter.c      # 内存适配
│   ├── queue_adapter.h       # 队列适配
│   ├── queue_adapter.c
│   ├── thread_adapter.c      # 线程适配
│   ├── thread_adapter.h
│   ├── time_adapter.c        # 时间适配
│   └── time_adapter.h
│
└── cmsis/                    # M-core 适配
    ├── memory_adapter.c
    ├── queue_adapter.c
    ├── thread_adapter.c
    ├── time_adapter.c
    └── cmsis_dispatch.h
```

**证据位置**: `samgr/adapter/BUILD.gn:26-51`

## 适配接口

### 线程适配

**M-core** (`cmsis/thread_adapter.c`):
```c
// 基于 CMSIS-RTOS
osThreadId_t SAMGR_CreateThread(const char *name, Entry entry, void *arg,
                                uint32 stackSize, uint8 priority);
```

**A-core** (`posix/thread_adapter.c`):
```c
// 基于 POSIX threads
pthread_t SAMGR_CreateThread(const char *name, Entry entry, void *arg,
                            uint32 stackSize, uint8 priority);
```

### 队列适配

**M-core** (`cmsis/queue_adapter.c`):
```c
// 基于 CMSIS-RTOS 消息队列
MQueueId SAMGR_CreateQueue(uint32 queueSize, uint32 msgSize);
int SAMGR_PushQueue(MQueueId queueId, void *msg);
```

**A-core** (`posix/queue_adapter.c`):
```c
// 基于 POSIX 消息队列或自定义实现
MQueueId SAMGR_CreateQueue(uint32 queueSize, uint32 msgSize);
int SAMGR_PushQueue(MQueueId queueId, void *msg);
```

### 时间适配

**M-core** (`cmsis/time_adapter.c`):
```c
// 基于 CMSIS-RTOS
uint64 SAMGR_GetTickTime(void);
void SAMGR_Sleep(uint32 seconds);
void SAMGR_MsDelay(uint32 ms);
```

**A-core** (`posix/time_adapter.c`):
```c
// 基于 POSIX time
uint64 SAMGR_GetTickTime(void);
void SAMGR_Sleep(uint32 seconds);
void SAMGR_MsDelay(uint32 ms);
```

## 条件编译

### 平台检测

```c
#if defined(__CMSIS_RTOS) || defined(__liteos_m__)
    // M-core 代码
#elif defined(__linux__) || defined(__unix__)
    // A-core 代码
#endif
```

### GN 配置

**M-core**:
```gn
if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
  static_library("samgr") {
    # 静态库构建
  }
}
```

**A-core**:
```gn
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  shared_library("samgr") {
    cflags = ["-fPIC"]  # 位置无关代码
  }
}
```

## 适配层使用

### 头文件包含

```c
#include "samgr/adapter/thread_adapter.h"
#include "samgr/adapter/queue_adapter.h"
#include "samgr/adapter/time_adapter.h"
```

### 创建线程

```c
static void TaskEntry(void *arg)
{
    while (1) {
        // 任务逻辑
        SAMGR_MsDelay(100);
    }
}

// 创建任务
osThreadId_t taskId = SAMGR_CreateThread(
    "ExampleTask",
    TaskEntry,
    NULL,
    0x800,          // 栈大小
    PRI_BELOW_NORMAL  // 优先级
);
```

### 消息队列

```c
// 创建队列
MQueueId queueId = SAMGR_CreateQueue(20, sizeof(Request));

// 发送消息
Request request = {...};
SAMGR_PushQueue(queueId, &request);

// 接收消息
Request msg;
SAMGR_PopQueue(queueId, &msg, 0);
```

## RPC 适配

### 标准 RPC（默认）

**证据位置**: `samgr_server/BUILD.gn:17-44`

```gn
shared_library("server") {
  sources = ["source/samgr_server_rpc.c"]
  # 使用 IPC (Binder) 框架
  public_deps = [
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
  ]
}
```

### Mini RPC（可选）

**证据位置**: `config.gni:20`

```gn
enable_ohos_systemabilitymgr_samgr_lite_rpc_mini = true
```

```gn
if (enable_ohos_systemabilitymgr_samgr_lite_rpc_mini) {
  static_library("server") {
    defines = ["MINI_SAMGR_LITE_RPC"]
    # 使用 DBinder 框架
    public_deps = [
      "//foundation/communication/ipc/interfaces/innerkits/c/dbinder:dbinder",
    ]
  }
}
```

## 平台选择建议

### M-core 适用场景

- 资源极度受限（RAM < 512KB）
- 简单的单进程应用
- 无需跨进程通信

### A-core 适用场景

- 需要多进程隔离
- 需要完整的 IPC 机制
- 需要动态加载能力
- 资源充足（RAM > 512KB）

## 下一章

- [内部实现](./10_Internal_Implementation.md) - 核心模块详解
- [GN 构建系统](./08_Build_System.md) - 构建配置
