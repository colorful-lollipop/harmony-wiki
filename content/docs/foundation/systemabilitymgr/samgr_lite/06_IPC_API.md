# IPC 跨进程接口

## 概述

IPC（Inter-Process Communication）跨进程接口用于 A-core 平台上的跨进程服务调用。基于 Binder 机制实现，支持同步和异步调用模式。

## 组件架构

```
┌──────────────┐                    ┌──────────────┐
│ Client Proc  │                    │ Server Proc  │
│              │                    │              │
│ ┌──────────┐ │    Binder IPC     │ ┌──────────┐ │
│ │samgr_cli │ │ ◄──────────────► │ │samgr_srv │ │
│ └────┬─────┘ │                    │ └────┬─────┘ │
│      │       │                    │      │       │
│      ▼       │                    │      ▼       │
│ ┌──────────┐ │                    │ ┌──────────┐ │
│ │IClientProxy│ │                   │ │IServerProxy│ │
│ └──────────┘ │                    │ └──────────┘ │
└──────────────┘                    └──────────────┘
```

## IClientProxy（客户端代理）

IClientProxy 是客户端用于发送跨进程请求的接口。

### 结构定义

```c
// evidence: interfaces/kits/registry/iproxy_client.h:91-114
struct IClientProxy {
    // 继承 IUnknown
    INHERIT_IUNKNOWN;

    // 发送 IPC 消息
    int (*Invoke)(IClientProxy *proxy, int funcId, IpcIo *request, 
                  IOwner owner, INotify notify);
};
```

### 版本定义

```c
// evidence: interfaces/kits/registry/iproxy_client.h:55
#define CLIENT_PROXY_VER (0x40 | (uint16)DEFAULT_VERSION)
```

### 继承宏

```c
// evidence: interfaces/kits/registry/iproxy_client.h:63-65
#define INHERIT_CLIENT_IPROXY \
        INHERIT_IUNKNOWN; \
        int (*Invoke)(IClientProxy *proxy, int funcId, IpcIo *request, \
                      IOwner owner, INotify notify)
```

### INotify 回调

```c
// evidence: interfaces/kits/registry/iproxy_client.h:77
typedef int (*INotify)(IOwner owner, int code, IpcIo *reply);
```

| 参数 | 说明 |
|------|------|
| owner | 响应接收者（客户端传入） |
| code | 响应码 |
| reply | 响应数据 |

### 使用示例

```c
typedef struct DemoClientProxy {
    INHERIT_CLIENT_IPROXY;
    // 自定义方法...
} DemoClientProxy;

typedef struct DemoClientEntry {
    INHERIT_IUNKNOWNENTRY(DemoClientProxy);
} DemoClientEntry;
```

## IServerProxy（服务端代理）

IServerProxy 是服务端用于接收和处理跨进程请求的接口。

### 结构定义

```c
// evidence: interfaces/kits/registry/iproxy_server.h:84-106
struct IServerProxy {
    // 继承 IUnknown
    INHERIT_IUNKNOWN;

    // 处理 IPC 消息
    int32 (*Invoke)(IServerProxy *iProxy, int funcId, void *origin, 
                    IpcIo *req, IpcIo *reply);
};
```

### 版本定义

```c
// evidence: interfaces/kits/registry/iproxy_server.h:62-63
#define SERVER_PROXY_VER 0x80
#define SERVER_IMPL_PROXY_VER ((uint16)SERVER_PROXY_VER | (uint16)DEFAULT_VERSION)
```

### Invoke 参数说明

| 参数 | 说明 |
|------|------|
| iProxy | 服务端代理指针 |
| funcId | 功能 ID |
| origin | 原始 IPC 消息头 |
| req | 请求数据（可从中取出参数） |
| reply | 响应数据（可向其中填充返回值） |

### 使用示例

```c
typedef struct DemoFeatureApi {
    INHERIT_SERVER_IPROXY;
    BOOL (*AsyncCall)(IUnknown *iUnknown, const char *buff);
    BOOL (*SyncCall)(IUnknown *iUnknown, struct Payload *payload);
} DemoFeatureApi;

static DemoFeatureApi g_example = {
    SERVER_IPROXY_IMPL_BEGIN,
    .Invoke = Invoke,
    .AsyncCall = AsyncCall,
    .SyncCall = SyncCall,
    IPROXY_END,
};

static int32 Invoke(IServerProxy *iProxy, int funcId, void *origin, 
                    IpcIo *req, IpcIo *reply)
{
    DemoFeatureApi *api = (DemoFeatureApi *)iProxy;
    switch (funcId) {
        case ID_ASYNCALL: {
            size_t len = 0;
            char *str = (char *)IpcIoPopString(req, &len);
            BOOL ret = api->AsyncCall((IUnknown *)iProxy, str);
            IpcIoPushBool(reply, ret);
            break;
        }
        case ID_SYNCCALL: {
            int32 id = IpcIoPopInt32(req);
            int32 value = IpcIoPopInt32(req);
            struct Payload payload = {.id = id, .value = value};
            BOOL ret = api->SyncCall((IUnknown *)iProxy, &payload);
            IpcIoPushString(reply, ret ? "TRUE" : "FALSE");
            break;
        }
        default:
            IpcIoPushBool(reply, FALSE);
            break;
    }
    return EC_SUCCESS;
}
```

## Registry API

### SAMGR_RegisterFactory

注册客户端代理工厂。

### 函数签名

```c
// evidence: interfaces/kits/registry/registry.h:104
int SAMGR_RegisterFactory(const char *service, const char *feature, 
                          Creator creator, Destroyer destroyer);
```

### 参数说明

| 参数 | 说明 |
|------|------|
| service | 服务名 |
| feature | Feature 名 |
| creator | 客户端代理创建函数 |
| destroyer | 客户端代理销毁函数 |

### Creator 类型

```c
// evidence: interfaces/kits/registry/registry.h:69
typedef void *(*Creator)(const char *service, const char *feature, uint32 size);
```

### Destroyer 类型

```c
// evidence: interfaces/kits/registry/registry.h:84
typedef void (*Destroyer)(const char *service, const char *feature, void *iproxy);
```

### 工厂示例

```c
void *DEMO_CreatClient(const char *service, const char *feature, uint32 size)
{
    uint32 len = size + sizeof(DemoClientEntry);
    uint8 *client = malloc(len);
    (void)memset_s(client, len, 0, len);
    
    DemoClientEntry *entry = (DemoClientEntry *)&client[size];
    entry->ver = CLIENT_PROXY_VER;
    entry->ref = 1;
    entry->iUnknown.QueryInterface = IUNKNOWN_QueryInterface;
    entry->iUnknown.AddRef = IUNKNOWN_AddRef;
    entry->iUnknown.Release = IUNKNOWN_Release;
    entry->iUnknown.Invoke = NULL;
    // 初始化自定义方法...
    return client;
}

void DEMO_DestroyClient(const char *service, const char *feature, void *iproxy)
{
    free(iproxy);
}

// 注册工厂
SAMGR_RegisterFactory(EXAMPLE_SERVICE, EXAMPLE_FEATURE, DEMO_CreatClient, DEMO_DestroyClient);
```

## 完整调用流程

### 服务端注册

```c
// 1. 定义 IServerProxy 实现
typedef struct DemoFeatureApi {
    INHERIT_SERVER_IPROXY;
    BOOL (*AsyncCall)(IUnknown *iUnknown, const char *buff);
    BOOL (*SyncCall)(IUnknown *iUnknown, struct Payload *payload);
} DemoFeatureApi;

static DemoFeatureApi g_example = {
    SERVER_IPROXY_IMPL_BEGIN,
    .Invoke = Invoke,
    .AsyncCall = AsyncCall,
    .SyncCall = SyncCall,
    IPROXY_END,
};

// 2. 实现 Invoke 处理消息
static int32 Invoke(IServerProxy *iProxy, int funcId, void *origin, 
                    IpcIo *req, IpcIo *reply)
{
    // 处理逻辑...
    return EC_SUCCESS;
}

// 3. 注册 Feature API（与服务端一致）
static void Init(void)
{
    SAMGR_GetInstance()->RegisterFeatureApi(EXAMPLE_SERVICE, EXAMPLE_FEATURE, 
                                             GET_IUNKNOWN(g_example));
}
SYSEX_FEATURE_INIT(Init);
```

### 客户端调用

```c
// 1. 获取客户端代理
IClientProxy *demoApi = NULL;
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(EXAMPLE_SERVICE, EXAMPLE_FEATURE);
if (iUnknown == NULL) {
    return NULL;
}
int result = iUnknown->QueryInterface(iUnknown, CLIENT_PROXY_VER, (void **)&demoApi);
if (result != 0 || demoApi == NULL) {
    return NULL;
}

// 2. 发送 IPC 请求
IpcIo request;
char data[250];
IpcIoInit(&request, data, sizeof(data), 0);
demoApi->Invoke(demoApi, ID_SYNCCALL, &request, NULL, NULL);

// 3. 释放
int32 ref = demoApi->Release((IUnknown *)demoApi);
```

### 带回调的异步调用

```c
// 回调处理
static int Callback(IOwner owner, int code, IpcIo *reply)
{
    size_t len = 0;
    char *response = (char *)IpcIoPopString(reply, &len);
    // 处理响应...
    return EC_SUCCESS;
}

// 发送异步请求
IpcIo request;
char data[MAX_DATA_LEN];
IpcIoInit(&request, data, MAX_DATA_LEN, 0);
IpcIoPushString(&request, "async data");
demoApi->Invoke(demoApi, ID_ASYNCALL, &request, owner, Callback);
```

## 消息序列化

### IpcIo 操作

| 函数 | 说明 |
|------|------|
| `IpcIoPushString()` | 写入字符串 |
| `IpcIoPushInt32()` | 写入 32 位整数 |
| `IpcIoPushBool()` | 写入布尔值 |
| `IpcIoPopString()` | 读取字符串 |
| `IpcIoPopInt32()` | 读取 32 位整数 |
| `IpcIoPopBool()` | 读取布尔值 |

**注意**：reply 最多包含 5 个对象和 200 字节数据。

## 下一章

- [广播服务 API](./07_Broadcast_API.md) - 发布/订阅模式
- [GN 构建系统](./08_Build_System.md) - 构建配置
