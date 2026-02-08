# 安全风险评审

## 目的

本文档基于代码证据对项目进行安全风险评审，识别攻击面、信任边界和潜在安全风险。

## 适用范围

- 需要进行安全审计的开发者
- 进行安全评估的安全工程师
- 了解项目安全机制的维护者

## 关键结论

1. **低风险**: 这是示例代码，攻击面有限
2. **无复杂权限机制**: 轻量级系统，无复杂权限控制
3. **基本安全实践**: 使用 securec 安全函数，有基本参数校验
4. **建议改进**: 添加更严格的输入验证和错误处理

## 检查范围与局限性

### 检查范围

| 类别 | 检查内容 | 代码位置 |
|------|---------|---------|
| 输入验证 | 参数校验、长度检查、边界检查 | 所有源文件 |
| 内存安全 | 指针使用、内存分配/释放 | 所有源文件 |
| 线程安全 | 竞态条件、锁机制 | 所有源文件 |
| 信息泄露 | 日志输出、敏感数据 | printf 调用 |
| 整数溢出 | 数组索引、大小计算 | 循环、数组访问 |
| 格式化字符串 | printf 参数 | printf 调用 |
| 资源管理 | 文件句柄、GPIO 资源 | led_example.c |
| 错误处理 | 返回值检查、异常处理 | 所有源文件 |

### 局限性

1. **外部依赖未检查**: utils_lite、liteos_m、peripheral 等外部组件的安全问题不在本文档范围内
2. **硬件层未检查**: GPIO HAL 层的安全问题不在本文档范围内
3. **示例代码性质**: 这是学习和参考代码，不用于生产环境，安全风险相对较低
4. **不涉及网络**: 项目无网络通信，无网络攻击面

## 攻击面分析

### 攻击面清单

| 攻击面 | 描述 | 证据位置 |
|-------|------|---------|
| GPIO 硬件控制 | 可通过代码控制 GPIO，影响硬件状态 | app/iothardware/led_example.c:40, 44, 48, 50 |
| SAMGR 服务注册 | 可注册自定义服务和特性 | app/samgr/*.c |
| 广播消息 | 可通过广播机制传递消息 | app/samgr/broadcast_example.c |
| printf 日志 | 可能泄露敏感信息 | 所有源文件 |

### 攻击面评级

| 攻击面 | 风险等级 | 说明 |
|-------|---------|------|
| GPIO 硬件控制 | 低 | 需要代码执行能力，示例代码仅控制 LED |
| SAMGR 服务注册 | 低 | 需要代码执行能力，轻量级系统无复杂权限 |
| 广播消息 | 低 | 仅限系统内部，无外部接口 |
| printf 日志 | 低 | 示例代码日志，无敏感信息 |

## 信任边界

### 信任边界图

```
┌──────────────────────────────────────────────────────────────┐
│                   不可信环境                               │
│         （攻击者可能控制输入或修改代码）                      │
└──────────────────────────────────────────────────────────────┘
                            ▲
                            │ 不受信任的输入
┌──────────────────────────────────────────────────────────────┐
│                   信任边界（边界检查）                       │
│              参数校验、长度验证、错误处理                      │
└──────────────────────────────────────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                   可信环境                                 │
│         （内部数据、已验证的输入、系统资源）                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│
│  │ demolink │  │iothardware│   │  samgr   │  │ startup  ││
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘│
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│
│  │utils_lite│  │liteos_m  │  │peripheral│  │samgr_lite││
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘│
└──────────────────────────────────────────────────────────────┘
```

### 数据流信任边界

```
外部输入（如配置、参数）
    ↓
[边界检查]
    - 参数非空检查
    - 长度验证
    - 类型检查
    ↓
内部处理
    ↓
系统资源（GPIO、SAMGR 服务）
```

## 安全机制评估

### 1. 输入验证

#### 1.1 参数非空检查

**发现位置**: `app/samgr/feature_example.c:133-139`

```c
static BOOL SyncCall(IUnknown *iUnknown, struct Payload *payload)
{
    (void)iUnknown;
    if (payload != NULL && payload->id >= 0 && payload->name != NULL) {
        printf("[LPC Test][TaskID:%u][Step:%u][SyncCall API] Id:%d, name:%s, value:%d \n",
               (int)osThreadGetId(), g_asyncStep++, payload->id, payload->name, payload->value);
        return TRUE;
    }
    printf("[LPC Test][TaskID:%u][Step:%u][SyncCall API] Input Error! \n", (int)osThreadGetId(), g_asyncStep++);
    return FALSE;
}
```

**评估**: 有基本的非空检查，但不够全面。

**风险**: 其他函数缺乏参数非空检查。

#### 1.2 长度验证

**发现位置**: `app/samgr/feature_example.c:145-154`

```c
static BOOL AsyncCall(IUnknown *iUnknown, const char *body)
{
    Request request = {.msgId = MSG_PROC, .msgValue = 0};
    request.len = (uint32_t)(strlen(body) + 1);
    request.data = malloc(request.len);
    if (request.data == NULL) {
        return FALSE;
    }
    if (strcpy_s(request.data, request.len, body) != EOK) {
        free(request.data);
        return FALSE;
    }
    // ...
}
```

**评估**: 有长度检查，使用 `strlen` 计算长度，然后使用 `strcpy_s` 安全复制。

**风险**: `strlen(body)` 假设 `body` 是以 `\0` 结尾的字符串，如果 `body` 不是合法字符串，可能导致越界读取。

**建议**: 添加最大长度限制：
```c
#define MAX_BODY_LEN 1024
if (strlen(body) >= MAX_BODY_LEN) {
    return FALSE;
}
```

#### 1.3 内存分配失败检查

**发现位置**: `app/samgr/feature_example.c:147-152`

```c
request.data = malloc(request.len);
if (request.data == NULL) {
    return FALSE;
}
```

**评估**: 有内存分配失败检查，正确处理 `malloc` 返回 `NULL` 的情况。

**风险**: 无。

### 2. 内存安全

#### 2.1 使用安全函数

**发现位置**: `app/samgr/feature_example.c:150-153`

```c
if (strcpy_s(request.data, request.len, body) != EOK) {
    free(request.data);
    return FALSE;
}
```

**评估**: 使用 `strcpy_s` 安全函数，正确处理长度和错误返回值。

**风险**: 无。

#### 2.2 内存释放

**发现位置**: `app/samgr/feature_example.c:173-179`

```c
static BOOL AsyncCallBack(IUnknown *iUnknown, const char *body, Handler handler)
{
    // ...
    request.data = malloc(request.len);
    if (request.data == NULL) {
        return FALSE;
    }
    if (strcpy_s(request.data, request.len, body) != EOK) {
        free(request.data);  // 正确释放内存
        return FALSE;
    }
    // ...
}
```

**评估**: 内存分配失败时正确释放已分配的资源。

**风险**: 无。

#### 2.3 潜在内存泄漏

**发现位置**: `app/samgr/feature_example.c:142-157`

```c
static BOOL AsyncCall(IUnknown *iUnknown, const char *body)
{
    Request request = {.msgId = MSG_PROC, .msgValue = 0};
    request.len = (uint32_t)(strlen(body) + 1);
    request.data = malloc(request.len);
    if (request.data == NULL) {
        return FALSE;
    }
    if (strcpy_s(request.data, request.len, body) != EOK) {
        free(request.data);
        return FALSE;
    }
    DemoFeature *feature = GET_OBJECT(iUnknown, DemoFeature, iUnknown);
    printf("[LPC Test][TaskID:%u][Step:%u][AsyncCall API] Send request! \n", (int)osThreadGetId(), g_asyncStep++);
    return SAMGR_SendRequest(&feature->identity, &request, NULL);
    // 问题：request.data 谁来释放？
}
```

**评估**: `request.data` 的内存由谁释放不明确。`SAMGR_SendRequest` 可能会复制数据，也可能直接使用指针。

**风险**: 如果 `SAMGR_SendRequest` 不释放 `request.data`，则会导致内存泄漏。

**建议**: 检查 `SAMGR_SendRequest` 的实现，确认 `request.data` 的生命周期管理。如果 SAMGR 不释放，需要在发送后手动释放。

### 3. 线程安全

#### 3.1 全局变量访问

**发现位置**: `app/iothardware/led_example.c:33`

```c
enum LedState g_ledState = LED_SPARK;
```

**评估**: 全局变量 `g_ledState` 被 `LedTask` 任务访问，但没有看到明确的同步机制。

**风险**: 如果多个任务访问 `g_ledState`，可能导致竞态条件。

**建议**: 添加互斥锁或使用原子操作：
```c
#include <pthread.h>

static pthread_mutex_t g_ledStateMutex = PTHREAD_MUTEX_INITIALIZER;

// 使用时
pthread_mutex_lock(&g_ledStateMutex);
enum LedState state = g_ledState;
pthread_mutex_unlock(&g_ledStateMutex);
```

#### 3.2 多任务并发

**发现位置**: `app/samgr/feature_example.c:71`

```c
static volatile uint32 g_asyncStep = 0;
```

**评估**: 使用 `volatile` 修饰全局变量，防止编译器优化，但 `volatile` 不能保证原子性。

**风险**: 如果多个任务并发修改 `g_asyncStep`，可能导致数据竞争。

**建议**: 对于简单的计数器，如果只是单生产者多消费者，`volatile` 可能足够。如果需要严格的线程安全，使用原子操作或互斥锁。

### 4. 信息泄露

#### 4.1 printf 日志输出

**发现位置**: 大量使用 `printf` 输出日志

```c
printf("[LPC Test][TaskID:%u][Step:%u][SyncCall API] Id:%d, name:%s, value:%d \n",
       (int)osThreadGetId(), g_asyncStep++, payload->id, payload->name, payload->value);
```

**评估**: `printf` 输出调试信息，可能泄露敏感数据（如内存地址、任务ID等）。

**风险**: 在生产环境中，过多的日志输出可能泄露系统内部信息。

**建议**:
1. 使用日志级别控制（DEBUG、INFO、WARN、ERROR）
2. 避免输出敏感信息（如内存地址、指针值）
3. 在生产环境中禁用或限制 DEBUG 日志

### 5. 整数溢出

#### 5.1 数组索引

**发现位置**: 未发现明显的数组索引操作，所有数组访问都是静态索引。

**评估**: 无风险。

#### 5.2 大小计算

**发现位置**: `app/samgr/feature_example.c:145`

```c
request.len = (uint32_t)(strlen(body) + 1);
```

**评估**: `strlen(body) + 1` 可能导致整数溢出，如果 `body` 是超长字符串（超过 `UINT32_MAX - 1`）。

**风险**: 理论上可能，但实际上字符串长度不会超过可用内存。

**建议**: 添加最大长度检查：
```c
size_t len = strlen(body);
if (len > MAX_BODY_LEN - 1) {
    return FALSE;
}
request.len = (uint32_t)(len + 1);
```

### 6. 格式化字符串漏洞

#### 6.1 printf 参数

**发现位置**: 所有 `printf` 调用

```c
printf("[LPC Test][TaskID:%u][Step:%u][OnMessage: S:%s, F:%s] msgId<MSG_PROC> %s \n",
       (int)osThreadGetId(), g_asyncStep++, EXAMPLE_SERVICE, feature->GetName(feature),
       (char *)request->data);
```

**评估**: 所有 `printf` 调用都使用格式化字符串，没有直接使用用户输入作为格式化字符串。

**风险**: 无。

## 可被利用点

### 1. 输入验证不足

**证据**: `app/samgr/feature_example.c:145`

```c
request.len = (uint32_t)(strlen(body) + 1);
```

**触发**: 传入超长字符串 `body`。

**影响**:
- 整数溢出（理论上）
- 内存分配失败
- 潜在的缓冲区溢出（如果 `strcpy_s` 实现有漏洞）

**修复建议**:
```c
#define MAX_BODY_LEN 1024
size_t len = strlen(body);
if (len > MAX_BODY_LEN - 1) {
    return FALSE;
}
request.len = (uint32_t)(len + 1);
request.data = malloc(request.len);
```

### 2. 内存泄漏

**证据**: `app/samgr/feature_example.c:142-157`

**触发**: 多次调用 `AsyncCall`。

**影响**:
- 内存逐渐耗尽
- 系统不稳定

**修复建议**: 确认 `SAMGR_SendRequest` 是否释放 `request.data`，如果不释放，在发送后手动释放。

### 3. 全局变量竞态

**证据**: `app/iothardware/led_example.c:33`

```c
enum LedState g_ledState = LED_SPARK;
```

**触发**: 多个任务并发访问 `g_ledState`。

**影响**:
- LED 控制逻辑混乱
- 未定义行为

**修复建议**: 添加互斥锁保护。

### 4. 日志信息泄露

**证据**: 所有 `printf` 调用

**触发**: 系统运行时输出日志。

**影响**:
- 泄露系统内部信息（任务ID、内存地址等）
- 可用于信息收集攻击

**修复建议**:
1. 使用日志级别控制
2. 避免输出敏感信息
3. 在生产环境中禁用 DEBUG 日志

### 5. 缺乏错误处理

**证据**: `app/demolink/demosdk.c:42-46`

```c
int ret = DemoSdkCreateTask(&handle, &para);
if (ret != 0) {
    printf("create task fail.\n");
    return -1;
}
```

**触发**: 任务创建失败。

**影响**:
- 错误处理不完善
- 可能导致资源泄漏

**修复建议**: 添加更详细的错误处理和资源清理。

## 检查结论

### 总体评估

| 类别 | 评级 | 说明 |
|------|------|------|
| 输入验证 | 中等 | 有基本验证，但不够全面 |
| 内存安全 | 良好 | 使用安全函数，有失败检查 |
| 线程安全 | 中等 | 部分全局变量无保护 |
| 信息泄露 | 低 | 仅日志输出，无敏感信息 |
| 整数溢出 | 低 | 理论上可能，实际风险低 |
| 格式化字符串 | 良好 | 正确使用格式化字符串 |

### 风险汇总

| 风险 | 严重性 | 可能性 | 优先级 |
|------|--------|--------|--------|
| 输入验证不足 | 中 | 中 | 中 |
| 内存泄漏 | 低 | 中 | 低 |
| 全局变量竞态 | 中 | 低 | 低 |
| 日志信息泄露 | 低 | 高 | 中 |
| 缺乏错误处理 | 低 | 低 | 低 |

### 建议优先级

| 优先级 | 建议内容 |
|-------|---------|
| 高 | 添加输入长度限制 |
| 中 | 检查内存泄漏（request.data 生命周期） |
| 中 | 添加日志级别控制 |
| 低 | 添加全局变量互斥锁 |
| 低 | 完善错误处理 |

## 检查范围声明

本文档检查了项目中的所有 C 源代码（约 1765 行），包括：
- `app/demolink/` - 3 个源文件
- `app/iothardware/` - 1 个源文件
- `app/samgr/` - 9 个源文件

**未检查**:
- 外部依赖组件（utils_lite、liteos_m、peripheral、samgr_lite）
- HAL 层实现
- 编译器/工具链安全特性
- 硬件层安全（GPIO 驱动）

## 相关跳转链接

- [项目概览](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [对外 API 文档](04_N-API_External.md)
- [常见问题与调试](09_QA_Troubleshooting.md)
