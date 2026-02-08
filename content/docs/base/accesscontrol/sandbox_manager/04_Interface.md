# 对外接口文档 (Interface)

> SandboxManagerKit C++ API 完整清单、参数说明与使用示例

---

## 4.1 API 概览

### SandboxManagerKit 类

**命名空间**：`OHOS::AccessControl::SandboxManager`

**头文件**：`interfaces/inner_api/sandbox_manager/include/sandbox_manager_kit.h`

**职责**：提供 C++ SDK 接口，供系统内部组件调用 Sandbox Manager 服务。

---

## 4.2 API 详细说明

### 4.2.1 持久化策略管理

#### PersistPolicy

```cpp
static int32_t PersistPolicy(
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result);

static int32_t PersistPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result);
```

**描述**：将文件访问策略持久化存储到数据库，策略在应用卸载前一直有效。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | （可选）指定的 Token ID，为 0 时使用调用者 Token |
| `policy` | `std::vector<PolicyInfo>` | 策略列表，每条策略包含 path、mode、type |
| `result` | `std::vector<uint32_t>&` | 返回每条策略的存储结果 |

**返回**：`SandboxManagerErrCode`
- `SANDBOX_MANAGER_OK`：成功
- `PERMISSION_DENIED`：权限不足
- `INVALID_PARAMTER`：参数错误

**result 元素含义**（`SandboxRetType`）：
| 值 | 宏定义 | 说明 |
|---|--------|------|
| 0 | `OPERATE_SUCCESSFULLY` | 成功 |
| 1 | `FORBIDDEN_TO_BE_PERSISTED` | 禁止持久化 |
| 2 | `INVALID_MODE` | 无效模式 |
| 3 | `INVALID_PATH` | 无效路径 |
| 5 | `POLICY_MAC_FAIL` | MAC 验证失败 |

**权限要求**：`ohos.permission.FILE_ACCESS_PERSIST`

**使用示例**：

```cpp
// 示例 1：使用调用者 Token 持久化
std::vector<PolicyInfo> policies;
PolicyInfo policy;
policy.path = "/storage/Users/100/appdata/com.example/shared";
policy.mode = READ_MODE | WRITE_MODE;  // 0x03
policy.type = AUTHORIZATION_PATH;
policies.push_back(policy);

std::vector<uint32_t> results;
int32_t ret = SandboxManagerKit::PersistPolicy(policies, results);

if (ret == SANDBOX_MANAGER_OK && results[0] == OPERATE_SUCCESSFULLY) {
    // 策略持久化成功
}
```

---

#### UnPersistPolicy

```cpp
static int32_t UnPersistPolicy(
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result);

static int32_t UnPersistPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result);
```

**描述**：从数据库中删除持久化的策略。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | （可选）指定的 Token ID |
| `policy` | `std::vector<PolicyInfo>` | 要删除的策略列表 |
| `result` | `std::vector<uint32_t>&` | 返回每条策略的删除结果 |

**权限要求**：`ohos.permission.FILE_ACCESS_PERSIST`

---

#### CheckPersistPolicy

```cpp
static int32_t CheckPersistPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    std::vector<bool> &result);
```

**描述**：检查指定 Token 的持久化策略是否存在。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | Token ID |
| `policy` | `std::vector<PolicyInfo>` | 要检查的策略列表 |
| `result` | `std::vector<bool>&` | 返回每条策略的存在状态 |

**权限要求**：`ohos.permission.CHECK_SANDBOX_POLICY`

---

### 4.2.2 临时策略管理

#### SetPolicy

```cpp
static int32_t SetPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    uint64_t policyFlag,
    std::vector<uint32_t> &result);

static int32_t SetPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    uint64_t policyFlag,
    std::vector<uint32_t> &result, 
    const SetInfo &setInfo);

static int32_t SetPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    uint64_t policyFlag,
    std::vector<uint32_t> &result, 
    uint64_t timestamp);
```

**描述**：将策略设置到 MAC 内核层，仅在当前应用生命周期内有效。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | 目标应用的 Token ID |
| `policy` | `std::vector<PolicyInfo>` | 策略列表 |
| `policyFlag` | `uint64_t` | 策略标志（0 或 1） |
| `result` | `std::vector<uint32_t>&` | 返回每条策略的设置结果 |
| `setInfo` | `SetInfo` | （可选）额外信息（bundleName、timestamp） |
| `timestamp` | `uint64_t` | （可选）时间戳 |

**权限要求**：`ohos.permission.SET_SANDBOX_POLICY`

**使用示例**：

```cpp
// 示例：设置临时读权限
std::vector<PolicyInfo> policies;
PolicyInfo policy;
policy.path = "/storage/Users/100/appdata/com.example/temp";
policy.mode = READ_MODE;  // 0x01
policy.type = SELF_PATH;
policies.push_back(policy);

std::vector<uint32_t> results;
int32_t ret = SandboxManagerKit::SetPolicy(targetTokenId, policies, 0, results);
```

---

#### UnSetPolicy

```cpp
static int32_t UnSetPolicy(
    uint32_t tokenId, 
    const PolicyInfo &policy);
```

**描述**：从 MAC 层删除临时策略。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | 目标应用的 Token ID |
| `policy` | `const PolicyInfo&` | 要删除的策略 |

**权限要求**：`ohos.permission.SET_SANDBOX_POLICY`

---

#### SetDenyPolicy / UnSetDenyPolicy

```cpp
static int32_t SetDenyPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy,
    std::vector<uint32_t> &result);

static int32_t UnSetDenyPolicy(
    uint32_t tokenId, 
    const PolicyInfo &policy);
```

**描述**：设置/删除拒绝策略，禁止对指定路径的访问。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | 目标应用的 Token ID |
| `policy` | `std::vector<PolicyInfo>` | 拒绝策略列表 |
| `result` | `std::vector<uint32_t>&` | 返回结果 |

**权限要求**：`SET_SANDBOX_POLICY` + UID == 7013 (Space Manager)

---

### 4.2.3 异步策略操作

#### SetPolicyAsync

```cpp
static int32_t SetPolicyAsync(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    uint64_t policyFlag);

static int32_t SetPolicyAsync(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    uint64_t policyFlag,
    uint64_t timestamp);
```

**描述**：异步设置临时策略，调用后立即返回，不等待 MAC 层操作完成。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | 目标应用的 Token ID |
| `policy` | `std::vector<PolicyInfo>` | 策略列表 |
| `policyFlag` | `uint64_t` | 策略标志 |
| `timestamp` | `uint64_t` | （可选）时间戳 |

**权限要求**：`ohos.permission.SET_SANDBOX_POLICY`

---

#### UnSetPolicyAsync

```cpp
static int32_t UnSetPolicyAsync(
    uint32_t tokenId, 
    const PolicyInfo &policy);
```

**描述**：异步删除临时策略。

---

### 4.2.4 策略检查

#### CheckPolicy

```cpp
static int32_t CheckPolicy(
    uint32_t tokenId, 
    const std::vector<PolicyInfo> &policy, 
    std::vector<bool> &result);
```

**描述**：检查指定应用是否对某路径拥有指定权限。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | 要检查的 Token ID |
| `policy` | `std::vector<PolicyInfo>` | 策略列表（path + mode） |
| `result` | `std::vector<bool>&` | 返回每条策略的检查结果 |

**权限要求**：
- 调用者 Token == 目标 Token，或
- 拥有 `ohos.permission.CHECK_SANDBOX_POLICY` 权限

**使用示例**：

```cpp
std::vector<PolicyInfo> policies;
PolicyInfo policy;
policy.path = "/storage/Users/100/appdata/com.example/data";
policy.mode = READ_MODE;
policies.push_back(policy);

std::vector<bool> results;
int32_t ret = SandboxManagerKit::CheckPolicy(targetTokenId, policies, results);

if (ret == SANDBOX_MANAGER_OK && results[0]) {
    // 目标应用拥有读权限
}
```

---

### 4.2.5 策略激活与停用

#### StartAccessingPolicy

```cpp
static int32_t StartAccessingPolicy(
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result);

static int32_t StartAccessingPolicy(
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result,
    bool useCallerToken, 
    uint32_t tokenId, 
    uint64_t timestamp);
```

**描述**：激活持久化策略，将策略从数据库加载到 MAC 层。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `policy` | `std::vector<PolicyInfo>` | 要激活的策略列表 |
| `result` | `std::vector<uint32_t>&` | 返回每条策略的激活结果 |
| `useCallerToken` | `bool` | 是否使用调用者 Token |
| `tokenId` | `uint32_t` | 指定的 Token ID |
| `timestamp` | `uint64_t` | 时间戳 |

**权限要求**：`ohos.permission.FILE_ACCESS_PERSIST`

---

#### StopAccessingPolicy

```cpp
static int32_t StopAccessingPolicy(
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result);
```

**描述**：停用持久化策略，从 MAC 层移除但保留数据库记录。

**权限要求**：`ohos.permission.FILE_ACCESS_PERSIST`

---

#### StartAccessingByTokenId

```cpp
static int32_t StartAccessingByTokenId(uint32_t tokenId);
static int32_t StartAccessingByTokenId(uint32_t tokenId, uint64_t timestamp);
```

**描述**：按 Token ID 自动激活所有已持久化的策略。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `tokenId` | `uint32_t` | 要激活的 Token ID |
| `timestamp` | `uint64_t` | （可选）时间戳 |

---

#### UnSetAllPolicyByToken

```cpp
static int32_t UnSetAllPolicyByToken(uint32_t tokenId);
static int32_t UnSetAllPolicyByToken(uint32_t tokenId, uint64_t timestamp);
```

**描述**：清除指定 Token 的所有临时策略。

---

### 4.2.6 策略清理

#### CleanPersistPolicyByPath

```cpp
static int32_t CleanPersistPolicyByPath(
    const std::vector<std::string> &filePathList);
```

**描述**：按路径清理持久化策略（仅文件管理器可调用）。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `filePathList` | `std::vector<std::string>` | 要清理的路径列表 |

**权限要求**：文件管理器服务 (tokenFileManagerId_)

---

#### CleanPolicyByUserId

```cpp
static int32_t CleanPolicyByUserId(
    uint32_t userId, 
    const std::vector<std::string> &filePathList);
```

**描述**：按用户 ID 清理持久化策略。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `userId` | `uint32_t` | 用户 ID |
| `filePathList` | `std::vector<std::string>` | 路径列表 |

---

### 4.2.7 其他操作

#### SetPolicyByBundleName

```cpp
static int32_t SetPolicyByBundleName(
    const std::string &bundleName, 
    int32_t appCloneIndex,
    const std::vector<PolicyInfo> &policy, 
    uint64_t policyFlag, 
    std::vector<uint32_t> &result);
```

**描述**：按包名设置策略。

**参数**：
| 参数 | 类型 | 说明 |
|-----|------|------|
| `bundleName` | `const std::string&` | 应用包名 |
| `appCloneIndex` | `int32_t` | 应用克隆索引 |
| `policy` | `std::vector<PolicyInfo>` | 策略列表 |
| `policyFlag` | `uint64_t` | 策略标志 |
| `result` | `std::vector<uint32_t>&` | 返回结果 |

**权限要求**：`ohos.permission.FILE_ACCESS_MANAGER`

---

## 4.3 错误码参考

### SandboxManagerErrCode

| 错误码 | 值 | 说明 |
|-------|-----|------|
| `SANDBOX_MANAGER_OK` | 0 | 成功 |
| `PERMISSION_DENIED` | 1 | 权限被拒绝 |
| `INVALID_PARAMTER` | 2 | 参数错误 |
| `SANDBOX_MANAGER_SERVICE_NOT_EXIST` | 3 | 服务不存在 |
| `SANDBOX_MANAGER_SERVICE_PARCEL_ERR` | 4 | Parcel 序列化错误 |
| `SANDBOX_MANAGER_SERVICE_REMOTE_ERR` | 5 | 远程调用错误 |
| `SANDBOX_MANAGER_DB_ERR` | 6 | 数据库错误 |
| `SANDBOX_MANAGER_MAC_NOT_INIT` | 8 | MAC 未初始化 |
| `SANDBOX_MANAGER_MAC_IOCTL_ERR` | 9 | MAC ioctl 错误 |
| `SANDBOX_MANAGER_DENY_ERR` | 10 | 拒绝策略错误 |

### SandboxRetType (操作结果)

| 值 | 宏定义 | 说明 |
|---|--------|------|
| 0 | `OPERATE_SUCCESSFULLY` | 操作成功 |
| 1 | `FORBIDDEN_TO_BE_PERSISTED` | 禁止持久化 |
| 2 | `INVALID_MODE` | 无效模式 |
| 3 | `INVALID_PATH` | 无效路径 |
| 4 | `POLICY_HAS_NOT_BEEN_PERSISTED` | 策略未持久化 |
| 5 | `POLICY_MAC_FAIL` | MAC 操作失败 |
| 6 | `FORBIDDEN_TO_BE_PERSISTED_BY_FLAG` | 被标志禁止 |

**证据来源**：
- API 签名：`interfaces/inner_api/sandbox_manager/include/sandbox_manager_kit.h`
- 错误码：`interfaces/inner_api/sandbox_manager/include/sandbox_manager_err_code.h`
- 策略结果：`interfaces/inner_api/sandbox_manager/include/policy_info.h:46-54`

---

## 4.4 数据结构参考

### PolicyInfo

```cpp
struct PolicyInfo final {
    std::string path;           // 策略路径
    uint64_t mode;              // 操作模式 (READ_MODE, WRITE_MODE, etc.)
    PolicyType type;            // 策略类型 (SELF_PATH, AUTHORIZATION_PATH, OTHERS_PATH)
};
```

### SetInfo

```cpp
struct SetInfo final {
    std::string bundleName;     // 包名
    uint64_t timestamp;         // 时间戳
    SetInfo() : bundleName(""), timestamp(0) {}
};
```

### PolicyType

| 枚举值 | 说明 |
|-------|------|
| `UNKNOWN` | 未知类型 |
| `SELF_PATH` | 自有路径 |
| `AUTHORIZATION_PATH` | 授权路径 |
| `OTHERS_PATH` | 其他路径 |

### OperateMode

| 枚举值 | 值 | 描述 |
|-------|-----|------|
| `READ_MODE` | 0x01 | 读权限 |
| `WRITE_MODE` | 0x02 | 写权限 |
| `CREATE_MODE` | 0x04 | 创建权限 |
| `DELETE_MODE` | 0x08 | 删除权限 |
| `RENAME_MODE` | 0x10 | 重命名权限 |
| `DENY_READ_MODE` | 0x20 | 拒绝读 |
| `DENY_WRITE_MODE` | 0x40 | 拒绝写 |

---

*文档更新时间: 2025-02-07*
