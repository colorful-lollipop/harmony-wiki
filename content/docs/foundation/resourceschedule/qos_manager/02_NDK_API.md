# NDK API 接口

> 本文档描述 qos_manager 面向 Native 应用的 NDK C 接口。

## 1. 概述

### 1.1 接口特性

| 特性 | 描述 |
|------|------|
| **语言** | C (extern "C") |
| **库** | libqos.so |
| **头文件** | `<qos/qos.h>` |
| **系统能力** | SystemCapability.Resourceschedule.QoS.Core |
| **起始版本** | 12 (基础 API), 20 (GEWU API) |

### 1.2 API 分类

| 分类 | API | 版本 | 描述 |
|------|-----|------|------|
| **QoS 基础** | `OH_QoS_SetThreadQoS` | 12 | 设置线程 QoS |
| | `OH_QoS_ResetThreadQoS` | 12 | 重置线程 QoS |
| | `OH_QoS_GetThreadQoS` | 12 | 获取线程 QoS |
| **GEWU AI 推理** | `OH_QoS_GewuCreateSession` | 20 | 创建推理会话 |
| | `OH_QoS_GewuDestroySession` | 20 | 销毁会话 |
| | `OH_QoS_GewuSubmitRequest` | 20 | 提交推理请求 |
| | `OH_QoS_GewuAbortRequest` | 20 | 中止推理请求 |

---

## 2. QoS 基础 API

### 2.1 OH_QoS_SetThreadQoS

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `int OH_QoS_SetThreadQoS(QoS_Level level)` |
| **头文件** | `interfaces/kits/c/qos.h:90` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:37` |
| **同步/异步** | 同步 |
| **返回值** | 0 成功，-1 失败 |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| `level` | `QoS_Level` | IN | QoS 等级枚举值 |

**QoS_Level 枚举值**:

| 枚举 | 值 | 描述 |
|------|-----|------|
| `QOS_BACKGROUND` | 0 | 后台任务 |
| `QOS_UTILITY` | 1 | 实用工具 |
| `QOS_DEFAULT` | 2 | 默认级别 |
| `QOS_USER_INITIATED` | 3 | 用户主动发起 |
| `QOS_DEADLINE_REQUEST` | 4 | 截止时间请求 |
| `QOS_USER_INTERACTIVE` | 5 | 用户交互 |

#### 参数校验

```cpp
// 证据: frameworks/native/qos_ndk.cpp:39-41
if (level < QOS_BACKGROUND || level > QOS_USER_INTERACTIVE) {
    return ERROR_NUM;  // 返回 -1
}
```

| 校验规则 | 失败处理 |
|----------|----------|
| `level >= QOS_BACKGROUND` | 返回 -1 |
| `level <= QOS_USER_INTERACTIVE` | 返回 -1 |

#### 返回值

| 返回值 | 含义 |
|--------|------|
| 0 | 成功 |
| -1 | 参数错误或设置失败 |

#### 调用链

```
Native App
    ↓ OH_QoS_SetThreadQoS()
qos_ndk.cpp:OH_QoS_SetThreadQoS()
    ↓
qos.cpp:SetThreadQos()
    ↓
qos_interface.cpp:QosApplyForOther()
    ↓
ioctl(sched_qos_ctrl)
```

---

### 2.2 OH_QoS_ResetThreadQoS

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `int OH_QoS_ResetThreadQoS()` |
| **头文件** | `interfaces/kits/c/qos.h:99` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:45` |
| **同步/异步** | 同步 |
| **返回值** | 0 成功，-1 失败 |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| 无 | - | - | 重置当前线程 QoS |

#### 调用链

```
Native App
    ↓ OH_QoS_ResetThreadQoS()
qos_ndk.cpp:OH_QoS_ResetThreadQoS()
    ↓
qos.cpp:ResetThreadQos()
    ↓
qos_interface.cpp:QosLeaveForOther()
    ↓
ioctl(sched_qos_ctrl)
```

---

### 2.3 OH_QoS_GetThreadQoS

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `int OH_QoS_GetThreadQoS(QoS_Level *level)` |
| **头文件** | `interfaces/kits/c/qos.h:110` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:50` |
| **同步/异步** | 同步 |
| **返回值** | 0 成功，-1 失败 |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| `level` | `QoS_Level *` | OUT | 输出当前线程 QoS 等级 |

#### 参数校验

```cpp
// 证据: frameworks/native/qos_ndk.cpp:52-54
if (level == nullptr) {
    return ERROR_NUM;  // 返回 -1
}
```

| 校验规则 | 失败处理 |
|----------|----------|
| `level != nullptr` | 返回 -1 |
| `*level` 范围 0-5 | 返回 -1 |

#### 返回值

| 返回值 | 含义 |
|--------|------|
| 0 | 成功 |
| -1 | 参数错误或获取失败 |

---

## 3. GEWU AI 推理 API

> GEWU (Generative Engine with Weight Update) AI 推理模块接口。

### 3.1 数据类型定义

#### 3.1.1 Session ID

```c
// 证据: interfaces/kits/c/qos.h:117-118
typedef unsigned int OH_QoS_GewuSession;
#define OH_QOS_GEWU_INVALID_SESSION_ID (static_cast<OH_QoS_GewuSession>(0xffffffffU))
```

#### 3.1.2 Request ID

```c
// 证据: interfaces/kits/c/qos.h:124-125
typedef unsigned int OH_QoS_GewuRequest;
#define OH_QOS_GEWU_INVALID_REQUEST_ID (static_cast<OH_QoS_GewuRequest>(0xffffffffU))
```

#### 3.1.3 错误码

```c
// 证据: interfaces/kits/c/qos.h:133-142
typedef enum {
    OH_QOS_GEWU_OK     = 0,   // 成功
    OH_QOS_GEWU_NOPERM = 201,  // 无权限
    OH_QOS_GEWU_NOMEM  = 203,  // 内存不足
    OH_QOS_GEWU_INVAL  = 401,  // 参数无效
    OH_QOS_GEWU_EXIST  = 501,  // 已存在
    OH_QOS_GEWU_NOENT  = 502,  // 不存在
    OH_QOS_GEWU_NOSYS  = 801,  // 系统错误
    OH_QOS_GEWU_FAULT  = 901,  // 故障
} OH_QoS_GewuErrorCode;
```

#### 3.1.4 创建会话结果

```c
// 证据: interfaces/kits/c/qos.h:153-156
typedef struct {
    OH_QoS_GewuSession session;
    OH_QoS_GewuErrorCode error;
} OH_QoS_GewuCreateSessionResult;
```

#### 3.1.5 提交请求结果

```c
// 证据: interfaces/kits/c/qos.h:168-170
typedef struct {
    OH_QoS_GewuRequest request;
    OH_QoS_GewuErrorCode error;
} OH_QoS_GewuSubmitRequestResult;
```

#### 3.1.6 响应回调

```c
// 证据: interfaces/kits/c/qos.h:189
typedef void (*OH_QoS_GewuOnResponse)(void* context, const char* response);
```

---

### 3.2 OH_QoS_GewuCreateSession

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `OH_QoS_GewuCreateSessionResult OH_QoS_GewuCreateSession(const char* attributes)` |
| **头文件** | `interfaces/kits/c/qos.h:214` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:140` |
| **同步/异步** | 同步 |
| **返回值** | `OH_QoS_GewuCreateSessionResult` |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| `attributes` | `const char *` | IN | JSON 格式的会话属性 |

**attributes JSON 格式**:

```json
{
    "model": "/data/storage/el2/base/files/qwen2/"
}
```

#### 动态库加载

```cpp
// 证据: frameworks/native/qos_ndk.cpp:27-32
const char* GEWU_CLIENT_LIB = "libgewu_client.z.so";

const char* GEWU_CREATE_SESSION_FUNC = "GewuCreateSession";
const char* GEWU_DESTROY_SESSION_FUNC = "GewuDestroySession";
const char* GEWU_SUBMIT_REQUEST_FUNC = "GewuSubmitRequest";
const char* GEWU_ABORT_REQUEST_FUNC = "GewuAbortRequest";
```

#### 初始化流程

```cpp
// 证据: frameworks/native/qos_ndk.cpp:118-138
static void InitializeGewu(void)
{
    // 1. dlopen 加载 libgewu_client.z.so
    g_gewuNdkLibHandler = dlopen(GEWU_CLIENT_LIB, RTLD_LAZY | RTLD_LOCAL);
    // 2. dlsym 获取函数指针
    g_CreateSession = reinterpret_cast<GewuCreateSessionFunc>(LoadSymbol(GEWU_CREATE_SESSION_FUNC));
    // 3. 设置初始化标志
    g_gewuInitialized = std::once_flag;
}
```

#### 返回值

| 字段 | 值 | 含义 |
|------|-----|------|
| `session` | 有效 session ID | 会话创建成功 |
| `session` | `OH_QOS_GEWU_INVALID_SESSION_ID` | 会话创建失败 |
| `error` | `OH_QOS_GEWU_OK` | 成功 |
| `error` | `OH_QOS_GEWU_NOMEM` | 内存不足 |
| `error` | `OH_QOS_GEWU_NOSYS` | 系统错误 (库未加载) |

---

### 3.3 OH_QoS_GewuDestroySession

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `OH_QoS_GewuErrorCode OH_QoS_GewuDestroySession(OH_QoS_GewuSession session)` |
| **头文件** | `interfaces/kits/c/qos.h:233` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:148` |
| **同步/异步** | 同步 |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| `session` | `OH_QoS_GewuSession` | IN | 要销毁的会话 ID |

#### 返回值

| 返回值 | 含义 |
|--------|------|
| `OH_QOS_GEWU_OK` | 成功 |
| `OH_QOS_GEWU_NOENT` | 会话不存在 |

---

### 3.4 OH_QoS_GewuSubmitRequest

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `OH_QoS_GewuSubmitRequestResult OH_QoS_GewuSubmitRequest(OH_QoS_GewuSession session, const char* request, OH_QoS_GewuOnResponse callback, void* context)` |
| **头文件** | `interfaces/kits/c/qos.h:294` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:156` |
| **同步/异步** | 异步 (回调模式) |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| `session` | `OH_QoS_GewuSession` | IN | 会话 ID |
| `request` | `const char *` | IN | JSON 格式的请求 |
| `callback` | `OH_QoS_GewuOnResponse` | IN | 响应回调函数 |
| `context` | `void *` | IN | 用户上下文 |

**request JSON 格式**:

```json
{
    "messages": [
        {
            "role": "user",
            "content": "What is OpenHarmony"
        }
    ],
    "stream": true
}
```

#### 回调函数签名

```c
typedef void (*OH_QoS_GewuOnResponse)(
    void* context,           // 用户传入的 context
    const char* response     // JSON 格式的响应
);
```

#### 返回值

| 字段 | 值 | 含义 |
|------|-----|------|
| `request` | 有效 request ID | 请求提交成功 |
| `request` | `OH_QOS_GEWU_INVALID_REQUEST_ID` | 请求提交失败 |
| `error` | `OH_QOS_GEWU_OK` | 成功 |
| `error` | `OH_QOS_GEWU_NOMEM` | 内存不足 |

---

### 3.5 OH_QoS_GewuAbortRequest

#### 基本信息

| 属性 | 值 |
|------|-----|
| **函数原型** | `OH_QoS_GewuErrorCode OH_QoS_GewuAbortRequest(OH_QoS_GewuSession session, OH_QoS_GewuRequest request)` |
| **头文件** | `interfaces/kits/c/qos.h:249` |
| **实现文件** | `frameworks/native/qos_ndk.cpp:165` |
| **同步/异步** | 同步 |

#### 参数说明

| 参数 | 类型 | 方向 | 描述 |
|------|------|------|------|
| `session` | `OH_QoS_GewuSession` | IN | 会话 ID |
| `request` | `OH_QoS_GewuRequest` | IN | 要中止的请求 ID |

#### 返回值

| 返回值 | 含义 |
|--------|------|
| `OH_QOS_GEWU_OK` | 成功 |
| `OH_QOS_GEWU_NOENT` | 请求不存在 |

---

## 4. 错误码汇总

| 错误码 | 宏定义 | 含义 | 适用 API |
|--------|--------|------|----------|
| 0 | `OH_QOS_GEWU_OK` | 成功 | GEWU 系列 |
| -1 | `ERROR_NUM` | 通用错误 | QoS 系列 |
| 201 | `OH_QOS_GEWU_NOPERM` | 无权限 | GEWU 系列 |
| 203 | `OH_QOS_GEWU_NOMEM` | 内存不足 | GEWU 系列 |
| 401 | `OH_QOS_GEWU_INVAL` | 参数无效 | GEWU 系列 |
| 501 | `OH_QOS_GEWU_EXIST` | 已存在 | GEWU 系列 |
| 502 | `OH_QOS_GEWU_NOENT` | 不存在 | GEWU 系列 |
| 801 | `OH_QOS_GEWU_NOSYS` | 系统错误 | GEWU 系列 |
| 901 | `OH_QOS_GEWU_FAULT` | 故障 | GEWU 系列 |

**证据**: `interfaces/kits/c/qos.h:133-142`, `frameworks/native/qos_ndk.cpp:25`

---

## 5. 使用示例

### 5.1 QoS 设置示例

```c
#include <qos/qos.h>

int main() {
    // 设置线程为用户交互级别
    int ret = OH_QoS_SetThreadQoS(QOS_USER_INTERACTIVE);
    if (ret != 0) {
        // 处理错误
        return -1;
    }

    // 获取当前 QoS 等级
    QoS_Level level;
    ret = OH_QoS_GetThreadQoS(&level);
    if (ret != 0) {
        return -1;
    }

    // 业务逻辑完成后重置
    OH_QoS_ResetThreadQoS();

    return 0;
}
```

### 5.2 GEWU AI 推理示例

```c
#include <qos/qos.h>

void response_callback(void* context, const char* response) {
    printf("Response: %s\n", response);
    // 处理响应
}

int main() {
    // 1. 创建会话
    const char* attrs = "{\"model\": \"/data/storage/el2/base/files/qwen2/\"}";
    OH_QoS_GewuCreateSessionResult createResult = OH_QoS_GewuCreateSession(attrs);
    if (createResult.error != OH_QOS_GEWU_OK) {
        return -1;
    }

    // 2. 提交推理请求
    const char* request = "{\"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}], \"stream\": true}";
    OH_QoS_GewuSubmitRequestResult submitResult = OH_QoS_GewuSubmitRequest(
        createResult.session,
        request,
        response_callback,
        NULL
    );

    // 3. 销毁会话
    OH_QoS_GewuDestroySession(createResult.session);

    return 0;
}
```

---

## 6. 相关文档

| 文档 | 描述 |
|------|------|
| [01_Architecture.md](./01_Architecture.md) | 架构图和调用关系 |
| [03_Inner_API.md](./03_Inner_API.md) | 内部 C++ API |
| [05_Security.md](./05_Security.md) | API 安全考量 |
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 详细调用链 |
