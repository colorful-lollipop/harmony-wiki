# 接口文档

本文档详细描述 `request_cangjie_wrapper` 的对外接口，包括 N-API 清单、配置说明和事件回调机制。

---

## 4.1 N-API 入口

### Task 类接口

**文件**: `ohos/request/agent.cj:1661`

```cj
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Request.FileTransferAgent"
]
public class Task {
    /**
     * 任务 ID，系统唯一
     */
    public let tid: String

    /**
     * 任务配置
     */
    public var config: Config
}
```

### 构造函数

**文件**: `ohos/request/agent.cj:1690-1693`

```cj
/**
 * Task 构造函数
 *
 * @param { String } tid - 任务 ID，应用自行指定，需保证唯一性
 * @param { Config } config - 任务配置
 */
public init(tid: String, config: Config)
```

**示例**:
```cj
let config = Config(
    action: Action.Download,
    url: "https://example.com/file.zip",
    saveas: "./downloads/file.zip"
)
let task = Task("my-download-task", config)
```

---

## 4.2 Config 类

### 配置字段

**文件**: `ohos/request/agent.cj:578-932`

| 字段 | 类型 | 必填 | 默认值 | 最大长度 | 说明 |
|------|------|------|--------|----------|------|
| action | Action | 是 | - | - | 任务类型 (Upload/Download) |
| url | String | 是 | - | 8192 | 请求 URL |
| title | ?String | 否 | action 名称 | 256 | 任务标题 |
| description | String | 否 | "" | 1024 | 任务描述 |
| mode | Mode | 否 | Background | - | 运行模式 |
| overwrite | Bool | 否 | false | - | 是否覆盖已存在文件 |
| method | ?String | 否 | 自动推断 | - | HTTP 方法 |
| headers | HashMap | 否 | {} | - | HTTP 请求头 |
| data | ?ConfigData | 否 | None | - | 请求数据 |
| saveas | String | 否 | "./" | - | 下载保存路径 |
| network | Network | 否 | AnyType | - | 网络类型限制 |
| metered | Bool | 否 | false | - | 计量网络 |
| roaming | Bool | 否 | true | - | 漫游 |
| retry | Bool | 否 | true | - | 自动重试 |
| redirect | Bool | 否 | true | - | 重定向 |
| index | UInt32 | 否 | 0 | - | 任务索引 |
| begins | Int64 | 否 | 0 | - | 起始偏移 |
| ends | Int64 | 否 | -1 | - | 结束偏移 (-1 表示结束) |
| gauge | Bool | 否 | false | - | 进度通知策略 |
| precise | Bool | 否 | false | - | 精确进度 |
| token | ?String | 否 | None | - | 任务令牌 |
| priority | UInt32 | 否 | 0 | - | 任务优先级 |
| extras | HashMap | 否 | {} | - | 扩展字段 |

### 构造函数

**文件**: `ohos/request/agent.cj:868-905`

```cj
public init(
    action: Action,
    url: String,
    title!: ?String = None,
    description!: String = "",
    mode!: Mode = Mode.Background,
    overwrite!: Bool = false,
    method!: ?String = None,
    headers!: HashMap<String, String> = HashMap<String, String>(),
    data!: ?ConfigData = None,
    saveas!: String = "./",
    network!: Network = Network.AnyType,
    metered!: Bool = false,
    roaming!: Bool = true,
    retry!: Bool = true,
    redirect!: Bool = true,
    index!: UInt32 = 0,
    begins!: Int64 = 0,
    ends!: Int64 = -1,
    gauge!: Bool = false,
    precise!: Bool = false,
    token!: ?String = None,
    priority!: UInt32 = 0,
    extras!: HashMap<String, String> = HashMap<String, String>()
)
```

---

## 4.3 事件回调

### on() - 订阅事件

**文件**: `ohos/request/agent.cj:1756-1787`

```cj
/**
 * 订阅任务进度、状态变更事件
 *
 * @param { EventCallbackType } event - 事件类型
 * @param { Callback1Argument<Progress> } callback - 回调函数
 * @throws { BusinessException } 参数错误时抛出
 */
public func on(event: EventCallbackType, callback: Callback1Argument<Progress>): Unit
```

**支持的事件类型**:
| 事件 | 说明 |
|------|------|
| Progress | 下载/上传进度更新 |
| Completed | 任务完成 |
| Failed | 任务失败 |
| Pause | 任务暂停 |
| Resume | 任务恢复 |
| Remove | 任务移除 |

**示例**:
```cj
task.on(.Progress) { progress =>
    let percent = (progress.processed * 100) / progress.sizes[0]
    println("进度: ${percent}%")
}

task.on(.Completed) { _ =>
    println("下载完成!")
}
```

### on(Response) - 订阅响应头

**文件**: `ohos/request/agent.cj:1715-1742`

```cj
/**
 * 订阅 HTTP 响应头事件
 *
 * @param { EventCallbackType } event - 固定为 Response
 * @param { Callback1Argument<HttpResponse> } callback - 回调函数
 */
public func on(event: EventCallbackType, callback: Callback1Argument<HttpResponse>): Unit
```

**示例**:
```cj
task.on(.Response) { response =>
    println("HTTP ${response.version} ${response.statusCode} ${response.reason}")
    for ((k, v) in response.headers) {
        println("Header: ${k} = ${v}")
    }
}
```

### off() - 取消订阅

**文件**: `ohos/request/agent.cj:1801-1829`

```cj
/**
 * 取消订阅事件
 *
 * @param { EventCallbackType } event - 事件类型
 * @param { ?CallbackObject } callback - 回调对象 (可选)
 */
public func off(event: EventCallbackType, callback!: ?CallbackObject = None): Unit
```

**示例**:
```cj
// 取消单个回调
task.off(.Progress, callbackObj)

// 取消所有回调
task.off(.Progress)
```

---

## 4.4 枚举类型

### Action - 任务类型

**文件**: `ohos/request/agent.cj:136-181`

```cj
public enum Action {
    Download  // 下载任务
    Upload    // 上传任务
}
```

### Mode - 运行模式

**文件**: `ohos/request/agent.cj:186-239`

```cj
public enum Mode {
    Background  // 后台任务
    Foreground  // 前台任务 (有 UI 回调)
}
```

### Network - 网络类型

**文件**: `ohos/request/agent.cj:244-300`

```cj
public enum Network {
    AnyType    // 任意网络
    Wifi       // 仅 WiFi
    Cellular   // 仅移动网络
}
```

### State - 任务状态

**文件**: `ohos/request/agent.cj:942-1059`

```cj
public enum State {
    Initialized = 0x00  // 已初始化
    Waiting     = 0x10  // 等待资源
    Running     = 0x20  // 运行中
    Retrying    = 0x21  // 重试中
    Paused      = 0x30  // 已暂停
    Stopped     = 0x31  // 已停止
    Completed   = 0x40  // 已完成
    Failed      = 0x41  // 已失败
    Removed     = 0x50  // 已移除
}
```

### EventCallbackType - 事件类型

**文件**: `ohos/request/agent.cj:48-132`

```cj
public enum EventCallbackType {
    Progress   // 进度更新
    Completed  // 任务完成
    Failed     // 任务失败
    Pause      // 任务暂停
    Resume     // 任务恢复
    Remove     // 任务移除
    Response   // 响应头
}
```

---

## 4.5 数据结构

### FileSpec - 文件规格

**文件**: `ohos/request/agent.cj:342-433`

```cj
public class FileSpec {
    /**
     * 文件路径
     * 支持格式:
     * - 相对路径: "./xxx/yyy/zzz.html"
     * - 内部协议: "internal://cache/path/to/file.txt"
     * - 应用存储: "/data/storage/el1/base/path/to/file.txt"
     * - 文件协议: "file://com.example.test/data/storage/el2/base/file.txt"
     * - 用户文件: "file://media/Photo/path/to/file.png"
     */
    public var path: String

    public var mimeType: ?String      // MIME 类型
    public var filename: ?String     // 文件名
    public var extras: HashMap<String, String>  // 扩展
}
```

### Progress - 进度信息

**文件**: `ohos/request/agent.cj:1076-1130`

```cj
public class Progress {
    public let state: State                    // 当前状态
    public let index: UInt32                   // 当前文件索引
    public let processed: Int64                // 已处理字节数
    public let sizes: Array<Int64>             // 各文件大小数组
    public let extras: HashMap<String, String> // 扩展信息
}
```

### HttpResponse - HTTP 响应

**文件**: `ohos/request/agent.cj:1140-1187`

```cj
public class HttpResponse {
    public let version: String                           // HTTP 版本
    public let statusCode: Int32                         // 状态码
    public let reason: String                            // 原因短语
    public let headers: HashMap<String, Array<String>>  // 响应头
}
```

---

## 4.6 错误码

### 业务异常码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 13400001 | EXCEPTION_FILEIO | 文件 IO 错误 |
| 13400002 | EXCEPTION_FILEPATH | 路径错误 |
| 13400003 | EXCEPTION_SERVICE | 服务错误 |
| 13499999 | EXCEPTION_OTHERS | 其他错误 |

### 失败原因码

| 原因码 | 常量名 | 说明 |
|--------|--------|------|
| 0xFF | Others | 其他 |
| 0x00 | Disconnected | 网络断开 |
| 0x10 | Timeout | 超时 |
| 0x20 | Protocol | 协议错误 |
| 0x40 | Fsio | 文件系统 IO |

---

## 4.7 FFI 接口

### 函数签名

**文件**: `ohos/request/ffi.cj:793-819`

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| FfiOHOSRequestCreateTask | context, config | RetReqData | 创建任务 |
| FfiOHOSRequestRemoveTask | taskId | RetError | 移除任务 |
| FfiOHOSRequestTaskStart | taskId | RetError | 启动任务 |
| FfiOHOSRequestTaskPause | taskId | RetError | 暂停任务 |
| FfiOHOSRequestTaskResume | taskId | RetError | 恢复任务 |
| FfiOHOSRequestTaskStop | taskId | RetError | 停止任务 |
| FfiOHOSRequestShowTask | taskId | RetTaskInfo | 查询任务 |
| FfiOHOSRequestTouchTask | taskId, token | RetTaskInfo | 带 token 查询 |
| FfiOHOSRequestSearchTask | filter | RetTaskArr | 搜索任务 |
| FfiOHOSRequestTaskProgressOn | event, taskId, callback | RetError | 注册回调 |
| FfiOHOSRequestTaskProgressOff | event, taskId, callback | RetError | 取消回调 |
