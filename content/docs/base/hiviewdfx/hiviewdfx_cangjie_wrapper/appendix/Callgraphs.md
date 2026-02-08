# 关键调用链

## 概述

本文档描述 hiviewdfx_cangjie_wrapper 的关键调用链，包括 API 调用路径、数据流和模块间交互。

---

## HiLog 调用链

### 日志输出调用链

```
Cangjie 应用
    │
    ▼
Hilog.info/debug/warn/error/fatal(domain, tag, format, args)
    │
    ├─► isLoggable() [可选检查]
    │   │
    │   ▼
    │   HiLogIsLoggable(domain, tag, level) [FFI]
    │       │
    │       ▼
    │   libhilog.so (Native)
    │       │
    │       ▼
    │   HiLog Service → 日志输出
    │
    ▼
parseLogContent(format, args)
    │
    ▼
LibC.mallocCString() → CString 转换
    │
    ▼
HiLogPrint(..., format, content) [FFI]
    │
    ▼
libhilog.so (Native)
    │
    ▼
HiLog Service → 输出到日志系统
```

**入口**: `ohos/hilog/hilog.cj:177` (info 方法)

**关键函数**:
- `parseLogContent()` - `ohos/hilog/hilog.cj:288`
- `HiLogIsLoggable()` - `ohos/hilog/hilog.cj:23`
- `HiLogPrint()` - `ohos/hilog/hilog.cj:25`

---

### 异常处理调用链

```
Thread Uncaught Exception
        │
        ▼
Hilog.init() [静态初始化]
        │
        ▼
Thread.handleUncaughtExceptionBy { ... }
        │
        ├─► Hilog.error("An exception has occurred:")
        │       │
        │       ▼
        │   HiLogPrint() → 日志输出
        │
        └─► Hilog.error(exception.toString())
                │
                ▼
            HiLogPrint() → 日志输出
```

**入口**: `ohos/hilog/hilog.cj:106`

---

## HiTraceMeter 调用链

### 性能追踪调用链

```
Cangjie 应用
        │
        ▼
HiTraceMeter.startTrace(name, taskId)
        │
        ▼
LibC.mallocCString(name) → CString 转换
        │
        ▼
FfiOHOSHiTraceStartAsyncTrace(name, taskId) [FFI]
        │
        ▼
cj_hitracemeter_ffi.so (Native)
        │
        ▼
HiTrace Service → bytrace 记录开始点
        │
        ▼
[Cangjie 应用执行业务逻辑]
        │
        ▼
HiTraceMeter.finishTrace(name, taskId)
        │
        ▼
LibC.mallocCString(name) → CString 转换
        │
        ▼
FfiOHOSHiTraceFinishAsyncTrace(name, taskId) [FFI]
        │
        ▼
cj_hitracemeter_ffi.so (Native)
        │
        ▼
HiTrace Service → bytrace 记录结束点
```

**入口**: `ohos/hi_trace_meter/hi_trace_meter.cj:55` (startTrace)

**关键函数**:
- `FfiOHOSHiTraceStartAsyncTrace()` - `ohos/hi_trace_meter/hi_trace_meter.cj:23`
- `FfiOHOSHiTraceFinishAsyncTrace()` - `ohos/hi_trace_meter/hi_trace_meter.cj:25`

---

## HiAppEvent 调用链

### 事件写入调用链

```
Cangjie 应用
        │
        ▼
HiAppEvent.write(AppEventInfo)
        │
        ├─► AppEventInfo.toCAppEventInfo()
        │       │
        │       ├─► LibC.mallocCString(domain)
        │       ├─► LibC.mallocCString(name)
        │       └─► toCArrParameters(params)
        │               │
        │               ▼
        │           cjArr2CArr() → C 数组转换
        │
        ▼
FfiOHOSHiAppEventWrite(cinfo) [FFI]
        │
        ▼
cj_hiappevent_ffi.so (Native)
        │
        ▼
HiAppEvent Service → 事件存储
```

**入口**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:82`

**关键函数**:
- `AppEventInfo.toCAppEventInfo()` - `ohos/hiviewdfx/hi_app_event/cj_event.cj:437`
- `toCArrParameters()` - `ohos/hiviewdfx/hi_app_event/cj_event.cj:463`
- `FfiOHOSHiAppEventWrite()` - `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:26`

---

### 事件订阅调用链

```
Cangjie 应用
        │
        ▼
HiAppEvent.addWatcher(Watcher)
        │
        ├─► RetWatcher(watcher) → C 结构转换
        │
        ▼
FfiOHOSHiAppEventAddWatcher(retWatcher) [FFI]
        │
        ▼
cj_hiappevent_ffi.so (Native)
        │
        ▼
HiAppEvent Service → 注册 Watcher
        │
        ▼
[事件触发]
        │
        ▼
onTrigger callback 回调
        │
        ▼
AppEventPackageHolder.takeNext()
        │
        ▼
FfiOHOSHiAppEventTakeNext(id) [FFI]
        │
        ▼
读取事件数据 → 返回 AppEventPackage
```

**入口**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:301`

**关键函数**:
- `FfiOHOSHiAppEventAddWatcher()` - `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:42`
- `FfiOHOSHiAppEventTakeNext()` - `ohos/hiviewdfx/hi_app_event/app_event_package_holder.cj:30`

---

### 配置调用链

```
Cangjie 应用
        │
        ▼
HiAppEvent.configure(ConfigOption)
        │
        ├─► CConfigOption(config) → C 结构转换
        │
        ▼
FfiOHOSHiAppEventConfigure(config) [FFI]
        │
        ▼
cj_hiappevent_ffi.so (Native)
        │
        ▼
HiAppEvent Service → 更新配置
```

**入口**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:105`

---

## 模块间调用关系

### 初始化调用链

```
kit.PerformanceAnalysisKit
        │
        ▼
import ohos.hi_trace_meter.*
import ohos.hiviewdfx.hi_app_event.*
import ohos.hilog.*
        │
        ▼
[运行时加载各模块 .so]
        │
        ▼
Hilog.init() [静态初始化]
        │
        ▼
Thread.handleUncaughtExceptionBy → 注册异常处理
```

---

### 日志依赖调用链

```
HiAppEvent 模块
        │
        ▼
import ohos.hilog.* (通过 cj_deps)
        │
        ▼
HI_APP_EVENT_LOG.error(...) [内部日志]
        │
        ▼
HiLogPrint() → 输出错误日志
```

**证据**: `ohos/hiviewdfx/hi_app_event/BUILD.gn:40`

---

## 资源生命周期

### CString 资源管理

```
LibC.mallocCString(str)
        │
        ▼
[分配 C 内存]
        │
        ▼
try (cStr = mallocCString(...).asResource()) {
        │
        ▼
    [使用 CString]
        │
        ▼
[自动释放内存]
```

**示例**: `ohos/hilog/hilog.cj:154-162`

---

## 并发调用链

### HiLog 并发调用

```
Thread 1: Hilog.info(...)
        │
        ▼
HiLogPrint() [线程安全]

Thread 2: Hilog.error(...)
        │
        ▼
HiLogPrint() [线程安全]

[Native 层保证线程安全]
```

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [03_Architecture.md](../03_Architecture.md) | 架构说明 |
| [04_External_API.md](../04_External_API.md) | API 参考 |
| [05_Internal_API.md](../05_Internal_API.md) | 内部 API |
