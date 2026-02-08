# Native API 参考 (C/C++)

## 模块概述

HiTrace Native API 提供 C 和 C++ 两层接口：

| 接口类型 | 头文件 | 稳定性 |
|----------|--------|--------|
| C API | `hitrace/hitracechainc.h` | 稳定 |
| C++ API | `hitrace/hitracechain.h` | 稳定 |
| Meter API | `hitrace_meter/hitrace_meter.h` | 稳定 |
| Meter C API | `hitrace_meter/hitrace_meter_c.h` | 稳定 |

**证据来源**: `interfaces/native/innerkits/include/` 目录结构

---

## HiTraceChain C API

### 头文件

```c
#include <hitrace/hitracechainc.h>
#include <hitrace/trace.h>
```

### 数据结构

#### HiTraceIdStruct

**定义位置**: `hitracechainc.h:82-102`

```c
typedef struct HiTraceIdStruct {
#if __BYTE_ORDER == __LITTLE_ENDIAN
    uint64_t valid : 1;        // 有效性标志
    uint64_t ver : 3;           // 版本号 (HITRACE_VER_1)
    uint64_t chainId : 60;      // 调用链 ID
    
    uint64_t flags : 12;        // 追踪标志位
    uint64_t spanId : 26;       // Span ID
    uint64_t parentSpanId : 26; // 父 Span ID
#endif
} HiTraceIdStruct;
```

**字段说明**:
| 字段 | 位宽 | 说明 |
|------|------|------|
| valid | 1 bit | 0=无效, 1=有效 |
| ver | 3 bits | 版本号，固定为 HITRACE_VER_1 |
| chainId | 60 bits | 全局唯一调用链 ID |
| flags | 12 bits | 追踪控制标志 |
| spanId | 26 bits | 当前 Span ID |
| parentSpanId | 26 bits | 父 Span ID |

### 追踪标志位

**定义位置**: `hitracechainc.h:32-50`

```c
#define HITRACE_FLAG_DEFAULT          (0)
#define HITRACE_FLAG_INCLUDE_ASYNC     (1ULL << 0)   // 追踪异步调用
#define HITRACE_FLAG_DONOT_CREATE_SPAN (1ULL << 1)   // 不创建 Span
#define HITRACE_FLAG_TP_INFO           (1ULL << 2)   // 输出追踪点信息
#define HITRACE_FLAG_NO_BE_INFO        (1ULL << 3)   // 不输出开始/结束信息
#define HITRACE_FLAG_DONOT_ENABLE_LOG  (1ULL << 4)   // 不关联日志
#define HITRACE_FLAG_FAULT_TRIGGER     (1ULL << 5)   // 故障触发
#define HITRACE_FLAG_D2D_TP_INFO       (1ULL << 6)   // 跨设备追踪点信息
```

### 通信模式

**定义位置**: `hitracechainc.h:73-80`

```c
typedef enum HiTraceCommunicationMode {
    HITRACE_CM_DEFAULT  = 0,  // 默认模式
    HITRACE_CM_THREAD   = 1,  // 线程间
    HITRACE_CM_PROCESS  = 2,  // 进程间 (IPC)
    HITRACE_CM_DEVICE   = 3,  // 设备间
} HiTraceCommunicationMode;
```

### 追踪点类型

**定义位置**: `hitracechainc.h:59-67`

```c
typedef enum HiTraceTracepointType {
    HITRACE_TP_CS      = 0,  // 客户端发送
    HITRACE_TP_CR      = 1,  // 客户端接收
    HITRACE_TP_SS      = 2,  // 服务端发送
    HITRACE_TP_SR      = 3,  // 服务端接收
    HITRACE_TP_GENERAL = 4,  // 通用信息
} HiTraceTracepointType;
```

### API 函数

#### HiTraceChainBegin

**声明位置**: `hitracechainc.h:144`

```c
HiTraceIdStruct HiTraceChainBegin(const char* name, unsigned int flags);
HiTraceIdStruct HiTraceChainBeginWithDomain(const char* name, unsigned int flags, unsigned int domain);
```

**参数**:
- `name`: 追踪名称字符串
- `flags`: 追踪标志位组合
- `domain`: HiLog domain (可选)

**返回值**: 新的 HiTraceIdStruct

**示例**:
```c
HiTraceIdStruct traceId = HiTraceChainBegin("myOperation", HITRACE_FLAG_DEFAULT);
if (traceId.valid) {
    // 追踪已开始
}
```

#### HiTraceChainEnd

**声明位置**: `hitracechainc.h:146`

```c
void HiTraceChainEnd(const HiTraceIdStruct* pId);
void HiTraceChainEndWithDomain(const HiTraceIdStruct* pId, unsigned int domain);
```

**参数**:
- `pId`: 要结束的追踪 ID

**示例**:
```c
HiTraceIdStruct traceId = HiTraceChainBegin("operation", 0);
HiTraceChainEnd(&traceId);
```

#### HiTraceChainGetId

**声明位置**: `hitracechainc.h:148`

```c
HiTraceIdStruct HiTraceChainGetId();
HiTraceIdStruct* HiTraceChainGetIdAddress();
```

**返回值**: 当前线程的 TraceId

**示例**:
```c
HiTraceIdStruct* pId = HiTraceChainGetIdAddress();
if (pId->valid) {
    // 当前有活跃追踪
}
```

#### HiTraceChainSetId

**声明位置**: `hitracechainc.h:150`

```c
void HiTraceChainSetId(const HiTraceIdStruct* pId);
```

**参数**:
- `pId`: 要设置的追踪 ID

**示例**:
```c
// 从 IPC 接收后设置到当前线程
HiTraceChainSetId(&receivedId);
```

#### HiTraceChainClearId

**声明位置**: `hitracechainc.h:151`

```c
void HiTraceChainClearId();
```

**功能**: 清除当前线程的追踪 ID

#### HiTraceChainCreateSpan

**声明位置**: `hitracechainc.h:153`

```c
HiTraceIdStruct HiTraceChainCreateSpan();
```

**返回值**: 新的子 Span ID

**示例**:
```c
HiTraceIdStruct parentId = HiTraceChainGetId();
HiTraceIdStruct childId = HiTraceChainCreateSpan();  // 生成子 Span
```

#### HiTraceChainTracepoint

**声明位置**: `hitracechainc.h:155-159`

```c
void HiTraceChainTracepoint(HiTraceTracepointType type, const HiTraceIdStruct* pId, const char* fmt, ...);
void HiTraceChainTracepointEx(HiTraceCommunicationMode mode, HiTraceTracepointType type, 
                               const HiTraceIdStruct* pId, const char* fmt, ...);
```

**参数**:
- `mode`: 通信模式
- `type`: 追踪点类型
- `pId`: 追踪 ID
- `fmt`: 格式化字符串

**示例**:
```c
HiTraceIdStruct traceId = HiTraceChainGetId();
HiTraceChainTracepointEx(HITRACE_CM_PROCESS, HITRACE_TP_CS, &traceId, 
                         "Sending request to service");
```

#### HiTraceChainSaveAndSetId / HiTraceChainRestoreId

**声明位置**: `hitracechainc.h:161-163`

```c
HiTraceIdStruct HiTraceChainSaveAndSetId(const HiTraceIdStruct* pId);
void HiTraceChainRestoreId(const HiTraceIdStruct* pId);
```

**功能**: 保存当前 ID 并设置新 ID / 恢复原 ID

**示例**:
```c
// 在异步任务中安全使用
HiTraceIdStruct savedId = HiTraceChainSaveAndSetId(&newId);
// ... 异步操作 ...
HiTraceChainRestoreId(&savedId);
```

#### 序列化 API

**声明位置**: `hitracechainc.h:228-257`

```c
static inline int HiTraceChainIdToBytes(const HiTraceIdStruct* pId, uint8_t* pIdArray, int len);
static inline HiTraceIdStruct HiTraceChainBytesToId(const uint8_t* pIdArray, int len);
```

**功能**: TraceId 与字节数组之间的转换（用于跨进程传输）

**示例**:
```c
// 序列化
uint8_t bytes[16];
HiTraceChainIdToBytes(&traceId, bytes, sizeof(bytes));

// 反序列化
HiTraceIdStruct newId = HiTraceChainBytesToId(bytes, sizeof(bytes));
```

---

## HiTraceChain C++ API

### 头文件

```cpp
#include <hitrace/hitracechain.h>
#include <hitrace/hitraceid.h>
```

### HiTraceChain 类

**定义位置**: `hitrace/hitracechain.h:27-123`

```cpp
namespace OHOS {
namespace HiviewDFX {
class HiTraceChain final {
public:
    static HiTraceId Begin(const std::string& name, int flags);
    static HiTraceId Begin(const std::string& name, int flags, unsigned int domain);
    static void End(const HiTraceId& id);
    static void End(const HiTraceId& id, unsigned int domain);
    static HiTraceId GetId();
    static HiTraceId* GetIdAddress();
    static void SetId(const HiTraceId& id);
    static void ClearId();
    static HiTraceId CreateSpan();
    static void Tracepoint(HiTraceTracepointType type, const HiTraceId& id, const char* fmt, ...)
        __attribute__((__format__(os_log, 3, 4)));
    static void Tracepoint(HiTraceCommunicationMode mode, HiTraceTracepointType type, 
                           const HiTraceId& id, const char* fmt, ...)
        __attribute__((__format__(os_log, 4, 5)));
    static void Tracepoint(HiTraceCommunicationMode mode, HiTraceTracepointType type, 
                           const HiTraceId& id, unsigned int domain, const char* fmt, ...)
        __attribute__((__format__(os_log, 5, 6)));
    static HiTraceId SaveAndSet(const HiTraceId& id);
    static void Restore(const HiTraceId& id);
};
} // namespace HiviewDFX
} // namespace OHOS
```

### HiTraceId 类

**定义位置**: `hitrace/hitraceid.h:25-94`

```cpp
namespace OHOS {
namespace HiviewDFX {
class HiTraceId final {
public:
    HiTraceId();
    HiTraceId(const HiTraceIdStruct& id);
    HiTraceId(const uint8_t* pIdArray, int len);
    ~HiTraceId() = default;

    bool IsValid() const;
    bool IsFlagEnabled(HiTraceFlag flag) const;
    void EnableFlag(HiTraceFlag flag);
    int GetFlags() const;
    void SetFlags(int flags);
    uint64_t GetChainId() const;
    void SetChainId(uint64_t chainId);
    uint64_t GetSpanId() const;
    void SetSpanId(uint64_t spanId);
    uint64_t GetParentSpanId() const;
    void SetParentSpanId(uint64_t parentSpanId);
    int ToBytes(uint8_t* pIdArray, int len) const;

private:
    HiTraceIdStruct id_;
    friend class HiTraceChain;
};
} // namespace HiviewDFX
} // namespace OHOS
```

### 使用示例

```cpp
#include <hitrace/hitracechain.h>
#include <hitrace/hitraceid.h>

using namespace OHOS::HiviewDFX;

void ProcessRequest() {
    // 开始追踪
    HiTraceId traceId = HiTraceChain::Begin("HandleRequest", HITRACE_FLAG_DEFAULT);
    
    if (traceId.IsValid()) {
        // 检查标志位
        if (traceId.IsFlagEnabled(HITRACE_FLAG_INCLUDE_ASYNC)) {
            // 异步追踪已启用
        }
        
        // 创建子 Span
        HiTraceId spanId = HiTraceChain::CreateSpan();
        
        // 输出追踪点
        HiTraceChain::Tracepoint(
            HITRACE_CM_PROCESS,
            HITRACE_TP_CS,
            traceId,
            "Request received"
        );
        
        // 保存并设置（用于异步任务）
        HiTraceId saved = HiTraceChain::SaveAndSet(spanId);
        // ... 异步操作 ...
        HiTraceChain::Restore(saved);
        
        // 结束追踪
        HiTraceChain::End(traceId);
    }
}
```

---

## HiTraceMeter API

### 头文件

```c
#include <hitrace_meter/hitrace_meter.h>
#include <hitrace_meter/hitrace_meter_c.h>
```

### C API

**定义位置**: `hitrace_meter/hitrace_meter_c.h`

```c
// 开始追踪
void StartTrace(const char* name, unsigned int taskId);

// 结束追踪
void FinishTrace(unsigned int taskId);

// 计数追踪
void TraceByValue(const char* name, int64_t count);

// 检查是否启用
bool IsTraceEnabled();
```

### C++ API

**定义位置**: `hitrace_meter/hitrace_meter.h`

```cpp
class HiTraceMeter {
public:
    static void StartTrace(const std::string& name, int taskId);
    static void FinishTrace(int taskId);
    static void TraceByValue(const std::string& name, int64_t count);
    static bool IsTraceEnabled();
};
```

### 使用示例

```cpp
#include <hitrace_meter/hitrace_meter.h>

using namespace OHOS::HiviewDFX;

void PerformanceTracking() {
    const int taskId = 1001;
    
    HiTraceMeter::StartTrace("DatabaseQuery", taskId);
    // ... 执行数据库查询 ...
    HiTraceMeter::FinishTrace(taskId);
    
    HiTraceMeter::TraceByValue("CacheHits", cacheHits);
}
```

---

## 其他 API

### HiTraceOption

**头文件**: `hitrace_option/hitrace_option.h`

**功能**: 追踪选项管理

### HiTraceDump

**头文件**: `hitrace_dump.h`

**功能**: 追踪数据转储

---

## 错误码

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| `HITRACE_INFO_FAIL` | -1 | 失败 |
| `HITRACE_INFO_ALL_VALID` | 0 | 所有字段有效 |
| `HITRACE_INFO_ALL_VALID_EXCEPT_SPAN` | 1 | 除 Span 外都有效 |

**证据来源**: `hitracechain_inner.h:25-27`
