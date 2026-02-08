# 对外接口文档

> 最后更新：2026-02-07
> 版本：v3.0.0

## 4.1 C API 参考

### 头文件包含

使用 DSL M SDK 需要包含以下头文件：

```cpp
#include "device_security_defines.h"  // 数据结构和错误码定义
#include "device_security_info.h"     // API 函数声明
```

**证据**：`interfaces/inner_api/include/device_security_info.h:21-22`

---

### API 函数清单

#### 1. RequestDeviceSecurityInfo - 同步查询

**函数签名**：

```c
int32_t RequestDeviceSecurityInfo(
    const DeviceIdentify *identify,      // [in] 设备标识
    const RequestOption *option,          // [in] 查询选项
    DeviceSecurityInfo **info            // [out] 输出的安全信息
);
```

**功能描述**：

同步获取指定设备的系统安全等级信息。该函数会阻塞调用线程，直到获取到结果或发生错误。

**参数说明**：

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| `identify` | `const DeviceIdentify *` | 输入 | 目标设备标识，传入 `NULL` 表示查询本机 |
| `option` | `const RequestOption *` | 输入 | 查询选项配置，传入 `NULL` 使用默认配置 |
| `info` | `DeviceSecurityInfo **` | 输出 | 输出参数，指向安全信息结构体的指针 |

**返回值**：

| 返回值 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `SUCCESS` | 成功 |
| 正数 | 见错误码表 | 错误码 |

**使用示例**：

```cpp
void SyncQueryExample(void)
{
    DeviceSecurityInfo *info = NULL;
    
    // 查询本机安全等级
    int32_t ret = RequestDeviceSecurityInfo(NULL, NULL, &info);
    if (ret != SUCCESS) {
        printf("查询失败，错误码: %d\n", ret);
        return;
    }
    
    int32_t level = 0;
    ret = GetDeviceSecurityLevelValue(info, &level);
    if (ret == SUCCESS) {
        printf("本机安全等级: SL%d\n", level);
    }
    
    FreeDeviceSecurityInfo(info);
}
```

**证据**：`interfaces/inner_api/include/device_security_info.h:42-43`

---

#### 2. RequestDeviceSecurityInfoAsync - 异步查询

**函数签名**：

```c
int32_t RequestDeviceSecurityInfoAsync(
    const DeviceIdentify *identify,              // [in] 设备标识
    const RequestOption *option,                 // [in] 查询选项
    DeviceSecurityInfoCallback callback          // [in] 回调函数
);
```

**功能描述**：

异步获取指定设备的系统安全等级信息。函数立即返回，结果通过回调函数异步通知。

**参数说明**：

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| `identify` | `const DeviceIdentify *` | 输入 | 目标设备标识，传入 `NULL` 表示查询本机 |
| `option` | `const RequestOption *` | 输入 | 查询选项配置，传入 `NULL` 使用默认配置 |
| `callback` | `DeviceSecurityInfoCallback` | 输入 | 结果回调函数，不能为 `NULL` |

**回调函数类型**：

```c
typedef void DeviceSecurityInfoCallback(
    const DeviceIdentify *identify,           // 查询的设备标识
    struct DeviceSecurityInfo *info            // 安全等级信息
);
```

**返回值**：

| 返回值 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `SUCCESS` | 成功发起请求，回调会被调用 |
| 正数 | 见错误码表 | 失败，回调不会被调用 |

**使用示例**：

```cpp
// 异步回调函数
void OnSecurityInfoResult(const DeviceIdentify *identify, 
                         DeviceSecurityInfo *info)
{
    int32_t level = 0;
    int32_t ret = GetDeviceSecurityLevelValue(info, &level);
    
    if (ret == SUCCESS) {
        printf("异步查询成功，设备安全等级: SL%d\n", level);
    } else {
        printf("异步查询失败，错误码: %d\n", ret);
    }
    
    // 必须在回调中释放资源
    FreeDeviceSecurityInfo(info);
}

void AsyncQueryExample(const DeviceIdentify *device)
{
    // 发起异步请求
    int32_t ret = RequestDeviceSecurityInfoAsync(device, NULL, 
                                                  OnSecurityInfoResult);
    if (ret != SUCCESS) {
        printf("发起异步请求失败，错误码: %d\n", ret);
        return;
    }
    
    printf("异步请求已发起\n");
    // 等待回调被调用
}
```

**证据**：`interfaces/inner_api/include/device_security_info.h:53-54`

---

#### 3. FreeDeviceSecurityInfo - 释放资源

**函数签名**：

```c
void FreeDeviceSecurityInfo(DeviceSecurityInfo *info);
```

**功能描述**：

释放由 `RequestDeviceSecurityInfo()` 或 `RequestDeviceSecurityInfoAsync()` 回调中分配的 `DeviceSecurityInfo` 对象。

**参数说明**：

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| `info` | `DeviceSecurityInfo *` | 输入 | 待释放的安全信息对象 |

**使用示例**：

```cpp
void CorrectUsage(void)
{
    DeviceSecurityInfo *info = NULL;
    
    int32_t ret = RequestDeviceSecurityInfo(NULL, NULL, &info);
    if (ret == SUCCESS) {
        int32_t level = 0;
        GetDeviceSecurityLevelValue(info, &level);
        printf("安全等级: SL%d\n", level);
    }
    
    // 正确释放
    if (info != NULL) {
        FreeDeviceSecurityInfo(info);
        info = NULL;
    }
}
```

**证据**：`interfaces/inner_api/include/device_security_info.h:60`

---

#### 4. GetDeviceSecurityLevelValue - 提取等级值

**函数签名**：

```c
int32_t GetDeviceSecurityLevelValue(
    const DeviceSecurityInfo *info,   // [in] 安全信息对象
    int32_t *level                    // [out] 输出的等级值
);
```

**功能描述**：

从安全信息对象中提取设备安全等级值。

**参数说明**：

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| `info` | `const DeviceSecurityInfo *` | 输入 | 安全信息对象 |
| `level` | `int32_t *` | 输出 | 输出的等级值 (SL1-SL5) |

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败，info 中的 result 字段值 |

**使用示例**：

```cpp
void ExtractLevelExample(const DeviceSecurityInfo *info)
{
    int32_t level = 0;
    int32_t ret = GetDeviceSecurityLevelValue(info, &level);
    
    if (ret == SUCCESS) {
        // 等级值范围: SL1(1) - SL5(5)
        printf("设备安全等级: SL%d\n", level);
        
        if (level >= 3) {
            printf("满足 SL3+ 要求\n");
        }
    }
}
```

**证据**：`interfaces/inner_api/include/device_security_info.h:68`

---

## 4.2 数据结构

### DeviceIdentify - 设备标识

```c
// interfaces/inner_api/include/device_security_defines.h:27-30
#define DEVICE_ID_MAX_LEN 64

typedef struct DeviceIdentify {
    uint32_t length;                   // 标识数据的实际长度
    uint8_t identity[DEVICE_ID_MAX_LEN]; // 标识数据（64字节）
} DeviceIdentify;
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| `length` | `uint32_t` | `identity` 数组中实际使用的字节数 |
| `identity` | `uint8_t[64]` | 设备标识的原始字节数据 |

**使用说明**：

- 标识长度为 64 字节的固定数组
- `length` 字段表示实际使用的字节数
- 标识由 Device Manager 生成和管理

---

### RequestOption - 查询选项

```c
// interfaces/inner_api/include/device_security_defines.h:32-36
typedef struct RequestOption {
    uint64_t challenge;      // 挑战值，用于防重放攻击
    uint32_t timeout;        // 超时时间（秒），范围 1-300，默认 45
    uint32_t extra;         // 保留字段
} RequestOption;
```

**字段说明**：

| 字段 | 类型 | 默认值 | 范围 | 说明 |
|------|------|--------|------|------|
| `challenge` | `uint64_t` | 0 | - | 挑战值，0 表示由服务生成 |
| `timeout` | `uint32_t` | 45 | 1-300 | 超时时间（秒） |
| `extra` | `uint32_t` | 0 | - | 保留字段 |

**预定义常量**：

```c
#define DEFAULT_OPTION NULL  // 使用所有默认配置
```

**使用示例**：

```cpp
void WithCustomOption(void)
{
    RequestOption option = {0};
    
    // 使用自定义配置
    option.challenge = 1234567890ULL;  // 自定义挑战值
    option.timeout = 60;                 // 60 秒超时
    
    DeviceSecurityInfo *info = NULL;
    int32_t ret = RequestDeviceSecurityInfo(device, &option, &info);
    // ...
}
```

---

### DeviceSecurityInfo - 安全信息对象

```c
// 内部结构定义（仅供参考）
struct DeviceSecurityInfo {
    uint32_t magicNum;     // 验证魔数：0xABCD1234
    uint32_t result;       // 操作结果码
    uint32_t level;        // 设备安全等级 (SL1-SL5)
};
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| `magicNum` | `uint32_t` | 验证魔数，用于确认对象有效性 |
| `result` | `uint32_t` | 操作结果码，0 表示成功 |
| `level` | `uint32_t` | 设备安全等级，范围 1-5 |

**重要说明**：

- 这是一个不透明类型（opaque type）
- 应用程序不应直接访问内部字段
- 使用 `GetDeviceSecurityLevelValue()` 提取等级值
- 使用 `FreeDeviceSecurityInfo()` 释放对象

---

### DslmCredInfo - 凭证信息

```c
// oem_property/include/dslm_cred.h
typedef struct DslmCredInfo {
    uint32_t magicNum;              // 验证魔数
    uint32_t credType;               // 凭证类型
    uint32_t securityLevel;         // 安全等级
    uint8_t udid[64];              // 设备唯一标识
    char manufacture[64];           // 制造商名称
    char brand[64];                 // 品牌
    char model[64];                 // 型号
    char softwareVersion[64];       // 软件版本
    char securityLevelStr[16];      // 安全等级字符串
    uint64_t signTime;              // 签名时间
    // ... 更多字段
} DslmCredInfo;
```

---

## 4.3 错误码参考

### 错误码定义

```c
// interfaces/inner_api/include/device_security_defines.h:40-92
enum {
    SUCCESS = 0,
    ERR_INVALID_PARA = 1,
    ERR_INVALID_LEN_PARA = 2,
    ERR_NO_MEMORY = 3,
    ERR_MEMORY_ERR = 4,
    ERR_NO_CHALLENGE = 5,
    ERR_NO_CRED = 6,
    ERR_SA_BUSY = 7,
    ERR_TIMEOUT = 8,
    ERR_NOEXIST_REQUEST = 9,
    ERR_INVALID_VERSION = 10,
    ERR_OEM_ERR = 11,
    ERR_HUKS_ERR = 12,
    ERR_CHALLENGE_ERR = 13,
    ERR_NOT_ONLINE = 14,
    ERR_INIT_SELF_ERR = 15,
    ERR_JSON_ERR = 16,
    ERR_IPC_ERR = 17,
    ERR_IPC_REGISTER_ERR = 18,
    ERR_IPC_REMOTE_OBJ_ERR = 19,
    ERR_IPC_PROXY_ERR = 20,
    ERR_IPC_RET_PARCEL_ERR = 21,
    ERR_PROXY_REMOTE_ERR = 22,
    ERR_MSG_NEIGHBOR_FULL = 23,
    ERR_MSG_FULL = 24,
    ERR_MSG_ADD_NEIGHBOR = 25,
    ERR_MSG_NOT_INIT = 26,
    ERR_MSG_CREATE_WORKQUEUE = 27,
    ERR_NEED_COMPATIBLE = 28,
    ERR_REG_CALLBACK = 29,
    ERR_PERMISSION_DENIAL = 30,
    ERR_REQUEST_CODE_ERR = 31,
    ERR_VERIFY_MODE_CRED_ERR = 32,
    ERR_VERIFY_SIGNED_MODE_CRED_ERR = 33,
    ERR_VERIFY_MODE_HUKS_ERR = 34,
    ERR_PROFILE_CONNECT_ERR = 35,
    ERR_MSG_OPEN_SESSION = 36,
    ERR_QUERY_WAITING = 37,
    ERR_NOEXIST_DEVICE = 38,
    ERR_NOEXIST_COMMON_PK_INFO = 39,
    ERR_ECC_VERIFY_ERR = 40,
    ERR_GET_CLOUD_CRED_INFO = 41,
    ERR_CALL_EXTERNAL_FUNC = 42,
    ERR_PARSE_NONCE = 43,
    ERR_ROOT_PUBKEY_NOT_RIGHT = 44,
    ERR_PARSE_CLOUD_CRED_DATA = 45,
    ERR_PARSE_PUBKEY_CHAIN = 46,
    ERR_CHECK_CRED_INFO = 47,
    ERR_ATTEST_NOT_READY = 48,
    ERR_CLOUD_CRED_NOT_EXIST = 49,
    ERR_DEFAULT = 0xFFFF
};
```

### 错误码分类

| 分类 | 错误码范围 | 说明 |
|------|------------|------|
| **成功** | 0 | `SUCCESS` |
| **参数错误** | 1-4 | 参数校验失败 |
| **凭证错误** | 5-6, 11, 32-34, 40 | 凭证相关错误 |
| **超时错误** | 8 | 操作超时 |
| **IPC 错误** | 17-22 | 进程间通信错误 |
| **消息错误** | 23-27, 36 | 消息处理错误 |
| **权限错误** | 30 | 权限校验失败 |
| **初始化错误** | 14-15, 26, 48 | 服务初始化错误 |
| **解析错误** | 16, 43, 45-47 | 数据解析错误 |
| **默认错误** | 0xFFFF | 未分类错误 |

### 常见错误处理

```cpp
void HandleErrors(int32_t ret)
{
    switch (ret) {
        case SUCCESS:
            printf("操作成功\n");
            break;
            
        case ERR_INVALID_PARA:
            printf("错误: 无效参数\n");
            break;
            
        case ERR_NO_MEMORY:
            printf("错误: 内存分配失败\n");
            break;
            
        case ERR_TIMEOUT:
            printf("错误: 操作超时\n");
            break;
            
        case ERR_NOT_ONLINE:
            printf("错误: 设备不在线\n");
            break;
            
        case ERR_PERMISSION_DENIAL:
            printf("错误: 权限拒绝\n");
            break;
            
        default:
            printf("错误: 未知错误码 %d\n", ret);
            break;
    }
}
```

---

## 4.4 完整使用示例

### 同步查询完整示例

```cpp
#include <stdio.h>
#include "device_security_defines.h"
#include "device_security_info.h"

void CheckDeviceSecurityLevel(const DeviceIdentify *device)
{
    DeviceSecurityInfo *info = NULL;
    
    // 步骤 1: 发起同步查询
    int32_t ret = RequestDeviceSecurityInfo(device, NULL, &info);
    if (ret != SUCCESS) {
        printf("[ERROR] 查询失败，错误码: %d\n", ret);
        HandleError(ret);
        return;
    }
    
    // 步骤 2: 提取安全等级
    int32_t level = 0;
    ret = GetDeviceSecurityLevelValue(info, &level);
    if (ret != SUCCESS) {
        printf("[ERROR] 提取等级失败，错误码: %d\n", ret);
        FreeDeviceSecurityInfo(info);
        return;
    }
    
    // 步骤 3: 判断安全等级
    printf("[INFO] 设备安全等级: SL%d\n", level);
    
    // 业务判断示例
    if (level >= 3) {
        printf("[PASS] 设备满足 SL3+ 安全要求\n");
    } else {
        printf("[WARN] 设备安全等级不足 (SL3+ required)\n");
    }
    
    // 步骤 4: 释放资源
    FreeDeviceSecurityInfo(info);
}
```

### 异步查询完整示例

```cpp
#include <stdio.h>
#include <pthread.h>
#include "device_security_defines.h"
#include "device_security_info.h"

static pthread_mutex_t result_mutex = PTHREAD_MUTEX_INITIALIZER;
static pthread_cond_t result_cond = PTHREAD_COND_INITIALIZER;
static volatile int32_t async_result = -1;

void AsyncCallback(const DeviceIdentify *identify, DeviceSecurityInfo *info)
{
    int32_t level = 0;
    int32_t ret = GetDeviceSecurityLevelValue(info, &level);
    
    pthread_mutex_lock(&result_mutex);
    if (ret == SUCCESS) {
        printf("[CALLBACK] 设备安全等级: SL%d\n", level);
        async_result = level;
    } else {
        printf("[CALLBACK] 查询失败，错误码: %d\n", ret);
        async_result = -1;
    }
    pthread_cond_signal(&result_cond);
    pthread_mutex_unlock(&result_mutex);
    
    // 必须在回调中释放
    FreeDeviceSecurityInfo(info);
}

int32_t AsyncQueryWithWait(const DeviceIdentify *device, int32_t timeout_sec)
{
    pthread_mutex_lock(&result_mutex);
    
    // 发起异步请求
    int32_t ret = RequestDeviceSecurityInfoAsync(device, NULL, AsyncCallback);
    if (ret != SUCCESS) {
        printf("[ERROR] 发起请求失败: %d\n", ret);
        pthread_mutex_unlock(&result_mutex);
        return -1;
    }
    
    // 等待回调完成
    struct timespec ts;
    clock_gettime(CLOCK_REALTIME, &ts);
    ts.tv_sec += timeout_sec;
    
    ret = pthread_cond_timedwait(&result_cond, &result_mutex, &ts);
    if (ret == ETIMEDOUT) {
        printf("[WARN] 等待超时\n");
    }
    
    int32_t result = async_result;
    async_result = -1;
    
    pthread_mutex_unlock(&result_mutex);
    
    return result;
}
```

---

## 4.5 最佳实践

### 资源管理

```cpp
// ✅ 正确: 在所有分支释放资源
void CorrectResourceManagement(void)
{
    DeviceSecurityInfo *info = NULL;
    
    int32_t ret = RequestDeviceSecurityInfo(device, NULL, &info);
    if (ret == SUCCESS) {
        int32_t level = 0;
        ret = GetDeviceSecurityLevelValue(info, &level);
        if (ret == SUCCESS) {
            // 业务处理
        }
    }
    
    // 所有执行路径都会释放
    if (info != NULL) {
        FreeDeviceSecurityInfo(info);
    }
}

// ❌ 错误: 提前返回未释放
void IncorrectResourceManagement(void)
{
    DeviceSecurityInfo *info = NULL;
    
    int32_t ret = RequestDeviceSecurityInfo(device, NULL, &info);
    if (ret != SUCCESS) {
        return;  // info 仍为 NULL，此处安全
    }
    
    int32_t level = 0;
    ret = GetDeviceSecurityLevelValue(info, &level);
    if (ret != SUCCESS) {
        return;  // ❌ 内存泄漏！未释放 info
    }
    
    FreeDeviceSecurityInfo(info);
}
```

### 超时设置

```cpp
void SetTimeoutExample(void)
{
    RequestOption option = {0};
    
    // 设置合理超时 (1-300 秒)
    option.timeout = 30;  // 30 秒超时
    
    DeviceSecurityInfo *info = NULL;
    int32_t ret = RequestDeviceSecurityInfo(device, &option, &info);
    // ...
}
```

### 错误处理

```cpp
void RobustErrorHandling(const DeviceIdentify *device)
{
    DeviceSecurityInfo *info = NULL;
    
    // 发起查询
    int32_t ret = RequestDeviceSecurityInfo(device, NULL, &info);
    if (ret != SUCCESS) {
        // 分类处理错误
        switch (ret) {
            case ERR_NOT_ONLINE:
                printf("设备不在线，请确认设备已连接\n");
                break;
            case ERR_TIMEOUT:
                printf("查询超时，请稍后重试\n");
                break;
            case ERR_PERMISSION_DENIAL:
                printf("权限不足，无法查询设备安全等级\n");
                break;
            default:
                printf("查询失败，错误码: %d\n", ret);
                break;
        }
        return;
    }
    
    // 提取等级
    int32_t level = 0;
    ret = GetDeviceSecurityLevelValue(info, &level);
    if (ret != SUCCESS) {
        printf("提取等级失败\n");
        FreeDeviceSecurityInfo(info);
        return;
    }
    
    printf("设备安全等级: SL%d\n", level);
    FreeDeviceSecurityInfo(info);
}
```

---

## 下一章

- [05_AttackSurface.md](./05_AttackSurface.md) - 攻击面分析
- [06_SecurityReview.md](./06_SecurityReview.md) - 安全风险评估
- [07_Build.md](./07_Build.md) - 构建与产物
