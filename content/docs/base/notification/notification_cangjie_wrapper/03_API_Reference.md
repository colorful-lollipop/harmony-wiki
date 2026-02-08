# API 参考

## CommonEventManager

公共事件管理器，提供发布、订阅、取消订阅等核心能力。

```cangjie
package ohos.common_event_manager

public class CommonEventManager {
    // 静态方法，无需实例化
}
```

### API 清单

| 方法 | 签名 | 同步/异步 | 线程 | 异常 | 起始版本 |
|------|------|---------|------|------|----------|
| `publish` | `func publish(event: String, options!: CommonEventPublishData): Unit` | 异步 | worker | 1500003, 1500007-1500009 | 22 |
| `createSubscriber` | `func createSubscriber(info: CommonEventSubscribeInfo): CommonEventSubscriber` | 异步 | worker | 1500008 | 22 |
| `subscribe` | `func subscribe(sub: CommonEventSubscriber, callback: AsyncCallback<CommonEventData>): Unit` | 异步 | main | 801, 1500007-1500010 | 22 |
| `unsubscribe` | `func unsubscribe(sub: CommonEventSubscriber): Unit` | 异步 | worker | 801, 1500007-1500008 | 22 |

---

### publish

发布一个公共事件。

```cangjie
CommonEventManager.publish(event: String, options!: CommonEventPublishData): Unit
```

**参数**

| 名称 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `event` | String | 是 | - | 事件名称，可使用 `Support` 常量 |
| `options` | CommonEventPublishData | 否 | `CommonEventPublishData()` | 事件发布属性 |

**错误码**

| 错误码 | 含义 |
|--------|------|
| 1500003 | 公共事件发送频率过高 |
| 1500007 | 向公共事件服务发送消息失败 |
| 1500008 | 公共事件服务初始化失败 |
| 1500009 | 获取系统参数失败 |

**示例**

```cangjie
import ohos.common_event_manager.{CommonEventManager, Support}

// 发布系统启动完成事件
CommonEventManager.publish(Support.COMMON_EVENT_BOOT_COMPLETED)

// 发布带数据的自定义事件
let publishData = CommonEventPublishData(
    data: "test message",
    code: 100
)
CommonEventManager.publish("com.example.MY_EVENT", options: publishData)
```

---

### createSubscriber

创建公共事件订阅者。

```cangjie
CommonEventManager.createSubscriber(subscribeInfo: CommonEventSubscribeInfo): CommonEventSubscriber
```

**参数**

| 名称 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `subscribeInfo` | CommonEventSubscribeInfo | 是 | 订阅信息，包含订阅的事件列表 |

**返回值**

| 类型 | 描述 |
|------|------|
| CommonEventSubscriber | 订阅者实例，用于执行订阅操作 |

**错误码**

| 错误码 | 含义 |
|--------|------|
| 1500008 | 公共事件服务未完成初始化 |

**示例**

```cangjie
import ohos.common_event_manager.{CommonEventManager, Support}
import ohos.common_event_subscribe_info.CommonEventSubscribeInfo

let subscribeInfo = CommonEventSubscribeInfo(
    events: [
        Support.COMMON_EVENT_BATTERY_CHANGED,
        Support.COMMON_EVENT_POWER_CONNECTED
    ],
    priority: 100
)
let subscriber = CommonEventManager.createSubscriber(subscribeInfo)
```

---

### subscribe

订阅公共事件。

```cangjie
CommonEventManager.subscribe(subscriber: CommonEventSubscriber, callback: AsyncCallback<CommonEventData>): Unit
```

**参数**

| 名称 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `subscriber` | CommonEventSubscriber | 是 | 已创建的订阅者实例 |
| `callback` | AsyncCallback\<CommonEventData\> | 是 | 事件到达时的回调 |

**错误码**

| 错误码 | 含义 |
|--------|------|
| 801 | 能力不支持 |
| 1500007 | 向公共事件服务发送消息失败 |
| 1500008 | 公共事件服务初始化失败 |
| 1500010 | 订阅者数量超过系统规格 |

**示例**

```cangjie
import ohos.common_event_manager.{CommonEventManager, Support}
import ohos.common_event_subscribe_info.CommonEventSubscribeInfo
import ohos.business_exception.AsyncCallback

let subscribeInfo = CommonEventSubscribeInfo(
    events: [Support.COMMON_EVENT_SCREEN_ON]
)
let subscriber = CommonEventManager.createSubscriber(subscribeInfo)

let callback = AsyncCallback<CommonEventData> { err, data ->
    if let e = err {
        println("Error: ${e}")
    } else if let event = data {
        println("Received event: ${event.event}")
        println("Code: ${event.code}, Data: ${event.data}")
    }
}

CommonEventManager.subscribe(subscriber, callback: callback)
```

**注意**: 回调在主线程执行。

---

### unsubscribe

取消订阅公共事件。

```cangjie
CommonEventManager.unsubscribe(subscriber: CommonEventSubscriber): Unit
```

**参数**

| 名称 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `subscriber` | CommonEventSubscriber | 是 | 要取消的订阅者实例 |

**错误码**

| 错误码 | 含义 |
|--------|------|
| 801 | 能力不支持 |
| 1500007 | 向公共事件服务发送消息失败 |
| 1500008 | 公共事件服务初始化失败 |

**示例**

```cangjie
// 取消订阅
CommonEventManager.unsubscribe(subscriber)

// 订阅者会被自动释放
```

---

## CommonEventData

公共事件数据，接收事件时使用。

```cangjie
package ohos.common_event_data

public class CommonEventData {
    public var event: String
    public var bundleName: String
    public var code: Int32
    public var data: String
    public var parameters: HashMap<String, CommonEventValueType>
}
```

### 属性

| 属性 | 类型 | 只读 | 描述 |
|------|------|------|------|
| `event` | String | 否 | 事件名称 |
| `bundleName` | String | 是 | 发布者包名 |
| `code` | Int32 | 是 | 事件码 |
| `data` | String | 是 | 事件数据 |
| `parameters` | HashMap\<String, CommonEventValueType\> | 是 | 扩展参数 |

---

## CommonEventPublishData

公共事件发布数据。

```cangjie
package ohos.common_event_publish_data

public class CommonEventPublishData {
    public var bundleName: String
    public var code: Int32
    public var data: String
    public var subscriberPermissions: Array<String>
    public var isOrdered: Bool
    public var isSticky: Bool
    public var parameters: HashMap<String, CommonEventValueType>
}
```

### 构造函数

```cangjie
public init(
    bundleName!: String = "",
    data!: String = "",
    code!: Int32 = 0,
    subscriberPermissions!: Array<String> = [],
    isOrdered!: Bool = false,
    isSticky!: Bool = false,
    parameters!: HashMap<String, CommonEventValueType> = []
)
```

### 属性

| 属性 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `bundleName` | String | 否 | "" | 目标订阅者包名限制 |
| `code` | Int32 | 否 | 0 | 事件码 |
| `data` | String | 否 | "" | 事件数据 (最大 64KB) |
| `subscriberPermissions` | Array\<String\> | 否 | [] | 订阅者权限要求 |
| `isOrdered` | Bool | 否 | false | 是否有序事件 |
| `isSticky` | Bool | 否 | false | 是否粘性事件 (需要权限) |
| `parameters` | HashMap\<String, CommonEventValueType\> | 否 | {} | 扩展参数 |

### 权限声明

`isSticky = true` 时需要权限：

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.COMMONEVENT_STICKY",
    syscap: "SystemCapability.Notification.CommonEvent"
]
public var isSticky: Bool
```

---

## CommonEventSubscribeInfo

公共事件订阅信息。

```cangjie
package ohos.common_event_subscribe_info

public class CommonEventSubscribeInfo {
    public var events: Array<String>
    public var publisherPermission: String
    public var publisherDeviceId: String
    public var userId: Int32
    public var priority: Int32
    public var publisherBundleName: String
}
```

### 构造函数

```cangjie
public init(
    events: Array<String>,
    publisherPermission!: String = "",
    publisherDeviceId!: String = "",
    userId!: Int32 = -3,  // UNDEFINED_USER
    priority!: Int32 = 0,
    publisherBundleName!: String = ""
)
```

### 属性

| 属性 | 类型 | 范围 | 描述 |
|------|------|------|------|
| `events` | Array\<String\> | - | 订阅的事件列表 |
| `publisherPermission` | String | - | 发布者权限要求 |
| `publisherDeviceId` | String | - | 发布者设备 ID (暂不支持) |
| `userId` | Int32 | 有效用户 ID | 用户 ID，不指定则使用当前用户 |
| `priority` | Int32 | -100 ~ 1000 | 订阅者优先级 |
| `publisherBundleName` | String | - | 发布者包名限制 |

### priority 范围限制

```cangjie
// common_event_subscribe_info.cj:160-170
func checkPriority(priority: Int32): Int32 {
    let maxValue = 1000i32
    let minValue = -100i32
    if (priority > maxValue) { maxValue }
    else if (priority < minValue) { minValue }
    else { priority }
}
```

---

## CommonEventSubscriber

公共事件订阅者。

```cangjie
package ohos.common_event_subscriber

public class CommonEventSubscriber <: RemoteDataLite {
    protected init(id: Int64)
    public func getID(): Int64
}
```

### 继承

- `RemoteDataLite` - 远程数据生命周期管理

### 生命周期

| 操作 | 触发 |
|------|------|
| 创建 | `CommonEventManager.createSubscriber()` |
| 订阅 | `CommonEventManager.subscribe()` |
| 取消订阅 | `CommonEventManager.unsubscribe()` |
| 资源释放 | 析构时调用 `releaseFFIData()` |

---

## Support

系统公共事件常量定义。

```cangjie
package ohos.common_event_manager

public class Support {
    public static const COMMON_EVENT_BOOT_COMPLETED: String
    public static const COMMON_EVENT_BATTERY_CHANGED: String
    // ... 150+ 常量
}
```

### 事件分类

| 分类 | 前缀 | 示例 |
|------|------|------|
| 系统事件 | `usual.event.*` | `BOOT_COMPLETED`, `SCREEN_ON` |
| 应用事件 | `usual.event.PACKAGE_*` | `PACKAGE_ADDED`, `PACKAGE_REMOVED` |
| 电源事件 | `usual.event.BATTERY_*` | `BATTERY_CHANGED`, `BATTERY_LOW` |
| 网络事件 | `usual.event.wifi.*` | `WIFI_POWER_STATE`, `WIFI_SCAN_FINISHED` |
| 蓝牙事件 | `usual.event.bluetooth.*` | `BLUETOOTH_HOST_STATE_UPDATE` |
| NFC 事件 | `usual.event.nfc.*` | `NFC_ACTION_ADAPTER_STATE_CHANGED` |
| 用户事件 | `usual.event.USER_*` | `USER_STARTED`, `USER_SWITCHED` |

### 完整常量列表

支持 **150+** 系统公共事件常量，详见 [mock/ohos.common_event_manager.cj](https://gitee.com/openharmony/notification_notification_cangjie_wrapper/blob/master/mock/ohos.common_event_manager.cj)。

---

## CommonEventValueType

事件参数的值的类型枚举。

```cangjie
package ohos.value_type

public enum CommonEventValueType {
    Int32Value(Int32)
    | Float64Value(Float64)
    | StringValue(String)
    | BoolValue(Bool)
    | FD(Int32)
    | ArrayString(Array<String>)
    | ArrayInt32(Array<Int32>)
    | ArrayInt64(Array<Int64>)
    | ArrayBool(Array<Bool>)
    | ArrayFloat64(Array<Float64>)
    | ArrayFD(Array<Int32>)
}
```

### 使用示例

```cangjie
import ohos.value_type.CommonEventValueType
import std.collection.HashMap

let params = HashMap<String, CommonEventValueType>()
params.add("intVal", CommonEventValueType.Int32Value(42))
params.add("strVal", CommonEventValueType.StringValue("hello"))
params.add("arrayVal", CommonEventValueType.ArrayInt32([1, 2, 3]))

let publishData = CommonEventPublishData(parameters: params)
```
