# 生命周期管理

## Service 生命周期

### 生命周期状态

```
┌─────────┐     Initialize()     ┌─────────┐     MessageHandle()    ┌─────────┐
│   INIT  │ ──────────────────► │  READY  │ ──────────────────► │ RUNNING │
└─────────┘                      └─────────┘                       └────┬────┘
     │                                │                                   │
     │                                │                                   │
     │                                │                                   │ UnregisterService()
     │                                │                                   ▼
     │                                │                              ┌─────────┐
     │                                └──────────────────────────────►│  STOP   │
     │                                                                   └─────────┘
     │                                                                     │
     │                                                                     ▼
     │                                                                ┌─────────┐
     └───────────────────────────────────────────────────────────────►│ UNREG   │
                                                                   └─────────┘
```

### 状态说明

| 状态 | 说明 |
|------|------|
| INIT | 构造函数已调用，等待 Initialize |
| READY | 已初始化，等待任务分配 |
| RUNNING | 任务已分配，正在处理消息 |
| STOP | 已停止，不再接收新消息 |
| UNREG | 已注销，从 Samgr 中移除 |

### 初始化流程

```c
// evidence: samgr/source/service.c
BOOL RegisterService(Service *service)
{
    // 1. 验证 Service 结构
    if (service->GetName == NULL ||
        service->Initialize == NULL ||
        service->MessageHandle == NULL ||
        service->GetTaskConfig == NULL) {
        return FALSE;
    }

    // 2. 创建任务
    TaskConfig config = service->GetTaskConfig(service);
    TaskPool *pool = CreateTask(service, config);
    if (pool == NULL) {
        return FALSE;
    }

    // 3. 调用 Initialize
    Identity identity = {...};
    BOOL result = service->Initialize(service, identity);
    if (!result) {
        DestroyTask(pool);
        return FALSE;
    }

    // 4. 启动消息循环
    StartMessageLoop(service);

    return TRUE;
}
```

### 消息处理

```c
// evidence: samgr/source/message.c
void ServiceMessageLoop(Service *service)
{
    while (service->status != STATUS_STOPPED) {
        // 1. 等待消息
        MessageWrapper *msg = WaitMessage(service->queueId);

        // 2. 处理消息
        if (msg != NULL) {
            BOOL result = service->MessageHandle(service, &msg->request);

            // 3. 发送响应（如果有）
            if (msg->handler != NULL && msg->responseNeeded) {
                SendResponse(msg);
            }

            FreeMessage(msg);
        }
    }
}
```

### 停止服务

```c
// evidence: samgr/source/service.c
Service *UnregisterService(const char *name)
{
    // 1. 查找服务
    ServiceWrapper *wrapper = FindServiceByName(name);
    if (wrapper == NULL) {
        return NULL;
    }

    // 2. 停止消息循环
    wrapper->service->status = STATUS_STOPPED;

    // 3. 注销所有 Feature
    UnregisterAllFeatures(wrapper);

    // 4. 销毁任务
    DestroyTask(wrapper->pool);

    // 5. 从向量移除
    VECTOR_Swap(&g_services, wrapper->index, NULL);

    return wrapper->service;
}
```

## Feature 生命周期

### 生命周期状态

```
┌─────────┐   OnInitialize()    ┌─────────┐    OnMessage()     ┌─────────┐
│   INIT  │ ──────────────────► │  READY  │ ─────────────────► │ ACTIVE  │
└─────────┘                      └─────────┘                    └────┬────┘
     │                                │                              │
     │                                │                              │ OnStop()
     │                                │                              ▼
     │                                │                         ┌─────────┐
     │                                └────────────────────────►│  STOP   │
     │                                                          └─────────┘
     │                                                            │
     │                                                            ▼
     │                                                       ┌─────────┐
     └───────────────────────────────────────────────────────►│ UNREG   │
                                                            └─────────┘
```

### 状态说明

| 状态 | 说明 |
|------|------|
| INIT | 构造函数已调用，等待 OnInitialize |
| READY | 已关联父服务 |
| ACTIVE | 正在处理消息 |
| STOP | 已停止 |
| UNREG | 已注销 |

### 初始化流程

```c
// evidence: samgr/source/feature.c
void FeatureOnInitialize(Feature *feature, Service *parent, Identity identity)
{
    // 1. 保存父服务引用
    feature->parent = parent;

    // 2. 保存身份标识
    feature->identity = identity;

    // 3. 调用用户初始化
    feature->OnInitialize(feature, parent, identity);
}
```

### 消息处理

```c
// evidence: samgr/source/feature.c
BOOL FeatureOnMessage(Feature *feature, Request *request)
{
    // 1. 检查 Feature 是否已停止
    if (feature->status == STATUS_STOPPED) {
        return FALSE;
    }

    // 2. 调用用户消息处理
    return feature->OnMessage(feature, request);
}
```

### 停止 Feature

```c
// evidence: samgr/source/feature.c
void FeatureOnStop(Feature *feature, Identity identity)
{
    // 1. 调用用户停止
    feature->OnStop(feature, identity);

    // 2. 清理资源
    feature->status = STATUS_STOPPED;
}
```

## Bootstrap 生命周期

### 引导消息

```c
// evidence: samgr_lite.h:79-88
typedef enum BootMessage {
    BOOT_SYS_COMPLETED,      // 系统服务初始化完成
    BOOT_APP_COMPLETED,       // 应用服务初始化完成
    BOOT_REG_SERVICE,         // 动态注册服务
    BOOTSTRAP_BUTT,           // 消息数量上限
} BootMessage;
```

### 引导流程

```c
// evidence: samgr/source/samgr_lite.c
void SAMGR_Bootstrap(void)
{
    // 1. 初始化 Samgr 实例
    InitSamgrInstance();

    // 2. 初始化任务池
    InitSharedPools();

    // 3. 加载系统服务（通过 SYSEX_SERVICE_INIT）
    LoadSystemServices();

    // 4. 发送 BOOT_SYS_COMPLETED
    SendBootMessage(BOOT_SYS_COMPLETED);

    // 5. 加载应用服务
    LoadAppServices();

    // 6. 发送 BOOT_APP_COMPLETED
    SendBootMessage(BOOT_APP_COMPLETED);
}
```

## 初始化宏

### SYSEX_SERVICE_INIT

```c
// 定义服务初始化入口
#define SYSEX_SERVICE_INIT(InitFunc) \
    static void InitFunction(void) __attribute__((constructor)); \
    static void InitFunction(void) { InitFunc(); }
```

**使用示例**:
```c
static void Init(void)
{
    SAMGR_GetInstance()->RegisterService((Service *)&g_example);
}
SYSEX_SERVICE_INIT(Init);
```

### SYSEX_FEATURE_INIT

```c
// 定义 Feature 初始化入口
#define SYSEX_FEATURE_INIT(InitFunc) \
    static void FeatureInitFunction(void) __attribute__((constructor)); \
    static void FeatureInitFunction(void) { InitFunc(); }
```

**使用示例**:
```c
static void Init(void)
{
    SAMGR_GetInstance()->RegisterFeature(SERVICE_NAME, (Feature *)&g_feature);
}
SYSEX_FEATURE_INIT(Init);
```

## 生命周期回调时序

### 服务启动时序

```
时间线:
─────────────────────────────────────────────────────────────────────►
  │              │              │              │
  │   Bootstrap() │              │              │
  │      │        │              │              │
  │      ▼        │              │              │
  │  InitSamgr    │              │              │
  │      │        │              │              │
  │      ▼        │              │              │
  │  InitPools    │              │              │
  │      │        │              │              │
  │      ▼        │              │              │
  │  LoadServices │              │              │
  │      │        │              │              │
  │      │        ▼              │              │
  │      │  InitFunction()       │              │
  │      │      │                │              │
  │      │      ▼                │              │
  │      │  RegisterService()    │              │
  │      │      │                │              │
  │      │      ▼                │              │
  │      │  CreateTask()         │              │
  │      │      │                │              │
  │      │      ▼                │              │
  │      │  Initialize() ◄───────┘              │
  │      │      │                              │
  │      │      ▼                              │
  │      │  MessageLoop() ◄────────────────────┘
```

### 消息处理时序

```
时间线:
─────────────────────────────────────────────────────────────────────►
  │              │
  │   SendReq()  │
  │      │        │
  │      ▼        │
  │  Enqueue()    │
  │      │        │
  │      ▼        │
  │  Dequeue() ◄──┘
  │      │
  │      ▼
  │  Handle() ◄──── Feature/Service
  │      │
  │      ▼
  │  SendResp()
```

## 最佳实践

### 资源管理

```c
static BOOL Initialize(Service *service, Identity identity)
{
    MyService *my = (MyService *)service;
    
    // 1. 保存身份标识
    my->identity = identity;
    
    // 2. 分配资源
    my->buffer = malloc(BUFFER_SIZE);
    if (my->buffer == NULL) {
        return FALSE;
    }
    
    // 3. 初始化状态
    my->state = STATE_READY;
    
    return TRUE;
}

static void OnStop(Service *service)
{
    MyService *my = (MyService *)service;
    
    // 释放资源
    free(my->buffer);
    my->buffer = NULL;
    
    // 重置状态
    my->state = STATE_STOPPED;
}
```

### 线程安全

```c
static BOOL Initialize(Service *service, Identity identity)
{
    MyService *my = (MyService *)service;
    
    // 初始化互斥锁
    my->mutex = SAMGR_CreateMutex();
    if (my->mutex == NULL) {
        return FALSE;
    }
    
    return TRUE;
}

static BOOL MessageHandle(Service *service, Request *request)
{
    MyService *my = (MyService *)service;
    
    // 加锁保护共享状态
    SAMGR_LockMutex(my->mutex);
    
    // 访问共享数据
    ProcessData(my);
    
    SAMGR_UnlockMutex(my->mutex);
    
    return TRUE;
}
```

## 下一章

- [最佳实践](./13_Best_Practices.md) - 开发规范与注意事项
- [安全风险评审](./12_Security_Review.md) - 安全考量
