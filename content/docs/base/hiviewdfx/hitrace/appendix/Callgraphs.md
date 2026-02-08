# 附录：关键调用链

## JS API 调用链

### hiTraceChain.begin() 调用链

```
JavaScript/ArkTS 层
    │
    ▼
┌─────────────────────────────────────────┐
│  napi_hitrace_js.cpp::Begin()           │
│  - 参数解析: name (string), flags        │
│  - 类型检查                              │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  napi_hitrace_util.cpp                 │
│  - CreateHiTraceIdJsObject()           │
│  - JS 对象创建                          │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  hitracechain.cpp::Begin()             │
│  - HiTraceChain C++ 包装器             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  hitracechainc.c::HiTraceChainBegin()  │
│  - TLS 存储                             │
│  - ChainId 生成                         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  hilog (日志输出)                       │
│  - 自动关联 TraceId                     │
└─────────────────────────────────────────┘
```

**证据来源**: `interfaces/js/kits/napi/src/napi_hitrace_js.cpp:73-103`

---

### hiTraceChain.tracepoint() 调用链

```
JavaScript/ArkTS 层
    │
    ▼
┌─────────────────────────────────────────────────────┐
│  napi_hitrace_js.cpp::Tracepoint()                  │
│  - 参数解析: mode, type, id, description            │
│  - 类型校验                                          │
└────────────────────────┬────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  hitracechain.cpp::Tracepoint()                    │
│  - 转发到 C 实现                                    │
└────────────────────────┬────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  hitracechainc.c::HiTraceChainTracepointInner()   │
│  - 参数验证                                         │
│  - 格式字符串处理                                   │
└────────────────────────┬────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  HiLog (hilog/log.h)                               │
│  - 输出带 TraceId 的日志                           │
│  - 格式: [HiTrace][chainId][spanId] message        │
└─────────────────────────────────────────────────────┘
```

**证据来源**: `frameworks/native/hitracechain.cpp:71-104`

---

### hiTraceChain.end() 调用链

```
JavaScript/ArkTS 层
    │
    ▼
┌─────────────────────────────────────────┐
│  napi_hitrace_js.cpp::End()            │
│  - 参数解析: HiTraceId 对象              │
│  - 类型检查                             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  hitracechain.cpp::End()               │
│  - 转发到 C 实现                        │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  hitracechainc.c::HiTraceChainEnd()    │
│  - TLS 清除                             │
│  - ChainId 验证                        │
└─────────────────────────────────────────┘
```

---

### 跨进程追踪调用链

```
┌──────────────────────────────┐      ┌──────────────────────────────┐
│        客户端进程            │      │         服务端进程            │
│                              │      │                              │
│  HiTraceChain::Begin()       │      │                              │
│  ↓                           │      │                              │
│  HiTraceIdStruct 生成        │      │                              │
│  ↓                           │ IPC  │                              │
│  序列化 ToBytes()            │─────▶│ 反序列化 BytesToId()         │
│  ↓                           │      │ ↓                            │
│  通过 Binder 发送            │      │ HiTraceChain::SetId()        │
│                              │      │ ↓                            │
│                              │      │ 使用 TraceId 追踪             │
│                              │      │ ↓                            │
│                              │ IPC  │                              │
│                              │◀─────│                              │
│  接收响应                    │      │ 发送响应                      │
│  ↓                           │      │                              │
│  HiTraceChain::End()         │      │                              │
└──────────────────────────────┘      └──────────────────────────────┘
```

**序列化关键函数**:
```c
// 客户端: hitracechainc.c
int HiTraceChainIdToBytes(const HiTraceIdStruct* pId, uint8_t* pIdArray, int len)

// 服务端: hitracechainc.c  
HiTraceIdStruct HiTraceChainBytesToId(const uint8_t* pIdArray, int len)
```

---

## Native API 调用链

### HiTraceChain::Begin() 核心流程

```
HiTraceChain::Begin(name, flags)
    │
    ├──> HiTraceChainCreateChainId()  // 生成 64-bit ChainId
    │   ├──> gettimeofday()          // 获取时间戳
    │   ├──> HiTraceChainGetDeviceId() // 获取设备 ID
    │   └──> HiTraceChainGetCpuId()    // 获取 CPU ID
    │
    ├──> HiTraceChainSetId(&id)       // TLS 存储
    │
    └──> 返回 HiTraceIdStruct
```

**ChainId 生成算法** (`hitracechainc.c:156-170`):
```
deviceId (20 bits) | cpuId (4 bits) | second (16 bits) | usec (20 bits)
```

---

### TLS 存储机制

```
┌─────────────────────────────────────────────────────────────┐
│                    Thread 1                               │
│  __thread HiTraceIdStruct g_hiTraceId                     │
│         ↓                                                 │
│  HiTraceChain::Begin()                                   │
│         ↓                                                 │
│  HiTraceChainSetId(&id)  ──────────────>  g_hiTraceId   │
│                                                           │
│  HiTraceChain::GetId()  ──────────────>  g_hiTraceId    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    Thread 2                               │
│  __thread HiTraceIdStruct g_hiTraceId                     │
│         ↓                                                 │
│  HiTraceId 独立于 Thread 1                                │
└─────────────────────────────────────────────────────────────┘
```

**证据来源**: `hitracechainc.c:90`

```c
static __thread HiTraceIdStructInner g_hiTraceId = {{0, 0, 0, 0, 0, 0}, {0, 0}};
```

---

## Span 创建调用链

```
HiTraceChain::CreateSpan()
    │
    ├──> HiTraceChainGetId()        // 获取当前 Span
    │       └──> g_hiTraceId        // 从 TLS 读取
    │
    ├──> BKDRHash(parentSpanId)     // 哈希计算
    │
    ├──> 生成新的 spanId
    │
    └──> HiTraceChainSetId(&newId)  // 更新 TLS
```

**哈希算法** (`hitracechainc.c:268-297`):
- 使用 BKDR 哈希函数
- 输入: deviceId, parentSpanId, spanId, timestamp
- 输出: 新的 spanId

---

## Trace 输出调用链

```
HiTraceChain::Tracepoint(mode, type, id, fmt, ...)
    │
    ├──> 参数校验 (mode, type, id.valid)
    │
    ├──> HiTraceChainTracepointInner()
    │       ├──> 格式化日志消息
    │       ├──> 构建 HiLogEntry
    │       └──> 调用 hilog_write()
    │
    └──> 返回
```

**日志域**:
```cpp
#define LOG_DOMAIN 0xD002D33
```

**证据来源**: `napi_hitrace_js.cpp:27-31`

---

## 异步任务追踪调用链

```
主线程                              异步线程
   │                                   │
   ├──> HiTraceChain::Begin()          │
   │        ↓                          │
   │   获取 currentId                   │
   │        ↓                          │
   ├──> HiTraceId* saved =             │
   │    HiTraceChain::SaveAndSet(id)   │
   │                                   │
   │        ↓  (提交异步任务)           │
   │                              ──────│──> 异步执行
   │                                   │
   │                              ──────│──> HiTraceChain::SetId(saved)
   │                                   │
   │                              ──────│──> 使用追踪
   │                                   │
   │                              ──────│──> HiTraceChain::End(saved)
   │                                   │
   ├──> HiTraceChain::Restore(saved)   │
   │                                   │
   └──> 继续主流程                     │
```

**关键 API**:
```cpp
HiTraceId saved = HiTraceChain::SaveAndSet(id);  // 保存并设置
HiTraceChain::Restore(saved);                     // 恢复
```

---

## Meter 追踪调用链

```
JS 层 (hiTraceMeter)
    │
    ▼
┌──────────────────────────────────────────────┐
│  napi_hitrace_meter.cpp::JSTraceStart()    │
│  - 参数: name (string), taskId (number)    │
│  - 验证追踪是否启用                         │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  hitrace_meter.cpp::StartTrace()           │
│  - 构建追踪标记                             │
│  - 写入追踪缓冲区                          │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  追踪数据写入 ftrace                        │
│  (/sys/kernel/debug/tracing/trace)          │
└──────────────────────────────────────────────┘
```
