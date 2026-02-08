# 内部 API 文档 (Innerkits)

> surface_lite 模块内部接口与实现细节

---

## 1. 概述

### 1.1 文档范围

本文档描述 **surface_lite** 模块的内部实现接口，面向系统开发者和维护人员。

### 1.2 头文件位置

```
interfaces/innerkits/
├── buffer_common.h              # 公共定义和宏
├── buffer_producer.h            # 生产者抽象基类
├── buffer_queue.h               # Buffer 队列核心
├── buffer_queue_consumer.h      # 消费者封装
├── surface_buffer_impl.h        # Buffer 实现
└── surface_impl.h               # Surface 实现
```

---

## 2. SurfaceImpl 类

### 2.1 类定义

```cpp
// interfaces/innerkits/surface_impl.h
class SurfaceImpl : public Surface {
public:
    // 工厂方法
    static Surface* GenericSurfaceByIpcIo(IpcIo& io);
    
    // 构造/析构
    SurfaceImpl();                          // Consumer 端
    ~SurfaceImpl();
    
    // Surface 接口实现...
    
    // IPC 相关
    void WriteIoIpcIo(IpcIo& io);
    int32_t DoIpcMsg(uint32_t code, IpcIo* data, IpcIo* reply, MessageOption option);
    bool Init();

private:
    SurfaceImpl(const SvcIdentity& sid);    // Producer 端
    static int32_t IpcRequestHandler(uint32_t code, IpcIo* data, 
                                      IpcIo* reply, MessageOption option);
    
    SvcIdentity sid_;                       // IPC 服务标识
    IpcObjectStub objectStub_;              // IPC 对象存根
    BufferQueueConsumer* consumer_;         // 消费者端
    BufferProducer* producer_;              // 生产者端
    bool IsConsumer_;                       // 是否为 Consumer 模式
};
```

### 2.2 构造方式

#### Consumer 模式

```cpp
// 位置: frameworks/surface.cpp:20
SurfaceImpl* surface = new SurfaceImpl();  // IsConsumer_ = true
if (surface->Init()) {
    return surface;
}
```

**初始化流程** (frameworks/surface_impl.cpp:54-99):
```cpp
bool SurfaceImpl::Init() {
    // 1. 初始化 BufferManager
    if (!BufferManager::GetInstance()->Init()) return false;
    
    if (IsConsumer_) {
        // 2. 创建 BufferQueue
        BufferQueue* bufferQueue = new BufferQueue();
        if (!bufferQueue->Init()) return false;
        
        // 3. 创建 Producer 和 Consumer
        producer_ = new BufferQueueProducer(bufferQueue);
        consumer_ = new BufferQueueConsumer(*bufferQueue);
        
        // 4. 注册 IPC 服务
        objectStub_.func = IpcRequestHandler;
        objectStub_.args = reinterpret_cast<void*>(producer_);
        sid_.token = SERVICE_TYPE_ANONYMOUS;
        sid_.cookie = reinterpret_cast<uintptr_t>(&objectStub_);
    } else {
        // Producer 模式: 创建 IPC 代理
        producer_ = new BufferClientProducer(sid_);
    }
    return true;
}
```

#### Producer 模式

```cpp
// 位置: frameworks/surface_impl.cpp:273-289
Surface* SurfaceImpl::GenericSurfaceByIpcIo(IpcIo& io) {
    SvcIdentity sid;
    if (ReadRemoteObject(&io, &sid)) {
        SurfaceImpl* surface = new SurfaceImpl(sid);  // IsConsumer_ = false
        if (surface->Init()) {
            return surface;
        }
    }
    return nullptr;
}
```

### 2.3 职责代理

SurfaceImpl 将操作代理给 Producer 或 Consumer:

```cpp
// 设置属性 → 代理给 Producer
void SurfaceImpl::SetWidthAndHeight(uint32_t width, uint32_t height) {
    RETURN_IF_FAIL(producer_ != nullptr);
    producer_->SetWidthAndHeight(width, height);
}

// 获取 Buffer → 代理给 Consumer
SurfaceBuffer* SurfaceImpl::AcquireBuffer() {
    RETURN_VAL_IF_FAIL(consumer_, nullptr);
    return consumer_->AcquireBuffer();
}
```

### 2.4 IPC 处理

**请求处理入口** (frameworks/surface_impl.cpp:259-271):
```cpp
int32_t SurfaceImpl::IpcRequestHandler(uint32_t code, IpcIo* data, 
                                        IpcIo* reply, MessageOption option) {
    // option.args 指向 BufferQueueProducer 实例
    BufferQueueProducer* product = reinterpret_cast<BufferQueueProducer*>(option.args);
    return product->OnIpcMsg(code, data, reply, option);
}
```

---

## 3. SurfaceBufferImpl 类

### 3.1 类定义

```cpp
// interfaces/innerkits/surface_buffer_impl.h
class SurfaceBufferImpl : public SurfaceBuffer {
public:
    SurfaceBufferImpl();
    ~SurfaceBufferImpl();
    
    // Getters / Setters
    int32_t GetKey() const;
    void SetKey(int32_t key);
    uint64_t GetPhyAddr() const override;
    void SetPhyAddr(uint64_t phyAddr);
    int32_t GetStride() const;
    void SetStride(int32_t stride);
    uint32_t GetMaxSize() const;
    void SetMaxSize(uint32_t size);
    uint8_t GetDeletePending() const;
    void SetDeletePending(uint8_t deletePending);
    BufferState GetState() const;
    void SetState(BufferState newState);
    
    // SurfaceBuffer 接口实现
    void* GetVirAddr() const override;
    void SetVirAddr(void* virAddr);
    uint32_t GetSize() const override;
    void SetSize(uint32_t size) override;
    int32_t SetInt32(uint32_t key, int32_t value) override;
    int32_t GetInt32(uint32_t key, int32_t& value) override;
    int32_t SetInt64(uint32_t key, int64_t value) override;
    int32_t GetInt64(uint32_t key, int64_t& value) override;
    
    // IPC 序列化
    void ReadFromIpcIo(IpcIo& io);
    void WriteToIpcIo(IpcIo& io);
    void CopyExtraData(SurfaceBufferImpl& buffer);
    void ClearExtraData();
    
    bool equals(const SurfaceBufferImpl& buffer) const;

private:
    int32_t SetData(uint32_t key, uint8_t type, const void* data, uint8_t size);
    int32_t GetData(uint32_t key, uint8_t* type, void** data, uint8_t* size);
    
    SurfaceBufferData bufferData_;           // 核心数据
    std::map<uint32_t, ExtraData> extDatas_; // 额外数据
    uint32_t len_;                           // 当前使用大小
};
```

### 3.2 核心数据结构

```cpp
// Buffer 状态枚举
enum BufferState {
    BUFFER_STATE_NONE = 0,
    BUFFER_STATE_REQUEST,
    BUFFER_STATE_FLUSH,
    BUFFER_STATE_ACQUIRE,
    BUFFER_STATE_RELEASE
};

// Buffer 句柄 (用于标识)
struct SurfaceBufferHandle {
    int32_t key;              // shm key 或 fd
    uint64_t phyAddr;         // 物理地址
    int32_t stride;           // 行步长
    uint32_t reserveFds;      // 保留 fd 数
    uint32_t reserveInts;     // 保留 int 数
};

// Buffer 完整数据
struct SurfaceBufferData {
    SurfaceBufferHandle handle;
    uint32_t size;            // 分配大小
    uint32_t usage;           // 使用类型
    uint8_t deletePending;    // 待删除标记
    BufferState state;        // 当前状态
    void* virAddr;            // 虚拟地址
};

// 额外数据条目
struct ExtraData {
    void* value;              // 数据指针
    uint8_t size;             // 数据大小
    uint8_t type;             // 数据类型
};
```

### 3.3 IPC 序列化格式

**序列化** (frameworks/surface_buffer_impl.cpp:161-190):
```cpp
void SurfaceBufferImpl::WriteToIpcIo(IpcIo& io) {
    // 基础元数据
    WriteInt32(&io, bufferData_.handle.key);
    WriteUint64(&io, bufferData_.handle.phyAddr);
    WriteUint32(&io, bufferData_.handle.reserveFds);
    WriteUint32(&io, bufferData_.handle.reserveInts);
    WriteUint32(&io, bufferData_.size);
    WriteUint32(&io, bufferData_.usage);
    WriteUint32(&io, len_);
    
    // 额外数据
    WriteUint32(&io, extDatas_.size());
    for (auto iter = extDatas_.begin(); iter != extDatas_.end(); ++iter) {
        WriteUint32(&io, iter->first);  // key
        WriteUint32(&io, iter->second.type);
        switch (iter->second.type) {
            case BUFFER_DATA_TYPE_INT_32:
                WriteInt32(&io, *(int32_t*)iter->second.value);
                break;
            case BUFFER_DATA_TYPE_INT_64:
                WriteInt64(&io, *(int64_t*)iter->second.value);
                break;
        }
    }
}
```

**反序列化** (frameworks/surface_buffer_impl.cpp:125-160):
```cpp
void SurfaceBufferImpl::ReadFromIpcIo(IpcIo& io) {
    ReadInt32(&io, &bufferData_.handle.key);
    ReadUint64(&io, &bufferData_.handle.phyAddr);
    // ... 读取其他字段
    
    uint32_t extDataSize;
    ReadUint32(&io, &extDataSize);
    if (extDataSize > 0 && extDataSize < MAX_USER_DATA_COUNT) {
        for (uint32_t i = 0; i < extDataSize; i++) {
            uint32_t key, type;
            ReadUint32(&io, &key);
            ReadUint32(&io, &type);
            switch (type) {
                case BUFFER_DATA_TYPE_INT_32:
                    int32_t value32;
                    ReadInt32(&io, &value32);
                    SetInt32(key, value32);
                    break;
                // ...
            }
        }
    }
}
```

---

## 4. BufferQueue 类

### 4.1 类定义

```cpp
// interfaces/innerkits/buffer_queue.h
class BufferQueue {
public:
    BufferQueue();
    ~BufferQueue();
    
    bool Init();
    
    // Buffer 操作
    SurfaceBufferImpl* RequestBuffer(uint8_t wait);
    int32_t FlushBuffer(SurfaceBufferImpl& buffer);
    SurfaceBufferImpl* AcquireBuffer();
    bool ReleaseBuffer(const SurfaceBufferImpl& buffer);
    int32_t CancelBuffer(const SurfaceBufferImpl& buffer);
    
    // 属性设置
    void SetQueueSize(uint8_t queueSize);
    uint8_t GetQueueSize();
    void SetWidthAndHeight(uint32_t width, uint32_t height);
    int32_t GetWidth();
    int32_t GetHeight();
    void SetFormat(uint32_t format);
    int32_t GetFormat();
    void SetStrideAlignment(uint32_t stride);
    int32_t GetStrideAlignment();
    int32_t GetStride();
    void SetSize(uint32_t size);
    int32_t GetSize();
    void SetUsage(uint32_t usage);
    int32_t GetUsage();
    void SetUserData(const std::string& key, const std::string& value);
    std::string GetUserData(const std::string& key);

private:
    bool CanRequest(uint8_t wait);
    int32_t isValidAttr(uint32_t width, uint32_t height, 
                        uint32_t format, uint32_t strideAlignment);
    int32_t Reset(uint32_t size = 0);
    void NeedAttach();
    void Detach(SurfaceBufferImpl* buffer);
    SurfaceBufferImpl* GetBuffer(const SurfaceBufferImpl& buffer);
    int32_t ReleaseBuffer(const SurfaceBufferImpl& buffer, BufferState state);
    
    // 属性
    uint32_t width_, height_, format_, stride_, usage_, size_;
    uint8_t queueSize_;
    uint32_t strideAlignment_;
    uint8_t attachCount_;
    bool customSize_;
    
    // Buffer 列表
    std::list<SurfaceBufferImpl *> freeList_;   // 空闲队列
    std::list<SurfaceBufferImpl *> dirtyList_;  // 待消费队列
    std::list<SurfaceBufferImpl *> allBuffers_; // 所有 Buffer
    
    // 同步原语
    pthread_mutex_t lock_;
    pthread_cond_t freeCond_;
    
    // 用户数据
    std::map<std::string, std::string> usrDataMap_;
};
```

### 4.2 Buffer 列表管理

```
┌─────────────────────────────────────────────────────────────────┐
│                     BufferQueue 内存结构                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────┐                                          │
│   │    allBuffers_  │───┐ 包含所有分配的 Buffer                  │
│   │  (list)         │   │                                      │
│   └─────────────────┘   │                                      │
│           │             │                                      │
│           ▼             │                                      │
│   ┌─────────────────┐   │    ┌─────────────────┐               │
│   │   freeList_     │   └───▶│ SurfaceBufferImpl │               │
│   │  (可用 Buffer)   │        │  - state         │               │
│   └─────────────────┘        │  - virAddr       │               │
│                              │  - size          │               │
│   ┌─────────────────┐        │  - usage         │               │
│   │   dirtyList_    │───────▶│  - extDatas_     │               │
│   │ (待消费 Buffer)  │        └─────────────────┘               │
│   └─────────────────┘                                          │
│                                                                 │
│   Buffer 状态转换:                                               │
│   freeList_ ──Request──▶ REQUEST ──Flush──▶ dirtyList_         │
│       ▲                                                  │      │
│       └────────────────Release───────────────────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 核心方法实现

#### RequestBuffer

```cpp
// frameworks/buffer_queue.cpp:132-150
SurfaceBufferImpl* BufferQueue::RequestBuffer(uint8_t wait) {
    SurfaceBufferImpl *buffer = nullptr;
    pthread_mutex_lock(&lock_);
    
    if (!CanRequest(wait)) {
        goto ERROR;  // 无可用 Buffer 且不等待
    }
    
    buffer = freeList_.front();
    freeList_.pop_front();
    buffer->SetState(BUFFER_STATE_REQUEST);
    
ERROR:
    pthread_mutex_unlock(&lock_);
    return buffer;
}
```

#### CanRequest

```cpp
// frameworks/buffer_queue.cpp:108-130
bool BufferQueue::CanRequest(uint8_t wait) {
    if (!freeList_.empty()) {
        return true;  // 有空闲 Buffer
    }
    
    if (attachCount_ < queueSize_) {
        NeedAttach();  // 分配新 Buffer
        return !freeList_.empty();
    }
    
    if (wait) {
        pthread_cond_wait(&freeCond_, &lock_);  // 阻塞等待
        return true;
    }
    
    return false;
}
```

#### NeedAttach (Buffer 分配)

```cpp
// frameworks/buffer_queue.cpp:79-106
void BufferQueue::NeedAttach() {
    if (queueSize_ == attachCount_) return;  // 已达上限
    
    BufferManager* bufferManager = BufferManager::GetInstance();
    SurfaceBufferImpl* buffer = nullptr;
    
    if (size_ != 0 && customSize_) {
        buffer = bufferManager->AllocBuffer(size_, usage_);
    } else {
        buffer = bufferManager->AllocBuffer(width_, height_, format_, usage_);
    }
    
    if (buffer != nullptr) {
        size_ = buffer->GetSize();
        stride_ = buffer->GetStride();
        attachCount_++;
        freeList_.push_back(buffer);
        allBuffers_.push_back(buffer);
    }
}
```

---

## 5. BufferManager 单例

### 5.1 类定义

```cpp
// frameworks/buffer_manager.h
class BufferManager {
public:
    static BufferManager* GetInstance();
    bool Init();
    
    // Buffer 分配
    SurfaceBufferImpl* AllocBuffer(uint32_t size, uint32_t usage);
    SurfaceBufferImpl* AllocBuffer(uint32_t width, uint32_t height, 
                                    uint32_t format, uint32_t usage);
    void FreeBuffer(SurfaceBufferImpl** buffer);
    
    // 内存映射
    bool MapBuffer(SurfaceBufferImpl& buffer) const;
    void UnmapBuffer(SurfaceBufferImpl& buffer) const;
    
    // Cache 刷新
    int32_t FlushCache(SurfaceBufferImpl& buffer) const;

private:
    BufferManager() : grallocFucs_(nullptr) {}
    ~BufferManager() {}
    
    BufferHandle* AllocateBufferHandle(SurfaceBufferImpl& buffer) const;
    SurfaceBufferImpl* AllocBuffer(AllocInfo info);
    bool ConvertUsage(uint64_t& destUsage, uint32_t srcUsage) const;
    bool ConvertFormat(PixelFormat& destFormat, uint32_t srcFormat) const;
    
    GrallocFuncs* grallocFucs_;  // 底层分配器
    
    struct BufferKey {
        int32_t key;
        uint64_t phyAddr;
        bool operator < (const BufferKey &x) const {
            return (key < x.key) || (key == x.key && phyAddr < x.phyAddr);
        }
    };
    std::map<BufferKey, BufferHandle*> bufferHandleMap_;
};
```

### 5.2 单例实现

```cpp
// frameworks/buffer_manager.cpp:23-27
BufferManager* BufferManager::GetInstance() {
    static BufferManager instance;  // C++11 线程安全
    return &instance;
}
```

### 5.3 Gralloc 接口

```cpp
// frameworks/buffer_manager.cpp:29-39
bool BufferManager::Init() {
    if (grallocFucs_ != nullptr) return true;  // 已初始化
    
    if (GrallocInitialize(&grallocFucs_) != DISPLAY_SUCCESS) {
        return false;
    }
    return true;
}
```

**GrallocFuncs 接口**:
```cpp
typedef struct {
    int32_t (*AllocMem)(AllocInfo*, BufferHandle**);
    void (*FreeMem)(BufferHandle*);
    void* (*Mmap)(BufferHandle*);
    void* (*MmapCache)(BufferHandle*);
    int32_t (*Unmap)(BufferHandle*);
    int32_t (*FlushCache)(BufferHandle*);
    int32_t (*FlushMCache)(BufferHandle*);
} GrallocFuncs;
```

---

## 6. IPC 请求处理

### 6.1 请求码定义

```cpp
// interfaces/innerkits/buffer_producer.h:26-47
typedef enum {
    REQUEST_BUFFER = 0,
    FLUSH_BUFFER,
    CANCEL_BUFFER,
    SET_QUEUE_SIZE,
    GET_QUEUE_SIZE,
    SET_WIDTH_AND_HEIGHT,
    GET_WIDTH,
    GET_HEIGHT,
    SET_FORMAT,
    GET_FORMAT,
    SET_STRIDE_ALIGNMENT,
    GET_STRIDE_ALIGNMENT,
    GET_STRIDE,
    SET_SIZE,
    GET_SIZE,
    SET_USAGE,
    GET_USAGE,
    SET_USER_DATA,
    GET_USER_DATA,
    MAX_REQUEST_CODE,  // = 20
} SURFACE_REQUEST_CODE;
```

### 6.2 处理函数表

```cpp
// frameworks/buffer_queue_producer.cpp:202-222
typedef int32_t (*IpcMsgHandle)(BufferQueueProducer* product, IpcIo *io, IpcIo *reply);

static IpcMsgHandle g_ipcMsgHandleList[] = {
    OnRequestBuffer,      // 0
    OnFlushBuffer,        // 1
    OnCancelBuffer,       // 2
    OnSetQueueSize,       // 3
    OnGetQueueSize,       // 4
    OnSetWidthAndHeight,  // 5
    OnGetWidth,           // 6
    OnGetHeight,          // 7
    OnSetFormat,          // 8
    OnGetFormat,          // 9
    OnSetStrideAlignment, // 10
    GetStrideAlignment,   // 11
    OnGetStride,          // 12
    OnSetSize,            // 13
    OnGetSize,            // 14
    OnSetUsage,           // 15
    OnGetUsage,           // 16
    OnSetUserData,        // 17
    OnGetUserData,        // 18
};
```

### 6.3 消息分发

```cpp
// frameworks/buffer_queue_producer.cpp:373-385
int32_t BufferQueueProducer::OnIpcMsg(uint32_t code, IpcIo *data, 
                                       IpcIo *reply, MessageOption option) {
    if (data == NULL) {
        return SURFACE_ERROR_INVALID_PARAM;
    }
    
    if (code >= MAX_REQUEST_CODE) {
        return SURFACE_ERROR_INVALID_REQUEST;
    }
    
    return g_ipcMsgHandleList[code](this, data, reply);
}
```

---

## 7. 工具宏

### 7.1 错误检查宏

```cpp
// interfaces/innerkits/buffer_common.h:22-34
#define RETURN_VAL_IF_FAIL(cond, val) { \
    if (!(cond)) {                      \
        GRAPHIC_LOGD("'%s' failed.", #cond);    \
        return val;                     \
    }                                   \
}

#define RETURN_IF_FAIL(cond) {          \
    if (!(cond)) {                      \
        GRAPHIC_LOGD("'%s' failed.", #cond);    \
        return;                         \
    }                                   \
}
```

### 7.2 使用示例

```cpp
void SurfaceImpl::SetSize(uint32_t size) {
    RETURN_IF_FAIL(producer_);
    RETURN_IF_FAIL(size > 0 && size < SURFACE_MAX_SIZE);
    producer_->SetSize(size);
}

uint32_t SurfaceImpl::GetSize() {
    RETURN_VAL_IF_FAIL(producer_, 0);
    return producer_->GetSize();
}
```

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
