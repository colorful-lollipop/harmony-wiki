# Inner API 接口文档

## 1. 概述

Inner API 是供 OpenHarmony 系统内部子系统使用的 Native C++ 接口，主要包括:

- **Native Client API**: 供系统服务调用认证能力
- **Executor Framework API**: 供认证执行器框架使用

**证据**: `interfaces/inner_api/` 目录下的头文件

---

## 2. Native Client API

### 2.1 头文件清单

```
interfaces/inner_api/
├── user_auth_client.h           # 用户认证客户端
├── user_auth_client_callback.h  # 认证回调接口
├── user_auth_client_defines.h   # 认证定义
├── user_idm_client.h            # 用户身份管理客户端
├── user_idm_client_callback.h   # IDM 回调接口
├── user_idm_client_defines.h    # IDM 定义
├── co_auth_client.h             # 协同认证客户端
├── co_auth_client_callback.h    # CoAuth 回调
├── co_auth_client_defines.h     # CoAuth 定义
├── user_access_ctrl_client.h    # 访问控制客户端
├── user_access_ctrl_client_callback.h
├── attributes.h                 # 属性接口
└── iam_common_defines.h         # 公共定义
```

### 2.2 UserAuthClient

**头文件**: `interfaces/inner_api/user_auth_client.h`

#### 主要方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `GetVersion()` | - | int32_t | 获取版本 |
| `GetAvailableStatus()` | authType, atl | int32_t | 获取可用状态 |
| `Auth()` | challenge, authType, atl, callback | uint64_t | 执行认证 |
| `CancelAuth()` | challenge | int32_t | 取消认证 |
| `GetProperty()` | authType, keys | AuthProperty | 获取属性 |
| `SetProperty()` | authType, attr | int32_t | 设置属性 |
| `VerifyAuthToken()` | token, authType, userId | int32_t | 验证令牌 |
| `GetEnrolledState()` | authType | EnrolledState | 获取注册状态 |
| `QueryReusableAuthResult()` | param | ReuseUnlockResult | 查询复用结果 |
| `GetAuthLockState()` | userId | uint64_t | 获取锁定状态 |

#### 使用示例

```cpp
#include "user_auth_client.h"
#include "user_auth_client_callback.h"

using namespace OHOS::UserIam::UserAuth;

auto client = UserAuthClient::GetInstance();
uint64_t challenge = 0n;

// 获取认证实例
int32_t result = client->GetAvailableStatus(FACE, ATL2);
```

### 2.3 UserIdmClient

**头文件**: `interfaces/inner_api/user_idm_client.h`

#### 主要方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `OpenSession()` | userId, callback | 打开会话 |
| `CloseSession()` | userId | 关闭会话 |
| `AddCredential()` | userId, credInfo, callback | 添加凭证 |
| `UpdateCredential()` | userId, credInfo, callback | 更新凭证 |
| `DelCredential()` | userId, credId, callback | 删除凭证 |
| `DelUser()` | userId, callback | 删除用户 |
| `GetCredentialInfo()` | userId, authType, callback | 获取凭证信息 |
| `EnforceDelUser()` | userId | 强制删除用户 |

### 2.4 CoAuthClient

**头文件**: `interfaces/inner_api/co_auth_client.h`

#### 主要方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `ExecutorRegister()` | executorInfo, callback | 注册执行器 |
| `ExecutorUnregister()` | executorId | 注销执行器 |

---

## 3. 回调接口

### 3.1 IUserAuthCallback

**头文件**: `interfaces/inner_api/user_auth_client_callback.h`

```cpp
class IUserAuthCallback {
public:
    virtual void OnResult(int32_t result, const UserAuthResultExtra &extra) = 0;
    virtual void OnAcquireInfo(int32_t module, int32_t acquireInfo) = 0;
};
```

### 3.2 IUserIdmCallback

**头文件**: `interfaces/inner_api/user_idm_client_callback.h`

```cpp
class IUserIdmCallback {
public:
    virtual void OnResult(int32_t result, const std::vector<CredentialInfo> &infoList) = 0;
};
```

### 3.3 ICoAuthCallback

**头文件**: `interfaces/inner_api/co_auth_client_callback.h`

```cpp
class ICoAuthCallback {
public:
    virtual void OnResult(int32_t result, int32_t executorId) = 0;
};
```

---

## 4. 公共定义

### 4.1 AuthTrustLevel (认证信任等级)

**头文件**: `interfaces/inner_api/iam_common_defines.h`

```cpp
enum AuthTrustLevel : int32_t {
    ATL1 = 1,  // 最低
    ATL2 = 2,  // 中等
    ATL3 = 3,  // 较高
    ATL4 = 4   // 最高
};
```

### 4.2 ExecutorSecureLevel (执行器安全等级)

```cpp
enum ExecutorSecureLevel : int32_t {
    ESL0 = 0,  // 无安全要求
    ESL1 = 1,  // 安全环境
    ESL2 = 2,  // 安全环境 + 硬件根信任
    ESL3 = 3   // 安全环境 + 硬件根信任 + 安全存储
};
```

### 4.3 ResultCode (结果码)

```cpp
enum ResultCode : int32_t {
    SUCCESS = 0,
    FAIL = 1,
    GENERAL_ERROR = 2,
    CANCELED = 3,
    TIMEOUT = 4,
    ...
};
```

---

## 5. Executor Framework API

### 5.1 头文件清单

```
interfaces/inner_api/iam_executor/
├── iam_executor_framework_types.h     # 框架类型定义
├── iam_executor_iauth_driver_hdi.h    # 认证驱动 HDI 接口
├── iam_executor_iauth_executor_hdi.h  # 执行器 HDI 接口
├── iam_executor_idriver_manager.h     # 驱动管理器接口
└── iam_executor_iexecute_callback.h   # 执行回调接口
```

### 5.2 IAuthDriverHDI

**头文件**: `iam_executor_iauth_driver_hdi.h`

认证驱动 (如人脸驱动、指纹驱动) 需要实现的接口。

### 5.3 IAuthExecutorHDI

**头文件**: `iam_executor_iauth_executor_hdi.h`

执行器抽象接口，用于与认证框架交互。

### 5.4 IDriverManager

**头文件**: `iam_executor_idriver_manager.h`

驱动管理器，管理所有认证驱动。

---

## 6. 稳定性标注

### 6.1 稳定接口 (Stable)

- `user_auth_client.h` - Inner API，已标记 platformsdk
- `user_idm_client.h` - Inner API
- `co_auth_client.h` - Inner API

### 6.2 不稳定接口 (Unstable)

以下接口为内部使用，可能随时变更:

- `iam_executor_iauth_driver_hdi.h` - HDI 接口，具体实现由驱动提供
- `iam_executor_iauth_executor_hdi.h` - 内部框架接口

> **证据**: `bundle.json` inner_kits 配置中的 header 导出

---

## 7. 依赖方向

```
┌─────────────────────────────────────────┐
│         interfaces/inner_api/            │
├─────────────────────────────────────────┤
│  user_auth_client.h                    │ ◀── frameworks/native/client/
│  user_idm_client.h                      │ ◀── services/
│  co_auth_client.h                       │ ◀── frameworks/native/executors/
│  iam_executor_*.h                       │ ◀── drivers/
└─────────────────────────────────────────┘
```

---

## 8. 版本兼容性

| Header | 版本 | 兼容性 |
|--------|------|--------|
| user_auth_client.h | 4.0 | 稳定 |
| user_idm_client.h | 4.0 | 稳定 |
| co_auth_client.h | 4.0 | 稳定 |

---

## 9. 相关文档

- [架构说明](01_Architecture.md)
- [N-API 接口](02_NAPI.md)
- [构建配置](04_Build.md)
