# FFRT API 参考

## 概述

FFRT 提供 C 和 C++ 两套 API，主要通过 `libffrt.z.so` 提供。

**API 分布**:
- **C API**: `interfaces/kits/c/`
- **C++ API**: `interfaces/kits/cpp/`
- **Inner API**: `interfaces/inner_api/` (内部使用)

## 头文件清单

### C API 头文件

| 头文件 | 说明 | 主要内容 |
|--------|------|----------|
| `task.h` | 任务管理 | 任务提交、属性、句柄 |
| `queue.h` | 队列管理 | 串行/并发队列 |
| `mutex.h` | 互斥锁 | mutex 实现 |
| `condition_variable.h` | 条件变量 | 条件变量实现 |
| `shared_mutex.h` | 共享互斥锁 | rwlock 实现 |
| `sleep.h` | 睡眠 | 线程睡眠 |
| `timer.h` | 定时器 | 定时器管理 |
| `loop.h` | 事件循环 | loop 接口 |
| `fiber.h` | 协程 | fiber 接口 |
| `type_def.h` | 类型定义 | QoS、错误码等 |

### C++ API 头文件

| 头文件 | 说明 | 主要内容 |
|--------|------|----------|
| `task.h` | 任务管理 | submit, wait, task_attr, task_handle |
| `queue.h` | 队列管理 | queue, serial_queue, concurrent_queue |
| `mutex.h` | 互斥锁 | mutex, lock, unique_lock |
| `condition_variable.h` | 条件变量 | condition_variable |
| `shared_mutex.h` | 共享互斥锁 | shared_mutex |
| `sleep.h` | 睡眠 | sleep_for, sleep_until |
| `pattern/` | 模式 | observer, publisher/subscriber |

## 核心 API 详解

### 任务提交 API

#### C API

| API | 说明 | 返回值 |
|-----|------|--------|
| `ffrt_task_attr_init()` | 初始化任务属性 | 0=成功, -1=失败 |
| `ffrt_task_attr_set_name()` | 设置任务名 | void |
| `ffrt_task_attr_get_name()` | 获取任务名 | const char* |
| `ffrt_task_attr_set_qos()` | 设置 QoS | void |
| `ffrt_task_attr_get_qos()` | 获取 QoS | ffrt_qos_t |
| `ffrt_task_attr_set_delay()` | 设置延迟 | void |
| `ffrt_task_attr_get_delay()` | 获取延迟 | uint64_t |
| `ffrt_submit_base()` | 提交任务 | void |
| `ffrt_submit_h_base()` | 提交并获取句柄 | ffrt_task_handle_t |
| `ffrt_submit_f()` | 简化提交任务 (since 20) | void |
| `ffrt_submit_h_f()` | 简化提交并获取句柄 (since 20) | ffrt_task_handle_t |
| `ffrt_wait()` | 等待所有任务完成 | void |
| `ffrt_wait_deps()` | 等待依赖完成 | void |
| `ffrt_this_task_get_id()` | 获取当前任务 ID | uint64_t |
| `ffrt_this_task_update_qos()` | 更新当前任务 QoS | 0=成功, -1=失败 |

**绑定位置**: `src/core/task.cpp`

#### C++ API

| API | 说明 | 返回值 |
|-----|------|--------|
| `ffrt::submit()` | 提交任务 | void |
| `ffrt::submit_h()` | 提交并获取句柄 | task_handle |
| `ffrt::wait()` | 等待任务 | void |
| `ffrt::this_task::get_id()` | 获取当前任务 ID | uint64_t |
| `ffrt::this_task::update_qos()` | 更新 QoS | int |

**绑定位置**: `interfaces/kits/cpp/task.h` (模板实现)

### 任务属性

```cpp
// C 示例
ffrt_task_attr_t attr;
ffrt_task_attr_init(&attr);
ffrt_task_attr_set_name(&attr, "my_task");
ffrt_task_attr_set_qos(&attr, ffrt_qos_user_initiated);
ffrt_task_attr_set_delay(&attr, 1000000); // 1秒延迟
ffrt_task_attr_destroy(&attr);

// C++ 示例
ffrt::task_attr attr("my_task");
attr.qos(ffrt::qos_user_initiated)
    .delay(1000000);
```

### 依赖管理

| API | 说明 | 参数 |
|-----|------|------|
| `dependence(data_ptr)` | 创建数据依赖 | const void* |
| `dependence(task_handle)` | 创建任务依赖 | task_handle |

```cpp
// C++ 示例
int* data = new int(0);

// 写入任务 - 输出依赖
ffrt::submit([](){ *data = 42; }, {}, {ffrt::dependence(data)});

// 读取任务 - 输入依赖
ffrt::submit([](){ printf("%d\n", *data); }, {ffrt::dependence(data)});
```

### 队列 API

#### C API

| API | 说明 | 返回值 |
|-----|------|--------|
| `ffrt_queue_attr_init()` | 初始化队列属性 | 0=成功 |
| `ffrt_queue_create()` | 创建队列 | ffrt_queue_t |
| `ffrt_queue_destroy()` | 销毁队列 | void |
| `ffrt_queue_submit()` | 提交任务到队列 | void |
| `ffrt_queue_wait()` | 等待队列任务完成 | void |

#### C++ API

| API | 说明 |
|-----|------|
| `ffrt::queue()` | 创建队列 |
| `ffrt::submit(queue, func)` | 提交任务到队列 |
| `ffrt::serial_queue()` | 创建串行队列 |
| `ffrt::concurrent_queue()` | 创建并发队列 |

### 同步原语 API

#### Mutex

| API | 说明 |
|-----|------|
| `ffrt_mutex_init()` | 初始化 mutex |
| `ffrt_mutex_lock()` | 加锁 |
| `ffrt_mutex_trylock()` | 尝试加锁 |
| `ffrt_mutex_unlock()` | 解锁 |
| `ffrt_mutex_destroy()` | 销毁 mutex |

#### Condition Variable

| API | 说明 |
|-----|------|
| `ffrt_cond_init()` | 初始化条件变量 |
| `ffrt_cond_signal()` | 通知单个等待者 |
| `ffrt_cond_broadcast()` | 通知所有等待者 |
| `ffrt_cond_wait()` | 等待条件 |
| `ffrt_cond_destroy()` | 销毁条件变量 |

### 定时器 API

| API | 说明 | 返回值 |
|-----|------|--------|
| `ffrt_timer_create()` | 创建定时器 | ffrt_timer_t |
| `ffrt_timer_start()` | 启动定时器 | 0=成功 |
| `ffrt_timer_stop()` | 停止定时器 | void |
| `ffrt_timer_destroy()` | 销毁定时器 | void |

## 类型定义

### QoS 枚举

```c
typedef enum {
    ffrt_qos_inherit = -1,          // 继承
    ffrt_qos_background,            // 后台任务
    ffrt_qos_utility,              // 实用工具
    ffrt_qos_default,              // 默认
    ffrt_qos_user_initiated,      // 用户发起
    ffrt_qos_deadline_request,    // Deadline 请求 (since 23)
    ffrt_qos_user_interactive,    // 用户交互 (since 23)
} ffrt_qos_default_t;
```

### 队列优先级

```c
typedef enum {
    ffrt_queue_priority_immediate,  // 立即执行
    ffrt_queue_priority_high,        // 高优先级
    ffrt_queue_priority_low,         // 低优先级
    ffrt_queue_priority_idle,        // 空闲优先级
} ffrt_queue_priority_t;
```

### 错误码

```c
typedef enum {
    ffrt_error = -1,           // 通用错误
    ffrt_success = 0,          // 成功
    ffrt_error_nomem = ENOMEM, // 内存不足
    ffrt_error_timedout,      // 超时
    ffrt_error_busy,          // 忙碌
    ffrt_error_inval = EINVAL, // 无效参数
} ffrt_error_t;
```

## 完整 API 清单

### C API 索引

| 分类 | API 数量 | 主要头文件 |
|------|----------|------------|
| 任务管理 | 20+ | task.h |
| 队列操作 | 15+ | queue.h |
| 同步原语 | 20+ | mutex.h, condvar.h |
| 定时器 | 10+ | timer.h |
| 事件循环 | 10+ | loop.h |
| 协程 | 10+ | fiber.h |

### C++ API 索引

| 分类 | API 数量 | 主要头文件 |
|------|----------|------------|
| 任务提交 | 30+ | cpp/task.h |
| 队列封装 | 20+ | cpp/queue.h |
| 同步封装 | 20+ | cpp/mutex.h, cpp/condvar.h |

## 使用示例

### 示例 1: 简单任务提交

```cpp
#include "ffrt.h"

int main() {
    // 提交一个简单任务
    ffrt::submit([]() {
        printf("Hello, FFRT!\n");
    });

    // 等待所有任务完成
    ffrt::wait();
    return 0;
}
```

### 示例 2: 带 QoS 的任务

```cpp
ffrt::task_attr attr("important_task");
attr.qos(ffrt::qos_user_initiated)
    .delay(1000000); // 1秒延迟

ffrt::submit([]() {
    // 重要任务逻辑
}, attr);
```

### 示例 3: 任务依赖

```cpp
int* shared_data = new int(0);

// 任务1: 写入数据
ffrt::submit([&]() {
    *shared_data = 42;
}, {}, {ffrt::dependence(shared_data)});

// 任务2: 读取数据 (依赖任务1)
ffrt::submit([&]() {
    printf("data = %d\n", *shared_data);
}, {ffrt::dependence(shared_data)});
```

### 示例 4: 串行队列

```cpp
ffrt::queue q(ffrt::queue_attr().qos(ffrt::qos_user_initiated));

// 任务按顺序执行
ffrt::submit(q, [](){ /* 任务1 */ });
ffrt::submit(q, [](){ /* 任务2 */ });
ffrt::submit(q, [](){ /* 任务3 */ });
```

## 相关文档

- [概览](01_Overview.md) - 基本概念
- [架构设计](02_Architecture.md) - 内部实现
- [编译构建](04_Build.md) - 构建配置
- [用户指南](docs/ffrt-api-guideline-c.md) - C API 详细指南
- [用户指南](docs/ffrt-api-guideline-cpp.md) - C++ API 详细指南
