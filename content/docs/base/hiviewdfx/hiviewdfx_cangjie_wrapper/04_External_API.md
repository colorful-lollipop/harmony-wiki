# 对外 API（N-API）

## 概述

本文档描述 hiviewdfx_cangjie_wrapper 对外暴露的 N-API 接口。所有 API 均为 Cangjie 接口，通过 FFI 调用 Native 实现。

**API Level**: 22+  
**系统能力**: SystemCapability.HiviewDFX.*

---

## PerformanceAnalysisKit

### 模块信息

| 属性 | 值 |
|------|-----|
| 包名 | `kit.PerformanceAnalysisKit` |
| 代码位置 | `kit/PerformanceAnalysisKit/index.cj` |
| 构建目标 | `kit.PerformanceAnalysisKit` |

### 导入方式

```cj
import kit.PerformanceAnalysisKit
```

### 子模块

| 子模块 | 导出内容 |
|--------|---------|
| `ohos.hi_trace_meter.*` | HiTraceMeter 类 |
| `ohos.hiviewdfx.hi_app_event.*` | HiAppEvent 及相关类型 |
| `ohos.hilog.*` | Hilog 类、LogLevel 枚举 |

**代码证据**: `kit/PerformanceAnalysisKit/index.cj:18-22`

---

## HiLog API

### LogLevel 枚举

**代码位置**: `ohos/hilog/hilog.cj:36-93`

| 枚举值 | 值 | 描述 |
|--------|-----|------|
| `Debug` | 3 | 调试级别 |
| `Info` | 4 | 信息级别 |
| `Warning` | 5 | 警告级别 |
| `Error` | 6 | 错误级别 |
| `Fatal` | 7 | 致命级别 |

**syscap**: `SystemCapability.HiviewDFX.HiLog`

---

### Hilog 类

**代码位置**: `ohos/hilog/hilog.cj:104-276`

#### isLoggable

检查指定标签和级别的日志是否可打印。

```cj
public static func isLoggable(domain: UInt32, tag: String, level: LogLevel): Bool
```

| 参数 | 类型 | 描述 |
|------|------|------|
| domain | UInt32 | 服务域，范围 0x0-0xFFFF |
| tag | String | 日志标签，最大 32 字节 |
| level | LogLevel | 日志级别 |

**返回值**: Bool - true 表示可打印，false 表示不可打印

**代码证据**: `ohos/hilog/hilog.cj:131-138`

---

#### debug

输出 Debug 级别日志。

```cj
public static func debug(domain: UInt32, tag: String, format: String,
                         args: Array<String>): Unit
```

| 参数 | 类型 | 描述 |
|------|------|------|
| domain | UInt32 | 服务域 |
| tag | String | 日志标签，最大 32 字节 |
| format | String | 格式字符串 |
| args | Array<String> | 格式化参数 |

**格式说明**:
- `%{public}s` - 公开内容
- `%{private}s` - 私有内容（受隐私开关控制）

**代码证据**: `ohos/hilog/hilog.cj:152-163`

---

#### info

输出 Info 级别日志。

```cj
public static func info(domain: UInt32, tag: String, format: String,
                        args: Array<String>): Unit
```

**同 debug**，仅日志级别不同

**代码证据**: `ohos/hilog/hilog.cj:177-194`

---

#### warn

输出 Warning 级别日志。

```cj
public static func warn(domain: UInt32, tag: String, format: String,
                        args: Array<String>): Unit
```

**同 debug**，仅日志级别不同

**代码证据**: `ohos/hilog/hilog.cj:208-225`

---

#### error

输出 Error 级别日志。

```cj
public static func error(domain: UInt32, tag: String, format: String,
                         args: Array<String>): Unit
```

**同 debug**，仅日志级别不同

**代码证据**: `ohos/hilog/hilog.cj:239-250`

---

#### fatal

输出 Fatal 级别日志。

```cj
public static func fatal(domain: UInt32, tag: String, format: String,
                         args: Array<String>): Unit
```

**同 debug**，仅日志级别不同

**代码证据**: `ohos/hilog/hilog.cj:264-275`

---

## HiTraceMeter API

### HiTraceMeter 类

**代码位置**: `ohos/hi_trace_meter/hi_trace_meter.cj:39-84`

#### startTrace

标记性能追踪开始。

```cj
public static func startTrace(name: String, taskId: Int32): Unit
```

| 参数 | 类型 | 描述 |
|------|------|------|
| name | String | 追踪任务名称 |
| taskId | Int32 | 任务唯一标识，用于与 finishTrace 配对 |

**说明**:
- 标记一个任务的开始
- 任务应持续至少 3ms
- 必须在同一 taskId 下调用 finishTrace

**syscap**: `SystemCapability.HiviewDFX.HiTrace`

**代码证据**: `ohos/hi_trace_meter/hi_trace_meter.cj:55-61`

---

#### finishTrace

标记性能追踪结束。

```cj
public static func finishTrace(name: String, taskId: Int32): Unit
```

| 参数 | 类型 | 描述 |
|------|------|------|
| name | String | 追踪任务名称（需与 startTrace 一致） |
| taskId | Int32 | 任务唯一标识（需与 startTrace 一致） |

**说明**:
- 标记一个任务的结束
- 必须与 startTrace 使用相同的 name 和 taskId

**syscap**: `SystemCapability.HiviewDFX.HiTrace`

**代码证据**: `ohos/hi_trace_meter/hi_trace_meter.cj:77-83`

---

## HiAppEvent API

### EventType 枚举

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:36-101`

| 枚举值 | 值 | 描述 |
|--------|-----|------|
| `Fault` | 1 | 故障事件 |
| `Statistic` | 2 | 统计事件 |
| `Security` | 3 | 安全事件 |
| `Behavior` | 4 | 行为事件 |

**syscap**: `SystemCapability.HiviewDFX.HiAppEvent`

---

### Domain 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:237-246`

| 属性 | 值 | 描述 |
|------|-----|------|
| `OS` | "OS" | 系统域 |

---

### Event 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:258-299`

| 属性 | 值 | 描述 |
|------|-----|------|
| `USER_LOGIN` | "hiappevent.user_login" | 用户登录事件 |
| `USER_LOGOUT` | "hiappevent.user_logout" | 用户登出事件 |
| `DISTRIBUTED_SERVICE_START` | "hiappevent.distributed_service_start" | 分布式服务启动 |
| `APP_CRASH` | "APP_CRASH" | 应用崩溃（系统事件） |
| `APP_FREEZE` | "APP_FREEZE" | 应用冻结（系统事件） |

---

### Param 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:308-333`

| 属性 | 值 | 描述 |
|------|-----|------|
| `USER_ID` | "user_id" | 自定义用户 ID |
| `DISTRIBUTED_SERVICE_NAME` | "ds_name" | 分布式服务名 |
| `DISTRIBUTED_SERVICE_INSTANCE_ID` | "ds_instance_id" | 分布式服务实例 ID |

---

### EventValueType 枚举

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:110-228`

| 枚举值 | 内部值 | 描述 |
|--------|--------|------|
| `IntValue(Int32)` | 0 | 整数值 |
| `FloatValue(Float64)` | 1 | 浮点值 |
| `StringValue(String)` | 2 | 字符串值 |
| `BoolValue(Bool)` | 3 | 布尔值 |
| `ArrString(Array<String>)` | 4 | 字符串数组 |
| `ArrInt32(Array<Int32>)` | 5 | 整型数组 |
| `ArrBool(Array<Bool>)` | 6 | 布尔数组 |
| `ArrFloat64(Array<Float64>)` | 7 | 浮点数组 |
| `Int64Value(Int64)` | 8 | 64位整数值 |
| `ArrInt64(Array<Int64>)` | 9 | 64位整型数组 |

---

### AppEventInfo 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:357-461`

#### 构造方法

```cj
public init(domain: String, name: String, event: EventType,
            params: HashMap<String, EventValueType>)
```

| 参数 | 类型 | 约束 |
|------|------|------|
| domain | String | 最多 32 字符，字母开头，不能以下划线结尾 |
| name | String | 最多 48 字符，字母或$开头，数字或字母结尾 |
| event | EventType | 事件类型 |
| params | HashMap | 最多 32 个参数 |

**代码证据**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:420-425`

---

### HiAppEvent 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:55-339`

#### write

写入事件。

```cj
public static func write(info: AppEventInfo): Unit
```

**异常**:
- `11100001` - 功能已禁用
- `11101001` - 无效事件域
- `11101002` - 无效事件名
- `11101003` - 无效事件参数数量
- `11101004` - 无效字符串长度
- `11101005` - 无效参数名
- `11101006` - 无效数组长度

**线程**: 工作线程执行 (`workerthread: true`)

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:82-91`

---

#### configure

配置事件功能。

```cj
public static func configure(config: ConfigOption): Unit
```

**异常**:
- `11103001` - 无效存储配额值

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:105-114`

---

#### addProcessor

添加数据处理器。

```cj
public static func addProcessor(processor: Processor): Int64
```

**返回值**: Int64 - 处理器 ID（失败返回 -1）

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:131-140`

---

#### removeProcessor

移除数据处理器。

```cj
public static func removeProcessor(id: Int64): Unit
```

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:152-159`

---

#### setUserId

设置用户 ID。

```cj
public static func setUserId(name: String, value: String): Unit
```

| 参数 | 类型 | 约束 |
|------|------|------|
| name | String | 最多 256 字符，字母/数字/_/$，不能以数字开头 |
| value | String | 最多 256 字符，为空或 null 清除 |

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:173-187`

---

#### getUserId

获取用户 ID。

```cj
public static func getUserId(name: String): String
```

**返回值**: String - 用户 ID，未找到返回空字符串

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:200-214`

---

#### setUserProperty

设置用户属性。

```cj
public static func setUserProperty(name: String, value: String): Unit
```

| 参数 | 类型 | 约束 |
|------|------|------|
| name | String | 最多 256 字符 |
| value | String | 最多 1024 字符 |

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:228-242`

---

#### getUserProperty

获取用户属性。

```cj
public static func getUserProperty(name: String): String
```

**返回值**: String - 用户属性，未找到返回空字符串

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:255-269`

---

#### clearData

清除本地日志数据。

```cj
public static func clearData(): Unit
```

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:279-281`

---

#### addWatcher

添加事件观察者。

```cj
public static func addWatcher(watcher: Watcher): Option<AppEventPackageHolder>
```

**异常**:
- `11102001` - 无效观察者名称
- `11102002` - 无效过滤事件域
- `11102003` - 无效行值
- `11102004` - 无效大小值
- `11102005` - 无效超时值

**返回值**: Option<AppEventPackageHolder> - 订阅数据持有者

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:301-314`

---

#### removeWatcher

移除事件观察者。

```cj
public static func removeWatcher(watcher: Watcher): Unit
```

**异常**:
- `11102001` - 无效观察者名称

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:328-338`

---

### ConfigOption 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:476-522`

| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| disable | Bool | false | 是否禁用事件日志功能 |
| maxStorage | String | "10M" | 存储配额，格式如 "10M"、"1G" |

**构造方法**:
```cj
public init(disable!: Bool = false, maxStorage!: String = "10M")
```

---

### Watcher 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:950-1029`

| 属性 | 类型 | 描述 |
|------|------|------|
| name | String | 观察者名称，最大 32 字符，字母开头 |
| triggerCondition | TriggerCondition | 触发条件 |
| appEventFilters | Array<AppEventFilter> | 事件过滤器 |
| onTrigger | Option<(Int32, Int32, AppEventPackageHolder) -> Unit> | 触发回调 |
| onReceive | Option<(String, Array<AppEventGroup>) -> Unit> | 实时订阅回调 |

---

### AppEventPackageHolder 类

**代码位置**: `ohos/hiviewdfx/hi_app_event/app_event_package_holder.cj:40-147`

#### 构造方法

```cj
public init(watcherName: String)
```

**异常**:
- `11105001` - 参数错误

**代码证据**: `app_event_package_holder.cj:62-72`

---

#### setSize

设置数据大小阈值。

```cj
public func setSize(size: Int32): Unit
```

| 参数 | 类型 | 约束 |
|------|------|------|
| size | Int32 | 范围 [0, 2^31-1] |

**异常**:
- `11104001` - 无效大小值
- `11105001` - 参数错误

**代码证据**: `app_event_package_holder.cj:87-94`

---

#### takeNext

获取下一个事件包。

```cj
public func takeNext(): Option<AppEventPackage>
```

**返回值**: Option<AppEventPackage> - 事件包，已取完返回 None

**代码证据**: `app_event_package_holder.cj:111-131`

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [03_Architecture.md](03_Architecture.md) | 架构说明 |
| [05_Internal_API.md](05_Internal_API.md) | 内部 API |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题 |
