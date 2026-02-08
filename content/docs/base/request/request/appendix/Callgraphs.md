# 关键调用链

## 目的

本文档记录 Request 服务的核心调用链。

## 适用范围

- 下载任务创建和执行
- 上传任务创建和执行
- 预下载流程
- IPC 通信链

---

## 下载任务创建调用链

### JavaScript → N-API → IPC → 服务

```
JavaScript 层
  └─> request.download(config)
      (frameworks/js/napi/request/src/request_module.cpp:264)
      
N-API 层
  └─> JsTask::JsDownload(env, info)
      (frameworks/js/napi/request/src/js_task.cpp:39)
      
  └─> JsInitialize::Initialize(env, info, API8, false)
      (frameworks/js/napi/request/src/js_initialize.cpp:61)
      
  └─> InitParam(env, argv, context, config)
      
  └─> GetContext(env, argv[0], context)
      (frameworks/js/napi/request/src/js_initialize.cpp:145)
      
  └─> ParseConfig(env, argv[1], config, errInfo)
      (frameworks/js/napi/request/src/js_initialize.cpp:1408)
      
  └─> CheckFilePath(context, config, error)
      (frameworks/js/napi/request/src/js_initialize.cpp:184)
      
  └─> CreateExec(context, seq)
      (frameworks/js/napi/request/src/js_task.cpp:93)
      
  └─> RequestManagerImpl::Create(contextInfo)
      (frameworks/native/request/src/request_manager_impl.cpp)
      
  └─> RequestServiceProxy::Create(taskInfo)
      (frameworks/native/request/src/request_service_proxy.cpp)
      
IPC 通信
  └─> proxy->SendRequest(CMD_CONSTRUCT, taskInfo)
      
服务层 (Rust)
  └─> RequestServiceStub::on_remote_request(code, data, reply)
      (services/src/service/stub.rs)
      
  └─> match code {
        CMD_CONSTRUCT => self.handle_construct(data, reply),
      }
      
  └─> construct::handle(data, reply)
      (services/src/service/command/construct.rs)
      
  └─> check_permission(token_id, INTERNET_PERMISSION)
      (services/src/service/permission.rs)
      
  └─> check_task_uid(task_id, uid) || has_manager_permission()
      
  └─> TaskManager::add_task(task)
      (services/src/manage/task_manager.rs)
      
  └─> database.insert_task(task)
      (common/database/src/cxx/c_request_database.cpp)
      
  └─> return taskId
```

---

## 上传任务创建调用链

### JavaScript → N-API → libcurl

```
JavaScript 层
  └─> request.upload(config)
      (frameworks/js/napi/request/src/request_module.cpp:265)
      
N-API 层
  └─> JsTask::JsUpload(env, info)
      (frameworks/js/napi/request/src/js_task.cpp:38)
      
  └─> JsInitialize::Initialize(env, info, API8, false)
      
  └─> InitParam(env, argv, context, config)
      
  └─> GetContext(env, argv[0], context)
      
  └─> ParseUploadConfig(env, jsConfig, config, errInfo)
      (frameworks/js/napi/request/src/js_initialize.cpp:1119)
      
  └─> CheckUploadFiles(context, config, error)
      (frameworks/js/napi/request/src/js_initialize.cpp:1231)
      
  └─> UploadTask::ExecuteTask()
      (frameworks/js/napi/request/src/upload/upload_task.cpp)
      
  └─> FileAdapter::OpenFiles(files)
      (frameworks/js/napi/request/src/upload/file_adapter.cpp)
      
  └─> curl_adp::InitUpload(config, files)
      (frameworks/js/napi/request/src/upload/curl_adp.cpp)
      
网络层
  └─> libcurl 执行 HTTP POST/PUT
      (第三方库)
      
  └─> 回调处理
      └─> ProgressEvent
      └─> HeaderReceiveEvent
      └─> CompleteEvent
      └─> FailEvent
      
N-API 层
  └─> callback 调用
      (JavaScript 回调)
```

---

## 预下载调用链

### JavaScript → N-API → Cache → Network

```
JavaScript 层
  └─> request.cacheDownload.download(url, options)
      (frameworks/js/napi/cache_download/src/preload_module.cpp:388)
      
N-API 层
  └─> download(env, info)
      
  └─> CheckInternetPermission()
      (frameworks/js/napi/cache_download/src/preload_module.cpp:318)
      
  └─> AccessTokenKit::VerifyAccessToken(tokenId, INTERNET_PERMISSION)
      
  └─> 失败 → 返回错误
      
  └─> 成功 → 继续
      
  └─> SetOptionsHeaders(env, args[1], options)
  └─> SetOptionsSslType(env, args[1], options)
  └─> GetCacheStrategy(env, args[1], isUpdate)
      
  └─> CreatePreloadCallback(env, url)
      
  └─> Preload::load(url, callback, options, isUpdate)
      (frameworks/native/cache_download/src/)
      
缓存层 (Rust)
  └─> CheckCache(url)
      (cache_core)
      
  └─> 缓存命中?
      ├─ 是 → 直接返回 data
      └─ 否 → 继续网络下载
      
网络层
  └─> ylong_http::load(url)
      (common/netstack_rs/)
      
  └─> HTTP GET 请求
      
  └─> 保存到文件
      
  └─> 更新缓存
      
  └─> callback->OnSuccess(data)
      
N-API 层
  └─> CallbackManager::InvokeSuccessCallbacks(url, data, env, taskId)
      (frameworks/js/napi/cache_download/src/preload_module.cpp:257)
      
  └─> napi_send_event(info->env_, callback)
      
JavaScript 层
  └─> success callback(data)
```

---

## 任务控制调用链

### 暂停任务

```
JavaScript 层
  └─> task.pause()
      (frameworks/js/napi/request/src/request_event.cpp)
      
N-API 层
  └─> RequestEvent::Pause(env, info)
      
  └─> AsyncCall::Execute(env, info, Pause)
      (frameworks/js/napi/request/src/async_call.cpp)
      
  └─> SendRequest(CMD_PAUSE, taskId)
      
IPC 通信
  └─> RequestServiceProxy::Pause(taskId)
      
服务层 (Rust)
  └─> RequestServiceStub::on_remote_request(code, data, reply)
      
  └─> pause::handle(data, reply)
      (services/src/service/command/pause.rs)
      
  └─> check_permission_or_uid(taskId)
      
  └─> TaskManager::pause_task(taskId)
      
  └─> database.update_status(taskId, PAUSED)
      
  └─> 通知客户端
```

---

### 恢复任务

```
JavaScript 层
  └─> task.resume()
      
N-API 层
  └─> RequestEvent::Resume(env, info)
      
  └─> AsyncCall::Execute(env, info, Resume)
      
  └─> SendRequest(CMD_RESUME, taskId)
      
IPC 通信
  └─> RequestServiceProxy::Resume(taskId)
      
服务层 (Rust)
  └─> resume::handle(data, reply)
      (services/src/service/command/resume.rs)
      
  └─> check_permission_or_uid(taskId)
      
  └─> TaskManager::resume_task(taskId)
      
  └─> 重新启动网络请求
      
  └─> 通知客户端
```

---

## 权限检查调用链

### 权限验证流程

```
应用请求
  └─> request.download(config)
      
获取调用者身份
  └─> IPCSkeleton::GetCallingFullTokenID()
      (common/utils/src/cxx/...)
      
  └─> AccessTokenKit::GetTokenTypeFlag(tokenId)
      
  └─> tokenType == TOKEN_INVALID?
      ├─ 是 → 拒绝请求
      └─ 否 → 继续
      
权限验证
  └─> AccessTokenKit::VerifyAccessToken(tokenId, ohos.permission.INTERNET)
      
  └─> result == PERMISSION_GRANTED?
      ├─ 否 → 返回 E_PERMISSION (201)
      └─ 是 → 允许请求
      
任务操作
  └─> 继续执行下载/上传
```

---

## 相关跳转

- [对外 N-API](03_NAPI_JS_API.md) - API 方法
- [内部 API](04_Inner_API.md) - 接口定义
- [架构说明](02_Architecture.md) - 数据流
- [安全风险评审](07_Security_Review.md) - 权限检查
