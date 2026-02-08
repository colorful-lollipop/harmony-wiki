# 概览

## 目的
本文档提供 graphic_surface 组件的高层概览，包括项目定位、核心能力、运行环境和关键概念。

## 适用范围
面向需要理解 graphic_surface 项目定位的：
- 新加入 OpenHarmony 图形子系统的开发者
- 需要与 Surface 组件集成的模块开发者
- 图形/媒体相关的系统工程师

## 项目定位

### 职责
graphic_surface 是 OpenHarmony 图形子系统的基础组件，负责：
- **共享内存管理**：为图形和媒体数据提供跨进程零拷贝传输机制
- **Buffer 生命周期管理**：管理图形 Buffer 的分配、分配、回收、释放
- **跨进程同步**：通过 IPC 传输 Buffer 句柄，通过共享内存传输数据
- **生产者-消费者协调**：实现 Free/Dirty 队列，协调 Buffer 在生产者和消费者之间的流转

### 边界
**负责**：
- Buffer 的分配、引用计数、释放
- BufferHandle 的跨进程传输
- 生产者-消费者队列管理
- 同步栅栏（SyncFence）机制

**不负责**：
- 图形内容的绘制（由上层 UI 框架负责）
- Buffer 的最终显示/合成（由 SurfaceFlinger/WMS 负责）
- 硬件加速的具体实现（通过 GPU HAL 接口）
- JavaScript/ArkTS API 绑定（由高层框架提供）

## 核心能力

### 1. 共享内存管理
- **证据**：`interfaces/inner_api/surface/surface_buffer.h` - SurfaceBuffer 接口定义
- **证据**：`surface/src/surface_buffer_impl.cpp` - Buffer 实现与生命周期管理
- 支持不同格式（RGBA, YUV, RGB 等）
- 支持连续物理内存（提高传输速率）
- 支持多种内存类型（虚拟内存、DMA-BUF、ION 等）

### 2. 跨进程 Buffer 传输
- **证据**：`interfaces/inner_api/buffer_handle/buffer_handle.h` - BufferHandle 结构定义
- **证据**：`buffer_handle/src/buffer_handle.cpp:WriteBufferHandle()` - IPC 序列化
- **证据**：`interfaces/inner_api/surface/ibuffer_producer.h` - IBufferProducer IPC 接口
- IPC 传输 BufferHandle（包含 fd、stride、size 等元数据）
- 共享内存传输像素数据（零拷贝）
- 支持元数据（HDR、颜色空间等）传输

### 3. 生产者-消费者队列
- **证据**：`surface/src/buffer_queue.cpp` - BufferQueue 核心实现
- **证据**：`surface/src/buffer_queue_producer.cpp` - 生产者端实现
- **证据**：`surface/src/buffer_queue_consumer.cpp` - 消费者端实现
- Free 队列（可用 Buffer）
- Dirty 队列（待消费 Buffer）
- Buffer 轮转机制（生产 → 消费 → 回收）

### 4. 同步栅栏（SyncFence）
- **证据**：`sync_fence/include/sync_fence.h` - SyncFence 接口定义
- **证据**：`sync_fence/src/sync_fence.cpp` - 同步栅栏实现
- 基于 Linux dma_fence 的同步机制
- 跨进程 GPU/CPU 同步
- Fence 合并与等待

### 5. 委托模式支持
- **证据**：`surface/src/producer_surface_delegator.cpp` - 生产者委托
- **证据**：`surface/src/consumer_surface_delegator.cpp` - 消费者委托
- **证据**：`surface/src/surface_delegate.h` - SurfaceDelegate 接口
- 支持跨进程委托（生产者/消费者分离）
- 支持插件扩展（APS 插件）

## 运行环境

### 系统要求
- **证据**：`bundle.json:15` - `"adapted_system_type": [ "standard" ]`
- **证据**：`bundle.json:16-17` - ROM 10000KB, RAM 10000KB
- OpenHarmony 标准系统（Standard System）
- Linux 内核（支持 dma_fence、dma-buf）
- OpenHarmony IPC 框架

### 依赖组件
| 组件 | 用途 | 证据 |
|------|------|------|
| ipc | 跨进程通信 | `bundle.json:35` |
| hilog | 日志记录 | `bundle.json:31` |
| hitrace | 性能追踪 | `bundle.json:32` |
| hisysevent | 系统事件 | `bundle.json:33` |
| samgr | 系统能力管理器 | `bundle.json:36` |
| access_token | 访问令牌/权限 | `bundle.json:23` |
| selinux_adapter | SELinux 安全策略 | `bundle.json:37` |
| c_utils | C 工具库 | `bundle.json:26` |
| eventhandler | 事件处理 | `bundle.json:29` |

### 编译产物
| 产物 | 类型 | 用途 |
|------|------|------|
| surface.so | 共享库 | Surface 核心库 |
| sync_fence.so | 共享库 | 同步栅栏库 |
| buffer_handle.so | 共享库 | Buffer 句柄库 |
| surface_static | 静态库 | Surface 静态版本（某些场景） |

## 关键概念

### Surface vs Buffer vs BufferHandle
- **Surface**：Buffer 的容器和管理器，提供队列和生命周期管理
- **Buffer**：实际的图形数据存储，指向共享内存
- **BufferHandle**：跨进程传输的句柄，包含 fd、stride、size 等元数据

### 生产者 vs 消费者
- **生产者**：创建/修改 Buffer 的组件（如 UI 框架、视频解码器）
- **消费者**：读取/使用 Buffer 的组件（如 WMS、SurfaceFlinger、视频编码器）
- **典型场景**：UI（生产者）→ Surface → WMS（消费者）

### Free/Dirty 队列
- **Free 队列**：可用的 Buffer 列表，生产者从这里获取
- **Dirty 队列**：已填充待消费的 Buffer 列表，消费者从这里获取
- **流转**：生产者从 Free 取 → 绘制 → 放入 Dirty → 消费者从 Dirty 取 → 合成 → 放回 Free

### IPC vs 共享内存
- **IPC 层**：传输控制信息（BufferHandle、请求、响应），有拷贝开销
- **共享内存层**：传输图形数据，零拷贝，但需要文件描述符传递

### SyncFence
- **用途**：同步 GPU/CPU 操作，避免数据竞争
- **机制**：基于 Linux dma_fence，支持跨进程信号传递
- **典型场景**：生产者写入 Buffer 后，设置 Fence 表示完成；消费者等待 Fence 后才读取

## Feature Flags
| Flag | 默认值 | 说明 | 证据 |
|------|--------|------|------|
| graphic_surface_feature_tv_metadata_enable | false | 启用 TV 元数据支持 | `bundle.json:19`, `graphic_surface_config.gni:19` |
| graphic_2d_ext_delegator | false | 启用 2D 扩展委托模式 | `graphic_surface_config.gni:17` |
| graphic_2d_ext_delegator_gni | "" | 委托模式 GNI 配置路径 | `graphic_surface_config.gni:18` |
| ROSEN_TRACE_DISABLE | 条件编译 | 在交叉平台禁用追踪 | `graphic_surface_config.gni:28` |

## 典型使用场景

### 场景 1：UI 送显
```
UI 框架（生产者）
  → RequestBuffer (IPC)
  → 写入共享内存
  → FlushBuffer (IPC) + 设置 AcquireFence
WMS（消费者）
  → AcquireBuffer
  → 等待 AcquireFence
  → 合成显示
  → ReleaseBuffer + 设置 ReleaseFence
```

### 场景 2：视频播放
```
视频解码器（生产者）
  → 请求 Buffer
  → 解码 YUV 数据写入 Buffer
  → FlushBuffer + Fence
SurfaceFlinger（消费者）
  → 获取 Buffer
  → 合成到显示设备
  → 释放 Buffer
```

### 场景 3：视频录制
```
摄像头（生产者）
  → 请求 Buffer
  → 捕获视频帧写入 Buffer
  → FlushBuffer
视频编码器（消费者）
  → 获取 Buffer
  → 编码 YUV 数据
  → 释放 Buffer
```

## 架构位置

graphic_surface 在 OpenHarmony 图形子系统中的位置：

```
┌─────────────────────────────────────────┐
│   应用层 (ArkTS/JS Applications)          │
│   @ohos.window, @ohos.graphics 等       │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│   图形框架层                              │
│   Window Manager, SurfaceFlinger          │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│   graphic_surface (本模块)               │
│   ┌──────────┐  ┌──────────┐          │
│   │ Surface  │  │ SyncFence│          │
│   └──────────┘  └──────────┘          │
│   ┌──────────────────────────────────┐ │
│   │ BufferQueue (Producer-Consumer)  │ │
│   └──────────────────────────────────┘ │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│   HAL / Driver 层                        │
│   GPU Driver, Display HAL, DMA-BUF     │
└─────────────────────────────────────────┘
```

## 性能特性

### 零拷贝传输
- 共享内存避免数据拷贝
- 仅传输 BufferHandle（轻量级控制结构）
- 性能优势：减少内存带宽消耗

### 连续物理内存
- 支持分配连续物理内存
- 适配硬件扫描（如摄像头、显示器）
- 性能优势：提高 DMA 传输效率

### 批量操作
- **证据**：`interfaces/inner_api/surface/ibuffer_producer.h` - REQUEST_BUFFERS, FLUSH_BUFFERS
- 支持批量请求/释放 Buffer
- 减少 IPC 调用次数
- 性能优势：降低跨进程通信开销

### 异步处理
- **证据**：`sync_fence/include/frame_sched.h` - FrameScheduler
- 支持异步 Buffer 申请/释放
- 支持回调机制
- 性能优势：避免阻塞调用线程

## 可靠性机制

### Buffer 引用计数
- **证据**：`surface/src/surface_buffer_impl.cpp` - SurfaceBufferImpl 引用计数
- 防止 Buffer 过早释放
- 支持多组件共享 Buffer

### 死亡通知
- **证据**：`surface/include/producer_surface.h` - ProducerSurfaceDeathRecipient
- 进程异常时自动清理资源
- 防止 Buffer 泄漏

### 错误传播
- **证据**：`interfaces/inner_api/common/graphic_common.h` - GSERROR_* 错误码
- 统一错误码定义
- 清晰的错误传播链路

### 队列容量控制
- **证据**：`surface/src/buffer_queue.cpp` - SetQueueSize
- 防止 Buffer 过度分配
- 避免内存浪费

## 相关跳转
- [目录结构与模块职责](01_Directory_Structure.md) - 详细代码组织
- [架构说明](02_Architecture.md) - 组件设计与数据流
- [对外 API](03_External_API.md) - Native C/C++ API 参考
- [内部 API](04_Internal_API.md) - 模块间接口
