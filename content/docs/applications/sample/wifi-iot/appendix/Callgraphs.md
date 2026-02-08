# 附录 A: 关键调用链

## 目的

本文档提供关键功能的详细调用链，帮助开发者理解代码执行流程。

## 服务注册流程

### 调用链

```
Init() [service_example.c:90]
  └─> SAMGR_GetInstance()->RegisterService((Service *)&g_example)
      └─> SAMGR 内部（外部组件）
          ├─> service->GetName(&g_example)
          │   └─> return "example" [service_example.c:39-43]
          ├─> service->GetTaskConfig(&g_example)
          │   └─> return TaskConfig{...} [service_example.c:63-69]
          ├─> 创建任务和消息队列
          └─> service->Initialize(identity)
              └─> printf(...) [service_example.c:51]
  └─> SAMGR_GetInstance()->RegisterDefaultFeatureApi(EXAMPLE_SERVICE, GET_IUNKNOWN(g_example))
      └─> SAMGR 内部（外部组件）
```

**代码证据**:
- `app/samgr/service_example.c:90-96` Init 函数
- `app/samgr/service_example.c:39-43` GetName 实现
- `app/samgr/service_example.c:63-69` GetTaskConfig 实现
- `app/samgr/service_example.c:47-54` Initialize 实现

## 特性调用流程（同步）

### 调用链

```
CASE_GetIUnknown() [service_example.c:102]
  └─> SAMGR_GetInstance()->GetDefaultFeatureApi(EXAMPLE_SERVICE)
      └─> 返回 IUnknown 指针 [service_example.c:107]
  └─> iUnknown->QueryInterface(DEFAULT_VERSION, &demoApi)
      └─> IUnknown 内部（SAMGR 框架）
          └─> 返回 DefaultFeatureApi 指针 [service_example.c:113]

CASE_SyncCall(demoApi) [service_example.c:127]
  └─> demoApi->SyncCall((IUnknown *)demoApi)
      └─> SyncCall() [service_example.c:73]
          └─> printf(...) [service_example.c:76]
```

**代码证据**:
- `app/samgr/service_example.c:102-125` CASE_GetIUnknown 实现
- `app/samgr/service_example.c:127-134` CASE_SyncCall 实现
- `app/samgr/service_example.c:73-78` SyncCall 实现

## 特性调用流程（异步）

### 调用链

```
CASE_GetIUnknown() [feature_example.c:198]
  └─> SAMGR_GetInstance()->GetFeatureApi(EXAMPLE_SERVICE, EXAMPLE_FEATURE)
      └─> 返回 IUnknown 指针 [feature_example.c:203]
  └─> iUnknown->QueryInterface(DEFAULT_VERSION, &demoApi)
      └─> IUnknown 内部（SAMGR 框架）
          └─> 返回 DemoApi 指针 [feature_example.c:209]

CASE_AsyncCall(demoApi) [feature_example.c:247]
  └─> demoApi->AsyncCall((IUnknown *)demoApi, "I want to async call good result!")
      └─> AsyncCall() [feature_example.c:142]
          ├─> malloc(request.len) [feature_example.c:147]
          ├─> strcpy_s(request.data, request.len, body) [feature_example.c:150]
          ├─> SAMGR_SendRequest(&feature->identity, &request, NULL)
          │   └─> SAMGR 内部（发送消息到队列）
          └─> 消息队列处理
              └─> FEATURE_OnMessage(request) [feature_example.c:105]
                  ├─> printf(...) [feature_example.c:109]
                  └─> SAMGR_SendResponse(request, &response) [feature_example.c:113]
```

**代码证据**:
- `app/samgr/feature_example.c:198-221` CASE_GetIUnknown 实现
- `app/samgr/feature_example.c:247-264` CASE_AsyncCall 实现
- `app/samgr/feature_example.c:142-157` AsyncCall 实现
- `app/samgr/feature_example.c:105-128` FEATURE_OnMessage 实现

## 广播发布订阅流程

### 调用链

```
Init() [broadcast_example.c:86]
  └─> SAMGR_GetInstance()->RegisterService(&g_testService)
      └─> SAMGR 内部（外部组件）

CASE_GetIUnknown() [broadcast_example.c:95]
  └─> SAMGR_GetInstance()->GetFeatureApi(BROADCAST_SERVICE, PUB_SUB_FEATURE)
      └─> 返回 PubSubInterface 指针 [broadcast_example.c:100]

CASE_AddAndUnsubscribeTopic(fapi) [broadcast_example.c:124]
  ├─> subscriber->AddTopic((IUnknown *)fapi, &topic0)
  │   └─> 广播服务内部（添加主题）
  ├─> subscriber->Subscribe((IUnknown *)fapi, &topic0, &c1)
  │   └─> 广播服务内部（注册消费者 c1）
  ├─> subscriber->Subscribe((IUnknown *)fapi, &topic0, &c2)
  │   └─> 广播服务内部（注册消费者 c2）
  └─> provider->Publish((IUnknown *)fapi, &topic0, (uint8_t *) "==>111<==", TEST_LEN)
      └─> 广播服务内部（发布消息）
          ├─> C1Callback(consumer, topic, request) [broadcast_example.c:31]
          │   └─> printf(...) [broadcast_example.c:35]
          └─> C2Callback(consumer, topic, request) [broadcast_example.c:39]
              └─> printf(...) [broadcast_example.c:43]
```

**代码证据**:
- `app/samgr/broadcast_example.c:86-91` Init 实现
- `app/samgr/broadcast_example.c:95-119` CASE_GetIUnknown 实现
- `app/samgr/broadcast_example.c:124-158` CASE_AddAndUnsubscribeTopic 实现
- `app/samgr/broadcast_example.c:31-45` C1Callback 和 C2Callback 实现

## GPIO 操作流程

### 调用链

```
SYS_RUN(LedExampleEntry) [led_example.c:81]
  └─> LedExampleEntry() [led_example.c:61]
      ├─> IoTGpioInit(LED_TEST_GPIO) [led_example.c:65]
      │   └─> HAL 层（初始化 GPIO）
      ├─> IoTGpioSetDir(LED_TEST_GPIO, IOT_GPIO_DIR_OUT) [led_example.c:66]
      │   └─> HAL 层（设置方向为输出）
      └─> osThreadNew(LedTask, NULL, &attr) [led_example.c:76]
          └─> LiteOS-M 内核（创建任务）
              └─> LedTask() [led_example.c:35]
                  └─> switch (g_ledState) [led_example.c:38]
                      ├─> LED_ON:
                      │   └─> IoTGpioSetOutputVal(LED_TEST_GPIO, 1) [led_example.c:40]
                      │       └─> HAL 层（设置 GPIO 高电平）
                      ├─> LED_OFF:
                      │   └─> IoTGpioSetOutputVal(LED_TEST_GPIO, 0) [led_example.c:44]
                      │       └─> HAL 层（设置 GPIO 低电平）
                      └─> LED_SPARK:
                          ├─> IoTGpioSetOutputVal(LED_TEST_GPIO, 0) [led_example.c:48]
                          │   └─> HAL 层（设置 GPIO 低电平）
                          └─> IoTGpioSetOutputVal(LED_TEST_GPIO, 1) [led_example.c:50]
                              └─> HAL 层（设置 GPIO 高电平）
```

**代码证据**:
- `app/iothardware/led_example.c:61-79` LedExampleEntry 实现
- `app/iothardware/led_example.c:35-59` LedTask 实现

## Demo SDK 启动流程

### 调用链

```
SYS_RUN(DemoSdkMain) [helloworld.c:24]
  └─> DemoSdkMain() [helloworld.c:19]
      └─> DemoSdkEntry() [demosdk.c:33]
          ├─> printf("it is demosdk entry.\n") [demosdk.c:35]
          ├─> DemoSdkCreateTask(&handle, &para) [demosdk.c:42]
          │   └─> 适配层内部
          │       └─> osThreadNew(func, arg, attr) [demosdk_adapter.c]
          │           └─> LiteOS-M 内核（创建任务）
          │               └─> DemoSdkBiz(arg) [demosdk.c:25]
          │                   ├─> printf("it is demo biz: hello world.\n") [demosdk.c:28]
          │                   └─> DemoSdkSleepMs(SECOND_CNT) [demosdk.c:29]
          │                       └─> LOS_Msleep(ms) [demosdk_adapter.c]
          │                           └─> LiteOS-M 内核（睡眠）
          └─> 返回 0 [demosdk.c:47]
```

**代码证据**:
- `app/demolink/helloworld.c:19-24` DemoSdkMain 实现
- `app/demolink/demosdk.c:33-48` DemoSdkEntry 实现
- `app/demolink/demosdk.c:25-31` DemoSdkBiz 实现

## 测试用例执行流程

### 调用链

```
LAYER_INITCALL_DEF(RunTestCase, test, "test") [service_example.c:187]
  └─> 系统初始化后执行
      └─> RunTestCase() [service_example.c:179]
          ├─> CASE_GetIUnknown() [service_example.c:102]
          ├─> CASE_RegisterInvalidService() [service_example.c:153]
          ├─> CASE_SyncCall(defaultApi) [service_example.c:127]
          └─> CASE_ReleaseIUnknown(defaultApi) [service_example.c:136]
```

**代码证据**:
- `app/samgr/service_example.c:179-185` RunTestCase 实现
- `app/samgr/service_example.c:187` LAYER_INITCALL_DEF 注册
- `app/samgr/feature_example.c:310-318` feature_example.c 中的 RunTestCase 实现

## 相关跳转链接

- [架构说明](03_Architecture.md)
- [对外 API 文档](04_N-API_External.md)
- [常见问题与调试](09_QA_Troubleshooting.md)
