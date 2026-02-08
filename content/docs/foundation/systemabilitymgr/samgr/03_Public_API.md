# 对外 API 文档

## 重要说明

**本组件不包含 N-API（JavaScript）接口**，仅提供以下原生接口：
- **C++ InnerKits** - 主要接口
- **Rust 绑定** - 语言绑定

## C++ 接口概览

### 核心接口类

#### ISystemAbilityManager

**文件**: `interfaces/innerkits/samgr_proxy/include/if_system_ability_manager.h:39`

```cpp
class ISystemAbilityManager : public IRemoteBroker {
public:
    // 服务发现
    virtual sptr<IRemoteObject> GetSystemAbility(int32_t systemAbilityId) = 0;
    virtual sptr<IRemoteObject> CheckSystemAbility(int32_t systemAbilityId) = 0;
    
    // 服务管理
    virtual int32_t AddSystemAbility(int32_t systemAbilityId, 
        const sptr<IRemoteObject>& ability, const SAExtraProp& extraProp) = 0;
    virtual int32_t RemoveSystemAbility(int32_t systemAbilityId) = 0;
    
    // 订阅通知
    virtual int32_t SubscribeSystemAbility(int32_t systemAbilityId,
        const sptr<ISystemAbilityStatusChange>& listener) = 0;
    virtual int32_t UnSubscribeSystemAbility(int32_t systemAbilityId,
        const sptr<ISystemAbilityStatusChange>& listener) = 0;
    
    // 动态加载
    virtual int32_t LoadSystemAbility(int32_t systemAbilityId,
        const sptr<ISystemAbilityLoadCallback>& callback) = 0;
    virtual int32_t UnloadSystemAbility(int32_t systemAbilityId) = 0;
    
    // 分布式
    virtual sptr<IRemoteObject> GetSystemAbility(int32_t systemAbilityId, 
        const std::string& deviceId) = 0;
};
```

### API 清单表

#### 服务发现 API

| API 名称 | 参数 | 返回值 | 同步/异步 | 对应 C++ 实现 |
|----------|------|--------|-----------|--------------|
| `GetSystemAbility` | `int32_t systemAbilityId` | `sptr<IRemoteObject>` | 同步(带重试) | `system_ability_manager.cpp:496` |
| `CheckSystemAbility` | `int32_t systemAbilityId` | `sptr<IRemoteObject>` | 同步 | `system_ability_manager.cpp:523` |
| `CheckSystemAbility` | `int32_t said, bool& isExist` | `sptr<IRemoteObject>` | 同步 | `system_ability_manager.cpp:540` |
| `GetSystemAbility` | `int32_t said, string& deviceId` | `sptr<IRemoteObject>` | 同步(分布式) | `system_ability_manager.cpp:600` |
| `CheckSystemAbility` | `int32_t said, string& deviceId` | `sptr<IRemoteObject>` | 同步(分布式) | `system_ability_manager.cpp:630` |
| `ListSystemAbilities` | `uint32_t dumpFlags` | `vector<u16string>` | 同步 | `system_ability_manager.cpp:460` |

#### 服务管理 API

| API 名称 | 参数 | 返回值 | 同步/异步 | 对应 C++ 实现 |
|----------|------|--------|-----------|--------------|
| `AddSystemAbility` | `int32_t said, IRemoteObject, SAExtraProp` | `int32_t` | 同步 | `system_ability_manager.cpp:1042` |
| `RemoveSystemAbility` | `int32_t systemAbilityId` | `int32_t` | 同步 | `system_ability_manager.cpp:760` |
| `AddOnDemandSystemAbilityInfo` | `int32_t said, u16string procName` | `int32_t` | 同步 | `system_ability_manager.cpp:655` |
| `AddSystemProcess` | `u16string procName, IRemoteObject` | `int32_t` | 同步 | `system_ability_manager.cpp:1100` |

#### 订阅通知 API

| API 名称 | 参数 | 返回值 | 同步/异步 | 对应 C++ 实现 |
|----------|------|--------|-----------|--------------|
| `SubscribeSystemAbility` | `int32_t said, ISystemAbilityStatusChange` | `int32_t` | 同步 | `system_ability_manager.cpp:907` |
| `UnSubscribeSystemAbility` | `int32_t said, ISystemAbilityStatusChange` | `int32_t` | 同步 | `system_ability_manager.cpp:971` |
| `SubscribeSystemProcess` | `ISystemProcessStatusChange` | `int32_t` | 同步 | `system_ability_manager.cpp:1400` |
| `UnSubscribeSystemProcess` | `ISystemProcessStatusChange` | `int32_t` | 同步 | `system_ability_manager.cpp:1450` |

#### 动态加载 API

| API 名称 | 参数 | 返回值 | 同步/异步 | 对应 C++ 实现 |
|----------|------|--------|-----------|--------------|
| `LoadSystemAbility` | `int32_t said, int32_t timeout` | `sptr<IRemoteObject>` | 同步(阻塞) | `system_ability_manager.cpp:163` |
| `LoadSystemAbility` | `int32_t said, ISystemAbilityLoadCallback` | `int32_t` | 异步 | `system_ability_manager.cpp:1614` |
| `LoadSystemAbility` | `int32_t said, string deviceId, callback` | `int32_t` | 异步(分布式) | `system_ability_manager.cpp:1633` |
| `UnloadSystemAbility` | `int32_t systemAbilityId` | `int32_t` | 同步 | `system_ability_manager.cpp:1675` |
| `CancelUnloadSystemAbility` | `int32_t systemAbilityId` | `int32_t` | 同步 | `system_ability_manager.cpp:1700` |

#### 进程管理 API

| API 名称 | 参数 | 返回值 | 同步/异步 | 对应 C++ 实现 |
|----------|------|--------|-----------|--------------|
| `GetSystemProcessInfo` | `int32_t said, SystemProcessInfo` | `int32_t` | 同步 | `system_ability_manager.cpp:1720` |
| `GetRunningSystemProcess` | `list<SystemProcessInfo>` | `int32_t` | 同步 | `system_ability_manager.cpp:1750` |
| `UnloadAllIdleSystemAbility` | - | `int32_t` | 同步 | `system_ability_manager.cpp:1780` |

### 回调接口

#### ISystemAbilityStatusChange

**文件**: `interfaces/innerkits/samgr_proxy/include/isystem_ability_status_change.h`

```cpp
class ISystemAbilityStatusChange : public IRemoteBroker {
public:
    virtual void OnAddSystemAbility(int32_t systemAbilityId, 
        const std::string& deviceId) = 0;
    virtual void OnRemoveSystemAbility(int32_t systemAbilityId,
        const std::string& deviceId) = 0;
};
```

#### ISystemAbilityLoadCallback

**文件**: `interfaces/innerkits/samgr_proxy/include/isystem_ability_load_callback.h`

```cpp
class ISystemAbilityLoadCallback : public IRemoteBroker {
public:
    virtual void OnLoadSystemAbilitySuccess(int32_t systemAbilityId,
        const sptr<IRemoteObject>& remoteObject) = 0;
    virtual void OnLoadSystemAbilityFail(int32_t systemAbilityId) = 0;
};
```

### 数据结构

#### SAExtraProp

**文件**: `interfaces/innerkits/samgr_proxy/include/if_system_ability_manager.h:151`

```cpp
struct SAExtraProp {
    bool isDistributed = false;           // 是否支持分布式
    unsigned int dumpFlags = 0;           // Dump 标志
    std::u16string capability;            // 能力描述
    std::u16string permission;            // 所需权限
};
```

#### SystemProcessInfo

**文件**: `interfaces/innerkits/samgr_proxy/include/isystem_process_status_change.h`

```cpp
struct SystemProcessInfo {
    std::u16string processName;
    int32_t pid = -1;
    int32_t uid = -1;
};
```

### 错误码

**文件**: `interfaces/innerkits/common/include/samgr_err_code.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `SAMGR_OK` | 0 | 成功 |
| `INVALID_SYSTEM_ABILITY_ID` | 1000 | 无效的 SA ID |
| `INVALID_INPUT_PARA` | 1001 | 输入参数无效 |
| `PROFILE_NOT_EXIST` | 1002 | 配置文件不存在 |
| `CALLBACK_NULL` | 1003 | 回调为空 |
| `SA_NOT_EXIST` | 1007 | SA 不存在 |
| `ONDEMAND_SIZE_LIMIT` | 1009 | 按需加载数量超限 |
| `SUBSCRIBE_SIZE_LIMIT` | 1010 | 订阅数量超限 |
| `ABILITY_MAP_SIZE_LIMIT` | 1011 | SA 映射表已满 |

### 使用示例

#### 1. 获取系统能力

```cpp
#include "iservice_registry.h"
#include "system_ability_definition.h"

// 获取 Samgr 客户端实例
sptr<ISystemAbilityManager> samgr = 
    SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();

// 获取 BundleManager 服务
sptr<IRemoteObject> bmObj = samgr->GetSystemAbility(
    BUNDLE_MGR_SERVICE_SYS_ABILITY_ID);
if (bmObj == nullptr) {
    // 服务不存在或获取失败
    return ERR_INVALID_VALUE;
}
```

#### 2. 注册系统能力

```cpp
#include "if_system_ability_manager.h"

// 创建 SA 实例
sptr<MySystemAbility> sa = new MySystemAbility();

// 准备额外属性
ISystemAbilityManager::SAExtraProp extraProp;
extraProp.isDistributed = false;
extraProp.dumpFlags = DUMP_FLAG_PRIORITY_DEFAULT;

// 注册 SA
int32_t ret = samgr->AddSystemAbility(MY_SA_ID, sa->AsObject(), extraProp);
if (ret != ERR_OK) {
    // 注册失败
    return ret;
}
```

#### 3. 订阅 SA 状态变化

```cpp
#include "system_ability_status_change_stub.h"

class MyStatusChangeListener : public SystemAbilityStatusChangeStub {
public:
    void OnAddSystemAbility(int32_t saId, const std::string& deviceId) override {
        // SA 上线处理
    }
    
    void OnRemoveSystemAbility(int32_t saId, const std::string& deviceId) override {
        // SA 下线处理
    }
};

// 订阅
sptr<ISystemAbilityStatusChange> listener = new MyStatusChangeListener();
int32_t ret = samgr->SubscribeSystemAbility(TARGET_SA_ID, listener);
```

#### 4. 动态加载 SA

```cpp
#include "system_ability_load_callback_stub.h"

class MyLoadCallback : public SystemAbilityLoadCallbackStub {
public:
    void OnLoadSystemAbilitySuccess(int32_t saId, 
        const sptr<IRemoteObject>& remoteObject) override {
        // 加载成功，使用 remoteObject
    }
    
    void OnLoadSystemAbilityFail(int32_t saId) override {
        // 加载失败
    }
};

// 发起异步加载
sptr<ISystemAbilityLoadCallback> callback = new MyLoadCallback();
int32_t ret = samgr->LoadSystemAbility(TARGET_SA_ID, callback);
// 注意：返回 ERR_OK 不代表加载成功，只代表请求已接收
```

## Rust 绑定

### 概述

Samgr 提供了 Rust 语言绑定，位于 `interfaces/innerkits/rust/`。

### 主要模块

| 模块 | 文件 | 说明 |
|------|------|------|
| `manage` | `src/manage.rs` | SystemAbilityManager 包装器 |
| `status_change` | `src/status_change.rs` | 状态变化监听 |
| `wrapper` | `src/cxx/wrapper.rs` | C++ FFI 包装 |

### 使用示例

```rust
use samgr::manage::SystemAbilityManager;

fn main() {
    // 获取 SA 管理器实例
    let samgr = SystemAbilityManager::get_instance();
    
    // 获取系统能力
    match samgr.get_system_ability(401) {  // BUNDLE_MGR_SERVICE
        Some(remote_obj) => {
            // 使用远程对象
        }
        None => {
            println!("Service not found");
        }
    }
}
```

## 调用链

### GetSystemAbility 调用链

```
┌─────────────────────────────────────────────────────────────┐
│  Client                                                     │
│  GetSystemAbility(saId)                                     │
│       │                                                     │
│       ▼                                                     │
│  SystemAbilityManagerProxy::GetSystemAbility()              │
│       │                                                     │
│       ▼                                                     │
│  SendRequest(GET_SYSTEM_ABILITY_TRANSACTION) ───────────────┼──► IPC
│                                                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Server                                                     │
│  SystemAbilityManagerStub::OnRemoteRequest()                │
│       │                                                     │
│       ▼                                                     │
│  GET_SYSTEM_ABILITY_TRANSACTION handler                     │
│       │                                                     │
│       ▼                                                     │
│  CheckInputSysAbilityId(saId) // 参数校验                   │
│       │                                                     │
│       ▼                                                     │
│  CheckGetSAPermission() // SELinux 权限检查                 │
│       │                                                     │
│       ▼                                                     │
│  SystemAbilityManager::GetSystemAbility()                   │
│       │                                                     │
│       ▼                                                     │
│  abilityMapLock_.lock_shared()                              │
│       │                                                     │
│       ▼                                                     │
│  abilityMap_.find(saId)                                     │
│       │                                                     │
│       ▼                                                     │
│  return SAInfo.remoteObj                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 权限要求

| API | 所需权限 | 检查位置 |
|-----|----------|----------|
| `GetSystemAbility` | SELinux `samgr_get` | `system_ability_manager_stub.cpp:48` |
| `AddSystemAbility` | SELinux `samgr_add` + AccessToken | `system_ability_manager_stub.cpp:61` |
| `RemoveSystemAbility` | SELinux `samgr_remove` | `system_ability_manager_stub.cpp:74` |
| `ListSystemAbilities` | SELinux `samgr_list` | `system_ability_manager_stub.cpp:87` |
| `LoadSystemAbility` | AccessToken 原生令牌 | `system_ability_manager_stub.cpp:1299` |
| `SubscribeSystemAbility` | 无特殊要求 | - |
