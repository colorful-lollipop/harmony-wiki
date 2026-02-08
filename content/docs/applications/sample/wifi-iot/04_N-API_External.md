# 对外 API 文档（C 层接口）

## 目的

本文档详细描述项目对外暴露的 C 层接口，包括 SAMGR_Lite 服务接口、GPIO 硬件操作接口和 Demo SDK 接口。

## 适用范围

- 需要调用项目功能的开发者
- 学习 SAMGR_Lite API 的开发者
- 集成 Demo SDK 的开发者

## 关键结论

1. **纯 C 接口**: 项目无 JS/N-API 绑定，所有接口为 C 函数
2. **SAMGR_Lite 核心**: 主要接口来自 SAMGR_Lite 框架
3. **GPIO 硬件接口**: 直接使用 HAL 层 API
4. **自定义 SDK**: demolink 提供可参考的 SDK 接口

> **重要**: 本项目是示例应用，不是生产库。对外接口主要用于学习参考。

## API 清单表

### 1. SAMGR_Lite 服务框架接口

| API 名称 | 命名空间 | 参数 | 返回值 | 同步/异步 | 实现位置 | 绑定位置 | 错误码 |
|---------|---------|------|--------|----------|---------|---------|--------|
| RegisterService | SAMGR_GetInstance() | Service* | BOOL | 同步 | SAMGR 框架 | app/samgr/service_example.c:92 | FALSE=失败 |
| RegisterFeature | SAMGR_GetInstance() | service, Feature* | BOOL | 同步 | SAMGR 框架 | app/samgr/feature_example.c:188 | FALSE=失败 |
| RegisterDefaultFeatureApi | SAMGR_GetInstance() | service, IUnknown* | BOOL | 同步 | SAMGR 框架 | app/samgr/service_example.c:93 | FALSE=失败 |
| RegisterFeatureApi | SAMGR_GetInstance() | service, feature, IUnknown* | BOOL | 同步 | SAMGR 框架 | app/samgr/feature_example.c:189 | FALSE=失败 |
| GetDefaultFeatureApi | SAMGR_GetInstance() | service | IUnknown* | 同步 | SAMGR 框架 | app/samgr/service_example.c:107 | NULL=失败 |
| GetFeatureApi | SAMGR_GetInstance() | service, feature | IUnknown* | 同步 | SAMGR 框架 | app/samgr/feature_example.c:203 | NULL=失败 |
| SendRequest | SAMGR | Identity*, Request*, Handler | BOOL | 异步 | SAMGR 框架 | app/samgr/feature_example.c:156 | FALSE=失败 |
| SendResponse | SAMGR | Request*, Response* | BOOL | 同步 | SAMGR 框架 | app/samgr/feature_example.c:113 | FALSE=失败 |

证据：
- `app/samgr/service_example.c:92-96` 服务注册示例
- `app/samgr/feature_example.c:188-192` 特性注册示例

### 2. SAMGR_Lite 示例服务接口

#### 2.1 服务示例接口 (DefaultFeatureApi)

| API 名称 | 参数 | 返回值 | 同步/异步 | 实现位置 | 说明 |
|---------|------|--------|----------|---------|------|
| SyncCall | IUnknown* | void | 同步 | app/samgr/service_example.c:73-78 | 同步调用示例 |

证据：
- `app/samgr/service_example.c:28-31` DefaultFeatureApi 定义

#### 2.2 特性示例接口 (DemoApi)

| API 名称 | 参数 | 返回值 | 同步/异步 | 实现位置 | 说明 |
|---------|------|--------|----------|---------|------|
| AsyncCall | IUnknown*, const char* | BOOL | 异步 | app/samgr/feature_example.c:142-157 | 异步调用示例 |
| AsyncTimeCall | IUnknown* | BOOL | 异步 | app/samgr/feature_example.c:159-166 | 定时异步调用 |
| SyncCall | IUnknown*, Payload* | BOOL | 同步 | app/samgr/feature_example.c:130-140 | 同步调用示例 |
| AsyncCallBack | IUnknown*, const char*, Handler | BOOL | 异步 | app/samgr/feature_example.c:168-184 | 异步回调示例 |

证据：
- `app/samgr/feature_example.c:42-48` DemoApi 定义

#### 2.3 广播接口 (PubSubInterface)

| API 名称 | 参数 | 返回值 | 同步/异步 | 实现位置 | 说明 |
|---------|------|--------|----------|---------|------|
| AddTopic | IUnknown*, Topic* | void | 同步 | SAMGR 广播服务 | 添加主题 |
| Subscribe | IUnknown*, Topic*, Consumer* | void | 同步 | SAMGR 广播服务 | 订阅主题 |
| Unsubscribe | IUnknown*, Topic*, Consumer* | Consumer* | 同步 | SAMGR 广播服务 | 取消订阅 |
| ModifyConsumer | IUnknown*, Topic*, Consumer*, Consumer* | void | 同步 | SAMGR 广播服务 | 修改消费者 |
| Publish | IUnknown*, Topic*, uint8_t*, uint32_t | void | 异步 | SAMGR 广播服务 | 发布消息 |

证据：
- `app/samgr/broadcast_example.c:124-158` 广播使用示例

### 3. IoT 硬件操作接口 (GPIO)

| API 名称 | 命名空间 | 参数 | 返回值 | 同步/异步 | 实现位置 | 绑定位置 | 错误码 |
|---------|---------|------|--------|----------|---------|---------|--------|
| IoTGpioInit | IoTGpio | uint16_t | int | 同步 | HAL 层 | app/iothardware/led_example.c:65 | 非0=失败 |
| IoTGpioSetDir | IoTGpio | uint16_t, IotGpioDir | int | 同步 | HAL 层 | app/iothardware/led_example.c:66 | 非0=失败 |
| IoTGpioSetOutputVal | IoTGpio | uint16_t, uint16_t | int | 同步 | HAL 层 | app/iothardware/led_example.c:40, 44, 48, 50 | 非0=失败 |

证据：
- `app/iothardware/led_example.c:65-66` GPIO 初始化
- `app/iothardware/led_example.c:40` GPIO 输出控制

### 4. Demo SDK 接口

| API 名称 | 参数 | 返回值 | 同步/异步 | 实现位置 | 说明 |
|---------|------|--------|----------|---------|------|
| DemoSdkEntry | void | int | 同步 | app/demolink/demosdk.c:33-48 | SDK 入口函数 |
| DemoSdkCreateTask | unsigned int*, TaskPara* | int | 同步 | app/demolink/demosdk_adapter.c | 创建任务（封装） |
| DemoSdkSleepMs | unsigned int | void | 同步 | app/demolink/demosdk_adapter.c | 睡眠（封装） |

证据：
- `app/demolink/demosdk.c:33` DemoSdkEntry 函数
- `app/demolink/demosdk_adapter.c` 适配层实现

## 接口详细说明

### SAMGR_Lite 服务注册接口

#### RegisterService

注册服务到 SAMGR 框架。

**函数签名**:
```c
BOOL RegisterService(Service *service);
```

**参数**:
- `service`: 服务对象指针，包含服务名称、初始化、消息处理等函数指针

**返回值**:
- `TRUE`: 注册成功
- `FALSE`: 注册失败

**示例**:
```c
static void Init(void) {
    SAMGR_GetInstance()->RegisterService((Service *)&g_example);
}
```

证据：
- `app/samgr/service_example.c:92` 使用示例

#### RegisterFeature

为已注册的服务注册特性。

**函数签名**:
```c
BOOL RegisterFeature(const char *serviceName, Feature *feature);
```

**参数**:
- `serviceName`: 服务名称
- `feature`: 特性对象指针

**返回值**:
- `TRUE`: 注册成功
- `FALSE`: 注册失败

**示例**:
```c
SAMGR_GetInstance()->RegisterFeature(EXAMPLE_SERVICE, (Feature *)&g_example);
```

证据：
- `app/samgr/feature_example.c:188` 使用示例

#### RegisterDefaultFeatureApi

注册服务的默认特性 API。

**函数签名**:
```c
BOOL RegisterDefaultFeatureApi(const char *serviceName, IUnknown *iUnknown);
```

**参数**:
- `serviceName`: 服务名称
- `iUnknown`: IUnknown 接口对象

**返回值**:
- `TRUE`: 注册成功
- `FALSE`: 注册失败

**示例**:
```c
SAMGR_GetInstance()->RegisterDefaultFeatureApi(EXAMPLE_SERVICE, GET_IUNKNOWN(g_example));
```

证据：
- `app/samgr/service_example.c:93` 使用示例

### SAMGR_Lite 服务发现接口

#### GetDefaultFeatureApi

获取服务的默认特性 API。

**函数签名**:
```c
IUnknown *GetDefaultFeatureApi(const char *serviceName);
```

**参数**:
- `serviceName`: 服务名称

**返回值**:
- 非 `NULL`: 成功返回 IUnknown 指针
- `NULL`: 获取失败

**示例**:
```c
IUnknown *iUnknown = SAMGR_GetInstance()->GetDefaultFeatureApi(EXAMPLE_SERVICE);
if (iUnknown == NULL) {
    printf("Error: GetDefaultFeatureApi failed!\n");
    return NULL;
}
```

证据：
- `app/samgr/service_example.c:107-112` 使用示例

#### GetFeatureApi

获取服务的指定特性 API。

**函数签名**:
```c
IUnknown *GetFeatureApi(const char *serviceName, const char *featureName);
```

**参数**:
- `serviceName`: 服务名称
- `featureName`: 特性名称

**返回值**:
- 非 `NULL`: 成功返回 IUnknown 指针
- `NULL`: 获取失败

**示例**:
```c
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(EXAMPLE_SERVICE, EXAMPLE_FEATURE);
if (iUnknown == NULL) {
    printf("Error: GetFeatureApi failed!\n");
    return NULL;
}
```

证据：
- `app/samgr/feature_example.c:203-208` 使用示例

### IUnknown 接口查询接口

#### QueryInterface

查询指定版本的接口。

**函数签名**:
```c
int QueryInterface(IUnknown *iUnknown, int version, void **api);
```

**参数**:
- `iUnknown`: IUnknown 对象
- `version`: 接口版本（通常为 DEFAULT_VERSION）
- `api`: 输出参数，返回 API 指针

**返回值**:
- `0`: 查询成功
- 非 `0`: 查询失败

**示例**:
```c
int result = iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&demoApi);
if (result != 0 || demoApi == NULL) {
    printf("Error: QueryInterface failed!\n");
    return NULL;
}
```

证据：
- `app/samgr/service_example.c:113-118` 使用示例

#### Release

释放接口引用（引用计数减1）。

**函数签名**:
```c
int32 Release(IUnknown *iUnknown);
```

**参数**:
- `iUnknown`: IUnknown 对象

**返回值**:
- `>0`: 剩余引用计数
- `<=0`: 对象已释放

**示例**:
```c
int32 ref = demoApi->Release((IUnknown *)demoApi);
if (ref <= 0) {
    printf("Warning: Object released, ref=%d\n", ref);
}
```

证据：
- `app/samgr/service_example.c:140-144` 使用示例

### GPIO 硬件接口

#### IoTGpioInit

初始化 GPIO 引脚。

**函数签名**:
```c
int IoTGpioInit(uint16_t id);
```

**参数**:
- `id`: GPIO 引脚号

**返回值**:
- `0`: 初始化成功
- 非 `0`: 初始化失败

**示例**:
```c
IoTGpioInit(LED_TEST_GPIO);
```

证据：
- `app/iothardware/led_example.c:65` 使用示例

#### IoTGpioSetDir

设置 GPIO 方向（输入/输出）。

**函数签名**:
```c
int IoTGpioSetDir(uint16_t id, IotGpioDir dir);
```

**参数**:
- `id`: GPIO 引脚号
- `dir`: 方向（IOT_GPIO_DIR_IN 或 IOT_GPIO_DIR_OUT）

**返回值**:
- `0`: 设置成功
- 非 `0`: 设置失败

**示例**:
```c
IoTGpioSetDir(LED_TEST_GPIO, IOT_GPIO_DIR_OUT);
```

证据：
- `app/iothardware/led_example.c:66` 使用示例

#### IoTGpioSetOutputVal

设置 GPIO 输出值。

**函数签名**:
```c
int IoTGpioSetOutputVal(uint16_t id, uint16_t val);
```

**参数**:
- `id`: GPIO 引脚号
- `val`: 输出值（0 或 1）

**返回值**:
- `0`: 设置成功
- 非 `0`: 设置失败

**示例**:
```c
IoTGpioSetOutputVal(LED_TEST_GPIO, 1);  // 高电平
IoTGpioSetOutputVal(LED_TEST_GPIO, 0);  // 低电平
```

证据：
- `app/iothardware/led_example.c:40, 44, 48, 50` 使用示例

### Demo SDK 接口

#### DemoSdkEntry

Demo SDK 入口函数。

**函数签名**:
```c
int DemoSdkEntry(void);
```

**参数**: 无

**返回值**:
- `0`: 成功
- 非 `0`: 失败

**示例**:
```c
static void DemoSdkMain(void) {
    DemoSdkEntry();
}
SYS_RUN(DemoSdkMain);
```

证据：
- `app/demolink/demosdk.c:33-48` 实现

## 调用链示例

### 同步调用流程

```
客户端代码
  └─> SAMGR_GetInstance()->GetDefaultFeatureApi(serviceName)
      └─> 返回 IUnknown 指针
  └─> iUnknown->QueryInterface(version, &api)
      └─> 返回具体 API 指针
  └─> api->SyncCall(payload)
      └─> 同步执行，返回结果
  └─> api->Release()
      └─> 释放引用
```

### 异步调用流程

```
客户端代码
  └─> api->AsyncCall(body)
      └─> SAMGR_SendRequest(&identity, &request, NULL)
          └─> 发送消息到服务队列
              └─> 服务任务: OnMessage(request)
                  └─> 处理逻辑
                  └─> SAMGR_SendResponse(response)
                      └─> 返回响应（如有）
```

证据：
- `app/samgr/feature_example.c:142-157` AsyncCall 实现
- `app/samgr/feature_example.c:105-128` OnMessage 实现

## 权限与前置条件

### SAMGR_Lite 接口
- **权限要求**: 无（轻量级系统不实现复杂权限机制）
- **前置条件**: SAMGR 框架已初始化
- **线程安全**: 大部分接口需要从初始化线程调用

### GPIO 接口
- **权限要求**: 需要硬件访问权限
- **前置条件**: GPIO 引脚未被占用
- **线程安全**: GPIO 操作需要外部同步（通常在任务内部）

### Demo SDK 接口
- **权限要求**: 无
- **前置条件**: LiteOS-M 任务调度器已初始化
- **线程安全**: 内部使用互斥机制保护

## 错误码与异常处理

### SAMGR_Lite 错误处理

| 错误场景 | 返回值 | 处理建议 |
|---------|--------|---------|
| 服务未注册 | NULL | 检查服务是否已注册 |
| 特性未注册 | NULL | 检查特性是否已注册 |
| 查询失败 | 非0 | 检查版本号是否正确 |
| 接口已释放 | <=0 | 避免重复释放 |

证据：
- `app/samgr/service_example.c:107-118` 错误处理示例
- `app/samgr/feature_example.c:204-214` 错误处理示例

### GPIO 错误处理

| 错误场景 | 返回值 | 处理建议 |
|---------|--------|---------|
| 引脚号无效 | 非0 | 检查引脚号范围 |
| 引脚被占用 | 非0 | 检查引脚使用情况 |
| 方向设置失败 | 非0 | 检查硬件状态 |

证据：
- 无显式错误处理示例（示例代码简化）

## 相关跳转链接

- [项目概览](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [内部 API](05_Inner_API.md)
- [附录 A: 关键调用链](appendix/Callgraphs.md)
