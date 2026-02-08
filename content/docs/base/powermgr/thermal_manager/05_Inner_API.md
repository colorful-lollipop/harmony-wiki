# 内部 API（Inner API）文档

## 目的

本文档说明 thermal_manager 内部使用的 API 接口，包括模块接口、依赖方向、稳定性和可替换点。

## 适用范围

- OpenHarmony thermal_manager 模块
- 内部 C/C++ API
- 模块间通信接口

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Module_Boundaries.md](01_Module_Boundaries.md) - 模块边界
- [03_Architecture.md](03_Architecture.md) - 架构设计

---

## 内部 API 分类

### 1. ThermalMgrClient API

**文件**: `interfaces/inner_api/native/include/thermal_mgr_client.h:26`

#### 客户端类

```cpp
class ThermalMgrClient final : public DelayedRefSingleton<ThermalMgrClient> {
    DECLARE_DELAYED_REF_SINGLETON(ThermalMgrClient)

public:
    DISALLOW_COPY_AND_MOVE(ThermalMgrClient);

    // 温度回调 API
    bool SubscribeThermalTempCallback(
        const std::vector<std::string>& typeList, const sptr<IThermalTempCallback>& callback);
    bool UnSubscribeThermalTempCallback(const sptr<IThermalTempCallback>& callback);

    // 热级别回调 API
    bool SubscribeThermalLevelCallback(const sptr<IThermalLevelCallback>& callback);
    bool UnSubscribeThermalLevelCallback(const sptr<IThermalLevelCallback>& callback);

    // 动作回调 API
    bool SubscribeThermalActionCallback(
        const std::vector<std::string>& actionList, const std::string& desc,
        const sptr<IThermalActionCallback>& callback);
    bool UnSubscribeThermalActionCallback(const sptr<IThermalActionCallback>& callback);

    // 查询 API
    int32_t GetThermalSensorTemp(const SensorType type);
    ThermalLevel GetThermalLevel();
    bool SetScene(const std::string& scene);
    bool UpdateThermalPolicy();
    bool UpdateThermalState(const std::string& tag, const std::string& val, bool isImmed = false);
    std::string Dump(const std::vector<std::string>& args);

#ifndef THERMAL_SERVICE_DEATH_UT
private:
#endif
    // 内部实现
    ErrCode Connect();
    void ResetProxy(const wptr<IRemoteObject>& remote);
    void GetLevel(ThermalLevel& level);
    bool GetThermalSrvSensorInfo(const SensorType& type, ThermalSrvSensorInfo& sensorInfo);
    sptr<IThermalSrv> thermalSrv_;
    sptr<IRemoteObject::DeathRecipient> deathRecipient_;
    std::mutex mutex_;
};
```

#### 稳定性

| 接口 | 稳定性 | 说明 |
|---|---|---|
| 客户端 API | 稳定 | 对外提供，内部使用 |
| IPC 代理 | 内部 | 实现细节可修改 |

#### 依赖方向

```
N-API / 其他模块
        ↓
ThermalMgrClient (内部客户端)
        ↓ (IPC Binder)
ThermalService (SA 3303)
```

---

### 2. 回调接口

#### IThermalLevelCallback

**文件**: `interfaces/inner_api/native/include/ithermal_level_callback.h:25`

```cpp
class IThermalLevelCallback : public IRemoteBroker {
public:
    virtual bool OnThermalLevelChanged(ThermalLevel level) = 0;
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.powermgr.IThermalLevelCallback");
};
```

**稳定性**: 稳定 - IPC 接口定义

#### IThermalTempCallback

**文件**: `interfaces/inner_api/native/include/ithermal_temp_callback.h:25`

```cpp
class IThermalTempCallback : public IRemoteBroker {
public:
    using TempCallbackMap = std::map<std::string, int32_t>;

    virtual bool OnThermalTempChanged(TempCallbackMap& tempCbMap) = 0;
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.powermgr.IThermalTempCallback");
};
```

**稳定性**: 稳定 - IPC 接口定义

#### IThermalActionCallback

**文件**: (从代码推断，services/zidl/include/)

**稳定性**: 稳定 - IPC 接口定义

---

### 3. ThermalService 接口

**文件**: `services/native/include/thermal_service.h:56`

#### 对外接口

| 方法 | 稳定性 | 说明 |
|---|---|---|
| `SubscribeThermalTempCallback()` | 稳定 | SA IPC 接口 |
| `UnSubscribeThermalTempCallback()` | 稳定 | SA IPC 接口 |
| `SubscribeThermalLevelCallback()` | 稳定 | SA IPC 接口 |
| `UnSubscribeThermalLevelCallback()` | 稳定 | SA IPC 接口 |
| `SubscribeThermalActionCallback()` | 稳定 | SA IPC 接口 |
| `UnSubscribeThermalActionCallback()` | 稳定 | SA IPC 接口 |
| `GetThermalSrvSensorInfo()` | 稳定 | SA IPC 接口 |
| `GetThermalLevel()` | 稳定 | SA IPC 接口 |
| `GetThermalInfo()` | 稳定 | SA IPC 接口 |
| `SetScene()` | 稳定 | SA IPC 接口 |
| `UpdateThermalState()` | 稳定 | SA IPC 接口 |
| `ShellDump()` | 稳定 | SA IPC 接口 |
| `Dump()` | 稳定 | SA Dump 接口 |

#### 对外接口获取器

**文件**: `services/native/include/thermal_service.h:98-136`

```cpp
std::shared_ptr<ThermalConfigBaseInfo> GetBaseinfoObj() const
{
    return baseInfo_;
}
std::shared_ptr<StateMachine> GetStateMachineObj() const
{
    return state_;
}
std::shared_ptr<ThermalActionManager> GetActionManagerObj() const
{
    return actionMgr_;
}
std::shared_ptr<ThermalPolicy> GetPolicy() const
{
    return policy_;
}
std::shared_ptr<ThermalObserver> GetObserver() const
{
    return observer_;
}
std::shared_ptr<ThermalSensorInfo> GetSensorInfo() const
{
    return info_;
}
std::shared_ptr<ThermalServiceSubscriber> GetSubscriber() const
{
    return serviceSubscriber_;
}
sptr<IThermalInterface> GetThermalInterface() const
{
    return thermalInterface_;
}
```

**稳定性**: 内部 API，不对外

---

### 4. 内部模块接口

#### ThermalObserver 接口

**文件**: `services/native/include/thermal_observer/thermal_observer.h:34`

**稳定接口**:

| 方法 | 稳定性 |
|---|---|
| `OnReceivedSensorInfo()` | 内部，可替换 |
| `SubscribeThermalTempCallback()` | 内部，可替换 |
| `UnSubscribeThermalTempCallback()` | 内部，可替换 |
| `SubscribeThermalActionCallback()` | 内部，可替换 |
| `UnSubscribeThermalActionCallback()` | 内部，可替换 |
| `SetRegisterCallback()` | 内部，可替换 |

#### ThermalPolicy 接口

**文件**: `services/native/include/thermal_policy/thermal_policy.h:40`

**稳定接口**:

| 方法 | 稳定性 |
|---|---|
| `OnSensorInfoReported()` | 内部，可替换 |
| `ExecutePolicy()` | 内部，可替换 |
| `SetPolicyMap()` | 内部，可替换 |
| `SetSensorClusterMap()` | 内部，可替换 |

#### ThermalActionManager 接口

**文件**: `services/native/include/thermal_action/thermal_action_manager.h:38`

**稳定接口**:

| 方法 | 稳定性 |
|---|---|
| `SubscribeThermalLevelCallback()` | 内部，可替换 |
| `UnSubscribeThermalLevelCallback()` | 内部，可替换 |
| `GetThermalLevel()` | 内部，可替换 |
| `SetActionItem()` | 内部，可替换 |
| `EnableMock()` | 测试接口 |

---

### 5. Action 接口

**文件**: `services/native/include/thermal_action/ithermal_action.h` (推测)

**稳定接口**:

| 方法 | 稳定性 |
|---|---|
| `Execute()` | 内部，可替换 |
| `Init()` | 内部，可替换 |

**具体动作实现**:
- `ActionCpuBig` - CPU big core 控制
- `ActionCpuMed` - CPU medium core 控制
- `ActionCpuLit` - CPU little core 控制
- `ActionGpu` - GPU 控制
- `ActionVoltage` - 电压控制
- `ActionCharger` - 充电控制
- `ActionDisplay` - 显示控制
- `ActionVolume` - 音量控制
- `ActionShutdown` - 关机动作

**稳定性**: 内部，可替换

---

### 6. StateMachine 接口

**文件**: `services/native/include/thermal_observer/state_machine/` (多个状态收集类)

**稳定接口**:

| 方法 | 稳定性 |
|---|---|
| `Init()` | 内部，可替换 |
| `UpdateState()` | 内部，可替换 |

**状态收集类**:
- `ScreenStateCollection` - 屏幕状态
- `ChargerStateCollection` - 充电状态
- `SceneStateCollection` - 场景状态
- `StartupDelayStateCollection` - 启动延迟

**稳定性**: 内部，可替换

---

### 7. ConfigParser 接口

**文件**: `services/native/include/thermal_policy/thermal_srv_config_parser.h`

**稳定接口**:

| 方法 | 稳定性 |
|---|---|
| `ThermalSrvConfigInit()` | 内部，可替换 |
| `ParseXmlFile()` | 内部，可替换 |

---

## 依赖方向

```
┌──────────────────────────────────────────────┐
│           N-API / 其他模块           │
└─────────────┬─────────────────────────────┘
              │ (单向依赖)
              ↓
┌──────────────────────────────────────────────┐
│        ThermalMgrClient (内部 API)        │
│  ┌──────────────────────────────────┐    │
│  │ 连接到 ThermalService          │    │
│  └──────────────────────────────────┘    │
└─────────────┬─────────────────────────────┘
              │ (IPC Binder)
              ↓
┌──────────────────────────────────────────────┐
│      ThermalService (SA 3303)            │
│  ┌─────────┬───────────┬────────────┐   │
│  │Observer │  Policy    │  ActionMgr  │   │
│  └─────────┴───────────┴────────────┘   │
│     (单向内部调用)                     │
└──────────────────────────────────────────────┘
```

**依赖规则**:
- ✅ 无环依赖：上层依赖下层，下层不依赖上层
- ✅ 单向依赖：ThermalMgrClient → ThermalService → 内部模块
- ✅ 接口隔离：内部模块通过 ThermalService 协调

---

## 可替换点

### 可替换的模块

| 模块 | 可替换性 | 替换点 |
|---|---|---|
| ThermalObserver | ✅ | 实现类可替换 |
| ThermalPolicy | ✅ | 策略算法可替换 |
| ThermalActionManager | ✅ | 动作集合可替换 |
| StateMachine | ✅ | 状态逻辑可替换 |
| 具体动作 | ✅ | 每个动作可独立替换 |

### 不可替换的点

| 模块 | 不可替换性 | 原因 |
|---|---|---|
| ThermalService 主类 | ❌ | SA 入口，固定接口 |
| IPC 接口定义 | ❌ | Binder 固定协议 |
| N-API 导出 | ❌ | Node.js 固定模块名 |

---

## 稳定性标注

### 稳定接口

这些接口对其他模块是稳定的，不应随意修改：

1. **ThermalMgrClient 公共 API** - N-API 和其他 Native 模块使用
2. **IPC 回调接口** - IThermalLevelCallback, IThermalTempCallback, IThermalActionCallback
3. **ThermalService 对外接口** - SA 提供的所有 IPC 方法

### 内部接口

这些接口仅 thermal_manager 内部使用，可随需求修改：

1. **ThermalObserver 方法** - 内部模块调用
2. **ThermalPolicy 方法** - 策略实现可替换
3. **ThermalActionManager 方法** - 动作管理可替换
4. **具体 Action 类** - 各动作实现独立

---

## 数据结构

### ThermalLevel 定义

**文件**: `interfaces/inner_api/native/include/thermal_level_info.h`

```cpp
enum class ThermalLevel : uint32_t {
    COOL = 0,
    NORMAL = 1,
    WARM = 2,
    HOT = 3,
    OVERHEATED = 4,
    WARNING = 5,
    EMERGENCY = 6,
    ESCAPE = 7
};
```

### SensorType 定义

**文件**: `interfaces/inner_api/native/include/thermal_srv_sensor_info.h`

```cpp
enum class SensorType : uint32_t {
    SOC = 0,
    BATTERY = 1,
    SHELL = 2,
    AMBIENT = 3,
    PA = 4,
    // ... 更多传感器类型
};
```

---

## 总结

Thermal Manager 内部 API 清晰分为三层：

1. **客户端层** (ThermalMgrClient) - 对外提供稳定接口
2. **服务层** (ThermalService) - SA 主入口，协调各模块
3. **实现层** (Observer/Policy/Action) - 内部模块，可替换

**设计原则**:
- ✅ 接口稳定：对外接口保持兼容
- ✅ 依赖单向：上层依赖下层，无环
- ✅ 可扩展性：内部模块可替换
- ✅ 线程安全：使用互斥锁保护共享资源
