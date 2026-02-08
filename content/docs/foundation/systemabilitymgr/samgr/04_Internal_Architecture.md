# 内部架构与实现

## 核心模块

### 1. SystemAbilityManager

**文件**: `services/samgr/native/include/system_ability_manager.h:61`

```cpp
class SystemAbilityManager : public DynamicCache, public SystemAbilityManagerStub {
    // 单例模式
    static sptr<SystemAbilityManager> GetInstance();
    
    // 核心数据
    std::map<int32_t, SAInfo> abilityMap_;           // SA 注册表
    std::map<int32_t, std::list<SAListener>> listenerMap_;  // 监听器
    std::map<int32_t, std::u16string> onDemandAbilityMap_;  // 按需加载表
    std::map<std::u16string, sptr<IRemoteObject>> systemProcessMap_;  // 进程表
};
```

**关键方法**:

| 方法 | 说明 | 实现位置 |
|------|------|----------|
| `Init()` | 初始化，加载配置，启动定时器 | `system_ability_manager.cpp:2050` |
| `AddSystemAbility()` | 注册 SA 到 abilityMap_ | `system_ability_manager.cpp:1042` |
| `CheckSystemAbility()` | 从 abilityMap_ 查询 SA | `system_ability_manager.cpp:523` |
| `LoadSystemAbility()` | 触发 SA 按需加载 | `system_ability_manager.cpp:1614` |
| `RemoveSystemAbility()` | 从 abilityMap_ 移除 SA | `system_ability_manager.cpp:760` |
| `InitSaProfile()` | 加载 SA 配置文件 | `system_ability_manager.cpp:1895` |

### 2. SystemAbilityManagerStub

**文件**: `services/samgr/native/include/system_ability_manager_stub.h`

IPC 存根实现，处理远程调用请求。

```cpp
class SystemAbilityManagerStub : public IRemoteStub<ISystemAbilityManager> {
public:
    int OnRemoteRequest(uint32_t code, MessageParcel& data,
        MessageParcel& reply, MessageOption& option) override;
};
```

**IPC 命令处理**:

| Code | 处理函数 | 说明 |
|------|----------|------|
| 1 | GetSystemAbility | 获取 SA |
| 2 | CheckSystemAbility | 检查 SA |
| 3 | AddSystemAbility | 注册 SA |
| 4 | RemoveSystemAbility | 移除 SA |
| 6 | SubscribeSystemAbility | 订阅 SA |
| 7 | LoadSystemAbility | 加载 SA |
| 40+ | ... | 其他命令 |

**权限检查**:

```cpp
// Stub 层统一进行权限检查
bool CheckGetSAPermission();        // 获取 SA 权限
bool CheckAddOrRemovePermission();  // 添加/移除权限
bool CheckPermission(const string& permission);  // AccessToken 权限
```

### 3. SystemAbilityStateScheduler

**文件**: `services/samgr/native/include/schedule/system_ability_state_scheduler.h`

管理 SA 和进程的生命周期状态。

```cpp
class SystemAbilityStateScheduler {
public:
    int32_t ScheduleAbilityState(int32_t saId, AbilityStateEvent event);
    int32_t ScheduleProcessState(const u16string& procName, ProcessStateEvent event);
    int32_t SendStrategyToAll(StateSchedulerStrategy strategy);
};
```

**状态定义**:

```cpp
enum class AbilityState {
    INIT,       // 初始状态
    STARTING,   // 启动中
    STARTED,    // 已启动
    IDLE,       // 空闲
    ACTIVE,     // 活跃
    STOPPED,    // 已停止
};

enum class ProcessState {
    INIT,
    STARTING,
    STARTED,
    IDLE,
};
```

### 4. DeviceStatusCollectManager

**文件**: `services/samgr/native/include/collect/device_status_collect_manager.h`

收集设备状态，触发按需加载/卸载。

```cpp
class DeviceStatusCollectManager {
public:
    int32_t StartCollect();
    int32_t StopCollect();
    int32_t AddCollectPlugin(std::shared_ptr<ICollectPlugin> plugin);
    
private:
    std::list<std::shared_ptr<ICollectPlugin>> collectPluginList_;
};
```

**收集插件**:

| 插件 | 功能 | 条件编译 |
|------|------|----------|
| CommonEventCollect | 通用事件监听 | `SUPPORT_COMMON_EVENT` |
| DeviceParamCollect | 设备参数变化 | - |
| DeviceNetworkingCollect | 网络状态变化 | `SUPPORT_DEVICE_MANAGER` |
| DeviceSwitchCollect | 设备开关事件 | `SUPPORT_SWITCH_COLLECT` |
| RefCountCollect | 引用计数统计 | - |

### 5. LocalAbilityManager

**文件**: `services/lsamgr/include/local_abilitys.h`

管理本地 SA 进程的启动和停止。

```cpp
class LocalAbilityManager {
public:
    bool StartProcess(const std::u16string& name);
    bool StopProcess(const std::u16string& name);
};
```

## 数据流详解

### 服务注册数据流

```
┌──────────────────────────────────────────────────────────────────────┐
│                          SA 注册流程                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. 客户端调用                                                       │
│     AddSystemAbility(saId, remoteObj, extraProp)                     │
│                      │                                               │
│                      ▼                                               │
│  2. IPC 传输到 Samgr                                                 │
│     SystemAbilityManagerStub::OnRemoteRequest()                      │
│                      │                                               │
│                      ▼                                               │
│  3. 权限检查                                                         │
│     CheckAddOrRemovePermission()                                     │
│     CheckPermission(accessToken)                                     │
│                      │                                               │
│                      ▼                                               │
│  4. 参数校验                                                         │
│     CheckInputSysAbilityId(saId)                                     │
│     ability != nullptr                                               │
│     abilityMap_.size() < MAX_SERVICES                                │
│                      │                                               │
│                      ▼                                               │
│  5. 注册到映射表                                                     │
│     lock(abilityMapLock_)                                            │
│     abilityMap_[saId] = SAInfo{remoteObj, isDistributed}             │
│     unlock(abilityMapLock_)                                          │
│                      │                                               │
│                      ▼                                               │
│  6. 设置死亡监听                                                     │
│     remoteObj->AddDeathRecipient(abilityDeath_)                      │
│                      │                                               │
│                      ▼                                               │
│  7. 通知订阅者                                                       │
│     SendSystemAbilityAddedMsg(saId)                                  │
│         -> NotifySystemAbilityChanged()                              │
│         -> NotifySystemAbilityAddedByAsync()                         │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### 按需加载数据流

```
┌──────────────────────────────────────────────────────────────────────┐
│                        按需加载流程                                  │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. 客户端请求加载                                                   │
│     LoadSystemAbility(saId, callback)                                │
│                      │                                               │
│                      ▼                                               │
│  2. 查找进程名                                                       │
│     GetSaProfile(saId) -> CommonSaProfile.process                    │
│                      │                                               │
│                      ▼                                               │
│  3. 状态机调度                                                       │
│     ScheduleProcessState(procName, START)                            │
│                      │                                               │
│                      ▼                                               │
│  4. 启动进程                                                         │
│     LocalAbilityManager::StartProcess()                              │
│     -> init 启动 SA 进程                                             │
│                      │                                               │
│                      ▼                                               │
│  5. SA 注册自己                                                      │
│     (新进程) AddSystemAbility(saId, ...)                             │
│                      │                                               │
│                      ▼                                               │
│  6. 通知客户端                                                       │
│     NotifySystemAbilityLoaded()                                      │
│     callback->OnLoadSystemAbilitySuccess(saId, remoteObj)            │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

## 关键数据结构

### SAInfo

```cpp
// services/samgr/native/include/system_ability_manager.h:37
struct SAInfo {
    sptr<IRemoteObject> remoteObj;  // 远程对象
    bool isDistributed = false;      // 是否分布式
};
```

### SAListener

```cpp
// services/samgr/native/include/system_ability_manager.h:53
struct SAListener {
    sptr<ISystemAbilityStatusChange> listener;  // 监听器
    int32_t callingPid;                          // 调用进程 PID
    ListenerState state = ListenerState::INIT;   // 监听状态
};
```

### CommonSaProfile

```cpp
// interfaces/innerkits/common/include/sa_profiles.h
struct CommonSaProfile {
    int32_t saId;
    std::u16string process;
    std::u16string name;
    bool isDistributed = false;
    bool cacheCommonEvent = false;
    RecycleStrategy recycleStrategy;
    std::vector<OnDemandEvent> onDemandEvents;
    std::map<std::string, std::string> extensionMap;
};
```

### OnDemandEvent

```cpp
// interfaces/innerkits/samgr_proxy/include/system_ability_on_demand_event.h
struct OnDemandEvent {
    OnDemandEventType eventId;    // 事件类型
    std::string eventName;        // 事件名称
    std::string value;            // 事件值
    int64_t extraDataId = -1;     // 额外数据 ID
};

enum class OnDemandEventType {
    DEVICE_ONLINE = 0,
    DEVICE_OFFLINE,
    COMMON_EVENT,
    PARAM,
    TIMED_EVENT,
    SWITCH_EVENT,
};
```

## 线程安全机制

### 锁策略

```cpp
// 1. 读写锁 - abilityMap_ (多读单写)
samgr::shared_mutex abilityMapLock_;
{
    shared_lock<shared_mutex> readLock(abilityMapLock_);  // 读
    auto iter = abilityMap_.find(saId);
}
{
    unique_lock<shared_mutex> writeLock(abilityMapLock_); // 写
    abilityMap_[saId] = saInfo;
}

// 2. 互斥锁 - listenerMap_
samgr::mutex listenerMapLock_;
{
    lock_guard<mutex> autoLock(listenerMapLock_);
    listenerMap_[saId].push_back(listener);
}

// 3. Locked 方法约定 - 调用者必须持有锁
void RemoveRemoteCallbackLocked(...);  // 必须以 Locked 结尾
```

### 锁层级（防止死锁）

```
abilityMapLock_ (最外层)
    └── 禁止访问其他锁

listenerMapLock_
    └── 可以访问 onDemandLock_

onDemandLock_
    └── 可以访问 systemProcessMapLock_
```

## 内存管理

### 智能指针使用

| 类型 | 用途 | 说明 |
|------|------|------|
| `sptr<IRemoteObject>` | 远程对象引用 | 自动引用计数 |
| `sptr<ISystemAbilityStatusChange>` | 监听器引用 | 自动释放 |
| `std::shared_ptr<SystemAbilityStateScheduler>` | 状态调度器 | 共享所有权 |
| `std::unique_ptr<Utils::Timer>` | 定时器 | 独占所有权 |

### 死亡监听机制

```cpp
// services/samgr/native/source/ability_death_recipient.cpp
class AbilityDeathRecipient : public IRemoteObject::DeathRecipient {
public:
    void OnRemoteDied(const wptr<IRemoteObject>& remote) override {
        // 1. 从 abilityMap_ 移除
        // 2. 通知订阅者 OnRemoveSystemAbility
        // 3. 清理相关资源
    }
};
```

## 配置解析

### SA 配置文件

**路径**: `/system/profile/` (运行时)

**格式** (JSON):

```json
{
    "sa_profiles": [
        {
            "sa_id": 401,
            "process": "bundle_daemon",
            "name": "BundleManagerService",
            "distributed": false,
            "recycle_strategy": "app_recycle",
            "ondemand_events": [
                {
                    "event_id": "COMMON_EVENT",
                    "event_name": "usual.event.TIME_TICK",
                    "value": "start"
                }
            ]
        }
    ]
}
```

**解析流程**:

```
InitSaProfile()
    -> ParseSaProfiles()
        -> GetRealPath()  // 路径规范化
        -> CheckPathExist()  // 检查文件存在
        -> LoadSaProfilesFromJson()  // JSON 解析
            -> ParseSystemAbilityFromJson()  // 字段解析
                -> CheckRecycleStrategy()  // 策略校验
                -> CheckLogicRelationship()  // 逻辑校验
        -> InsertSaProfileToMap()  // 插入 saProfileMap_
```

## 错误处理

### 错误传播

```cpp
// 1. 参数校验层
if (!CheckInputSysAbilityId(saId)) {
    HILOGE("Invalid SA ID: %{public}d", saId);
    return INVALID_SYSTEM_ABILITY_ID;
}

// 2. 权限检查层
if (!CheckGetSAPermission()) {
    HILOGE("Permission denied for GetSA");
    return PERMISSION_DENIED;
}

// 3. 业务逻辑层
auto iter = abilityMap_.find(saId);
if (iter == abilityMap_.end()) {
    HILOGW("SA not found: %{public}d", saId);
    return SA_NOT_EXIST;
}

// 4. 成功返回
return ERR_OK;
```

### 日志级别

| 级别 | 用途 | 示例 |
|------|------|------|
| HILOGD | 调试信息 | 进入函数、变量值 |
| HILOGI | 关键流程 | 服务注册成功 |
| HILOGW | 警告 | 服务未找到 |
| HILOGE | 错误 | 参数无效、权限拒绝 |

## 性能优化

### 1. DynamicCache

```cpp
// 缓存 SA 查询结果，减少 IPC
class DynamicCache : public IRemoteObject::DeathRecipient {
    sptr<IRemoteObject> QueryResult(int32_t saId, int32_t code) {
        // 检查缓存
        if (cacheValid) return cachedObj;
        // 重新计算
        return Recompute(saId, code);
    }
};
```

### 2. FFRT 任务调度

```cpp
// 使用 FFRT 处理异步任务
std::shared_ptr<FFRTHandler> workHandler_;
workHandler_->PostTask([this, saId]() {
    // 异步执行
    SendSystemAbilityAddedMsg(saId);
});
```

### 3. 读写锁分离

```cpp
// abilityMap_ 使用 shared_mutex
// 读操作可以并发，写操作独占
```
