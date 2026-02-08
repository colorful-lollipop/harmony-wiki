# FFRT 安全风险评审

## 评审范围

本节对 FFRT 项目进行安全风险分析，基于以下范围的代码审计：

| 范围 | 说明 |
|------|------|
| **核心模块** | `src/core/`, `src/sched/`, `src/eu/` |
| **同步模块** | `src/sync/` |
| **队列模块** | `src/queue/` |
| **任务管理** | `src/tm/` |
| **依赖管理** | `src/dm/` |
| **工具函数** | `src/util/` |

**不在评审范围内**:
- 测试代码 (`test/`)
- 第三方依赖 (`third_party/`)
- 构建脚本

## 攻击面分析

### 外部输入点

| 输入类型 | 入口文件 | 说明 |
|----------|----------|------|
| **API 参数** | `interfaces/kits/c/*.h` | 所有 C API 的输入参数 |
| **函数闭包** | `src/core/task.cpp` | 任务函数指针和参数 |
| **内存指针** | `type_def.h` | 依赖指针、数据指针 |
| **配置参数** | `ffrt.gni` | 构建时配置 |

### 敏感操作

| 操作类型 | 实现位置 | 风险等级 |
|----------|----------|----------|
| **内存分配** | `src/util/slab.cpp` | 中 |
| **线程创建** | `src/eu/worker_thread.cpp` | 低 |
| **系统调用** | `src/sync/thread.cpp` | 低 |
| **文件操作** | `src/dfx/dump/` | 中 |
| **IPC 通信** | `src/ipc/ipc.cpp` | 低 |

### 信任边界

```
┌─────────────────────────────────────────────────────────┐
│                   不可信区域                            │
│  ┌─────────────────────────────────────────────────┐   │
│  │           用户代码 (Tasks)                       │   │
│  │  - 任务函数可能包含恶意代码                       │   │
│  │  - 依赖指针可能指向非法内存                       │   │
│  └─────────────────────────────────────────────────┘   │
│                         │                              │
│                         ▼ FFRT API                     │
│  ┌─────────────────────────────────────────────────┐   │
│  │           FFRT 运行时 (libffrt.z.so)             │   │
│  │  - 需要防御恶意输入                               │   │
│  │  - 需要安全内存管理                               │   │
│  └─────────────────────────────────────────────────┘   │
│                         │                              │
│                         ▼ 系统调用                     │
│  ┌─────────────────────────────────────────────────┐   │
│  │           内核 / 系统服务                         │   │
│  └─────────────────────────────────────────────────┘   │
│                   可信区域                              │
└─────────────────────────────────────────────────────────┘
```

## 已识别风险点

### 风险 1: 空指针解引用 (中等风险)

**证据**: `src/core/task.cpp:70-73`

```cpp
void DestroyFunctionWrapper(ffrt_function_header_t* f,
    ffrt_function_kind_t kind = ffrt_function_kind_general)
{
    if (f == nullptr || f->destroy == nullptr) {  // ✓ 有检查
        return;
    }
    // ...
}
```

**分析**: 在部分路径中存在空指针检查不完整的情况

**触发条件**:
```cpp
// 用户可能传递空函数指针
ffrt_submit_f(nullptr, nullptr, nullptr, nullptr, nullptr);
```

**影响**: 程序崩溃 (SIGSEGV)

**修复建议**:
```cpp
// 在所有 API 入口添加参数验证
FFRT_API int ffrt_submit_base(ffrt_function_header_t* f, ...) {
    if (f == nullptr || f->exec == nullptr) {
        return ffrt_error_inval;
    }
    // ...
}
```

---

### 风险 2: 栈溢出 (中等风险)

**证据**: `src/eu/co_routine.cpp:55-69`

```cpp
static inline void CoStackCheck(CoRoutine* co)
{
    if (unlikely(co->stkMem.magic != STACK_MAGIC)) {
        // 栈溢出检测 - 仅在 debug 模式有效
        FFRT_SYSEVENT_LOGE("stack over flow...\n");
        abort();
    }
}
```

**分析**: 
- 协程栈大小可通过 `ffrt_task_attr_set_stack_size()` 配置
- 默认栈大小为 1MB (`ffrt.gni:18`: `ffrt_stack_size = "1 << 20"`)
- 栈溢出检测仅在 magic number 不匹配时触发

**触发条件**:
```cpp
// 任务中使用大数组或深层递归
ffrt::task_attr attr;
attr.stack_size(64 * 1024);  // 64KB，可能不够

ffrt::submit([&]() {
    int large_array[100000];  // 400KB 栈空间
    // ...
}, attr);
```

**影响**: 内存损坏、未定义行为、安全漏洞

**修复建议**:
```cpp
// 1. 增加默认栈大小
// 2. 提供运行时栈使用监控
// 3. 在文档中明确提示大栈需求
```

---

### 风险 3: 整数溢出 (低风险)

**证据**: `src/core/task.cpp:47-48`

```cpp
constexpr uint64_t MAX_DELAY_US_COUNT = 1000000ULL * 100 * 60 * 60 * 24 * 365; // 100 year
constexpr uint64_t MAX_TIMEOUT_US_COUNT = 1000000ULL * 100 * 60 * 60 * 24 * 365; // 100 year
```

**分析**: 计算使用了 64 位无符号整数，理论上安全，但依赖编译器优化

**触发条件**:
```cpp
// 用户传递超大数据
ffrt_task_attr_set_delay(attr, UINT64_MAX);
```

**影响**: 环绕导致意外行为

**修复建议**:
```cpp
// 添加溢出检查
void ffrt_task_attr_set_delay(ffrt_task_attr_t* attr, uint64_t delay_us) {
    if (delay_us > MAX_DELAY_US_COUNT) {
        delay_us = MAX_DELAY_US_COUNT;  // 截断或报错
    }
    // ...
}
```

---

### 风险 4: 资源泄漏 (低风险)

**证据**: `src/sync/mutex.cpp`

**分析**: 
- 互斥锁、条件变量等同步原语需要显式销毁
- 依赖用户正确调用销毁 API

**触发条件**:
```cpp
// 忘记销毁
ffrt_mutex_t mutex;
ffrt_mutex_init(&mutex, nullptr);
// 使用...
// 忘记 ffrt_mutex_destroy(&mutex)
```

**影响**: 资源泄漏、内存泄漏

**修复建议**:
```cpp
// C++ 封装自动管理
class mutex_guard {
    ffrt_mutex_t& mutex_;
public:
    ~mutex_guard() { ffrt_mutex_unlock(&mutex_); }
};
```

---

### 风险 5: 竞态条件 (低风险)

**证据**: `src/sync/sync.cpp` 及相关文件

**分析**: 多线程环境下存在竞态风险

**触发条件**:
```cpp
// 非原子操作
ffrt_task_attr_get_qos(attr);  // 读取
ffrt_task_attr_set_qos(attr, new_qos);  // 写入
// 中间可能被其他线程修改
```

**影响**: 数据竞争、不一致状态

**修复建议**:
```cpp
// 使用原子操作或锁保护
std::atomic<int> qos_value;
// 或
pthread_mutex_t attr_mutex = PTHREAD_MUTEX_INITIALIZER;
```

---

## 安全最佳实践

### 开发者建议

1. **参数验证**
   - 所有 API 入口添加参数检查
   - 拒绝空指针、无效值

2. **内存安全**
   - 避免裸指针传递，使用智能指针
   - 协程栈大小合理配置

3. **资源管理**
   - RAII 模式自动释放资源
   - C++ 封装优于 C API

4. **并发安全**
   - 使用原子操作
   - 最小化锁范围

### 构建时配置

| 配置 | 建议 | 说明 |
|------|------|------|
| `FFRT_BBOX_ENABLE` | 生产环境开启 | 记录 crash 状态 |
| `FFRT_LOG_LEVEL` | 生产环境设为 1 | 减少日志泄露 |
| `ASAN/TSAN` | 开发环境开启 | 检测内存/线程问题 |

## 审计结论

### 风险评估总结

| 风险类型 | 数量 | 严重程度 |
|----------|------|----------|
| 空指针解引用 | 1 | 中等 |
| 栈溢出 | 1 | 中等 |
| 整数溢出 | 1 | 低 |
| 资源泄漏 | 1 | 低 |
| 竞态条件 | 1 | 低 |

### 整体评估

**总体风险等级**: 🟡 中等

**评估理由**:
- 核心代码有基本的安全措施
- 存在一些边界条件处理不完善
- 建议在生产环境开启安全相关配置

### 改进建议

1. **高优先级**: 在 API 入口添加完整的参数验证
2. **中优先级**: 增强协程栈溢出检测
3. **低优先级**: 提供 C++ RAII 封装

## 相关文档

- [概览](01_Overview.md) - 项目介绍
- [架构设计](02_Architecture.md) - 安全架构
- [编译构建](04_Build.md) - 安全配置
- [故障排查](06_Troubleshooting.md) - 问题定位
