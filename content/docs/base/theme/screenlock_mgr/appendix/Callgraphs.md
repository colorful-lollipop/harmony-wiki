# appendix/Callgraphs - 关键调用链

## 1. 概述

本文档记录 `screenlock_mgr` 子系统中的关键调用链，用于理解代码执行流程和调试定位。

---

## 2. 屏幕解锁流程

### 2.1 JS API 调用链

```
用户调用
    │
    ▼
JS Runtime (ACE)
    │
    │ napi_call_function
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  napi_screenlock_ability.cpp                                   │
│  NAPI_UnlockScreen(env, info)                                   │
│      │                                                         │
│      │ 1. 参数解析 (argc, argv)                                 │
│      │ 2. 参数类型检查                                          │
│      │ 3. 创建 EventListener                                   │
│      │ 4. 调用 AsyncCallFunc                                   │
│      ▼                                                         │
│  AsyncCallFunc(env, listener, "unLockScreen")                   │
│      │                                                         │
│      │ napi_queue_async_work_with_qos                           │
│      ▼                                                         │
│  execute callback (工作队列线程)                                  │
│      │                                                         │
│      │ ScreenLockManager::GetInstance()->Unlock()               │
│      ▼                                                         │
│  screenlock_manager.cpp                                         │
│  ScreenLockManager::Unlock()                                    │
│      │                                                         │
│      │ IPC 调用 (Binder)                                        │
│      ▼                                                         │
│  screenlock_manager_proxy.cpp                                   │
│  ScreenLockManagerProxy::Unlock()                               │
│      │                                                         │
│      │ Parcel序列化                                             │
│      │ Remote()->SendRequest()                                  │
│      ▼                                                         │
│  Binder Driver                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Foundation 进程                                                │
│                                                                 │
│  IPC Thread                                                     │
│      │                                                         │
│      │ Binder 解码                                              │
│      ▼                                                         │
│  screenlock_manager_stub.cpp                                    │
│  ScreenLockManagerStub::OnRemoteRequest()                       │
│      │                                                         │
│      │ case UNLOCK:                                            │
│      ▼                                                         │
│  ScreenLockManagerStub::Unlock()                                │
│      │                                                         │
│      │ 权限检查                                                 │
│      │ 调用服务实现                                             │
│      ▼                                                         │
│  ScreenLockSystemAbility::Unlock()                              │
│      │                                                         │
│      │ 1. 验证强认证状态                                        │
│      │ 2. 执行解锁操作                                          │
│      │ 3. 更新状态                                              │
│      │ 4. 发布事件                                              │
│      ▼                                                         │
│  StrongAuthManager                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**关键文件**:
- `frameworks/js/napi/src/napi_screenlock_ability.cpp` (入口: 310-349 行)
- `frameworks/native/src/screenlock_manager.cpp` (代理)
- `services/src/screenlock_manager_stub.cpp` (IPC 存根)
- `services/src/screenlock_system_ability.cpp` (SA 实现)

---

## 3. 屏幕锁定流程

### 3.1 JS API 调用链

```
用户调用 screenLock.lock()
    │
    ▼
NAPI_Lock(env, info)
    │
    │ napi_get_cb_info
    │ 参数解析
    │
    ├───> callback 模式: 创建 callback 引用
    │
    └───> Promise 模式: 创建 deferred
    │
    ▼
AsyncCallFunc(env, listener, "lock")
    │
    │ napi_create_async_work
    │ napi_queue_async_work_with_qos
    │
    ▼
工作队列线程
    │
    │ execute callback
    │
    ▼
ScreenLockManager::GetInstance()->Lock(callback)
    │
    │ IPC 调用
    │
    ▼
ScreenLockSystemAbility::Lock()
    │
    │ 1. 设置锁屏标志
    │ 2. 触发锁屏动画
    │ 3. 发布 LOCK 事件
    │
    ▼
CommonEventManager::PublishEvent()
```

**关键文件**:
- `frameworks/js/napi/src/napi_screenlock_ability.cpp` (入口: 263-308 行)
- `services/src/screenlock_system_ability.cpp` (实现)

---

## 4. 系统事件回调流程

### 4.1 onSystemEvent 注册

```
用户调用 screenLock.onSystemEvent(callback)
    │
    ▼
NAPI_OnSystemEvent(env, info)
    │
    │ 1. 解析 callback 参数
    │ 2. 创建 napi_ref 引用
    │ 3. 保存到内部监听器列表
    │
    ▼
ScreenLockSystemAbility::OnSystemEvent(listener)
    │
    │ 1. 注册内部监听器
    │ 2. 返回成功
    │
    ▼
InnerListenerManager::RegisterInnerListener()
```

### 4.2 事件触发

```
系统事件发生 (屏幕开关、用户切换等)
    │
    ▼
ScreenLockSystemAbility (收到事件)
    │
    │ 1. 识别事件类型
    │ 2. 查找对应监听器
    │
    ▼
InnerListenerManager::NotifyListeners()
    │
    │ 遍历监听器列表
    │
    ├───> JS Callback:
    │     │
    │     │ napi_call_function
    │     ▼
    │   JS Runtime
    │
    └───> Native Listener:
          │
          │ IRemoteBroker 调用
          ▼
        监听者处理
```

**事件类型定义** (`interfaces/inner_api/include/screenlock_common.h`):
```cpp
inline const std::string BEGIN_WAKEUP = "beginWakeUp";
inline const std::string END_WAKEUP = "endWakeUp";
inline const std::string BEGIN_SCREEN_ON = "beginScreenOn";
inline const std::string END_SCREEN_ON = "endScreenOn";
inline const std::string BEGIN_SLEEP = "beginSleep";
inline const std::string END_SLEEP = "endSleep";
inline const std::string BEGIN_SCREEN_OFF = "beginScreenOff";
inline const std::string END_SCREEN_OFF = "endScreenOff";
inline const std::string STRONG_AUTH_CHANGED = "strongAuthChanged";
inline const std::string CHANGE_USER = "changeUser";
inline const std::string SCREENLOCK_ENABLED = "screenlockEnabled";
inline const std::string EXIT_ANIMATION = "beginExitAnimation";
```

---

## 5. 状态查询流程

### 5.1 isScreenLocked 调用

```
JS: screenLock.isScreenLocked()
    │
    ▼
NAPI_IsScreenLocked(env, info)
    │
    │ 1. 获取 ScreenLockManager 单例
    │ 2. 调用 IsScreenLocked()
    │ 3. 返回 boolean
    │
    ▼
ScreenLockManager::IsScreenLocked()
    │
    │ IPC: SendRequest(IS_SCREEN_LOCKED)
    │
    ▼
ScreenLockSystemAbility::IsScreenLocked()
    │
    │ 读取内部状态变量
    │
    ▼
返回状态
```

---

## 6. 权限检查流程

```
API 调用入口
    │
    ▼
权限检查
    │
    ├───> 需要权限:
    │     │
    │     ▼
    │   AccessTokenKit::VerifyAccessToken()
    │     │
    │     │ 检查 permission
    │     ▼
    │   权限验证结果
    │     │
    │     ├───> 通过: 继续执行
    │     │
    │     └───> 失败: 返回错误码 201
    │
    └───> 不需要权限:
          │
          ▼
        继续执行
```

**涉及权限**:
- `ohos.permission.ACCESS_SCREEN_LOCK`
- `ohos.permission.ACCESS_SCREEN_LOCK_INNER`
- `ohos.permission.DUMP`

---

## 7. 错误处理流程

```
错误发生
    │
    ▼
本地错误码转换
    │
    │ ERROR_CODE_CONVERSION 映射
    │
    ▼
JsErrorCode 转换
    │
    │ ERR_NO_PERMISSION (201)
    │ ERR_INVALID_PARAMS (401)
    │ ERR_SERVICE_ABNORMAL (13200002)
    │ ...
    │
    ▼
napi_throw_error / Promise reject
    │
    ▼
JS 层捕获
```

**错误码映射** (`frameworks/js/napi/src/napi_screenlock_ability.cpp`):
```cpp
const std::map<int, uint32_t> ERROR_CODE_CONVERSION = {
    { E_SCREENLOCK_NO_PERMISSION, JsErrorCode::ERR_NO_PERMISSION },
    { E_SCREENLOCK_PARAMETERS_INVALID, JsErrorCode::ERR_INVALID_PARAMS },
    { E_SCREENLOCK_WRITE_PARCEL_ERROR, JsErrorCode::ERR_SERVICE_ABNORMAL },
    // ...
};
```

---

## 8. 线程模型

### 8.1 线程分布

| 线程 | 职责 | 代码位置 |
|------|------|----------|
| JS Main Thread | N-API 入口、Promise/Callback 管理 | napi_screenlock_ability.cpp |
| Worker Thread | 异步任务执行 | async_call.cpp |
| SA IPC Thread | Binder 消息处理 | screenlock_manager_stub.cpp |
| FFRT Queue | 异步回调分发 | screenlock_system_ability.cpp |

### 8.2 线程同步

```cpp
// services/src/screenlock_system_ability.cpp
std::mutex ScreenLockSystemAbility::instanceLock_;
std::mutex ScreenLockSystemAbility::queueLock_;
std::shared_ptr<ffrt::queue> ScreenLockSystemAbility::queue_;
```

---

## 9. 相关文档

| 文档 | 说明 |
|------|------|
| [01_NAPI_Reference](01_NAPI_Reference.md) | 接口参考 |
| [02_Architecture](02_Architecture.md) | 系统架构 |
| [04_Security_Review](04_Security_Review.md) | 安全评审 |
