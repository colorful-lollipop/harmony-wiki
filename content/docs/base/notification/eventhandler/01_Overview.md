# 项目概览

## 目的

本文档介绍 EventHandler 部件的定位、核心能力、运行环境和关键概念，帮助新人快速理解项目整体架构。

## 适用范围

- 项目：@ohos/eventhandler
- 子系统：notification
- 版本：3.1
- 代码位置：`base/notification/eventhandler`

## 项目定位

EventHandler 是 OpenHarmony 事件处理基础库，提供线程间通信的核心能力。通过 EventRunner 创建新线程，将耗时操作抛到新线程执行，实现在不阻塞原线程的基础上合理处理耗时任务。

### 核心能力

1. **事件驱动编程模型**：提供事件发布-订阅机制
2. **线程管理**：支持新线程创建和事件循环
3. **优先级队列**：支持四级事件优先级（IMMEDIATE > HIGH > LOW > IDLE）
4. **跨线程通信**：通过 InnerEvent 实现线程间消息传递
5. **文件描述符监听**：支持 I/O 事件通知机制
6. **N-API 绑定**：为 JavaScript/ArkTS 提供 Emitter API
7. **CJ FFI 接口**：支持 Cangjie 语言调用

### 适用场景

- 需要在非主线程执行耗时任务的应用
- 跨线程事件发布和订阅
- 文件描述符（socket、pipe 等）I/O 事件监听
- 需要 FFRT（Flexible Function Runtime）调度的应用

## 运行环境

### 系统要求

- **操作系统**：OpenHarmonyOS（Linux 内核）
- **编译器**：Clang
- **构建系统**：GN + Ninja

### 依赖组件

| 组件 | 用途 |
|-------|------|
| hilog | 日志输出 |
| hitrace | 调用链追踪 |
| hichecker | 性能检测 |
| napi | N-API 绑定 |
| ffrt | Flexible Function Runtime |
| c_utils | 通用工具库 |
| init | 系统初始化 |
| ipc | IPC 通信（可选） |
| resource_schedule_service | 资源调度（可选） |
| runtime_core | 运行时核心（ANI 支持） |

### 编译特性（Feature Flags）

| 特性 | 默认值 | 描述 |
|------|---------|------|
| `eventhandler_feature_enable_pgo` | false | 启用 PGO 优化 |
| `eventhandler_feature_pgo_path` | "" | PGO profile 文件路径 |
| `eventhandler_feature_enable_main_runner_priority_lock` | false | 启用主线程优先级锁 |
| `eventhandler_ffrt_usage` | true | 启用 FFRT 支持 |
| `eh_hitrace_usage` | 自动检测 | 启用 HiTrace 追踪 |
| `resource_schedule_usage` | 自动检测 | 启用资源调度 |

### 资源占用

- **ROM**：500 KB
- **RAM**：1000 KB

## 关键概念

### EventRunner（事件运行器）

消息队列的循环分发器，每个线程只有一个 EventRunner，主要负责：
- 管理事件队列 EventQueue
- 不断从队列中取出 InnerEvent 分发至对应的 EventHandler 处理
- 支持两种线程模式：NEW_THREAD（创建新线程）和 FFRT（使用 FFRT）

**代码证据**：
- 定义：`interfaces/inner_api/event_runner.h:41`
- 实现：`frameworks/eventhandler/src/event_runner.cpp:292`（EventRunnerImpl 类）

### InnerEvent（内部事件）

线程间消息传递的实体封装，EventHandler 接收和处理的消息对象。

**特性**：
- 支持优先级：IMMEDIATE、HIGH、LOW、IDLE
- 支持延迟时间（delayTime）
- 支持定时任务（taskTime）
- 支持自定义数据（通过 variant 或 shared_ptr）

**代码证据**：
- 定义：`interfaces/inner_api/inner_event.h:76`
- 优先级枚举：`interfaces/inner_api/event_queue.h:42-46`

### EventHandler（事件处理器）

发送和处理消息的核心类，通过绑定 EventRunner 实现消息队列循环分发功能。

**代码证据**：
- 定义：`interfaces/inner_api/event_handler.h:51`
- 实现：`frameworks/eventhandler/src/event_handler.cpp:242`

### EventQueue（事件队列）

线程消息队列，管理 InnerEvent，在初始化 EventRunner 对象时需要创建与之关联的 EventQueue。

**代码证据**：
- 定义：`interfaces/inner_api/event_queue.h:15`
- 实现：`frameworks/eventhandler/src/event_queue.cpp:1`

### IoWaiter（I/O 等待器）

抽象接口，用于等待 I/O 事件或超时唤醒。

**实现类**：
- `EpollIoWaiter`：基于 Linux epoll 实现
- `FfrtDescriptorListener`：FFRT 文件描述符监听
- `DeamonIoWaiter`：后台 I/O 等待器
- `NoneIoWaiter`：空实现（默认）

**代码证据**：
- 接口定义：`frameworks/eventhandler/include/io_waiter.h:45`
- Epoll 实现：`frameworks/eventhandler/include/epoll_io_waiter.h:35`

## 系统能力（Syscap）

- **SystemCapability.Notification.Emitter**

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 详细代码组织
- [架构说明](03_Architecture.md) - 组件设计和数据流
- [N-API 接口](04_NAPI_API.md) - JS API 完整参考
- [内部 API](05_Inner_API.md) - C++ 接口文档
