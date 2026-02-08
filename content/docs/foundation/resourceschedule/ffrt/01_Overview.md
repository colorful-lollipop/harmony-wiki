# FFRT 项目概览

## 项目定位

FFRT (Function Flow Runtime) 是 OpenHarmony 系统中的**并发编程框架**，提供以数据依赖方式构建异步并发任务的能力。

### 核心能力

| 能力 | 说明 |
|------|------|
| **任务调度** | 基于 QoS 的任务调度，支持优先级和 deadline |
| **协程执行** | 用户态轻量级协程，减少线程开销 |
| **依赖管理** | 数据依赖和任务依赖图管理 |
| **同步原语** | 互斥锁、条件变量、读写锁等 |
| **队列支持** | 串行队列和并发队列 |
| **定时器** | 高精度定时器支持 |

### 设计目标

1. **提高并行度**: 充分利用多核平台计算资源
2. **提升线程利用率**: 协程调度减少线程切换开销
3. **降低线程总数**: 解决系统线程资源滥用问题
4. **集约化管理**: 保证系统对所有资源的集约化管理

## 运行环境

| 环境 | 要求 |
|------|------|
| **操作系统** | OpenHarmony Standard System / Linux |
| **系统能力** | SystemCapability.Resourceschedule.Ffrt.Core |
| **编译工具** | CMake 3.10+ / GN |
| **依赖库** | bounds_checking_function, c_utils, hilog, hisysevent, faultloggerd, napi |

## 基本概念

### 任务 (Task)

任务是 FFRT 编程模型的基本执行单元：

```cpp
// C++ 示例
ffrt::submit([]() {
    // 任务执行逻辑
}, ffrt::task_attr().qos(ffrt::qos_user_initiated));
```

**关键特性**:
- 任务可设置 QoS 等级 (background → user_interactive)
- 任务支持延迟提交 (delay)
- 任务支持命名和追踪
- 任务颗粒度建议最小 100us 量级

### 任务依赖 (Task Dependency)

任务之间可以声明依赖关系：

| 依赖类型 | 说明 | 示例 |
|----------|------|------|
| **数据依赖** | 通过数据指针表达依赖 | `dependence(data_ptr)` |
| **任务依赖** | 通过任务句柄表达依赖 | `dependence(task_handle)` |

```cpp
// C++ 示例：数据依赖
int* shared_data = new int(0);
ffrt::submit([](){ /* 写入 */ }, {}, {dependence(shared_data)});
ffrt::submit([](){ /* 读取 */ }, {dependence(shared_data)});
```

### QoS (Quality of Service)

QoS 定义任务的服务质量等级：

| QoS 等级 | 值 | 适用场景 |
|----------|-----|----------|
| `qos_background` | -1 | 后台任务，低优先级 |
| `qos_utility` | 0 | 实用工具任务 |
| `qos_default` | 1 | 默认优先级 |
| `qos_user_initiated` | 2 | 用户发起的重要任务 |
| `qos_deadline_request` | 3 | 有 deadline 要求的任务 |
| `qos_user_interactive` | 4 | 用户交互任务，最高优先级 |

### 协程 (Coroutine)

FFRT 采用**协程执行模型**：

- 每个协程有独立的执行上下文 (栈空间)
- 协程在用户态切换，无需内核参与
- 单个 Worker 线程可运行多个协程

### 队列 (Queue)

FFRT 支持两种任务队列：

| 队列类型 | 说明 | 适用场景 |
|----------|------|----------|
| **串行队列** | 任务按提交顺序依次执行 | 需要保持执行顺序的任务流 |
| **并发队列** | 多个任务可同时执行 | 并行计算、资源访问控制 |

## 编程模型对比

### 线程模型 vs FFRT 任务模型

| 维度 | 线程编程模型 | FFRT 任务模型 |
|------|-------------|---------------|
| **并行度挖掘** | 程序员手动创建多线程 | 编译器/调度器自动挖掘 |
| **线程创建** | 程序员负责，可能滥用 | FFRT 运行时统一管理 |
| **负载均衡** | 静态映射，可能不均 | 运行时动态调度 |
| **调度开销** | 内核态调度，开销大 | 用户态协程调度，轻量 |
| **依赖表达** | 隐式同步，增加切换 | 显式依赖声明 |

## 项目结构

```
ffrt/
├── interfaces/          # 接口目录
│   ├── kits/           # 对外接口 (C/C++)
│   │   ├── c/          # C API 头文件
│   │   └── cpp/        # C++ API 头文件
│   └── inner_api/      # 内部接口
├── src/                # 源码实现
│   ├── core/          # 核心模块
│   ├── sched/         # 调度器
│   ├── eu/            # 执行单元 (协程/Worker)
│   ├── tm/            # 任务管理
│   ├── queue/         # 队列管理
│   ├── sync/          # 同步原语
│   ├── dm/            # 依赖管理
│   ├── ipc/           # IPC 通信
│   ├── dfx/           # 维测功能
│   └── util/          # 工具函数
├── docs/              # 用户文档
├── examples/           # 示例代码
├── BUILD.gn           # GN 构建入口
└── CMakeLists.txt     # CMake 构建入口
```

## 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| 4.0 | 2025-02 | 当前版本 |

## 相关文档

- [架构设计](02_Architecture.md) - 深入理解内部实现
- [API 参考](03_API_Reference.md) - 接口使用说明
- [编译构建](04_Build.md) - 构建配置指南
- [用户指南](docs/README.md) - 官方文档索引
