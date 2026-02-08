# 最佳实践

## 服务开发规范

### 1. 服务名命名规范

```c
// ✅ 正确
#define EXAMPLE_SERVICE "ExampleService"
#define FEATURE_NAME   "Feature"

// ❌ 错误 - 超过 16 字节
#define BAD_SERVICE "ThisIsAVeryLongServiceName"

// ❌ 错误 - 非常量
char serviceName[20] = "DynamicName";  // 禁止使用变量
```

**约束来源**: `README.md` 约束章节

### 2. 结构体初始化

```c
// ✅ 正确 - 使用 designated initializer
static ExampleService g_example = {
    .GetName = GetName,
    .Initialize = Initialize,
    .MessageHandle = MessageHandle,
    .GetTaskConfig = GetTaskConfig,
};

// ❌ 错误 - 顺序初始化容易出错
static ExampleService g_bad = {GetName, Initialize, MessageHandle, GetTaskConfig};
```

### 3. 错误处理

```c
// ✅ 正确 - 显式检查返回值
BOOL Initialize(Service *service, Identity identity)
{
    if (service == NULL || identity.queueId == NULL) {
        return FALSE;
    }
    
    // 分配资源
    Resource *res = AllocateResource();
    if (res == NULL) {
        return FALSE;
    }
    
    return TRUE;
}

// ❌ 错误 - 忽略返回值
void BadInitialize(Service *service, Identity identity)
{
    AllocateResource();  // 忽略返回值
}
```

## 消息处理规范

### 1. 消息 ID 定义

```c
// ✅ 正确 - 使用有意义的枚举
typedef enum {
    MSG_START = 0x1000,    // 开始
    MSG_STOP,              // 停止
    MSG_DATA,              // 数据
    MSG_SYNC,              // 同步请求
    MSG_ASYNC,             // 异步请求
} MessageId;

// ❌ 错误 - 使用魔法数字
if (request->msgId == 0x1234) {  // 难以理解
```

### 2. 消息验证

```c
BOOL MessageHandle(Service *service, Request *request)
{
    // ✅ 验证消息 ID
    if (!IsValidMessageId(request->msgId)) {
        return FALSE;
    }
    
    // ✅ 验证数据长度
    if (request->len > MAX_DATA_SIZE) {
        return FALSE;
    }
    
    // ✅ 验证数据指针
    if (request->len > 0 && request->data == NULL) {
        return FALSE;
    }
    
    // 安全处理
    ProcessMessage(service, request);
    return TRUE;
}
```

### 3. 内存管理

```c
void ResponseHandler(const Request *request, const Response *response)
{
    // ✅ 正确 - 检查空指针
    if (response == NULL || response->data == NULL) {
        return;
    }
    
    // 使用数据
    ProcessData(response->data, response->len);
    
    // 注意：数据由 Samgr 管理，不要手动释放
}
```

## 并发编程规范

### 1. 消息队列优于共享内存

```c
// ✅ 正确 - 使用消息传递
static void SendWork(Service *service, WorkItem *item)
{
    Request request = {
        .msgId = MSG_WORK,
        .data = item,
        .len = sizeof(WorkItem),
    };
    SAMGR_SendRequest(&service->identity, &request, NULL);
}

// ❌ 错误 - 直接修改共享状态
static void ModifySharedState(Service *service, int value)
{
    service->sharedCounter = value;  // 竞态条件
}
```

### 2. 避免阻塞消息循环

```c
// ✅ 正确 - 快速返回
BOOL MessageHandle(Service *service, Request *request)
{
    // 发起异步操作后立即返回
    if (request->msgId == MSG_ASYNC_START) {
        StartAsyncOperation(service, request);
        return TRUE;
    }
    return TRUE;
}

// ❌ 错误 - 长时间阻塞
BOOL BadMessageHandle(Service *service, Request *request)
{
    // 阻塞 10 秒
    SAMGR_Sleep(10);  // 会阻塞整个服务
    return TRUE;
}
```

## 内存安全规范

### 1. 使用安全字符串函数

```c
// ✅ 正确 - 使用安全函数
char buffer[64];
strcpy_s(buffer, sizeof(buffer), src);           // 复制字符串
strcat_s(buffer, sizeof(buffer), suffix);        // 拼接字符串
int len = strnlen_s(src, sizeof(buffer));        // 安全获取长度

// ❌ 错误 - 使用不安全函数
strcpy(buffer, src);     // 缓冲区溢出
strcat(buffer, suffix); // 缓冲区溢出
strlen(src);             // 未检查长度
```

### 2. 内存复制

```c
// ✅ 正确 - 使用带大小检查的复制
memcpy_s(dst, dstSize, src, srcSize);

// ❌ 错误 - 无检查复制
memcpy(dst, src, len);  // 可能溢出
```

### 3. 指针验证

```c
// ✅ 正确 - 验证指针
BOOL ProcessData(Service *service, Request *request)
{
    if (request == NULL || request->data == NULL) {
        return FALSE;
    }
    
    if (!IsMemoryReadable(request->data, request->len)) {
        return FALSE;
    }
    
    // 安全使用
    return TRUE;
}
```

## API 设计规范

### 1. 接口版本管理

```c
// ✅ 正确 - 版本化接口
#define MY_API_VER 0x20

typedef struct MyApi {
    INHERIT_IUNKNOWN;
    BOOL (*DoSomething)(IUnknown *iUnknown, int32 param);
    BOOL (*DoOther)(IUnknown *iUnknown, const char *data);
} MyApi;

static const MyApi g_api = {
    DEFAULT_IUNKNOWN_ENTRY_BEGIN,
        .DoSomething = DoSomethingImpl,
        .DoOther = DoOtherImpl,
    DEFAULT_IUNKNOWN_ENTRY_END,
};
```

### 2. 错误码规范

```c
// ✅ 正确 - 统一的错误码
typedef enum {
    MY_EC_SUCCESS = 0,
    MY_EC_INVALID_PARAM,
    MY_EC_NOT_FOUND,
    MY_EC_ALREADY_EXISTS,
    MY_EC_NO_MEMORY,
    MY_EC_TIMEOUT,
} MyErrorCode;

// 使用系统统一错误码
int32 result = SAMGR_SendRequest(...);
if (result != EC_SUCCESS) {
    // 处理错误
}
```

## 性能优化

### 1. 消息批处理

```c
// ✅ 正确 - 批处理多个操作
typedef struct BatchRequest {
    int16 count;
    Operation ops[MAX_BATCH_SIZE];
} BatchRequest;

BOOL BatchHandle(Service *service, Request *request)
{
    BatchRequest *batch = (BatchRequest *)request->data;
    for (int i = 0; i < batch->count; i++) {
        ProcessOperation(&batch->ops[i]);
    }
    return TRUE;
}
```

### 2. 避免频繁创建对象

```c
// ✅ 正确 - 复用对象
static WorkItem g_workItems[MAX_ITEMS];
static int16 g_nextItem = 0;

WorkItem *GetWorkItem(void)
{
    return &g_workItems[g_nextItem++ % MAX_ITEMS];
}

// ❌ 错误 - 每次创建新对象
WorkItem *item = malloc(sizeof(WorkItem));  // 频繁分配/释放
```

### 3. 选择合适的任务类型

```c
// ✅ 正确 - 根据需求选择
static TaskConfig GetTaskConfig(Service *service)
{
    // 高优先级实时任务
    if (service->priority == PRI_REALTIME) {
        return (TaskConfig){
            .level = LEVEL_HIGH,
            .priority = PRI_NORMAL,
            .queueSize = 50,
            .taskFlags = SINGLE_TASK,  // 独占任务
        };
    }
    
    // 普通任务使用共享
    return (TaskConfig){
        .level = LEVEL_MIDDLE,
        .priority = PRI_BELOW_NORMAL,
        .queueSize = 20,
        .taskFlags = SHARED_TASK,  // 共享任务
    };
}
```

## 日志规范

### 1. 合适的日志级别

```c
// ✅ 正确 - 使用合适的级别
HILOG_DEBUG(HILOG_MODULE_APP, "Entering function X");   // 调试信息
HILOG_INFO(HILOG_MODULE_APP, "Operation completed");    // 普通信息
HILOG_WARN(HILOG_MODULE_APP, "High latency detected");  // 警告
HILOG_ERROR(HILOG_MODULE_APP, "Failed to open file");    // 错误
```

### 2. 日志内容

```c
// ✅ 正确 - 包含上下文信息
HILOG_INFO(HILOG_MODULE_APP, "Request processed: id=%d, size=%d",
           request->msgId, request->len);

// ❌ 错误 - 信息不足
HILOG_INFO(HILOG_MODULE_APP, "Done");  // 难以追踪问题
```

## 代码组织规范

### 1. 文件结构

```
my_service/
├── include/
│   ├── my_service.h          // 公共头文件
│   └── my_types.h           // 类型定义
├── src/
│   ├── my_service.c          // 服务实现
│   ├── my_feature.c          // Feature 实现
│   └── my_client.c           // 客户端示例
└── BUILD.gn                  // 构建配置
```

### 2. 头文件保护

```c
// ✅ 正确
#ifndef MY_SERVICE_H
#define MY_SERVICE_H

// 头文件内容

#endif // MY_SERVICE_H
```

## 测试规范

### 1. 模块化测试

```c
// 测试消息处理
void TestMessageHandle(void)
{
    Service service = {...};
    Request request = {...};
    
    BOOL result = MessageHandle(&service, &request);
    
    TEST_ASSERT_TRUE(result);
    TEST_ASSERT_EQUAL(EXPECTED_STATE, service.state);
}
```

## 相关文档

- [安全风险评审](./12_Security_Review.md)
- [常见问题](./appendix/FAQ.md)
- [术语表](./appendix/Glossary.md)
