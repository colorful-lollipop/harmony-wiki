# 架构说明

## 整体架构

### 系统架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Cangjie Application                                 │
│                                                                               │
│   import kit.PerformanceAnalysisKit                                           │
│   ├── import ohos.hi_trace_meter.*                                           │
│   ├── import ohos.hiviewdfx.hi_app_event.*                                   │
│   └── import ohos.hilog.*                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Kit Layer (kit.PerformanceAnalysisKit)              │
│                                                                               │
│   kit/PerformanceAnalysisKit/index.cj                                        │
│   └── public import ohos.hi_trace_meter.*                                    │
│       public import ohos.hiviewdfx.hi_app_event.*                            │
│       public import ohos.hilog.*                                             │
└─────────────────────────────────────────────────────────────────────────────┘
          │                           │                           │
          ▼                           ▼                           ▼
┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
│   Ohos Module     │   │   Ohos Module     │   │   Ohos Module     │
│ ohos.hi_trace_meter│   │ ohos.hiviewdfx   │   │    ohos.hilog     │
│                   │   │   .hi_app_event   │   │                   │
│ hi_trace_meter.cj │   │   hi_app_event.cj │   │    hilog.cj       │
│                   │   │   cj_event.cj     │   │ hilog_channel.cj  │
└───────────────────┘   └───────────────────┘   └───────────────────┘
          │                           │                           │
          ▼                           ▼                           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FFI Bridge Layer                                      │
│                                                                               │
│   foreign func FfiOHOSHiTraceStartAsyncTrace(...)                            │
│   foreign func FfiOHOSHiTraceFinishAsyncTrace(...)                           │
│   foreign func FfiOHOSHiAppEventWrite(...)                                   │
│   foreign func FfiOHOSHiAppEventConfigure(...)                               │
│   foreign func HiLogIsLoggable(...)                                          │
│   foreign func HiLogPrint(...)                                               │
└─────────────────────────────────────────────────────────────────────────────┘
          │                           │                           │
          ▼                           ▼                           ▼
┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
│   Native Lib      │   │   Native Lib      │   │   Native Lib      │
│  cj_hitracemeter_ │   │  cj_hiappevent_   │   │     libhilog      │
│      ffi          │   │       ffi         │   │                   │
└───────────────────┘   └───────────────────┘   └───────────────────┘
          │                           │                           │
          └───────────────────────────┼───────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      OpenHarmony Native Services                              │
│                                                                               │
│   HiTrace Service    │    HiAppEvent Service    │    HiLog Service           │
│   (性能追踪)         │    (事件打点)            │    (日志打印)              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**数据来源**: `README.md:9-26` 及代码架构分析

---

## 模块职责

### 1. Kit Layer

**组件**: `kit/PerformanceAnalysisKit/`

**职责**:
- 提供统一的模块入口
- 聚合所有 DFX 子模块
- 暴露给 Cangjie 应用使用

**关键代码**:

```cj
// kit/PerformanceAnalysisKit/index.cj:18-22
package kit.PerformanceAnalysisKit

public import ohos.hi_trace_meter.*
public import ohos.hiviewdfx.hi_app_event.*
public import ohos.hilog.*
```

---

### 2. Ohos Module Layer

#### 2.1 HiTraceMeter 模块

**组件**: `ohos/hi_trace_meter/`

**职责**:
- 封装 HiTraceMeter Cangjie 接口
- 转换 Cangjie 类型到 C 类型
- 异常处理与日志记录

**关键代码**:

```cj
// ohos/hi_trace_meter/hi_trace_meter.cj:22-26
foreign {
    func FfiOHOSHiTraceStartAsyncTrace(name: CString, taskId: Int32): Unit
    func FfiOHOSHiTraceFinishAsyncTrace(name: CString, taskId: Int32): Unit
}

// ohos/hi_trace_meter/hi_trace_meter.cj:55-61
public static func startTrace(name: String, taskId: Int32): Unit {
    unsafe {
        try (cName = LibC.mallocCString(name).asResource()) {
            FfiOHOSHiTraceStartAsyncTrace(cName.value, taskId)
        }
    }
}
```

#### 2.2 HiAppEvent 模块

**组件**: `ohos/hiviewdfx/hi_app_event/`

**职责**:
- 封装 HiAppEvent Cangjie 接口
- 提供事件类型定义
- 提供 Watcher/Processor 机制

**关键代码**:

```cj
// ohos/hiviewdfx/hi_app_event/hi_app_event.cj:24-44
foreign func FfiOHOSHiAppEventConfigure(config: CConfigOption): Int32
foreign func FfiOHOSHiAppEventWrite(params: CAppEventInfo): Int32
foreign func FfiOHOSHiAppEventAddProcessor(processor: CProcessor): RetDataBool
// ... 更多 FFI 声明

// ohos/hiviewdfx/hi_app_event/hi_app_event.cj:82-91
public static func write(info: AppEventInfo): Unit {
    let cinfo = info.toCAppEventInfo()
    let code = unsafe { FfiOHOSHiAppEventWrite(cinfo) }
    cinfo.free()
    if (code != SUCCESS_CODE) {
        HI_APP_EVENT_LOG.error(getErrorInfo(code))
        throw BusinessException(code, getErrorInfo(code))
    }
}
```

#### 2.3 HiLog 模块

**组件**: `ohos/hilog/`

**职责**:
- 封装 HiLog Cangjie 接口
- 提供日志级别管理
- 处理格式字符串

**关键代码**:

```cj
// ohos/hilog/hilog.cj:23-27
foreign func HiLogIsLoggable(domain: UInt32, tag: CString, level: UInt32): Bool
foreign func HiLogPrint(ty: UInt32, level: UInt32, domain: UInt32, tag: CString,
                        format: CString, content: CString): Unit
foreign func IsPrivateSwitchOn(): Bool

// ohos/hilog/hilog.cj:152-163
public static func debug(domain: UInt32, tag: String, format: String,
                         args: Array<String>): Unit {
    let logContent = parseLogContent(format, args)
    unsafe {
        try (cTag = LibC.mallocCString(tag).asResource(),
             cFormat = LibC.mallocCString(logContent).asResource()) {
            HiLogPrint(DEFAULT_LOG_TYPE, LogLevel.Debug.getValue(), domain,
                       cTag.value, STANDARD_FORMAT, cFormat.value)
        }
    }
}
```

---

### 3. FFI Bridge Layer

**职责**:
- 定义 Cangjie 与 C 语言的互操作接口
- 处理类型转换
- 管理内存分配与释放

**关键技术**:
- `foreign func` 声明外部 C 函数
- `CString` / `CPointer` 处理 C 字符串
- `unsafe` 块处理指针操作
- `asResource()` 自动资源管理

---

### 4. Native Services Layer

**组件**: OpenHarmony 系统服务

**职责**:
- 实际的日志打印
- 事件存储与分发
- 性能追踪数据收集

**依赖库**:
- `libhilog` - HiLog Native 实现
- `cj_hiappevent_ffi` - HiAppEvent FFI 桥接
- `cj_hitracemeter_ffi` - HiTraceMeter FFI 桥接

---

## 数据流

### 日志输出数据流

```
Cangjie 应用
    │
    ▼
Hilog.debug/info/warn/error/fatal()
    │
    ▼
parseLogContent() ──▶ 格式化字符串处理
    │
    ▼
LibC.mallocCString() ──▶ C 字符串转换
    │
    ▼
HiLogPrint() (FFI 调用)
    │
    ▼
libhilog (Native)
    │
    ▼
HiLog Service → 日志输出
```

---

### 事件打点数据流

```
Cangjie 应用
    │
    ▼
HiAppEvent.write(AppEventInfo)
    │
    ▼
AppEventInfo.toCAppEventInfo() ──▶ C 结构转换
    │
    ▼
FfiOHOSHiAppEventWrite() (FFI 调用)
    │
    ▼
cj_hiappevent_ffi (Native)
    │
    ▼
HiAppEvent Service → 事件存储
```

---

### 性能追踪数据流

```
Cangjie 应用
    │
    ▼
HiTraceMeter.startTrace(name, taskId)
    │
    ▼
FfiOHOSHiTraceStartAsyncTrace() (FFI 调用)
    │
    ▼
cj_hitracemeter_ffi (Native)
    │
    ▼
HiTrace Service → bytrace 捕获
```

---

## 线程模型

### 线程安全机制

#### 1. HiLog 线程安全

- **实现方式**: Native 层保证线程安全
- **Cangjie 层**: 无特殊同步措施
- **异常处理**: 线程异常自动捕获

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

#### 2. HiAppEvent 线程安全

- **workerthread**: write() API 可在工作线程执行
- **异常传播**: BusinessException 抛至调用方

```cj
//!APILevel[..., workerthread: true]
public static func write(info: AppEventInfo): Unit
```

#### 3. HiTraceMeter 线程安全

- **实现方式**: 异步追踪，无阻塞
- **建议**: 在业务线程调用

---

## 错误处理

### 错误码体系

| 模块 | 错误码范围 | 说明 |
|------|-----------|------|
| 通用 | 11105001 | 参数错误 |
| HiAppEvent | 11100001-11104001 | 功能禁用、无效参数 |
| HiLog | - | 无特定错误码 |
| HiTraceMeter | - | 无特定错误码 |

### 异常处理模式

```cj
// 模式1: 返回值检查
if (code != SUCCESS_CODE) {
    throw BusinessException(code, getErrorInfo(code))
}

// 模式2: unsafe 块捕获
try (...) {
    // FFI 调用
} catch (e: Exception) {
    throw BusinessException(11105001, "Invalid parameter error.")
}
```

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构 |
| [04_External_API.md](04_External_API.md) | API 参考 |
| [08_Security_Review.md](08_Security_Review.md) | 安全评审 |
