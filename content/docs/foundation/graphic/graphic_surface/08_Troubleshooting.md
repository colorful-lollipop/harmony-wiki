# 常见问题与定位路径

## 目的
本文档提供 graphic_surface 组件的常见问题、调试方法和故障定位路径。

## 适用范围
面向需要：
- 调试 Surface 相关问题的开发者
- 分析性能瓶颈的性能工程师
- 处理生产环境问题的运维工程师
- 定位 Buffer 泄漏问题的内存工程师

## 问题分类

### 1. 连接问题

#### 问题 1.1：无法连接到 Surface

**症状**：
- `Connect()` 返回 `GSERROR_CONSUMER_IS_CONNECTED` (41206000)
- `RequestBuffer()` 返回 `GSERROR_NO_CONSUMER` (41202000)

**可能原因**：
1. 另一进程已连接
2. Consumer 已断开但未清理
3. PID 检查失败

**定位步骤**：
```cpp
// 1. 检查当前连接状态
auto consumer = IConsumerSurface::Create();
auto producer = consumer->GetProducer();

GSError ret = producer->Connect(listener, API_VER);
if (ret == GSERROR_CONSUMER_IS_CONNECTED) {
    // 记录日志
    BLOGW("Consumer already connected, PID=%d", GetCallingPid());

    // 2. 检查是否为意外状态
    // 查看 /proc/[pid]/status 确认进程状态
}

// 3. 检查死亡通知
// 查看日志中是否有 "OnRemoteDied" 消息
```

**解决方案**：
1. 确认无其他进程连接
2. 如果 Consumer 死亡，等待死亡通知清理
3. 重试连接（带退避）

#### 问题 1.2：连接后立即断开

**症状**：
- `Connect()` 成功
- 第一次 `RequestBuffer()` 返回 `GSERROR_CONSUMER_DISCONNECTED` (41211000)

**可能原因**：
1. Consumer 进程在连接后崩溃
2. Consumer 主动调用 `Disconnect()`
3. 死亡监听器未正确触发

**定位步骤**：
```bash
# 1. 查看 Consumer 进程日志
hdc shell hilog -T Surface -T Graphic

# 2. 查看 PID 变化
watch -n 1 'cat /proc/[consumer-pid]/status | grep State'

# 3. 查看 Surface 日志
hdc shell hilog -T Graphic -v | grep "connectedPid"
```

**证据位置**：
- `surface/src/buffer_queue_producer.cpp:1595-1603` - 死亡通知
- `surface/src/buffer_queue_producer.cpp:1730-1745` - Disconnect 实现

---

### 2. Buffer 队列问题

#### 问题 2.1：RequestBuffer 返回 GSERROR_NO_BUFFER

**症状**：
- `RequestBuffer()` 返回 `GSERROR_NO_BUFFER` (40601000)
- 队列已满，无可用 Buffer

**可能原因**：
1. Buffer 队列大小过小
2. Consumer 处理速度慢
3. 应用 Request 后未 Flush 或 Cancel

**定位步骤**：
```cpp
// 1. 查询当前队列大小
uint32_t queueSize = 0;
producer->GetQueueSize(queueSize);
BLOGI("Current queue size: %u", queueSize);

// 2. 检查阻塞模式
// 查看 buffer_queue.cpp 的 requestBufferNoBlockMode_

// 3. 统计 Buffer 流转
// 在关键点添加日志：
// - RequestBuffer 调用次数
// - FlushBuffer 调用次数
// - ReleaseBuffer 调用次数
```

**解决方案**：
1. 增加队列大小：
   ```cpp
   producer->SetQueueSize(6);  // 默认是 3
   ```
2. 使用非阻塞模式（noblock）
3. 检查 Consumer 是否正常 Release Buffer

**证据位置**：
- `surface/src/buffer_queue.cpp:RequestBufferLocked()` - 队列满处理
- `interfaces/inner_api/surface/ibuffer_producer.h:BUFFER_PRODUCER_GET_QUEUE_SIZE = 3` - 队列大小查询

#### 问题 2.2：Buffer 状态无效错误

**症状**：
- `FlushBuffer()` 返回 `GSERROR_BUFFER_STATE_INVALID` (41207000)
- `ReleaseBuffer()` 返回 `GSERROR_BUFFER_STATE_INVALID`

**可能原因**：
1. 在错误状态下操作 Buffer
2. Buffer 被多次 Flush
3. Buffer 已被释放

**定位步骤**：
```cpp
// 1. 检查 Buffer 序列号
int32_t seqNum = OH_NativeBuffer_GetSeqNum(buffer);
BLOGI("Buffer seq=%d, state unknown", seqNum);

// 2. 追踪 Buffer 状态
// 在 RequestBuffer/FlushBuffer/ReleaseBuffer/CancelBuffer 前后记录状态

// 3. 检查引用计数
int refCount = OH_NativeBuffer_GetRefCount(buffer);
BLOGW("Buffer ref count=%d", refCount);
```

**解决方案**：
1. 遵循正确生命周期：
   - Request → Render → Flush（或 Cancel）
   - Acquire → Process → Release
2. 不要重复 Flush 同一 Buffer
3. 正确使用 Reference/Unreference

**证据位置**：
- `surface/src/buffer_queue.cpp` - Buffer 状态机
- `interfaces/inner_api/surface/surface_buffer.h` - Buffer 状态定义

---

### 3. 同步问题

#### 问题 3.1：Fence 超时

**症状**：
- `SyncFence->Wait()` 超时
- 画面撕裂或闪烁
- 帧率下降

**可能原因**：
1. GPU 操作未完成
2. Fence 信号丢失
3. Fence 等待时间过短

**定位步骤**：
```cpp
// 1. 检查 Fence 状态
sptr<SyncFence> fence = acquireFence;
auto status = fence->GetStatus();
if (status == FenceStatus::INVALID) {
    BLOGE("Invalid fence");
} else if (status == FenceStatus::SIGNALED) {
    BLOGI("Fence already signaled");
}

// 2. 使用合理超时
const int32_t FENCE_TIMEOUT_MS = 5000;  // 5 秒
GSError ret = fence->Wait(FENCE_TIMEOUT_MS);
if (ret != GSERROR_OK) {
    BLOGW("Fence wait timeout: %d ms", FENCE_TIMEOUT_MS);
}

// 3. 使用 SyncFenceTracker 追踪
// utils/trace/surface_trace.h 提供追踪宏
```

**解决方案**：
1. 增加 Fence 等待超时
2. 检查 GPU 驱动是否正常
3. 使用 Fence 合并优化等待

**证据位置**：
- `sync_fence/include/sync_fence.h` - SyncFence 接口
- `sync_fence/include/sync_fence_tracker.h` - Fence 追踪器

#### 问题 3.2：画面撕裂（Tearing）

**症状**：
- 显示画面不完整
- 画面闪烁
- 帧与帧之间有撕裂

**可能原因**：
1. Producer 未正确设置 AcquireFence
2. Consumer 未正确等待 AcquireFence
3. Swap Interval 配置不当

**定位步骤**：
```cpp
// 1. 验证 AcquireFence 传递
int32_t acquireFence = -1;
OHNativeWindow_NativeWindowAcquireBuffer(window, &buffer, &acquireFence);
BLOGI("Acquire fence=%d", acquireFence);

// 2. 验证 ReleaseFence 设置
int32_t releaseFence = -1;
OHNativeWindow_NativeWindowReleaseBuffer(window, buffer, &releaseFence);
BLOGI("Release fence=%d", releaseFence);

// 3. 检查 GPU 驱动日志
hdc shell hilog -T Hwcomposer -v
```

**解决方案**：
1. 确保 FlushBuffer 时正确传递 Fence
2. 确保 AcquireBuffer 后正确等待 Fence
3. 使用 Double Buffering 或 Triple Buffering

---

### 4. 内存问题

#### 问题 4.1：内存泄漏

**症状**：
- 系统可用内存持续下降
- `free` 命令显示内存不足
- 最终系统重启或应用 OOM

**可能原因**：
1. 进程异常死亡，未回收共享内存
2. Buffer 引用计数泄漏
3. 死亡监听器未触发

**定位步骤**：
```bash
# 1. 查看进程内存使用
cat /proc/[pid]/status | grep -E "VmSize|VmRSS"

# 2. 查看 shared memory 使用
ipcs -m | grep [key]

# 3. 查看 Surface 统计
hdc shell hilog -T Graphic | grep "buffer" | grep "alloc"

# 4. 使用 hitrace 追踪
hdc shell hitrace -t 10 -p [pid] --surface
```

**代码级定位**：
```cpp
// 1. 在 Buffer 分配/释放处添加日志
// surface/src/surface_buffer_impl.cpp

// 2. 在 Surface 销毁时检查队列
// surface/src/buffer_queue.cpp

// 3. 在死亡通知中检查清理
// surface/src/buffer_queue_producer.cpp:HandleDeathRecipient
```

**解决方案**：
1. 确保死亡监听器正常工作
2. 使用 `CleanCache(true)` 强制清理
3. 设置 Buffer 队列大小限制
4. 监控进程健康状况

**证据位置**：
- `README.md:59-60` - 内存泄漏风险提示
- `surface/src/buffer_queue_producer.cpp:1595-1603` - 死亡通知

#### 问题 4.2：内存碎片化

**症状**：
- 连续物理内存分配失败
- 大 Buffer 分配缓慢
- 性能逐渐下降

**可能原因**：
1. 频繁分配/释放小 Buffer
2. HEBC 白名单配置不当
3. 内存管理策略不优化

**定位步骤**：
```bash
# 1. 查看 gralloc 日志
hdc shell hilog -T Gralloc -v | grep "alloc"

# 2. 查看 HEBC 使用情况
cat /etc/graphics_game/config/graphics_game.json

# 3. 查看 Buffer 分配大小分布
hdc shell hilog -T Graphic | grep "RequestBuffer" | wc -l
```

**解决方案**：
1. 避免在小内存场景使用 Surface
2. 调整 HEBC 白名单
3. 使用固定大小的 Buffer 池

**证据位置**：
- `README.md:60` - 内存碎片化风险提示
- `utils/hebc_white_list/hebc_white_list.cpp` - HEBC 白名单

---

### 5. IPC 问题

#### 问题 5.1：Binder 调用失败

**症状**：
- IPC 返回 `GSERROR_BINDER` (50401000)
- 跨进程调用超时
- 操作无响应

**可能原因**：
1. Binder 事务超时
2. Binder 驱动异常
3. 目标进程崩溃

**定位步骤**：
```bash
# 1. 查看 Binder 日志
hdc shell hilog -T Binder -v

# 2. 查看 Surface IPC 日志
hdc shell hilog -T Graphic -v | grep "SendRequest"

# 3. 查看 Binder 统计
cat /sys/kernel/debug/binder/stats
```

**解决方案**：
1. 增加 Binder 超时时间
2. 检查目标进程状态
3. 重试机制

**证据位置**：
- `interfaces/inner_api/common/graphic_common.h` - GSERROR_BINDER 定义
- `surface/src/buffer_client_producer.cpp` - IPC 客户端

#### 问题 5.2：IPC 死锁

**症状**：
- 应用无响应（ANR）
- 进程卡在 IPC 调用
- 日志显示等待状态

**可能原因**：
1. Producer 和 Consumer 互相等待
2. 队列满且 noblock 模式未启用
3. Condition Variable 未正确唤醒

**定位步骤**：
```cpp
// 1. 查看等待日志
// surface/src/buffer_queue.cpp

// 2. 查看调用堆栈
// 使用 gdb 或 libunwindstack

// 3. 查看线程状态
cat /proc/[pid]/task/[tid]/stat | awk '{print $3}'
```

**解决方案**：
1. 使用非阻塞模式
2. 检查队列大小配置
3. 审查消费者 ReleaseBuffer 逻辑

---

### 6. 性能问题

#### 问题 6.1：帧率低

**症状**：
- 画面卡顿
- 帧率低于预期
- Input 延迟高

**可能原因**：
1. Buffer 分配慢
2. IPC 调用频繁
3. Consumer 处理慢

**定位步骤**：
```bash
# 1. 使用 hitrace 追踪帧流程
hdc shell hitrace -t 10 --surface frame_report

# 2. 查看 Buffer 分配时间
hdc shell hilog -T Graphic -v | grep "AllocBuffer"

# 3. 查看 IPC 调用频率
hdc shell hilog -T Graphic | grep "RequestBuffer" | wc -l
```

**代码级定位**：
```cpp
// 1. 在关键路径添加性能追踪
#include <utils/trace/surface_trace.h>

SURFACE_TRACE_FUNC();

// 2. 使用 frame_report
#include <frame_report.h>

FrameReport::ReportFrame(frameNum, timestamp, cost);
```

**解决方案**：
1. 增加 Buffer 队列大小
2. 批量操作（RequestBuffers, FlushBuffers）
3. 优化 Consumer 处理速度

**证据位置**：
- `utils/frame_report/export/frame_report.h` - 帧上报接口
- `utils/trace/surface_trace.h` - 追踪宏

#### 问题 6.2：延迟高

**症状**：
- 输入到显示延迟高
- 画面响应慢
- 帧间延迟不一致

**可能原因**：
1. AcquireFence 等待时间长
2. Buffer 在队列中停留久
3. IPC 延迟

**定位步骤**：
```cpp
// 1. 记录各阶段时间戳
int64_t requestTime = GetTimestamp();
RequestBuffer(...);
int64_t flushTime = GetTimestamp();
FlushBuffer(...);
BLOGI("Buffer lifecycle: request->flush = %lld ns", flushTime - requestTime);

// 2. 查看 Fence 等待时间
// sync_fence/src/sync_fence.cpp
```

**解决方案**：
1. 优化 GPU 操作，减少 Fence 等待
2. 减少 IPC 调用
3. 使用 Triple Buffering

---

## 调试工具与方法

### 1. 日志系统

#### hilog 使用
```bash
# 查看 Surface 相关日志
hdc shell hilog -T Graphic -v

# 查看 Buffer 相关日志
hdc shell hilog -T Graphic | grep "Buffer"

# 查看 IPC 相关日志
hdc shell hilog -T Graphic | grep "IPC"

# 查看错误日志
hdc shell hilog -T Graphic -e
```

#### 日志级别
graphic_surface 使用的日志标签：
- `Surface` - Surface 主模块
- `BufferQueue` - Buffer 队列
- `SyncFence` - 同步栅栏
- `BufferHandle` - Buffer 句柄

### 2. 性能追踪

#### hitrace 使用
```bash
# 追踪 Surface 操作 10 秒
hdc shell hitrace -t 10 --surface

# 追踪特定进程
hdc shell hitrace -t 10 -p [pid] --surface

# 查看追踪结果
hdc shell hitrace --dump
```

#### SurfaceTrace 宏
```cpp
#include <utils/trace/surface_trace.h>

void MyFunction() {
    SURFACE_TRACE_FUNC();  // 自动记录函数入口/退出

    // ... 代码 ...

    SURFACE_TRACE_TIME("Operation", costMs);
}
```

### 3. 帧率统计

#### FrameReport 使用
```cpp
#include <frame_report.h>

// 上报帧时间
FrameReport::ReportFrame(frameNum, timestamp, cost);

// 查询帧率
uint32_t fps = FrameReport::GetFPS();
```

#### 统计命令
```bash
# 查看帧率
hdc shell hidumper -s FrameReport -a fps
```

### 4. 内存分析

#### 查看 Surface 统计
```bash
# 查看 Buffer 统计
hdc shell hidumper -s Surface -a buffer

# 查看队列状态
hdc shell hidumper -s Surface -a queue
```

#### 查看共享内存
```bash
# 查看 IPC 共享内存
ipcs -m | grep [key]

# 查看进程映射
cat /proc/[pid]/maps | grep surface
```

### 5. IPC 调试

#### Binder 调试
```bash
# 查看 Binder 事务
hdc shell hilog -T Binder -v

# 查看 Binder 统计
cat /sys/kernel/debug/binder/stats

# 查看 Binder 事务延迟
cat /sys/kernel/debug/binder/latency
```

#### Surface IPC 追踪
```cpp
// 在 BufferClientProducer 中添加日志
// surface/src/buffer_client_producer.cpp

GSError ret = SendRequest(BUFFER_PRODUCER_REQUEST_BUFFER, data, reply);
BLOGI("IPC call: RequestBuffer, ret=%d", ret);
```

### 6. GDB 调试

#### GDB 命令
```bash
# 附加到进程
gdb -p [pid]

# 设置断点
(gdb) break BufferQueue::RequestBuffer

# 查看调用栈
(gdb) bt

# 查看变量
(gdb) print *this
(gdb) print bufferQueueCache_.size()

# 查看线程
(gdb) info threads
(gdb) thread apply all bt
```

### 7. 常用调试宏

graphic_surface 提供的调试工具：
```cpp
// surface/src/buffer_queue.cpp
BLOGE("Error message");           // Error
BLOGW("Warning message");         // Warning
BLOGI("Info message");           // Info
BLOGD("Debug message");           // Debug

// 性能追踪
SURFACE_TRACE_FUNC();             // 函数追踪
SURFACE_TRACE_TIME(name, time);   // 时间追踪
```

## 故障定位清单

### 新问题定位流程
1. **确认症状**
   - 记录复现步骤
   - 记录错误码
   - 记录日志输出

2. **检查日志**
   - 查看 hilog 输出
   - 筛选相关标签（Surface, BufferQueue, SyncFence）
   - 查找错误和警告

3. **检查状态**
   - 进程状态：`cat /proc/[pid]/status`
   - Binder 状态：`hdc shell hilog -T Binder`
   - 共享内存：`ipcs -m`

4. **代码级定位**
   - 根据 API 返回码定位错误位置
   - 查看错误码定义
   - 追踪调用链

5. **性能分析**
   - 使用 hitrace 追踪性能
   - 使用 frame_report 查看帧率
   - 分析瓶颈

6. **验证修复**
   - 重新编译测试
   - 验证问题解决
   - 回归测试

### 证据查找指南

| 问题类型 | 查看文件 | 关键符号 |
|---------|---------|---------|
| 连接问题 | `buffer_queue_producer.cpp` | `Connect()`, `connectedPid_` |
| 队列问题 | `buffer_queue.cpp` | `RequestBufferLocked()`, `freeList_`, `dirtyList_` |
| 状态问题 | `buffer_queue.cpp` | `BUFFER_STATE_*` |
| IPC 问题 | `buffer_client_producer.cpp` | `SendRequest()`, `MessageParcel` |
| 内存问题 | `surface_buffer_impl.cpp` | `IncRef()`, `DecRef()` |
| 同步问题 | `sync_fence.cpp` | `Wait()`, `GetStatus()` |

## 相关跳转
- [目录结构与模块职责](01_Directory_Structure.md) - 源码位置
- [架构说明](02_Architecture.md) - 组件交互与数据流
- [对外 API](03_External_API.md) - API 使用与错误码
- [内部 API](04_Internal_API.md) - 模块间接口
- [安全风险评审](07_Security_Review.md) - 安全相关调试
