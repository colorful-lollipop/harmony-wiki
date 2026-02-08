# 架构说明

## 目的

本文档详细说明 thermal_manager 的架构设计，包括组件图、数据流、线程模型和关键时序。

## 适用范围

- OpenHarmony thermal_manager 模块
- 标准系统类型

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Module_Boundaries.md](01_Module_Boundaries.md) - 模块边界
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构

---

## 组件架构图

```mermaid
graph TB
    subgraph "应用层"
        APP[JS/ArkTS 应用]
        NAPI[N-API Layer<br/>thermal.so]
    end

    subgraph "IPC 通信层"
        CLIENT[ThermalMgrClient]
        PROXY[IPC Proxy]
        STUB[IPC Stub]
    end

    subgraph "服务层 - Thermal Service SA:3303"
        OBSERVER[ThermalObserver<br/>温度监控]
        POLICY[ThermalPolicy<br/>策略决策]
        ACTION[ThermalActionManager<br/>动作执行]
        STATE[StateMachine<br/>状态管理]
        DUMPER[ThermalMgrDumper<br/>调试接口]
    end

    subgraph "硬件层"
        HDI[HDI Interface<br/>thermal_interface_service]
        DRIVERS[Thermal Drivers<br/>温度传感器/风扇]
    end

    APP -->|调用| NAPI
    NAPI -->|IPC Binder| CLIENT
    CLIENT --> PROXY
    PROXY --> STUB

    STUB --> OBSERVER
    STUB --> POLICY
    STUB --> ACTION
    STUB --> STATE
    STUB --> DUMPER

    OBSERVER -->|温度上报| POLICY
    POLICY -->|策略决策| ACTION
    STATE -->|状态通知| POLICY

    OBSERVER -->|HDI 回调| HDI
    HDI -->|驱动指令| DRIVERS
    DRIVERS -->|温度数据| HDI
    HDI --> OBSERVER

    style APP fill:#e1f5fe
    style NAPI fill:#ff9800
    style HDI fill:#ff6b6b
    style DRIVERS fill:#4caf50
```

---

## 数据流

### 1. 温度监控流程

```mermaid
sequenceDiagram
    participant APP as JS 应用
    participant NAPI as N-API
    participant CLIENT as ThermalMgrClient
    participant SA as ThermalService
    participant OBSERVER as ThermalObserver
    participant HDI as Thermal HDI
    participant DRIVERS as 温度传感器

    APP->>NAPI: registerThermalLevelCallback(callback)
    NAPI->>CLIENT: SubscribeThermalLevelCallback(callback)
    CLIENT->>SA: SubscribeThermalLevelCallback(callback)
    SA->>OBSERVER: SubscribeThermalLevelCallback(callback)

    Note over DRIVERS: 传感器持续监测温度
    DRIVERS->>HDI: 定期上报温度
    HDI->>SA: 回调 OnThermalInfo(info)
    SA->>OBSERVER: OnReceivedSensorInfo(tempMap)
    OBSERVER->>POLICY: OnSensorInfoReported(tempMap)
    POLICY->>POLICY: LevelDecision()
    POLICY->>POLICY: ActionDecision()
    POLICY->>ACTION: SetActionItem(actionList)
    ACTION->>OBSERVER: 回调 OnThermalLevelChanged(level)
    OBSERVER->>SA: OnThermalLevelChanged(level)
    SA->>CLIENT: OnThermalLevelChanged(level)
    CLIENT->>NAPI: 回调 JS function
    NAPI->>APP: callback(level)
```

### 2. 策略执行流程

```mermaid
graph LR
    subgraph "输入"
        TEMP[温度传感器数据]
        STATE[设备状态<br/>屏幕/充电/场景]
        CONFIG[配置文件<br/>thermal_service_config.xml]
    end

    subgraph "决策"
        CLUSTER[传感器集群<br/>SensorCluster]
        LEVEL[级别决策<br/>LevelDecision]
        POLICY[策略决策<br/>PolicyDecision]
    end

    subgraph "执行"
        CPU[CPU降频]
        GPU[GPU降频]
        VOLT[电压限制]
        CURR[电流限制]
        DISPLAY[显示控制]
        POPUP[弹窗警告]
    end

    TEMP --> CLUSTER
    STATE --> LEVEL
    CONFIG --> POLICY

    CLUSTER --> LEVEL
    LEVEL --> POLICY
    POLICY --> CPU
    POLICY --> GPU
    POLICY --> VOLT
    POLICY --> CURR
    POLICY --> DISPLAY
    POLICY --> POPUP
```

---

## 线程模型

### 主线程

| 组件 | 线程模型 | 说明 |
|---|---|---|
| Thermal Service | 主线程 | SA 回调在主线程执行 |
| Policy 执行 | 主线程同步 | ExecutePolicy() 同步执行 |
| IPC 回调 | 主线程 | IRemoteBroker 回调在主线程 |

### 工作队列

| 组件 | 队列类型 | 用途 |
|---|---|---|
| 全局队列 | FFRTQueue | thermal_service 工作队列 |
| 证据 | `FFRTQueue g_queue("thermal_service")` | services/native/src/thermal_service.cpp:58 |

### 异步回调

| 组件 | 异步方式 | 说明 |
|---|---|---|
| N-API 回调 | napi_send_event | JS 回调异步执行 |
| 证据 | `napi_send_event(env_, uvcallback, napi_eprio_low, __func__)` | frameworks/napi/thermal_manager_napi.cpp:77 |

---

## 关键时序

### 1. 服务启动时序

```mermaid
sequenceDiagram
    participant INIT as Init Process
    participant SA as ThermalService
    participant CONFIG as ConfigParser
    participant OBSERVER as ThermalObserver
    participant POLICY as ThermalPolicy
    participant ACTION as ActionManager
    participant HDI as Thermal HDI
    participant SAMGR as SystemAbilityManager

    Note over SA: SystemAbility 构造
    INIT->>SA: 构造 ThermalService(SA_ID=3303)
    SA->>SAMGR: Publish(ThermalService)
    SAMGR->>INIT: OnStart()
    INIT->>SA: Init()

    par 初始化模块
        INIT->>CONFIG: CreateConfigModule()
        INIT->>CONFIG: InitConfigFile()
        CONFIG->>CONFIG: ParseXmlFile(thermal_service_config.xml)
        CONFIG-->>SA: baseInfo_, state_, policy_, actionMgr_ 初始化

        INIT->>OBSERVER: InitThermalObserver()
        OBSERVER->>OBSERVER: InitSensorTypeMap()
        OBSERVER-->>SA: observer_ 初始化

        INIT->>POLICY: InitThermalPolicy()
        POLICY->>POLICY: SetPolicyMap()
        POLICY->>POLICY: SetSensorClusterMap()
        POLICY-->>SA: policy_ 初始化

        INIT->>ACTION: InitActionManager()
        ACTION->>ACTION: 初始化所有 action 对象
        ACTION-->>SA: actionMgr_ 初始化
    end

    SA->>HDI: RegisterHdiStatusListener()
    HDI->>SAMGR: RegisterServiceStatusListener()

    Note over SA,HDI: HDI 服务就绪
    HDI->>SA: OnHdiServiceStatusChanged(STARTED)
    SA->>HDI: RegisterThermalHdiCallback()
    SA->>HDI: RegisterFanHdiCallback()

    SA->>SAMGR: Publish(ready)
    SA-->>INIT: Init() 完成
```

### 2. 温度变化处理时序

```mermaid
sequenceDiagram
    participant HDI as Thermal HDI
    participant CB as ThermalCallback
    participant SA as ThermalService
    participant OBSERVER as ThermalObserver
    participant POLICY as ThermalPolicy
    participant ACTION as ActionManager
    participant ACTION as Thermal Actions

    HDI->>CB: 温度传感器数据变化
    CB->>SA: OnThermalCallback(info)

    SA->>OBSERVER: OnReceivedSensorInfo(typeTempMap)

    par 并行处理
        OBSERVER->>POLICY: OnSensorInfoReported(tempMap)
        POLICY->>POLICY: 更新 typeTempMap_
        POLICY->>POLICY: SortLevel()
        POLICY->>POLICY: FindSubscribeActionValue()
        POLICY->>POLICY: LevelDecision()
        POLICY->>POLICY: PolicyDecision()
        POLICY->>POLICY: ActionDecision(actionList)
    end

    POLICY->>ACTION: SetActionItem(actionList)
    ACTION->>ACTION: 遍历 actionList 执行

    par 动作执行
        ACTION->>CPU: 执行 CPU 降频
        ACTION->>GPU: 执行 GPU 降频
        ACTION->>VOLT: 执行电压限制
        ACTION->>CURR: 执行电流限制
    end
```

### 3. 回调通知时序

```mermaid
sequenceDiagram
    participant APP as JS 应用
    participant NAPI as N-API
    participant LEVEL_CB as ThermalLevelCallback
    participant CLIENT as ThermalMgrClient
    participant SA as ThermalService
    participant STUB as ThermalLevelCallbackStub
    participant PROXY as ThermalLevelCallbackProxy

    APP->>NAPI: registerThermalLevelCallback(jsCallback)
    NAPI->>LEVEL_CB: UpdateCallback(env, jsCallback)
    LEVEL_CB->>LEVEL_CB: 保存 napi_ref callbackRef_
    LEVEL_CB->>CLIENT: SubscribeThermalLevelCallback(this)
    CLIENT->>SA: SubscribeThermalLevelCallback(callback)

    Note over SA: 订阅者列表
    SA->>STUB: 保存到 sensorTempListeners_

    Note over SA,ACTION: 热级别变化
    ACTION->>STUB: OnThermalLevelChanged(level)
    STUB->>PROXY: 回调客户端

    PROXY->>CLIENT: OnThermalLevelChanged(level)
    CLIENT->>LEVEL_CB: OnThermalLevelChanged(level)

    Note over LEVEL_CB: 异步通知 JS
    LEVEL_CB->>LEVEL_CB: 创建 uv_work_t
    LEVEL_CB->>NAPI: napi_send_event(callback, level)
    NAPI->>APP: jsCallback(level)
```

---

## 组件通信

### IPC 通信

| 通信方向 | 接口 | 协议 | 说明 |
|---|---|---|---|
| 应用 → Service | IThermalSrv | Binder | N-API 通过 ThermalMgrClient 调用 |
| Service → 应用 | IThermalLevelCallback | Binder | 回调通知应用 |
| 应用 → Service | IThermalTempCallback | Binder | 温度变化订阅 |
| 应用 → Service | IThermalActionCallback | Binder | 动作变化订阅 |

### HDI 通信

| 通信方向 | 接口 | 说明 |
|---|---|---|
| Service → Driver | IThermalInterface | HDF | 温度查询、指令下发 |
| Driver → Service | IThermalCallback | HDF | 温度数据上报 |

---

## 数据结构

### 1. 温度数据

```cpp
// services/native/include/thermal_srv_sensor_info.h
using TypeTempMap = std::map<std::string, int32_t>;

// services/native/include/ithermal_temp_callback.h
using TempCallbackMap = std::map<std::string, int32_t>;
```

### 2. 策略数据

```cpp
// services/native/include/thermal_policy/thermal_policy.h
struct PolicyAction {
    std::string actionName;
    std::string actionValue;
    std::map<std::string, std::string> actionPropMap;
    bool isProp;
};

struct PolicyConfig {
    uint32_t level;
    std::vector<PolicyAction> policyActionList;
};
```

### 3. 动作数据

```cpp
// services/native/include/thermal_action/thermal_action_manager.h
struct ActionItem {
    std::string name;
    std::string params;
    std::string uid;
    std::string protocol;
    bool strict;
    bool enableEvent;
};
```

---

## 错误处理

### 错误码

| 错误码 | 值 | 含义 |
|---|---|---|
| ERR_OK | 0 | 成功 |
| ERR_FAIL | -1 | 失败 |
| ERR_NO_INIT | -2 | 未初始化 |
| ERR_PERMISSION_DENIED | -3 | 权限拒绝 |

**证据**: services/native/src/thermal_service.cpp:60, 441, 448, 461, 466, 472, 475, 505, 519, 525, 533, 541, 554, 563, 576, 586, 634, 639, 668, 709, 711, 732, 738, 753, 756, 769, 774

---

## 总结

Thermal Manager 采用经典的分层架构，通过 IPC 和 HDI 实现跨进程和跨层通信。

**关键特点**:
- ✅ 清晰的分层：N-API → Service → HDI → Driver
- ✅ 观察者模式：Observer 监控温度，通知 Policy 和 Action
- ✅ 策略模式：基于配置和状态进行决策
- ✅ 回调机制：通过 Binder 回调通知订阅者
- ✅ 线程安全：使用互斥锁保护共享资源
- ✅ 异步通知：使用 napi_send_event 避免 JS 阻塞
