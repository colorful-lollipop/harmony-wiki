# 04 - 对外接口文档

**文档目的**: 详细说明所有对外暴露的接口，包括 N-API 和 IPC 接口  
**目标受众**: 新人学习者 + 安全研究员  
**阅读时间**: 约 20 分钟

---

## 接口概览

```mermaid
graph LR
    subgraph JS[JS/TS 应用]
        API1[@ohos.request]
        API2[@ohos.request.agent]
    end
    
    subgraph Native[Native 应用]
        API3[Native Framework]
    end
    
    subgraph IPC[IPC 通信]
        CMD[22 个 IPC 命令]
    end
    
    JS -->|N-API| API1
    JS -->|N-API| API2
    Native -->|C++| API3
    API1 -->|IPC| CMD
    API2 -->|IPC| CMD
    API3 -->|IPC| CMD
```

---

## N-API 接口

### 模块信息

| 属性 | 值 | 代码位置 |
|------|-----|----------|
| **模块名** | `request` | `frameworks/js/napi/request/src/request_module.cpp:282` |
| **注册函数** | `RegisterModule()` | `frameworks/js/napi/request/src/request_module.cpp:276` |
| **初始化函数** | `Init()` / `InitAgent()` | `frameworks/js/napi/request/src/request_module.cpp:159` |

### 传统 API（兼容模式）

| JS API | C++ 入口 | 权限 | 说明 |
|--------|----------|------|------|
| `download(config)` | `JsTask::JsDownload` | INTERNET | 创建下载任务 |
| `upload(config)` | `JsTask::JsUpload` | INTERNET | 创建上传任务 |
| `downloadFile(context, config)` | `JsTask::JsRequestFile` | INTERNET | 带 Context 的下载 |
| `uploadFile(context, config)` | `JsTask::JsRequestFile` | INTERNET | 带 Context 的上传 |
| `onDownloadComplete()` | `Legacy::RequestManager::OnDownloadComplete` | - | 监听下载完成 |

**代码位置**: `frameworks/js/napi/request/src/request_module.cpp:264-268`

### Agent API（API 10+）

#### 任务管理

| JS API | C++ 入口 | 权限 | 同步/异步 |
|--------|----------|------|----------|
| `agent.create(config)` | `JsTask::JsCreate` | INTERNET | 异步 |
| `agent.getTask(tid)` | `JsTask::GetTask` | INTERNET | 异步 |
| `agent.remove(tid)` | `JsTask::Remove` | INTERNET | 异步 |
| `agent.show(tid, token)` | `JsTask::Show` | INTERNET | 异步 |
| `agent.touch(tid, token)` | `JsTask::Touch` | INTERNET | 异步 |
| `agent.search(filter)` | `JsTask::Search` | INTERNET | 异步 |
| `agent.query(tid)` | `JsTask::Query` | INTERNET | 异步 |

**代码位置**: `frameworks/js/napi/request/src/request_module.cpp:139-145`

#### 任务分组

| JS API | C++ 入口 | 权限 |
|--------|----------|------|
| `agent.createGroup(name)` | `createGroup` | INTERNET |
| `agent.attachGroup(tid, name)` | `attachGroup` | INTERNET |
| `agent.deleteGroup(name)` | `deleteGroup` | INTERNET |

**代码位置**: `frameworks/js/napi/request/src/request_module.cpp:146-148`

### 常量定义

#### Action（操作类型）

```cpp
// frameworks/js/napi/request/src/request_module.cpp:38-39
Action.DOWNLOAD = 0
Action.UPLOAD = 1
```

#### Mode（任务模式）

```cpp
// frameworks/js/napi/request/src/request_module.cpp:45-46
Mode.BACKGROUND = 0  // 后台模式
Mode.FOREGROUND = 1  // 前台模式
```

#### Network（网络类型）

```cpp
// frameworks/js/napi/request/src/request_module.cpp:52-54
Network.ANY = 0       // 任意网络
Network.WIFI = 1      // 仅 WiFi
Network.CELLULAR = 2  // 仅蜂窝网络
```

#### State（任务状态）

```cpp
// frameworks/js/napi/request/src/request_module.cpp:60-68
State.INITIALIZED = 0
State.WAITING = 1
State.RUNNING = 2
State.RETRYING = 3
State.PAUSED = 4
State.STOPPED = 5
State.COMPLETED = 6
State.FAILED = 7
State.REMOVED = 8
```

#### Faults（错误类型）

```cpp
// frameworks/js/napi/request/src/request_module.cpp:74-84
Faults.OTHERS = 0
Faults.DISCONNECTED = 1
Faults.TIMEOUT = 2
Faults.PROTOCOL = 3
Faults.PARAM = 4
Faults.FSIO = 5
Faults.DNS = 6
Faults.TCP = 7
Faults.SSL = 8
Faults.REDIRECT = 9
Faults.LOW_SPEED = 10
```

### Task 对象方法

| 方法 | 说明 | 权限 |
|------|------|------|
| `pause()` | 暂停任务 | INTERNET |
| `resume()` | 恢复任务 | INTERNET |
| `remove()` | 移除任务 | INTERNET |
| `query()` | 查询任务信息 | INTERNET |
| `queryMimeType()` | 查询 MIME 类型 | INTERNET |

### Task 事件

| 事件名 | 回调参数 | 说明 |
|--------|----------|------|
| `progress` | `(received, total)` | 进度更新 |
| `complete` | `()` | 任务完成 |
| `fail` | `(error)` | 任务失败 |
| `pause` | `()` | 任务暂停 |
| `remove` | `()` | 任务移除 |

### 配置参数限制

| 参数 | 最大长度 | 代码位置 |
|------|----------|----------|
| URL | 8192 bytes | `frameworks/native/request_action/src/task_builder.cpp:232` |
| Title | 256 bytes | `frameworks/native/request_action/src/task_builder.cpp:326` |
| Description | 1024 bytes | `frameworks/cj/ffi/src/cj_initialize.cpp:53` |
| Proxy | 512 bytes | `frameworks/native/request_action/src/task_builder.cpp:309` |
| Token | 8-2048 bytes | `frameworks/native/request_action/src/task_builder.cpp:339-340` |
| 上传文件数 | 100 | `frameworks/cj/ffi/src/cj_initialize.cpp:54` |

---

## IPC 接口

### 接口定义

**Interface Token**: `"OHOS.Download.RequestServiceInterface"`

**代码位置**:
- 定义: `frameworks/native/request/include/request_service_interface.h`
- Proxy: `frameworks/native/request/src/request_service_proxy.cpp`
- Stub: `services/src/service/stub.rs`

### IPC 命令码

| 命令码 | 值 | 名称 | 说明 |
|--------|-----|------|------|
| CONSTRUCT | 0 | 创建任务 | `services/src/service/interface.rs:21` |
| PAUSE | 1 | 暂停任务 | `services/src/service/interface.rs:23` |
| QUERY | 2 | 查询任务 | `services/src/service/interface.rs:25` |
| QUERY_MIME_TYPE | 3 | 查询 MIME | `services/src/service/interface.rs:27` |
| REMOVE | 4 | 移除任务 | `services/src/service/interface.rs:29` |
| RESUME | 5 | 恢复任务 | `services/src/service/interface.rs:31` |
| START | 6 | 启动任务 | `services/src/service/interface.rs:33` |
| STOP | 7 | 停止任务 | `services/src/service/interface.rs:35` |
| SHOW | 8 | 显示任务 | `services/src/service/interface.rs:37` |
| TOUCH | 9 | 触摸任务 | `services/src/service/interface.rs:39` |
| SEARCH | 10 | 搜索任务 | `services/src/service/interface.rs:41` |
| GET_TASK | 11 | 获取任务 | `services/src/service/interface.rs:43` |
| CLEAR | 12 | 清除任务 | `services/src/service/interface.rs:45` |
| OPEN_CHANNEL | 13 | 打开通道 | `services/src/service/interface.rs:47` |
| SUBSCRIBE | 14 | 订阅通知 | `services/src/service/interface.rs:49` |
| UNSUBSCRIBE | 15 | 取消订阅 | `services/src/service/interface.rs:51` |
| SUB_RUN_COUNT | 16 | 订阅计数 | `services/src/service/interface.rs:53` |
| UNSUB_RUN_COUNT | 17 | 取消计数订阅 | `services/src/service/interface.rs:55` |
| CREATE_GROUP | 18 | 创建分组 | `services/src/service/interface.rs:57` |
| ATTACH_GROUP | 19 | 附加分组 | `services/src/service/interface.rs:59` |
| DELETE_GROUP | 20 | 删除分组 | `services/src/service/interface.rs:61` |
| SET_MAX_SPEED | 21 | 设置限速 | `services/src/service/interface.rs:63` |
| SHOW_PROGRESS | 22 | 显示进度 | `services/src/service/interface.rs:65` |
| SET_MODE | 100 | 设置模式 | `services/src/service/interface.rs:67` |
| DISABLE_TASK_NOTIFICATION | 101 | 禁用通知 | `services/src/service/interface.rs:69` |

### IPC 方法签名

#### 任务生命周期

```cpp
// frameworks/native/request/include/request_service_interface.h

// 创建任务
virtual int32_t Create(
    const Config &config, 
    int32_t &taskId
) = 0;

// 启动任务
virtual int32_t Start(
    int32_t taskId
) = 0;

// 暂停任务
virtual int32_t Pause(
    int32_t taskId, 
    const std::string &token
) = 0;

// 恢复任务
virtual int32_t Resume(
    int32_t taskId, 
    const std::string &token
) = 0;

// 停止任务
virtual int32_t Stop(
    int32_t taskId, 
    const std::string &token
) = 0;

// 移除任务
virtual int32_t Remove(
    int32_t taskId, 
    const std::string &token
) = 0;
```

#### 查询操作

```cpp
// 查询单个任务
virtual int32_t Query(
    int32_t taskId, 
    const std::string &token,
    TaskInfo &info
) = 0;

// 查询多个任务
virtual int32_t QueryTasks(
    const Filter &filter,
    std::vector<TaskInfo> &infos
) = 0;

// 搜索任务
virtual int32_t Search(
    const Filter &filter,
    std::vector<TaskInfo> &infos
) = 0;

// 查询 MIME 类型
virtual int32_t QueryMimeType(
    int32_t taskId,
    const std::string &token,
    std::string &mimeType
) = 0;
```

#### 订阅通知

```cpp
// 订阅任务通知
virtual int32_t Subscribe(
    int32_t taskId,
    const std::string &token,
    const sptr<NotifyInterface> &listener
) = 0;

// 取消订阅
virtual int32_t Unsubscribe(
    int32_t taskId,
    const std::string &token
) = 0;

// 打开通知通道
virtual int32_t OpenChannel(
    int32_t taskId,
    const std::string &token,
    int32_t &channelId
) = 0;
```

### IPC 数据校验

#### Interface Token 校验

```cpp
// frameworks/native/request/src/runcount_notify_stub.cpp:44-60
int32_t RunCountNotifyStub::OnRemoteRequest(
    uint32_t code, 
    MessageParcel &data, 
    MessageParcel &reply, 
    MessageOption &option)
{
    // 校验 Interface Token
    auto descriptorToken = data.ReadInterfaceToken();
    if (descriptorToken != GetDescriptor()) {
        REQUEST_HILOGE("Remote descriptor not the same as local descriptor.");
        return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
    }
    // ...
}
```

---

## 权限要求

### 必需权限

| 权限 | 用途 | 检查位置 |
|------|------|----------|
| `ohos.permission.INTERNET` | 网络访问 | `frameworks/js/napi/cache_download/src/preload_module.cpp:317` |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | `frameworks/ets/ani/cache_download/src/cxx/permission_verification.cpp:36` |

### 管理权限

| 权限 | 用途 | 检查位置 |
|------|------|----------|
| `ohos.permission.DOWNLOAD_SESSION_MANAGER` | 管理下载任务 | `services/src/service/permission.rs:27` |
| `ohos.permission.UPLOAD_SESSION_MANAGER` | 管理上传任务 | `services/src/service/permission.rs:29` |

### 权限检查实现

```cpp
// services/src/cxx/request_utils.cpp:92-105
bool CheckPermission(uint64_t tokenId, rust::str permission)
{
    auto perm = std::string(permission);
    TypeATokenTypeEnum tokenType = 
        AccessTokenKit::GetTokenTypeFlag(static_cast<AccessTokenID>(tokenId));
    if (tokenType == TOKEN_INVALID) {
        REQUEST_HILOGE("invalid token id");
        return false;
    }
    int result = AccessTokenKit::VerifyAccessToken(tokenId, perm);
    if (result != PERMISSION_GRANTED) {
        return false;
    }
    return true;
}
```

---

## 错误码

### N-API 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| EXCEPTION_PERMISSION | -1 | 权限不足 |
| EXCEPTION_PARAMETER_CHECK | -2 | 参数错误 |
| EXCEPTION_UNSUPPORTED | -3 | 不支持的操作 |
| EXCEPTION_FILE_IO | -4 | 文件 IO 错误 |
| EXCEPTION_FILE_PATH | -5 | 文件路径错误 |
| EXCEPTION_SERVICE_ERROR | -6 | 服务错误 |
| EXCEPTION_OTHERS | -7 | 其他错误 |

### 下载错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERROR_CANNOT_RESUME | 0 | 无法恢复 |
| ERROR_DEVICE_NOT_FOUND | 1 | 设备未找到 |
| ERROR_FILE_ALREADY_EXISTS | 2 | 文件已存在 |
| ERROR_FILE_ERROR | 3 | 文件错误 |
| ERROR_HTTP_DATA_ERROR | 4 | HTTP 数据错误 |
| ERROR_INSUFFICIENT_SPACE | 5 | 空间不足 |
| ERROR_TOO_MANY_REDIRECTS | 6 | 重定向过多 |
| ERROR_UNHANDLED_HTTP_CODE | 7 | 未处理的 HTTP 码 |
| ERROR_UNKNOWN | 8 | 未知错误 |
| ERROR_OFFLINE | 9 | 离线状态 |
| ERROR_UNSUPPORTED_NETWORK_TYPE | 10 | 不支持的网路类型 |

---

## 相关文档

- **代码位置**: [03_CodeMap.md](03_CodeMap.md)
- **架构理解**: [02_Architecture.md](02_Architecture.md)
- **安全分析**: [05_AttackSurface.md](05_AttackSurface.md)
