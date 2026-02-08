# AI Engine 内部 API 文档

## 模块概览

AI Engine 内部 API 分为以下模块：

| 模块 | 路径 | 职责 |
|------|------|------|
| 客户端 API | `services/client/client_executor/` | 客户端生命周期管理 |
| 通信适配层 | `services/*/communication_adapter/` | IPC 通信封装 |
| 服务端执行器 | `services/server/server_executor/` | 引擎管理和任务调度 |
| 插件接口 | `services/server/plugin/` | 插件接口定义 |
| 插件管理器 | `services/server/plugin_manager/` | 插件生命周期管理 |
| 协议定义 | `services/common/protocol/` | 数据结构和 IPC 接口 |
| 平台抽象 | `services/common/platform/` | 线程池、队列、锁等 |

**证据**：`services/server/BUILD.gn:15-42` 模块定义

---

## 客户端 API

### ClientFactory 客户端工厂

**文件**：`services/client/client_executor/include/client_factory.h`

**职责**：管理客户端生命周期，提供核心 API

| 方法 | 功能 | 同步/异步 |
|------|------|----------|
| ClientInit() | 初始化客户端，连接服务器 | 同步 |
| ClientPrepare() | 加载算法插件 | 同步 |
| ClientSyncProcess() | 同步执行推理 | 同步 |
| ClientAsyncProcess() | 异步执行推理 | 同步 |
| ClientRelease() | 卸载插件 | 同步 |
| ClientDestroy() | 销毁客户端 | 同步 |

**线程模型**：
- ClientInit/ClientDestroy：可能阻塞等待连接
- ClientProcess：非阻塞，立即返回

**稳定性**：**稳定** - 客户端开发主要使用此接口

### IClientCb 客户端回调

**文件**：`services/client/client_executor/include/i_client_cb.h`

```cpp
class IClientCb {
public:
    virtual ~IClientCb() = default;
    virtual int OnResult(int requestId, const DataInfo &result, int status) = 0;
};
```

**职责**：接收异步推理结果

### AsyncHandler 异步处理器

**文件**：`services/client/client_executor/include/async_handler.h`

**职责**：管理异步回调的注册、注销和分发

---

## 通信适配层

### SaClientAdapter 客户端适配器

**文件**：`services/client/communication_adapter/include/sa_client_adapter.h`

**继承**：ClientFactory

**职责**：
- 实现 SAMGR IPC 客户端连接
- 继承 ClientFactory 的所有方法
- 封装具体的 IPC 调用

**关键方法**：

| 方法 | 功能 |
|------|------|
| Init() | 连接 SAMGR，获取远程 IUnknown |
| SendRequest() | 发送 IPC 请求 |
| OnServiceDead() | 处理服务端死亡 |

**依赖**：`samgr_lite` - OpenHarmony 系统能力管理器

**证据**：`services/client/communication_adapter/source/sa_client.cpp:131-132`

### SaServerAdapter 服务端适配器

**文件**：`services/server/communication_adapter/include/sa_server_adapter.h`

**职责**：
- 实现 SAMGR IPC 服务端处理
- 接收客户端请求
- 调度到引擎管理器

**关键方法**：

| 方法 | 功能 |
|------|------|
| LoadAlgorithm() | 处理插件加载请求 |
| SyncExecuteAlgorithm() | 处理同步推理请求 |
| AsyncExecuteAlgorithm() | 处理异步推理请求 |
| UnloadAlgorithm() | 处理插件卸载请求 |

**IPC 函数 ID**：`ai_service.h:29-40`

**证据**：`services/common/protocol/ipc_interface/ai_service.h`

---

## 服务端执行器

### IEngineManager 引擎管理器

**文件**：`services/server/server_executor/include/i_engine_manager.h`

**职责**：
- 管理引擎的生命周期
- 创建/销毁引擎实例
- 调度任务到引擎

| 方法 | 功能 |
|------|------|
| StartEngine() | 创建引擎，加载插件 |
| StopEngine() | 停止引擎，卸载插件 |
| ExecuteSync() | 同步执行推理 |
| ExecuteAsync() | 异步执行推理 |

### Engine 引擎

**文件**：`services/server/server_executor/include/engine.h`

**职责**：
- 封装插件实例
- 管理任务队列
- 协调 EngineWorker 执行任务

| 属性 | 类型 | 说明 |
|------|------|------|
| plugin_ | IPlugin* | 插件实例指针 |
| taskQueue_ | Queue<IRequest> | 任务队列 |
| worker_ | EngineWorker* | 工作线程 |

### EngineWorker 工作线程

**文件**：`services/server/server_executor/include/engine_worker.h`

**职责**：
- 从任务队列取任务
- 调用插件执行推理
- 处理同步/异步结果

**线程模型**：独立工作线程，循环执行任务

---

## 插件接口

### IPlugin 插件接口

**文件**：`services/server/plugin/i_plugin.h`

**稳定性**：**稳定** - 插件开发必须实现此接口

```cpp
class IPlugin {
public:
    virtual const long long GetVersion() const = 0;
    virtual const char *GetName() const = 0;
    virtual const char *GetInferMode() const = 0;  // "sync" 或 "async"
    virtual int SyncProcess(IRequest *request, IResponse *&response) = 0;
    virtual int AsyncProcess(IRequest *request, IPluginCallback *callback) = 0;
    virtual int Prepare(long long transactionId, const DataInfo &inputInfo, DataInfo &outputInfo) = 0;
    virtual int Release(bool isFullUnload, long long transactionId, const DataInfo &inputInfo) = 0;
    virtual int SetOption(int optionType, const DataInfo &inputInfo) = 0;
    virtual int GetOption(int optionType, const DataInfo &inputInfo, DataInfo &outputInfo) = 0;
};
```

**证据**：`services/server/plugin/i_plugin.h:34-105`

### IPluginCallback 异步回调

**文件**：`services/server/plugin/i_plugin_callback.h`

```cpp
enum PluginEvent {
    ON_PLUGIN_SUCCEED,
    ON_PLUGIN_FAIL,
};

class IPluginCallback {
public:
    virtual int OnEvent(PluginEvent event, IResponse *response) = 0;
};
```

**职责**：插件异步推理时返回结果

### 插件导出宏

```cpp
#define PLUGIN_INTERFACE_IMPL(PluginName) \
    extern "C" IPlugin* PLUGIN_INTERFACE() \
    { \
        return new PluginName(); \
    }
```

**用法**：插件实现文件必须使用此宏导出插件实例

**证据**：`services/server/plugin/i_plugin.h:28-32`

---

## 插件管理器

### IPluginManager 接口

**文件**：`services/server/plugin_manager/include/i_plugin_manager.h`

**职责**：
- 加载/卸载插件动态库
- 获取插件实例
- 管理插件生命周期

| 方法 | 功能 |
|------|------|
| LoadPlugin() | 加载插件动态库 |
| UnloadPlugin() | 卸载插件 |
| GetPlugin() | 获取插件实例 |

---

## 协议定义

### IRequest 请求接口

**文件**：`services/common/protocol/data_channel/include/i_request.h`

```cpp
class IRequest {
public:
    virtual int GetRequestId() const = 0;
    virtual int GetOperationId() const = 0;
    virtual long long GetTransactionId() const = 0;
    virtual uid_t GetClientUid() const = 0;
    virtual int GetAlgoPluginType() const = 0;
    virtual DataInfo GetMsg() const = 0;
    // ... 其他方法
};
```

### IResponse 响应接口

**文件**：`services/common/protocol/data_channel/include/i_response.h`

```cpp
class IResponse {
public:
    virtual int GetResultCode() const = 0;
    virtual DataInfo GetResult() const = 0;
    // ... 其他方法
};
```

### AiInterface IPC 接口

**文件**：`services/common/protocol/ipc_interface/ai_service.h`

**函数 ID**：

| ID | 函数 | 功能 |
|----|------|------|
| 0 | InitEngine | 初始化引擎 |
| 1 | LoadAlgorithm | 加载算法 |
| 2 | SyncExecuteAlgorithm | 同步执行 |
| 3 | AsyncExecuteAlgorithm | 异步执行 |
| 4 | UnloadAlgorithm | 卸载算法 |
| 5 | DestroyEngine | 销毁引擎 |
| 6 | SetOption | 设置选项 |
| 7 | GetOption | 获取选项 |
| 8 | RegisterCallback | 注册回调 |
| 9 | UnregisterCallback | 注销回调 |

**证据**：`services/common/protocol/ipc_interface/ai_service.h:29-40`

### 数据结构定义

**文件**：`services/common/protocol/struct_definition/aie_info_define.h`

```cpp
typedef struct ClientInfo {
    long long clientVersion;
    int clientId;        // 服务器生成
    int sessionId;       // 客户端生成
    uid_t serverUid;     // 服务器 UID
    uid_t clientUid;     // 客户端 UID
    int extendLen;
    unsigned char *extendMsg;
} ClientInfo;

typedef struct AlgorithmInfo {
    long long clientVersion;
    bool isAsync;              // 同步/异步标志
    int algorithmType;         // 算法类型
    long long algorithmVersion;
    bool isCloud;
    int operateId;
    int requestId;
    int extendLen;
    unsigned char *extendMsg;
} AlgorithmInfo;
```

---

## 平台抽象

### ThreadPool 线程池

**文件**：`services/common/platform/threadpool/include/thread_pool.h`

**特性**：
- 单例模式
- 支持自定义栈大小（64KB - 64MB）
- 管理 busy/idle 线程队列

```cpp
class ThreadPool {
public:
    static ThreadPool* GetInstance();
    static void ReleaseInstance();
    int Start(size_t stackSize = 0);
    int Stop();
};
```

### Queue 无锁队列

**文件**：`services/common/platform/queuepool/queue.h`

**特性**：
- 基于原子操作的无锁实现
- 支持 PushBack/PopFront
- 线程安全

### RwLock 读写锁

**文件**：`services/common/platform/lock/include/rw_lock.h`

**接口**：

| 方法 | 功能 |
|------|------|
| LockRead() / UnLockRead() | 读锁 |
| LockWrite() / UnLockWrite() | 写锁 |
| ReadGuard / WriteGuard | RAII 守卫类 |

---

## 稳定性标注

### 稳定接口（推荐使用）

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| IPlugin | **稳定** | 插件开发必须实现 |
| IPluginCallback | **稳定** | 异步回调接口 |
| ClientFactory | **稳定** | 客户端工厂基类 |
| IClientCb | **稳定** | 客户端回调接口 |
| IRequest/IResponse | **稳定** | 请求/响应数据接口 |
| DataInfo/ClientInfo/AlgorithmInfo/ConfigInfo | **稳定** | 核心数据结构 |

### 内部接口（可能变动）

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| SaClientAdapter | 内部 | IPC 适配实现 |
| SaServerAdapter | 内部 | IPC 适配实现 |
| Engine/EngineWorker | 内部 | 引擎实现细节 |
| ThreadPool/QueuePool | 内部 | 平台实现细节 |
| DataEncoder/DataDecoder | 内部 | 序列化实现 |

**证据**：`services/client/client_executor/include/client_factory.h` 客户端架构，`services/server/plugin/i_plugin.h` 插件接口定义
