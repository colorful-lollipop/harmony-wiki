# N-API 接口参考

## 概述

本文档描述 request_cangjie_wrapper 对外暴露的 Cangjie API 接口。

### API 分组

| 分组 | 描述 | 主要类 |
|------|------|--------|
| 任务创建 | 创建下载/上传任务 | `RequestAgent` |
| 任务管理 | 启停、暂停、恢复、移除 | `RequestAgent`, `RequestTask` |
| 任务查询 | 查询任务详情和进度 | `RequestAgent`, `RequestTask` |
| 事件订阅 | 订阅任务状态变更事件 | `RequestTask` |

### 命名空间

所有 API 位于 `@ohos.request` 命名空间下。

## API 清单

### RequestAgent 工厂类

#### createDownload

```cangjie
static func createDownload(config: Config): RequestTask
```

**功能描述**: 创建下载任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| config | Config | 是 | 下载任务配置 |

**返回值**:
| 类型 | 描述 |
|------|------|
| RequestTask | 新创建的任务对象 |

**异常**:
- `BusinessException` - 配置无效时抛出

**使用示例**:
```cangjie
let config: Config = {
    url: "https://example.com/file.zip",
    savePath: "/data/storage/el2/base/files/downloaded.zip"
}
let task = RequestAgent.createDownload(config)
```

**证据来源**: `agent.cj` (详细参数定义需参考外部 API 文档)

#### createUpload

```cangjie
static func createUpload(config: Config): RequestTask
```

**功能描述**: 创建上传任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| config | Config | 是 | 上传任务配置 |

**返回值**:
| 类型 | 描述 |
|------|------|
| RequestTask | 新创建的任务对象 |

**使用示例**:
```cangjie
let config: Config = {
    url: "https://example.com/upload",
    files: [
        { path: "/data/storage/el1/base/files/file1.txt" },
        { path: "/data/storage/el1/base/files/file2.txt" }
    ]
}
let task = RequestAgent.createUpload(config)
```

### 任务管理 API

#### remove

```cangjie
func remove(taskId: Int): Bool
```

**功能描述**: 移除指定任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| taskId | Int | 是 | 要移除的任务 ID |

**返回值**:
| 类型 | 描述 |
|------|------|
| Bool | 是否成功移除 |

**前置条件**:
- 任务必须属于调用方

**证据来源**: `agent.cj` - `remove()` 方法

#### start

```cangjie
func start(taskId: Int): Bool
```

**功能描述**: 启动指定任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| taskId | Int | 是 | 要启动的任务 ID |

**返回值**:
| 类型 | 描述 |
|------|------|
| Bool | 是否成功启动 |

#### stop

```cangjie
func stop(taskId: Int): Bool
```

**功能描述**: 停止指定任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| taskId | Int | 是 | 要停止的任务 ID |

**返回值**:
| 类型 | 描述 |
|------|------|
| Bool | 是否成功停止 |

#### pause

```cangjie
func pause(taskId: Int): Bool
```

**功能描述**: 暂停指定任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| taskId | Int | 是 | 要暂停的任务 ID |

**返回值**:
| 类型 | 描述 |
|------|------|
| Bool | 是否成功暂停 |

**说明**: 支持断点续传，暂停后可恢复

#### resume

```cangjie
func resume(taskId: Int): Bool
```

**功能描述**: 恢复指定任务

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| taskId | Int | 是 | 要恢复的任务 ID |

**返回值**:
| 类型 | 描述 |
|------|------|
| Bool | 是否成功恢复 |

### 任务查询 API

#### query

```cangjie
func query(taskId: Int): RequestTask?
```

**功能描述**: 根据任务 ID 查询任务信息

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| taskId | Int | 是 | 要查询的任务 ID |

**返回值**:
| 类型 | 描述 |
|------|------|
| RequestTask? | 任务对象，不存在时返回 null |

### 事件订阅 API

#### onProgress

```cangjie
func onProgress(callback: (progress: Progress) -> Void): RequestTask
```

**功能描述**: 订阅任务进度事件

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| callback | (Progress) -> Void | 是 | 进度回调函数 |

**返回值**:
| 类型 | 描述 |
|------|------|
| RequestTask | 当前任务对象，支持链式调用 |

**Progress 类型**:
```cangjie
class Progress {
    let taskId: Int           // 任务 ID
    let currentSize: Int64     // 已传输字节数
    let totalSize: Int64       // 总字节数
    let progress: Float        // 进度百分比 0-100
}
```

**使用示例**:
```cangjie
task.onProgress { progress ->
    print("下载进度: ${progress.progress}%")
    print("已下载: ${progress.currentSize}/${progress.totalSize}")
}
```

#### onSuccess

```cangjie
func onSuccess(callback: (result: TaskResult) -> Void): RequestTask
```

**功能描述**: 订阅任务成功事件

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| callback | (TaskResult) -> Void | 是 | 成功回调函数 |

#### onFail

```cangjie
func onFail(callback: (result: TaskResult) -> Void): RequestTask
```

**功能描述**: 订阅任务失败事件

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| callback | (TaskResult) -> Void | 是 | 失败回调函数 |

**TaskResult 类型**:
```cangjie
class TaskResult {
    let taskId: Int            // 任务 ID
    let code: Int              // 状态码
    let message: String        // 状态消息
    let responseCode: Int?    // HTTP 响应码
}
```

## Config 配置详解

### 基础配置

```cangjie
class Config {
    let url: String                  // 请求地址 (必填)
    let method?: String              // 请求方法，默认 GET
    let token?: String               // 认证 Token
    let header?: [String: String]    // 请求头
}
```

### 下载配置扩展

```cangjie
class DownloadConfig extends Config {
    let savePath: String             // 文件保存路径 (必填)
    let network?: NetworkType        // 网络类型限制
    let metered?: Bool               // 是否允许计量网络
}
```

### 上传配置扩展

```cangjie
class UploadConfig extends Config {
    let files: Array<UploadFile>     // 上传文件列表 (必填)
    let data?: [String: String]      // 表单数据
    let multi?: UploadStrategy        // 多文件上传策略
}
```

### UploadFile 类型

```cangjie
class UploadFile {
    let path: String                 // 文件路径 (必填)
    let name?: String                // 表单字段名
    let fileName?: String            // 服务器端文件名
    let mimeType?: String             // MIME 类型
}
```

## 完整使用示例

### 下载文件示例

```cangjie
import { RequestAgent, Config } from '@ohos.request'

// 创建下载配置
let downloadConfig: Config = {
    url: "https://example.com/large-file.zip",
    savePath: "/data/storage/el2/base/files/downloaded.zip"
}

// 创建下载任务
let task = RequestAgent.createDownload(downloadConfig)

// 订阅进度事件
task.onProgress { progress ->
    print("进度: ${progress.progress.toFixed(2)}%")
    print("已下载: ${progress.currentSize} / ${progress.totalSize}")
}

// 订阅成功事件
task.onSuccess { result ->
    print("下载成功! 响应码: ${result.responseCode}")
}

// 订阅失败事件
task.onFail { result ->
    print("下载失败: ${result.message}")
}

// 启动任务
task.start()
```

### 上传文件示例

```cangjie
import { RequestAgent, Config } from '@ohos.request'

// 创建上传配置
let uploadConfig: Config = {
    url: "https://example.com/upload",
    files: [
        { path: "/data/storage/el1/base/files/doc1.pdf" },
        { path: "/data/storage/el1/base/files/doc2.pdf" }
    ]
}

// 创建上传任务
let task = RequestAgent.createUpload(uploadConfig)

// 订阅进度事件
task.onProgress { progress ->
    print("上传进度: ${progress.progress.toFixed(2)}%")
}

// 启动任务
task.start()
```

### 任务管理示例

```cangjie
// 根据 ID 查询任务
let task = RequestAgent.query(taskId)

// 暂停任务
let paused = RequestAgent.pause(taskId)
if (paused) {
    print("任务已暂停")
}

// 恢复任务
let resumed = RequestAgent.resume(taskId)
if (resumed) {
    print("任务已恢复")
}

// 停止任务
let stopped = RequestAgent.stop(taskId)
if (stopped) {
    print("任务已停止")
}

// 移除任务
let removed = RequestAgent.remove(taskId)
if (removed) {
    print("任务已移除")
}
```

## 错误码参考

### 系统错误码

| 错误码 | 常量名 | 描述 |
|--------|--------|------|
| 0 | SUCCESS | 成功 |
| 1 | UNKNOWN | 未知错误 |

### 业务错误码

| 错误码 | 描述 |
|--------|------|
| 下载文件已存在 | savePath 指定的文件已存在 |
| 服务器不支持 HEAD | 下载服务器无法响应 HEAD 请求 |
| 网络错误 | 网络连接失败 |
| 超时错误 | 请求超时 |

**证据来源**: `error.cj` 文件定义

## API 稳定性

| API | 稳定性 | 说明 |
|-----|--------|------|
| RequestAgent.createDownload | Beta | 主功能接口 |
| RequestAgent.createUpload | Beta | 主功能接口 |
| RequestAgent.remove | Beta | 任务管理 |
| RequestAgent.start/stop/pause/resume | Beta | 任务控制 |
| RequestAgent.query | Beta | 任务查询 |
| RequestTask.onProgress/onSuccess/onFail | Beta | 事件订阅 |

**说明**: 当前版本为 Beta，API 可能在后续版本中调整

## 外部文档引用

更多 API 详细信息请参考：

1. **[Upload/Download API Reference](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/BasicServicesKit/cj-apis-request-agent.md)**
2. **[Upload/Download 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/basic-services/request/cj-app-file-upload-download.md)**
