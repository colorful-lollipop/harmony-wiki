# 内核抽象层 - 05_KAL

## 概述

KAL (Kernel Abstraction Layer) 提供标准 API 接口封装，使上层代码可移植。

## 目录结构

```
kal/
├── cmsis/          # CMSIS-RTOS API
├── posix/          # POSIX API
├── libc/           # C 标准库
├── libsec/         # 安全库
└── BUILD.gn
```

## CMSIS-RTOS 支持

### 概述

`kal/cmsis/` 提供 ARM CMSIS-RTOS 标准接口，方便从其他 RTOS 移植。

### 主要 API

| CMSIS API | 对应 LOS API |
|-----------|--------------|
| `osThreadNew` | `LOS_TaskCreate` |
| `osMutexNew` | `LOS_MuxCreate` |
| `osSemaphoreNew` | `LOS_SemCreate` |
| `osMessageQueueNew` | `LOS_QueueCreate` |
| `osTimerNew` | `LOS_SwtmrCreate` |

### 头文件

```
kal/cmsis/
└── include/
    ├── os.h              # CMSIS-RTOS 主头文件
    ├── osKernel.h        # 内核控制
    ├── osThread.h        # 线程管理
    ├── osMutex.h         # 互斥锁
    ├── osSemaphore.h     # 信号量
    └── osMessageQueue.h  # 消息队列
```

## POSIX API 支持

### 概述

`kal/posix/` 提供 POSIX 标准接口，兼容 Unix/Linux 应用。

### 主要 API 分类

| 分类 | 头文件 | 说明 |
|------|--------|------|
| 线程 | `pthread.h` | POSIX 线程 |
| 互斥锁 | `pthread.h` | 互斥锁、条件变量 |
| 定时器 | `time.h` | 定时器接口 |
| 消息队列 | `mqueue.h` | POSIX 消息队列 |
| 信号 | `signal.h` | 信号处理 |

### 示例

```c
#include "pthread.h"

void* thread_func(void* arg)
{
    // POSIX 线程函数
    return NULL;
}

void create_posix_thread(void)
{
    pthread_t thread;
    pthread_attr_t attr;

    pthread_attr_init(&attr);
    pthread_attr_setstacksize(&attr, 0x2000);
    pthread_create(&thread, &attr, thread_func, NULL);
}
```

## C 标准库

### 概述

`kal/libc/` 提供 C 标准库实现，用于内核环境的字符串、内存操作等。

### 主要功能

| 分类 | 头文件 | 说明 |
|------|--------|------|
| 字符串 | `string.h` | 字符串操作 |
| 内存 | `stdlib.h` | 内存分配 |
| IO | `stdio.h` | 标准 IO (受限) |

## 安全库

### 概述

`kal/libsec/` 提供基础安全功能，如加密、哈希等。

### 组件

| 组件 | 说明 |
|------|------|
| 基础加密 | 对称/非对称加密接口 |
| 哈希 | SHA256 等摘要算法 |
| HMAC | 消息认证码 |

## API 映射关系

```
用户代码 (CMSIS/POSIX)
       ↓
kal/cmsis/ 或 kal/posix/
       ↓
内部转换层
       ↓
LOS_* 内核 API (kernel/include/los_*.h)
       ↓
kernel/src/los_*.c
       ↓
ArchTaskSchedule (arch/)
```

## 相关文档

- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [内核 API](04_Kernel_API.md)
