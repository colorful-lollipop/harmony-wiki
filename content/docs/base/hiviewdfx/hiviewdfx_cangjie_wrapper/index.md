# hiviewdfx_cangjie_wrapper - DFX Cangjie 封装

## 项目简介

`hiviewdfx_cangjie_wrapper` 是 OpenHarmony DFX (Design for X) 能力的 **Cangjie 语言封装**，为 Cangjie 开发者提供 DF（Design for Reliability，可靠性设计）和 DT（Design for Testability，可测试性设计）能力。

**项目状态**: Beta  
**支持设备**: standard (标准设备)  
**代码仓库**: `base/hiviewdfx/hiviewdfx_cangjie_wrapper`

---

## 核心能力

本项目提供三大核心 DFX 能力：

### 1. HiLog - 流水日志系统

提供灵活的日志输出能力，支持多级别日志打印。

**功能特性**:
- 多级别日志: Debug / Info / Warning / Error / Fatal
- 日志级别检查: `isLoggable()`
- 格式字符串支持: `%{public}s`, `%{private}s`
- 线程安全的日志打印

**代码位置**: `ohos/hilog/hilog.cj:104-276`

**使用示例**:
```cj
import ohos.hilog.{Hilog, LogLevel}

// 检查日志级别
Hilog.isLoggable(0xD002800, "MyTag", LogLevel.Info)

// 输出日志
Hilog.info(0xD002800, "MyTag", "User login: %{public}s", [userName])
```

### 2. HiAppEvent - 应用事件打点

提供应用事件打点和订阅能力，用于应用质量分析。

**功能特性**:
- 事件定义与打点: Fault / Statistic / Security / Behavior
- 事件订阅: Watcher 机制
- 数据处理器: Processor 云端上报
- 用户属性管理: UserId / UserProperty
- 事件存储与清理

**代码位置**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:55-339`

**使用示例**:
```cj
import ohos.hiviewdfx.hi_app_event.{HiAppEvent, AppEventInfo, EventType}

// 写入事件
let info = AppEventInfo("USER_OPERATION", "button_click", EventType.Behavior,
                        HashMap<String, EventValueType>())
HiAppEvent.write(info)

// 订阅事件
let watcher = Watcher("MyWatcher", onTrigger: Some((row, size, holder) => {
    // 处理事件
}))
HiAppEvent.addWatcher(watcher)
```

### 3. HiTraceMeter - 性能打点

提供性能追踪能力，支持通过 bytrace 工具捕获打点数据。

**功能特性**:
- 异步追踪: startTrace / finishTrace
- 性能分析: 最小 3ms 的任务追踪
- 任务匹配: 通过 taskId 关联起止点

**代码位置**: `ohos/hi_trace_meter/hi_trace_meter.cj:39-84`

**使用示例**:
```cj
import ohos.hi_trace_meter.HiTraceMeter

HiTraceMeter.startTrace("network_request", 1001)
// 业务逻辑
HiTraceMeter.finishTrace("network_request", 1001)
```

---

## 快速入门

### 环境要求

- OpenHarmony SDK API Level 22+
- Cangjie 编译器
- standard 设备目标

### 添加依赖

在 `bundle.json` 中添加依赖：
```json
{
    "module": {
        "requestPermissions": [
            {"name": "SystemCapability.HiviewDFX.HiLog"},
            {"name": "SystemCapability.HiviewDFX.HiAppEvent"},
            {"name": "SystemCapability.HiviewDFX.HiTrace"}
        ]
    }
}
```

### 导入模块

```cj
// 方式1: 导入完整Kit
import kit.PerformanceAnalysisKit

// 方式2: 单独导入模块
import ohos.hilog.Hilog
import ohos.hiviewdfx.hi_app_event.HiAppEvent
import ohos.hi_trace_meter.HiTraceMeter
```

---

## 性能指标

| 指标 | 数值 | 说明 |
|------|------|------|
| ROM占用 | 400KB | 编译产物大小 |
| RAM占用 | 420KB | 运行时内存 |
| API Level | 22 | 最低系统版本 |

**数据来源**: `bundle.json:20-21`

---

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                    Cangjie Application                        │
├─────────────────────────────────────────────────────────────┤
│  kit.PerformanceAnalysisKit (统一入口)                        │
│  ├── ohos.hilog.*                                            │
│  ├── ohos.hiviewdfx.hi_app_event.*                           │
│  └── ohos.hi_trace_meter.*                                   │
├─────────────────────────────────────────────────────────────┤
│                    FFI Bridge                                 │
│  ├── hilog: libhilog                                         │
│  ├── hiappevent: cj_hiappevent_ffi                           │
│  └── hitrace: cj_hitracemeter_ffi                             │
├─────────────────────────────────────────────────────────────┤
│                    Native Services                            │
│  ├── HiLog Service                                           │
│  ├── HiAppEvent Service                                      │
│  └── HiTrace Service                                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [README.md](README.md) | Wiki 说明与导航 |
| [SUMMARY.md](SUMMARY.md) | 全站导航 |
| [01_Overview.md](01_Overview.md) | 详细项目概览 |
| [04_External_API.md](04_External_API.md) | API 参考手册 |

---

## 相关链接

- **项目仓库**: `base/hiviewdfx/hiviewdfx_cangjie_wrapper`
- **README**: [English](../../README.md) / [中文](../../README_zh.md)
- **API 文档**: [Performance Analysis Kit API Reference](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- **开发指南**: [Cangjie DFX 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
