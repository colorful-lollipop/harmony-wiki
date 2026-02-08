# AI Engine 关键调用链

## 客户端调用链

### KWSSdk::Create()

```
KWSSdk::Create()
└── KWSSdkImpl::Create()
    ├── InitComponents()
    │   ├── PluginHelper::Init()
    │   └── MFCCProcessor::Init()
    ├── AieClientInit(configInfo_, clientInfo_, ...)
    │   └── SaClientAdapter::Init()
    │       ├── SAMGR_GetRemoteIdentity(AI_SERVICE)
    │       └── AddDeathRecipient()
    ├── AieClientPrepare(clientInfo_, algorithmInfo_, ...)
    │   └── SaClientAdapter::LoadAlgorithm()
    │       ├── IPC 请求 → SaServerAdapter
    │       └── PluginManager::GetPlugin()
    └── PluginHelper::UnSerializeHandle()
```

**证据**：`README.md:198-241` SDK Create 实现

### KWSSdk::SyncExecute()

```
KWSSdk::SyncExecute(audioInput)
└── KWSSdkImpl::SyncExecute()
    ├── PluginHelper::SerializeInputData()
    ├── AieClientSyncProcess(clientInfo_, ...)
    │   └── SaClientAdapter::SyncExecuteAlgorithm()
    │       ├── IPC 请求序列化
    │       ├── SendRequest() → SAMGR
    │       └── 等待响应
    ├── PluginHelper::UnSerializeOutputData()
    └── 回调 OnResult(result)
```

**证据**：`README.md:243-290` SDK SyncExecute 实现

### KWSSdk::Destroy()

```
KWSSdk::Destroy()
└── KWSSdkImpl::Destroy()
    ├── PluginHelper::SerializeHandle()
    ├── AieClientRelease(clientInfo_, ...)
    │   └── SaClientAdapter::UnloadAlgorithm()
    │       ├── IPC 请求
    │       └── PluginManager::ReleasePlugin()
    ├── AieClientDestroy(clientInfo_)
    │   └── SaClientAdapter::DestroyEngine()
    │       ├── IPC 请求
    │       └── 断开 SAMGR 连接
    └── 释放内部资源
```

**证据**：`README.md:292-321` SDK Destroy 实现

---

## 服务端调用链

### SaServerAdapter::LoadAlgorithm()

```
SaServerAdapter::LoadAlgorithm()
├── 参数校验
├── 查找 PluginInfo
├── IEngineManager::StartEngine()
│   ├── PluginManager::LoadPlugin()
│   │   ├── dlopen(pluginPath)
│   │   ├── dlsym(PLUGIN_INTERFACE)
│   │   └── new Plugin()
│   ├── Engine::Engine()
│   ├── EngineWorker::Start()
│   └── IPlugin::Prepare()
└── 返回 sessionId
```

**证据**：`services/server/communication_adapter/source/sa_server_adapter.cpp`

### SaServerAdapter::SyncExecuteAlgorithm()

```
SaServerAdapter::SyncExecuteAlgorithm()
├── 路由到对应 Engine
├── IEngineManager::ExecuteSync()
│   ├── Engine::PushTask(request)
│   ├── EngineWorker::Process()
│   │   ├── Queue::PopFront()
│   │   └── IPlugin::SyncProcess()
│   │       ├── 算法推理
│   │       └── 返回 response
│   └── Future::Wait()
└── 返回结果
```

**证据**：`services/server/server_executor/source/engine.cpp`

### 异步执行调用链

```
SaServerAdapter::AsyncExecuteAlgorithm()
├── 注册回调
├── IEngineManager::ExecuteAsync()
│   ├── Engine::PushTask(request)
│   ├── EngineWorker::Process()
│   │   ├── Queue::PopFront()
│   │   ├── IPlugin::AsyncProcess()
│   │   └── callback->OnEvent()
│   └── SaAsyncHandler::OnResult()
│       ├── FutureListener 通知
│       └── IPC 回调到客户端
└── 返回 requestId
```

**证据**：`services/server/server_executor/source/async_msg_handler.cpp`

---

## 插件调用链

### IPlugin::Prepare()

```
IPlugin::Prepare(transactionId, inputInfo, outputInfo)
├── 输入参数校验
├── 加载模型文件
├── 初始化推理引擎
├── 预热/warmup
└── 返回 outputInfo (handle)
```

**证据**：`services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp:45-80`

### IPlugin::SyncProcess()

```
IPlugin::SyncProcess(request, response)
├── request 解包
├── 数据预处理
├── 推理执行
├── 结果后处理
├── response 打包
└── 返回 RETCODE
```

**证据**：`services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp:82-120`

### IPlugin::Release()

```
IPlugin::Release(isFullUnload, transactionId, inputInfo)
├── 释放模型资源
├── 关闭推理引擎
├── 清理临时数据
└── 返回 RETCODE
```

**证据**：`services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp:122-150`

---

## IPC 调用链

### 客户端发送请求

```
SaClientAdapter::SendRequest()
├── 参数序列化
├── IPCMessage 构建
├── ClientProxy Send()
├── SAMGR 路由
├── SaServerAdapter 接收
└── 返回响应
```

**证据**：`services/client/communication_adapter/source/sa_client_proxy.cpp`

### 服务端处理请求

```
SaServerAdapter::Invoke()
├── 函数分发 (switch FUNC_ID)
├── 参数反序列化
├── 调用对应处理函数
├── 结果序列化
└── IPC 响应返回
```

**证据**：`services/server/communication_adapter/source/sa_server_adapter.cpp`

---

## 线程创建与销毁

### 客户端线程创建

```
ClientInit()
└── ConnectMgrWorker::Start()
    └── ThreadPool::GetInstance()->Start()
        └── 创建工作线程

SetCallback(callback)
└── AsyncHandler::Start()
    └── 创建异步回调线程
```

### 服务端线程创建

```
StartEngine()
├── EngineWorker::Start()
│   └── 创建 EngineWorker 线程
└── ThreadPool::GetInstance()->Start()
    └── 创建线程池线程
```

---

## 共享内存操作

### 发送方（客户端）

```
AieClientSyncProcess()
├── 检查数据大小 (> IPC_MAX_TRANS_CAPACITY)
├── 大数据 → 创建共享内存
│   ├── shmget()
│   ├── shmat()
│   ├── 写入数据
│   └── 设置接收方 UID 权限
└── 小数据 → 直接 IPC 传输
```

**证据**：`services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:96`

### 接收方（服务端）

```
SaServerAdapter::Invoke()
├── 检查消息类型
├── 大数据 → attach 共享内存
│   ├── shmget()
│   ├── shmat()
│   ├── 读取数据
│   └── shmdt()
└── 小数据 → 直接解析
```
