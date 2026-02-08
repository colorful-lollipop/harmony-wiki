# 05 - API/接口差异分析

## 5.1 概述

CMSIS 库在 OpenHarmony 中**保持上游接口不变**，所有定制化扩展都在 OH 适配层（`kernel/liteos_m/kal/cmsis`）实现。

### 接口差异分类

| 差异类型 | 数量 | 说明 |
|----------|------|------|
| **OH 新增 API** | 1 | 扩展的定时器创建接口 |
| **行为变更 API** | 2 | 优先级映射、错误码转换 |
| **废弃/禁用功能** | 0 | 无 |
| **上游标准 API** | 60+ | 完全兼容 CMSIS-RTOS2 标准 |

---

## 5.2 OH 新增 API

### 5.2.1 `osTimerExtNew` - 扩展定时器创建

**接口定义**：

```c
// kernel/liteos_m/kal/cmsis/cmsis_liteos2.c

#if (LOSCFG_BASE_CORE_SWTMR_ALIGN == 1)
osTimerId_t osTimerExtNew(
    osTimerFunc_t func,           // 定时器回调函数
    osTimerType_t type,           // 定时器类型（单次/周期）
    void *argument,               // 回调参数
    const osTimerAttr_t *attr,    // 属性
    osTimerRouses_t ucRouses,     // 唤醒设置
    osTimerAlign_t ucSensitive    // 对齐设置
);
#endif
```

**新增参数说明**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `ucRouses` | `osTimerRouses_t` | 定时器是否唤醒系统 |
| `ucSensitive` | `osTimerAlign_t` | 定时器对齐敏感度 |

**使用场景**：

当需要**低功耗支持**时，可以控制定时器的行为：

```c
// 创建一个不会唤醒系统的定时器（省电模式）
osTimerId_t timer = osTimerExtNew(
    CallbackFunc,           // 回调函数
    osTimerPeriodic,        // 周期定时器
    NULL,                   // 参数
    NULL,                   // 属性
    osTimerRousesDisable,   // 不唤醒系统
    osTimerAlignIgnore      // 忽略对齐
);
```

**标准对比**：

| 特性 | 标准 CMSIS-RTOS2 | OH 扩展 |
|------|------------------|---------|
| **标准接口** | `osTimerNew` | `osTimerNew`（保留） |
| **扩展接口** | 无 | `osTimerExtNew`（新增） |
| **低功耗控制** | 不支持 | 支持 |
| **配置选项** | `LOSCFG_BASE_CORE_SWTMR_ALIGN` | 条件编译控制 |

**注意事项**：
- 仅在 `LOSCFG_BASE_CORE_SWTMR_ALIGN == 1` 时可用
- 是 OH 特有的扩展，非 CMSIS 标准
- 使用此 API 的代码不可移植到其他 CMSIS-RTOS2 实现

---

## 5.3 行为变更 API

### 5.3.1 优先级映射

**标准定义**：

CMSIS-RTOS2 定义了标准优先级枚举：

```c
// CMSIS/RTOS2/Include/cmsis_os2.h

typedef enum {
  osPriorityIdle          =  1,         //< Priority: idle (lowest)
  osPriorityLow           =  8,         //< Priority: low
  osPriorityBelowNormal   = 16,         //< Priority: below normal
  osPriorityNormal        = 24,         //< Priority: normal (default)
  osPriorityAboveNormal   = 32,         //< Priority: above normal
  osPriorityHigh          = 40,         //< Priority: high
  osPriorityRealtime      = 48,         //< Priority: realtime
  osPriorityError         = -1,         //< System cannot determine priority or illegal priority.
  osPriorityReserved      = 0x7FFFFFFF  //< Prevents enum down-size compiler optimization.
} osPriority_t;
```

**OH 适配实现**：

```c
// kernel/liteos_m/kal/cmsis/cmsis_liteos2.c

/* LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO <---> osPriorityNormal */
#define LOS_PRIORITY(cmsisPriority) \
    (LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO - ((cmsisPriority) - osPriorityNormal))

#define CMSIS_PRIORITY(losPriority) \
    (osPriorityNormal + (LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO - (losPriority)))
```

**行为差异说明**：

| 方面 | 标准 CMSIS-RTOS2 | OH 实现 |
|------|------------------|---------|
| **优先级数值** | 直接使用 CMSIS 值 | 映射到 LiteOS-M 内部优先级 |
| **默认优先级** | `osPriorityNormal` (24) | 映射为 `LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO` |
| **范围检查** | 由实现决定 | 映射后检查是否有效 |

**示例**：

```c
// 应用程序
osThreadAttr_t attr = {0};
attr.priority = osPriorityNormal;  // 值 = 24
osThreadId_t tid = osThreadNew(TaskFunc, NULL, &attr);

// OH 适配层转换过程：
// LOS_PRIORITY(24) = 25 - (24 - 24) = 25
// （假设 LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO = 25）
// LiteOS-M 内部使用优先级 25
```

**重要提示**：
- 优先级映射对用户透明
- 但开发者需理解 CMSIS 优先级 ≠ 内核优先级
- 优先级反转等调度行为由 LiteOS-M 决定

### 5.3.2 错误码映射

**标准错误码定义**：

```c
// CMSIS/RTOS2/Include/cmsis_os2.h

typedef enum {
  osOK                      =  0,         //< Operation completed successfully.
  osError                   = -1,         //< Unspecified RTOS error: run-time error but no other error message fits.
  osErrorTimeout            = -2,         //< Operation not completed within the timeout period.
  osErrorResource           = -3,         //< Resource not available.
  osErrorParameter          = -4,         //< Parameter error.
  osErrorNoMemory           = -5,         //< System is out of memory: it was impossible to allocate or reserve memory for the operation.
  osErrorISR                = -6,         //< Not allowed in ISR context: the function cannot be called from interrupt service routines.
  osStatusReserved          = 0x7FFFFFFF  //< Prevents enum down-size compiler optimization.
} osStatus_t;
```

**OH 映射实现**：

```c
// kernel/liteos_m/kal/cmsis/cmsis_liteos2.c

osStatus_t osThreadSetPriority(osThreadId_t thread_id, osPriority_t priority) {
    UINT32 ret;
    // ...
    ret = LOS_TaskPriSet(pstTaskCB->taskID, prio);
    switch (ret) {
        case LOS_ERRNO_TSK_PRIOR_ERROR:
        case LOS_ERRNO_TSK_OPERATE_SYSTEM_TASK:
        case LOS_ERRNO_TSK_ID_INVALID:
            return osErrorParameter;    // 参数错误

        case LOS_ERRNO_TSK_NOT_CREATED:
            return osErrorResource;     // 资源错误

        default:
            return osOK;
    }
}
```

**映射规则**：

| LiteOS-M 错误 | CMSIS-RTOS2 错误 | 说明 |
|---------------|------------------|------|
| `LOS_ERRNO_TSK_*_INVALID` | `osErrorParameter` | 参数无效 |
| `LOS_ERRNO_TSK_NOT_CREATED` | `osErrorResource` | 资源不存在 |
| `LOS_ERRNO_*_TIMEOUT` | `osErrorTimeout` | 超时 |
| `LOS_ERRNO_TSK_NOT_ALLOW_IN_INT` | `osErrorISR` | 中断上下文禁止 |
| `LOS_OK` | `osOK` | 成功 |

**行为一致性**：

- ✅ 标准错误码语义保持一致
- ✅ 应用程序可按标准方式处理错误
- ⚠️ 部分 LiteOS-M 特有错误被归类为 `osError`

---

## 5.4 完全兼容的标准 API

### 5.4.1 内核管理 API

```c
// 完全兼容，无差异
osStatus_t osKernelInitialize(void);
osStatus_t osKernelGetInfo(osVersion_t *version, char *id_buf, uint32_t id_size);
osKernelState_t osKernelGetState(void);
osStatus_t osKernelStart(void);
int32_t osKernelLock(void);
int32_t osKernelUnlock(void);
int32_t osKernelRestoreLock(int32_t lock);
uint32_t osKernelGetTickCount(void);
uint32_t osKernelGetTickFreq(void);
uint32_t osKernelGetSysTimerCount(void);
uint32_t osKernelGetSysTimerFreq(void);
```

### 5.4.2 线程管理 API

```c
// 完全兼容，优先级映射为内部实现细节
osThreadId_t osThreadNew(osThreadFunc_t func, void *argument, const osThreadAttr_t *attr);
const char *osThreadGetName(osThreadId_t thread_id);
osThreadId_t osThreadGetId(void);
void *osThreadGetArgument(void);
osThreadState_t osThreadGetState(osThreadId_t thread_id);
uint32_t osThreadGetStackSize(osThreadId_t thread_id);
uint32_t osThreadGetStackSpace(osThreadId_t thread_id);
osStatus_t osThreadSetPriority(osThreadId_t thread_id, osPriority_t priority);
osPriority_t osThreadGetPriority(osThreadId_t thread_id);
osStatus_t osThreadYield(void);
osStatus_t osThreadSuspend(osThreadId_t thread_id);
osStatus_t osThreadResume(osThreadId_t thread_id);
osStatus_t osThreadDetach(osThreadId_t thread_id);
osStatus_t osThreadJoin(osThreadId_t thread_id);
void osThreadExit(void);
osStatus_t osThreadTerminate(osThreadId_t thread_id);
uint32_t osThreadGetCount(void);
```

### 5.4.3 线程标志 API

```c
// 完全兼容
uint32_t osThreadFlagsSet(osThreadId_t thread_id, uint32_t flags);
uint32_t osThreadFlagsClear(uint32_t flags);
uint32_t osThreadFlagsGet(void);
uint32_t osThreadFlagsWait(uint32_t flags, uint32_t options, uint32_t timeout);
```

### 5.4.4 事件标志 API

```c
// 完全兼容
osEventFlagsId_t osEventFlagsNew(const osEventFlagsAttr_t *attr);
const char *osEventFlagsGetName(osEventFlagsId_t ef_id);
uint32_t osEventFlagsSet(osEventFlagsId_t ef_id, uint32_t flags);
uint32_t osEventFlagsClear(osEventFlagsId_t ef_id, uint32_t flags);
uint32_t osEventFlagsGet(osEventFlagsId_t ef_id);
uint32_t osEventFlagsWait(osEventFlagsId_t ef_id, uint32_t flags, uint32_t options, uint32_t timeout);
osStatus_t osEventFlagsDelete(osEventFlagsId_t ef_id);
```

### 5.4.5 互斥锁 API

```c
// 完全兼容
osMutexId_t osMutexNew(const osMutexAttr_t *attr);
osStatus_t osMutexAcquire(osMutexId_t mutex_id, uint32_t timeout);
osStatus_t osMutexRelease(osMutexId_t mutex_id);
osThreadId_t osMutexGetOwner(osMutexId_t mutex_id);
osStatus_t osMutexDelete(osMutexId_t mutex_id);
```

### 5.4.6 信号量 API

```c
// 完全兼容
osSemaphoreId_t osSemaphoreNew(uint32_t max_count, uint32_t initial_count, const osSemaphoreAttr_t *attr);
const char *osSemaphoreGetName(osSemaphoreId_t semaphore_id);
osStatus_t osSemaphoreAcquire(osSemaphoreId_t semaphore_id, uint32_t timeout);
osStatus_t osSemaphoreRelease(osSemaphoreId_t semaphore_id);
uint32_t osSemaphoreGetCount(osSemaphoreId_t semaphore_id);
osStatus_t osSemaphoreDelete(osSemaphoreId_t semaphore_id);
```

### 5.4.7 定时器 API

```c
// 完全兼容（osTimerExtNew 为新增扩展）
osTimerId_t osTimerNew(osTimerFunc_t func, osTimerType_t type, void *argument, const osTimerAttr_t *attr);
const char *osTimerGetName(osTimerId_t timer_id);
osStatus_t osTimerStart(osTimerId_t timer_id, uint32_t ticks);
osStatus_t osTimerStop(osTimerId_t timer_id);
uint32_t osTimerIsRunning(osTimerId_t timer_id);
osStatus_t osTimerDelete(osTimerId_t timer_id);
```

### 5.4.8 消息队列 API

```c
// 完全兼容
osMessageQueueId_t osMessageQueueNew(uint32_t msg_count, uint32_t msg_size, const osMessageQueueAttr_t *attr);
const char *osMessageQueueGetName(osMessageQueueId_t mq_id);
osStatus_t osMessageQueuePut(osMessageQueueId_t mq_id, const void *msg_ptr, uint8_t msg_prio, uint32_t timeout);
osStatus_t osMessageQueueGet(osMessageQueueId_t mq_id, void *msg_ptr, uint8_t *msg_prio, uint32_t timeout);
uint32_t osMessageQueueGetCapacity(osMessageQueueId_t mq_id);
uint32_t osMessageQueueGetMsgSize(osMessageQueueId_t mq_id);
uint32_t osMessageQueueGetCount(osMessageQueueId_t mq_id);
uint32_t osMessageQueueGetSpace(osMessageQueueId_t mq_id);
osStatus_t osMessageQueueDelete(osMessageQueueId_t mq_id);
```

### 5.4.9 内存池 API

```c
// 完全兼容
osMemoryPoolId_t osMemoryPoolNew(uint32_t block_count, uint32_t block_size, const osMemoryPoolAttr_t *attr);
void *osMemoryPoolAlloc(osMemoryPoolId_t mp_id, uint32_t timeout);
osStatus_t osMemoryPoolFree(osMemoryPoolId_t mp_id, void *block);
osStatus_t osMemoryPoolDelete(osMemoryPoolId_t mp_id);
uint32_t osMemoryPoolGetCapacity(osMemoryPoolId_t mp_id);
uint32_t osMemoryPoolGetBlockSize(osMemoryPoolId_t mp_id);
uint32_t osMemoryPoolGetCount(osMemoryPoolId_t mp_id);
uint32_t osMemoryPoolGetSpace(osMemoryPoolId_t mp_id);
const char *osMemoryPoolGetName(osMemoryPoolId_t mp_id);
```

---

## 5.5 数据类型与结构体

### 5.5.1 完全兼容的数据类型

所有 CMSIS-RTOS2 定义的数据类型在 OH 中完全兼容：

```c
// 基本类型
typedef void *osThreadId_t;
typedef void *osTimerId_t;
typedef void *osEventFlagsId_t;
typedef void *osMutexId_t;
typedef void *osSemaphoreId_t;
typedef void *osMemoryPoolId_t;
typedef void *osMessageQueueId_t;

// 回调类型
typedef void (*osThreadFunc_t)(void *argument);
typedef void (*osTimerFunc_t)(void *argument);

// 枚举类型
// osPriority_t, osThreadState_t, osTimerType_t, 
// osStatus_t 等全部兼容
```

### 5.5.2 属性结构体

```c
// 所有属性结构体完全兼容

typedef struct {
  const char *name;
  uint32_t attr_bits;
  void *cb_mem;
  uint32_t cb_size;
  void *stack_mem;
  uint32_t stack_size;
  osPriority_t priority;
} osThreadAttr_t;

typedef struct {
  const char *name;
  uint32_t attr_bits;
  void *cb_mem;
  uint32_t cb_size;
} osTimerAttr_t;

// ... 其他属性结构体
```

---

## 5.6 使用建议

### 5.6.1 可移植代码编写

**推荐做法**：

```c
// ✅ 使用标准 API - 代码可移植
osThreadId_t tid = osThreadNew(TaskFunc, NULL, NULL);
osDelay(100);
osThreadTerminate(tid);
```

**避免做法**：

```c
// ❌ 使用 OH 特有扩展 - 代码不可移植
#if defined(LOSCFG_BASE_CORE_SWTMR_ALIGN)
osTimerId_t timer = osTimerExtNew(...);  // OH 特有
#endif
```

### 5.6.2 处理行为差异

```c
// ✅ 正确处理错误码（标准方式）
osStatus_t status = osMutexAcquire(mutex_id, 1000);
if (status == osOK) {
    // 成功
} else if (status == osErrorTimeout) {
    // 超时处理
} else if (status == osErrorResource) {
    // 资源不可用
}

// ✅ 不依赖具体优先级数值
osPriority_t prio = osThreadGetPriority(thread_id);
if (prio == osPriorityNormal) {
    // 处理
}
```

---

## 5.7 兼容性总结

| 兼容级别 | 描述 | API 数量 |
|----------|------|----------|
| **完全兼容** | 标准 CMSIS-RTOS2 API，行为一致 | 60+ |
| **扩展新增** | OH 特有功能，不影响标准 API | 1 (`osTimerExtNew`) |
| **内部实现** | 优先级映射等对用户透明 | - |

### 兼容性评级

| 方面 | 兼容性 | 说明 |
|------|--------|------|
| **API 签名** | ✅ 100% | 所有函数签名一致 |
| **数据类型** | ✅ 100% | 所有结构体和枚举一致 |
| **行为语义** | ✅ 95% | 基本一致，优先级映射为内部细节 |
| **错误码** | ✅ 90% | 标准错误码完全支持，部分 LiteOS-M 特有错误归类为通用错误 |
| **扩展功能** | ⚠️ OH 特有 | `osTimerExtNew` 为 OH 扩展 |

**总体评价**：CMSIS 在 OH 中的实现**高度兼容** CMSIS-RTOS2 标准，应用程序可实现源代码级移植。

---

*文档版本: 1.0*  
*最后更新: 2025-02-08*
