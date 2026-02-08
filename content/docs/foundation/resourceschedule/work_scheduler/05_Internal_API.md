# 内部 API 说明

> **目的**: 列出 Work Scheduler 模块的所有内部接口和依赖关系
> **适用范围**: 模块开发者、系统开发者、跨模块集成者

---

## 模块接口总览

### Frameworks 层接口

#### WorkSchedulerSrvClient

**文件**: `frameworks/include/workscheduler_srv_client.h`

| 方法 | 说明 | 稳定性 |
|------|------|--------|
| `GetInstance()` | 获取单例实例 | 🟢 稳定 |
| `Connect()` | 连接 SA 1904 | 🟢 稳定 |
| `ResetProxy()` | 重置代理（死亡回调） | 🟢 稳定 |
| `StartWork(WorkInfo)` | 启动任务 | 🟢 稳定 |
| `StopWork(WorkInfo)` | 停止任务 | 🟢 稳定 |
| `StopAndCancelWork(WorkInfo)` | 停止并取消任务 | 🟢 稳定 |
| `GetWorkStatus(workId, WorkInfo)` | 获取任务状态 | 🟢 稳定 |
| `ObtainAllWorks(list)` | 获取所有任务 | 🟢 稳定 |
| `StopAndClearWorks()` | 清空任务 | 🟢 稳定 |
| `IsLastWorkTimeout(workId, bool)` | 检查超时 | 🟢 稳定 |
| `GetAllRunningWorks(list)` | 获取所有运行中任务 | 🟡 中等（内部接口） |

**证据**: `frameworks/include/workscheduler_srv_client.h`

#### WorkInfo

**文件**: `frameworks/include/work_info.h`

| 方法 | 说明 | 稳定性 |
|------|------|--------|
| `GetBundleName()` | 获取包名 | 🟢 稳定 |
| `GetAbilityName()` | 获取 Ability 名称 | 🟢 稳定 |
| `GetWorkId()` | 获取任务 ID | 🟢 稳定 |
| `GetCondition()` | 获取条件 | 🟢 稳定 |
| `GetNetworkType()` | 获取网络类型 | 🟢 稳定 |
| `GetRepeatInfo()` | 获取重复信息 | 🟢 稳定 |
| `GetExtrasInfo()` | 获取额外参数 | 🟢 稳定 |

**证据**: `frameworks/include/work_info.h`

---

### Services 层接口

#### WorkSchedulerService (SA)

**文件**: `services/native/include/work_scheduler_service.h`

**继承关系**:
```cpp
class WorkSchedulerService final : public SystemAbility,
                                   public WorkSchedServiceStub,
                                   public std::enable_shared_from_this<WorkSchedulerService>
```

| 公共方法 | 说明 | 稳定性 |
|---------|------|--------|
| `OnStart()` | SA 启动 | 🟢 稳定 |
| `OnStop()` | SA 停止 | 🟢 稳定 |
| `OnAddSystemAbility()` | 依赖 SA 启动回调 | 🟢 稳定 |
| `OnRemoveSystemAbility()` | 依赖 SA 停止回调 | 🟢 稳定 |

| 服务方法 (来自 IDL) | 说明 | 稳定性 |
|---------------------|------|--------|
| `StartWork(WorkInfo)` | 启动任务 | 🟢 稳定 |
| `StartWorkForInner(WorkInfo)` | 内部启动任务 | 🟡 内部接口 |
| `StopWork(WorkInfo)` | 停止任务 | 🟢 稳定 |
| `StopWorkForInner(WorkInfo, bool)` | 内部停止任务 | 🟡 内部接口 |
| `StopAndCancelWork(WorkInfo)` | 停止并取消 | 🟢 稳定 |
| `StopAndClearWorks()` | 清空任务 | 🟢 稳定 |
| `IsLastWorkTimeout(workId, bool)` | 检查超时 | 🟢 稳定 |
| `ObtainAllWorks(list)` | 获取所有任务 | 🟢 稳定 |
| `GetWorkStatus(workId, WorkInfo)` | 获取任务状态 | 🟢 稳定 |
| `GetAllRunningWorks(list)` | 获取运行中任务 | 🟡 内部接口 |
| `PauseRunningWorks(uid)` | 暂停任务 | 🟡 内部接口 |
| `ResumePausedWorks(uid)` | 恢复任务 | 🟡 内部接口 |
| `SetWorkSchedulerConfig(data, type)` | 设置配置 | 🟡 内部接口 |
| `StopWorkForSA(saId)` | 停止 SA 任务 | 🟡 内部接口 |

| 扩展方法 | 说明 | 稳定性 |
|---------|------|--------|
| `OnWorkStart(WorkInfo)` | Extension 回调通知 | 🟢 稳定 |
| `OnWorkStop(WorkInfo)` | Extension 回调通知 | 🟢 稳定 |

**证据**: `services/native/include/work_scheduler_service.h`

#### WorkQueueManager

**文件**: `services/native/include/work_queue_manager.h`

| 方法 | 说明 | 稳定性 |
|------|------|--------|
| `AddWork(WorkInfo)` | 添加任务 | 🟢 稳定 |
| `RemoveWork(WorkInfo)` | 移除任务 | 🟢 稳定 |
| `PauseWorks(uid)` | 暂停任务 | 🟢 稳定 |
| `ResumeWorks(uid)` | 恢复任务 | 🟢 稳定 |
| `TriggerWork(WorkInfo)` | 触发执行 | 🟢 稳定 |
| `GetWorkInfo(workId)` | 获取任务 | 🟢 稳定 |
| `GetAllWorks(list)` | 获取所有任务 | 🟢 稳定 |
| `GetRunningWorksCount()` | 获取运行中数量 | 🟢 稳定 |

**证据**: `services/native/include/work_queue_manager.h:11-37`

#### WorkPolicyManager

**文件**: `services/native/include/work_policy_manager.h`

| 方法 | 说明 | 稳定性 |
|------|------|--------|
| `AddPolicyListener(listener)` | 添加策略监听器 | 🟢 稳定 |
| `RemovePolicyListener(listener)` | 移除策略监听器 | 🟢 稳定 |
| `CheckAllPolicies(workInfo)` | 检查所有策略 | 🟢 稳定 |

**证据**: `services/native/include/work_policy_manager.h:11`

#### ConditionChecker

**文件**: `services/native/include/conditions/condition_checker.h`

| 方法 | 说明 | 稳定性 |
|------|------|--------|
| `CheckCondition(WorkInfo)` | 检查条件是否满足 | 🟢 稳定 |
| `UpdateCondition(conditionType, value)` | 更新条件状态 | 🟢 稳定 |
| `RegisterConditionListener(listener)` | 注册条件监听器 | 🟢 稳定 |

**证据**: `services/native/include/conditions/condition_checker.h`

#### 条件监听器接口

| 监听器 | 文件 | 主要方法 | 稳定性 |
|---------|------|---------|--------|
| `NetworkListener` | `network_listener.h` | `CheckNetworkType()` | 🟢 稳定 |
| `BatteryLevelListener` | `battery_level_listener.h` | `CheckBatteryLevel()` | 🟢 稳定 |
| `BatteryStatusListener` | `battery_status_listener.h` | `CheckBatteryStatus()` | 🟢 稳定 |
| `ChargerListener` | `charger_listener.h` | `CheckChargingType()` | 🟢 稳定 |
| `StorageListener` | `storage_listener.h` | `CheckStorageRequest()` | 🟢 稳定 |
| `ScreenListener` | `screen_listener.h` | `CheckScreenStatus()` | 🟢 稳定 |
| `TimerListener` | `timer_listener.h` | `CheckTimerInfo()` | 🟢 稳定 |
| `GroupListener` | `group_listener.h` | `OnAppGroupChanged()` | 🟢 稳定 |

#### 策略过滤器接口

| 过滤器 | 文件 | 主要方法 | 稳定性 |
|---------|------|---------|--------|
| `CpuPolicy` | `cpu_policy.h` | `Check()` | 🟢 稳定 |
| `MemoryPolicy` | `memory_policy.h` | `Check()` | 🟢 稳定 |
| `ThermalPolicy` | `thermal_policy.h` | `Check()` | 🟢 稳定 |
| `PowerModePolicy` | `power_mode_policy.h` | `Check()` | 🟢 稳定 |

---

### Utils 层接口

#### WorkSchedUtils

**文件**: `utils/native/include/work_sched_utils.h`

| 方法 | 说明 | 稳定性 |
|------|------|--------|
| `IsSystemApp()` | 判断是否为系统应用 | 🟢 稳定 |
| `CheckExtensionInfos(bundleName, abilityName, uid)` | 检查 Extension 信息 | 🟢 稳定 |
| `GetAppIndexAndBundleNameByUid(uid)` | 获取应用信息 | 🟢 稳定 |
| `GetUidByBundleName(bundleName)` | 获取 UID | 🟢 稳定 |

**证据**: `utils/native/include/work_sched_utils.h:112-161`

---

## 接口稳定性说明

### 稳定接口（建议使用）

| 接口类型 | 说明 | 约束 |
|-----------|------|------|
| **公共 N-API** | 对外 JS 接口 | 大版本间保持稳定 |
| **IPC IDL** | HIDL/ZIDL 接口 | 跨模块通信协议 |
| **SA SystemAbility** | 系统服务接口 | 版本管理严格 |
| **常量和宏** | 工具函数 | 通常稳定 |

### 不稳定接口（谨慎使用）

| 接口类型 | 说明 | 风险 |
|-----------|------|------|
| **内部 C++ 接口** | 服务端实现方法 | 可能在重构时变化 |
| **测试接口** | 测试专用方法 | 可能不适用生产环境 |
| **Feature Flag 接口** | 条件编译的方法 | 行为依赖配置 |

---

## 依赖关系

### 框架层依赖

```
Frameworks/
├── workschedclient
│   ├── 依赖: workschedutils (工具库）
│   └── 依赖: IPC, SAMGR, Ability Base
├── work_sched_service_proxy
│   ├── 依赖: workschedutils (工具库）
│   └── 生成自: work_sched_service_interface (IDL）
└── work_sched_service_stub
    ├── 依赖: workschedutils (工具库）
    └── 生成自: work_sched_service_interface (IDL）
```

### 服务层依赖

```
Services/
├── workschedservice
│   ├── 依赖: workschedclient (客户端库）
│   ├── 依赖: workschedservice_zidl_proxy (Extension Proxy）
│   ├── 依赖: workschedutils (工具库）
│   ├── 依赖: [条件监听器] (内部）
│   ├── 依赖: [策略过滤器] (内部）
│   ├── 依赖: [外部服务]
│   │   ├── Ability Runtime
│   │   ├── Bundle Manager
│   │   ├── Common Event Service
│   │   ├── Device Usage Statistics
│   │   ├── Device Standby
│   │   ├── Background Task Manager
│   │   ├── Battery Manager
│   │   ├── Thermal Manager
│   │   ├── Power Manager
│   │   └── Network Manager
│   └── 依赖: FFRT, Event Handler
└── workschedservice_static
    └── 与 workschedservice 相同依赖（静态版本）
```

### 接口层依赖

```
Interfaces/kits/
├── workscheduler (N-API)
│   ├── 依赖: workschedclient (框架客户端）
│   └── 依赖: workschedutils (工具库）
├── cj_work_scheduler_ffi (Cangjie FFI）
│   ├── 依赖: workschedclient (框架客户端）
│   └── 依赖: workschedutils (工具库）
└── [ArkTS/Taihe 模块]
    ├── 依赖: workschedclient (框架客户端）
    └── 依赖: workschedutils (工具库）
```

---

## 可替换点

### 条件监听器扩展

**可扩展点**: 条件监听器列表

**添加新条件监听器**:
1. 创建新的监听器类（继承 `IConditionListener`）
2. 在 `ConditionChecker` 中注册监听器
3. 在 `work_policy_manager.h` 中添加对应的过滤策略
4. 在 `services/BUILD.gn` 中添加到 sources

**示例**: 添加新的网络状态监听器
```cpp
// services/native/include/conditions/network_listener.h
class MyCustomNetworkListener : public IConditionListener {
    void OnNetworkStateChanged(NetworkType type) override;
};

// services/native/src/conditions/network_listener.cpp
MyCustomNetworkListener::OnNetworkStateChanged(NetworkType type) {
    ConditionChecker::UpdateCondition(ConditionType::NETWORK, type);
}
```

### 策略过滤器扩展

**可扩展点**: 策略过滤器列表

**添加新策略过滤器**:
1. 创建新的策略类（继承 `IPolicyFilter`）
2. 在 `WorkPolicyManager` 中注册策略
3. 在 `work_policy_manager.h` 中声明策略类型
4. 在 `services/BUILD.gn` 中添加到 sources

**示例**: 添加新的应用状态过滤策略
```cpp
// services/native/include/policy/app_status_policy.h
class AppStatusPolicy : public IPolicyFilter {
    bool Check(const WorkInfo& workInfo) override;
};

// services/native/src/policy/app_status_policy.cpp
bool AppStatusPolicy::Check(const WorkInfo& workInfo) {
    // 自定义逻辑
}
```

---

## 接口使用建议

### 选择稳定接口

**推荐优先级**:
1. ✅ **N-API 接口**（JS/TS）- 对外稳定性最高
2. ✅ **IDL 接口**（HIDL/ZIDL）- 跨模块通信标准
3. 🟡 **公共 C++ 头文件** - 需评估后使用
4. 🟢 **内部实现方法** - 仅限同模块使用
5. ❌ **条件编译方法** - 仅用于内部适配

### 版本兼容性

**接口版本管理**:
- N-API 接口通过 `@ohos` 命名空间管理版本
- IDL 接口通过 HIDL/ZIDL 版本控制
- C++ 头文件通过宏定义版本

---

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 架构设计
- [06_GN_Targets.md](06_GN_Targets.md) - 构建系统
- [04_External_API.md](04_External_API.md) - 对外 API 文档

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| 客户端接口 | `frameworks/include/workscheduler_srv_client.h` |
| 服务接口 | `services/native/include/work_scheduler_service.h` |
| 队列管理接口 | `services/native/include/work_queue_manager.h` |
| 策略管理接口 | `services/native/include/work_policy_manager.h` |
| 条件检查接口 | `services/native/include/conditions/condition_checker.h` |
| 工具接口 | `utils/native/include/work_sched_utils.h` |
| 条件监听器接口 | `services/native/include/conditions/icondition_listener.h` |
| 策略过滤接口 | `services/native/include/policy/ipolicy_filter.h` |
