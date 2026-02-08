# 04 - 接口文档

**目的**: 提供完整的API参考和使用指南  
**适用范围**: 仓颉应用开发者  
**前置知识**: 阅读 [01_Overview.md](./01_Overview.md)

---

## API概览

### 公开接口清单

| 类型 | 名称 | 功能 | 异常 |
|------|------|------|------|
| **类** | `SystemDateTime` | 系统时间/时区访问 | - |
| **枚举** | `TimeType` | 运行时间类型定义 | - |
| **方法** | `SystemDateTime.getTime()` | 获取系统时间 | 是 |
| **方法** | `SystemDateTime.getUptime()` | 获取运行时间 | 否 |
| **方法** | `SystemDateTime.getTimezone()` | 获取系统时区 | 是 |

### 包信息

```cangjie
package ohos.system_date_time
```

### 导入方式

```cangjie
// 导入整个包
import ohos.system_date_time.*

// 或只导入特定类型
import ohos.system_date_time.SystemDateTime
import ohos.system_date_time.TimeType
```

---

## SystemDateTime 类

### 类定义

**位置**: `ohos/system_date_time/system_date_time.cj:48-102`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.MiscServices.Time"
]
public class SystemDateTime {
    // 静态方法...
}
```

**特性**:
- 纯静态方法类，无需实例化
- 所有方法线程安全
- API级别: 22
- 系统能力: SystemCapability.MiscServices.Time

---

### getTime() 方法

获取从Unix纪元(1970-01-01 00:00:00 UTC)到当前系统时间的经过时间。

#### 方法签名

**位置**: `ohos/system_date_time/system_date_time.cj:56-65`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.MiscServices.Time",
    throwexception: true
]
public static func getTime(isNanoseconds!: Bool = false): Int64
```

#### 参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `isNanoseconds` | `Bool` | 否 | `false` | `true`返回纳秒，`false`返回毫秒 |

#### 返回值

| 类型 | 说明 |
|------|------|
| `Int64` | Unix时间戳，单位取决于`isNanoseconds`参数 |

#### 异常

| 异常类型 | 错误码 | 说明 |
|----------|--------|------|
| `BusinessException` | 16000050 | 内部错误 |

#### 使用示例

```cangjie
import ohos.system_date_time.*
import ohos.business_exception.BusinessException

func getSystemTimeExample(): Unit {
    try {
        // 获取毫秒级时间戳
        let timeMs = SystemDateTime.getTime()
        println("当前时间戳(毫秒): ${timeMs}")
        // 输出示例: 当前时间戳(毫秒): 1707312000000
        
        // 获取纳秒级时间戳
        let timeNs = SystemDateTime.getTime(isNanoseconds: true)
        println("当前时间戳(纳秒): ${timeNs}")
        // 输出示例: 当前时间戳(纳秒): 1707312000000000000
        
        // 转换为秒
        let timeSec = timeMs / 1000
        println("当前时间戳(秒): ${timeSec}")
        
    } catch (e: BusinessException) {
        println("获取时间失败: ${e.message}")
    }
}
```

#### 实现细节

```cangjie
// 内部实现
public static func getTime(isNanoseconds!: Bool = false): Int64 {
    let cValue = unsafe { FfiOHOSSysDateTimeGetTime(isNanoseconds) }
    throwIfNotSuccess(cValue.code)
    return cValue.data
}
```

**FFI函数**: `FfiOHOSSysDateTimeGetTime(isNano: Bool): RetDataI64`

---

### getUptime() 方法

获取系统自启动以来经过的时间。

#### 方法签名

**位置**: `ohos/system_date_time/system_date_time.cj:74-82`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.MiscServices.Time"
]
public static func getUptime(timeType: TimeType, isNanoseconds!: Bool = false): Int64
```

#### 参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `timeType` | `TimeType` | 是 | - | 运行时间类型：`Startup` 或 `Active` |
| `isNanoseconds` | `Bool` | 否 | `false` | `true`返回纳秒，`false`返回毫秒 |

#### 返回值

| 类型 | 说明 |
|------|------|
| `Int64` | 系统运行时间，单位取决于`isNanoseconds`参数 |

#### 异常

| 异常类型 | 错误码 | 说明 |
|----------|--------|------|
| `BusinessException` | 401 | 参数错误（当传入无效的TimeType时） |

**注意**: 虽然方法签名未标记 `throwexception`，但通过 `TimeType.getValue()` 可能抛出参数错误。

#### 使用示例

```cangjie
import ohos.system_date_time.*
import ohos.business_exception.BusinessException

func getUptimeExample(): Unit {
    try {
        // 获取系统启动至今的总时间（含深度睡眠）
        let startupTimeMs = SystemDateTime.getUptime(TimeType.Startup)
        println("系统启动至今(毫秒): ${startupTimeMs}")
        
        // 转换为更易读的格式
        let startupSec = startupTimeMs / 1000
        let minutes = startupSec / 60
        let hours = minutes / 60
        println("系统已运行: ${hours}小时${minutes % 60}分钟")
        
        // 获取系统活跃时间（不含深度睡眠）
        let activeTimeMs = SystemDateTime.getUptime(TimeType.Active)
        println("系统活跃时间(毫秒): ${activeTimeMs}")
        
        // 计算深度睡眠时间
        let sleepTimeMs = startupTimeMs - activeTimeMs
        println("深度睡眠时间(毫秒): ${sleepTimeMs}")
        
        // 获取纳秒级精度
        let startupTimeNs = SystemDateTime.getUptime(
            TimeType.Startup, 
            isNanoseconds: true
        )
        println("系统启动至今(纳秒): ${startupTimeNs}")
        
    } catch (e: BusinessException) {
        println("获取运行时间失败: ${e.message}")
    }
}
```

#### 实现细节

```cangjie
// 内部实现
public static func getUptime(timeType: TimeType, isNanoseconds!: Bool = false): Int64 {
    let cValue = unsafe { 
        FfiOHOSSysDateTimeGetUptime(timeType.getValue(), isNanoseconds) 
    }
    throwIfNotSuccess(cValue.code)
    return cValue.data
}
```

**FFI函数**: `FfiOHOSSysDateTimeGetUptime(timeType: Int32, isNano: Bool): RetDataI64`

---

### getTimezone() 方法

获取系统当前设置的时区标识符。

#### 方法签名

**位置**: `ohos/system_date_time/system_date_time.cj:90-101`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.MiscServices.Time",
    throwexception: true
]
public static func getTimezone(): String
```

#### 参数

无

#### 返回值

| 类型 | 说明 |
|------|------|
| `String` | 时区标识符，格式遵循IANA时区数据库标准（如："Asia/Shanghai"） |

#### 异常

| 异常类型 | 错误码 | 说明 |
|----------|--------|------|
| `BusinessException` | 16000050 | 内部错误 |

#### 使用示例

```cangjie
import ohos.system_date_time.*
import ohos.business_exception.BusinessException

func getTimezoneExample(): Unit {
    try {
        let timezone = SystemDateTime.getTimezone()
        println("系统时区: ${timezone}")
        // 输出示例: 系统时区: Asia/Shanghai
        
        // 根据时区做不同处理
        match (timezone) {
            case "Asia/Shanghai" => {
                println("当前为中国标准时间")
            }
            case "Asia/Tokyo" => {
                println("当前为日本标准时间")
            }
            case "America/New_York" => {
                println("当前为美国东部时间")
            }
            case _ => {
                println("当前为其他时区: ${timezone}")
            }
        }
        
    } catch (e: BusinessException) {
        println("获取时区失败: ${e.message}")
    }
}
```

#### 实现细节

```cangjie
// 内部实现
public static func getTimezone(): String {
    let ret = unsafe { FfiOHOSSysGetTimezone() }
    throwIfNotSuccess(ret.code)
    let time = ret.data.toString()
    unsafe { LibC.free(ret.data) }  // 手动释放内存
    return time
}
```

**FFI函数**: `FfiOHOSSysGetTimezone(): RetDataCString`

**⚠️ 注意**: 本方法涉及手动内存管理。FFI返回的C字符串通过 `LibC.free()` 释放，避免内存泄漏。

---

## TimeType 枚举

### 枚举定义

**位置**: `ohos/system_date_time/cj_date_time_common.cj:30-57`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.MiscServices.Time"
]
public enum TimeType {
    @!APILevel[since: "22", syscap: "SystemCapability.MiscServices.Time"]
    Startup
    |
    @!APILevel[since: "22", syscap: "SystemCapability.MiscServices.Time"]
    Active
    | ...

    func getValue(): Int32
}
```

### 枚举值

| 值 | 内部值 | 说明 | 使用场景 |
|----|--------|------|----------|
| `Startup` | 0 | 系统启动至今的总时间，**包含**深度睡眠时间 | 计算总运行时长 |
| `Active` | 1 | 系统启动至今的活跃时间，**不包含**深度睡眠时间 | 计算实际工作时间 |

### 方法

#### getValue()

获取枚举值的内部整数值。

```cangjie
func getValue(): Int32
```

**实现**:

```cangjie
func getValue(): Int32 {
    match (this) {
        case Startup => 0
        case Active => 1
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**注意**: 由于match语句已覆盖所有枚举值，运行时不会到达 `case _`，但为了编译器完整性仍保留。

### 使用示例

```cangjie
import ohos.system_date_time.*

func timeTypeExample(): Unit {
    // 直接使用枚举值
    let type1 = TimeType.Startup
    let type2 = TimeType.Active
    
    // 获取内部值（通常不需要直接使用）
    let value = type1.getValue()
    println("Startup的内部值: ${value}")  // 输出: 0
}
```

---

## 错误处理

### 错误码定义

| 错误码 | 含义 | 触发场景 |
|--------|------|----------|
| 0 | 成功 | 操作成功完成 |
| -1 | 内部错误 | FFI调用底层失败 |
| 401 | 参数错误 | TimeType.getValue()传入无效值 |
| 16000050 | 内部错误 | 业务错误码映射 |

### 错误处理机制

```cangjie
import ohos.system_date_time.*
import ohos.business_exception.BusinessException

func errorHandlingExample(): Unit {
    try {
        let time = SystemDateTime.getTime()
        println("时间: ${time}")
    } catch (e: BusinessException) {
        // 根据错误码处理
        match (e.code) {
            case 16000050 => {
                println("内部错误: ${e.message}")
                // 可能需要重试或记录日志
            }
            case _ => {
                println("其他错误: ${e.code} - ${e.message}")
            }
        }
    }
}
```

### 错误处理实现

**位置**: `ohos/system_date_time/cj_date_time_error.cj`

```cangjie
func throwIfNotSuccess(code: Int32): Unit {
    if (code != SUCCESS_CODE) {
        if (code == -1) {
            throw BusinessException(16000050, "Internal error.")
        }
        Hilog.error(SYSTEM_DATE_TIME_DOMAIN_ID, "Date-Time", getErrorInfo(code))
        throw BusinessException(code, getErrorInfo(code))
    }
}
```

---

## API使用最佳实践

### 1. 始终处理异常

```cangjie
// ✅ 正确：处理异常
try {
    let time = SystemDateTime.getTime()
} catch (e: BusinessException) {
    // 处理错误
}

// ❌ 错误：忽略异常
let time = SystemDateTime.getTime()  // 可能崩溃
```

### 2. 缓存时间戳（如果频繁使用）

```cangjie
// ✅ 正确：避免频繁调用
var cachedTime: Int64 = 0
var lastUpdate: Int64 = 0

func getCachedTime(): Int64 {
    let now = SystemDateTime.getTime()
    if (now - lastUpdate > 1000) {  // 每秒更新一次
        cachedTime = now
        lastUpdate = now
    }
    return cachedTime
}

// ❌ 错误：每次循环都调用
for (i in 0..10000) {
    let time = SystemDateTime.getTime()  // 频繁FFI调用
}
```

### 3. 选择合适的精度

```cangjie
// ✅ 正确：根据需求选择精度

// 一般时间戳，毫秒足够
let timestamp = SystemDateTime.getTime()

// 性能测量，可能需要纳秒
let startNs = SystemDateTime.getTime(isNanoseconds: true)
// ... 执行操作
let endNs = SystemDateTime.getTime(isNanoseconds: true)
let duration = endNs - startNs
```

### 4. 区分Startup和Active时间

```cangjie
// ✅ 正确：根据场景选择

// 计算设备总开机时长
let totalUptime = SystemDateTime.getUptime(TimeType.Startup)

// 计算设备实际工作时间（排除休眠）
let workTime = SystemDateTime.getUptime(TimeType.Active)

// 计算休眠时间
let sleepTime = totalUptime - workTime
```

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - 项目概览
- [02_Architecture.md](./02_Architecture.md) - 架构分析
- [05_AttackSurface.md](./05_AttackSurface.md) - 攻击面分析

---

**更新记录**

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-07 | v1.0 | 初始版本，基于代码分析创建 |
