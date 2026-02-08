# 关键调用链

## 概述

本文档描述电池统计模块的关键调用链路，包括 API 调用流程、服务启动流程、事件处理流程等。

## 目录

- [N-API 调用链](#n-api-调用链)
- [服务启动流程](#服务启动流程)
- [IPC 通信流程](#ipc-通信流程)
- [事件处理流程](#事件处理流程)
- [数据统计流程](#数据统计流程)

---

## N-API 调用链

### getBatteryStats 调用链

```mermaid
sequenceDiagram
    participant JS as "JS/ArkTS 应用"
    participant NAPI as "N-API 层"
    participant Client as "BatteryStatsClient"
    participant IPC as "IPC Proxy/Stub"
    participant Service as "BatteryStatsService"
    participant Core as "BatteryStatsCore"

    JS->>NAPI: getBatteryStats(callback?)
    NAPI->>NAPI: Check argc <= 1
    alt callback 模式
        NAPI->>NAPI: StatsAsyncCallBack()
        NAPI->>NAPI: napi_create_async_work()
        NAPI->>Client: BatteryStatsClient::GetBatteryStats()
    else promise 模式
        NAPI->>NAPI: StatsPromise()
        NAPI->>NAPI: napi_create_async_work()
        NAPI->>Client: BatteryStatsClient::GetBatteryStats()
    end
    Client->>IPC: GetBatteryStatsIpc()
    IPC->>Service: OnRemoteRequest(COMMAND_GET_BATTERY_STATS_IPC)
    Service->>Core: GetBatteryStats()
    Core->>Core: 遍历所有 Entity 计算功耗
    Service-->>IPC: ParcelableBatteryStatsList
    IPC-->>Client: BatteryStatsInfoList
    Client-->>NAPI: BatteryStatsInfoList
    NAPI->>NAPI: CreateJSArray()
    alt callback 模式
        NAPI->>JS: callback(err, result)
    else promise 模式
        NAPI->>JS: resolve(result)
    end
```

**证据来源**：`frameworks/napi/src/battery_stats_module.cpp:33-55`

### getAppPowerValue 调用链

```mermaid
sequenceDiagram
    participant JS as "JS/ArkTS 应用"
    participant NAPI as "N-API 层"
    participant Client as "BatteryStatsClient"
    participant Service as "BatteryStatsService"
    participant Core as "BatteryStatsCore"
    participant Entity as "UidEntity"

    JS->>NAPI: getAppPowerValue(uid)
    NAPI->>NAPI: Check type is napi_number
    NAPI->>NAPI: napi_get_value_int32()
    NAPI->>Client: BatteryStatsClient::GetAppStatsMah(uid)
    Client->>Service: GetAppStatsMahIpc(uid)
    Service->>Core: GetAppStatsMah(uid)
    Core->>Entity: GetEntityPowerMah(uid)
    Entity-->>Core: power (mAh)
    Core-->>Service: power (mAh)
    Service-->>Client: power (mAh)
    Client-->>NAPI: power (mAh)
    NAPI->>NAPI: napi_create_double()
    NAPI->>JS: Promise.resolve(power)
```

**证据来源**：`frameworks/napi/src/battery_stats.cpp:140-150`

### getHardwareUnitPowerValue 调用链

```mermaid
sequenceDiagram
    participant JS as "JS/ArkTS 应用"
    participant NAPI as "N-API 层"
    participant Client as "BatteryStatsClient"
    participant Service as "BatteryStatsService"
    participant Core as "BatteryStatsCore"
    participant Entity as "ScreenEntity/WifiEntity..."

    JS->>NAPI: getHardwareUnitPowerValue(ConsumptionType)
    NAPI->>NAPI: Check type is napi_number
    NAPI->>NAPI: napi_get_value_int32() -> ConsumptionType
    NAPI->>Client: BatteryStatsClient::GetPartStatsMah(type)
    Client->>Service: GetPartStatsMahIpc(type)
    Service->>Core: GetPartStatsMah(type)
    Core->>Entity: GetEntityPowerMah(type)
    Entity-->>Core: power (mAh)
    Core-->>Service: power (mAh)
    Service-->>Client: power (mAh)
    Client-->>NAPI: power (mAh)
    NAPI->>NAPI: napi_create_double()
    NAPI->>JS: Promise.resolve(power)
```

---

## 服务启动流程

```mermaid
sequenceDiagram
    participant System as "SystemAbilityMgr"
    participant Service as "BatteryStatsService"
    participant Core as "BatteryStatsCore"
    participant Detector as "BatteryStatsDetector"
    participant Subscriber as "BatteryStatsSubscriber"
    participant Parser as "BatteryStatsParser"

    System->>Service: OnStart()
    Service->>Service: Init()
    Service->>Parser: Init()
    Parser->>Parser: Parse power_average.json
    Parser-->>Service: 配置加载完成
    Service->>Core: Init()
    Core->>Detector: Init()
    Detector->>Detector: Register HiSysEvent listener
    Service->>Subscriber: Init()
    Subscriber->>Subscriber: Subscribe common events
    Service->>System: Publish()
    System->>Service: Service ready
    Service->>Core: Start collecting stats
```

**证据来源**：`services/native/src/battery_stats_service.cpp:68-102`

---

## IPC 通信流程

### 客户端连接流程

```mermaid
sequenceDiagram
    participant Client as "BatteryStatsClient"
    participant SAM as "SystemAbilityManager"
    participant Service as "BatteryStatsService"

    Client->>Client: Connect()
    Client->>SAM: GetSystemAbilityManager()
    SAM-->>Client: SA Manager
    Client->>SAM: GetSystemAbility(3304)
    SAM-->>Client: RemoteObject
    Client->>Client: iface_cast<IBatteryStats>()
    Client->>Service: Register death recipient
    Service-->>Client: Connection established
```

**证据来源**：`frameworks/native/src/battery_stats_client.cpp:42-52`

### IPC 请求处理流程

```mermaid
sequenceDiagram
    participant Client as "BatteryStatsProxy"
    participant Stub as "BatteryStatsStub"
    participant Service as "BatteryStatsService"

    Client->>Stub: SendRequest(ipcCode, data, reply, option)
    Stub->>Stub: OnRemoteRequest(code, data, reply, option)
    Stub->>Stub: Dispatch to method by code
    alt code == COMMAND_GET_BATTERY_STATS_IPC
        Stub->>Service: GetBatteryStats()
    alt code == COMMAND_GET_APP_STATS_MAH_IPC
        Stub->>Service: GetAppStatsMah()
    alt code == COMMAND_GET_PART_STATS_MAH_IPC
        Stub->>Service: GetPartStatsMah()
    end
    Service->>Stub: result
    Stub->>Client: reply
```

**证据来源**：`services/IBatteryStats.idl`

---

## 事件处理流程

### HiSysEvent 处理流程

```mermaid
sequenceDiagram
    participant Kernel as "Linux Kernel"
    participant Event as "HiSysEvent"
    participant Detector as "BatteryStatsDetector"
    participant Core as "BatteryStatsCore"
    participant Entity as "Entity"

    Kernel->>Event: Raise HiSysEvent
    Event->>Detector: OnEvent(event)
    Detector->>Core: HandleEvent(event)
    Core->>Core: Route to specific handler
    Core->>Entity: ProcessXXXEvent()
    Entity->>Entity: UpdateStats()
    Entity->>Entity: CalculatePower()
    Entity-->>Core: Updated stats
    Core-->>Detector: Event processed
```

**证据来源**：`services/native/src/battery_stats_detector.cpp`

### Common Event 处理流程

```mermaid
sequenceDiagram
    participant Event as "CommonEventService"
    participant Subscriber as "BatteryStatsSubscriber"
    participant Core as "BatteryStatsCore"
    participant Service as "BatteryStatsService"

    Event->>Subscriber: OnReceiveEvent(data)
    Subscriber->>Core: HandleCommonEvent(data)
    Core->>Core: Process event type
    alt COMMON_EVENT_SHUTDOWN
        Core->>Core: ResetAllStats()
    alt COMMON_EVENT_BATTERY_CHANGED
        Core->>Core: UpdateBatteryState()
    alt COMMON_EVENT_BATTERY_LOW
        Core->>Core: NotifyLowPower()
    end
```

**证据来源**：`services/native/src/battery_stats_subscriber.cpp`

---

## 数据统计流程

### 实体数据收集流程

```mermaid
sequenceDiagram
    participant Core as "BatteryStatsCore"
    participant Entity as "Entity"
    participant Timer as "Timer/Counter"
    participant Calc as "Power Calculator"

    Core->>Entity: StartCollect()
    Entity->>Entity: Initialize()
    loop Collecting
        Entity->>Timer: StartTimer()
        Entity->>Timer: CountData()
        Timer-->>Entity: Elapsed time
        Timer-->>Entity: Data bytes
        Entity->>Calc: CalculatePower()
        Calc-->>Entity: Power (mAh)
    end
    Entity->>Core: ReportStats()
    Core->>Core: Aggregate to total
```

### 耗电计算流程

```mermaid
sequenceDiagram
    participant Core as "BatteryStatsCore"
    participant Entity as "Entity"
    participant Parser as "BatteryStatsParser"
    participant Config as "Power Profile"

    Core->>Entity: CalculatePower()
    Entity->>Parser: GetAveragePower(type)
    Parser->>Config: Lookup power value
    Config-->>Parser: Average power (mAh)
    Parser-->>Entity: Power value
    Entity->>Entity: power = time * averagePower
    Entity-->>Core: Power value
    Core->>Core: totalPower += entityPower
```

**证据来源**：`services/native/src/battery_stats_core.cpp`

---

## 关键入口点汇总

| 入口点 | 文件:行号 | 说明 |
|--------|-----------|------|
| N-API 模块注册 | `battery_stats_module.cpp:155-170` | napi_module 结构体定义 |
| JS API 导出 | `battery_stats_module.cpp:136-149` | BatteryStatsInit |
| 服务启动 | `battery_stats_service.cpp:68-86` | OnStart() |
| 服务停止 | `battery_stats_service.cpp:88-102` | OnStop() |
| IPC 处理 | `IBatteryStats.idl` | 10 个 IPC 方法 |
| 事件检测 | `battery_stats_detector.cpp` | HiSysEvent 处理 |
| 核心计算 | `battery_stats_core.cpp` | 功耗聚合 |

---

## 相关文档

- [架构设计](../02_Architecture.md) - 整体架构
- [N-API 接口](../03_NAPI.md) - JS API 详细说明
- [Inner API](../04_Inner_API.md) - C++ Kit 接口
- [安全评审](../07_Security.md) - 安全机制分析
