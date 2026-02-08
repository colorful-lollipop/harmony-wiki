# 内部实现

## SamgrLite 实现

### SamgrLiteImpl 结构

```c
// evidence: samgr_lite_inner.h:54-60
struct SamgrLiteImpl {
    SamgrLite vtbl;           // 函数表
    MutexId mutex;           // 互斥锁
    BootStatus status;       // 启动状态
    Vector services;         // 服务向量
    TaskPool *sharedPool[MAX_POOL_NUM];  // 共享任务池
};
```

### 单例获取

```c
// evidence: samgr/source/samgr_lite.c
SamgrLite *SAMGR_GetInstance(void)
{
    static SamgrLiteImpl instance = {
        .vtbl = {
            .RegisterService = RegisterService,
            .UnregisterService = UnregisterService,
            // ...
        },
    };
    return &instance;
}
```

## 服务管理实现

### 服务注册

```c
// evidence: samgr/source/service.c
BOOL RegisterService(Service *service)
{
    // 1. 验证服务
    if (service == NULL || service->GetName == NULL) {
        return FALSE;
    }

    // 2. 检查重复
    if (FindServiceByName(service->GetName(service)) != NULL) {
        return FALSE;
    }

    // 3. 创建任务（如果需要）
    Identity identity = AssignIdentity(service);
    CreateTask(service, identity);

    // 4. 添加到向量
    ServiceWrapper *wrapper = CreateWrapper(service, identity);
    VECTOR_Add(&g_services, wrapper);

    return TRUE;
}
```

### 服务发现

```c
// evidence: samgr/source/service.c
Service *FindServiceByName(const char *serviceName)
{
    for (int16 i = 0; i < VECTOR_Num(&g_services); i++) {
        ServiceWrapper *wrapper = VECTOR_At(&g_services, i);
        if (strcmp(wrapper->service->GetName(wrapper->service), serviceName) == 0) {
            return wrapper->service;
        }
    }
    return NULL;
}
```

## 消息路由实现

### 消息发送

```c
// evidence: samgr/source/message.c
int32 SAMGR_SendRequest(const Identity *identity, const Request *request, Handler handler)
{
    // 1. 验证身份
    if (!IsValidIdentity(identity)) {
        return EC_INVALID;
    }

    // 2. 创建消息
    MessageWrapper *wrapper = CreateMessage(request, handler);
    if (wrapper == NULL) {
        return EC_NOMEMORY;
    }

    // 3. 发送到队列
    int32 ret = SAMGR_PushQueue(identity->queueId, wrapper);
    if (ret != EC_SUCCESS) {
        FreeMessage(wrapper);
        return ret;
    }

    return EC_SUCCESS;
}
```

### 消息处理循环

```c
// evidence: samgr/source/task_manager.c
void MessageProcessingLoop(Service *service)
{
    while (1) {
        // 1. 从队列接收消息
        MessageWrapper *wrapper = NULL;
        int32 ret = SAMGR_PopQueue(service->queueId, &wrapper, WAIT_FOREVER);
        if (ret != EC_SUCCESS) {
            continue;
        }

        // 2. 调用 MessageHandle
        BOOL handled = service->MessageHandle(service, &wrapper->request);

        // 3. 如果有响应回调
        if (wrapper->handler != NULL && wrapper->request.msgValue != 0) {
            Response response = {...};
            SAMGR_SendResponse(&wrapper->request, &response);
        }

        // 4. 释放消息
        FreeMessage(wrapper);
    }
}
```

## Feature 管理实现

### Feature 注册

```c
// evidence: samgr/source/feature.c
BOOL RegisterFeature(const char *serviceName, Feature *feature)
{
    // 1. 查找服务
    ServiceWrapper *wrapper = FindServiceByName(serviceName);
    if (wrapper == NULL) {
        return FALSE;
    }

    // 2. 创建 Feature 包装
    FeatureWrapper *featureWrapper = CreateFeatureWrapper(feature, wrapper);

    // 3. 添加到服务的 Feature 列表
    VECTOR_Add(&wrapper->features, featureWrapper);

    return TRUE;
}
```

## IUnknown 默认实现

### QueryInterface

```c
// evidence: samgr/source/iunknown.c
int IUNKNOWN_QueryInterface(IUnknown *iUnknown, int version, void **target)
{
    IUnknownEntry *entry = GET_OBJECT(iUnknown, IUnknownEntry, iUnknown);
    
    // 版本匹配检查
    if ((version & 0xFF00) != (entry->ver & 0xFF00)) {
        return EC_FAILURE;
    }
    
    // 返回接口
    *target = entry;
    entry->ref++;
    return EC_SUCCESS;
}
```

### AddRef/Release

```c
// evidence: samgr/source/iunknown.c
int IUNKNOWN_AddRef(IUnknown *iUnknown)
{
    IUnknownEntry *entry = GET_OBJECT(iUnknown, IUnknownEntry, iUnknown);
    return ++entry->ref;
}

int IUNKNOWN_Release(IUnknown *iUnknown)
{
    IUnknownEntry *entry = GET_OBJECT(iUnknown, IUnknownEntry, iUnknown);
    if (--entry->ref <= 0) {
        // 注意：默认实现不释放内存
        // 子类可重写此函数以释放内存
    }
    return entry->ref;
}
```

## 任务池管理

### TaskPool 结构

```c
// evidence: samgr/source/task_manager.h
typedef struct TaskPool {
    TaskId taskId;           // 任务 ID
    MQueueId queueId;         // 消息队列 ID
    TaskType type;           // 任务类型
    int16 level;             // 优先级级别
} TaskPool;
```

### 共享任务分配

```c
// evidence: samgr/source/task_manager.c
TaskPool *AllocateSharedTask(TaskConfig config)
{
    // 1. 查找匹配的共享任务池
    TaskPool *pool = FindAvailablePool(config.level);
    if (pool != NULL) {
        return pool;
    }

    // 2. 创建新的共享任务池
    pool = CreateTaskPool(config.level);
    if (pool == NULL) {
        return NULL;
    }

    // 3. 启动处理线程
    StartPoolThread(pool);
    
    return pool;
}
```

## 向量容器实现

### Vector 结构

```c
// evidence: common.h:105-126
typedef struct SimpleVector {
    int16 max;              // 最大容量
    int16 top;              // 当前元素数（含已释放）
    int16 fre;              // 已释放数量
    void **data;            // 数据指针数组
    VECTOR_Key key;         // 键提取函数
    VECTOR_Compare compare; // 比较函数
} Vector;
```

### Add 操作

```c
// evidence: samgr/source/common.c
int16 VECTOR_Add(Vector *vector, void *element)
{
    if (vector->top >= vector->max) {
        // 尝试扩容
        if (!ExpandVector(vector)) {
            return INVALID_INDEX;
        }
    }
    
    // 添加元素
    vector->data[vector->top++] = element;
    return vector->top - 1;
}
```

## IPC 端点实现

### Endpoint 结构

```c
// evidence: samgr_endpoint/source/endpoint.h
typedef struct Endpoint {
    int32 handle;           // 端点句柄
    MQueueId queueId;       // 消息队列
    Vector clients;         // 客户端列表
    TokenBucket *tokenBucket; // 限流令牌桶
} Endpoint;
```

### 消息路由

```c
// evidence: samgr_endpoint/source/endpoint_rpc.c
int32 RouteMessage(IpcIo *req, IpcIo *reply)
{
    // 1. 解析消息头
    MessageHeader *header = (MessageHeader *)req->data;
    
    // 2. 查找目标服务
    ServiceWrapper *wrapper = FindServiceBySaId(header->saId);
    if (wrapper == NULL) {
        return EC_NOSERVICE;
    }
    
    // 3. 限流检查
    if (!TokenBucketConsume(endpoint->tokenBucket)) {
        return EC_BUSY;
    }
    
    // 4. 路由到服务端
    return DispatchToServer(wrapper, req, reply);
}
```

## 下一章

- [生命周期管理](./11_Lifecycle.md) - 服务/Feature 生命周期详解
- [安全风险评审](./12_Security_Review.md) - 安全考量
