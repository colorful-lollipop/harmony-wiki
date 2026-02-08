# 内部 API

## 概述

本文档描述 hiviewdfx_cangjie_wrapper 内部的模块接口、依赖方向和稳定性标注。这些 API 主要用于模块间通信，不直接暴露给应用开发者。

**注意**: 内部 API 可能在版本升级中发生变化，使用时需谨慎。

---

## 模块依赖方向

### 依赖关系图

```
kit.PerformanceAnalysisKit
│
├── depends on --> ohos.hi_trace_meter
│
├── depends on --> ohos.hiviewdfx.hi_app_event
│
└── depends on --> ohos.hilog

ohos.hi_trace_meter
├── depends on --> ohos.hilog (通过 cj_deps)
│
└── depends on --> cangjie_ark_interop:ohos.ffi
└── depends on --> cangjie_ark_interop:ohos.labels

ohos.hiviewdfx.hi_app_event
├── depends on --> ohos.hilog (通过 cj_deps)
│
├── depends on --> cangjie_ark_interop:ohos.business_exception
├── depends on --> cangjie_ark_interop:ohos.ffi
└── depends on --> cangjie_ark_interop:ohos.labels

ohos.hilog
├── depends on --> cangjie_ark_interop:ohos.business_exception
└── depends on --> cangjie_ark_interop:ohos.labels
```

**数据来源**: 各模块 `BUILD.gn` 文件

---

## 稳定性标注

### 稳定接口

以下接口标记为稳定（public + APILevel 注解）：

| 模块 | 接口 | 稳定性 | 证据 |
|------|------|--------|------|
| HiLog | Hilog 类所有 public static 方法 | 稳定 | `hilog.cj:104` `@!APILevel` |
| HiLog | LogLevel 枚举 | 稳定 | `hilog.cj:36` `@!APILevel` |
| HiTraceMeter | HiTraceMeter 类 | 稳定 | `hi_trace_meter.cj:39` `@!APILevel` |
| HiAppEvent | HiAppEvent 类 | 稳定 | `hi_app_event.cj:55` `@!APILevel` |
| HiAppEvent | EventType 枚举 | 稳定 | `cj_event.cj:36` `@!APILevel` |
| HiAppEvent | EventValueType 枚举 | 稳定 | `cj_event.cj:110` `@!APILevel` |
| HiAppEvent | AppEventInfo 类 | 稳定 | `cj_event.cj:357` `@!APILevel` |

### 内部接口

以下接口为内部使用，不建议外部调用：

| 接口 | 位置 | 说明 |
|------|------|------|
| `Parameters` 类 | `cj_event.cj:335-348` | 内部辅助类 |
| `readValueType` 函数 | `cj_event.cj:1064-1078` | 内部类型转换 |
| `toCArrParameters` 函数 | `cj_event.cj:1463-1467` | 内部 C 转换 |
| `SUCCESS_CODE` 常量 | `cj_event.cj:26` | 错误码常量 |

---

## 核心内部类型

### HiAppEvent 模块内部类型

#### 1. Parameters 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:335-348`

```cj
class Parameters {
    let key: String
    let value: EventValueType

    init(key: String, value: EventValueType)
    init(ret: CParameters)
}
```

**职责**: 内部使用的参数封装类

#### 2. CArrParameters 类型

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:463-467`

```cj
func toCArrParameters(params: Array<Parameters>): CArrParameters
```

**职责**: 将 Cangjie 参数数组转换为 C 结构

#### 3. readValueType 函数

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:1064-1078`

```cj
func readValueType(valueType: UInt8, value: CPointer<Unit>,
                   size: Int64): EventValueType
```

**职责**: 根据类型值读取 C 内存中的 EventValueType

#### 4. 错误码处理

**代码位置**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:26`

```cj
const SUCCESS_CODE: Int32 = 0

func getErrorInfo(code: Int32): String {
    // 根据错误码返回错误信息
    match (code) {
        case 11100001 => "Function is disabled."
        case 11101001 => "Invalid event domain."
        // ... 更多错误码
    }
}
```

---

### HiLog 模块内部类型

#### 1. LogContentInfo 结构体

**代码位置**: `ohos/hilog/hilog.cj:278-286`

```cj
struct LogContentInfo {
    var count: Int64
    var isPriv: Bool

    init(count!: Int64 = 0, isPriv!: Bool = true)
}
```

**职责**: 日志内容解析的中间状态

#### 2. parseLogContent 函数

**代码位置**: `ohos/hilog/hilog.cj:288-340`

```cj
func parseLogContent(formatStr: String, args: Array<String>): String
```

**职责**:
- 解析格式字符串
- 处理 `%{public}s` 和 `%{private}s` 说明符
- 根据隐私开关替换内容

**处理逻辑**:
```cj
// hilog.cj:315-322
if ((pos + PUBLIC_LEN + PROPERTY_POS) < formatStr.size &&
    formatStr[(pos + PROPERTY_POS)..(pos + PROPERTY_POS + PUBLIC_LEN)] == "public") {
    pos += (PUBLIC_LEN + PROPERTY_POS)
    showPriv = false
} else if ((pos + PRIVATE_LEN + PROPERTY_POS) < formatStr.size &&
    formatStr[(pos + PROPERTY_POS)..(pos + PROPERTY_POS + PRIVATE_LEN)] == "private") {
    pos += (PRIVATE_LEN + PROPERTY_POS)
}

info.isPriv = showPriv && unsafe { IsPrivateSwitchOn() }

if (info.isPriv) {
    logContent += "<private>"
} else {
    logContent += args[info.count]
}
```

---

## FFI 接口定义

### HiLog FFI

**代码位置**: `ohos/hilog/hilog.cj:23-27`

```cj
foreign func HiLogIsLoggable(domain: UInt32, tag: CString,
                             level: UInt32): Bool

foreign func HiLogPrint(ty: UInt32, level: UInt32, domain: UInt32,
                        tag: CString, format: CString,
                        content: CString): Unit

foreign func IsPrivateSwitchOn(): Bool
```

### HiTraceMeter FFI

**代码位置**: `ohos/hi_trace_meter/hi_trace_meter.cj:22-26`

```cj
foreign {
    func FfiOHOSHiTraceStartAsyncTrace(name: CString, taskId: Int32): Unit
    func FfiOHOSHiTraceFinishAsyncTrace(name: CString, taskId: Int32): Unit
}
```

### HiAppEvent FFI

**代码位置**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:24-44`

```cj
foreign func FfiOHOSHiAppEventConfigure(config: CConfigOption): Int32
foreign func FfiOHOSHiAppEventWrite(params: CAppEventInfo): Int32
foreign func FfiOHOSHiAppEventAddProcessor(processor: CProcessor): RetDataBool
foreign func FfiOHOSHiAppEventRemoveProcessor(id: Int64): Int32
foreign func FfiOHOSHiAppEventSetUserId(name: CString, value: CString): Int32
foreign func FfiOHOSHiAppEventGetUserId(name: CString): RetDataCString
foreign func FfiOHOSHiAppEventSetUserProperty(name: CString, value: CString): Int32
foreign func FfiOHOSHiAppEventgetUserProperty(name: CString): RetDataCString
foreign func FfiOHOSHiAppEventclearData(): Unit
foreign func FfiOHOSHiAppEventAddWatcher(Watcher: RetWatcher): RetDataI64
foreign func FfiOHOSHiAppEventRemoveWatcher(watcher: RetWatcher): Int32
```

---

## 资源生命周期

### C 字符串资源管理

**模式**: 使用 `asResource()` 自动释放

```cj
// hilog.cj:154-162
unsafe {
    try (cTag = LibC.mallocCString(tag).asResource(),
         cFormat = LibC.mallocCString(logContent).asResource()) {
        HiLogPrint(..., cTag.value, cFormat.value)
    }  // 自动释放 cTag 和 cFormat
}
```

### FFI 数据释放

**模式**: 使用 `free()` 方法释放

```cj
// hi_app_event.cj:83-90
let cinfo = info.toCAppEventInfo()
let code = unsafe { FfiOHOSHiAppEventWrite(cinfo) }
cinfo.free()  // 释放 C 结构
if (code != SUCCESS_CODE) {
    throw BusinessException(code, ...)
}
```

---

## 线程模型

### 工作线程支持

**HiAppEvent.write()** 支持工作线程执行：

```cj
//!APILevel[..., workerthread: true]
public static func write(info: AppEventInfo): Unit
```

**数据来源**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:80`

### 异常处理线程

**HiLog** 自动捕获线程异常：

```cj
// hilog.cj:106-117
static init() {
    Thread.handleUncaughtExceptionBy {
        _: Thread, exception: Exception =>
        Hilog.error(0, "exception", "An exception has occurred:")
        Hilog.error(0, "exception", exception.toString())
    }
}
```

---

## 错误传播机制

### 错误码映射

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| 0 | 成功 | 正常返回 |
| 负数 | FFI 错误 | 抛出 BusinessException |
| 正数 (1110xxxx) | 业务错误 | 抛出 BusinessException |

### 异常抛出模式

```cj
// hi_app_event.cj:86-90
if (code != SUCCESS_CODE) {
    HI_APP_EVENT_LOG.error(getErrorInfo(code))
    throw BusinessException(code, getErrorInfo(code))
}
```

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [03_Architecture.md](03_Architecture.md) | 架构说明 |
| [04_External_API.md](04_External_API.md) | 对外 API |
| [06_GN_Build.md](06_GN_Build.md) | 构建系统 |
| [08_Security_Review.md](08_Security_Review.md) | 安全评审 |
