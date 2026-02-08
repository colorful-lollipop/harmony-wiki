# N-API 接口文档

## 1. 模块概述

user_auth_framework 提供多层次的 N-API 接口，支持 JavaScript/ArkTS 应用调用认证能力。

**API 模块**:

| 模块 | 命名空间 | 路径 |
|------|----------|------|
| UserAuth | `userIAM.userAuth` | `frameworks/js/napi/user_auth/` |
| UserAccessCtrl | `userIAM.userAccessCtrl` | `frameworks/js/napi/user_access_ctrl/` |
| UserAuthIcon | `userIAM.userAuthIcon` | `frameworks/js/napi/user_auth_icon/` |
| UserAuthExtension | `app.ability.UserAuthExtensionAbility` | `frameworks/js/napi/user_auth_extension/` |

> **证据**: `frameworks/js/napi/*/src/*_entry.cpp` N-API 注册点

---

## 2. userIAM.userAuth 模块

### 2.1 模块信息

| 属性 | 值 |
|------|-----|
| 注册文件 | `frameworks/js/napi/user_auth/src/user_auth_entry.cpp` |
| 命名空间 | `OHOS::UserIam::UserAuth` |
| 导出方式 | `napi_module_register()` |

### 2.2 API 清单

#### 2.2.1 模块级函数

| JS API | C++ 实现 | 说明 |
|--------|----------|------|
| `getAuthenticator()` | `ConstructorForApi6` | 获取认证器实例 |
| `getAvailableStatus(authType, authTrustLevel)` | `GetAvailableStatusV9` | 获取认证可用状态 |
| `getAuthInstance(authType, authTrustLevel)` | `GetAuthInstanceV9` | 获取认证实例 (V9) |
| `getUserAuthInstance(authType, authTrustLevel)` | `GetUserAuthInstanceV10` | 获取认证实例 (V10) |
| `getUserAuthWidgetMgr()` | `GetUserAuthWidgetMgrV10` | 获取 Widget 管理器 |
| `getEnrolledState(authType)` | `GetEnrolledState` | 获取认证类型注册状态 |
| `sendNotice(notice)` | `SendNotice` | 发送通知 |
| `queryReusableAuthResult(param)` | `QueryReusableAuthResult` | 查询可复用认证结果 |
| `getAuthLockState(userId)` | `GetAuthLockState` | 获取认证锁定状态 |

> **证据**: `user_auth_entry.cpp` 模块导出函数

#### 2.2.2 UserAuth 实例方法

| JS API | C++ 实现 | 说明 |
|--------|----------|------|
| `getVersion()` | `GetVersion` | 获取版本号 |
| `getAvailableStatus(authType, atl)` | `GetAvailableStatus` | 获取认证可用状态 |
| `auth(challenge, authType, atl, callback)` | `Auth` | 执行认证 |
| `cancelAuth()` | `CancelAuth` | 取消认证 |

#### 2.2.3 AuthInstanceV9 方法

| JS API | C++ 实现 | 说明 |
|--------|----------|------|
| `on(type, callback)` | `On` | 订阅事件 |
| `off(type)` | `Off` | 取消订阅 |
| `start()` | `Start` | 开始认证 |
| `cancel()` | `Cancel` | 取消认证 |

#### 2.2.4 UserAuthInstanceV10 方法

| JS API | C++ 实现 | 说明 |
|--------|----------|------|
| `on(type, callback)` | `OnV10` | 订阅事件 |
| `off(type)` | `OffV10` | 取消订阅 |
| `start()` | `StartV10` | 开始认证 |
| `cancel()` | `CancelV10` | 取消认证 |

#### 2.2.5 UserAuthWidgetMgr 方法

| JS API | C++ 实现 | 说明 |
|--------|----------|------|
| `on(type, callback)` | `WidgetOn` | 订阅 Widget 事件 |
| `off(type)` | `WidgetOff` | 取消订阅 |

---

### 2.3 枚举定义

#### AuthTrustLevel (认证信任等级)

```typescript
enum AuthTrustLevel {
    ATL1 = 1,  // 最低信任等级
    ATL2 = 2,  // 中等信任等级
    ATL3 = 3,  // 较高信任等级
    ATL4 = 4   // 最高信任等级
}
```

#### UserAuthType (认证类型)

```typescript
enum UserAuthType {
    PIN = 1,
    FACE = 2,
    FINGERPRINT = 4,
    PRIVATE_PIN = 8
}
```

#### ResultCode (结果码)

```typescript
enum ResultCode {
    SUCCESS = 0,
    FAIL = 1,
    GENERAL_ERROR = 2,
    CANCELED = 3,
    TIMEOUT = 4,
    ...
}
```

#### UserAuthResultCode

```typescript
enum UserAuthResultCode {
    SUCCESS = 0,
    FAIL = 1,
    GENERAL_ERROR = 2,
    CANCELED = 3,
    TIMEOUT = 4,
    ...
}
```

#### AuthenticationResult (认证结果)

```typescript
enum AuthenticationResult {
    NO_SUPPORT = 0,
    SUCCESS = 1,
    COMPARE_FAILURE = 2,
    ...
}
```

#### FingerprintTips (指纹提示)

```typescript
enum FingerprintTips {
    FINGERPRINT_AUTH_TIP_GOOD = 0,
    FINGERPRINT_AUTH_TIP_DIRTY = 1,
    FINGERPRINT_AUTH_TIP_NO_TOUCH = 2,
    ...
}
```

#### FaceTips (人脸提示)

```typescript
enum FaceTips {
    FACE_AUTH_TIP_TOO_BRIGHT = 0,
    FACE_AUTH_TIP_TOO_DARK = 1,
    ...
}
```

#### NoticeType (通知类型)

```typescript
enum NoticeType {
    WIDGET_NOTICE = 0
}
```

#### WindowModeType (窗口模式)

```typescript
enum WindowModeType {
    DIALOG_BOX = 0,
    FULLSCREEN = 1,
    NONE_INTERRUPTION_DIALOG_BOX = 2
}
```

#### ReuseMode (复用模式)

```typescript
enum ReuseMode {
    AUTH_TYPE_RELEVANT = 0,
    AUTH_TYPE_IRRELEVANT = 1
}
```

---

### 2.4 常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_ALLOWABLE_REUSE_DURATION` | 数值 | 最大可复用时长 |
| `PERMANENT_LOCKOUT_DURATION` | 数值 | 永久锁定时长 |

---

### 2.5 使用示例

#### 获取认证实例并认证

```typescript
import userIAM from '@ohos.userIAM.userAuth';

// 获取认证实例
let authInstance = userIAM.getAuthInstance(userIAM.UserAuthType.FACE, userIAM.AuthTrustLevel.ATL2);

// 订阅结果事件
authInstance.on('result', (code, extra) => {
    console.log(`Auth result: ${code}`);
});

// 开始认证
authInstance.start();
```

#### 直接认证

```typescript
let authenticator = userIAM.getAuthenticator();

authenticator.auth(0n, userIAM.UserAuthType.FACE, userIAM.AuthTrustLevel.ATL2, {
    onResult: (code, result) => {
        console.log(`Auth result: ${code}`);
    },
    onAcquireInfo: (module, acquireInfo) => {
        console.log(`Acquire info: ${module}, ${acquireInfo}`);
    }
});
```

---

## 3. userIAM.userAccessCtrl 模块

### 3.1 模块信息

| 属性 | 值 |
|------|-----|
| 注册文件 | `frameworks/js/napi/user_access_ctrl/src/user_access_ctrl_entry.cpp` |
| 命名空间 | `OHOS::UserIam::UserAccessCtrl` |

### 3.2 API 清单

| JS API | C++ 实现 | 说明 |
|--------|----------|------|
| `verifyAuthToken(token, authType, userId)` | `VerifyAuthToken` | 验证认证令牌 |

### 3.3 枚举定义

#### AuthTokenType

```typescript
enum AuthTokenType {
    TOKEN_TYPE_LOCAL_AUTH = 0,    // 本地认证令牌
    TOKEN_TYPE_LOCAL_RESIGN = 1,  // 本地重新签名令牌
    TOKEN_TYPE_LOCAL_COAUTH = 2   // 本地协同认证令牌
}
```

---

## 4. userIAM.userAuthIcon 模块

**说明**: 资源模块，导出认证图标组件的 ArkTS/ABC 字节码。

**文件**: `frameworks/js/napi/user_auth_icon/user_auth_icon.cpp`

---

## 5. app.ability.UserAuthExtensionAbility

**说明**: Extension Ability 模块，用于自定义认证界面。

**文件**: `frameworks/js/napi/user_auth_extension/user_auth_extension/user_auth_extension_ability_module.cpp`

---

## 6. API 调用链

### 6.1 认证调用链

```
JS API (user_auth_entry.cpp)
    │
    ▼
UserAuthImpl::Auth() (user_auth_impl.cpp)
    │
    ▼
UserAuthClientImpl (frameworks/native/client/src/)
    │
    ▼
IpcClientUtils::GetRemoteObject(901)
    │
    ▼
UserAuthService (services/ipc/src/user_auth_service.cpp)
    │
    ▼
Context (services/context/src/)
    │
    ▼
Executor Framework (frameworks/native/executors/)
    │
    ▼
HDI (drivers_interface_user_auth)
```

---

## 7. 错误码参考

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | SUCCESS | 成功 |
| 1 | FAIL | 失败 |
| 2 | GENERAL_ERROR | 一般错误 |
| 3 | CANCELED | 取消 |
| 4 | TIMEOUT | 超时 |

> **完整错误码**: 见 `user_auth_entry.cpp` 中的枚举定义

---

## 8. 权限要求

| API | 权限 | 说明 |
|-----|------|------|
| auth() | 无 (系统 API) | 需要系统能力 |
| getAuthInstance() | 无 | 需配置使用 |
| verifyAuthToken() | 无 | 系统内部使用 |

---

## 9. 相关文档

- [架构说明](01_Architecture.md)
- [Inner API](03_InnerAPI.md)
- [安全评审](05_Security.md)
