# 02 - Patch 详细分析

## 概述

⚠️ **重要说明**: libuv 在 OpenHarmony 中**没有使用传统的 .patch 文件**进行适配。所有 OH 定制化内容通过**条件编译宏**和**新增源文件**的方式实现。

---

## 2.1 "无 Patch" 设计

### 为什么选择无 Patch？

| 优势 | 说明 |
|-----|------|
| **降低维护成本** | 升级上游版本时无需重新应用 Patch |
| **清晰的功能边界** | OH 特有功能通过宏隔离，代码结构清晰 |
| **可配置性** | 通过 GN 参数动态启用/禁用功能 |
| **易于测试** | 可以分别测试原生功能和 OH 扩展 |

### 适配方式对比

| 方式 | 传统 Patch | OH 宏隔离 |
|-----|-----------|----------|
| 实现形式 | .patch 文件 | #ifdef 宏 |
| 代码位置 | 分散在各文件 | 集中在 src/unix/ohos/ |
| 升级难度 | 需处理冲突 | 直接替换文件 |
| 灵活性 | 固定 | 可配置 |

---

## 2.2 条件编译宏清单

### 宏定义总览

| 宏名称 | 定义位置 | 默认值 | 功能 |
|-------|---------|-------|------|
| `USE_FFRT` | libuv.gni | false | FFRT 任务调度集成 |
| `USE_OHOS_DFX` | libuv.gni | true | OHOS 诊断框架集成 |
| `ASYNC_STACKTRACE` | libuv.gni | true | 异步堆栈跟踪 |
| `SUPPORT_INTERRUPT` | libuv.gni | true | 中断支持 (emulator 除外) |
| `ENABLE_WORKER_PRIORITY` | libuv.gni | true | Worker 线程优先级 |
| `enable_uv_statisic` | libuv.gni | false | UV 统计功能 |

### libuv.gni 配置

```gn
declare_args() {
  libuv_use_ffrt = false
  enable_async_stack = true
  enable_uv_statisic = false
  use_ohos_dfx = true
  enable_worker_prio = true
}
```

---

## 2.3 OH 特有源文件分析

### 2.3.1 src/unix/ohos/trace_ohos.c

**功能**: HiTrace 性能跟踪集成

**代码分析**:

```c
#include "uv_trace.h"
#include "hitrace_meter_c.h"

void uv_start_trace(uint64_t tag, const char* name) {
  HiTraceStartTrace(tag, name);  // 调用 OHOS HiTrace API
}

void uv_end_trace(uint64_t tag) {
  HiTraceFinishTrace(tag);       // 结束跟踪
}
```

**OH 价值**:
- 提供细粒度的性能跟踪能力
- 与 OHOS 性能分析工具链集成
- 支持系统级性能调优

**使用场景**:
```c
#ifdef USE_OHOS_DFX
uv_start_trace(HITRACE_TAG_APP, "UV_WORK_START");
// 执行任务
uv_end_trace(HITRACE_TAG_APP);
#endif
```

---

### 2.3.2 src/unix/ohos/log_ohos.c

**功能**: 模拟器环境 HiLog 日志适配

**代码分析**:

```c
#include "uv_log.h"
#include "hilog/log.h"

LogLevel convert_uv_log_level(enum uv__log_level level) {
  switch (level) {
    case UV_DEBUG: return LOG_DEBUG;
    case UV_INFO:  return LOG_INFO;
    case UV_WARN:  return LOG_WARN;
    case UV_ERROR: return LOG_ERROR;
    case UV_FATAL: return LOG_FATAL;
    default:       return LOG_LEVEL_MIN;
  }
}

int uv__log_impl(enum uv__log_level level, const char *fmt, ...) {
  va_list args;
  va_start(args, fmt);
  int ret = HiLogPrintArgs(LOG_CORE, convert_uv_log_level(level), 
                           0xD003301, "LIBUV", fmt, args);
  va_end(args);
  return ret;
}
```

**OH 价值**:
- 统一的日志格式 (domain 0xD003301)
- 支持日志级别过滤
- 与 OHOS 日志系统无缝集成

**条件编译**:
- 仅在 `is_emulator` 环境下编译
- 真实设备使用 src/uv_log.h 中的宏定义

---

### 2.3.3 src/dfx/async_stack/libuv_async_stack.c

**功能**: 异步堆栈跟踪支持

**代码分析**:

```c
static UvCollectAsyncStackFunc g_collectAsyncStackFunc = NULL;
static UvSetStackIdFunc g_setStackIdFunc = NULL;

void LibuvSetAsyncStackFunc(UvCollectAsyncStackFunc collectFunc, 
                            UvSetStackIdFunc setStackIdFunc) {
    g_collectAsyncStackFunc = collectFunc;
    g_setStackIdFunc = setStackIdFunc;
}

uint64_t LibuvCollectAsyncStack(uint64_t type) {
    if (g_collectAsyncStackFunc != NULL) {
        return g_collectAsyncStackFunc(type);
    }
    return 0;
}

void LibuvSetStackId(uint64_t stackId) {
    if (g_setStackIdFunc != NULL) {
        return g_setStackIdFunc(stackId);
    }
}
```

**异步类型定义** (libuv_async_stack.h):
```c
#define ASYNC_TYPE_LIBUV_TIMER (1ULL << 0)   // 定时器事件
#define ASYNC_TYPE_LIBUV_QUEUE (1ULL << 1)   // 队列操作
#define ASYNC_TYPE_LIBUV_SEND  (1ULL << 2)   // 发送操作
```

**OH 价值**:
- 解决异步编程中的堆栈断裂问题
- 支持崩溃时的调用链分析
- 与 OHOS DFX 框架集成

---

## 2.4 USE_FFRT 深度分析

### 2.4.1 功能概述

FFRT (Fast Function Runtime) 是 OpenHarmony 的高性能任务调度框架。libuv 通过 `USE_FFRT` 宏集成 FFRT，实现更细粒度的任务 QoS 管理。

### 2.4.2 核心变更点

#### 1. 线程池任务结构

**原始 libuv** (src/uv/threadpool.h):
```c
struct uv__work {
  void (*work)(struct uv__work *w);   // 无 QoS 参数
  void (*done)(struct uv__work *w, int status);
  struct uv_loop_s* loop;
  struct uv__queue wq;
};
```

**OH 版本** (USE_FFRT):
```c
struct uv__work {
  void (*work)(struct uv__work *w, int qos);  // 新增 QoS 参数
  void (*done)(struct uv__work *w, int status);
  struct uv_loop_s* loop;
  struct uv__queue wq;
};
```

#### 2. QoS 分级定义

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

#### 3. Loop Magic 校验

```c
#define UV_LOOP_MAGIC 0x100B100B  // 魔数定义

// Loop 初始化时设置
loop->magic = UV_LOOP_MAGIC;

// Loop 关闭时清除
loop->magic = ~UV_LOOP_MAGIC;

// 使用时校验
int is_uv_loop_good_magic(const uv_loop_t* loop) {
  if (loop->magic == UV_LOOP_MAGIC) {
    return 1;
  }
  UV_LOGE("loop invalid: %{public}#x", loop->magic);
  return 0;
}
```

**设计目的**:
- 检测已关闭的 loop 被误用
- 防止 Use-After-Free 问题
- 增强调试信息

#### 4. 多队列支持

```c
#ifdef USE_FFRT
  // 为每个 QoS 级别创建独立队列
  uv__queue_init(&(lfields->wq_sub[uv_qos_background]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_utility]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_default]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_user_initiated]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_reserved]));
  uv__queue_init(&(lfields->wq_sub[uv_qos_user_interactive]));
#endif
```

### 2.4.3 FFRT 兼容性断言

```c
// 确保 libuv QoS 与 FFRT QoS 值一致
STATIC_ASSERT(uv_qos_background == ffrt_qos_background);
STATIC_ASSERT(uv_qos_utility == ffrt_qos_utility);
STATIC_ASSERT(uv_qos_default == ffrt_qos_default);
STATIC_ASSERT(uv_qos_user_initiated == ffrt_qos_user_initiated);
STATIC_ASSERT(uv_qos_user_interactive == ffrt_qos_user_interactive);
```

### 2.4.4 使用场景

| 场景 | QoS 级别 | 说明 |
|-----|---------|------|
| 后台同步 | uv_qos_background | 数据同步、日志写入 |
| 网络请求 | uv_qos_utility | HTTP 请求、下载 |
| 定时任务 | uv_qos_default | 默认定时器任务 |
| 用户点击 | uv_qos_user_initiated | 用户触发的操作 |
| UI 渲染 | uv_qos_user_interactive | 动画、列表滚动 |

---

## 2.5 USE_OHOS_DFX 深度分析

### 2.5.1 功能概述

DFX (Design For X) 是 OHOS 的诊断框架，包含日志、跟踪、性能分析等能力。

### 2.5.2 核心组件

#### 1. 统一日志宏 (src/uv_log.h)

```c
#ifdef USE_OHOS_DFX
  #include "hilog/log.h"
  #define UV_LOGI(fmt, ...) HILOG_IMPL(LOG_CORE, LOG_INFO, 
                                      0xD003301, "LIBUV", fmt, ##__VA_ARGS__)
  #define UV_LOGD(fmt, ...) HILOG_IMPL(LOG_CORE, LOG_DEBUG, 
                                      0xD003301, "LIBUV", fmt, ##__VA_ARGS__)
  // ... E, F 等
#else
  // 使用自定义实现
  #define UV_LOGI(...) LOGI_IMPL(UV_INFO, ##__VA_ARGS__)
#endif
```

#### 2. 线程 ID 追踪

```c
#ifdef USE_OHOS_DFX
void uv__init_thread_id(uv_loop_t* loop) {
    // 初始化线程 ID 用于诊断
}
#endif
```

#### 3. fdsan 集成

```c
#ifdef USE_OHOS_DFX
  // 使用 fdsan 安全关闭 FD
  fdsan_close_with_tag(loop->backend_fd, 
                       uv__get_addr_tag((void *)&loop->backend_fd));
#else
  uv__close(loop->backend_fd);
#endif
```

#### 4. 致命错误处理 (include/uv.h)

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

### 2.5.3 诊断事件

```c
#ifdef USE_OHOS_DFX
#define MIN_REQS_THRESHOLD 100   // 最小请求阈值
#define MAX_REQS_THRESHOLD 300   // 最大请求阈值
#define CURSOR 5                 // 游标位置
#endif
```

---

## 2.6 ASYNC_STACKTRACE 分析

### 2.6.1 功能说明

异步堆栈跟踪解决异步编程中的"堆栈断裂"问题。当异步任务执行时，原始的调用堆栈已经丢失，这使得问题定位变得困难。

### 2.6.2 实现原理

```c
// 1. 设置堆栈收集回调
LibuvSetAsyncStackFunc(collectFunc, setStackIdFunc);

// 2. 异步任务提交时收集堆栈
uint64_t stackId = LibuvCollectAsyncStack(ASYNC_TYPE_LIBUV_TIMER);

// 3. 异步任务执行时设置堆栈 ID
LibuvSetStackId(stackId);

// 4. 崩溃时可以通过 stackId 还原调用链
```

### 2.6.3 使用位置

| 文件 | 位置 | 类型 |
|-----|------|-----|
| src/timer.c | uv_timer_start | 定时器 |
| src/threadpool.c | 任务提交 | 线程池 |
| src/unix/async.c | uv_async_send | 异步句柄 |

---

## 2.7 SUPPORT_INTERRUPT 分析

### 2.7.1 功能说明

支持事件循环的中断检查，用于响应更高优先级的事件。

### 2.7.2 实现细节

```c
#ifdef SUPPORT_INTERRUPT
  lfields->uv_params = 0;
  lfields->uv_interrupt_task_type = -1;
  lfields->last_check_stamp = INT64_MAX;
  lfields->check_pending_higher_event = NULL;
#endif
```

### 2.7.3 使用场景

- 用户交互事件需要中断后台任务
- 系统需要响应紧急事件
- 实现协作式多任务

---

## 2.8 代码变更统计

### 2.8.1 文件级变更

| 类别 | 文件数 | 说明 |
|-----|-------|------|
| 新增文件 | 4 | ohos/*.c, dfx/async_stack/* |
| 修改文件 | 16+ | 添加条件编译 |
| 总行变更 | ~500+ | 新增代码 + 条件编译 |

### 2.8.2 宏使用统计

| 宏 | 使用次数 | 主要文件 |
|---|---------|---------|
| USE_FFRT | 45+ | threadpool.c, unix/*.c |
| USE_OHOS_DFX | 30+ | loop.c, signal.c, async.c |
| ASYNC_STACKTRACE | 12+ | threadpool.c, timer.c |
| SUPPORT_INTERRUPT | 3 | loop.c, core.c |

---

## 2.9 升级建议

### 2.9.1 风险等级

| 功能 | 升级风险 | 说明 |
|-----|---------|------|
| USE_FFRT | 🔴 高 | 大量代码依赖 FFRT API |
| USE_OHOS_DFX | 🟡 中 | 接口相对稳定 |
| ASYNC_STACKTRACE | 🟢 低 | 独立模块 |
| SUPPORT_INTERRUPT | 🟢 低 | 影响范围小 |

### 2.9.2 升级检查清单

- [ ] 检查 FFRT API 是否有变更
- [ ] 检查 DFX 头文件路径是否变更
- [ ] 验证 loop magic 定义是否冲突
- [ ] 测试 QoS 分级是否正常工作
- [ ] 验证异步堆栈功能
- [ ] 检查 fdsan 集成是否正常

### 2.9.3 回归测试建议

1. **功能测试**: 验证所有 QoS 级别任务执行
2. **稳定性测试**: 长时间运行事件循环
3. **性能测试**: 对比升级前后性能指标
4. **兼容性测试**: 验证下游模块正常工作

---

## 2.10 总结

libuv 在 OH 中采用**无 Patch**设计，通过**条件编译**和**新增源文件**实现定制化：

| 特性 | 实现方式 | 重要性 |
|-----|---------|-------|
| FFRT 集成 | USE_FFRT 宏 | ⭐⭐⭐⭐⭐ |
| DFX 诊断 | USE_OHOS_DFX 宏 | ⭐⭐⭐⭐ |
| 异步堆栈 | ASYNC_STACKTRACE 宏 | ⭐⭐⭐ |
| 中断支持 | SUPPORT_INTERRUPT 宏 | ⭐⭐ |

**维护建议**:
- 保持条件编译宏的命名一致性
- 定期检查上游版本的新特性
- 完善回归测试覆盖 OH 特有功能

---

*注: 由于无 Patch 文件，本文档分析的是源码级别的定制化内容*
