# 关键调用链

## 目的
本文档说明 graphic_surface 的关键代码执行路径和调用链。

## 适用范围
面向需要：
- 理解代码执行流程的开发者
- 进行性能分析的性能工程师
- 调试复杂问题的工程师
- 学习系统行为的初学者

## 调用链分类

### 1. Surface 创建与连接流程

#### 流程概述
```
应用（UI 框架）
  ├─► IConsumerSurface::Create()
  │     └─► ConsumerSurface::ConsumerSurface()
  │           └─► BufferQueue::BufferQueue()
  │           └─► BufferQueueProducer::BufferQueueProducer()
  │
  ├─► ConsumerSurface::GetProducer()
  │     └─► 返回 sptr<IBufferProducer> (BufferQueueProducer)
  │
  └─► 传递 IBufferProducer 给应用（通过 Binder）

应用（生产者）
  └─► Surface::CreateSurfaceAsProducer(IBufferProducer)
        └─► ProducerSurface::ProducerSurface(producer)
```

#### 详细调用链

**消费者创建**：
```cpp
// 1. 创建消费者 Surface（WindowManager）
// surface/src/consumer_surface.cpp:ConsumerSurface::ConsumerSurface()
sptr<IConsumerSurface> consumer = IConsumerSurface::Create();
  └─► new ConsumerSurface()
        ├─► CreateSurface(name)  // 创建 BufferQueue 名称
        ├─► new BufferQueue(name)
        │     ├─► new SurfaceBufferImpl[]
        │     ├─► new BufferElement[]
        │     └─► new BufferQueueProducer(this)
        └─► new BufferQueueConsumer(this)

// 2. 获取生产者接口
sptr<IBufferProducer> producer = consumer->GetProducer();
  └─► consumer->bufferQueueProducer_  (BufferQueueProducer)
```

**生产者连接**：
```cpp
// surface/src/buffer_queue_producer.cpp:BufferQueueProducer::Connect()
GSError Connect(listener, producerApi) {
  └─► CheckConnectLocked()
        └─► CheckIsAlive()
              └─► CheckConnectLocked()
                    ├─► if (connectedPid_ == 0) return GSERROR_NO_CONSUMER
                    └─► if (connectedPid_ != GetCallingPid()) return CONSUMER_IS_CONNECTED

  └─► connectedPid_ = GetCallingPid()
  └─► token_ = listener->AsObject()
  └─► HandleDeathRecipient()
        └─► token_->AddDeathRecipient(producerSurfaceDeathRecipient_)
}
```

---

### 2. Buffer Request 流程

#### 流程概述
```
生产者应用
  ├─► ProducerSurface::RequestBuffer(config, fence)
  │     └─► bufferProducerCache_[config] = RequestBuffer()
  │           └─► BufferClientProducer::RequestBuffer(config, fence)
  │                 └─► SendRequest(BUFFER_PRODUCER_REQUEST_BUFFER)
  │
  └──────────────────────► Binder IPC ──────────────────────►
                          │
                          ▼
                        BufferQueue（消费者进程）
  ├─► BufferQueueProducer::OnRemoteRequest()
  │     └─► memberFuncMap_[code](data, reply)
  │           └─► RequestBufferLocked(data, reply)
  │                 ├─► CheckConnectLocked()
  │                 ├─► GetFreeBufferLocked(config)
  │                 │     ├─► 遍历 freeList_
  │                 │     │     ├─► buffer->Matches(config)
  │                 │     │     │     └─► 找到匹配 Buffer
  │                 │     │     └─► return buffer
  │                 │     └─► if (!found) {
  │                 │           ├─► if (bufferQueueCache_.size() < queueSize_)
  │                 │           │     └─► AllocBufferLocked(config)
  │                 │           │           └─► SurfaceBuffer::Alloc(config)
  │                 │           │                 └─► GraphicBufferMapper::Map()
  │                 │           │                       └─► gralloc_mapper_interface
  │                 │           └─► PushToQueueCacheLocked(newBuffer)
  │                 │           └─► return newBuffer
  │                 │           └─► GSERROR_NO_BUFFER
  │                 │     }
  │                 │     └─► if (requestBufferNoBlockMode_) {
  │                 │           └─► return GSERROR_NO_BUFFER
  │                 │     } else {
  │                 │           └─► waitReqCon_.wait(lock)
  │                 │     }
  │                 └─► buffer->SetState(BUFFER_STATE_REQUESTED)
  │                 └─► SetFence(acquireFence)
  │                 └─► reply.WriteBuffer(buffer)
  │
  └──────────────────────► Binder IPC ──────────────────────►
                          │
                          ▼
                      生产者应用
  └─► ProducerSurface::RequestBuffer() 返回
        └─► bufferProducerCache_[buffer] = buffer
        └─► return GSERROR_OK
```

#### 关键代码位置

**生产者请求入口**：
- `surface/src/producer_surface.cpp:ProducerSurface::RequestBuffer()` - Line ~250

**IPC 传输**：
- `surface/src/buffer_client_producer.cpp:BufferClientProducer::SendRequest()` - Line ~100

**消费者请求处理**：
- `surface/src/buffer_queue_producer.cpp:BufferQueueProducer::RequestBufferLocked()` - Line ~450

**Buffer 分配**：
- `surface/src/buffer_queue.cpp:BufferQueue::AllocBufferLocked()` - Line ~800

**Gralloc 映射**：
- `surface/src/surface_buffer_impl.cpp:SurfaceBufferImpl::Alloc()` - Line ~150

---

### 3. Buffer Flush 流程

#### 流程概述
```
生产者应用
  ├─► ProducerSurface::FlushBuffer(buffer, fence, config)
  │     └─► CheckBufferStatusLocked(buffer)
  │           └─► CheckIsAlive()
  │
  ├─► buffer->SetTimestamp(config.timestamp)
  ├─► buffer->SetDamage(config.damages)
  ├─► buffer->SetFence(fence)  // ReleaseFence
  ├─► buffer->SetReleaseFence(fence)
  ├─► buffer->SetSurfaceSourceType(config.sourceType)
  ├─► buffer->SetTransform(config.transform)
  └─► buffer->SetScalingMode(config.scalingMode)
  │
  ├─► bufferProducerCache_.FlushBuffer(buffer)
  │     └─► bufferProducer_->FlushBuffer(buffer, fence, config)
  │           └─► SendRequest(BUFFER_PRODUCER_FLUSH_BUFFER)
  │
  └──────────────────────► Binder IPC ──────────────────────►
                          │
                          ▼
                        BufferQueue（消费者进程）
  ├─► BufferQueueProducer::OnRemoteRequest()
  │     └─► memberFuncMap_[BUFFER_PRODUCER_FLUSH_BUFFER](data, reply)
  │           └─► FlushBufferLocked(data, reply)
  │                 ├─► CheckConnectLocked()
  │                 ├─► GetBufferElementLocked(buffer)
  │                 │     └─► bufferQueueCache_[buffer->GetSeqNum()]
  │                 ├─► buffer->SetState(BUFFER_STATE_FLUSHED)
  │                 ├─► buffer->SetTimestamp(timestamp)
  │                 ├─► buffer->SetFence(fence)
  │                 ├─► buffer->SetSurfaceSourceType(sourceType)
  │                 ├─► buffer->SetTransform(transform)
  │                 ├─► buffer->SetScalingMode(scalingMode)
  │                 ├─► PushToDirtyListLocked(buffer)
  │                 │     └─► dirtyList_.push_back(bufferElement)
  │                 ├─► NotifyBufferAvailable()  // 如果是第一个 dirty buffer
  │                 │     └─► listener_->OnBufferAvailable()
  │                 └─► reply.WriteInt32(GSERROR_OK)
  │
  └──────────────────────► Binder IPC ──────────────────────►
                          │
                          ▼
                      生产者应用
  └─► ProducerSurface::FlushBuffer() 返回
        └─► RemoveFromBufferProducerCache(buffer)
        └─► return GSERROR_OK
```

#### 关键代码位置

**生产者 Flush 入口**：
- `surface/src/producer_surface.cpp:ProducerSurface::FlushBuffer()` - Line ~300

**消费者 Flush 处理**：
- `surface/src/buffer_queue_producer.cpp:BufferQueueProducer::FlushBufferLocked()` - Line ~550

**队列操作**：
- `surface/src/buffer_queue.cpp:BufferQueue::PushToDirtyListLocked()` - Line ~950

**Listener 回调**：
- `surface/include/buffer_queue.h:IBufferConsumerListener::OnBufferAvailable()` - Line ~30

---

### 4. Buffer Acquire 流程

#### 流程概述
```
消费者应用（WindowManager/RS）
  ├─► ConsumerSurface::AcquireBuffer(buffer, fence, timestamp, damage)
  │     └─► bufferQueueConsumer_->AcquireBuffer(buffer, fence, timestamp, damage)
  │           └─► BufferQueueConsumer::AcquireBufferLocked()
  │                 ├─► CheckIsAlive()
  │                 ├─► PopFromDirtyListLocked()
  │                 │     └─► dirtyList_.pop_front()
  │                 │     └─► bufferElement = element
  │                 ├─► bufferElement->buffer->SetState(BUFFER_STATE_ACQUIRED)
  │                 ├─► bufferElement->buffer->SetAcquireFence(fence)
  │                 ├─► *buffer = bufferElement->buffer
  │                 ├─► *acquireFence = bufferElement->fence
  │                 └─► return GSERROR_OK
  │
  └─► return GSERROR_OK
```

#### 关键代码位置

**消费者 Acquire 入口**：
- `surface/src/consumer_surface.cpp:ConsumerSurface::AcquireBuffer()` - Line ~200

**队列弹出**：
- `surface/src/buffer_queue_consumer.cpp:BufferQueueConsumer::AcquireBufferLocked()` - Line ~150

**Dirty 队列操作**：
- `surface/src/buffer_queue.cpp:BufferQueue::PopFromDirtyListLocked()` - Line ~700

---

### 5. Buffer Release 流程

#### 流程概述
```
消费者应用（WindowManager/RS）
  ├─► ConsumerSurface::ReleaseBuffer(buffer, fence)
  │     └─► bufferQueueConsumer_->ReleaseBuffer(buffer, fence)
  │           └─► BufferQueueConsumer::ReleaseBufferLocked()
  │                 ├─► CheckIsAlive()
  │                 ├─► GetBufferElementLocked(buffer)
  │                 │     └─► bufferQueueCache_[buffer->GetSeqNum()]
  │                 ├─► bufferElement->buffer->SetState(BUFFER_STATE_RELEASED)
  │                 ├─► bufferElement->fence = fence
  │                 ├─► bufferElement->buffer->SetReleaseFence(fence)
  │                 ├─► PushToFreeListLocked(bufferElement)
  │                 │     └─► freeList_.push_back(bufferElement)
  │                 ├─► NotifyBufferReleased()  // 如果队列为空
  │                 │     └─► producerListener_->OnBufferReleasedWithFence(fence)
  │                 └─► waitReqCon_.notify_all()  // 唤醒等待的生产者
  │
  └─► return GSERROR_OK
```

#### 关键代码位置

**消费者 Release 入口**：
- `surface/src/consumer_surface.cpp:ConsumerSurface::ReleaseBuffer()` - Line ~250

**队列释放操作**：
- `surface/src/buffer_queue_consumer.cpp:BufferQueueConsumer::ReleaseBufferLocked()` - Line ~200

**Free 队列操作**：
- `surface/src/buffer_queue.cpp:BufferQueue::PushToFreeListLocked()` - Line ~650

**Producer 回调**：
- `surface/include/ibuffer_producer_listener.h:IProducerListener::OnBufferReleasedWithFence()` - Line ~30

---

### 6. 进程死亡处理流程

#### 流程概述
```
生产者进程崩溃
  │
  ├─► IPC Binder 检测到死亡
  │
  ▼
消费者进程（WindowManager）
  ├─► ProducerSurfaceDeathRecipient::OnRemoteDied(remoteObject)
  │     └─► BufferQueueProducer::HandleDeathRecipient(token)
  │           ├─► token_->RemoveDeathRecipient(producerSurfaceDeathRecipient_)
  │           ├─► token_ = nullptr
  │           ├─► connectedPid_ = 0
  │           ├─► producerListener_ = nullptr
  │           ├─► DisconnectAllProducersLocked()
  │           ├─► CleanCache(true)  // 强制清理所有 Buffer
  │           │     └─► RemoveBufferFromCacheLocked() for all buffers
  │           │     └─► SurfaceBufferImpl::Free()
  │           │     └─► GraphicBufferMapper::Unmap()
  │           └─► return true
```

#### 关键代码位置

**死亡监听器接口**：
- `surface/src/buffer_queue_producer.cpp:ProducerSurfaceDeathRecipient::OnRemoteDied()` - Line ~1750

**死亡处理**：
- `surface/src/buffer_queue_producer.cpp:BufferQueueProducer::HandleDeathRecipient()` - Line ~1595

**缓存清理**：
- `surface/src/buffer_queue.cpp:BufferQueue::CleanCacheLocked()` - Line ~1100

**Buffer 释放**：
- `surface/src/surface_buffer_impl.cpp:SurfaceBufferImpl::Free()` - Line ~300

---

### 7. Buffer 生命周期状态机

#### 状态转换图

```
   REQUESTED                FLUSHED                ACQUIRED
  ┌──────────┐          ┌──────────┐          ┌──────────┐
  │          │          │          │          │          │
  ▼          │          ▼          │          ▼          │
RELEASED ─────► RequestBuffer() ─────► FlushBuffer() ◄─────► AcquireBuffer()
  │          │          │          │          │          │
  │          │          │          │          │          │
  │          └──────────┴──────────┘          └──────────┘
  │
  └────────────► ReleaseBuffer()
```

#### 状态转换表

| 当前状态 | 操作 | 下一状态 | 条件 | 位置 |
|---------|------|---------|------|--------|
| RELEASED | RequestBuffer() | REQUESTED | Buffer 从 freeList_ 取出 |
| REQUESTED | FlushBuffer() | FLUSHED | Buffer 提交到 dirtyList_ |
| REQUESTED | CancelBuffer() | RELEASED | Buffer 返回到 freeList_ |
| FLUSHED | AcquireBuffer() | ACQUIRED | Buffer 从 dirtyList_ 取出 |
| ACQUIRED | ReleaseBuffer() | RELEASED | Buffer 返回到 freeList_ |

#### 状态检查代码
```cpp
// surface/src/buffer_queue.cpp:BufferQueue::CheckBufferStateLocked()
GSError CheckBufferStateLocked(sptr<SurfaceBuffer>& buffer, BufferState expected)
{
    BufferState state = buffer->GetState();
    if (state != expected) {
        BLOGE("Invalid buffer state: expected=%d, actual=%d", expected, state);
        return GSERROR_BUFFER_STATE_INVALID;
    }
    return GSERROR_OK;
}
```

---

### 8. IPC 序列化/反序列化流程

#### BufferHandle 序列化（Producer → Consumer）

**流程**：
```
Producer FlushBuffer()
  │
  ├─► SurfaceBufferImpl::WriteToMessageParcel(reply)
  │     └─► WriteBufferHandle(parcel, buffer->GetBufferHandle())
  │           └─► buffer_handle/src/buffer_handle.cpp:WriteBufferHandle()
  │                 ├─► WriteUint32(handle.width)
  │                 ├─► WriteUint32(handle.height)
  │                 ├─► WriteUint32(handle.stride)
  │                 ├─► WriteUint32(handle.size)
  │                 ├─► WriteInt32(handle.format)
  │                 ├─► WriteUint64(handle.usage)
  │                 ├─► WriteInt32(handle.fd)
  │                 ├─► WriteFileDescriptor(handle.fd)  // fd passing
  │                 ├─► WriteUint32(handle.reserveFds)
  │                 └─► WriteUint32(handle.reserveInts)
  │
  └─► SendRequest(BUFFER_PRODUCER_FLUSH_BUFFER, reply)
```

#### 关键代码位置

**Buffer 序列化**：
- `surface/src/surface_buffer_impl.cpp:SurfaceBufferImpl::WriteToMessageParcel()` - Line ~350

**BufferHandle 序列化**：
- `buffer_handle/src/buffer_handle.cpp:WriteBufferHandle()` - Line ~50

**IPC 传输**：
- `surface/src/buffer_client_producer.cpp:BufferClientProducer::SendRequest()` - Line ~120

---

### 9. SyncFence 同步流程

#### GPU 完成同步

**流程**：
```
Producer 渲染
  │
  ├─► GPU Render(buffer)
  │
  ├─► GPU HAL::CompleteRender(buffer, fence)
  │     └─► Signal Linux dma_fence
  │
  ▼
Producer 设置 AcquireFence
  │
  ├─► SurfaceBufferImpl::SetAcquireFence(gpuFence)
  │     └─► acquireFence_ = new SyncFence(gpuFence)
  │
  ├─► FlushBuffer(buffer, gpuFence)
  │
  └──────────────────────► Binder IPC ──────────────────────►
                          │
                          ▼
Consumer AcquireBuffer()
  │
  ├─► AcquireBuffer(buffer, fence, ...)
  │     └─► acquireFence_ = fence
  │
  ├─► acquireFence->Wait(timeout)
  │     └─► sync_fence/src/sync_fence.cpp:SyncFence::Wait()
  │           └─► fence_->Wait(timeout)
  │                 ├─► fence_->SyncWait()  // Linux dma_fence_wait
  │                 └─► return GSERROR_OK 或超时错误
  │
  ├─► GPU 已完成，安全读取 Buffer
  │
  ▼
Consumer 合成
  │
  ├─► Composite(buffer)
  │
  ├─► Display HAL::CompleteDisplay(buffer, fence)
  │     └─► Signal Linux dma_fence
  │
  ▼
Consumer 设置 ReleaseFence
  │
  ├─► ReleaseBuffer(buffer, displayFence)
  │     └─► ReleaseBuffer(buffer, displayFence)
  │
  └──────────────────────► Binder IPC ──────────────────────►
                          │
                          ▼
Producer（下一帧）
  │
  ├─► GetReleaseFence() = displayFence
  │     └─► displayFence->Wait(timeout)
  │     └─► 等待显示完成后再请求下一帧
  │
  ▼
Producer RequestBuffer()  // 下一帧
```

#### 关键代码位置

**Fence 创建**：
- `sync_fence/src/sync_fence.cpp:SyncFence::Create()` - Line ~100

**Fence 等待**：
- `sync_fence/src/sync_fence.cpp:SyncFence::Wait()` - Line ~150

**Fence 信号**：
- `sync_fence/src/native_fence.cpp:NativeFenceSignal()` - Line ~200

**Linux dma_fence 接口**：
- `sys/cdefs.h` - dma_fence_wait, dma_fence_signal, etc.

---

### 10. Buffer 分配与释放流程

#### Buffer 分配

**流程**：
```
BufferQueue::AllocBufferLocked(config)
  │
  ├─► SurfaceBufferImpl::Alloc(config)
  │     └─► GraphicBufferMapper::Map(width, height, format, usage)
  │           └─► GraphicBufferMapper::RequestBuffer(width, height, format, usage)
  │                 ├─► Gralloc HAL 分配
  │                 │     └─► gralloc_alloc(width, height, format, usage)
  │                 │           └─► 返回 shared_memory fd
  │                 │           └─► 返回 phy_addr（物理地址）
  │                 ├─► new SurfaceBufferImpl(fd, phy_addr, config)
  │                 ├─► buffer->SetState(BUFFER_STATE_REQUESTED)
  │                 ├─► buffer->SetSeqNum(nextSeqNum_++)
  │                 └─► return buffer
  │
  └─► PushToQueueCacheLocked(buffer)
  │     └─► bufferQueueCache_[seqNum] = buffer
```

#### Buffer 释放

**流程**：
```
BufferQueue::CleanCacheLocked(cleanAll)
  │
  ├─► for each buffer in bufferQueueCache_
  │     │
  │     ├─► SurfaceBufferImpl::Free()
  │     │     └─► GraphicBufferMapper::Unmap(buffer->GetPhyAddr())
  │     │           └─► gralloc_free(buffer->GetFd())
  │     │
  │     └─► RemoveBufferFromCacheLocked(buffer)
  │
  └─► bufferQueueCache_.clear()
```

#### 关键代码位置

**Buffer 分配**：
- `surface/src/surface_buffer_impl.cpp:SurfaceBufferImpl::Alloc()` - Line ~150

**Buffer 释放**：
- `surface/src/surface_buffer_impl.cpp:SurfaceBufferImpl::Free()` - Line ~300

**Gralloc 接口**：
- `interfaces/inner_api/buffer_handle/buffer_handle.h` - GraphicBufferMapper（声明）

---

## 调试与追踪

### 添加追踪点到调用链

在关键函数入口添加日志：

```cpp
// surface/src/producer_surface.cpp
GSError ProducerSurface::RequestBuffer(sptr<SurfaceBuffer>& buffer,
                                     const BufferRequestConfig& config,
                                     sptr<SyncFence>& acquireFence)
{
    SURFACE_TRACE_FUNC();  // 函数入口追踪
    BLOGI("RequestBuffer: config=%dx%d, usage=%lu",
            config.width, config.height, config.usage);

    // ... 函数逻辑 ...

    BLOGI("RequestBuffer: return=%d, seq=%d", ret, buffer->GetSeqNum());
    return ret;
}
```

### 使用 hitrace 追踪调用链

```bash
# 启用 Surface 追踪
hdc shell hitrace --surface

# 追踪 10 秒的调用
hdc shell hitrace -t 10 --surface frame_report

# 查看追踪结果
hdc shell hitrace --dump
```

## 相关跳转
- [架构说明](02_Architecture.md) - 组件设计与数据流
- [对外 API](03_External_API.md) - API 使用示例
- [常见问题](08_Troubleshooting.md) - 问题定位与调试
- [内部 API](04_Internal_API.md) - 模块间接口
