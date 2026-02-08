# 问题排查指南

> surface_lite 常见问题与调试方法

---

## 1. 常见问题速查

### 1.1 Buffer 申请失败

**现象**: `RequestBuffer()` 返回 `nullptr`

**可能原因与排查**:

| 原因 | 检查方法 | 解决 |
|------|----------|------|
| 队列已满 | 检查 `GetQueueSize()` 和已分配 Buffer 数 | 增大队列大小或释放 Buffer |
| 参数未设置 | 检查 `GetWidth()`/`GetHeight()` 是否为 0 | 先调用 `SetWidthAndHeight()` |
| 内存不足 | 查看系统内存 `/proc/meminfo` | 释放其他应用内存 |
| Gralloc 失败 | 检查日志 "Alloc graphic buffer failed" | 检查显示驱动 |

**调试代码**:
```cpp
SurfaceBuffer* buffer = surface->RequestBuffer(1);
if (buffer == nullptr) {
    // 打印调试信息
    printf("QueueSize: %d\n", surface->GetQueueSize());
    printf("Width: %d, Height: %d\n", surface->GetWidth(), surface->GetHeight());
    printf("Format: %d\n", surface->GetFormat());
}
```

---

### 1.2 FlushBuffer 失败

**现象**: `FlushBuffer()` 返回非 0

**错误码含义**:

| 返回值 | 含义 | 排查 |
|--------|------|------|
| -1 | 通用错误 | 检查参数是否为有效 Buffer 指针 |
| `SURFACE_ERROR_BUFFER_NOT_EXISTED` | Buffer 不存在 | Buffer 可能已被释放或非本 Surface 的 Buffer |
| `SURFACE_ERROR_INVALID_PARAM` | 参数错误 | Buffer 状态不是 BUFFER_STATE_REQUEST |

**调试方法**:
```cpp
// 检查 Buffer 状态
SurfaceBufferImpl* impl = reinterpret_cast<SurfaceBufferImpl*>(buffer);
// 需要在 SurfaceBufferImpl 中添加调试接口获取状态
```

---

### 1.3 跨进程连接失败

**现象**: Producer 无法连接到 Consumer 的 Surface

**排查步骤**:

1. **检查 IPC 服务是否注册**
   ```cpp
   // Consumer 端检查 sid 是否有效
   surface->WriteIoIpcIo(io);
   // 检查 io 中的 sid 是否非零
   ```

2. **检查 sid 传递是否正确**
   ```cpp
   // 确保 sid 在进程间正确传递
   SvcIdentity sid;
   if (!ReadRemoteObject(&io, &sid)) {
       printf("Read sid failed\n");
   }
   ```

3. **检查权限**
   - 确保两进程有 IPC 通信权限
   - 检查 SELinux/访问控制策略

---

### 1.4 内存泄漏

**现象**: 进程内存持续增长

**常见泄漏点**:

| 泄漏场景 | 原因 | 修复 |
|----------|------|------|
| Buffer 未 Release | Consumer 未调用 `ReleaseBuffer()` | 确保消费后释放 |
| Cancel 未调用 | Producer 未 Cancel 未使用的 Buffer | 异常路径添加 Cancel |
| Surface 未删除 | 未 `delete surface` | 确保析构 |
| Map 未 Unmap | 跨进程 Buffer 未 Unmap | `FlushBuffer` 会自动 Unmap |

**检测方法**:
```cpp
// 定期打印 Buffer 统计
// 在 BufferQueue 中添加调试接口
class BufferQueue {
public:
    void DumpStats() {
        printf("free: %zu, dirty: %zu, all: %zu\n",
               freeList_.size(), dirtyList_.size(), allBuffers_.size());
    }
};
```

---

### 1.5 画面花屏/撕裂

**现象**: 显示内容异常，有残留或撕裂

**可能原因**:

1. **Cache 未刷新**
   ```cpp
   // 如果使用 CACHE 模式，确保正确设置
   surface->SetUsage(BUFFER_CONSUMER_USAGE_HARDWARE_PRODUCER_CACHE);
   // FlushBuffer 会自动刷新 Cache
   ```

2. **Buffer 被提前释放**
   - 检查 Consumer 是否在合成完成前 Release
   - 检查 Producer 是否多次 Flush 同一 Buffer

3. **Stride 不匹配**
   ```cpp
   // 确保使用正确的 stride
   uint32_t stride = surface->GetStride();
   // 绘制时使用 stride 而非 width
   ```

---

## 2. 调试方法

### 2.1 日志级别

surface_lite 使用 `graphic_utils_lite` 的日志系统:

```cpp
// 日志宏定义 (来自 graphic_utils_lite)
GRAPHIC_LOGE("Error message");   // 错误
GRAPHIC_LOGW("Warning message"); // 警告
GRAPHIC_LOGI("Info message");    // 信息
GRAPHIC_LOGD("Debug message");   // 调试
```

**开启调试日志**:
```bash
# 在构建时启用调试
hb build -b debug surface_lite
```

### 2.2 关键日志位置

| 位置 | 日志关键词 | 说明 |
|------|-----------|------|
| `buffer_manager.cpp` | "Alloc graphic buffer" | Buffer 分配 |
| `buffer_queue.cpp` | "has alloced N buffer" | 队列状态 |
| `surface_impl.cpp` | "surface init failed" | 初始化失败 |
| `buffer_client_producer.cpp` | "SendRequest failed" | IPC 失败 |

### 2.3 GDB 调试

**调试步骤**:

```bash
# 1. 编译 Debug 版本
hb build -b debug surface_lite

# 2. 启动 GDB
gdb ./out/{product}/bin/your_app

# 3. 设置断点
(gdb) break BufferQueue::RequestBuffer
(gdb) break BufferManager::AllocBuffer

# 4. 运行
(gdb) run

# 5. 查看调用栈
(gdb) bt

# 6. 查看变量
(gdb) p freeList_.size()
(gdb) p queueSize_
```

### 2.4 性能分析

**Buffer 轮转延迟测量**:

```cpp
#include <chrono>

// Producer 端
auto start = std::chrono::high_resolution_clock::now();
SurfaceBuffer* buffer = surface->RequestBuffer(1);
surface->FlushBuffer(buffer);
auto end = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
printf("Producer cycle: %ld us\n", duration.count());
```

---

## 3. 错误码速查

### 3.1 BufferErrorCode

```cpp
enum BufferErrorCode {
    SURFACE_ERROR_INVALID_PARAM = -10,      // 参数无效
    SURFACE_ERROR_INVALID_REQUEST,          // 请求无效 (如越界请求码)
    SURFACE_ERROR_NOT_READY,                // BufferManager 未初始化
    SURFACE_ERROR_SYSTEM_ERROR,             // 系统错误 (内存分配失败)
    SURFACE_ERROR_BUFFER_NOT_EXISTED,       // Buffer 不存在
    SURFACE_ERROR_OK = 0,                   // 成功
};
```

### 3.2 常见返回值

| 方法 | 返回值 | 含义 |
|------|--------|------|
| `RequestBuffer()` | `nullptr` | 无可用 Buffer |
| `FlushBuffer()` | `-1` | Buffer 状态错误或不存在 |
| `AcquireBuffer()` | `nullptr` | Dirty 队列为空 |
| `ReleaseBuffer()` | `false` | Buffer 状态不是 ACQUIRE |
| `SetInt32()` | `-1` | key 不存在或类型不匹配 |

---

## 4. 调试接口

### 4.1 添加自定义调试接口

在 `buffer_queue.h` 中添加:

```cpp
class BufferQueue {
public:
    // 调试接口
    void DumpState(const char* tag);
    size_t GetFreeCount() const { return freeList_.size(); }
    size_t GetDirtyCount() const { return dirtyList_.size(); }
    size_t GetTotalCount() const { return allBuffers_.size(); }
};
```

实现:

```cpp
void BufferQueue::DumpState(const char* tag) {
    GRAPHIC_LOGI("[%s] BufferQueue State:", tag);
    GRAPHIC_LOGI("  QueueSize: %d, AttachCount: %d", queueSize_, attachCount_);
    GRAPHIC_LOGI("  Free: %zu, Dirty: %zu, Total: %zu",
                 freeList_.size(), dirtyList_.size(), allBuffers_.size());
    GRAPHIC_LOGI("  Width: %d, Height: %d, Format: %d",
                 width_, height_, format_);
}
```

### 4.2 Buffer 状态跟踪

添加状态转换日志:

```cpp
void SurfaceBufferImpl::SetState(BufferState newState) {
    GRAPHIC_LOGD("Buffer %p: %d -> %d", this, bufferData_.state, newState);
    bufferData_.state = newState;
}
```

---

## 5. 典型问题案例

### 案例 1: 启动时初始化失败

**现象**: `Surface::CreateSurface()` 返回 `nullptr`

**排查**:

1. 检查日志是否显示 "Failed init buffer manager"
2. 检查 Gralloc 驱动是否加载
   ```bash
   lsmod | grep gralloc
   ```
3. 检查显示驱动是否正常
   ```bash
   cat /proc/fb  # 查看 framebuffer
   ```

**解决**: 确保显示驱动先于 surface_lite 初始化

---

### 案例 2: 跨进程 Buffer 数据不同步

**现象**: Consumer 看到的 Buffer 数据不完整

**原因**: Producer 未正确刷新 Cache

**解决**:
```cpp
// Producer 端确保使用正确的 Usage
surface->SetUsage(BUFFER_CONSUMER_USAGE_HARDWARE_PRODUCER_CACHE);

// FlushBuffer 内部会自动刷新 Cache
// buffer_queue_producer.cpp:257-271
if (buffer->GetUsage() == BUFFER_CONSUMER_USAGE_HARDWARE_PRODUCER_CACHE) {
    manager->FlushCache(*buffer);  // 生产者 cache → 物理内存
}
```

---

### 案例 3: 多线程竞争导致死锁

**现象**: 应用卡死，无响应

**排查**:

1. 使用 GDB 查看线程状态
   ```bash
   (gdb) info threads
   (gdb) thread apply all bt
   ```

2. 检查是否在同一线程中嵌套调用带锁的函数

**常见死锁场景**:
```cpp
// 错误: 在 OnBufferAvailable 回调中调用 AcquireBuffer
class Listener : public IBufferConsumerListener {
    void OnBufferAvailable() override {
        // 如果 OnBufferAvailable 在锁内被调用
        // 而 AcquireBuffer 也需要同一把锁
        buffer = surface->AcquireBuffer();  // 死锁!
    }
};
```

**解决**: 确保回调不在锁内执行 (代码已正确处理)

---

## 6. 性能优化建议

### 6.1 队列大小调优

| 场景 | 推荐队列大小 | 说明 |
|------|-------------|------|
| 低延迟 | 1-2 | 最小内存占用 |
| 平衡 | 2-3 | 推荐值 |
| 高吞吐 | 5-10 | 减少阻塞 |

### 6.2 内存类型选择

| 场景 | 推荐 Usage | 说明 |
|------|-----------|------|
| CPU 绘制 | `SORTWARE` | 虚拟内存，CPU 访问最快 |
| GPU 渲染 → 显示 | `HARDWARE_PRODUCER_CACHE` | GPU cache 优化 |
| 相机 → 显示 | `HARDWARE_CONSUMER_CACHE` | 显示端 cache 优化 |
| 硬件编解码 | `HARDWARE` | 无 cache，硬件直接访问 |

### 6.3 避免频繁属性修改

**低效**:
```cpp
for (int i = 0; i < 10; i++) {
    surface->SetWidthAndHeight(w, h);  // 每次都会触发 Reset
    buffer = surface->RequestBuffer(1);
    // ...
}
```

**高效**:
```cpp
surface->SetWidthAndHeight(w, h);  // 只设置一次
surface->SetQueueSize(10);         // 预分配
for (int i = 0; i < 10; i++) {
    buffer = surface->RequestBuffer(1);  // 复用
    // ...
}
```

---

## 7. 联系与支持

### 7.1 获取帮助

1. **查看日志**: `hilog` 或 `/var/log/messages`
2. **检查版本**: `bundle.json` 中的 version 字段
3. **复现问题**: 提供最小可复现代码

### 7.2 提交 Issue

当发现疑似 Bug 时，请提供:
- OpenHarmony 版本
- 设备型号
- 复现步骤
- 相关日志
- 最小复现代码

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
