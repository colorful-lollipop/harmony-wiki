# 对外 N-API (JS API) 文档

## 目的

本文档详细描述 Request 服务对外暴露的 JavaScript API（N-API）。

## 适用范围

- `@ohos.request` 模块（下载/上传）
- `@ohos.request.cacheDownload` 模块（预下载）
- API 8/9/10 版本差异
- 参数校验和错误处理

## 关键结论

1. **两个主要模块**：`request`（下载/上传）和 `cacheDownload`（预下载）
2. **API 版本演进**：API8 基础功能 → API9 增强功能 → API10 统一接口
3. **异步模式**：支持 Callback 和 Promise 两种模式
4. **任务对象**：DownloadTask 和 UploadTask 提供任务管理能力

## API 模块概览

### 模块 1: @ohos.request

**模块名**: `request`  
**注册文件**: `frameworks/js/napi/request/src/request_module.cpp:276`  
**导出常量**:

| 常量名称 | 值 | 类型 | 描述 |
|-----------|-----|------|------|
| `EXCEPTION_PERMISSION` | 201 | int | 权限检查失败 |
| `EXCEPTION_PARAMCHECK` | 202 | int | 参数校验失败 |
| `EXCEPTION_UNSUPPORTED` | 203 | int | 不支持的操作 |
| `EXCEPTION_FILEIO` | 204 | int | 文件 I/O 错误 |
| `EXCEPTION_FILEPATH` | 205 | int | 文件路径错误 |
| `EXCEPTION_SERVICE` | 206 | int | 服务错误 |
| `EXCEPTION_OTHERS` | 207 | int | 其他错误 |
| `NETWORK_MOBILE` | 1 | int | 移动网络 |
| `NETWORK_WIFI` | 2 | int | WiFi 网络 |
| `PAUSED_QUEUED_FOR_WIFI` | 0 | int | 等待 WiFi |
| `PAUSED_WAITING_FOR_NETWORK` | 1 | int | 等待网络 |
| `PAUSED_WAITING_TO_RETRY` | 2 | int | 等待重试 |
| `PAUSED_BY_USER` | 3 | int | 用户暂停 |
| `PAUSED_UNKNOWN` | 4 | int | 未知暂停原因 |

**导出方法**：

#### 1. download()

**证据**: `frameworks/js/napi/request/src/request_module.cpp:264`

```javascript
// API 8 版本（旧接口）
request.download(config: DownloadConfig, callback: AsyncCallback<DownloadTask>): void
request.download(config: DownloadConfig): Promise<DownloadTask>

// API 9 版本（推荐）
request.download(context: BaseContext, config: DownloadConfig, callback: AsyncCallback<DownloadTask>): void
request.download(context: BaseContext, config: DownloadConfig): Promise<DownloadTask>
```

**参数**: DownloadConfig

| 参数 | 类型 | 必填 | 描述 | 证据 |
|-------|------|--------|------|------|
| url | string | 是 | 目标 URL | `js_initialize.cpp:736` |
| header | Object | 否 | 请求头 | `js_initialize.cpp:367` |
| enableMetered | boolean | 否 | 是否允许计费网络 | `js_initialize.cpp:1154` |
| enableRoaming | boolean | 否 | 是否允许漫游 | `js_initialize.cpp:1155` |
| description | string | 否 | 任务描述 | `js_initialize.cpp:1109` |
| networkType | number | 否 | 网络类型（1=移动，2=WiFi）| `js_initialize.cpp:1157` |
| filePath | string | 否 | 文件保存路径 | `js_initialize.cpp:1165` |
| title | string | 否 | 任务标题 | `js_initialize.cpp:1812` |
| background | boolean | 否 | 后台下载 | `js_initialize.cpp:1168` |

**参数校验**:
- URL 最大长度: 8192 字符 (`js_initialize.cpp:739`)
- Description 最大长度: 1024 字符 (`js_initialize.cpp:1712`)
- Title 最大长度: 256 字符 (`js_initialize.cpp:1814`)
- URL 必须以 `http://` 或 `https://` 开头 (`js_initialize.cpp:754`)
- 根据 NetworkSecurityConfig 检查明文传输是否允许 (`js_initialize.cpp:746-752`)

**错误码**: EXCEPTION_PERMISSION (201), EXCEPTION_PARAMCHECK (202), EXCEPTION_FILEIO (204), EXCEPTION_FILEPATH (205)

---

#### 2. upload()

**证据**: `frameworks/js/napi/request/src/request_module.cpp:265`

```javascript
// API 8 版本
request.upload(config: UploadConfig, callback: AsyncCallback<UploadTask>): void
request.upload(config: UploadConfig): Promise<UploadTask>

// API 9 版本
request.upload(context: BaseContext, config: UploadConfig, callback: AsyncCallback<UploadTask>): void
request.upload(context: BaseContext, config: UploadConfig): Promise<UploadTask>
```

**参数**: UploadConfig

| 参数 | 类型 | 必填 | 描述 | 证据 |
|-------|------|--------|------|------|
| url | string | 是 | 目标 URL | `js_initialize.cpp:736` |
| header | Object | 否 | 请求头 | `js_initialize.cpp:367` |
| files | Array<File> | 是 | 上传文件列表 | `js_initialize.cpp:9129` |
| data | Array<RequestData> | 否 | 表单数据 | `js_initialize.cpp:10903` |
| method | string | 否 | HTTP 方法（POST/PUT）| `js_initialize.cpp:1886` |

**参数**: File

| 参数 | 类型 | 必填 | 描述 | 证据 |
|-------|------|--------|------|------|
| filename | string | 否 | 文件名 | `js_initialize.cpp:1066` |
| name | string | 否 | 表单项名称（默认 "file"）| `js_initialize.cpp:1184` |
| uri | string | 是 | 文件 URI | `js_initialize.cpp:1061` |
| type | string | 否 | 内容类型 | `js_initialize.cpp:1069` |

**URI 类型支持**:
- `internal://cache/` - 应用缓存目录
- `dataability://` - DataAbility 文件协议
- 用户自定义路径（API10+）- 需要后台模式 (`js_initialize.cpp:1192`）

**参数校验**:
- URL 最大长度: 8192 字符
- 文件数量限制: API15 最多 100 个文件 (`js_initialize.cpp:1236-1239`)
- Path 验证: `NapiUtils::IsPathValid()` (`js_initialize.cpp:1349`)
- 文件存在性检查 (`js_initialize.cpp:1306-1317`)

---

#### 3. DownloadTask 对象

**证据**: `frameworks/js/napi/request/include/js_task.h:25`

**方法**:

| 方法 | 描述 | 证据 |
|-------|------|------|
| `on('progress', callback)` | 监听下载进度 | `request_event.cpp` |
| `on('complete', callback)` | 监听完成事件 | `request_event.cpp` |
| `on('fail', callback)` | 监听失败事件 | `request_event.cpp` |
| `on('pause', callback)` | 监听暂停事件 | `request_event.cpp` |
| `on('remove', callback)` | 监听删除事件 | `request_event.cpp` |
| `off('progress', callback)` | 取消进度监听 | `request_event.cpp` |
| `off('complete', callback)` | 取消完成监听 | `request_event.cpp` |
| `off('fail', callback)` | 取消失败监听 | `request_event.cpp` |
| `remove()` | 删除任务 | `request_event.cpp` |
| `remove(callback)` | 删除任务（回调）| `request_event.cpp` |
| `pause()` | 暂停任务 | `request_event.cpp` |
| `pause(callback)` | 暂停任务（回调）| `request_event.cpp` |
| `resume()` | 恢复任务 | `request_event.cpp` |
| `resume(callback)` | 恢复任务（回调）| `request_event.cpp` |
| `query()` | 查询任务信息 | `request_event.cpp` |
| `query(callback)` | 查询任务信息（回调）| `request_event.cpp` |
| `queryMimeType()` | 查询 MIME 类型 | `request_event.cpp` |
| `queryMimeType(callback)` | 查询 MIME 类型（回调）| `request_event.cpp` |

---

#### 4. UploadTask 对象

**证据**: `frameworks/js/napi/request/include/upload/upload_task.h`

**方法**:

| 方法 | 描述 | 证据 |
|-------|------|------|
| `on('progress', callback)` | 监听上传进度 | `upload_task.cpp` |
| `on('headerReceive', callback)` | 监听响应头 | `upload_task.cpp` |
| `on('fail', callback)` | 监听失败事件 | `upload_task.cpp` |
| `on('complete', callback)` | 监听完成事件 | `upload_task.cpp` |
| `off('progress', callback)` | 取消进度监听 | `upload_task.cpp` |
| `off('headerReceive', callback)` | 取消头监听 | `upload_task.cpp` |
| `remove()` | 删除任务 | `upload_task.cpp` |
| `remove(callback)` | 删除任务（回调）| `upload_task.cpp` |

---

### 模块 2: @ohos.request.cacheDownload

**模块名**: `request.cacheDownload`  
**注册文件**: `frameworks/js/napi/cache_download/src/preload_module.cpp:765`  
**导出常量**:

| 常量名称 | 值 | 类型 | 描述 |
|-----------|-----|------|------|
| `SslType.TLS` | "TLS" | string | TLS 协议 |
| `SslType.TLCP` | "TLCP" | string | TLCP 协议 |
| `CacheStrategy.FORCE` | 0 | uint32 | 强制更新 |
| `CacheStrategy.LAZY` | 1 | uint32 | 延迟加载 |
| `ErrorCode.OTHERS` | 0 | uint32 | 其他错误 |
| `ErrorCode.DNS` | 1 | uint32 | DNS 错误 |
| `ErrorCode.TCP` | 2 | uint32 | TCP 错误 |
| `ErrorCode.SSL` | 3 | uint32 | SSL 错误 |
| `ErrorCode.HTTP` | 4 | uint32 | HTTP 错误 |

**导出方法**：

#### 1. download(url, options)

**证据**: `preload_module.cpp:388`

```javascript
request.cacheDownload.download(url: string, options: DownloadOptions): void
```

**参数**: DownloadOptions

| 参数 | 类型 | 必填 | 描述 | 证据 |
|-------|------|--------|------|------|
| headers | Object | 否 | 请求头 | `preload_module.cpp:409` |
| sslType | string | 否 | SSL 类型（TLS/TLCP）| `preload_module.cpp:410` |
| caPath | string | 否 | CA 证书路径 | `preload_module.cpp:411-414` |
| cacheStrategy | number | 否 | 缓存策略 | `preload_module.cpp:416-417` |

**权限校验**:
- 需要 `ohos.permission.INTERNET` 权限 (`preload_module.cpp:318-328`)
- 使用 `AccessTokenKit::VerifyAccessToken()` 验证 (`preload_module.cpp:325`)

**参数限制**:
- URL 最大长度: 8192 字符 (`preload_module.cpp:403`)

---

#### 2. cancel(url)

**证据**: `preload_module.cpp:423`

```javascript
request.cacheDownload.cancel(url: string): void
```

---

#### 3. setMemoryCacheSize(size)

**证据**: `preload_module.cpp:442`

```javascript
request.cacheDownload.setMemoryCacheSize(size: number): void
```

**限制**: 最大 1GB (`preload_module.cpp:453`)

---

#### 4. setFileCacheSize(size)

**证据**: `preload_module.cpp:461`

```javascript
request.cacheDownload.setFileCacheSize(size: number): void
```

**限制**: 最大 4GB (`preload_module.cpp:472`)

---

#### 5. getDownloadInfo(url)

**证据**: `preload_module.cpp:503`

```javascript
request.cacheDownload.getDownloadInfo(url: string): DownloadInfo | undefined
```

**权限校验**:
- 需要 `ohos.permission.GET_NETWORK_INFO` 权限 (`preload_module.cpp:331-342`)

---

#### 6. onDownloadSuccess(url, callback)

**证据**: `preload_module.cpp:544`

```javascript
request.cacheDownload.onDownloadSuccess(url: string, callback: (data: Data) => void): void
```

---

#### 7. onDownloadError(url, callback)

**证据**: `preload_module.cpp:582`

```javascript
request.cacheDownload.onDownloadError(url: string, callback: (error: DownloadError) => void): void
```

---

#### 8. offDownloadSuccess(url, callback?)

**证据**: `preload_module.cpp:618`

```javascript
request.cacheDownload.offDownloadSuccess(url: string, callback?: (data: Data) => void): void
```

---

#### 9. offDownloadError(url, callback?)

**证据**: `preload_module.cpp:660`

```javascript
request.cacheDownload.offDownloadError(url: string, callback?: (error: DownloadError) => void): void
```

---

## API 10: request.agent 统一接口

**证据**: `request_module.cpp:107`

### agent.create(config)

```javascript
request.agent.create(context: BaseContext, config: AgentConfig): Promise<AgentTask>
```

**参数**: AgentConfig

| 参数 | 类型 | 必填 | 描述 | 证据 |
|-------|------|--------|------|------|
| action | number | 是 | 操作类型（0=下载，1=上传）| `js_initialize.cpp:636-649` |
| url | string | 是 | 目标 URL | `js_initialize.cpp:736` |
| data | string | 否 | 下载 POST 数据 | `js_initialize.cpp:914` |
| method | string | 否 | HTTP 方法 | `js_initialize.cpp:1886` |
| files | Array | 否 | 上传文件列表 | `js_initialize.cpp:9976` |
| overwrite | boolean | 否 | 是否覆盖文件 | `js_initialize.cpp:358` |
| mode | number | 否 | 前台/后台（0=前台，1=后台）| `js_initialize.cpp:366` |
| network | number | 否 | 网络类型 | `js_initialize.cpp:577` |
| headers | Object | 否 | 请求头 | `js_initialize.cpp:367` |
| extras | Object | 否 | 扩展参数 | `js_initialize.cpp:368` |
| ... | | | |

---

### agent.getTask(context, tid, token)

```javascript
request.agent.getTask(context: BaseContext, tid: string, token: string): Promise<AgentTask>
```

---

### agent.remove(tid)

```javascript
request.agent.remove(tid: string): Promise<boolean>
```

---

### agent.show(tid)

```javascript
request.agent.show(tid: string): Promise<TaskInfo>
```

---

### agent.touch(tid, token)

```javascript
request.agent.touch(tid: string, token: string): Promise<boolean>
```

---

### agent.search(filter)

```javascript
request.agent.search(filter: SearchFilter): Promise<TaskInfo[]>
```

---

### agent.query(tid)

```javascript
request.agent.query(tid: string): Promise<TaskInfo>
```

---

## 事件类型

### 下载事件

| 事件名 | 回调参数 | 描述 | 证据 |
|-------|----------|------|------|
| `progress` | `(receivedSize, totalSize)` | 下载进度 | `request_event.cpp` |
| `complete` | `()` | 下载完成 | `request_event.cpp` |
| `fail` | `(errorCode)` | 下载失败 | `request_event.cpp` |
| `pause` | `()` | 任务暂停 | `request_event.cpp` |
| `remove` | `()` | 任务删除 | `request_event.cpp` |

### 上传事件

| 事件名 | 回调参数 | 描述 | 证据 |
|-------|----------|------|------|
| `progress` | `(uploadedSize, totalSize)` | 上传进度 | `upload_task.cpp` |
| `headerReceive` | `(headers)` | 接收响应头 | `upload_task.cpp` |
| `fail` | `(errorCode)` | 上传失败 | `upload_task.cpp` |
| `complete` | `()` | 上传完成 | `upload_task.cpp` |

---

## 权限要求

### request.download/upload

| 权限 | 用途 | 证据 |
|-------|------|------|
| `ohos.permission.INTERNET` | 所有网络操作 | `preload_module.cpp:48` |

### cacheDownload

| 权限 | 用途 | 证据 |
|-------|------|------|
| `ohos.permission.INTERNET` | 下载操作 | `preload_module.cpp:48` |
| `ohos.permission.GET_NETWORK_INFO` | 查询下载信息 | `preload_module.cpp:49` |

---

## 调用链示例

### 下载任务创建

```
JavaScript 应用
  ↓ request.download(config)
N-API 层 (JsTask::JsDownload)
  ↓ ParseConfig(), CheckFilePath()
  ↓ GetContext()
  ↓ CreateExec()
IPC (RequestServiceProxy::Create)
  ↓ SendRequest(CMD_CONSTRUCT)
服务层 (construct.rs)
  ↓ CheckPermission()
  ↓ TaskManager::add_task()
数据库 (database)
  ↓ InsertTask()
```

### 上传任务创建

```
JavaScript 应用
  ↓ request.upload(config)
N-API 层 (JsTask::JsUpload)
  ↓ ParseUploadConfig(), CheckUploadFiles()
  ↓ CreateExec()
上传任务 (UploadTask)
  ↓ FileAdapter::OpenFiles()
  ↓ curl_adp::InitUpload()
网络层 (libcurl)
  ↓ HTTP POST/PUT
```

---

## 相关跳转

- [内部 API](04_Inner_API.md) - 内部模块接口
- [架构说明](02_Architecture.md) - 架构和时序图
- [安全风险评审](07_Security_Review.md) - 安全分析
