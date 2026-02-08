# 调用链分析

## 核心调用链

### 1. 服务注册调用链

```
AddSystemAbility
│
├─▶ SystemAbilityManagerProxy::AddSystemAbility
│   │
│   ├─▶ CheckInputSysAbilityId (客户端校验)
│   │
│   └─▶ SendRequest (IPC 调用)
│       │
│       └─▶ SystemAbilityManagerStub::OnRemoteRequest
│           │
│           ├─▶ CheckInputSysAbilityId (服务端校验)
│           │
│           ├─▶ CheckAddOrRemovePermission (SELinux)
│           │
│           ├─▶ CheckPermission (AccessToken)
│           │
│           └─▶ SystemAbilityManager::AddSystemAbility
│               │
│               ├─▶ CheckInputSysAbilityId
│               │
│               ├─▶ Check ability != nullptr
│               │
│               ├─▶ Check abilityMap_.size() < MAX_SERVICES
│               │
│               ├─▶ abilityMapLock_.lock (unique_lock)
│               │
│               ├─▶ abilityMap_[saId] = saInfo
│               │
│               ├─▶ abilityMapLock_.unlock
│               │
│               ├─▶ AddDeathRecipient (abilityDeath_)
│               │
│               ├─▶ SendSystemAbilityAddedMsg (异步通知)
│               │   │
│               │   └─▶ NotifySystemAbilityChanged
│               │       │
│               │       └─▶ listener->OnAddSystemAbility (回调)
│               │
│               └─▶ Report event (HiSysEvent)
│
└─▶ Return ERR_OK
```

**入口**: `frameworks/native/source/system_ability_manager_proxy.cpp:363`
**核心逻辑**: `services/samgr/native/source/system_ability_manager.cpp:1042`
**返回**: `interfaces/innerkits/samgr_proxy/include/if_system_ability_manager.h:177`

---

### 2. 服务查询调用链

```
GetSystemAbility
│
├─▶ SystemAbilityManagerProxy::GetSystemAbility
│   │
│   ├─▶ DynamicCache::QueryResult (尝试缓存)
│   │   │
│   │   ├─▶ Cache Hit: 返回缓存对象
│   │   │
│   │   └─▶ Cache Miss: 继续 IPC
│   │
│   └─▶ SendRequest (IPC 调用)
│       │
│       └─▶ SystemAbilityManagerStub::OnRemoteRequest
│           │
│           ├─▶ CheckInputSysAbilityId
│           │
│           ├─▶ CheckGetSAPermission (SELinux)
│           │
│           └─▶ SystemAbilityManager::GetSystemAbility
│               │
│               ├─▶ 重试循环 (7次, 200ms间隔)
│               │   │
│               │   └─▶ CheckSystemAbility
│               │       │
│               │       ├─▶ CheckInputSysAbilityId
│               │       │
│               │       ├─▶ abilityMapLock_.lock_shared
│               │       │
│               │       ├─▶ abilityMap_.find(saId)
│               │       │
│               │       ├─▶ abilityMapLock_.unlock
│               │       │
│               │       └─▶ Return SAInfo.remoteObj (或 nullptr)
│               │
│               └─▶ UpdateFrequencyMap (访问频率统计)
│
├─▶ DynamicCache::UpdateCache
│
└─▶ Return IRemoteObject
```

**入口**: `frameworks/native/source/system_ability_manager_proxy.cpp:159`
**核心逻辑**: `services/samgr/native/source/system_ability_manager.cpp:496`
**缓存**: `interfaces/innerkits/dynamic_cache/include/dynamic_cache.h`

---

### 3. 动态加载调用链

```
LoadSystemAbility (异步)
│
├─▶ SystemAbilityManagerProxy::LoadSystemAbility
│   │
│   └─▶ SendRequest (IPC 调用)
│       │
│       └─▶ SystemAbilityManagerStub::OnRemoteRequest
│           │
│           ├─▶ CheckPermission (AccessToken)
│           │
│           └─▶ SystemAbilityManager::LoadSystemAbility
│               │
│               ├─▶ GetSaProfile (从配置文件)
│               │   │
│               │   └─▶ saProfileMap_.find(saId)
│               │
│               ├─▶ CheckInputSysAbilityId
│               │
│               ├─▶ Check is already loaded
│               │
│               └─▶ DoLoadSystemAbility
│                   │
│                   ├─▶ StartingSystemProcess
│                   │   │
│                   │   ├─▶ ScheduleProcessState (START)
│                   │   │   │
│                   │   │   └─▶ SystemAbilityStateScheduler::ScheduleProcessState
│                   │   │       │
│                   │   │       ├─▶ GetProcessContext
│                   │   │       │
│                   │   │       ├─▶ TransitProcessState
│                   │   │       │
│                   │   │       └─▶ StartProcess (LSAMgr)
│                   │   │           │
│                   │   │           └─▶ LocalAbilityManager::StartProcess
│                   │   │               │
│                   │   │               └─▶ init 启动进程
│                   │   │
│                   │   └─▶ Add starting callback
│                   │
│                   └─▶ SendCheckLoadedMsg (定时检查)
│
└─▶ Return ERR_OK (不表示加载成功)

# 加载完成后
SA Process
│
├─▶ AddSystemAbility (注册自己)
│   │
│   └─▶ ... (参见服务注册调用链)
│
└─▶ Trigger notify
    │
    └─▶ SystemAbilityManager::NotifySystemAbilityLoaded
        │
        ├─▶ Find callback in startingAbilityMap_
        │
        └─▶ callback->OnLoadSystemAbilitySuccess
```

**入口**: `services/samgr/native/source/system_ability_manager_stub.cpp:567`
**核心逻辑**: `services/samgr/native/source/system_ability_manager.cpp:1614`
**状态机**: `services/samgr/native/source/schedule/system_ability_state_scheduler.cpp`

---

### 4. 订阅状态变化调用链

```
SubscribeSystemAbility
│
├─▶ SystemAbilityManagerProxy::SubscribeSystemAbility
│   │
│   └─▶ SendRequest (IPC 调用)
│       │
│       └─▶ SystemAbilityManagerStub::OnRemoteRequest
│           │
│           └─▶ SystemAbilityManager::SubscribeSystemAbility
│               │
│               ├─▶ CheckInputSysAbilityId
│               │
│               ├─▶ listener != nullptr
│               │
│               ├─▶ Check existing listeners
│               │
│               ├─▶ listenerMapLock_.lock
│               │
│               ├─▶ listenerMap_[saId].push_back(listener)
│               │
│               ├─▶ listenerMapLock_.unlock
│               │
│               ├─▶ AddDeathRecipient (abilityStatusDeath_)
│               │
│               └─▶ CheckListenerNotify (检查是否需要立即通知)
│                   │
│                   └─▶ If SA already exists: OnAddSystemAbility
│
└─▶ Return ERR_OK
```

**入口**: `services/samgr/native/source/system_ability_manager_stub.cpp:610`
**核心逻辑**: `services/samgr/native/source/system_ability_manager.cpp:907`

---

### 5. 死亡监听调用链

```
SA Process Death
│
├─▶ Kernel 通知 Binder
│
├─▶ IRemoteObject::DeathRecipient::OnRemoteDied
│   │
│   └─▶ AbilityDeathRecipient::OnRemoteDied
│       │
│       └─▶ SystemAbilityManager::RemoveSystemAbility
│           │
│           ├─▶ abilityMapLock_.lock
│           │
│           ├─▶ abilityMap_.find (通过 remote 对象查找)
│           │
│           ├─▶ abilityMap_.erase
│           │
│           ├─▶ abilityMapLock_.unlock
│           │
│           ├─▶ SendSystemAbilityRemovedMsg
│           │   │
│           │   └─▶ Notify listeners (OnRemoveSystemAbility)
│           │
│           └─▶ Report event (HiSysEvent)
│
└─▶ Cleanup complete
```

**入口**: `services/samgr/native/source/ability_death_recipient.cpp`
**核心逻辑**: `services/samgr/native/source/system_ability_manager.cpp:760`

---

### 6. 配置文件加载调用链

```
InitSaProfile
│
├─▶ GetProfilePaths (获取所有配置文件路径)
│
├─▶ For each profile:
│   │
│   └─▶ LoadSaProfiles
│       │
│       ├─▶ GetRealPath (路径规范化)
│       │
│       ├─▶ CheckPathExist
│       │
│       ├─▶ LoadSaProfilesFromJson
│       │   │
│       │   ├─▶ Open file stream
│       │   │
│       │   ├─▶ Check file size < MAX_JSON_OBJECT_SIZE
│       │   │
│       │   ├─▶ nlohmann::json::parse
│       │   │
│       │   └─▶ For each SA in profile:
│       │       │
│       │       └─▶ ParseSystemAbilityFromJson
│       │           │
│       │           ├─▶ Parse saId, process, name
│       │           │
│       │           ├─▶ Parse ondemand_events
│       │           │
│       │           ├─▶ CheckRecycleStrategy
│       │           │
│       │           ├─▶ CheckLogicRelationship
│       │           │
│       │           └─▶ Create CommonSaProfile
│       │
│       └─▶ InsertSaProfileToMap
│           │
│           ├─▶ saProfileMapLock_.lock
│           │
│           ├─▶ saProfileMap_[saId] = profile
│           │
│           └─▶ saProfileMapLock_.unlock
│
└─▶ Init complete
```

**入口**: `services/samgr/native/source/system_ability_manager.cpp:2050`
**解析逻辑**: `services/common/src/parse_util.cpp:230`

---

## 关键函数索引

| 函数 | 文件 | 行号 | 说明 |
|------|------|------|------|
| AddSystemAbility | system_ability_manager.cpp | 1042 | 服务注册 |
| CheckSystemAbility | system_ability_manager.cpp | 523 | 服务查询 |
| GetSystemAbility | system_ability_manager.cpp | 496 | 阻塞获取服务 |
| LoadSystemAbility | system_ability_manager.cpp | 1614 | 动态加载 |
| RemoveSystemAbility | system_ability_manager.cpp | 760 | 服务移除 |
| SubscribeSystemAbility | system_ability_manager.cpp | 907 | 订阅状态 |
| DoLoadSystemAbility | system_ability_manager.cpp | 1680 | 实际加载逻辑 |
| StartingSystemProcess | system_ability_manager.cpp | 1845 | 启动进程 |
| NotifySystemAbilityLoaded | system_ability_manager.cpp | 1980 | 加载完成通知 |
| OnRemoteDied | ability_death_recipient.cpp | 45 | 死亡处理 |
| InitSaProfile | system_ability_manager.cpp | 2050 | 初始化配置 |
| ParseSaProfiles | parse_util.cpp | 597 | 解析配置 |
| ScheduleProcessState | system_ability_state_scheduler.cpp | 120 | 调度进程状态 |
| TransitProcessState | system_ability_state_machine.cpp | 85 | 状态转换 |
