# 内部 API

## 目的
本文档说明 graphic_surface 模块间的内部 API，包括接口定义、依赖方向、稳定性和可替换点。

## 适用范围
面向需要：
- 理解模块间交互的内部开发者
- 进行模块重构的架构师
- 集成 Surface 模块的组件开发者
- 替换或扩展 Surface 功能的开发者

**注意**：内部 API 主要用于模块间通信，稳定性低于对外 API。

## API 分类

### 稳定内部 API（长期支持）

#### Surface 模块内部 API

| 接口 | 头文件 | 依赖模块 | 用途 |
|--------|---------|---------|------|
| `IBufferProducer` | `interfaces/inner_api/surface/ibuffer_producer.h` | 所有消费者模块 | 跨进程 Buffer 操作（50+ 方法） |
| `IConsumerSurface` | `interfaces/inner_api/surface/iconsumer_surface.h` | 所有生产者模块 | 消费者接口创建和操作 |
| `IBufferConsumerListener` | `interfaces/inner_api/surface/ibuffer_consumer_listener.h` | 生产者模块 | Buffer 可用回调 |
| `IProducerListener` | `interfaces/inner_api/surface/ibuffer_producer_listener.h` | 消费者模块 | Buffer 释放回调 |

#### SyncFence 模块内部 API

| 接口 | 头文件 | 依赖模块 | 用途 |
|--------|---------|---------|------|
| `SyncFence` | `interfaces/inner_api/sync_fence/sync_fence.h` | Surface, GPU HAL | GPU/CPU 同步 |
| `NativeFence` | `interfaces/inner_api/utils/native_fence.h` | Surface, GPU HAL | Linux dma_fence 包装 |

#### BufferHandle 模块内部 API

| 接口 | 头文件 | 依赖模块 | 用途 |
|--------|---------|---------|------|
| `BufferHandle` | `interfaces/inner_api/buffer_handle/buffer_handle.h` | 所有模块 | 跨进程 Buffer 句柄 |
| `BufferHandleUtils` | `interfaces/inner_api/buffer_handle/buffer_handle_utils.h` | Surface, SyncFence | Buffer 句柄工具函数 |

### 半稳定内部 API（可能变更）

#### Surface 核心实现类

| 类 | 头文件 | 稳定性 | 可替换性 |
|------|---------|--------|---------|
| `BufferQueue` | `surface/include/buffer_queue.h` | 半稳定 | 不可替换（核心逻辑） |
| `BufferQueueProducer` | `surface/include/buffer_queue_producer.h` | 半稳定 | 可扩展（IRemoteStub 实现） |
| `BufferQueueConsumer` | `surface/include/buffer_queue_consumer.h` | 半稳定 | 可替换（消费者包装） |
| `ProducerSurface` | `surface/include/producer_surface.h` | 半稳定 | 可扩展（客户端实现） |
| `ConsumerSurface` | `surface/include/consumer_surface.h` | 半稳定 | 可替换（消费者实现） |
| `SurfaceBuffer` | `interfaces/inner_api/surface/surface_buffer.h` | 半稳定 | 可扩展（实现接口） |

#### Delegator 模式

| 接口 | 头文件 | 用途 |
|--------|---------|------|
| `ISurfaceDelegate` | `surface/include/surface_delegate.h` | Surface 委托接口 |
| `DelegatorAdapter` | `surface/include/delegator_adapter.h` | 委托函数注册 |
| `ProducerSurfaceDelegator` | `surface/include/producer_surface_delegator.h` | 生产者委托实现 |
| `ConsumerSurfaceDelegator` | `surface/include/consumer_surface_delegator.h` | 消费者委托实现 |

### 不稳定内部 API（实现细节）

以下接口为 `surface/include/` 中的实现细节，**不建议直接依赖**：

| 类 | 用途 | 稳定性 |
|------|------|--------|
| `BufferClientProducer` | IPC 客户端代理 | 低 |
| `SurfaceBufferImpl` | Buffer 实现 | 低 |
| `ProducerSurfaceDeathRecipient` | 死亡监听器 | 低 |
| `BufferUtils` | Buffer 工具 | 低 |

## 模块依赖关系

### 依赖方向图

```
┌─────────────────────────────────────────────────┐
│         高层框架                               │
│  (WindowManager, RS, UI Framework)            │
└────────────────┬────────────────────────────────┘
                 │ depends on
                 ▼
┌─────────────────────────────────────────────────┐
│         Surface 模块                         │
│  ┌───────────────────────────────────────┐   │
│  │ Surface, IConsumerSurface          │   │
│  │ ProducerSurface, ConsumerSurface     │   │
│  └───────────────────┬───────────────┘   │
│                      │                    │
└──────────────────────┼────────────────────┘
                       │ depends on
         ┌─────────────┼─────────────┐
         │             │             │
         ▼             ▼             ▼
┌──────────────┐  ┌──────────┐  ┌──────────┐
│ SyncFence    │  │BufferHandle│  │  Utils    │
│             │  │          │  │           │
└──────┬───────┘  └─────┬────┘  └─────┬─────┘
       │                   │             │
       │                   │             ▼
       │                   │      ┌──────────────┐
       │                   │      │  Sandbox     │
       │                   │      └──────────────┘
       │                   │
       ▼                   ▼
┌──────────────────────────────────────┐
│      底层依赖                    │
│  (c_utils, hilog, ipc,           │
│   eventhandler, hitrace, ...)        │
└──────────────────────────────────────┘
```

### 关键依赖矩阵

| 模块 | 依赖 | 使用目的 | 证据 |
|--------|--------|---------|------|
| Surface | SyncFence | GPU/CPU 同步（AcquireFence/ReleaseFence） | `surface/include/buffer_queue.h` |
| Surface | BufferHandle | Buffer 跨进程传输（fd passing） | `surface/src/buffer_queue_producer.cpp` |
| Surface | Sandbox | 跨平台 PID 获取 | `surface/src/buffer_queue_producer.cpp` |
| Surface | FrameReport | 帧性能统计 | `surface/src/consumer_surface.cpp` |
| Surface | HEBCWhiteList | Buffer 缓存白名单 | `surface/src/buffer_queue.cpp` |
| BufferHandle | c_utils | 字符串、内存工具 | `buffer_handle/src/buffer_handle.cpp` |
| SyncFence | eventhandler | 异步 Fence 等待 | `sync_fence/src/sync_fence.cpp` |

## 内部 API 详细说明

### IBufferProducer（跨进程生产者接口）

#### 接口定义
```cpp
class IBufferProducer : virtual public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.Surface.IBufferProducer");

    // 连接管理
    virtual GSError Connect(const sptr<IProducerListener>& listener,
                       uint32_t producerApi) = 0;
    virtual GSError Disconnect() = 0;

    // Buffer 核心操作
    virtual GSError RequestBuffer(sptr<SurfaceBuffer>& buffer,
                             const BufferRequestConfig& config,
                             sptr<SyncFence>& acquireFence) = 0;
    virtual GSError CancelBuffer(sptr<SurfaceBuffer>& buffer) = 0;
    virtual GSError FlushBuffer(sptr<SurfaceBuffer>& buffer,
                            const sptr<SyncFence>& fence,
                            const BufferFlushConfig& config) = 0;

    // 队列管理
    virtual GSError GetQueueSize() = 0;
    virtual GSError SetQueueSize(uint32_t queueSize) = 0;

    // 属性设置
    virtual GSError SetTransform(GraphicTransformType transform) = 0;
    virtual GSError SetMetaData(const std::vector<GraphicHDRMetaData>& metaData) = 0;

    // 批量操作
    virtual GSError RequestBuffers(std::vector<sptr<SurfaceBuffer>>& buffers,
                               const BufferRequestConfig& config) = 0;
    virtual GSError FlushBuffers(const std::vector<BufferFlushConfig>& configs) = 0;
};
```

#### 使用模块
- **WindowManager** - 为应用创建 ProducerSurface
- **UI Framework** - Request/Flush Buffer
- **Video Encoder** - 提交视频帧
- **Camera HAL** - 提交摄像头捕获帧

#### 调用模式
```
应用层
  ↓ ProducerSurface (客户端）
  ↓ BufferClientProducer (IPC 客户端）
  ↓ IBufferProducer 接口
  ↓ BufferQueueProducer (IPC 服务端）
  ↓ BufferQueue (核心逻辑）
  ↓ ConsumerSurface (服务端）
  ↓ 显示合成
```

### IConsumerSurface（消费者接口）

#### 接口定义
```cpp
class IConsumerSurface : virtual public RefBase {
public:
    // 工厂方法
    static sptr<IConsumerSurface> Create();

    // 获取生产者（用于跨进程）
    virtual sptr<IBufferProducer> GetProducer() = 0;

    // 消费者操作
    virtual GSError AcquireBuffer(sptr<SurfaceBuffer>& buffer,
                             sptr<SyncFence>& acquireFence,
                             int64_t timestamp,
                             Rect* damage) = 0;
    virtual GSError ReleaseBuffer(sptr<SurfaceBuffer>& buffer,
                             const sptr<SyncFence>& releaseFence) = 0;

    // 监听器
    virtual GSError RegisterConsumerListener(
        const sptr<IBufferConsumerListener>& listener) = 0;
    virtual GSError UnregisterConsumerListener() = 0;

    // 默认配置
    virtual GSError SetDefaultWidthAndHeight(uint32_t width, uint32_t height) = 0;
    virtual GSError SetDefaultUsage(uint64_t usage) = 0;
};
```

#### 使用模块
- **SurfaceFlinger/WMS** - 创建消费者 Surface
- **Render Service (RS)** - 合成显示
- **Video Decoder** - 获取解码帧
- **Camera Preview** - 获取摄像头帧

#### 典型实现
- `ConsumerSurface` - 消费者主实现（拥有 BufferQueue）
- `ConsumerSurfaceDelegator` - 委托实现（跨进程）

### SyncFence（同步栅栏接口）

#### 接口定义
```cpp
class SyncFence : virtual public RefBase {
public:
    // 状态查询
    enum FenceStatus {
        INVALID,     // 无效
        ACTIVE,      // 等待中
        SIGNALED,    // 已信号
        ERROR         // 错误
    };
    virtual FenceStatus GetStatus() const = 0;

    // 等待操作
    virtual GSError Wait(int32_t timeout) = 0;
    virtual GSError WaitAndReset(int32_t timeout, sptr<SyncFence>& next) = 0;

    // 复制/合并
    virtual sptr<SyncFence> Merge(const sptr<SyncFence>& fence) const = 0;
    virtual sptr<SyncFence> Clone() const = 0;
};
```

#### 使用模块
- **Surface** - AcquireFence/ReleaseFence 传递
- **GPU HAL** - GPU 操作完成信号
- **VSync** - 垂直同步信号
- **Display HAL** - 显示操作完成信号

#### 典型场景
```cpp
// 生产者端：设置 AcquireFence
sptr<SyncFence> acquireFence = producer->RequestBuffer(...);
// GPU 渲染...
GLESRender(buffer);
buffer->SetAcquireFence(gpuFence);  // GPU 完成信号

// 消费者端：等待 AcquireFence
sptr<SurfaceFence> fence = consumer->AcquireBuffer(...);
fence->Wait(5000);  // 等待 GPU 完成
Composite(buffer);

// 消费者端：设置 ReleaseFence
DisplayAndSignal(fence);
buffer->SetReleaseFence(displayFence);  // 显示完成信号
```

### BufferHandle（Buffer 句柄）

#### 结构定义
```cpp
struct BufferHandle {
    // 尺寸
    int32_t width;
    int32_t height;
    int32_t stride;
    int32_t size;

    // 格式
    int32_t format;      // RGBA, YUV420, etc.
    uint64_t usage;      // CPU/GPU 使用标志

    // 内存
    int32_t fd;          // 共享内存文件描述符
    uint64_t phyAddr;    // 物理地址（可选）

    // 保留字段
    int32_t key;
    int32_t reserveFds;
    int32_t reserveInts;

    // 扩展
    std::vector<int> reserveFds;
    std::vector<int> reserveInts;
};
```

#### IPC 序列化
```cpp
// buffer_handle.cpp:WriteBufferHandle()
bool WriteBufferHandle(MessageParcel &parcel, const BufferHandle &handle) {
    // 1. 写入元数据
    parcel.WriteUint32(handle.width);
    parcel.WriteUint32(handle.height);
    parcel.WriteInt32(handle.stride);
    parcel.WriteInt32(handle.size);
    parcel.WriteInt32(handle.format);
    parcel.WriteUint64(handle.usage);

    // 2. 写入 fd（关键：跨进程共享内存）
    parcel.WriteBool(validFd);
    parcel.WriteFileDescriptor(handle.fd);  // fd passing

    // 3. 写入保留字段
    parcel.WriteUint32(handle.reserveFds);
    parcel.WriteUint32(handle.reserveInts);
    // ...
}
```

#### 使用模块
- **Surface** - IPC 传输 Buffer
- **SyncFence** - Buffer 附加数据（fence fd）
- **GPU HAL** - Buffer 物理地址映射
- **Camera HAL** - 摄像头输出 Buffer

## 可替换点与扩展点

### 1. 委托模式（Delegation Pattern）

#### 扩展点：SurfaceDelegate
**接口**：`surface/include/surface_delegate.h`

**用途**：允许第三方接管 Surface 操作，用于：
- 跨进程委托（生产者和消费者在不同进程）
- 插件扩展（APS 插件）
- 自定义 Buffer 管理策略

**示例**：
```cpp
class CustomSurfaceDelegate : public ISurfaceDelegate {
public:
    GSError CreateSurface() override {
        // 自定义 Surface 创建逻辑
    }

    GSError RequestBuffer(...) override {
        // 自定义 Buffer 请求逻辑
    }

    GSError FlushBuffer(...) override {
        // 自定义 Buffer 提交流程
    }
};
```

**注册方式**：
```cpp
// delegator_adapter.cpp
DelegatorAdapter::GetInstance()->RegisterFunc(
    FunctionFlags::PRODUCER_CREATE_FUNC,
    reinterpret_cast<void*>(CustomCreateSurface));
```

#### 扩展点：ProducerSurfaceDelegator
**接口**：`surface/include/producer_surface_delegator.h`

**用途**：生产者端远程委托

**实现位置**：`surface/src/producer_surface_delegator.cpp`

#### 扩展点：ConsumerSurfaceDelegator
**接口**：`surface/include/consumer_surface_delegator.h`

**用途**：消费者端远程委托

**实现位置**：`surface/src/consumer_surface_delegator.cpp`

### 2. Buffer 队列策略

#### 扩展点：BufferQueue::RequestBufferLocked()
**位置**：`surface/src/buffer_queue.cpp`

**用途**：自定义 Buffer 分配策略
- Buffer 复用算法
- 队列满时策略（等待/拒绝/返回 oldest）
- Pre-allocation 策略

**修改方式**：继承 BufferQueue 类（不推荐）或修改源码

### 3. IPC 传输优化

#### 扩展点：BufferHandle 序列化
**位置**：`buffer_handle/src/buffer_handle.cpp`

**用途**：自定义 BufferHandle 序列化逻辑
- 添加加密层
- 添加压缩
- 自定义元数据

### 4. Frame 报告扩展

#### 扩展点：FrameReport
**接口**：`utils/frame_report/export/frame_report.h`

**用途**：自定义帧性能上报
- 上报到自定义监控系统
- 添加额外性能指标
- 集成第三方 APM

**实现位置**：`utils/frame_report/src/frame_report.cpp`

### 5. SyncFence 实现

#### 扩展点：SyncFence
**接口**：`interfaces/inner_api/sync_fence/sync_fence.h`

**用途**：自定义同步机制
- 不同内核 fence 实现
- 跨平台 fence 适配

**实现位置**：`sync_fence/src/sync_fence.cpp`

## 稳定性说明

### API 稳定性级别

| 级别 | 标记 | 含义 | 兼容性保证 |
|--------|--------|------|------------|
| **稳定** | interfaces/inner_api/ | 跨版本保证 | 必须向后兼容 |
| **半稳定** | surface/include/ | 小版本可能变更 | 尽量向后兼容 |
| **不稳定** | surface/src/ 实现细节 | 随时变更 | 不保证兼容性 |

### 接口稳定性标注

#### 稳定接口（可长期依赖）
- ✅ `IBufferProducer` - 跨进程 IPC 契约
- ✅ `IConsumerSurface` - 消费者接口
- ✅ `IBufferConsumerListener` - 消费者回调
- ✅ `IProducerListener` - 生产者回调
- ✅ `SyncFence` - 同步接口
- ✅ `BufferHandle` - Buffer 句柄结构

#### 半稳定接口（模块间使用）
- ⚠️ `BufferQueue` - 可能优化队列算法
- ⚠️ `BufferQueueProducer` - 可能优化 IPC 路径
- ⚠️ `ProducerSurface` - 可能优化客户端缓存
- ⚠️ `ConsumerSurface` - 可能优化消费者逻辑
- ⚠️ `SurfaceBuffer` - 可能扩展功能

#### 不稳定接口（实现细节）
- ❌ `BufferClientProducer` - 实现细节
- ❌ `SurfaceBufferImpl` - 实现细节
- ❌ 内部辅助类 - 可能删除

### 版本兼容性策略

#### 对稳定接口
- 不得移除现有方法
- 新增方法使用默认实现（向后兼容）
- 不得修改方法签名

#### 对半稳定接口
- 可能优化算法（如 Buffer 复用策略）
- 可能新增功能
- 重大变更需要版本号递增

#### 对不稳定接口
- 可随时重构
- 可删除或重命名
- 不建议跨模块直接使用

## 模块间通信示例

### 示例 1：WindowManager 创建 Surface
```cpp
// WindowManager（消费者端）
#include <iconsumer_surface.h>

// 1. 创建消费者 Surface
sptr<IConsumerSurface> consumer = IConsumerSurface::Create();

// 2. 设置默认参数
consumer->SetDefaultWidthAndHeight(1920, 1080);
consumer->SetDefaultUsage(BUFFER_USAGE_HARDWARE_CONSUMER);

// 3. 注册监听器
sptr<IBufferConsumerListener> listener = new MyConsumerListener();
consumer->RegisterConsumerListener(listener);

// 4. 获取生产者接口（跨进程）
sptr<IBufferProducer> producer = consumer->GetProducer();

// 5. 将 producer 传递给应用（通过 Binder）
RegisterSurfaceToApp(producer);
```

### 示例 2：UI 框架使用 Surface
```cpp
// UI Framework（生产者端）
#include <surface.h>

// 1. 获取生产者接口（从 WindowManager）
sptr<IBufferProducer> producer = GetSurfaceFromWindowManager();

// 2. 创建 ProducerSurface（客户端）
sptr<Surface> surface = Surface::CreateSurfaceAsProducer(producer);

// 3. 请求 Buffer
sptr<SurfaceBuffer> buffer;
sptr<SyncFence> acquireFence;
BufferRequestConfig config = { .width = 1920, .height = 1080 };
surface->GetProducer()->RequestBuffer(buffer, config, acquireFence);

// 4. 等待 GPU 完成渲染
acquireFence->Wait(1000);

// 5. 获取 Buffer 地址
void* virAddr = buffer->GetVirAddr();

// 6. 渲染内容
RenderToBuffer(virAddr);

// 7. 提交 Buffer
sptr<SyncFence> releaseFence = nullptr;
BufferFlushConfig flushConfig = { .timestamp = GetCurrentTime() };
surface->GetProducer()->FlushBuffer(buffer, releaseFence, flushConfig);
```

## 相关跳转
- [目录结构与模块职责](01_Directory_Structure.md) - 接口文件位置
- [架构说明](02_Architecture.md) - 组件交互与数据流
- [对外 API](03_External_API.md) - 对外与内部 API 区分
- [GN Targets 与编译产物](05_GN_Targets.md) - 模块编译与依赖
