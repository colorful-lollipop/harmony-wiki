# Inner API 参考

> 目的：提供设备互信认证模块的 C/C++ 内部接口参考，包括主要模块的接口定义、数据结构和回调机制。
>
> 适用范围：需要使用 C/C++ 接口集成设备认证能力的开发者。

---

## 1. 概述

### 1.1 Inner API 位置

| 路径 | 说明 |
|------|------|
| `interfaces/inner_api/device_auth.h` | 主接口定义 |
| `interfaces/inner_api/device_auth_defines.h` | 错误码与数据结构 |
| `interfaces/inner_api/device_auth_ext.h` | 扩展接口 |

### 1.2 主要模块接口

| 模块 | 头文件 | 职责 |
|------|--------|------|
| GroupManager | `services/legacy/group_manager/inc/group_manager.h` | 群组管理 |
| GroupAuthManager | `services/legacy/group_auth/inc/group_auth_manager.h` | 群组认证 |
| IdentityService | `services/identity_service/inc/identity_service.h` | 凭证管理 |
| KeyAgreeSDK | `services/key_agree_sdk/inc/key_agree_sdk.h` | 密钥协商 |

---

## 2. GroupManager 接口

### 2.1 头文件信息

**路径**：`services/legacy/group_manager/inc/group_manager.h`

### 2.2 接口定义

```cpp
/**
 * 设备群组管理器接口
 */
typedef struct {
    // 实例获取
    const DeviceGroupManager *(*GetGmInstance)(void);

    // 回调注册
    int32_t (*RegCallback)(const char *appId, const DeviceAuthCallback *callback);

    // 群组生命周期管理
    int32_t (*CreateGroup)(int32_t osAccountId, int64_t requestId,
                           const char *appId, const char *createParams);
    int32_t (*DeleteGroup)(int32_t osAccountId, int64_t requestId,
                           const char *appId, const char *deleteParams);

    // 成员管理
    int32_t (*AddMemberToGroup)(int32_t osAccountId, int64_t requestId,
                                 const char *appId, const char *addParams);
    int32_t (*DeleteMemberFromGroup)(int32_t osAccountId, int64_t requestId,
                                      const char *appId, const char *deleteParams);

    // 数据处理
    int32_t (*ProcessData)(int64_t requestId, const uint8_t *data, uint32_t dataLen);

    // 查询
    int32_t (*GetGroupInfo)(int32_t osAccountId, const char *appId,
                             const char *queryParams, char **returnGroupVec,
                             uint32_t *groupNum);
} DeviceGroupManager;
```

### 2.3 关键参数说明

#### 2.3.1 appId

| 类型 | 说明 |
|------|------|
| `const char *` | 业务应用标识，由业务方自定义 |

#### 2.3.2 DeviceAuthCallback

```cpp
/**
 * 设备认证回调结构体
 */
typedef struct {
    void (*onTransmit)(int64_t requestId, const uint8_t *data, uint32_t dataLen);
    int32_t (*onSessionKeyReturned)(int64_t requestId, const uint8_t *sessionKey,
                                     uint32_t sessionKeyLen);
    void (*onFinish)(int64_t requestId, const char *result);
    void (*onError)(int64_t requestId, int32_t errorCode, const char *errorMsg);
    int32_t (*onRequest)(int64_t requestId, int32_t requestType,
                          const char *reqParams);
} DeviceAuthCallback;
```

**证据**：`interfaces/inner_api/device_auth.h`

---

## 3. GroupAuthManager 接口

### 3.1 头文件信息

**路径**：`services/legacy/group_auth/inc/group_auth_manager.h`

### 3.2 接口定义

```cpp
/**
 * 设备群组认证管理器接口
 */
typedef struct {
    // 实例获取
    const GroupAuthManager *(*GetGaInstance)(void);

    // 设备认证
    int32_t (*AuthDevice)(int32_t osAccountId, int64_t authReqId,
                           const char *authParams, const DeviceAuthCallback *gaCallback);

    // 数据处理
    int32_t (*ProcessData)(int64_t authReqId, const uint8_t *data,
                            uint32_t dataLen, const DeviceAuthCallback *gaCallback);

    // 取消认证
    int32_t (*CancelAuth)(int64_t authReqId);
} GroupAuthManager;
```

### 3.3 认证参数格式

`authParams` JSON 结构：
```json
{
  "groupId": "string",       // 群组 ID
  "peerDeviceId": "string",  // 对端设备 ID
  "authType": number,         // 认证类型
  "sessionId": number         // 会话 ID（可选）
}
```

---

## 4. IdentityService 接口

### 4.1 头文件信息

**路径**：`services/identity_service/inc/identity_service.h`

### 4.2 CredManager 接口

```cpp
/**
 * 凭证管理器接口
 */
typedef struct {
    // 凭证管理
    int32_t (*addCredential)(int32_t osAccountId, const char *requestParams,
                              char **returnData);
    int32_t (*exportCredential)(int32_t osAccountId, const char *credId,
                                 char **returnData);
    int32_t (*queryCredentialByParams)(int32_t osAccountId,
                                        const char *requestParams,
                                        char **returnData);
    int32_t (*queryCredInfoByCredId)(int32_t osAccountId,
                                       const char *credId, char **returnData);
    int32_t (*deleteCredential)(int32_t osAccountId, const char *credId);
    int32_t (*updateCredInfo)(int32_t osAccountId, const char *credId,
                               const char *requestParams);
    int32_t (*agreeCredential)(int32_t osAccountId, const char *selfCredId,
                                const char *requestParams, char **returnData);

    // 监听器管理
    int32_t (*registerChangeListener)(const char *appId,
                                       CredChangeListener *listener);
    int32_t (*unregisterChangeListener)(const char *appId);

    // 批量操作
    int32_t (*deleteCredByParams)(int32_t osAccountId,
                                   const char *requestParams,
                                   char **returnData);
    int32_t (*batchUpdateCredentials)(int32_t osAccountId,
                                       const char *requestParams,
                                       char **returnData);

    // 资源释放
    void (*destroyInfo)(char **returnData);
} CredManager;
```

### 4.3 CredChangeListener 接口

```cpp
/**
 * 凭证变更监听器
 */
typedef struct {
    void (*onCredAdd)(int32_t osAccountId, const char *credId);
    void (*onCredDelete)(int32_t osAccountId, const char *credId);
    void (*onCredUpdate)(int32_t osAccountId, const char *credId);
} CredChangeListener;
```

**证据**：`services/identity_service/inc/identity_service.h`

---

## 5. 错误码

### 5.1 错误码范围

| 范围 | 值 | 说明 |
|------|-----|------|
| 通用错误 | 0x00000000 ~ 0x00000FFF | 基础错误码 |
| 算法适配器 | 0x00001000 ~ 0x00001FFF | HUKS/mbedtls |
| JSON 工具 | 0x00002000 ~ 0x00002FFF | JSON 解析 |
| IPC | 0x00003000 ~ 0x00003FFF | IPC 通信 |
| 模块错误 | 0x00004000 ~ 0x00004FFF | 协议模块 |
| 群组错误 | 0x00005000 ~ 0x00005FFF | 群组管理 |
| 数据库 | 0x00006000 ~ 0x00006FFF | 数据存储 |
| IS 服务 | 0x00010000 ~ | 身份服务 |

### 5.2 常用错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `HC_SUCCESS` | 成功 |
| 2 | `HC_ERR_INVALID_PARAMS` | 参数错误 |
| 4 | `HC_ERR_NULL_PTR` | 空指针 |
| 5 | `HC_ERR_ALLOC_MEMORY` | 内存分配失败 |
| 9 | `HC_ERR_TIME_OUT` | 超时 |
| 12289 | `HC_ERR_IPC_INTERNAL_FAILED` | IPC 内部错误 |
| 16385 | `HC_ERR_ACCESS_DENIED` | 访问拒绝 |
| 20483 | `HC_ERR_SERVICE_NEED_RESTART` | 服务需重启 |
| 65537 | `IS_ERR_HUKS_GENERATE_KEY_FAILED` | HUKS 密钥生成失败 |
| 65545 | `IS_ERR_LOCAL_CRED_NOT_EXIST` | 本地凭据不存在 |
| 65551 | `IS_ERR_AUTH_ERR_PIN_NOT_MATCH` | PIN 码不匹配 |

**证据**：`interfaces/inner_api/device_auth_defines.h`

---

## 6. 使用示例

### 6.1 获取实例

```cpp
#include "device_auth.h"

const DeviceGroupManager *gm = GetGmInstance();
if (gm == nullptr) {
    printf("获取 GroupManager 失败\n");
    return;
}
```

### 6.2 创建群组

```cpp
int32_t CreateTrustedGroup(int32_t osAccountId, const char *appId)
{
    const DeviceGroupManager *gm = GetGmInstance();
    if (gm == nullptr) {
        return HC_ERR_NULL_PTR;
    }

    char *result = nullptr;
    uint32_t groupNum = 0;

    // 构造创建参数
    const char *createParams = R"({"groupName":"test","groupType":1})";

    int32_t ret = gm->GetGroupInfo(osAccountId, appId,
                                     "{\"queryType\":1}", &result,
                                     &groupNum);
    if (ret != HC_SUCCESS) {
        printf("查询群组失败: %d\n", ret);
        return ret;
    }

    gm->destroyInfo(&result);
    return HC_SUCCESS;
}
```

### 6.3 注册回调

```cpp
void OnTransmit(int64_t requestId, const uint8_t *data, uint32_t dataLen)
{
    // 发送数据到对端设备
    SendToPeer(data, dataLen);
}

void OnFinish(int64_t requestId, const char *result)
{
    printf("操作完成: %s\n", result);
}

void OnError(int64_t requestId, int32_t errorCode, const char *errorMsg)
{
    printf("操作失败 [%d]: %s\n", errorCode, errorMsg);
}

DeviceAuthCallback callback = {
    .onTransmit = OnTransmit,
    .onFinish = OnFinish,
    .onError = OnError,
};

int32_t RegAppCallback(const char *appId)
{
    const DeviceGroupManager *gm = GetGmInstance();
    return gm->RegCallback(appId, &callback);
}
```

---

## 7. 相关跳转

| 内容 | 文档 |
|------|------|
| 项目概览 | [01_Overview.md](./01_Overview.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| N-API | [03_API_Reference.md](./03_API_Reference.md) |
| 构建配置 | [05_Build_Config.md](./05_Build_Config.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*本文档最后更新：2026-02-06*
