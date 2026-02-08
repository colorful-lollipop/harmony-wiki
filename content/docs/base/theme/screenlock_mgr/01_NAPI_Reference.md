# 01_NAPI_Reference - N-API 接口参考

## 1. 概述

本文档描述 `screenlock_mgr` 子系统对外暴露的 N-API 接口，供第三方应用和系统应用调用。

> **证据来源**: `frameworks/js/napi/src/napi_screenlock_ability.cpp:67-88`

---

## 2. API 清单

### 2.1 查询类 API

| JS API | 命名空间 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `isScreenLocked()` | ohos.screenLock | 同步 | 查询屏幕是否锁定 |
| `isLocked()` | ohos.screenLock | 同步 | 查询锁定状态 |
| `isSecureMode()` | ohos.screenLock | 同步 | 查询是否处于安全模式 |
| `isScreenLockDisabled(userId)` | ohos.screenLock | 同步 | 查询指定用户的锁屏是否禁用 |
| `getScreenLockAuthState(userId)` | ohos.screenLock | 同步 | 获取用户认证状态 |
| `getStrongAuth(userId)` | ohos.screenLock | 同步 | 获取强认证状态 |
| `isDeviceLocked(userId)` | ohos.screenLock | 同步 | 查询设备是否锁定 |

### 2.2 操作类 API

| JS API | 命名空间 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `lock()` | ohos.screenLock | 异步 | 锁定屏幕 (Promise/Callback) |
| `unlockScreen()` | ohos.screenLock | 异步 | 解锁屏幕 (Promise/Callback) |
| `unlock()` | ohos.screenLock | 异步 | 解锁 (内部使用) |
| `setScreenLockDisabled(disable, userId)` | ohos.screenLock | 同步 | 设置锁屏禁用状态 |
| `setScreenLockAuthState(authState, userId, authToken)` | ohos.screenLock | 同步 | 设置认证状态 |
| `requestStrongAuth(reasonFlag, userId)` | ohos.screenLock | 同步 | 请求强认证 |

### 2.3 事件类 API

| JS API | 命名空间 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `onSystemEvent(callback)` | ohos.screenLock | 异步 | 监听系统事件 |
| `sendScreenLockEvent(event, param)` | ohos.screenLock | 同步 | 发送锁屏事件 |

---

## 3. 详细接口说明

### 3.1 isScreenLocked

查询屏幕是否锁定。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:70
DECLARE_NAPI_FUNCTION("isScreenLocked", OHOS::ScreenLock::NAPI_IsScreenLocked)
```

**参数**: 无

**返回值**: `boolean` - true 表示屏幕已锁定，false 表示未锁定

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 13200002 | 服务异常 |

---

### 3.2 lock

锁定屏幕。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:72
DECLARE_NAPI_FUNCTION("lock", OHOS::ScreenLock::NAPI_Lock)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| callback | Function | 否 | 回调函数 (可选) |

**返回值**: `Promise<void>` 或 void (当传入 callback 时)

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 无权限 |
| 401 | 参数错误 |
| 13200002 | 服务异常 |

**使用示例**:
```javascript
// Promise 方式
screenLock.lock().then(() => {
    console.log('Lock success');
}).catch((err) => {
    console.error('Lock failed:', err);
});

// Callback 方式
screenLock.lock((err) => {
    if (err) {
        console.error('Lock failed:', err);
    } else {
        console.log('Lock success');
    }
});
```

---

### 3.3 unlockScreen

解锁屏幕。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:73
DECLARE_NAPI_FUNCTION("unlockScreen", OHOS::ScreenLock::NAPI_UnlockScreen)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| callback | Function | 否 | 回调函数 (可选) |

**返回值**: `Promise<void>` 或 void (当传入 callback 时)

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 无权限 |
| 401 | 参数错误 |
| 13200001 | 用户取消解锁 |
| 13200002 | 服务异常 |
| 13200003 | 非法使用 |

**使用示例**:
```javascript
// Promise 方式
screenLock.unlockScreen().then(() => {
    console.log('Unlock success');
}).catch((err) => {
    console.error('Unlock failed:', err);
});
```

---

### 3.4 onSystemEvent

监听系统事件。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:76
DECLARE_NAPI_FUNCTION("onSystemEvent", NAPI_OnSystemEvent)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| callback | Function | 是 | 事件回调函数 |

**回调参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| err | BusinessError | 错误信息 |
| event | Object | 事件对象 |

**事件类型**:
| 事件类型 | 说明 |
|----------|------|
| `unlockScreen` | 解锁屏幕事件 |
| `lockScreen` | 锁定屏幕事件 |
| `screenDrawDone` | 屏幕绘制完成 |
| `unlockScreenResult` | 解锁结果 |

**使用示例**:
```javascript
screenLock.onSystemEvent((err, event) => {
    if (err) {
        console.error('Event error:', err);
        return;
    }
    console.log('Received event:', event);
});
```

---

### 3.5 isScreenLockDisabled

查询指定用户的锁屏是否禁用。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:78
DECLARE_NAPI_FUNCTION("isScreenLockDisabled", OHOS::ScreenLock::NAPI_IsScreenLockDisabled)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | number | 是 | 用户 ID |

**返回值**: `boolean` - true 表示锁屏已禁用，false 表示启用

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 13200004 | 用户 ID 无效 |
| 13200002 | 服务异常 |

---

### 3.6 setScreenLockDisabled

设置指定用户的锁屏禁用状态。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:79
DECLARE_NAPI_FUNCTION("setScreenLockDisabled", OHOS::ScreenLock::NAPI_SetScreenLockDisabled)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| disable | boolean | 是 | 是否禁用锁屏 |
| userId | number | 是 | 用户 ID |

**返回值**: void

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 无权限 (需要系统应用) |
| 202 | 非系统应用 |
| 401 | 参数错误 |
| 13200004 | 用户 ID 无效 |
| 13200002 | 服务异常 |

---

### 3.7 setScreenLockAuthState

设置屏幕锁定认证状态。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:80
DECLARE_NAPI_FUNCTION("setScreenLockAuthState", OHOS::ScreenLock::NAPI_SetScreenLockAuthState)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| authState | number | 是 | 认证状态 |
| userId | number | 是 | 用户 ID |
| authToken | string | 是 | 认证令牌 |

**返回值**: void

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 无权限 |
| 401 | 参数错误 |
| 13200002 | 服务异常 |

---

### 3.8 getScreenLockAuthState

获取屏幕锁定认证状态。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:81
DECLARE_NAPI_FUNCTION("getScreenLockAuthState", OHOS::ScreenLock::NAPI_GetScreenLockAuthState)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | number | 是 | 用户 ID |

**返回值**: `number` - 认证状态

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 13200004 | 用户 ID 无效 |
| 13200002 | 服务异常 |

---

### 3.9 requestStrongAuth

请求强认证。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:82
DECLARE_NAPI_FUNCTION("requestStrongAuth", OHOS::ScreenLock::NAPI_RequestStrongAuth)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| reasonFlag | number | 是 | 强认证原因标志 |
| userId | number | 是 | 用户 ID |

**返回值**: void

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 无权限 |
| 401 | 参数错误 |
| 13200002 | 服务异常 |

---

### 3.10 getStrongAuth

获取强认证状态。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:83
DECLARE_NAPI_FUNCTION("getStrongAuth", OHOS::ScreenLock::NAPI_GetStrongAuth)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | number | 是 | 用户 ID |

**返回值**: `number` - 强认证状态

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 13200002 | 服务异常 |

---

### 3.11 isDeviceLocked

查询指定用户的设备是否锁定。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:84
DECLARE_NAPI_FUNCTION("isDeviceLocked", OHOS::ScreenLock::NAPI_IsDeviceLocked)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | number | 是 | 用户 ID |

**返回值**: `boolean` - true 表示设备已锁定，false 表示未锁定

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 13200004 | 用户 ID 无效 |
| 13200002 | 服务异常 |

---

### 3.12 sendScreenLockEvent

发送屏幕锁定事件。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:77
DECLARE_NAPI_FUNCTION("sendScreenLockEvent", OHOS::ScreenLock::NAPI_ScreenLockSendEvent)
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| event | string | 是 | 事件类型 |
| param | number | 是 | 事件参数 |

**返回值**: void

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 13200002 | 服务异常 |

---

### 3.13 isSecureMode

查询是否处于安全模式。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:75
DECLARE_NAPI_FUNCTION("isSecureMode", OHOS::ScreenLock::NAPI_IsSecureMode)
```

**参数**: 无

**返回值**: `boolean` - true 表示安全模式，false 表示非安全模式

---

### 3.14 isLocked

查询锁定状态。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:71
DECLARE_NAPI_FUNCTION("isLocked", OHOS::ScreenLock::NAPI_IsLocked)
```

**参数**: 无

**返回值**: `boolean` - true 表示已锁定，false 表示未锁定

---

### 3.15 unlock

内部解锁接口。

**函数签名**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:74
DECLARE_NAPI_FUNCTION("unlock", OHOS::ScreenLock::NAPI_Unlock)
```

**参数**: 无 (内部使用)

---

## 4. 错误码定义

错误码定义在 `interfaces/inner_api/include/screenlock_common.h:90-98`

```cpp
enum JsErrorCode : uint32_t {
    ERR_NO_PERMISSION = 201,           // 无权限
    ERR_NOT_SYSTEM_APP = 202,          // 非系统应用
    ERR_INVALID_PARAMS = 401,          // 参数错误
    ERR_CANCEL_UNLOCK = 13200001,      // 用户取消解锁
    ERR_SERVICE_ABNORMAL = 13200002,   // 服务异常
    ERR_ILLEGAL_USE = 13200003,        // 非法使用
    ERR_USER_ID_INVALID = 13200004,    // 用户 ID 无效
};
```

---

## 5. 权限要求

| API | 所需权限 | 说明 |
|-----|----------|------|
| `setScreenLockDisabled` | 系统应用 | 需要 ohos.permission.ACCESS_SCREEN_LOCK |
| `setScreenLockAuthState` | ohos.permission.ACCESS_SCREEN_LOCK | 设置认证状态 |
| `getScreenLockAuthState` | ohos.permission.ACCESS_SCREEN_LOCK | 获取认证状态 |
| 其他查询类 API | 无特殊权限 | 一般应用可用 |

---

## 6. 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [02_Architecture](02_Architecture.md) | 系统架构 |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 调用链分析 |
