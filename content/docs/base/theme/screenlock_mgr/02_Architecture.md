# 02_Architecture - 系统架构

## 1. 架构概述

### 1.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  第三方应用   │  │  系统应用     │  │    运营管理           │  │
│  │  (JS/ETS)    │  │  (Native)     │  │    (系统事件)         │  │
│  └──────┬───────┘  └──────┬───────┘  └───────────┬──────────┘  │
└─────────┼──────────────────┼──────────────────────┼─────────────┘
          │                  │                      │
          ▼                  ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                     接口层 (Frameworks)                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              N-API (frameworks/js/napi/)                 │   │
│  │  ┌─────────────────────────────────────────────────┐     │   │
│  │  │  napi_screenlock_ability.cpp                    │     │   │
│  │  │  - Init() [注册所有 JS API]                    │     │   │
│  │  │  - AsyncCall [异步调用封装]                    │     │   │
│  │  └─────────────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              ETS/ANI (frameworks/ets/ani/)                │   │
│  │  ┌─────────────────────────────────────────────────┐     │   │
│  │  │  @ohos.screenLock.ets                           │     │   │
│  │  │  - ArkUI 接口声明                               │     │   │
│  │  └─────────────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Native (frameworks/native/)                  │   │
│  │  ┌─────────────────────────────────────────────────┐     │   │
│  │  │  screenlock_manager_proxy.cpp                  │     │   │
│  │  │  - IPC 代理实现                                 │     │   │
│  │  └─────────────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                      服务层 (Services)                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              ScreenLockSystemAbility                      │   │
│  │  (services/src/screenlock_system_ability.cpp)            │   │
│  │  - SA ID: 3704                                          │   │
│  │  - 运行在 foundation 进程                                │   │
│  │  - 单例模式                                              │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                       │
│                          ▼                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              核心模块                                     │   │
│  │  ┌─────────────────┐  ┌─────────────────┐              │   │
│  │  │ StrongAuthManager│  │InnerListenerMgr │              │   │
│  │  │ - 强认证管理     │  │ - 内部监听器管理 │              │   │
│  │  └─────────────────┘  └─────────────────┘              │   │
│  │  ┌─────────────────┐  ┌─────────────────┐              │   │
│  │  │ Command         │  │DumpHelper       │              │   │
│  │  │ - 命令处理      │  │ - 调试信息输出  │              │   │
│  │  └─────────────────┘  └─────────────────┘              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                     依赖子系统                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   IPC/Binder │  │AccessToken   │  │   UserAuth          │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  OsAccount   │  │ WindowManager│  │   CommonEvent        │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

> **证据来源**: 目录结构分析 + sa_profile/3704.json + services/BUILD.gn

---

## 2. 组件说明

### 2.1 N-API 层 (frameworks/js/napi/)

**职责**: 将 Native 接口暴露给 JS/ETS 应用

**关键文件**:
| 文件 | 职责 |
|------|------|
| `napi_screenlock_ability.cpp` | N-API 注册和实现 (证据: 67-88 行 Init 函数) |
| `async_call.cpp` | 异步调用封装 |
| `screenlock_callback.cpp` | 回调处理 |
| `uv_queue.cpp` | UV 队列管理 |

**注册机制**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:67-88
napi_status Init(napi_env env, napi_value exports)
{
    napi_property_descriptor exportFuncs[] = {
        DECLARE_NAPI_FUNCTION("isScreenLocked", OHOS::ScreenLock::NAPI_IsScreenLocked),
        DECLARE_NAPI_FUNCTION("lock", OHOS::ScreenLock::NAPI_Lock),
        // ... 更多 API
    };
    napi_define_properties(env, exports, sizeof(exportFuncs) / sizeof(*exportFuncs), exportFuncs);
    return napi_ok;
}
```

---

### 2.2 ETS/ANI 层 (frameworks/ets/ani/)

**职责**: 提供 ArkUI/ETS 声明式接口

**关键文件**:
| 文件 | 职责 |
|------|------|
| `@ohos.screenLock.ets` | ETS API 接口声明 |
| `ani_screenlock_ability.cpp` | ANI 实现 |

---

### 2.3 Native 层 (frameworks/native/)

**职责**: 提供系统应用使用的 Native 接口

**关键文件**:
| 文件 | 职责 |
|------|------|
| `screenlock_manager_proxy.cpp` | IPC 代理客户端 |
| `screenlock_system_ability_stub.cpp` | IPC 存根服务端 |
| `screenlock_manager.cpp` | 管理器实现 |

**接口定义**:
```cpp
// interfaces/inner_api/include/screenlock_manager_interface.h:29-57
class ScreenLockManagerInterface : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.screenlock.ScreenLockManagerInterface");
    virtual int32_t IsLocked(bool &isLocked) = 0;
    virtual bool IsScreenLocked() = 0;
    virtual int32_t Unlock(const sptr<ScreenLockCallbackInterface> &listener) = 0;
    // ... 更多接口
};
```

---

### 2.4 服务层 (services/)

**职责**: 核心锁屏逻辑实现

**关键组件**:
| 组件 | 职责 |
|------|------|
| `ScreenLockSystemAbility` | SA 主类，负责生命周期管理 |
| `StrongAuthManager` | 强认证状态管理 |
| `InnerListenerManager` | 内部事件监听器管理 |
| `Command` | 命令行支持 |
| `DumpHelper` | 调试信息输出 |

**SA 注册**:
```cpp
// services/src/screenlock_system_ability.cpp:72
REGISTER_SYSTEM_ABILITY_BY_ID(ScreenLockSystemAbility, SCREENLOCK_SERVICE_ID, true);
```

**SA 配置**:
```json
// sa_profile/3704.json
{
    "process": "foundation",
    "systemability": [{
        "name": 3704,
        "libpath": "libscreenlock_server.z.so",
        "run-on-create": true,
        "distributed": false
    }]
}
```

---

## 3. 数据流

### 3.1 屏幕解锁流程

```
JS/N-API                    Native Proxy              ScreenLock SA
    │                            │                          │
    │ unlockScreen()             │                          │
    │───────────────────────────>│                          │
    │                            │ IPC (Binder)             │
    │                            │─────────────────────────>│
    │                            │                          │ 验证权限
    │                            │                          │ 检查状态
    │                            │                          │ 执行解锁
    │                            │                          │ 发布事件
    │                            │                          │───────
    │                            │                          │     │ 异步
    │                            │                          │ <────│ 解锁结果
    │                            │ IPC Response             │
    │                            │<──────────────────────────│
    │ Promise resolve            │                          │
    │<───────────────────────────│                          │
```

---

### 3.2 系统事件回调流程

```
ScreenLock SA              CommonEvent              JS Callback
       │                        │                        │
       │ OnSystemEvent          │                        │
       │────────────────────────>│                        │
       │                         │ Publish Event          │
       │                         │───────────────────────>│
       │                         │                        │ Trigger callback
```

---

## 4. 线程模型

### 4.1 线程说明

| 线程 | 职责 |
|------|------|
| 主线程 (UI Thread) | JS 执行、N-API 调用入口 |
| SA 线程 (Foundation) | SA 消息处理、业务逻辑 |
| FFRT 线程池 | 异步任务执行 |
| UV 线程 | Node.js 事件循环集成 |

### 4.2 异步调用实现

```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:221-261
void AsyncCallFunc(napi_env env, EventListener *listener, const std::string &resourceName)
{
    napi_create_async_work(
        env, nullptr, resource, execute, CompleteAsyncWork, 
        static_cast<void *>(listener), &(listener->work));
    napi_queue_async_work_with_qos(env, listener->work, napi_qos_user_initiated);
}
```

---

## 5. 进程模型

```
┌─────────────────────────────────────────────────────────────┐
│                      应用进程                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  JS/ETS Runtime                                    │   │
│  │    │                                               │   │
│  │    ▼                                               │   │
│  │  libscreenlock.so (N-API)                         │   │
│  │    │                                               │   │
│  │    ▼                                               │   │
│  │  libscreenlock_client.so (IPC Proxy)              │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                         IPC (Binder)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Foundation 进程                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  libscreenlock_server.z.so (SA)                    │   │
│  │    │                                               │   │
│  │    ▼                                               │   │
│  │  ScreenLockSystemAbility (SA 3704)                │   │
│  │    │                                               │   │
│  │    ▼                                               │   │
│  │  StrongAuthManager / InnerListenerManager         │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. 模块依赖关系

```
                    ┌─────────────────────────────────────┐
                    │         N-API (screenlock)          │
                    │     (frameworks/js/napi/BUILD.gn)   │
                    └─────────────────┬───────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────┐
                    │      Native Client (screenlock_client)
                    │   (frameworks/native/BUILD.gn)      │
                    └─────────────────┬───────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────┐
                    │       Inner API (screenlock_client) │
                    │   (interfaces/inner_api/BUILD.gn)  │
                    └─────────────────┬───────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────┐
                    │    ScreenLock SA (screenlock_server)│
                    │       (services/BUILD.gn)          │
                    └─────────────────┬───────────────────┘
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          ▼                           ▼                           ▼
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│StrongAuthManager│      │InnerListenerMgr │      │   Watch 模块    │
│                 │      │                 │      │  (可选编译)     │
└─────────────────┘      └─────────────────┘      └─────────────────┘
```

---

## 7. 稳定性标注

| 模块 | 稳定性 | 依据 |
|------|--------|------|
| N-API 接口 | 稳定 | 官方文档支持的 JS API |
| Inner API | 稳定 | 带 `innerapi_tags = ["platformsdk", "sasdk"]` |
| SA 内部实现 | 不稳定 | 服务内部逻辑，可能变更 |

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_NAPI_Reference](01_NAPI_Reference.md) | 接口参考 |
| [03_GN_Build](03_GN_Build.md) | 构建系统 |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 详细调用链 |
| [04_Security_Review](04_Security_Review.md) | 安全评审 |
