# 快速开始

## 环境准备

### 开发环境要求

| 组件 | 要求 |
|------|------|
| 操作系统 | Linux (推荐 Ubuntu 20.04+) |
| 构建工具 | GN + Ninja |
| 编译链 | ARM GCC / LLVM |
| Python | 3.7+ |

### 获取源码

```bash
# 克隆 OpenHarmony 仓库
git clone https://gitee.com/openharmony/systemabilitymgr_samgr_lite.git
cd systemabilitymgr_samgr_lite
```

## 构建

### 配置构建

```bash
# 设置环境变量
export PATH_TO_OHOS_ROOT=/path/to/openharmony
export PATH=$PATH_TO_OHOS_ROOT/prebuilts/gcc/linux-x86/arm/gcc-linaro-7.5.0/bin

# 使用 hb (OpenHarmony 构建工具)
hb set
# 选择: samgr_lite

# 构建
hb build -f
```

### 单独构建 samgr_lite

```bash
# 进入构建根目录
cd /path/to/build

# 配置
gn gen out/samgr_lite --args="ohos_kernel_type=\"liteos_a\""

# 构建
ninja -C out/samgr_lite samgr
```

### M-core 构建

```bash
# liteos_m 配置
gn gen out/samgr_lite_m --args="ohos_kernel_type=\"liteos_m\""

ninja -C out/samgr_lite_m samgr
```

## 编译产物

### M-core 平台

```
out/
└── samgr_lite_m/
    └── lib/
        ├── libsamgr.a          # Samgr 核心静态库
        └── libbroadcast.a      # 广播服务静态库
```

### A-core 平台

```
out/
└── samgr_lite/
    └── lib/
        ├── libsamgr.so         # Samgr 核心动态库
        ├── libbroadcast.so     # 广播服务动态库
        ├── libsamgr_server.so  # IPC 服务端
        └── libsamgr_client.so  # IPC 客户端
```

## Hello World 示例

### 完整服务示例

```c
// example_service.c
#include "samgr_lite.h"
#include "hilog_lite.h"

#define EXAMPLE_SERVICE "ExampleService"
#define MSG_SYNC 0x1001

typedef struct ExampleService {
    INHERIT_SERVICE;
    INHERIT_IUNKNOWNENTRY(DefaultFeatureApi);
    Identity identity;
} ExampleService;

// 消息处理
static BOOL MessageHandle(Service *service, Request *msg)
{
    ExampleService *example = (ExampleService *)service;
    switch (msg->msgId) {
        case MSG_SYNC:
            HILOG_INFO(HILOG_MODULE_APP, "Received MSG_SYNC");
            Response response = {.data = "ACK", .len = 0};
            SAMGR_SendResponse(msg, &response);
            return TRUE;
        default:
            break;
    }
    return FALSE;
}

// 获取服务名
static const char *GetName(Service *service)
{
    (void)service;
    return EXAMPLE_SERVICE;
}

// 初始化
static BOOL Initialize(Service *service, Identity identity)
{
    ExampleService *example = (ExampleService *)service;
    example->identity = identity;
    return TRUE;
}

// 获取任务配置
static TaskConfig GetTaskConfig(Service *service)
{
    (void)service;
    TaskConfig config = {LEVEL_HIGH, PRI_BELOW_NORMAL, 0x800, 20, SHARED_TASK};
    return config;
}

// 定义服务对象
static ExampleService g_example = {
    .GetName = GetName,
    .Initialize = Initialize,
    .MessageHandle = MessageHandle,
    .GetTaskConfig = GetTaskConfig,
    SERVER_IPROXY_IMPL_BEGIN,
    .Invoke = NULL,
    IPROXY_END,
};

// 注册服务
static void Init(void)
{
    SAMGR_GetInstance()->RegisterService((Service *)&g_example);
    SAMGR_GetInstance()->RegisterDefaultFeatureApi(EXAMPLE_SERVICE, GET_IUNKNOWN(g_example));
}

// 初始化入口
SYSEX_SERVICE_INIT(Init);
```

### 客户端调用示例

```c
// example_client.c
#include "samgr_lite.h"
#include "hilog_lite.h"

#define EXAMPLE_SERVICE "ExampleService"
#define MSG_SYNC 0x1001

static void ResponseHandler(const Request *request, const Response *response)
{
    HILOG_INFO(HILOG_MODULE_APP, "Response: %s", (char *)response->data);
}

void ClientExample(void)
{
    // 获取服务 API
    IUnknown *iUnknown = SAMGR_GetInstance()->GetDefaultFeatureApi(EXAMPLE_SERVICE);
    if (iUnknown == NULL) {
        HILOG_ERROR(HILOG_MODULE_APP, "Failed to get service API");
        return;
    }

    DefaultFeatureApi *api = NULL;
    int result = iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&api);
    if (result != 0 || api == NULL) {
        HILOG_ERROR(HILOG_MODULE_APP, "Failed to query interface");
        return;
    }

    // 发送请求（示例，实际需要获取 Identity）
    // Request request = {
    //     .msgId = MSG_SYNC,
    //     .data = NULL,
    //     .len = 0,
    //     .msgValue = 0,
    // };
    // SAMGR_SendRequest(&targetIdentity, &request, ResponseHandler);

    // 释放
    api->Release((IUnknown *)api);
}
```

## 调试技巧

### 日志输出

```c
#include "hilog_lite.h"

HILOG_INFO(HILOG_MODULE_APP, "Info message: %d", value);
HILOG_WARN(HILOG_MODULE_APP, "Warning message");
HILOG_ERROR(HILOG_MODULE_APP, "Error message: %s", errorStr);
```

### 查看服务列表

```c
// 在服务中调用
SAMGR_PrintServices();  // 打印所有已注册服务
SAMGR_PrintOperations();  // 打印所有操作
```

## 常见问题

### Q: 服务注册失败

A: 检查以下内容：
1. 服务名是否 < 16 字节且为常量字符串
2. 所有函数指针是否已初始化
3. SYSEX_SERVICE_INIT 宏是否正确定义

### Q: 消息无法送达

A: 检查以下内容：
1. Identity 是否有效（serviceId, featureId, queueId）
2. 目标服务是否已初始化
3. 消息队列是否已满

### Q: 跨进程调用失败

A: 检查以下内容：
1. 是否在 A-core 平台
2. 服务是否注册了 IServerProxy
3. 权限配置是否正确

## 相关文档

- [SamgrLite API](./04_SamgrLite_API.md)
- [消息通信 API](./05_Message_API.md)
- [IPC 跨进程接口](./06_IPC_API.md)
