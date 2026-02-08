# C++ API 参考

## 头文件

| 头文件 | 描述 |
|--------|------|
| `hisysevent.h` | C++ 主接口（模板接口） |
| `hisysevent_c.h` | C 接口（纯 C 兼容） |
| `def.h` | 常量定义（错误码、限制） |

## HiSysEvent 类

### 头文件

```cpp
#include "hisysevent.h"
```

### 命名空间

```cpp
using namespace OHOS::HiviewDFX;
```

### Domain 预定义类

```cpp
class HiSysEvent::Domain {
public:
    static constexpr char AAFWK[] = "AAFWK";
    static constexpr char ACCESS_TOKEN[] = "ACCESS_TOKEN";
    static constexpr char ACCOUNT[] = "ACCOUNT";
    static constexpr char ACE[] = "ACE";
    static constexpr char AI[] = "AI";
    static constexpr char APPEXECFWK[] = "APPEXECFWK";
    // ... 共 100+ 个预定义领域
    static constexpr char OTHERS[] = "OTHERS";
};
```

### EventType 枚举

```cpp
enum EventType {
    FAULT     = 1,    // 系统故障事件
    STATISTIC = 2,    // 系统统计事件
    SECURITY  = 3,    // 系统安全事件
    BEHAVIOR  = 4     // 系统行为事件
};
```

### Write() 方法

#### 函数签名

```cpp
// 运行时 domain
template<typename... Types>
static int Write(const char* func, int64_t line, const std::string &domain,
    const std::string &eventName, EventType type, Types... keyValues);

// 编译期 domain（推荐）
template<const char* domain, typename... Types, std::enable_if_t<!isMasked<domain>>* = nullptr>
static int Write(const char* func, int64_t line, const std::string& eventName,
    EventType type, Types... keyValues);
```

#### 参数

| 参数 | 类型 | 描述 |
|------|------|------|
| func | const char* | 函数名（使用 `__FUNCTION__`） |
| line | int64_t | 行号（使用 `__LINE__`） |
| domain | const std::string& | 事件领域 |
| eventName | const std::string& | 事件名称 |
| type | EventType | 事件类型 |
| keyValues | Types... | 键值对参数（可变参数） |

#### 返回值

| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| > 0 | 成功，但部分数据被忽略 |
| < 0 | 失败 |

#### 使用示例

```cpp
#include "hisysevent.h"

using namespace OHOS::HiviewDFX;

// 方式 1: 使用预定义 Domain（推荐）
HiSysEvent::Write<HiSysEvent::Domain::AAFWK>(
    __FUNCTION__, __LINE__,
    "start_app",
    HiSysEvent::EventType::BEHAVIOR,
    "app_name", "com.demo",
    "duration", 1500
);

// 方式 2: 使用字符串 Domain
HiSysEvent::Write(
    __FUNCTION__, __LINE__,
    HiSysEvent::Domain::AAFWK,
    "start_app",
    HiSysEvent::EventType::BEHAVIOR,
    "app_name", "com.demo"
);

// 方式 3: 自定义领域
HiSysEvent::Write(
    __FUNCTION__, __LINE__,
    "MY_DOMAIN",  // 自定义领域（16字符内）
    "my_event",
    HiSysEvent::EventType::BEHAVIOR,
    "key1", "value1",
    "key2", 123
);
```

### 便捷宏

#### HiSysEventWrite

```cpp
#define HiSysEventWrite(domain, eventName, type, ...) \
({ \
    int hiSysEventWriteRet2023___ = OHOS::HiviewDFX::ERR_DOMAIN_MASKED; \
    if constexpr (!OHOS::HiviewDFX::isMasked<domain>) { \
        hiSysEventWriteRet2023___ = OHOS::HiviewDFX::HiSysEvent::Write<domain>(__FUNCTION__, __LINE__, \
            eventName, type, ##__VA_ARGS__); \
    } \
    hiSysEventWriteRet2023___; \
})
```

#### 使用示例

```cpp
// 最简洁的使用方式
HiSysEventWrite(HiSysEvent::Domain::AAFWK, "start_app",
    HiSysEvent::EventType::BEHAVIOR,
    "app_name", "com.demo",
    "duration", 1500);
```

## C 接口

### 头文件

```cpp
#include "hisysevent_c.h"
```

### 枚举类型

```c
enum HiSysEventEventType {
    HISYSEVENT_FAULT = 1,
    HISYSEVENT_STATISTIC = 2,
    HISYSEVENT_SECURITY = 3,
    HISYSEVENT_BEHAVIOR = 4
};

enum HiSysEventParamType {
    HISYSEVENT_INVALID = 0,
    HISYSEVENT_BOOL = 1,
    HISYSEVENT_INT8 = 2,
    HISYSEVENT_UINT8 = 3,
    HISYSEVENT_INT16 = 4,
    HISYSEVENT_UINT16 = 5,
    HISYSEVENT_INT32 = 6,
    HISYSEVENT_UINT32 = 7,
    HISYSEVENT_INT64 = 8,
    HISYSEVENT_UINT64 = 9,
    HISYSEVENT_FLOAT = 10,
    HISYSEVENT_DOUBLE = 11,
    HISYSEVENT_STRING = 12,
    HISYSEVENT_BOOL_ARRAY = 13,
    HISYSEVENT_INT8_ARRAY = 14,
    // ... 共 24 种类型
    HISYSEVENT_STRING_ARRAY = 24
};
```

### 参数结构体

```c
struct HiSysEventParam {
    char name[MAX_LENGTH_OF_PARAM_NAME];
    HiSysEventParamType t;
    HiSysEventParamValue v;
    size_t arraySize;
};

union HiSysEventParamValue {
    bool b;
    int8_t i8;
    uint8_t ui8;
    int16_t i16;
    uint16_t ui16;
    int32_t i32;
    uint32_t ui32;
    int64_t i64;
    uint64_t ui64;
    float f;
    double d;
    char *s;
    void *array;
};
```

### 写入函数

```c
#define OH_HiSysEvent_Write(domain, name, type, params, size) \
    HiSysEvent_Write(__FUNCTION__, __LINE__, domain, name, type, params, size)

int HiSysEvent_Write(const char* func, int64_t line, const char* domain, const char* name,
    HiSysEventEventType type, const HiSysEventParam params[], size_t size);
```

### C 接口使用示例

```cpp
#include "hisysevent_c.h"

// 准备参数
HiSysEventParam params[] = {
    {
        .name = "app_name",
        .t = HISYSEVENT_STRING,
        .v.s = "com.demo",
        .arraySize = 0
    },
    {
        .name = "duration",
        .t = HISYSEVENT_INT32,
        .v.i32 = 1500,
        .arraySize = 0
    }
};

// 写入事件
int ret = OH_HiSysEvent_Write(
    "AAFWK",
    "start_app",
    HISYSEVENT_BEHAVIOR,
    params,
    2  // 参数数量
);

if (ret < 0) {
    // 处理错误
}
```

## 错误码

### 基础错误码

| 常量 | 值 | 描述 |
|------|-----|------|
| SUCCESS | 0 | 成功 |
| ERR_DOMAIN_NAME_INVALID | -1 | 无效的领域名 |
| ERR_EVENT_NAME_INVALID | -2 | 无效的事件名 |
| ERR_DOES_NOT_INIT | -3 | 未初始化 |
| ERR_OVER_SIZE | -4 | 超出大小限制 |
| ERR_SEND_FAIL | -5 | 发送失败 |
| ERR_WRITE_IN_HIGH_FREQ | -6 | 高频写入 |
| ERR_DOMAIN_MASKED | -7 | 领域被屏蔽 |
| ERR_EMPTY_EVENT | -8 | 空事件 |
| ERR_RAW_DATA_WROTE_EXCEPTION | -9 | 原始数据写入异常 |

### 参数级错误码

| 常量 | 值 | 描述 |
|------|-----|------|
| ERR_KEY_NAME_INVALID | 1 | 无效的参数名 |
| ERR_VALUE_LENGTH_TOO_LONG | 2 | 字符串值太长 |
| ERR_KEY_NUMBER_TOO_MUCH | 3 | 参数个数超过 128 |
| ERR_ARRAY_TOO_MUCH | 4 | 数组项超过 100 |
| ERR_VALUE_INVALID | 5 | 无效的值 |

## 限制常量

| 常量 | 值 | 描述 |
|------|-----|------|
| MAX_DOMAIN_LENGTH | 16 | 领域名最大长度 |
| MAX_EVENT_NAME_LENGTH | 32 | 事件名最大长度 |
| MAX_PARAM_NAME_LENGTH | 48 | 参数名最大长度 |
| MAX_ARRAY_SIZE | 100 | 数组最大长度 |
| MAX_PARAM_NUMBER | 128 | 参数最大个数 |
| MAX_STRING_LENGTH | 256 * 1024 | 字符串最大长度 (256KB) |
| MAX_DATA_SIZE | 384 * 1024 | 数据最大大小 (384KB) |

## 编译配置

### GN 依赖

```gn
external_deps = [ "hisysevent:libhisysevent" ]
```

### 头文件路径

```
//base/hiviewdfx/hisysevent/interfaces/native/innerkits/hisysevent/include
```

## 相关链接

- [概览](00_Overview.md) - 项目整体介绍
- [架构说明](01_Architecture.md) - 内部架构
- [N-API 参考](02_N-API.md) - JS/ArkTS 接口
- [构建与编译](04_Build.md) - 编译配置
