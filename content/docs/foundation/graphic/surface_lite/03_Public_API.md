# 对外 API 文档 (Kits)

> surface_lite 对外公开接口说明

---

## 1. 概述

### 1.1 模块说明

本模块提供 **C++ 原生接口**，用于在图形和媒体场景中申请和释放共享内存。**不提供 N-API (JavaScript) 接口**。

### 1.2 头文件位置

```
interfaces/kits/
├── surface.h                      # 主接口
├── surface_buffer.h               # Buffer 接口
├── surface_type.h                 # 类型定义
└── ibuffer_consumer_listener.h    # 监听器接口
```

### 1.3 命名空间

所有接口位于 `OHOS` 命名空间：

```cpp
namespace OHOS {
    class Surface;
    class SurfaceBuffer;
    class IBufferConsumerListener;
}
```

---

## 2. Surface 类

### 2.1 类定义

```cpp
// interfaces/kits/surface.h
class Surface {
public:
    static Surface* CreateSurface();
    virtual ~Surface();
    
    // 队列管理
    virtual void SetQueueSize(uint8_t queueSize) = 0;
    virtual uint8_t GetQueueSize() = 0;
    
    // Buffer 属性
    virtual void SetWidthAndHeight(uint32_t width, uint32_t height) = 0;
    virtual uint32_t GetWidth() = 0;
    virtual uint32_t GetHeight() = 0;
    virtual void SetFormat(uint32_t format) = 0;
    virtual uint32_t GetFormat() = 0;
    virtual void SetStrideAlignment(uint32_t strideAlignment) = 0;
    virtual uint32_t GetStrideAlignment() = 0;
    virtual uint32_t GetStride() = 0;
    virtual void SetSize(uint32_t size) = 0;
    virtual uint32_t GetSize() = 0;
    virtual void SetUsage(uint32_t usage) = 0;
    virtual uint32_t GetUsage() = 0;
    
    // 用户数据
    virtual void SetUserData(const std::string& key, const std::string& value) = 0;
    virtual std::string GetUserData(const std::string& key) = 0;
    
    // Buffer 操作
    virtual SurfaceBuffer* RequestBuffer(uint8_t wait = 0) = 0;
    virtual int32_t FlushBuffer(SurfaceBuffer* buffer) = 0;
    virtual SurfaceBuffer* AcquireBuffer() = 0;
    virtual bool ReleaseBuffer(SurfaceBuffer* buffer) = 0;
    virtual void CancelBuffer(SurfaceBuffer* buffer) = 0;
    
    // 监听器
    virtual void RegisterConsumerListener(IBufferConsumerListener& listener) = 0;
    virtual void UnregisterConsumerListener() = 0;
};
```

### 2.2 静态方法

#### CreateSurface

```cpp
static Surface* CreateSurface();
```

**描述**: 创建 Surface 实例 (Consumer 端使用)

**返回值**:
- 成功: 有效的 Surface 指针
- 失败: `nullptr`

**调用位置**: `frameworks/surface.cpp:20`

**使用示例**:
```cpp
OHOS::Surface* surface = OHOS::Surface::CreateSurface();
if (surface == nullptr) {
    // 处理错误
}
```

**注意**:
- 创建的 Surface 处于 Consumer 模式
- 内部创建 BufferQueue、BufferQueueProducer、BufferQueueConsumer
- 需要手动 `delete` 释放

---

### 2.3 队列管理

#### SetQueueSize

```cpp
virtual void SetQueueSize(uint8_t queueSize) = 0;
```

**描述**: 设置 Buffer 队列大小

**参数**:
| 参数 | 类型 | 范围 | 说明 |
|------|------|------|------|
| queueSize | uint8_t | [1, 10] | 队列大小 |

**默认值**: 1

**校验位置**: `frameworks/surface_impl.cpp:179-183`

```cpp
void SurfaceImpl::SetQueueSize(uint8_t queueSize) {
    RETURN_IF_FAIL(producer_);
    RETURN_IF_FAIL(queueSize >= SURFACE_MIN_QUEUE_SIZE && 
                   queueSize <= SURFACE_MAX_QUEUE_SIZE);
    producer_->SetQueueSize(queueSize);
}
```

**错误处理**: 参数越界时静默返回 (记录日志)

#### GetQueueSize

```cpp
virtual uint8_t GetQueueSize() = 0;
```

**描述**: 获取当前队列大小

**返回值**: 队列大小 [1, 10]

---

### 2.4 Buffer 属性设置

#### SetWidthAndHeight

```cpp
virtual void SetWidthAndHeight(uint32_t width, uint32_t height) = 0;
```

**描述**: 设置 Buffer 宽度和高度，用于计算 Buffer 大小

**参数**:
| 参数 | 类型 | 范围 | 说明 |
|------|------|------|------|
| width | uint32_t | (0, 7680] | 宽度(像素) |
| height | uint32_t | (0, 7680] | 高度(像素) |

**校验位置**: `frameworks/surface_impl.cpp:102-107`

```cpp
void SurfaceImpl::SetWidthAndHeight(uint32_t width, uint32_t height) {
    RETURN_IF_FAIL(producer_ != nullptr);
    RETURN_IF_FAIL(width > 0 && width <= SURFACE_MAX_WIDTH);
    RETURN_IF_FAIL(height > 0 && height <= SURFACE_MAX_HEIGHT);
    producer_->SetWidthAndHeight(width, height);
}
```

**副作用**: 会触发 Buffer 重新分配 (Reset)

#### SetFormat

```cpp
virtual void SetFormat(uint32_t format) = 0;
```

**描述**: 设置像素格式

**参数**: 参见 `ImageFormat` 枚举 (graphic_utils_lite)

**默认值**: `IMAGE_PIXEL_FORMAT_RGB565`

#### SetStrideAlignment

```cpp
virtual void SetStrideAlignment(uint32_t strideAlignment) = 0;
```

**描述**: 设置 stride 对齐字节数

**参数范围**: [4, 32]

**默认值**: 4

#### SetSize

```cpp
virtual void SetSize(uint32_t size) = 0;
```

**描述**: 直接设置 Buffer 大小 (覆盖自动计算)

**参数范围**: (0, 58982400] (8K × 8K)

#### SetUsage

```cpp
virtual void SetUsage(uint32_t usage) = 0;
```

**描述**: 设置 Buffer 使用场景

**参数**: `BufferConsumerUsage` 枚举值

| 值 | 含义 |
|----|------|
| 0 | `BUFFER_CONSUMER_USAGE_SORTWARE` - 虚拟内存 |
| 1 | `BUFFER_CONSUMER_USAGE_HARDWARE` - 物理内存 |
| 2 | `BUFFER_CONSUMER_USAGE_HARDWARE_CONSUMER_CACHE` - 消费者 Cache |
| 3 | `BUFFER_CONSUMER_USAGE_HARDWARE_PRODUCER_CACHE` - 生产者 Cache |

---

### 2.5 Buffer 操作

#### RequestBuffer

```cpp
virtual SurfaceBuffer* RequestBuffer(uint8_t wait = 0) = 0;
```

**描述**: 请求一个空闲 Buffer (Producer 操作)

**参数**:
| 参数 | 类型 | 值 | 说明 |
|------|------|-----|------|
| wait | uint8_t | 0 | 不等待，立即返回 |
| | | 1 | 阻塞等待直到有可用 Buffer |

**返回值**:
- 成功: 有效的 SurfaceBuffer 指针
- 失败: `nullptr`

**状态变化**: BUFFER_STATE_NONE → BUFFER_STATE_REQUEST

**实现位置**: `frameworks/buffer_queue.cpp:132-150`

**调用链**:
```
SurfaceImpl::RequestBuffer()
  └── BufferProducer::RequestBuffer()
      ├── BufferQueueProducer: BufferQueue::RequestBuffer()
      └── BufferClientProducer: SendRequest(REQUEST_BUFFER)
```

#### FlushBuffer

```cpp
virtual int32_t FlushBuffer(SurfaceBuffer* buffer) = 0;
```

**描述**: 提交 Buffer 到消费队列 (Producer 操作)

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| buffer | SurfaceBuffer* | 要提交的 Buffer |

**返回值**:
| 值 | 含义 |
|----|------|
| 0 | 成功 |
| -1 | 失败 (参数错误或状态无效) |
| SURFACE_ERROR_BUFFER_NOT_EXISTED | Buffer 不存在 |

**状态变化**: BUFFER_STATE_REQUEST → BUFFER_STATE_FLUSH

**副作用**: 触发 `OnBufferAvailable()` 回调

#### AcquireBuffer

```cpp
virtual SurfaceBuffer* AcquireBuffer() = 0;
```

**描述**: 获取待消费的 Buffer (Consumer 操作)

**返回值**:
- 成功: 有效的 SurfaceBuffer 指针
- 无可用 Buffer: `nullptr`

**状态变化**: BUFFER_STATE_FLUSH → BUFFER_STATE_ACQUIRE

#### ReleaseBuffer

```cpp
virtual bool ReleaseBuffer(SurfaceBuffer* buffer) = 0;
```

**描述**: 释放 Buffer 回空闲队列 (Consumer 操作)

**返回值**:
| 值 | 含义 |
|----|------|
| true | 成功 |
| false | 失败 (参数错误或状态无效) |

**状态变化**: BUFFER_STATE_ACQUIRE → BUFFER_STATE_RELEASE

#### CancelBuffer

```cpp
virtual void CancelBuffer(SurfaceBuffer* buffer) = 0;
```

**描述**: 取消 Buffer，返回空闲队列 (Producer 操作)

**使用场景**: 申请了 Buffer 但未绘制，放弃使用

**状态变化**: BUFFER_STATE_REQUEST → BUFFER_STATE_RELEASE

---

### 2.6 监听器

#### RegisterConsumerListener

```cpp
virtual void RegisterConsumerListener(IBufferConsumerListener& listener) = 0;
```

**描述**: 注册消费者监听器

**特性**:
- 每个 Surface 只有一个监听器
- 重复注册会覆盖之前的监听器
- 在 Buffer 提交到 dirty 队列时触发回调

**触发位置**: `frameworks/buffer_queue_producer.cpp:245-254`

```cpp
int32_t BufferQueueProducer::EnqueueBuffer(SurfaceBufferImpl& buffer) {
    int32_t ret = bufferQueue_->FlushBuffer(buffer);
    if (ret == 0 && consumerListener_ != nullptr) {
        consumerListener_->OnBufferAvailable();
    }
    return ret;
}
```

#### UnregisterConsumerListener

```cpp
virtual void UnregisterConsumerListener() = 0;
```

**描述**: 注销消费者监听器

---

## 3. SurfaceBuffer 类

### 3.1 类定义

```cpp
// interfaces/kits/surface_buffer.h
class SurfaceBuffer {
public:
    virtual void* GetVirAddr() const = 0;      // 虚拟地址
    virtual uint64_t GetPhyAddr() const = 0;   // 物理地址
    virtual uint32_t GetSize() const = 0;      // 大小
    virtual void SetSize(uint32_t size) = 0;   // 设置大小
    
    // 额外数据 (int32)
    virtual int32_t SetInt32(uint32_t key, int32_t value) = 0;
    virtual int32_t GetInt32(uint32_t key, int32_t& value) = 0;
    
    // 额外数据 (int64)
    virtual int32_t SetInt64(uint32_t key, int64_t value) = 0;
    virtual int32_t GetInt64(uint32_t key, int64_t& value) = 0;
    
protected:
    SurfaceBuffer() {}
    virtual ~SurfaceBuffer() {}
};
```

### 3.2 内存访问

#### GetVirAddr

```cpp
virtual void* GetVirAddr() const = 0;
```

**描述**: 获取 Buffer 的虚拟地址

**使用场景**: CPU 访问像素数据

```cpp
SurfaceBuffer* buffer = surface->RequestBuffer(1);
void* addr = buffer->GetVirAddr();
// 写入像素数据到 addr
```

#### GetPhyAddr

```cpp
virtual uint64_t GetPhyAddr() const = 0;
```

**描述**: 获取 Buffer 的物理地址

**使用场景**: 硬件 DMA 访问

### 3.3 额外数据

**用途**: 存储 Buffer 的附加信息 (时间戳、变换矩阵等)

**存储方式**: `std::map<uint32_t, ExtraData>`

**限制**: 
- 最大条目数: 1000 (`MAX_USER_DATA_COUNT`)
- 单条数据大小: ≤ 8 bytes (sizeof(int64_t))

#### SetInt32 / GetInt32

```cpp
virtual int32_t SetInt32(uint32_t key, int32_t value) = 0;
virtual int32_t GetInt32(uint32_t key, int32_t& value) = 0;
```

**返回值**:
| 值 | 含义 |
|----|------|
| 0 | 成功 |
| -1 | 失败 (key 不存在或类型不匹配) |

---

## 4. IBufferConsumerListener 类

### 4.1 类定义

```cpp
// interfaces/kits/ibuffer_consumer_listener.h
class IBufferConsumerListener {
public:
    virtual void OnBufferAvailable() = 0;
};
```

### 4.2 回调方法

#### OnBufferAvailable

```cpp
virtual void OnBufferAvailable() = 0;
```

**触发时机**: Producer 调用 `FlushBuffer()` 后

**使用示例**:
```cpp
class MyConsumer : public OHOS::IBufferConsumerListener {
public:
    void OnBufferAvailable() override {
        // 有新 Buffer 可消费
        OHOS::SurfaceBuffer* buffer = surface_>AcquireBuffer();
        if (buffer != nullptr) {
            // 处理 Buffer (如合成显示)
            ProcessBuffer(buffer);
            surface_>ReleaseBuffer(buffer);
        }
    }
private:
    OHOS::Surface* surface_;
};
```

**注意事项**:
- 回调在内部锁外执行
- 回调中可安全调用 Surface API

---

## 5. 类型与常量

### 5.1 尺寸限制 (surface_type.h)

```cpp
constexpr uint16_t SURFACE_MAX_WIDTH = 7680;
constexpr uint16_t SURFACE_MAX_HEIGHT = 7680;
constexpr uint16_t SURFACE_MAX_QUEUE_SIZE = 10;
constexpr uint16_t SURFACE_MIN_QUEUE_SIZE = 1;
constexpr uint16_t SURFACE_DEFAULT_QUEUE_SIZE = 1;
constexpr uint16_t SURFACE_MAX_STRIDE_ALIGNMENT = 32;
constexpr uint16_t SURFACE_MIN_STRIDE_ALIGNMENT = 4;
constexpr uint16_t SURFACE_DEFAULT_STRIDE_ALIGNMENT = 4;
#define SURFACE_MAX_SIZE 58982400  // 8K * 8K
```

### 5.2 BufferConsumerUsage 枚举

```cpp
enum BufferConsumerUsage {
    BUFFER_CONSUMER_USAGE_SORTWARE = 0,                    // 虚拟内存
    BUFFER_CONSUMER_USAGE_HARDWARE,                        // 物理内存
    BUFFER_CONSUMER_USAGE_HARDWARE_CONSUMER_CACHE,         // 消费者 Cache
    BUFFER_CONSUMER_USAGE_HARDWARE_PRODUCER_CACHE,         // 生产者 Cache
    BUFFER_CONSUMER_USAGE_MAX
};
```

### 5.3 像素格式 (graphic_utils_lite)

```cpp
enum ImageFormat {
    IMAGE_PIXEL_FORMAT_NONE = 0,
    IMAGE_PIXEL_FORMAT_RGB565,
    IMAGE_PIXEL_FORMAT_ARGB1555,
    IMAGE_PIXEL_FORMAT_RGB888,
    IMAGE_PIXEL_FORMAT_ARGB8888,
    IMAGE_PIXEL_FORMAT_NV12,
    IMAGE_PIXEL_FORMAT_NV21,
    IMAGE_PIXEL_FORMAT_YUV420,
    IMAGE_PIXEL_FORMAT_YVU420,
};
```

---

## 6. 错误码

### 6.1 BufferErrorCode (buffer_common.h)

```cpp
enum BufferErrorCode {
    SURFACE_ERROR_INVALID_PARAM = -10,      // 无效参数
    SURFACE_ERROR_INVALID_REQUEST,          // 无效请求
    SURFACE_ERROR_NOT_READY,                // 未就绪
    SURFACE_ERROR_SYSTEM_ERROR,             // 系统错误
    SURFACE_ERROR_BUFFER_NOT_EXISTED,       // Buffer 不存在
    SURFACE_ERROR_OK = 0,                   // 成功
};
```

### 6.2 错误处理模式

大部分接口使用静默失败模式，记录日志但不抛异常：

```cpp
#define RETURN_IF_FAIL(cond) {          \
    if (!(cond)) {                      \
        GRAPHIC_LOGD("'%s' failed.", #cond);    \
        return;                         \
    }                                   \
}
```

---

## 7. 使用模式

### 7.1 单进程模式

```cpp
// Consumer + Producer 在同一进程
OHOS::Surface* surface = OHOS::Surface::CreateSurface();
surface->SetWidthAndHeight(1920, 1080);
surface->SetFormat(IMAGE_PIXEL_FORMAT_ARGB8888);

// Producer 线程
SurfaceBuffer* buffer = surface->RequestBuffer(1);
// 绘制...
surface->FlushBuffer(buffer);

// Consumer 线程
SurfaceBuffer* buffer = surface->AcquireBuffer();
// 合成...
surface->ReleaseBuffer(buffer);
```

### 7.2 跨进程模式

```cpp
// Consumer 进程
OHOS::Surface* surface = OHOS::Surface::CreateSurface();
// 序列化 sid 传递给 Producer
IpcIo io;
surface->WriteIoIpcIo(io);
// 发送 io 到 Producer 进程...

// Producer 进程 (接收 sid 后)
OHOS::Surface* surface = OHOS::SurfaceImpl::GenericSurfaceByIpcIo(io);
// 使用 surface 进行 Buffer 操作...
```

---

## 8. 线程安全

| 方法 | 线程安全 | 说明 |
|------|----------|------|
| CreateSurface() | ✅ | 线程安全 |
| Set*() / Get*() | ✅ | 内部有锁保护 |
| RequestBuffer() | ✅ | 内部有锁保护 |
| FlushBuffer() | ✅ | 内部有锁保护 |
| AcquireBuffer() | ✅ | 内部有锁保护 |
| ReleaseBuffer() | ✅ | 内部有锁保护 |

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
