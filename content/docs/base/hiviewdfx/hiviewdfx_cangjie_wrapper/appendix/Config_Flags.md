# 配置开关

## 概述

本文档描述 hiviewdfx_cangjie_wrapper 相关的配置开关、宏定义和系统能力标识。

---

## 系统能力 (syscap)

### 声明位置

在 `bundle.json` 中声明系统能力：

```json
{
    "syscap": [
        "SystemCapability.HiviewDFX.HiLog",
        "SystemCapability.HiviewDFX.HiAppEvent",
        "SystemCapability.HiviewDFX.HiTrace"
    ]
}
```

**数据来源**: `bundle.json` (组件配置)

### 能力清单

| 系统能力 | 对应模块 | 用途 |
|---------|---------|------|
| `SystemCapability.HiviewDFX.HiLog` | HiLog | 日志功能 |
| `SystemCapability.HiviewDFX.HiAppEvent` | HiAppEvent | 事件功能 |
| `SystemCapability.HiviewDFX.HiTrace` | HiTraceMeter | 追踪功能 |

---

## API 级别注解

### @APILevel 注解

所有公开 API 均使用 `@!APILevel` 注解：

```cj
//!APILevel[
    since: "22",
    syscap: "SystemCapability.HiviewDFX.*",
    throwexception: true,    // 可选
    workerthread: true       // 可选
]
```

### 注解参数

| 参数 | 类型 | 描述 | 使用位置 |
|------|------|------|---------|
| since | String | 最低 API 版本 | 所有 public API |
| syscap | String | 系统能力标识 | 所有 public API |
| throwexception | Bool | 是否抛出异常 | HiAppEvent API |
| workerthread | Bool | 是否支持工作线程 | HiAppEvent.write() |
| isChecked | Bool | 内部检查 | 内部类型 |

**代码证据**: `ohos/hilog/hilog.cj:100-103`

---

## 模块配置

### 平台条件编译

**Windows/macOS Mock 配置**:

```gn
# ohos/hilog/BUILD.gn
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.hilog.cj" ]
} else {
    sources = [ "hilog.cj", "hilog_channel.cj" ]
}
```

**影响的模块**:
- `ohos/hilog` - `ohos/hilog/BUILD.gn:21-28`
- `ohos/hi_trace_meter` - `ohos/hi_trace_meter/BUILD.gn:21-25`
- `ohos/hiviewdfx` - `ohos/hiviewdfx/BUILD.gn:21-25`
- `ohos/hiviewdfx/hi_app_event` - `ohos/hiviewdfx/hi_app_event/BUILD.gn:21-32`

---

## 日志级别配置

### LogLevel 枚举值

```cj
// hilog.cj:36-81
public enum LogLevel {
    Debug    // 值: 3
    Info     // 值: 4
    Warning  // 值: 5
    Error    // 值: 6
    Fatal    // 值: 7
}
```

### 日志级别检查

```cj
// hilog.cj:131-138
public static func isLoggable(domain: UInt32, tag: String, level: LogLevel): Bool {
    unsafe {
        let cstr = LibC.mallocCString(tag)
        let result = HiLogIsLoggable(domain, cstr, level.getValue())
        LibC.free(cstr)
        return result
    }
}
```

---

## 事件类型配置

### EventType 枚举值

```cj
// cj_event.cj:36-72
public enum EventType {
    Fault      // 值: 1
    Statistic  // 值: 2
    Security   // 值: 3
    Behavior   // 值: 4
}
```

### EventValueType 枚举值

```cj
// cj_event.cj:110-200
public enum EventValueType {
    IntValue(Int32)       // 值: 0
    FloatValue(Float64)   // 值: 1
    StringValue(String)   // 值: 2
    BoolValue(Bool)       // 值: 3
    ArrString(Array<String>)   // 值: 4
    ArrInt32(Array<Int32>)      // 值: 5
    ArrBool(Array<Bool>)        // 值: 6
    ArrFloat64(Array<Float64>)  // 值: 7
    Int64Value(Int64)           // 值: 8
    ArrInt64(Array<Int64>)      // 值: 9
}
```

---

## 存储配置

### ConfigOption 配置

```cj
// cj_event.cj:476-522
public class ConfigOption {
    public var disable: Bool           // 默认: false
    public var maxStorage: String      // 默认: "10M"
}
```

### 存储配额格式

| 格式 | 说明 | 示例 |
|------|------|------|
| b | 字节 | "100b" |
| k/K | 千字节 | "10k", "10K" |
| m/M | 兆字节 | "10m", "10M" |
| g/G | 吉字节 | "1g", "1G" |
| t/T | 太字节 | "1t", "1T" |

**建议值**: <= 10 MB

---

## 错误码配置

### 错误码常量

```cj
// cj_event.cj:26
const SUCCESS_CODE: Int32 = 0
```

### HiAppEvent 错误码范围

| 错误码 | 模块 | 含义 |
|--------|------|------|
| 11100001 | HiAppEvent | 功能禁用 |
| 11101001-11101006 | HiAppEvent | 事件参数错误 |
| 11102001-11102005 | HiAppEvent | Watcher 错误 |
| 11103001 | HiAppEvent | 存储配额错误 |
| 11104001 | HiAppEvent | 大小值错误 |
| 11105001 | 通用 | 参数错误 |

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:64-75`

---

## 隐私配置

### 隐私开关

```cj
// hilog.cj:27
foreign func IsPrivateSwitchOn(): Bool

// hilog.cj:324
info.isPriv = showPriv && unsafe { IsPrivateSwitchOn() }
```

### 格式说明符

| 说明符 | 行为 |
|--------|------|
| `%{public}s` | 公开内容 |
| `%{private}s` | 私有内容（受隐私开关控制） |

---

## 构建配置

### GN 变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| is_mingw | Windows 平台 | - |
| is_mac | macOS 平台 | - |
| subsystem_name | 子系统名 | "hiviewdfx" |
| part_name | 组件名 | "hiviewdfx_cangjie_wrapper" |

**代码证据**: 各模块 `BUILD.gn`

---

## 组件配置

### bundle.json 配置

```json
{
    "name": "@ohos/hiviewdfx_cangjie_wrapper",
    "version": "6.1",
    "adapted_system_type": ["standard"],
    "rom": "400KB",
    "ram": "420KB"
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | @ohos/hiviewdfx_cangjie_wrapper | 包名 |
| version | 6.1 | 版本号 |
| adapted_system_type | standard | 适配设备类型 |
| rom | 400KB | ROM 占用 |
| ram | 420KB | RAM 占用 |

**数据来源**: `bundle.json:2-21`

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [06_GN_Build.md](../06_GN_Build.md) | 构建系统 |
| [04_External_API.md](../04_External_API.md) | API 参考 |
| [09_Troubleshooting.md](../09_Troubleshooting.md) | 常见问题 |
