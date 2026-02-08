# 项目概览

## 项目定位

`hiviewdfx_cangjie_wrapper` 是 OpenHarmony DFX 能力的 **Cangjie 语言封装层**，负责将 Native 层的 DFX 能力以类型安全的方式暴露给 Cangjie 开发者。

**项目目标**:
- 为 Cangjie 应用提供 DFX 开发能力
- 保持与 ArkTS API 的功能对等（Beta 阶段）
- 提供良好的开发体验和类型安全

**项目性质**: Beta 特性（`README.md:1`）

---

## 核心能力详解

### 1. HiLog - 流水日志系统

#### 能力描述

HiLog 是 OpenHarmony 的日志系统，允许应用按照指定的级别、标识和格式字符串输出日志内容。

**代码位置**: `ohos/hilog/hilog.cj:104-276`

#### 功能特性

| 特性 | 描述 | 代码位置 |
|------|------|---------|
| 日志级别 | 支持 Debug/Info/Warning/Error/Fatal 五个级别 | `hilog.cj:36-81` |
| 级别检查 | isLoggable() 判断日志是否可打印 | `hilog.cj:131-138` |
| 格式化输出 | 支持 public/private 格式说明符 | `hilog.cj:288-340` |
| 异常处理 | 自动捕获未处理异常并打印 | `hilog.cj:106-117` |

#### 日志级别定义

```cj
// 代码位置: hilog.cj:36-81
public enum LogLevel {
    Debug    // 值为 3
    Info     // 值为 4
    Warning  // 值为 5
    Error    // 值为 6
    Fatal    // 值为 7
}
```

#### syscap 要求

```cj
SystemCapability.HiviewDFX.HiLog
```

---

### 2. HiAppEvent - 应用事件打点

#### 能力描述

HiAppEvent 提供应用事件打点和订阅能力，支持故障、统计、安全和用户行为事件的记录与分析。

**代码位置**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:55-339`

#### 功能特性

| 特性 | 描述 | 代码位置 |
|------|------|---------|
| 事件打点 | write() 写入各类事件 | `hi_app_event.cj:82-91` |
| 事件配置 | configure() 配置存储和行为 | `hi_app_event.cj:105-114` |
| 数据处理 | addProcessor/removeProcessor | `hi_app_event.cj:131-159` |
| 用户属性 | setUserId/getUserId 等 | `hi_app_event.cj:173-269` |
| 事件订阅 | addWatcher/removeWatcher | `hi_app_event.cj:301-338` |
| 数据清除 | clearData() 清除本地数据 | `hi_app_event.cj:279-281` |

#### 事件类型

```cj
// 代码位置: cj_event.cj:36-72
public enum EventType {
    Fault      // 故障事件，值为 1
    Statistic  // 统计事件，值为 2
    Security   // 安全事件，值为 3
    Behavior   // 行为事件，值为 4
}
```

#### syscap 要求

```cj
SystemCapability.HiviewDFX.HiAppEvent
```

---

### 3. HiTraceMeter - 性能打点

#### 能力描述

HiTraceMeter 提供性能追踪能力，用于标记和测量关键业务流程的执行时间。

**代码位置**: `ohos/hi_trace_meter/hi_trace_meter.cj:39-84`

#### 功能特性

| 特性 | 描述 | 代码位置 |
|------|------|---------|
| 开始追踪 | startTrace() 标记任务开始 | `hi_trace_meter.cj:55-61` |
| 结束追踪 | finishTrace() 标记任务结束 | `hi_trace_meter.cj:77-83` |
| 异步追踪 | 支持同一 taskId 的多次调用 | API 设计 |
| 最小粒度 | 建议追踪超过 3ms 的任务 | `hi_trace_meter.cj:29-34` |

#### syscap 要求

```cj
SystemCapability.HiviewDFX.HiTrace
```

---

## 系统能力要求

### 最小 API Level

所有 API 均要求 **API Level 22** 或更高版本。

```cj
//!APILevel[since: "22", ...]
```

### 系统能力声明

在 `bundle.json` 中声明所需系统能力：

```json
{
    "adapted_system_type": ["standard"],
    "syscap": [
        "SystemCapability.HiviewDFX.HiLog",
        "SystemCapability.HiviewDFX.HiAppEvent",
        "SystemCapability.HiviewDFX.HiTrace"
    ]
}
```

**数据来源**: `bundle.json:17-28`

---

## 依赖关系

### 外部依赖

| 依赖组件 | 用途 | 数据来源 |
|---------|------|---------|
| `cangjie_ark_interop` | Cangjie FFI 基础类 | `cj_external_deps` 配置 |
| `hiappevent` | 应用事件 Native 服务 | `external_deps` 配置 |
| `hilog` | 日志 Native 服务 | `external_deps` 配置 |
| `hitrace` | 性能追踪 Native 服务 | `external_deps` 配置 |

### 内部模块依赖

```
kit.PerformanceAnalysisKit
├── ohos.hi_trace_meter (可选依赖)
├── ohos.hilog (HiAppEvent 的日志依赖)
└── ohos.hiviewdfx.hi_app_event (可选依赖)
```

**数据来源**: `kit/PerformanceAnalysisKit/BUILD.gn:22-26`

---

## 版本信息

| 项目 | 版本 | 说明 |
|------|------|------|
| 项目版本 | 6.1 | `bundle.json:4` |
| API Level | 22 | 最低支持版本 |
| ROM | 400KB | 编译产物大小 |
| RAM | 420KB | 运行时内存 |

**数据来源**: `bundle.json:20-21`

---

## 与 ArkTS 的差异

### 已知限制

| 特性 | ArkTS | Cangjie | 状态 |
|------|-------|---------|------|
| HiLog | ✅ | ✅ | 支持 |
| HiAppEvent | ✅ | ✅ | 支持 |
| HiTraceMeter | ✅ | ✅ | 支持 |
| HiView | ✅ | ❌ | Beta不支持 |
| FaultLoggerd | ✅ | ❌ | Beta不支持 |

**数据来源**: `README_zh.md:66-70`

---

## 适用场景

### 1. 应用调试

使用 HiLog 输出调试信息：
```cj
Hilog.info(domain, tag, "Operation completed: %{public}s", [result])
```

### 2. 质量监控

使用 HiAppEvent 收集关键事件：
```cj
let event = AppEventInfo("USER_ACTION", "login", EventType.Behavior, params)
HiAppEvent.write(event)
```

### 3. 性能分析

使用 HiTraceMeter 追踪关键路径：
```cj
HiTraceMeter.startTrace("data_loading", taskId)
// 加载数据
HiTraceMeter.finishTrace("data_loading", taskId)
```

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [index.md](index.md) | 项目首页 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构 |
| [04_External_API.md](04_External_API.md) | API 参考 |
| [README.md](README.md) | Wiki 导航 |
