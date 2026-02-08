# 对外接口文档

## 4.1 接口概览

### 多语言接口矩阵

HiSysEvent 提供 5 种编程接口，覆盖不同开发场景和编程语言。Native C++ API 是最完整的接口，适用于系统级应用；N-API 适用于 JavaScript/ArkTS 应用；Rust API 面向 Rust 生态；Easy C API 提供简化的 C 语言接口；ANI 则是 ArkTS Native Interface 的最新接口。

| 接口类型 | 语言 | 完整度 | 适用场景 | 入口文件 |
|----------|------|--------|----------|----------|
| **Native C++ API** | C++ | ⭐⭐⭐⭐⭐ | 系统服务、Native 应用 | `hisysevent.h` |
| **N-API** | JavaScript/ArkTS | ⭐⭐⭐⭐ | JS/ArkTS 应用 | `napi_hisysevent_js.cpp` |
| **Native C API** | C | ⭐⭐⭐ | C 模块、轻量级应用 | `hisysevent_c.h` |
| **Easy C API** | C | ⭐⭐ | 快速集成场景 | `hisysevent_easy.h` |
| **Rust FFI API** | Rust | ⭐⭐⭐⭐ | Rust 子系统 | `lib.rs` |
| **ANI API** | ArkTS | ⭐⭐⭐⭐ | ArkTS Native 开发 | `@ohos.hiSysEvent.ets` |

---

## 4.2 Native C++ API

### 接口清单表

| JS API | 参数类型 | 返回类型 | 同步/异步 | C++ 入口 | 权限要求 |
|--------|----------|----------|-----------|----------|----------|
| `Write()` | `Domain, string, EventType, ...keyValues` | `int` | 同步 | `HiSysEvent::Write()` | `ACCESS_SYSTEM_SERVICE` |
| `Create()` | `Domain, string, EventType` | `HiSysEvent` | 同步 | `HiSysEvent::Create()` | `ACCESS_SYSTEM_SERVICE` |
| `AddListener()` | `HiSysEventRule, HiSysEventListener` | `int` | 异步 | `HiSysEventManager::AddListener()` | `ACCESS_SYSTEM_SERVICE` |
| `RemoveListener()` | `int listenerId` | `int` | 同步 | `HiSysEventManager::RemoveListener()` | `ACCESS_SYSTEM_SERVICE` |
| `Query()` | `HiSysEventQueryArgument` | `HiSysEventRecordCollection` | 异步 | `HiSysEventManager::Query()` | `ACCESS_SYSTEM_SERVICE` |

### 核心 API 详细说明

#### HiSysEvent::Write

**功能描述**：写入系统事件，是 HiSysEvent 最核心的 API。

**函数签名**（`hisysevent.h:78`）：

```cpp
template<typename... Types>
static int Write(const std::string& domain,
                 const std::string& eventName,
                 EventType type,
                 Types... keyValues);
```

**参数说明**：

| 参数 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `domain` | `std::string` | 1-16 字符，字母开头 | 事件所属域 |
| `eventName` | `std::string` | 1-32 字符，字母开头 | 事件名称 |
| `type` | `EventType` | 枚举类型 | 事件类型 |
| `keyValues` | 可变参数 | 最多 128 个 | 键值对参数 |

**返回值**：

| 返回值 | 说明 |
|--------|------|
| `0` | 成功 |
| 负值 | 失败（错误码见 4.7 节） |

**使用示例**：

```cpp
#include "hisysevent.h"

void LogAppStart(const std::string& appName) {
    int ret = HiSysEvent::Write(
        HiSysEvent::Domain::APPEXECFWK,
        "start_app",
        HiSysEvent::EventType::BEHAVIOR,
        "app_name", appName,
        "pid", getpid(),
        "uid", getuid()
    );
    if (ret != 0) {
        // 处理错误
    }
}
```

#### HiSysEvent::Create

**功能描述**：创建 HiSysEvent 对象，支持链式调用。

**函数签名**（`hisysevent.h`）：

```cpp
static HiSysEvent Create(const std::string& domain,
                        const std::string& eventName,
                        EventType type);
```

**成员方法**：

| 方法 | 说明 |
|------|------|
| `PutString(key, value)` | 添加字符串参数 |
| `PutInt(key, value)` | 添加整型参数 |
| `PutLong(key, value)` | 添加长整型参数 |
| `Write()` | 写入事件 |

**使用示例**：

```cpp
auto event = HiSysEvent::Create(
    HiSysEvent::Domain::APPEXECFWK,
    "app_start",
    HiSysEvent::EventType::BEHAVIOR
).PutString("app_name", appName)
 .PutInt("pid", getpid())
 .PutLong("time", currentTime);

event.Write();
```

### EventType 枚举

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `FAULT` | 1 | 系统故障事件 |
| `STATISTIC` | 2 | 系统统计事件 |
| `SECURITY` | 3 | 安全相关事件 |
| `BEHAVIOR` | 4 | 系统行为事件 |

### Domain 常用常量

| 域常量 | 值 | 说明 |
|--------|-----|------|
| `AAFWK` | `"AAFWK"` | 能力框架 |
| `APPEXECFWK` | `"APPEXECFWK"` | 应用框架 |
| `ACTS` | `"ACTS"` | 分布式调度 |
| `BMS` | `"BMS"` | 包管理 |
| `DISTRIBUTED_DATA` | `"DISTRIBUTED_DATA"` | 分布式数据 |

---

## 4.3 N-API（JavaScript/ArkTS）

### 接口清单表

| JS API | 参数类型 | 返回类型 | 同步/异步 | C++ 入口 | 权限要求 |
|--------|----------|----------|-----------|----------|----------|
| `write()` | `HiSysEventParam` | `number` | 同步 | `HiSysEventWriteNapi` | `ACCESS_SYSTEM_SERVICE` |
| `query()` | `HiSysEventQueryParam` | `HiSysEventQueryResult` | 异步 | `HiSysEventQueryNapi` | `ACCESS_SYSTEM_SERVICE` |
| `addListener()` | `HiSysEventRule` | `number` | 异步 | `HiSysEventAddListenerNapi` | `ACCESS_SYSTEM_SERVICE` |
| `removeListener()` | `number` | `boolean` | 同步 | `HiSysEventRemoveListenerNapi` | `ACCESS_SYSTEM_SERVICE` |

### write()

**功能描述**：从 JavaScript/ArkTS 代码写入系统事件。

**函数签名**（`napi_hisysevent_js.cpp`）：

```typescript
declare namespace hiSysEvent {
    function write(options: HiSysEventWriteParam): number;
}
```

**参数类型**（`@ohos.hiSysEvent.ets`）：

```typescript
interface HiSysEventWriteParam {
    domain: string;
    name: string;
    type: EventType;
    params?: { [key: string]: any };
}
```

**EventType 枚举**：

```typescript
enum EventType {
    FAULT = 1,
    STATISTIC = 2,
    SECURITY = 3,
    BEHAVIOR = 4
}
```

**返回值**：

| 返回值 | 说明 |
|--------|------|
| `0` | 成功 |
| 负值 | 失败（错误码见 4.7 节） |

**使用示例**：

```typescript
import hiSysEvent from '@ohos.hiSysEvent';

@Entry
@Component
struct Index {
  build() {
    Column() {
      Button('Log Event')
        .onClick(() => {
          const ret = hiSysEvent.write({
            domain: 'APPEXECFWK',
            name: 'app_start',
            type: hiSysEvent.EventType.BEHAVIOR,
            params: {
              app_name: 'com.example.app',
              pid: 12345
            }
          });
          console.info(`HiSysEvent write result: ${ret}`);
        })
    }
  }
}
```

### query()

**功能描述**：异步查询历史事件。

**函数签名**：

```typescript
function query(options: HiSysEventQueryParam): Promise<HiSysEventRecord[]>;
```

**参数类型**：

```typescript
interface HiSysEventQueryParam {
    domain?: string;
    eventName?: string;
    type?: EventType;
    beginTime?: number;
    endTime?: number;
    maxSize?: number;
}
```

**使用示例**：

```typescript
import hiSysEvent from '@ohos.hiSysEvent';

async function QueryEvents() {
  const records = await hiSysEvent.query({
    domain: 'APPEXECFWK',
    beginTime: Date.now() - 3600000, // 最近1小时
    maxSize: 100
  });
  records.forEach(record => {
    console.info(`Event: ${record.domain}.${record.name}`);
  });
}
```

### addListener()

**功能描述**：添加事件监听器，实时接收符合规则的事件。

**函数签名**：

```typescript
function addListener(options: HiSysEventRule): number;
```

**监听回调**：

```typescript
hiSysEvent.on('新型Event', (record: HiSysEventRecord) => {
    console.info(`Received event: ${record.domain}.${record.name}`);
});
```

---

## 4.4 Native C API

### 接口清单表

| C API | 参数 | 返回类型 | 说明 |
|-------|------|----------|------|
| `OH_HiSysEvent_Write()` | `domain, eventName, type, params, paramCnt` | `int` | 完整参数写入 |
| `OH_HiSysEvent_WriteSimple()` | `domain, eventName, type, key, value` | `int` | 简化写入 |

### OH_HiSysEvent_Write

**功能描述**：完整的 C API 事件写入接口。

**函数签名**（`hisysevent_c.h`）：

```c
int OH_HiSysEvent_Write(const char *domain,
                       const char *eventName,
                       enum HiSysEventType type,
                       HiSysEventParamList params,
                       int paramCnt);
```

**参数类型定义**（`hisysevent_c.h`）：

```c
typedef enum {
    HISYSEVENT_FAULT = 1,
    HISYSEVENT_STATISTIC = 2,
    HISYSEVENT_SECURITY = 3,
    HISYSEVENT_BEHAVIOR = 4
} HiSysEventType;

typedef struct {
    char key[33];
    char value[49];
} HiSysEventParam;

typedef HiSysEventParam HiSysEventParamList[];
```

**返回值**：

| 返回值 | 说明 |
|--------|------|
| `0` | 成功 |
| 负值 | 失败 |

**使用示例**：

```c
#include "hisysevent_c.h"

int LogAppStart(const char* appName) {
    HiSysEventParam params[] = {
        {.key = "app_name", .value = "com.demo"},
        {.key = "pid", .value = "12345"}
    };
    
    return OH_HiSysEvent_Write(
        "APPEXECFWK",
        "start_app",
        HISYSEVENT_BEHAVIOR,
        params,
        2
    );
}
```

---

## 4.5 Easy C API

### 接口清单表

| C API | 参数 | 返回类型 | 说明 |
|-------|------|----------|------|
| `OH_HiSysEvent_WriteEasy()` | `domain, eventName, type, key, value` | `int` | 单参数简化写入 |
| `OH_HiSysEvent_WriteEasyV()` | `domain, eventName, type, keys, values, cnt` | `int` | 多参数简化写入 |

### OH_HiSysEvent_WriteEasy

**功能描述**：极简 C API，适合快速集成。

**函数签名**（`hisysevent_easy.h`）：

```c
int OH_HiSysEvent_WriteEasy(const char *domain,
                           const char *eventName,
                           int type,
                           const char *key,
                           const char *value);
```

**使用示例**：

```c
#include "hisysevent_easy.h"

// 单参数写入
OH_HiSysEvent_WriteEasy(
    "APPEXECFWK",
    "start_app",
    4,  // BEHAVIOR
    "app_name",
    "com.demo"
);

// 多参数写入
const char* keys[] = {"app_name", "pid"};
const char* values[] = {"com.demo", "12345"};
OH_HiSysEvent_WriteEasyV(
    "APPEXECFWK",
    "start_app",
    4,
    keys,
    values,
    2
);
```

---

## 4.6 Rust FFI API

### 接口清单表

| Rust API | 参数类型 | 返回类型 | 说明 |
|----------|----------|----------|------|
| `write()` | `&str, &str, EventType, &[(&str, ParamType)]` | `Result<()>` | 事件写入 |
| `add_listener()` | `&dyn EventListener` | `Result<u32>` | 添加监听器 |
| `remove_listener()` | `u32` | `Result<()>` | 移除监听器 |
| `query()` | `&QueryRule` | `Result<Vec<HiSysEventRecord>>` | 查询事件 |

### write

**功能描述**：Rust 事件写入接口。

**函数签名**（`lib.rs`）：

```rust
pub fn write(domain: &str,
             name: &str,
             event_type: EventType,
             params: &[(&str, ParamType)])
    -> Result<()>
```

**ParamType 枚举**：

```rust
pub enum ParamType<'a> {
    String(&'a str),
    Int(i64),
    Long(i64),
}
```

**使用示例**：

```rust
use hisysevent::{write, EventType, ParamType};

fn log_app_start(app_name: &str) -> Result<()> {
    write(
        "APPEXECFWK",
        "start_app",
        EventType::BEHAVIOR,
        &[
            ("app_name", ParamType::String(app_name)),
            ("pid", ParamType::Int(12345)),
        ],
    )
}
```

---

## 4.7 错误码说明

### 错误码清单

| 错误码 | 宏定义 | 说明 | 可能原因 |
|--------|--------|------|----------|
| `0` | - | 成功 | - |
| `-1` | `ERR_INVALID_PARAM` | 参数无效 | domain、name 或 key 格式错误 |
| `-2` | `ERR_PERMISSION_DENIED` | 权限不足 | 未声明 requiredSysCap |
| `-3` | `ERR_WRITE_FAILED` | 写入失败 | Socket 连接失败 |
| `-4` | `ERR_RATE_LIMITED` | 超过速率限制 | 写入过于频繁 |
| `-5` | `ERR_BUFFER_FULL` | 缓冲区满 | 待发送数据过多 |
| `-6` | `ERR_SERVICE_UNAVAILABLE` | 服务不可用 | SA 服务未启动 |
| `-7` | `ERR_ENCODING_FAILED` | 编码失败 | 参数序列化错误 |
| `-8` | `ERR_MEMORY_ALLOC` | 内存分配失败 | 堆内存不足 |

**证据来源**：`interfaces/native/innerkits/hisysevent/include/def.h`

### 错误处理建议

| 错误码 | 处理建议 |
|--------|----------|
| `-1` | 检查参数格式，确保 domain 和 name 符合约束 |
| `-2` | 在 config.json 中声明 `SystemCapability.HiviewDFX.HiSysEvent` |
| `-3` | 检查系统服务状态，尝试重新连接 |
| `-4` | 实现退避重试机制 |
| `-5` | 降低写入频率，等待缓冲区释放 |
| `-6` | 系统启动完成前不要调用 |
| `-7` | 检查参数值是否包含非法字符 |
| `-8` | 检查内存使用情况 |

---

## 4.8 IPC 接口定义

### ISystemEventService

**服务 ID**：`SA_ID_SYSTEM_EVENT_SERVICE`

**接口定义**（`ISysEventService.idl`）：

| 方法 | 功能 | 参数类型 | 返回类型 |
|------|------|----------|----------|
| `AddListener` | 添加事件监听器 | `SysEventRule` | `int32` |
| `RemoveListener` | 移除事件监听器 | `int32` | `int32` |
| `Query` | 查询历史事件 | `QueryArgument` | `int32` |
| `AddSubscriber` | 添加订阅者 | `SysEventRule` | `int32` |
| `RemoveSubscriber` | 移除订阅者 | `int32` | `int32` |
| `Export` | 导出事件到文件 | `string, int32` | `int32` |

### ISystemEventCallback

**接口定义**（`ISysEventCallback.idl`）：

| 方法 | 功能 | 参数类型 | 返回类型 |
|------|------|----------|----------|
| `OnQuery` | 查询结果回调 | `int32, IQuerySysEventCallback` | `void` |
| `OnListen` | 监听事件回调 | `int32, SysEventRecord` | `void` |

---

*文档版本：1.0*
*创建时间：2026-02-07*
*最后更新：2026-02-07*
