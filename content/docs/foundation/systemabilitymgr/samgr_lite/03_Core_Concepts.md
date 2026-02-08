# 核心概念

## Service（服务）

Service 是 samgr_lite 中的基本功能单元，代表系统中的一个可管理能力。

### 结构定义

```c
// evidence: interfaces/kits/samgr/service.h:149-205
struct Service {
    // 获取服务名称（必须实现）
    const char *(*GetName)(Service *service);

    // 初始化服务（必须实现）
    BOOL (*Initialize)(Service *service, Identity identity);

    // 处理消息（必须实现）
    BOOL (*MessageHandle)(Service *service, Request *request);

    // 获取任务配置（必须实现）
    TaskConfig (*GetTaskConfig)(Service *service);
};
```

### 服务定义宏

```c
// evidence: interfaces/kits/samgr/service.h:213-217
#define INHERIT_SERVICE                                          \
    const char *(*GetName)(Service * service);                   \
    BOOL (*Initialize)(Service * service, Identity identity);  \
    BOOL (*MessageHandle)(Service * service, Request * request); \
    TaskConfig (*GetTaskConfig)(Service * service)
```

### 服务示例

```c
typedef struct ExampleService {
    INHERIT_SERVICE;
    INHERIT_IUNKNOWNENTRY(DefaultFeatureApi);
    Identity identity;
} ExampleService;

static const char *GetName(Service *service)
{
    return EXAMPLE_SERVICE;  // 常量字符串，长度 < 16
}

static BOOL Initialize(Service *service, Identity identity)
{
    ExampleService *example = (ExampleService *)service;
    example->identity = identity;  // 保存身份标识
    return TRUE;
}

static BOOL MessageHandle(Service *service, Request *msg)
{
    // 处理消息
    return TRUE;
}

static TaskConfig GetTaskConfig(Service *service)
{
    TaskConfig config = {LEVEL_HIGH, PRI_BELOW_NORMAL, 0x800, 20, SHARED_TASK};
    return config;
}
```

## Feature（功能）

Feature 是 Service 的子功能模块，可以独立提供 API。

### 结构定义

```c
// evidence: interfaces/kits/samgr/feature.h:65-124
struct Feature {
    // 获取 Feature 名称（必须实现）
    const char *(*GetName)(Feature *feature);

    // 初始化（必须实现）
    void (*OnInitialize)(Feature *feature, Service *parent, Identity identity);

    // 停止（必须实现）
    void (*OnStop)(Feature *feature, Identity identity);

    // 处理消息（必须实现）
    BOOL (*OnMessage)(Feature *feature, Request *request);
};
```

### Feature 示例

```c
typedef struct DemoFeature {
    INHERIT_FEATURE;
    INHERIT_IUNKNOWNENTRY(DemoApi);
    Identity identity;
    Service *parent;
} DemoFeature;

static const char *FEATURE_GetName(Feature *feature)
{
    return EXAMPLE_FEATURE;
}

static void FEATURE_OnInitialize(Feature *feature, Service *parent, Identity identity)
{
    DemoFeature *demoFeature = (DemoFeature *)feature;
    demoFeature->identity = identity;
    demoFeature->parent = parent;
}

static void FEATURE_OnStop(Feature *feature, Identity identity)
{
    g_example.identity.queueId = NULL;
    g_example.identity.featureId = -1;
    g_example.identity.serviceId = -1;
}

static BOOL FEATURE_OnMessage(Feature *feature, Request *request)
{
    if (request->msgId == MSG_PROC) {
        Response response = {.data = "Yes!", .len = 0};
        SAMGR_SendResponse(request, &response);
        return TRUE;
    }
    return FALSE;
}
```

## IUnknown（接口基类）

IUnknown 是所有对外接口的基类，提供接口查询和引用计数功能。

### 结构定义

```c
// evidence: interfaces/kits/samgr/iunknown.h:150-160
struct IUnknown {
    // 查询接口（必须实现）
    int (*QueryInterface)(IUnknown *iUnknown, int version, void **target);

    // 增加引用计数
    int (*AddRef)(IUnknown *iUnknown);

    // 释放引用
    int (*Release)(IUnknown *iUnknown);
};
```

### 版本定义

```c
// evidence: interfaces/kits/samgr/iunknown.h:63
#define DEFAULT_VERSION 0x20    // 进程内 IUnknown 版本

// evidence: interfaces/kits/registry/iproxy_client.h:55
#define CLIENT_PROXY_VER (0x40 | (uint16)DEFAULT_VERSION)  // 客户端代理版本

// evidence: interfaces/kits/registry/iproxy_server.h:62
#define SERVER_PROXY_VER 0x80  // 服务端代理版本
```

### 接口继承宏

```c
// 继承 IUnknown 接口
#define INHERIT_IUNKNOWN                                                   \
    int (*QueryInterface)(IUnknown *iUnknown, int version, void **target); \
    int (*AddRef)(IUnknown *iUnknown);                                     \
    int (*Release)(IUnknown *iUnknown)

// 继承 IUnknown 实现类
#define INHERIT_IUNKNOWNENTRY(T) \
    uint16 ver;                   \
    int16 ref;                   \
    T iUnknown
```

### 使用示例

```c
// 定义自定义 API
typedef struct DemoApi {
    INHERIT_IUNKNOWN;
    BOOL (*AsyncCall)(IUnknown *iUnknown, const char *buff);
    BOOL (*SyncCall)(IUnknown *iUnknown, struct Payload *payload);
} DemoApi;

// 获取并使用 API
DemoApi *demoApi = NULL;
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(SERVICE_NAME, FEATURE_NAME);
int result = iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&demoApi);
if (result == EC_SUCCESS) {
    demoApi->AsyncCall((IUnknown *)demoApi, "data");
}
demoApi->Release((IUnknown *)demoApi);
```

## Identity（身份标识）

Identity 用于唯一标识 Service 或 Feature。

### 结构定义

```c
// evidence: interfaces/kits/samgr/message.h:77-84
struct Identity {
    int16 serviceId;   // 服务 ID（由 Samgr 分配）
    int16 featureId;   // 功能 ID（由 Samgr 分配）
    MQueueId queueId;  // 消息队列 ID
};
```

### 用途

- 消息发送时指定目标
- 异步响应的路由依据
- 服务/Feature 的唯一标识

## Request/Response（消息对）

Request 和 Response 用于进程内消息通信。

### Request 结构

```c
// evidence: interfaces/kits/samgr/message.h:95-104
struct Request {
    int16 msgId;      // 消息 ID（自定义）
    int16 len;        // 数据长度
    void *data;       // 数据指针
    uint32 msgValue;  // 消息值（自定义）
};
```

### Response 结构

```c
// evidence: interfaces/kits/samgr/message.h:114-120
struct Response {
    void *data;   // 响应数据
    int16 len;    // 数据长度
    void *reply;  // 额外回复
};
```

## 概念关系图

```
                    ┌─────────────────┐
                    │     Service      │
                    │  ┌───────────┐  │
                    │  │ Feature1  │──┼──► IUnknown (API)
                    │  ├───────────┤  │
                    │  │ Feature2  │──┼──► IUnknown (API)
                    │  └───────────┘  │
                    └─────────────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
         GetName()     Initialize()   MessageHandle()
         GetTaskConfig()                │
                                        ▼
                                  Identity
                                  (serviceId)
```

## 下一章

- [SamgrLite API](./04_SamgrLite_API.md) - API 详细使用说明
- [消息通信 API](./05_Message_API.md) - 请求/响应机制
