# SamgrLite API

## 概述

SamgrLite 是 samgr_lite 的核心管理类，提供服务的注册、发现和系统能力管理功能。

## API 清单

### 获取单例

| API | 说明 | 同步/异步 | 头文件 |
|-----|------|----------|--------|
| `SAMGR_GetInstance()` | 获取 SamgrLite 单例 | 同步 | `samgr_lite.h:299` |

**函数签名**:
```c
SamgrLite *SAMGR_GetInstance(void);
```

**返回值**:
- 成功：SamgrLite 实例指针
- 失败：NULL

**使用示例**:
```c
SamgrLite *samgr = SAMGR_GetInstance();
if (samgr == NULL) {
    // 处理错误
}
```

### 服务生命周期管理

| API | 说明 | 同步/异步 | 头文件 |
|-----|------|----------|--------|
| `SAMGR_Bootstrap()` | 启动系统服务 | 同步 | `samgr_lite.h:315` |
| `RegisterService()` | 注册服务 | 同步 | `samgr_lite.h:111` |
| `UnregisterService()` | 注销服务 | 同步 | `samgr_lite.h:125` |
| `RegisterFeature()` | 注册 Feature | 同步 | `samgr_lite.h:139` |
| `UnregisterFeature()` | 注销 Feature | 同步 | `samgr_lite.h:155` |

**RegisterService** - `samgr_lite.h:111`
```c
BOOL (*RegisterService)(Service *service);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| service | Service* | 要注册的服务 |

| 返回值 | 说明 |
|--------|------|
| TRUE | 注册成功 |
| FALSE | 注册失败 |

**UnregisterService** - `samgr_lite.h:125`
```c
Service *(*UnregisterService)(const char *name);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| name | const char* | 要注销的服务名 |

| 返回值 | 说明 |
|--------|------|
| 非 NULL | 注销成功，返回服务对象（调用者负责释放） |
| NULL | 注销失败 |

**RegisterFeature** - `samgr_lite.h:139`
```c
BOOL (*RegisterFeature)(const char *serviceName, Feature *feature);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| serviceName | const char* | 所属服务名 |
| feature | Feature* | 要注册的 Feature |

| 返回值 | 说明 |
|--------|------|
| TRUE | 注册成功 |
| FALSE | 注册失败 |

### API 注册与发现

| API | 说明 | 同步/异步 | 头文件 |
|-----|------|----------|--------|
| `RegisterDefaultFeatureApi()` | 注册默认 Feature API | 同步 | `samgr_lite.h:172` |
| `UnregisterDefaultFeatureApi()` | 注销默认 Feature API | 同步 | `samgr_lite.h:187` |
| `RegisterFeatureApi()` | 注册 Feature API | 同步 | `samgr_lite.h:204` |
| `UnregisterFeatureApi()` | 注销 Feature API | 同步 | `samgr_lite.h:218` |
| `GetDefaultFeatureApi()` | 获取默认 Feature API | 同步 | `samgr_lite.h:232` |
| `GetFeatureApi()` | 获取 Feature API | 同步 | `samgr_lite.h:249` |
| `GetRemoteDefaultFeatureApi()` | 获取远程默认 Feature API (可选) | 同步 | `samgr_lite.h:234` |

**RegisterDefaultFeatureApi** - `samgr_lite.h:172`
```c
BOOL (*RegisterDefaultFeatureApi)(const char *service, IUnknown *publicApi);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| service | const char* | 服务名 |
| publicApi | IUnknown* | 要注册的 API 接口 |

**GetDefaultFeatureApi** - `samgr_lite.h:232`
```c
IUnknown *(*GetDefaultFeatureApi)(const char *service);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| service | const char* | 服务名 |

| 返回值 | 说明 |
|--------|------|
| 非 NULL | API 接口指针 |
| NULL | 获取失败 |

### 系统能力管理

| API | 说明 | 同步/异步 | 头文件 |
|-----|------|----------|--------|
| `AddSystemCapability()` | 添加系统能力 | 同步 | `samgr_lite.h:261` |
| `HasSystemCapability()` | 检查系统能力是否存在 | 同步 | `samgr_lite.h:273` |
| `GetSystemAvailableCapabilities()` | 获取所有可用系统能力 | 同步 | `samgr_lite.h:287` |

**AddSystemCapability** - `samgr_lite.h:261`
```c
int32 (*AddSystemCapability)(const char *sysCap);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| sysCap | const char* | 系统能力名（长度 < 64） |

| 返回值 | 说明 |
|--------|------|
| EC_SUCCESS | 添加成功 |
| 其他错误码 | 添加失败 |

**HasSystemCapability** - `samgr_lite.h:273`
```c
BOOL (*HasSystemCapability)(const char *sysCap);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| sysCap | const char* | 系统能力名 |

| 返回值 | 说明 |
|--------|------|
| TRUE | 能力存在 |
| FALSE | 能力不存在 |

## 完整使用示例

### 服务注册

```c
// 定义服务
static ExampleService g_example = {
    .GetName = GetName,
    .Initialize = Initialize,
    .MessageHandle = MessageHandle,
    .GetTaskConfig = GetTaskConfig,
    SERVER_IPROXY_IMPL_BEGIN,
        .Invoke = NULL,
        .SyncCall = SyncCall,
    IPROXY_END,
};

// 注册服务和默认 API
static void Init(void)
{
    SAMGR_GetInstance()->RegisterService((Service *)&g_example);
    SAMGR_GetInstance()->RegisterDefaultFeatureApi(EXAMPLE_SERVICE, GET_IUNKNOWN(g_example));
}

// 定义初始化入口
SYSEX_SERVICE_INIT(Init);
```

### Feature 注册

```c
// 定义 Feature
static DemoFeature g_example = {
    .GetName = FEATURE_GetName,
    .OnInitialize = FEATURE_OnInitialize,
    .OnStop = FEATURE_OnStop,
    .OnMessage = FEATURE_OnMessage,
    DEFAULT_IUNKNOWN_ENTRY_BEGIN,
        .AsyncCall = AsyncCall,
        .SyncCall = SyncCall,
    DEFAULT_IUNKNOWN_ENTRY_END,
    .identity = {-1, -1, NULL},
};

// 注册 Feature 和 API
static void Init(void)
{
    SAMGR_GetInstance()->RegisterFeature(EXAMPLE_SERVICE, (Feature *)&g_example);
    SAMGR_GetInstance()->RegisterFeatureApi(EXAMPLE_SERVICE, EXAMPLE_FEATURE, GET_IUNKNOWN(g_example));
}

// 定义初始化入口
SYSEX_FEATURE_INIT(Init);
```

### 获取并使用 API

```c
// 获取默认 Feature API
DemoApi *demoApi = NULL;
IUnknown *iUnknown = SAMGR_GetInstance()->GetDefaultFeatureApi(EXAMPLE_SERVICE);
if (iUnknown == NULL) {
    return NULL;
}
int result = iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&demoApi);
if (result != 0 || demoApi == NULL) {
    return NULL;
}

// 使用 API
demoApi->SyncCall((IUnknown *)demoApi, &payload);

// 释放
int32 ref = demoApi->Release((IUnknown *)demoApi);
```

### 系统能力管理

```c
// 添加系统能力
int32 ret = SAMGR_GetInstance()->AddSystemCapability("SampleCapability");
if (ret != EC_SUCCESS) {
    // 处理错误
}

// 检查系统能力
BOOL exists = SAMGR_GetInstance()->HasSystemCapability("SampleCapability");
if (exists) {
    // 能力存在
}

// 获取所有可用能力
char sysCaps[MAX_SYSCAP_NUM][MAX_SYSCAP_NAME_LEN];
int32_t sysCapNum = 0;
ret = SAMGR_GetInstance()->GetSystemAvailableCapabilities(sysCaps, &sysCapNum);
```

## 约束与限制

| 约束 | 说明 | 证据 |
|------|------|------|
| 服务名长度 | 必须 < 16 字节 | `README.md` 约束章节 |
| Feature 名长度 | 必须 < 16 字节 | `README.md` 约束章节 |
| 系统能力名长度 | 必须 < 64 字节 | `samgr_lite.h:70` |
| 最大系统能力数 | 512 个 | `samgr_lite.h:65` |

## 下一章

- [消息通信 API](./05_Message_API.md) - 请求/响应机制
- [IPC 跨进程接口](./06_IPC_API.md) - 跨进程调用
