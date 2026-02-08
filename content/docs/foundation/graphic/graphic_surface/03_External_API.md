# 对外 API

## 目的
本文档说明 graphic_surface 组件提供的对外 API。

## 适用范围
面向需要在应用或模块中直接使用 graphic_surface 功能的：
- 原生应用开发者（C/C++）
- 图形框架开发者
- 媒体框架开发者
- 跨进程 IPC 通信开发者

**重要说明**：graphic_surface **不提供 JavaScript/ArkTS API**。JS 应用通过高层框架（如 `@ohos.window`、`@ohos.graphics`）间接使用 graphic_surface 的功能。

## API 概览

### API 分类

| API 类型 | 头文件 | 语言 | 稳定性 |
|---------|---------|-------|---------|
| Native Window C API | `external_window.h` | C | **稳定** |
| Native Buffer C API | `native_buffer.h` | C | **稳定** |
| Surface C++ API | `surface.h` | C++ | **半稳定**（模块间） |
| Consumer Surface C++ API | `iconsumer_surface.h` | C++ | **半稳定**（模块间） |
| IPC Producer 接口 | `ibuffer_producer.h` | C++ | **稳定**（跨进程） |

### 无 JavaScript API

**证据**：
- 搜索 `napi_*` 函数：未找到
- 搜索 `NAPI_MODULE`：未找到
- 搜索 `napi_define_properties`：未找到

graphic_surface 是纯原生 C++ 库，位于图形子系统的底层。JavaScript/ArkTS API 绑定由上层框架提供：
- `window_window_manager` - 提供 @ohos.window
- `arkui_ace_engine` - 提供 UI 相关 JS API

## Native Window C API

### 头文件
`interfaces/inner_api/surface/external_window.h`

### 主要类型

#### OHNativeWindow
Native Window 句柄，表示窗口 Surface。

```c
typedef struct OHNativeWindow* OHNativeWindow;
```

#### OHNativeWindowBuffer
Native Buffer 句柄，表示图形 Buffer。

```c
typedef struct OHNativeWindowBuffer* OHNativeWindowBuffer;
```

#### OHNativeWindowBufferOps
Buffer 操作函数表。

```c
typedef struct OHNativeWindowBufferOps {
    int32_t (*IncRef)(OHNativeWindowBuffer* buffer);
    int32_t (*DecRef)(OHNativeWindowBuffer* buffer);
    int32_t (*GetNativeBuffer)(OHNativeWindowBuffer* buffer, void** nativeBuffer);
} OHNativeWindowBufferOps;
```

### 核心函数

#### 创建/销毁

| 函数 | 签名 | 说明 | 返回值 |
|------|--------|------|--------|
| OHNativeWindow_CreateSurface | `OHNativeWindow* OHNativeWindow_CreateSurface()` | 创建 Native Window Surface | 成功返回句柄，失败返回 NULL |
| OHNativeWindow_DestroySurface | `int32_t OHNativeWindow_DestroySurface(OHNativeWindow* window)` | 销毁 Native Window | GSERROR_* |

#### Buffer 操作

| 函数 | 签名 | 说明 | 返回值 |
|------|--------|------|--------|
| OHNativeWindow_RequestBuffer | `int32_t OHNativeWindow_RequestBuffer(OHNativeWindow* window, OHNativeWindowBuffer** buffer, int32_t* fence)` | 请求 Buffer（从 Free 队列获取） | GSERROR_* |
| OHNativeWindow_NativeWindowFlushBuffer | `int32_t OHNativeWindow_NativeWindowFlushBuffer(OHNativeWindow* window, OHNativeWindowBuffer* buffer, int fence, int32_t region[])` | 提交 Buffer（放入 Dirty 队列） | GSERROR_* |
| OHNativeWindow_NativeWindowCancelBuffer | `int32_t OHNativeWindow_NativeWindowCancelBuffer(OHNativeWindow* window, OHNativeWindowBuffer* buffer)` | 取消 Buffer（返回 Free 队列） | GSERROR_* |
| OHNativeWindow_NativeWindowQueueBuffer | `int32_t OHNativeWindow_NativeWindowQueueBuffer(OHNativeWindow* window, OHNativeWindowBuffer* buffer, int fence, int32_t region[])` | 队列化 Buffer（消费者端） | GSERROR_* |
| OHNativeWindow_NativeWindowAcquireBuffer | `int32_t OHNativeWindow_NativeWindowAcquireBuffer(OHNativeWindow* window, OHNativeWindowBuffer** buffer, int32_t* fence)` | 获取 Buffer（从 Dirty 队列） | GSERROR_* |
| OHNativeWindow_NativeWindowReleaseBuffer | `int32_t OHNativeWindow_NativeWindowReleaseBuffer(OHNativeWindow* window, OHNativeWindowBuffer* buffer, int32_t fence)` | 释放 Buffer（返回 Free 队列） | GSERROR_* |

#### 属性设置

| 函数 | 签名 | 说明 | 参数 |
|------|--------|------|------|
| OHNativeWindow_NativeWindowHandleOptimize | `int32_t OHNativeWindow_NativeWindowHandleOptimize(OHNativeWindow* window, bool enable)` | 设置优化标志 | enable: true/false |
| OHNativeWindow_SetNativeWindowColorGamut | `int32_t OHNativeWindow_SetNativeWindowColorGamut(OHNativeWindow* window, GraphicColorGamut colorGamut)` | 设置色域 | GraphicColorGamut 枚举 |

#### Buffer 引用计数

| 函数 | 签名 | 说明 |
|------|--------|------|
| OHNativeWindow_NativeObjectReference | `int32_t OHNativeWindow_NativeObjectReference(OHNativeWindowBuffer* buffer)` | 增加引用计数 |
| OHNativeWindow_NativeObjectUnreference | `int32_t OHNativeWindow_NativeObjectUnreference(OHNativeWindowBuffer* buffer)` | 减少引用计数 |

## Native Buffer C API

### 头文件
`interfaces/inner_api/surface/native_buffer.h`

### 主要类型

#### OHNativeBuffer
Native Buffer 结构，包含 Buffer 的元数据和句柄。

```c
typedef struct OHNativeBuffer {
    int32_t width;            // 宽度
    int32_t height;           // 高度
    int32_t stride;           // 步长（每行像素数）
    int32_t format;           // 格式（RGBA、YUV 等）
    uint64_t usage;           // 使用标志
    void* virAddr;           // 虚拟地址
    int32_t size;            // 大小
    void* fence;             // 同步栅栏
    void* fd;               // 文件描述符
    int32_t key;            // Buffer 标识符
    int64_t timestamp;        // 时间戳
    void* extraData;         // 扩展数据
} OHNativeBuffer;
```

#### GraphicColorGamut
色域枚举。

```c
typedef enum {
    GRAPHIC_COLOR_GAMUT_SRGB,
    GRAPHIC_COLOR_GAMUT_DISPLAY_P3,
    GRAPHIC_COLOR_GAMUT_BT2020,
    // ...
} GraphicColorGamut;
```

### 核心函数

#### Buffer 管理

| 函数 | 签名 | 说明 |
|------|--------|------|
| OH_NativeBuffer_GetSeqNum | `int32_t OH_NativeBuffer_GetSeqNum(OHNativeBuffer* buffer)` | 获取 Buffer 序列号 |
| OH_NativeBuffer_GetWidth | `int32_t OH_NativeBuffer_GetWidth(OHNativeBuffer* buffer)` | 获取宽度 |
| OH_NativeBuffer_GetHeight | `int32_t OH_NativeBuffer_GetHeight(OHNativeBuffer* buffer)` | 获取高度 |
| OH_NativeBuffer_GetStride | `int32_t OH_NativeBuffer_GetStride(OHNativeBuffer* buffer)` | 获取步长 |
| OH_NativeBuffer_GetFormat | `int32_t OH_NativeBuffer_GetFormat(OHNativeBuffer* buffer)` | 获取格式 |
| OH_NativeBuffer_GetUsage | `uint64_t OH_NativeBuffer_GetUsage(OHNativeBuffer* buffer)` | 获取使用标志 |
| OH_NativeBuffer_GetVirAddr | `void* OH_NativeBuffer_GetVirAddr(OHNativeBuffer* buffer)` | 获取虚拟地址 |
| OH_NativeBuffer_GetSize | `int32_t OH_NativeBuffer_GetSize(OHNativeBuffer* buffer)` | 获取大小 |
| OH_NativeBuffer_GetFence | `void* OH_NativeBuffer_GetFence(OHNativeBuffer* buffer)` | 获取同步栅栏 |
| OH_NativeBuffer_GetNativeBuffer | `int32_t OH_NativeBuffer_GetNativeBuffer(OHNativeBuffer* buffer, void** nativeBuffer)` | 获取原生 Buffer |
| OH_NativeBuffer_GetExtraData | `int32_t OH_NativeBuffer_GetExtraData(OHNativeBuffer* buffer, void** extraData)` | 获取扩展数据 |

#### Buffer 引用计数

| 函数 | 签名 | 说明 |
|------|--------|------|
| OH_NativeBuffer_Reference | `int32_t OH_NativeBuffer_Reference(OHNativeBuffer* buffer)` | 增加引用计数 |
| OH_NativeBuffer_Unreference | `int32_t OH_NativeBuffer_Unreference(OHNativeBuffer* buffer)` | 减少引用计数 |

## Surface C++ API

### 头文件
`interfaces/inner_api/surface/surface.h`

### 核心类

#### Surface
Surface 基类，提供 Surface 创建和查询功能。

```cpp
class Surface {
public:
    // Surface 类型
    enum SurfaceType {
        SURFACE_TYPE_INVALID,
        SURFACE_TYPE_SURFACE,
        SURFACE_TYPE_CONSUMER,
    };

    // 创建 Surface
    static sptr<Surface> CreateSurface();
    static sptr<Surface> CreateSurfaceAsConsumer();
    static sptr<Surface> CreateSurfaceAsProducer(const sptr<IBufferProducer>& producer);

    // 查询 Surface 信息
    SurfaceType GetSurfaceType() const;
    const std::string& GetName() const;

    // 获取生产者/消费者接口
    sptr<IBufferProducer> GetProducer() const;
    sptr<IConsumerSurface> GetConsumer() const;
};
```

### Surface 创建方法

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `Surface::CreateSurface()` | 创建完整 Surface（包含生产者和消费者） | sptr<Surface> |
| `Surface::CreateSurfaceAsConsumer()` | 创建消费者 Surface（创建 BufferQueue） | sptr<Surface> |
| `Surface::CreateSurfaceAsProducer(const IBufferProducer&)` | 从 IBufferProducer 创建生产者 Surface | sptr<Surface> |

### Surface 信息查询

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `GetSurfaceType()` | 获取 Surface 类型 | SurfaceType |
| `GetName()` | 获取 Surface 名称 | std::string |

### 接口获取

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `GetProducer()` | 获取生产者接口（用于 Request/Flush Buffer） | sptr<IBufferProducer> |
| `GetConsumer()` | 获取消费者接口（用于 Acquire/Release Buffer） | sptr<IConsumerSurface> |

## Consumer Surface C++ API

### 头文件
`interfaces/inner_api/surface/iconsumer_surface.h`

### 核心类

#### IConsumerSurface
消费者 Surface 接口，提供 Buffer 获取和消费者操作。

```cpp
class IConsumerSurface : virtual public RefBase {
public:
    // 创建消费者 Surface
    static sptr<IConsumerSurface> Create();

    // 获取生产者接口
    virtual sptr<IBufferProducer> GetProducer() = 0;

    // 消费者操作
    virtual GSError AcquireBuffer(sptr<SurfaceBuffer>& buffer, sptr<SyncFence>& acquireFence,
                                int64_t timestamp, Rect* damage) = 0;
    virtual GSError ReleaseBuffer(sptr<SurfaceBuffer>& buffer, const sptr<SyncFence>& releaseFence) = 0;

    // 监听器注册
    virtual GSError RegisterConsumerListener(const sptr<IBufferConsumerListener>& listener) = 0;
    virtual GSError UnregisterConsumerListener() = 0;

    // 属性设置
    virtual GSError SetDefaultWidthAndHeight(uint32_t width, uint32_t height) = 0;
    virtual GSError SetDefaultUsage(uint64_t usage) = 0;
};
```

### 消费者操作方法

| 方法 | 说明 | 同步/异步 |
|------|------|-----------|
| `GetProducer()` | 获取生产者接口（用于跨进程共享） | 同步 |
| `AcquireBuffer()` | 从 Dirty 队列获取 Buffer（阻塞或非阻塞） | 可异步（noblock 模式） |
| `ReleaseBuffer()` | 释放 Buffer 到 Free 队列 | 同步 |
| `RegisterConsumerListener()` | 注册 Buffer 可用回调 | 异步回调 |
| `UnregisterConsumerListener()` | 取消注册 | 同步 |
| `SetDefaultWidthAndHeight()` | 设置默认宽高 | 同步 |
| `SetDefaultUsage()` | 设置默认使用标志 | 同步 |

### 消费者监听器

#### IBufferConsumerListener
Buffer 可用回调接口。

```cpp
class IBufferConsumerListener : virtual public RefBase {
public:
    virtual void OnBufferAvailable() = 0;  // Buffer 可用时调用
};
```

## IPC Producer 接口

### 头文件
`interfaces/inner_api/surface/ibuffer_producer.h`

### 核心接口

#### IBufferProducer
跨进程生产者接口（继承自 IRemoteBroker），提供 Buffer 操作方法。

```cpp
class IBufferProducer : virtual public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.Surface.IBufferProducer");

    // 连接/断开
    virtual GSError Connect(const sptr<IProducerListener>& listener, uint32_t producerApi) = 0;
    virtual GSError Disconnect() = 0;

    // Buffer 操作
    virtual GSError RequestBuffer(sptr<SurfaceBuffer>& buffer, const BufferRequestConfig& config,
                             sptr<SyncFence>& acquireFence) = 0;
    virtual GSError CancelBuffer(sptr<SurfaceBuffer>& buffer) = 0;
    virtual GSError FlushBuffer(sptr<SurfaceBuffer>& buffer, const sptr<SyncFence>& fence,
                            const BufferFlushConfig& config) = 0;
    virtual GSError AttachBuffer(const sptr<SurfaceBuffer>& buffer) = 0;
    virtual GSError DetachBuffer(const sptr<SurfaceBuffer>& buffer) = 0;

    // 查询操作
    virtual GSError GetQueueSize() = 0;
    virtual GSError SetQueueSize(uint32_t queueSize) = 0;
    virtual GSError GetName(std::string& name) = 0;

    // 属性设置
    virtual GSError SetTransform(GraphicTransformType transform) = 0;
    virtual GSError SetScalingMode(GraphicScalingMode mode) = 0;
    virtual GSError SetMetaData(const std::vector<GraphicHDRMetaData>& metaData) = 0;
    virtual GSError SetSurfaceSourceType(GraphicSurfaceSourceType sourceType) = 0;

    // 批量操作
    virtual GSError RequestBuffers(std::vector<sptr<SurfaceBuffer>>& buffers,
                               const BufferRequestConfig& config) = 0;
    virtual GSError FlushBuffers(const std::vector<BufferFlushConfig>& configs) = 0;
};
```

### IPC 方法分类

#### 连接管理
| 方法 | 说明 | 证据 |
|------|------|--------|
| `Connect()` | 建立连接，设置生产者监听器 | `ibuffer_producer.h:BUFFER_PRODUCER_CONNECT = 35` |
| `Disconnect()` | 断开连接 | `ibuffer_producer.h:BUFFER_PRODUCER_DISCONNECT = 36` |

#### Buffer 核心操作
| 方法 | 说明 | 证据 |
|------|------|--------|
| `RequestBuffer()` | 请求 Buffer（从 Free 队列） | `ibuffer_producer.h:BUFFER_PRODUCER_REQUEST_BUFFER = 0` |
| `FlushBuffer()` | 提交 Buffer（放入 Dirty 队列） | `ibuffer_producer.h:BUFFER_PRODUCER_FLUSH_BUFFER = 2` |
| `CancelBuffer()` | 取消 Buffer（返回 Free 队列） | `ibuffer_producer.h:BUFFER_PRODUCER_CANCEL_BUFFER = 1` |
| `AttachBuffer()` | 附加 Buffer 到队列 | `ibuffer_producer.h:BUFFER_PRODUCER_ATTACH_BUFFER = 43` |
| `DetachBuffer()` | 从队列分离 Buffer | `ibuffer_producer.h:BUFFER_PRODUCER_DETACH_BUFFER = 44` |

#### 队列管理
| 方法 | 说明 | 证据 |
|------|------|--------|
| `GetQueueSize()` | 获取队列大小 | `ibuffer_producer.h:BUFFER_PRODUCER_GET_QUEUE_SIZE = 3` |
| `SetQueueSize()` | 设置队列大小（默认 3，最大 64） | `ibuffer_producer.h:BUFFER_PRODUCER_SET_QUEUE_SIZE = 4` |

#### 属性设置
| 方法 | 说明 | 证据 |
|------|------|--------|
| `SetTransform()` | 设置变换（旋转/翻转） | `ibuffer_producer.h:BUFFER_PRODUCER_SET_TRANSFORM = 9` |
| `SetScalingMode()` | 设置缩放模式 | `ibuffer_producer.h:BUFFER_PRODUCER_SET_SCALING_MODE = 10` |
| `SetMetaData()` | 设置 HDR 元数据 | `ibuffer_producer.h:BUFFER_PRODUCER_SET_METADATA = 24` |
| `SetSurfaceSourceType()` | 设置 Surface 源类型 | `ibuffer_producer.h:BUFFER_PRODUCER_SET_SURFACE_SOURCE_TYPE = 26` |

#### 批量操作
| 方法 | 说明 | 证据 |
|------|------|--------|
| `RequestBuffers()` | 批量请求 Buffer | `ibuffer_producer.h:BUFFER_PRODUCER_REQUEST_BUFFERS = 38` |
| `FlushBuffers()` | 批量提交 Buffer | `ibuffer_producer.h:BUFFER_PRODUCER_FLUSH_BUFFERS = 39` |

### IPC 方法码（部分）
```cpp
enum {
    BUFFER_PRODUCER_REQUEST_BUFFER = 0,
    BUFFER_PRODUCER_CANCEL_BUFFER,
    BUFFER_PRODUCER_FLUSH_BUFFER,
    BUFFER_PRODUCER_GET_QUEUE_SIZE,
    BUFFER_PRODUCER_SET_QUEUE_SIZE,
    BUFFER_PRODUCER_GET_NAME,
    BUFFER_PRODUCER_SET_NAME,
    BUFFER_PRODUCER_SET_DEFAULT_WIDTH_AND_HEIGHT,
    BUFFER_PRODUCER_SET_DEFAULT_USAGE,
    BUFFER_PRODUCER_CONNECT,
    BUFFER_PRODUCER_DISCONNECT,
    BUFFER_PRODUCER_SET_SCALING_MODE,
    BUFFER_PRODUCER_SET_METADATA,
    // ... 共 50+ 方法
};
```

### 生产者监听器

#### IProducerListener
生产者监听器接口（用于 Buffer 释放回调）。

```cpp
class IProducerListener : virtual public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.Surface.IProducerListener");

    virtual void OnBufferReleased() = 0;
    virtual void OnBufferReleasedWithFence(sptr<SyncFence> fence) = 0;
    virtual void OnPropertyChange(uint64_t propertyKey, uint64_t propertyValue) = 0;
};
```

## 错误码

### 错误码范围
| 范围 | 类别 | 示例 |
|-------|-------|------|
| 0 | 成功 | GSERROR_OK = 0 |
| 30001000 | 内存操作 | GSERROR_NO_MEMORY = 30001000 |
| 40001000 | 无效参数 | GSERROR_INVALID_ARGUMENTS = 40001000 |
| 40301000 | 权限错误 | GSERROR_NO_PERMISSION = 40301000 |
| 404xxxxx | 连接错误 | GSERROR_NO_CONSUMER = 41202000 |
| 406xxxxx | Buffer/条目/范围错误 | GSERROR_NO_BUFFER = 40601000 |
| 412xxxxx | Surface 状态错误 | GSERROR_BUFFER_STATE_INVALID = 41207000 |
| 500xxxxx | 内部/API 错误 | GSERROR_API_FAILED = 50001000 |
| 50401000 | Binder 错误 | GSERROR_BINDER = 50401000 |

### 常用错误码
| 错误码 | 含义 | 触发条件 |
|---------|------|---------|
| GSERROR_OK | 成功 | 操作成功完成 |
| GSERROR_NO_BUFFER | 无可用 Buffer | Free 队列为空且队列已满 |
| GSERROR_NO_CONSUMER | 无消费者 | Consumer 未连接 |
| GSERROR_CONSUMER_IS_CONNECTED | 消费者已连接 | 另一进程已连接 |
| GSERROR_CONSUMER_DISCONNECTED | 消费者断开 | Consumer 进程死亡或断开连接 |
| GSERROR_BUFFER_STATE_INVALID | Buffer 状态无效 | 在错误状态下操作 Buffer |
| GSERROR_BUFFER_QUEUE_FULL | 队列已满 | Buffer 数量达到上限 |
| GSERROR_BINDER | IPC 错误 | Binder 通信失败 |

## API 使用示例

### 示例 1：生产者（UI 渲染）
```cpp
#include <surface/external_window.h>
#include <surface/native_buffer.h>

// 1. 创建 Native Window
OHNativeWindow* window = OHNativeWindow_CreateSurface();

// 2. 请求 Buffer
OHNativeWindowBuffer* buffer = nullptr;
int32_t fence = -1;
int ret = OHNativeWindow_RequestBuffer(window, &buffer, &fence);
if (ret != GSERROR_OK) {
    // 处理错误
}

// 3. 等待 Fence（GPU 完成）
if (fence >= 0) {
    SyncFenceWait(fence);
}

// 4. 获取 Buffer 地址并渲染
OHNativeBuffer* nativeBuffer = nullptr;
OHNativeWindow_NativeObjectReference(buffer);
OH_NativeBuffer_GetNativeBuffer(buffer, (void**)&nativeBuffer);
void* virAddr = OH_NativeBuffer_GetVirAddr(nativeBuffer);

// 渲染内容到 virAddr...

// 5. 提交 Buffer
int32_t region[4] = {0, 0, width, height};
ret = OHNativeWindow_NativeWindowFlushBuffer(window, buffer, fence, region);

// 6. 清理
OHNativeWindow_NativeObjectUnreference(buffer);
OHNativeWindow_DestroySurface(window);
```

### 示例 2：消费者（显示合成）
```cpp
#include <surface/iconsumer_surface.h>
#include <surface/ibuffer_producer.h>

// 1. 创建消费者 Surface
sptr<IConsumerSurface> consumer = IConsumerSurface::Create();
sptr<IBufferProducer> producer = consumer->GetProducer();

// 2. 设置默认参数
consumer->SetDefaultWidthAndHeight(1920, 1080);
consumer->SetDefaultUsage(BUFFER_USAGE_CPU_READ | BUFFER_USAGE_CPU_WRITE);

// 3. 注册监听器
sptr<IBufferConsumerListener> listener = new MyConsumerListener();
consumer->RegisterConsumerListener(listener);

// 4. 循环获取 Buffer
while (running) {
    sptr<SurfaceBuffer> buffer;
    sptr<SyncFence> acquireFence;
    int64_t timestamp;
    Rect damage;

    GSError ret = consumer->AcquireBuffer(buffer, acquireFence, timestamp, &damage);
    if (ret == GSERROR_OK) {
        // 等待 acquireFence
        acquireFence->Wait(100);  // 超时 100ms

        // 合成 Buffer
        CompositeBuffer(buffer);

        // 释放 Buffer
        sptr<SyncFence> releaseFence = nullptr;
        consumer->ReleaseBuffer(buffer, releaseFence);
    }
}

// 5. 清理
consumer->UnregisterConsumerListener();
```

## API 稳定性说明

### 稳定 API（可长期依赖）
- `external_window.h` - Native Window C API
- `native_buffer.h` - Native Buffer C API
- `ibuffer_producer.h` - IPC Producer 接口

### 半稳定 API（模块间使用）
- `surface.h` - Surface C++ 类
- `iconsumer_surface.h` - Consumer Surface C++ 接口

**注意**：半稳定 API 可能在版本间变更，模块间使用时需关注版本兼容性。

## 相关跳转
- [目录结构与模块职责](01_Directory_Structure.md) - API 文件位置
- [架构说明](02_Architecture.md) - Surface 架构与数据流
- [内部 API](04_Internal_API.md) - 模块间接口
- [安全风险评审](07_Security_Review.md) - 安全注意事项
