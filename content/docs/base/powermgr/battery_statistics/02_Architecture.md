# 架构设计

## 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Application Layer                         │
│  ┌───────────────────┐    ┌───────────────────┐                 │
│  │  JS/TS Application │    │  ArkTS Application │                 │
│  └─────────┬─────────┘    └─────────┬─────────┘                 │
└────────────┼────────────────────────┼────────────────────────────┘
             │                        │
             ▼                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Framework Layer                              │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                   N-API (batterystatistics)              │     │
│  │   getBatteryStats / getAppPowerValue / getAppPowerPercent│     │
│  │   getHardwareUnitPowerValue / getHardwareUnitPowerPercent │     │
│  └──────────────────────────┬──────────────────────────────┘     │
│                             │                                     │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │            BatteryStatsClient (Inner API)               │     │
│  │              Connect() / GetBatteryStats()               │     │
│  └──────────────────────────┬──────────────────────────────┘     │
└─────────────────────────────┼────────────────────────────────────┘
                              │
                              ▼ IPC (Binder)
┌─────────────────────────────────────────────────────────────────┐
│                     Service Layer                                │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │             BatteryStatsService (SA 3304)                │     │
│  │   OnStart() / OnStop() / GetBatteryStats() / Dump()     │     │
│  └──────────────────────────┬──────────────────────────────┘     │
│                             │                                     │
│              ┌──────────────┼──────────────┐                     │
│              ▼              ▼              ▼                     │
│  ┌────────────────┐ ┌────────────┐ ┌─────────────────┐          │
│  │ BatteryStatsCore│ │ Detectors │ │    Parsers      │          │
│  │  (Core Logic)  │ │(HiSysEvent)│ │(Power Profile) │          │
│  └────────────────┘ └────────────┘ └─────────────────┘          │
│                             │                                     │
│              ┌──────────────┼──────────────┐                     │
│              ▼              ▼              ▼                     │
│  ┌────────────────┐ ┌────────────┐ ┌─────────────────┐          │
│  │   Entities     │ │ Subscribers│ │   Dump/Utils    │          │
│  │ (15 modules)   │ │ (Events)   │ │                 │          │
│  └────────────────┘ └────────────┘ └─────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Kernel / Native                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐   │
│  │ CPU Time    │  │ HiSysEvent  │  │  Battery Manager        │   │
│  │ (cpuacct)   │  │   Kernel    │  │  (Battery State)       │   │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## 组件说明

### 1. N-API 层（frameworks/napi/）

**职责**：向 JS/TS 应用暴露 API。

**关键组件**：
- `BatteryStats` - NAPI 主类
- `BatteryStatsInit()` - 模块初始化
- `GetBatteryStats()` / `GetAppStatsMah()` 等 - 5 个导出函数

**证据来源**：`frameworks/napi/src/battery_stats_module.cpp:136-170`

### 2. Inner API 层（interfaces/inner_api/）

**职责**：供 C++ 模块调用的 Kit。

**关键组件**：
- `BatteryStatsClient` - 客户端单例
- `GetBatteryStatsInfo()` - 获取统计数据

**证据来源**：`interfaces/inner_api/include/battery_stats_client.h`

### 3. Service 层（services/native/）

**职责**：核心耗电统计逻辑。

**关键组件**：
- `BatteryStatsService` - 主服务（继承 SystemAbility + BatteryStatsStub）
- `BatteryStatsCore` - 核心计算逻辑
- `BatteryStatsDetector` - HiSysEvent 事件检测
- `BatteryStatsParser` - 功耗配置解析

**证据来源**：`services/native/include/battery_stats_service.h:33-41`

### 4. 实体类层（entities/）

**职责**：各硬件/软件组件的耗电跟踪。

**15 个实体**：
- 基础类：`BatteryStatsEntity`
- 跟踪类：`CpuEntity`, `WifiEntity`, `BluetoothEntity`, `ScreenEntity` 等

**证据来源**：`services/native/include/entities/*.h`

## 数据流

### 1. 耗电数据收集流程

```
HiSysEvent (内核)
        │
        ▼
BatteryStatsDetector::OnEvent()
        │
        ▼
BatteryStatsCore::HandleXXXEvent()
        │
        ▼
对应 Entity::ProcessXXX()
        │
        ▼
Entity::CalculatePower() / Entity::UpdateStats()
        │
        ▼
BatteryStatsCore::汇总到 totalPower
```

### 2. API 调用流程

```
应用 (JS/TS)
        │
        ▼
N-API: GetBatteryStats()
        │
        ▼
BatteryStatsClient::GetBatteryStats()
        │
        ▼
IPC: SendRequest(COMMAND_GET_BATTERY_STATS_IPC)
        │
        ▼
BatteryStatsStub::OnRemoteRequest()
        │
        ▼
BatteryStatsService::GetBatteryStats()
        │
        ▼
BatteryStatsCore::GetBatteryStats()
        │
        ▼
返回统计数据
```

## 线程模型

### 服务线程

| 线程/任务 | 职责 |
|-----------|------|
| SA 主线程 | OnStart/OnStop/Dump/System API |
| IPC 处理线程 | 处理跨进程请求 |
| Event Handler | 处理 HiSysEvent 事件 |
| Common Event 线程 | 处理公共事件 |

**证据来源**：`services/native/src/battery_stats_service.cpp:68-102`

### 互斥锁保护

| 互斥锁 | 保护对象 | 位置 |
|--------|----------|------|
| `BatteryStatsService::mutex_` | 服务状态 | `battery_stats_service.h:92` |
| `BatteryStatsCore::mutex_` | 核心统计数据 | `battery_stats_core.h:85` |
| `BatteryStatsClient::mutex_` | 客户端连接 | `battery_stats_client.cpp:38,58` |

**证据来源**：`services/native/include/battery_stats_core.h:85`

## 服务生命周期

### OnStart() 流程

```cpp
void BatteryStatsService::OnStart()
{
    if (ready_) return;
    // 1. 初始化服务
    if (!(Init())) return;
    // 2. 添加系统能力监听
    AddSystemAbilityListener(DFX_SYS_EVENT_SERVICE_ABILITY_ID);
    AddSystemAbilityListener(COMMON_EVENT_SERVICE_ID);
    // 3. 发布服务
    if (!Publish(BatteryStatsService::GetInstance())) return;
    // 4. 注册启动完成回调
    RegisterBootCompletedCallback();
    ready_ = true;
}
```

**证据来源**：`services/native/src/battery_stats_service.cpp:68-86`

### OnStop() 流程

```cpp
void BatteryStatsService::OnStop()
{
    if (!ready_) return;
    ready_ = false;
    isBootCompleted_ = false;
    // 移除监听器
    RemoveSystemAbilityListener(...);
    // 移除 HiSysEvent 监听
    HiviewDFX::HiSysEventManager::RemoveListener(listenerPtr_);
    // 取消公共事件订阅
    CommonEventManager::UnSubscribeCommonEvent(subscriberPtr_);
}
```

**证据来源**：`services/native/src/battery_stats_service.cpp:88-102`

## 时序图

### API 调用时序（N-API）

```
participant App as "JS Application"
participant NAPI as "BatteryStats N-API"
participant Client as "BatteryStatsClient"
participant IPC as "IPC Proxy/Stub"
participant Service as "BatteryStatsService"
participant Core as "BatteryStatsCore"

App->>NAPI: getBatteryStats()
NAPI->>NAPI: 创建 AsyncWork
NAPI->>NAPI: napi_queue_async_work()
NAPI-->>App: Promise

Core->>IPC: 执行异步工作
IPC->>IPC: SendRequest(COMMAND_GET_BATTERY_STATS_IPC)
IPC->>Service: OnRemoteRequest()
Service->>Core: GetBatteryStats()
Core->>Core: 收集统计数据
Core-->>Service: ParcelableBatteryStatsList
Service-->>IPC: reply
IPC-->>NAPI: async_complete
NAPI->>App: resolve(Promise)
```

### 服务启动时序

```
participant System as "SystemAbilityMgr"
participant Service as "BatteryStatsService"
participant Core as "BatteryStatsCore"
participant Detector as "BatteryStatsDetector"
participant Subscriber as "BatteryStatsSubscriber"

System->>Service: OnStart()
Service->>Core: Init()
Core->>Detector: Init()
Detector->>Detector: 注册 HiSysEvent 监听
Service->>Subscriber: Init()
Subscriber->>Subscriber: 订阅公共事件
Service->>System: Publish()
System->>Service: 启动完成
Service->>Core: 启动统计
```

## 关键接口

### IPC 接口（IBatteryStats.idl）

| 方法 | IPC Code | 说明 |
|------|----------|------|
| `GetBatteryStatsIpc` | 0 | 获取所有统计 |
| `GetAppStatsMahIpc` | 1 | 获取 App 耗电 mAh |
| `GetAppStatsPercentIpc` | 2 | 获取 App 耗电百分比 |
| `GetPartStatsMahIpc` | 3 | 获取部件耗电 mAh |
| `GetPartStatsPercentIpc` | 4 | 获取部件耗电百分比 |
| `GetTotalTimeSecondIpc` | 5 | 获取总时间 |
| `GetTotalDataBytesIpc` | 6 | 获取数据量 |
| `ResetIpc` | 7 | 重置统计 |
| `SetOnBatteryIpc` | 8 | 设置电池状态 |
| `ShellDumpIpc` | 9 | Shell Dump |

**证据来源**：`services/IBatteryStats.idl`

### 实体类接口

```cpp
class BatteryStatsEntity {
public:
    virtual void ProcessEventXXX() = 0;  // 处理事件
    virtual double CalculatePower() = 0;  // 计算耗电
    virtual void Reset() = 0;              // 重置统计
    virtual void UpdateStats() = 0;        // 更新统计
};
```

**证据来源**：`services/native/include/entities/battery_stats_entity.h`

## 相关文档

- [N-API 接口](03_NAPI.md) - JS API 详细说明
- [Inner API](04_Inner_API.md) - C++ Kit 接口
- [安全评审](07_Security.md) - 安全机制分析
- [调用链](appendix/Callgraphs.md) - 详细调用链路
