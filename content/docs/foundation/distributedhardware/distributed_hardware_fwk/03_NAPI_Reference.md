# N-API 接口参考

本文档描述分布式硬件管理框架对外提供的 N-API（Native API）接口，供 JavaScript/TypeScript 应用调用。

> **适用范围**: 需要使用 JS API 开发分布式硬件应用的开发者

---

## 概述

**模块名**: `@ohos/distributedHardware.hardwareManager`
**模块路径**: `interfaces/kits/napi/`
**实现文件**: `native_distributedhardwarefwk_js.cpp`

**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:455`
```cpp
.nm_modname = "distributedHardware.hardwareManager",
```

---

## API 清单表

| JS API 名称 | C/C++ 实现 | 同步/异步 | 说明 |
|-------------|------------|----------|------|
| `pauseDistributedHardware` | `PauseDistributedHardware` | Promise/Callback | 暂停分布式硬件 |
| `resumeDistributedHardware` | `ResumeDistributedHardware` | Promise/Callback | 恢复分布式硬件 |
| `stopDistributedHardware` | `StopDistributedHardware` | Promise/Callback | 停止分布式硬件 |

---

## 常量定义

### DistributedHardwareType (硬件类型)

| JS 常量 | C/C++ 值 | 说明 |
|---------|----------|------|
| `DistributedHardwareType.ALL` | `ALL` (0) | 所有硬件类型 |
| `DistributedHardwareType.CAMERA` | `CAMERA` (1) | 分布式相机 |
| `DistributedHardwareType.SCREEN` | `SCREEN` (2) | 分布式屏幕 |
| `DistributedHardwareType.MODEM_MIC` | `MODEM_MIC` (3) | Modem 麦克风 |
| `DistributedHardwareType.MODEM_SPEAKER` | `MODEM_SPEAKER` (4) | Modem 扬声器 |
| `DistributedHardwareType.MIC` | `MIC` (5) | 本地麦克风 |
| `DistributedHardwareType.SPEAKER` | `SPEAKER` (6) | 本地扬声器 |

**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:403-411`
```cpp
napi_property_descriptor desc[] = {
    DECLARE_NAPI_STATIC_PROPERTY("ALL", all),
    DECLARE_NAPI_STATIC_PROPERTY("CAMERA", camera),
    DECLARE_NAPI_STATIC_PROPERTY("SCREEN", screen),
    DECLARE_NAPI_STATIC_PROPERTY("MODEM_MIC", modemMic),
    DECLARE_NAPI_STATIC_PROPERTY("MODEM_SPEAKER", modemSpk),
    DECLARE_NAPI_STATIC_PROPERTY("MIC", mic),
    DECLARE_NAPI_STATIC_PROPERTY("SPEAKER", speaker)
};
```

### DistributedHardwareErrorCode (错误码)

| JS 常量 | C/C++ 值 | 说明 |
|---------|----------|------|
| `DistributedHardwareErrorCode.ERR_CODE_DISTRIBUTED_HARDWARE_NOT_STARTED` | `24200101` | 分布式硬件未启动 |
| `DistributedHardwareErrorCode.ERR_CODE_DEVICE_NOT_CONNECTED` | `24200102` | 源设备未连接 |

**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:426-428`
```cpp
DECLARE_NAPI_STATIC_PROPERTY("ERR_CODE_DISTRIBUTED_HARDWARE_NOT_STARTED", dhNotStart),
DECLARE_NAPI_STATIC_PROPERTY("ERR_CODE_DEVICE_NOT_CONNECTED", deviceNotConnect),
```

---

## API 详细说明

### pauseDistributedHardware

暂停指定设备的分布式硬件。

**函数签名**:
```typescript
function pauseDistributedHardware(description: DistributedHardwareDescription, callback?: AsyncCallback<void>): Promise<void>;
```

**参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `description` | `DistributedHardwareDescription` | 是 | 硬件描述对象 |
| `callback` | `AsyncCallback<void>` | 否 | 回调函数（可选） |

**DistributedHardwareDescription**:

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | `DistributedHardwareType` | 是 | 硬件类型 |
| `srcNetworkId` | `string` | 是 | 源设备网络 ID |

**返回值**:
- `Promise<void>`: 成功时 resolve，失败时 reject
- `AsyncCallback<void>`: 通过 callback 返回结果

**C/C++ 实现**:
**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:212-263`
```cpp
napi_value DistributedHardwareManager::PauseDistributedHardware(napi_env env, napi_callback_info info)
{
    // 1. 解析参数
    // 2. 验证系统应用权限
    // 3. 验证 ACCESS_DISTRIBUTED_HARDWARE 权限
    // 4. 调用 DistributedHardwareFwkKit::PauseDistributedHardware()
}
```

---

### resumeDistributedHardware

恢复指定设备的分布式硬件。

**函数签名**:
```typescript
function resumeDistributedHardware(description: DistributedHardwareDescription, callback?: AsyncCallback<void>): Promise<void>;
```

**参数**: 同 `pauseDistributedHardware`

**返回值**: 同 `pauseDistributedHardware`

**C/C++ 实现**:
**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:265-316`
```cpp
napi_value DistributedHardwareManager::ResumeDistributedHardware(napi_env env, napi_callback_info info)
```

---

### stopDistributedHardware

停止指定设备的分布式硬件。

**函数签名**:
```typescript
function stopDistributedHardware(description: DistributedHardwareDescription, callback?: AsyncCallback<void>): Promise<void>;
```

**参数**: 同 `pauseDistributedHardware`

**返回值**: 同 `pauseDistributedHardware`

**C/C++ 实现**:
**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:318-369`
```cpp
napi_value DistributedHardwareManager::StopDistributedHardware(napi_env env, napi_callback_info info)
```

---

## 错误码

### 业务错误码

| 错误码 | 名称 | 说明 |
|--------|------|------|
| 201 | `ERR_NO_PERMISSION` | 权限校验失败 |
| 202 | `ERR_NOT_SYSTEM_APP` | 调用者非系统应用 |
| 401 | `ERR_INVALID_PARAMS` | 输入参数错误 |
| 24200101 | `ERR_CODE_DH_NOT_START` | 分布式硬件未启动 |
| 24200102 | `ERR_CODE_DEVICE_NOT_CONNECT` | 源设备未连接 |

**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:70-81`
```cpp
enum DHBussinessErrorCode {
    // Permission verify failed.
    ERR_NO_PERMISSION = 201,
    // The caller is not a system application.
    ERR_NOT_SYSTEM_APP = 202,
    // Input parameter error.
    ERR_INVALID_PARAMS = 401,
    // The distributed hardware is not started.
    ERR_CODE_DH_NOT_START = 24200101,
    // The source device is not connected.
    ERR_CODE_DEVICE_NOT_CONNECT = 24200102,
};
```

---

## 权限要求

### 必要权限

调用 N-API 需要以下权限：

| 权限名 | 说明 | 必需 |
|--------|------|------|
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 访问分布式硬件 | ✅ 必需 |
| 系统应用身份 | 调用者必须是系统应用 | ✅ 必需 |

**证据**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:149-182`
```cpp
bool DistributedHardwareManager::IsSystemApp()
{
    uint64_t tokenId = OHOS::IPCSkeleton::GetSelfTokenID();
    return OHOS::Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(tokenId);
}

bool DistributedHardwareManager::HasAccessDHPermission()
{
    OHOS::Security::AccessToken::AccessTokenID callerToken = OHOS::IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE";
    int32_t result = OHOS::Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == OHOS::Security::AccessToken::PERMISSION_GRANTED);
}

bool DistributedHardwareManager::Verify(napi_env env, int32_t type)
{
    if (!IsSystemApp()) {
        CreateBusinessErr(env, ERR_NOT_SYSTEM_APP);
        return false;
    }
    if (!HasAccessDHPermission()) {
        CreateBusinessErr(env, ERR_NO_PERMISSION);
        return false;
    }
    // ...
}
```

---

## 使用示例

### Promise 模式

```typescript
import { distributedHardwareManager } from '@ohos/distributedHardware.hardwareManager';

let description = {
  type: distributedHardwareManager.DistributedHardwareType.CAMERA,
  srcNetworkId: 'ABC123DEF456'
};

try {
  await distributedHardwareManager.pauseDistributedHardware(description);
  console.info('pauseDistributedHardware success');
} catch (error) {
  console.error('pauseDistributedHardware failed:', error.code, error.message);
}
```

### Callback 模式

```typescript
import { distributedHardwareManager } from '@ohos/distributedHardware.hardwareManager';

let description = {
  type: distributedHardwareManager.DistributedHardwareType.CAMERA,
  srcNetworkId: 'ABC123DEF456'
};

distributedHardwareManager.pauseDistributedHardware(description, (err) => {
  if (err) {
    console.error('pauseDistributedHardware failed:', err.code, err.message);
    return;
  }
  console.info('pauseDistributedHardware success');
});
```

---

## 调用链

```
JavaScript/TypeScript
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│  native_distributedhardwarefwk_js.cpp                   │
│  - 参数解析 (JsObjectToString, JsObjectToInt)           │
│  - 权限验证 (IsSystemApp, HasAccessDHPermission)       │
│  - 参数校验 (CheckArgsType, IsSupportType)              │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  DistributedHardwareFwkKit                             │
│  - PauseDistributedHardware()                           │
│  - ResumeDistributedHardware()                          │
│  - StopDistributedHardware()                            │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  DistributedHardwareProxy (IPC)                         │
│  - SendRequest()                                        │
│  - MessageParcel 序列化                                 │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  DistributedHardwareService (SA)                        │
│  - OnRemoteRequest()                                    │
│  - Task Dispatcher                                      │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  ComponentManager                                       │
│  - EnableSink / DisableSink                             │
└─────────────────────────────────────────────────────────┘
```

---

## 后续文档

- 内部 API 参考 → [04_Inner_API.md](04_Inner_API.md)
- 架构说明 → [02_Architecture.md](02_Architecture.md)
- 安全评审 → [07_Security_Review.md](07_Security_Review.md)
