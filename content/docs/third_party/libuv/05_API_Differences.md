# 05 - API/接口差异

## 5.1 概述

OpenHarmony 对 libuv 的扩展主要体现在**新增功能**和**行为增强**，保持了与上游 API 的兼容性。

### 差异分类

| 类别 | 说明 | 影响 |
|-----|------|------|
| **新增 API** | OH 特有的接口扩展 | 仅在 OH 环境可用 |
| **行为变更** | 原有 API 的行为增强 | 需条件编译控制 |
| **宏定义** | 编译期配置 | 影响功能启用 |

---

## 5.2 新增 API

### 5.2.1 异步堆栈跟踪 API

**头文件**: `src/dfx/async_stack/libuv_async_stack.h`

```c
// 异步类型定义
#define ASYNC_TYPE_LIBUV_TIMER (1ULL << 0)   // 定时器事件
#define ASYNC_TYPE_LIBUV_QUEUE (1ULL << 1)   // 队列操作
#define ASYNC_TYPE_LIBUV_SEND  (1ULL << 2)   // 发送操作

// 回调函数类型
typedef void(*UvSetStackIdFunc)(uint64_t stackId);
typedef uint64_t(*UvCollectAsyncStackFunc)(uint64_t type);

// API 函数
UV_EXTERN void LibuvSetAsyncStackFunc(
    UvCollectAsyncStackFunc collectAsyncStackFunc, 
    UvSetStackIdFunc setStackIdFunc
);

uint64_t LibuvCollectAsyncStack(uint64_t type);
void LibuvSetStackId(uint64_t stackId);
```

**使用示例**:
```c
#include "dfx/async_stack/libuv_async_stack.h"

// 1. 注册回调（由 DFX 框架提供）
LibuvSetAsyncStackFunc(myCollectFunc, mySetStackIdFunc);

// 2. 异步任务提交时收集堆栈
uint64_t stackId = LibuvCollectAsyncStack(ASYNC_TYPE_LIBUV_TIMER);

// 3. 异步任务执行时设置堆栈 ID
LibuvSetStackId(stackId);
```

**OH 价值**:
- 解决异步调用堆栈断裂问题
- 支持崩溃时的完整调用链分析
- 与 OHOS DFX 框架集成

---

### 5.2.2 FFRT 扩展 API

#### QoS 分级枚举

**头文件**: `include/uv.h`

```c
typedef enum {
  uv_qos_background = 0,        // 后台任务 - 最低优先级
  uv_qos_utility = 1,           // 实用工具
  uv_qos_default = 2,           // 默认优先级
  uv_qos_user_initiated = 3,    // 用户发起
  uv_qos_reserved = 4,          // 保留 - 不使用
  uv_qos_user_interactive = 5,  // 用户交互 - 最高优先级
} uv_qos_t;
```

#### 带 QoS 的工作队列 API

```c
// 原始 API (无 QoS)
int uv_queue_work(uv_loop_t* loop,
                  uv_work_t* req,
                  uv_work_cb work_cb,
                  uv_after_work_cb after_work_cb);

// OH 扩展 API (带 QoS)
#ifdef USE_FFRT
int uv_qos_queue_work(uv_loop_t* loop,
                      uv_work_t* req,
                      uv_qos_work_cb work_cb,
                      uv_after_work_cb after_work_cb,
                      uv_qos_t qos);
#endif
```

#### Loop Magic 校验 API

```c
// 内部使用，用于检测已关闭的 loop
#define UV_LOOP_MAGIC 0x100B100B

// 校验函数
int is_uv_loop_good_magic(const uv_loop_t* loop);
```

**使用示例**:
```c
#ifdef USE_FFRT
// 提交后台任务
uv_qos_queue_work(loop, &req, work_cb, after_cb, uv_qos_background);

// 提交用户交互任务（高优先级）
uv_qos_queue_work(loop, &req, work_cb, after_cb, uv_qos_user_interactive);
#endif
```

---

### 5.2.3 性能跟踪 API

**头文件**: `src/uv_trace.h`

```c
void uv_start_trace(uint64_t tag, const char* name);
void uv_end_trace(uint64_t tag);
```

**实现**: `src/unix/ohos/trace_ohos.c`

```c
#include "hitrace_meter_c.h"

void uv_start_trace(uint64_t tag, const char* name) {
  HiTraceStartTrace(tag, name);
}

void uv_end_trace(uint64_t tag) {
  HiTraceFinishTrace(tag);
}
```

**使用示例**:
```c
#ifdef USE_OHOS_DFX
uv_start_trace(HITRACE_TAG_APP, "UV_WORK_START");
// 执行任务...
uv_end_trace(HITRACE_TAG_APP);
#endif
```

---

### 5.2.4 日志 API

**头文件**: `src/uv_log.h`

```c
// 统一日志宏
#define UV_LOGI(fmt, ...)  // Info
#define UV_LOGD(fmt, ...)  // Debug
#define UV_LOGW(fmt, ...)  // Warn
#define UV_LOGE(fmt, ...)  // Error
#define UV_LOGF(fmt, ...)  // Fatal
```

**OHOS 环境实现**:
```c
#ifdef USE_OHOS_DFX
  #include "hilog/log.h"
  #define UV_LOGI(fmt, ...) HILOG_IMPL(LOG_CORE, LOG_INFO, 
                                      0xD003301, "LIBUV", fmt, ##__VA_ARGS__)
  // ... D, W, E, F
#else
  // 模拟器环境使用自定义实现
  int uv__log_impl(enum uv__log_level level, const char* fmt, ...);
#endif
```

**使用示例**:
```c
UV_LOGI("Loop init: %{public}zu", (size_t)loop % UV_ADDR_MOD);
UV_LOGE("Invalid loop magic: %{public}#x", loop->magic);
```

---

## 5.3 行为变更

### 5.3.1 线程池任务结构

**原始定义**:
```c
struct uv__work {
  void (*work)(struct uv__work *w);  // 无 QoS 参数
  void (*done)(struct uv__work *w, int status);
  struct uv_loop_s* loop;
  struct uv__queue wq;
};
```

**OH 版本** (USE_FFRT):
```c
struct uv__work {
  void (*work)(struct uv__work *w, int qos);  // 新增 qos 参数
  void (*done)(struct uv__work *w, int status);
  struct uv_loop_s* loop;
  struct uv__queue wq;
};
```

**影响**: 使用 USE_FFRT 时，work 回调签名变更

### 5.3.2 Loop 初始化

**新增行为** (USE_FFRT):
```c
int uv_loop_init(uv_loop_t* loop) {
  // ... 原始初始化代码
  
#ifdef USE_FFRT
  // 为每个 QoS 级别创建独立队列
  uv__queue_init(&(lfields->wq_sub[uv_qos_background]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_utility]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_default]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_user_initiated]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_reserved]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_user_interactive]));
  
  // 设置 Magic
  loop->magic = UV_LOOP_MAGIC;
#endif
  
  return 0;
}
```

### 5.3.3 文件描述符关闭

**原始行为**:
```c
uv__close(loop->backend_fd);
```

**OH 行为** (USE_OHOS_DFX):
```c
#ifdef USE_OHOS_DFX
  fdsan_close_with_tag(loop->backend_fd, 
                       uv__get_addr_tag((void *)&loop->backend_fd));
#else
  uv__close(loop->backend_fd);
#endif
```

**价值**: 使用 fdsan 进行文件描述符安全检查

### 5.3.4 错误处理

**新增行为** (USE_OHOS_DFX):
```c
#ifdef USE_OHOS_DFX
#include "info/fatal_message.h"
#define UV_ERRNO_ABORT(...)                                                   \
  do {                                                                        \
    char errno_message[1024];                                                 \
    snprintf(errno_message, sizeof(errno_message), __VA_ARGS__);              \
    set_fatal_message(errno_message);                                         \
    abort();                                                                  \
  } while(0)
#endif
```

**价值**: 崩溃时记录错误信息到 DFX 框架

---

## 5.4 条件编译宏

### 5.4.1 功能开关

| 宏 | 定义位置 | 默认值 | 功能 |
|---|---------|-------|------|
| `USE_FFRT` | libuv.gni | false | 启用 FFRT 集成 |
| `USE_OHOS_DFX` | libuv.gni | true | 启用 DFX 框架 |
| `ASYNC_STACKTRACE` | libuv.gni | true | 启用异步堆栈 |
| `SUPPORT_INTERRUPT` | libuv.gni | true | 启用中断支持 |
| `ENABLE_WORKER_PRIORITY` | libuv.gni | true | 启用 Worker 优先级 |

### 5.4.2 内部宏

| 宏 | 定义位置 | 值 | 用途 |
|---|---------|---|------|
| `UV_LOOP_MAGIC` | src/unix/internal.h | 0x100B100B | Loop 魔数校验 |
| `UV_ADDR_MOD` | src/uv_log.h | 10000 | 地址取模 |
| `ASYNC_TYPE_LIBUV_*` | libuv_async_stack.h | 位掩码 | 异步类型标记 |

---

## 5.5 兼容性说明

### 5.5.1 与上游 API 兼容性

| 方面 | 状态 | 说明 |
|-----|------|------|
| 标准 API | ✅ 完全兼容 | 所有 uv_* 函数 |
| 结构体布局 | ⚠️ 条件变更 | USE_FFRT 时 uv__work 变更 |
| 枚举值 | ✅ 新增 | uv_qos_t 为新增 |
| 回调签名 | ⚠️ 条件变更 | USE_FFRT 时 work 回调变更 |

### 5.5.2 跨平台兼容性

```c
// 编写可移植代码
#if defined(USE_FFRT)
  // 使用 FFRT 扩展
  uv_qos_queue_work(loop, &req, work_cb, after_cb, uv_qos_default);
#elif defined(USE_OHOS_DFX)
  // 使用 DFX 扩展
  uv_start_trace(tag, name);
#endif

// 标准 libuv API 始终可用
uv_queue_work(loop, &req, work_cb, after_cb);
```

---

## 5.6 使用建议

### 5.6.1 新代码编写

**推荐做法**:
```c
// 检查宏定义后再使用 OH 特有 API
#ifdef USE_FFRT
  // 使用 QoS 功能
  uv_qos_queue_work(loop, &req, work_cb, after_cb, uv_qos_user_initiated);
#else
  // 回退到标准 API
  uv_queue_work(loop, &req, work_cb, after_work_cb);
#endif
```

### 5.6.2 库开发者注意事项

1. **ABI 兼容性**: 注意 USE_FFRT 时的结构体变更
2. **条件编译**: 使用 #ifdef 隔离 OH 特有代码
3. **测试覆盖**: 测试启用/禁用各宏时的行为

### 5.6.3 应用开发者建议

- 优先使用标准 libuv API
- 仅在需要 OH 特有功能时使用扩展 API
- 注意扩展 API 的可移植性限制

---

## 5.7 API 变更汇总表

| API/宏 | 类型 | OH 特有 | 说明 |
|-------|------|--------|------|
| `LibuvSetAsyncStackFunc` | 新增 | ✅ | 异步堆栈回调注册 |
| `LibuvCollectAsyncStack` | 新增 | ✅ | 收集异步堆栈 |
| `LibuvSetStackId` | 新增 | ✅ | 设置堆栈 ID |
| `uv_qos_queue_work` | 新增 | ✅ | 带 QoS 的任务提交 |
| `uv_qos_t` | 新增 | ✅ | QoS 分级枚举 |
| `uv_start_trace` | 新增 | ✅ | 开始性能跟踪 |
| `uv_end_trace` | 新增 | ✅ | 结束性能跟踪 |
| `UV_LOGI/D/W/E/F` | 新增 | ✅ | 统一日志宏 |
| `uv__work.work` | 变更 | ✅ | USE_FFRT 时签名变更 |
| `UV_LOOP_MAGIC` | 新增 | ✅ | Loop 魔数 |

---

*注: 所有 OH 特有 API 都在条件编译保护下，不影响标准 libuv 的兼容性*
