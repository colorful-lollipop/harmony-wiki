# 内部 API (Inner API)

## 目的

本文档描述 distributed_input 模块的内部 API，包括模块间接口、依赖方向、稳定性和可替换点。

## 适用范围

本文档适用于以下场景：
- 理解模块间的接口依赖关系
- 进行功能扩展和定制化开发
- 分析模块稳定性边界
- 识别可替换组件和扩展点

## 关键结论

1. **接口分层**: IPC 接口（Source/Sink Stub）→ Manager → Transport → 基础层
2. **依赖单向**: 清晰的依赖方向，无循环依赖
3. **接口稳定性**: 公共接口（frameworks/）相对稳定，内部接口可能变化
4. **可扩展点**: Transport 层、Handler 层提供明确的扩展接口

## 模块间接口层次

### IPC 接口层（Source/Sink）

#### IDistributedSourceInput（Source 侧 IPC）

**位置**: [i_distributed_source_input.h](../frameworks/include/i_distributed_source_input.h:1-106)

**接口定义**:

```cpp
class IDistributedSourceInput : public IRemoteBroker {
public:
    // SA 生命周期
    virtual int32_t Init() = 0;
    virtual int32_t Release() = 0;

    // 分布式硬件注册/注销
    virtual int32_t RegisterDistributedHardware(const std::string &devId, const std::string &dhId,
        const std::string &parameters, sptr<IRegisterDInputCallback> callback) = 0;
    virtual int32_t UnregisterDistributedHardware(const std::string &devId, const std::string &dhId,
        sptr<IUnregisterDInputCallback> callback) = 0;

    // 远程输入控制
    virtual int32_t PrepareRemoteInput(const std::string &deviceId, sptr<IPrepareDInputCallback> callback) = 0;
    virtual int32_t UnprepareRemoteInput(const std::string &deviceId, sptr<IUnprepareDInputCallback> callback) = 0;
    virtual int32_t StartRemoteInput(const std::string &deviceId, const uint32_t &inputTypes,
        sptr<IStartDInputCallback> callback) = 0;
    virtual int32_t StopRemoteInput(const std::string &deviceId, const uint32_t &inputTypes,
        sptr<IStopDInputCallback> callback) = 0;

    // Relay 模式（SrcId + SinkId）
    virtual int32_t PrepareRemoteInput(const std::string &srcId, const std::string &sinkId,
        sptr<IPrepareDInputCallback> callback) = 0;
    virtual int32_t UnprepareRemoteInput(const std::string &srcId, const std::string &sinkId,
        sptr<IUnprepareDInputCallback> callback) = 0;
    virtual int32_t StartRemoteInput(const std::string &srcId, const std::string &sinkId,
        const uint32_t &inputTypes, sptr<IStartDInputCallback> callback) = 0;
    virtual int32_t StopRemoteInput(const std::string &srcId, const std::string &sinkId,
        const uint32_t &inputTypes, sptr<IStopDInputCallback> callback) = 0;

    // 按 dhId 列表控制
    virtual int32_t StartRemoteInput(const std::string &sinkId, const std::vector<std::string> &dhIds,
        sptr<IStartStopDInputsCallback> callback) = 0;
    virtual int32_t StopRemoteInput(const std::string &sinkId, const std::vector<std::string> &dhIds,
        sptr<IStartStopDInputsCallback> callback) = 0;
    virtual int32_t StartRemoteInput(const std::string &srcId, const std::string &sinkId,
        const std::vector<std::string> &dhIds, sptr<IStartStopDInputsCallback> callback) = 0;
    virtual int32_t StopRemoteInput(const std::string &srcId, const std::string &sinkId,
        const std::vector<std::string> &dhIds, sptr<IStartStopDInputsCallback> callback) = 0;

    // 白名单回调注册
    virtual int32_t RegisterAddWhiteListCallback(sptr<IAddWhiteListInfosCallback> addWhiteListCallback) = 0;
    virtual int32_t RegisterDelWhiteListCallback(sptr<IDelWhiteListInfosCallback> delWhiteListCallback) = 0;

    // 事件监听器注册
    virtual int32_t RegisterSimulationEventListener(sptr<ISimulationEventListener> listener) = 0;
    virtual int32_t UnregisterSimulationEventListener(sptr<ISimulationEventListener> listener) = 0;

    // 会话状态回调
    virtual int32_t RegisterSessionStateCb(sptr<ISessionStateCallback> callback) = 0;
    virtual int32_t UnregisterSessionStateCb() = 0;

    virtual ~IDistributedSourceInput() = default;
};
```

**实现位置**: [distributed_input_source_stub.cpp](../interfaces/ipc/src/distributed_input_source_stub.cpp)

**稳定性**: **高** - Source SA 与外部系统的主接口，变更影响大

---

#### IDistributedSinkInput（Sink 侧 IPC）

**位置**: [i_distributed_sink_input.h](../frameworks/include/i_distributed_sink_input.h:1-62)

**接口定义**:

```cpp
class IDistributedSinkInput : public IRemoteBroker {
public:
    // SA 生命周期
    virtual int32_t Init() = 0;
    virtual int32_t Release() = 0;

    // 屏幕信息回调
    virtual int32_t RegisterGetSinkScreenInfosCallback(sptr<IGetSinkScreenInfosCallback> callback) = 0;

    // 屏幕通知
    virtual int32_t NotifyStartDScreen(const SrcScreenInfo &srcScreenInfo) = 0;
    virtual int32_t NotifyStopDScreen(const std::string &srcScreenInfoKey) = 0;

    // 共享 dhId 监听器注册
    virtual int32_t RegisterSharingDhIdListener(sptr<ISharingDhIdListener> sharingDhIdListener) = 0;

    virtual ~IDistributedSinkInput() = default;
};
```

**实现位置**: [distributed_input_sink_stub.cpp](../interfaces/ipc/src/distributed_input_sink_stub.cpp)

**稳定性**: **高** - Sink SA 与外部系统的主接口，变更影响大

---

### Transport 层接口

#### DistributedInputTransportBase（传输基类）

**位置**: [distributed_input_transport_base.h](../services/transportbase/include/distributed_input_transport_base.h:1-105)

**接口定义**:

```cpp
class DistributedInputTransportBase {
public:
    // 会话管理
    virtual int32_t CreateSession(const std::string &peerNetworkId, int32_t sessionId) = 0;
    virtual int32_t CloseSession(int32_t sessionId) = 0;

    // 消息发送/接收
    virtual int32_t SendMessage(int32_t sessionId, const std::string &message) = 0;

    // SoftBus 回调
    virtual void OnSessionOpened(int32_t sessionId) {}
    virtual void OnBytesReceived(int32_t sessionId, const std::string &message) {}
    virtual void OnSessionClosed(int32_t sessionId) {}

    virtual ~DistributedInputTransportBase() = default;
};
```

**实现位置**: [distributed_input_transport_base.cpp](../services/transportbase/src/distributed_input_transport_base.cpp)

**稳定性**: **中** - Transport 基类定义协议接口，具体实现类继承扩展

---

#### Transport 回调接口

**Source Transport 回调**（DInputSourceTransport 实现）:

| 接口 | 说明 | 位置 |
|------|------|------|
| `DInputSourceTransCallback` | Source 传输回调，处理 Sink 发来的消息 | [dinput_source_trans_callback.h](../services/common/include/dinput_source_trans_callback.h) |

**Sink Transport 回调**（DInputSinkTransport 实现）:

| 接口 | 说明 | 位置 |
|------|------|------|
| `DInputSinkTransCallback` | Sink 传输回调，处理 Source 的消息 | [dinput_sink_trans_callback.h](../services/common/include/dinput_sink_trans_callback.h) |

**TransportBase 回调**（DistributedInputTransportBase 使用）:

| 接口 | 说明 | 位置 |
|------|------|------|
| `DInputTransbaseSourceCallback` | TransportBase 侧 Source 回调 | [dinput_transbase_source_callback.h](../services/common/include/dinput_transbase_source_callback.h) |
| `DInputTransbaseSinkCallback` | TransportBase 侧 Sink 回调 | [dinput_transbase_sink_callback.h](../services/common/include/dinput_transbase_sink_callback.h) |

**稳定性**: **低** - Transport 回调为内部接口，可能随需求调整

---

### Manager 层回调接口

#### Source Manager 回调

| 接口 | 说明 | 位置 | 稳定性 |
|------|------|------|------|
| `DInputSourceManagerCallback` | Source Manager 的回调接口 | [dinput_source_manager_callback.h](../services/common/include/dinput_source_manager_callback.h) | **中** |

**Sink Manager 回调**:

| 接口 | 说明 | 位置 | 稳定性 |
|------|------|------|------|
| `DInputSinkManagerCallback` | Sink Manager 的回调接口 | [dinput_sink_manager_callback.h](../services/common/include/dinput_sink_manager_callback.h) | **中** |

**稳定性说明**:
- Manager 回调为内部接口，连接 Transport、State、DFX 等模块
- 相对外 IPC 接口（IDistributedSourceInput/SinkInput），内部回调调整影响范围较小

---

### 服务组件接口

#### 事件采集接口

**位置**: [distributed_input_collector.h](../services/sink/inputcollector/include/distributed_input_collector.h:1-84)

**接口定义**:

```cpp
class DistributedInputCollector {
public:
    // 采集线程
    virtual int32_t StartCollectionThread() = 0;

    // 共享设置
    virtual void SetSharingTypes(const uint32_t &inputTypes) = 0;

    // 设备查询
    virtual std::vector<std::string> GetSharingDhIds() = 0;
    virtual void ClearResourcesStatus() = 0;

    virtual ~DistributedInputCollector() = default;
};
```

**实现位置**: [distributed_input_collector.cpp](../services/sink/inputcollector/src/distributed_input_collector.cpp)

**稳定性**: **中** - 采集接口为内部接口，依赖 inputdevicehandler 提供的驱动访问能力

---

#### 事件注入接口

**位置**: [distributed_input_inject.h](../services/source/inputinject/include/distributed_input_inject.h:1-64)

**接口定义**:

```cpp
class DistributedInputInject {
public:
    // 虚拟驱动管理
    virtual int32_t RegisterDistributedHardware(const std::string &dhId, const std::string &dhType,
        const std::string &parameters, sptr<IInputNodeListener> callback) = 0;
    virtual void UnregisterDistributedHardware(const std::string &dhId) = 0;

    // 事件注册
    virtual int32_t RegisterDistributedEvent(const std::string &dhId, const BusinessEvent &event) = 0;
    virtual void UnregisterDistributedEvent(const std::string &dhId) = 0;

    virtual ~DistributedInputInject() = default;
};
```

**实现位置**: [distributed_input_inject.cpp](../services/source/inputinject/src/distributed_input_inject.cpp)

**稳定性**: **中** - 注入接口为内部接口，依赖 inputdevicehandler 提供的虚拟驱动能力

---

#### 状态管理接口

**位置**: [dinput_sink_state.h](../services/state/include/dinput_sink_state.h:1-103)

**接口定义**:

```cpp
class DInputSinkStateManager {
public:
    // 设备状态管理
    virtual void SetDhIdState(const std::string &dhId, DhIdState state) = 0;
    virtual DhIdState GetDhIdState(const std::string &dhId) = 0;

    // 按键状态
    virtual void UpdateKeyState(const std::string &dhId, const uint32_t &type,
        const uint32_t &code, const uint32_t &value) = 0;

    // 触摸板事件片段
    virtual void AddTouchpadEventFragment(const std::string &dhId, const TouchpadEventFragment &fragment) = 0;

    // 资源清理
    virtual void ClearResourcesStatus() = 0;

    virtual ~DInputSinkStateManager() = default;
};
```

**实现位置**: [dinput_sink_state.cpp](../services/state/src/dinput_sink_state.cpp)

**稳定性**: **中** - 状态管理为内部接口，仅 Sink Manager 和 Collector 使用

---

### 框架集成接口

#### Source Handler 接口

**位置**: [distributed_input_source_handler.h](../sourcehandler/include/distributed_input_source_handler.h:1-105)

**接口定义**（来自分布式硬件框架）:

```cpp
class IDistributedInputSourceHandler : public IRemoteBroker {
public:
    // Source 初始化
    virtual int32_t InitSource(const std::string &params) = 0;

    // 分布式硬件注册
    virtual int32_t RegisterDistributedHardware(const std::string &devId, const std::string &dhId,
        const std::string &parameters) = 0;
    virtual int32_t UnregisterDistributedHardware(const std::string &devId, const std::string &dhId) = 0;

    // 配置下发
    virtual int32_t ConfigDistributedHardware(const std::string &devId, const std::string &dhId,
        const std::string &parameters) = 0;

    virtual ~IDistributedInputSourceHandler() = default;
};
```

**实现位置**: [distributed_input_source_handler.cpp](../sourcehandler/src/distributed_input_source_handler.cpp)

**稳定性**: **中** - Source Handler 作为框架适配层，接口由分布式硬件框架定义，相对稳定

---

#### Sink Handler 接口

**位置**: [distributed_input_sink_handler.h](../sinkhandler/include/distributed_input_sink_handler.h:1-98)

**接口定义**（来自分布式硬件框架）:

```cpp
class IDistributedInputSinkHandler : public IRemoteBroker {
public:
    // Sink 初始化
    virtual int32_t InitSink(const std::string &params) = 0;

    // 本地硬件订阅
    virtual int32_t SubscribeLocalHardware(const std::string &params) = 0;
    virtual int32_t UnsubscribeLocalHardware(const std::string &params) = 0;

    // 暂停/恢复/停止
    virtual int32_t PauseDistributedHardware(const std::string &dhId) = 0;
    virtual int32_t ResumeDistributedHardware(const std::string &dhId) = 0;
    virtual int32_t StopDistributedHardware(const std::string &dhId) = 0;

    // 隐私资源注册
    virtual int32_t RegisterPrivacyResources(std::shared_ptr<PrivacyResourcesListener> listener) = 0;

    virtual ~IDistributedInputSinkHandler() = default;
};
```

**实现位置**: [distributed_input_sink_handler.cpp](../sinkhandler/src/distributed_input_sink_handler.cpp)

**稳定性**: **中** - Sink Handler 作为框架适配层，接口由分布式硬件框架定义，相对稳定

---

#### 设备能力查询接口

**位置**: [distributed_input_handler.h](../inputdevicehandler/include/distributed_input_handler.h:1-87)

**接口定义**（来自分布式硬件框架）:

```cpp
class IDistributedInputHandler : public IRemoteBroker {
public:
    // 初始化
    virtual int32_t Initialize() = 0;

    // 查询
    virtual int32_t Query() = 0;
    virtual int32_t QueryMeta(const std::string &dhId) = 0;
    virtual int32_t FindDevicesInfoByType(uint32_t inputType, std::vector<std::string> &dhIds) = 0;

    virtual ~IDistributedInputHandler() = default;
};
```

**实现位置**: [distributed_input_handler.cpp](../inputdevicehandler/src/distributed_input_handler.cpp)

**稳定性**: **中** - 设备能力查询为内部接口，仅框架和 State Manager 使用

---

### 公共常量和数据结构

#### 核心常量（constants_dinput.h）

**位置**: [constants_dinput.h](../common/include/constants_dinput.h:1-360)

**稳定性**: **高** - 常量定义是模块的基础，变更影响大

**关键常量**:

| 常量名 | 值 | 说明 | 用途 |
|---------|-----|------|------|
| `MOUSE` | 0x1 | 鼠标输入类型 |
| `KEYBOARD` | 0x2 | 键盘输入类型 |
| `TOUCHSCREEN` | 0x4 | 触摸屏输入类型 |
| `JOYSTICK` | 0x8 | 摇杆输入类型 |
| `SESSION_STATE_INIT` | 0 | 会话初始化状态 |
| `SESSION_STATE_PREPARED` | 1 | 会话已准备状态 |
| `SESSION_STATE_STARTED` | 2 | 会话已启动状态 |
| `SESSION_STATE_STOPPED` | 3 | 会话已停止状态 |
| `SESSION_STATE_UNPREPARED` | 4 | 会话已取消准备状态 |
| `THROUGH_IN` | 0 | 设备在跨设备输入状态 |
| `THROUGH_OUT` | 1 | 设备在本地生效状态 |

**关键数据结构**:

| 结构体 | 字段 | 说明 |
|--------|------|------|
| `RawEvent` | type, code, value | 原始输入事件 |
| `InputDevice` | dhId, dhType, networkId, name | 输入设备信息 |
| `BusinessEvent` | type, code, value | 业务事件（用于过滤） |
| `SrcScreenInfo` | srcDevId, srcWinId, srcWidth, srcHeight | Source 屏幕信息 |
| `SinkScreenInfo` | sinkWidth, sinkHeight, sinkPixelRatio | Sink 屏幕信息 |

---

## 依赖方向图

```
           基础层 (Base)
                  │
                  │
        ┌─────────┼────────┐
        │         │         │
        ▼         ▼         ▼
┌─────────────┐ ┌────────────────┐ ┌─────────────┐
│ interfaces/ │ │ frameworks/    │ │   utils/   │
│ IPC层      │ │ (Callbacks)   │ │  (工具类)   │
└─────────────┘ └────────────────┘ └─────────────┘
        │                │
        └────────┬─────────┘
                 │
        ┌────────┼────────┐
        │        │         │
        ▼        ▼         ▼
┌──────────────┐ ┌──────────────────┐
│ IPC Stubs   │ │ TransportBase    │
│ (IPC 实现) │ │ (传输基类)     │
└──────────────┘ └──────────────────┘
        │                │
        └────────┬─────────┘
                 │
        ┌────────┼────────┐
        │        │         │
        ▼        ▼         ▼
┌──────────────┐ ┌──────────────────┐
│ Source/Sink  │ │ Services/       │
│ Manager SA  │ │ (业务层)       │
└──────────────┘ └──────────────────┘
        │                │
        └────────┬─────────┘
                 │
        ┌────────┼────────┐
        │        │         │
        ▼        ▼         ▼
┌──────────────┐ ┌──────────────────┐
│ Transport    │ │ Source/Sink      │
│ 实现        │ │ (具体传输)      │
└──────────────┘ └──────────────────┘
        │                │
        └────────┬─────────┘
                 │
        ┌────────┼────────┐
        │        │         │
        ▼        ▼         ▼
┌──────────────┐ ┌──────────────────┐
│ State/      │ │ Inject/Collector│
│ (状态管理)  │ │ (事件注入/采集) │
└──────────────┘ └──────────────────┘
        │                │
        └────────┬─────────┘
                 │
        ┌────────┼────────┐
        │        │         │
        ▼        ▼         ▼
┌──────────────┐ ┌──────────────────┐
│ Handlers     │ │ DFX Utils      │
│ (框架集成)  │ │ (诊断工具)      │
└──────────────┘ └──────────────────┘
```

**依赖说明**:

1. **无循环依赖**: 所有依赖都是单向的，从基础层向上传递
2. **分层清晰**: 每一层只依赖下层，避免复杂耦合
3. **接口隔离**: 框架接口（Source/Sink Handler）和服务实现通过 IPC 隔离

---

## 接口稳定性分级

### 高稳定性接口（不轻易修改）

| 接口 | 稳定性 | 理由 |
|------|---------|------|
| `IDistributedSourceInput` | **高** | Source SA 的主 IPC 接口，外部系统直接调用 |
| `IDistributedSinkInput` | **高** | Sink SA 的主 IPC 接口，外部系统直接调用 |
| `IDistributedInputSourceHandler` | **中** | Source Handler 接口，由分布式硬件框架定义 |
| `IDistributedInputSinkHandler` | **中** | Sink Handler 接口，由分布式硬件框架定义 |
| `IDistributedInputHandler` | **中** | 设备能力查询接口，由分布式硬件框架定义 |

**证据**:
- IDistributedSourceInput 和 IDistributedSinkInput 在 frameworks/ 中定义
- Source/Sink Handler 在 sourcehandler/ 和 sinkhandler/ 中实现框架接口
- 这些接口由外部框架定义，本模块仅实现

### 中稳定性接口（可调整）

| 接口 | 稳定性 | 理由 |
|------|---------|------|
| `DistributedInputTransportBase` | **中** | Transport 基类，定义协议接口 |
| `DistributedInputCollector` | **中** | 事件采集接口，内部接口 |
| `DistributedInputInject` | **中** | 事件注入接口，内部接口 |
| `DInputSinkStateManager` | **中** | 状态管理接口，内部接口 |

**证据**:
- TransportBase、Collector、Inject、State 都在 services/ 内部
- 仅相关的 Manager 使用，不对外暴露

### 低稳定性接口（内部实现）

| 接口 | 稳定性 | 理由 |
|------|---------|------|
| Transport 回调接口 | **低** | Transport 回调（SourceTransCallback、SinkTransCallback 等） |
| Manager 回调接口 | **低** | Manager 回调（SourceManagerCallback、SinkManagerCallback 等） |
| State 回调接口 | **低** | State 回调（DInputSourceManagerCallback、DInputSinkManagerCallback 等） |

**证据**:
- 回调接口在 services/common/ 中定义
- 为内部服务间通信设计

---

## 可替换点

### Transport 层替换

**位置**: [services/transportbase/](../services/transportbase/)

**可替换组件**:
- `DistributedInputTransportBase` - 可替换传输实现（如使用其他传输协议）
- `SoftBusPermissionCheck` - 可替换权限检查逻辑

**如何扩展**:
1. 继承 `DistributedInputTransportBase` 实现新的传输类
2. 在 Source Manager 或 Sink Manager 中替换 Transport 实例

---

### 事件注入替换

**位置**: [services/source/inputinject/](../services/source/inputinject/)

**可替换组件**:
- `DistributedInputInject` - 可替换事件注入实现（如使用其他注入机制）

**如何扩展**:
1. 实现 `DistributedInputInject` 接口
2. 替换 Source Manager 中的 `DistributedInputInject` 实例

---

### 事件采集替换

**位置**: [services/sink/inputcollector/](../services/sink/inputcollector/)

**可替换组件**:
- `DistributedInputCollector` - 可替换事件采集实现（如使用其他输入驱动）

**如何扩展**:
1. 实现 `DistributedInputCollector` 接口
2. 替换 Sink Manager 中的 `DistributedInputCollector` 实例

---

### Handler 层替换

**位置**: [sourcehandler/](../sourcehandler/) 和 [sinkhandler/](../sinkhandler/)

**可替换组件**:
- `DistributedInputSourceHandler` - Source Handler 实现
- `DistributedInputSinkHandler` - Sink Handler 实现

**如何扩展**:
1. 实现 Handler 接口（通常由外部框架要求）
2. 修改 SA 加载逻辑使用新的 Handler 实现

---

## 模块职责边界

### Source Manager 职责

| 负责 | 不负责 |
|------|--------|
| ✅ 管理虚拟输入驱动生命周期 | ❌ 不负责虚拟驱动的具体注入实现（委托给 Inject） |
| ✅ 管理跨设备输入会话（Prepare/Start/Stop） | ❌ 不负责事件采集（Sink 侧） |
| ✅ 接收来自 Sink 的输入事件 | ❌ 不负责事件注入的具体实现（委托给 Inject） |
| ✅ 注册/注销回调 | ❌ 不负责回调的具体实现 |
| ✅ 权限检查 | ❌ 不负责权限定义（由框架提供） |

### Sink Manager 职责

| 负责 | 不负责 |
|------|--------|
| ✅ 管理本地输入采集状态 | ❌ 不负责事件采集的具体实现（委托给 Collector） |
| ✅ 发送输入事件到 Source | ❌ 不负责事件发送的具体实现（委托给 Transport） |
| ✅ 管理屏幕信息同步 | ❌ 不负责屏幕坐标转换 |
| ✅ 响应 Source 的 Prepare/Start/Stop 请求 | ❌ 不负责 IPC 传输（委托给 Transport） |
| ✅ 权限检查 | ❌ 不负责权限定义（由框架提供） |

## 接口调用链示例

### PrepareRemoteInput 调用链

```
[多模输入应用]
      │
      ▼
[DistributedInputKit.PrepareRemoteInput()]
      │ (interfaces/inner_kits/src/distributed_input_kit.cpp)
      ├──────────────────┐
      ▼                 │
[DistributedInputClient]
      │ (interfaces/ipc/src/distributed_input_client.cpp)
      ├──────────────────┐
      ▼                 │
[DistributedInputSourceProxy]
      │ (interfaces/ipc/src/distributed_input_source_proxy.cpp)
      ├──────────────────┐
      ▼                 │
[IPC Binder 调用]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSourceStub.OnRemoteRequest()]
      │ (interfaces/ipc/src/distributed_input_source_stub.cpp)
      ├──────────────────┐
      ▼                 │
[权限检查: HasAccessDHPermission()]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSourceManager.HandlePrepareRemoteInput()]
      │ (services/source/sourcemanager/src/distributed_input_source_manager.cpp)
      ├──────────────────┐
      ▼                 │
[创建 SoftBus 会话]
      │
      ├──────────────────┐
      ▼                 │
[DInputSourceTransport.CreateSession()]
      │ (services/source/transport/src/distributed_input_source_transport.cpp)
      ├──────────────────┐
      ▼                 │
[SoftBus 会话创建成功]
      │
      └───────────────────┘
```

### StartRemoteInput 调用链

```
[多模输入应用]
      │
      ▼
[DistributedInputKit.StartRemoteInput()]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputClient]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSourceProxy]
      │
      ├──────────────────┐
      ▼                 │
[IPC Binder 调用]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSourceStub.OnRemoteRequest()]
      │
      ├──────────────────┐
      ▼                 │
[权限检查: HasAccessDHPermission()]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSourceManager.HandleStartRemoteInput()]
      │
      ├──────────────────┐
      ▼                 │
[发送 SoftBus 消息: START_REMOTE_INPUT]
      │
      ├──────────────────┐
      ▼                 │
[SoftBus 传输]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSinkManager]
      │ (services/sink/sinkmanager/src/distributed_input_sink_manager.cpp)
      ├──────────────────┐
      ▼                 │
[权限检查: HasEnableDHPermission()]
      │
      ├──────────────────┐
      ▼                 │
[DistributedInputSinkStub.HandleStartDScreenInner()]
      │ (interfaces/ipc/src/distributed_input_sink_stub.cpp)
      ├──────────────────┐
      ▼                 │
[回调通知]
      │
      └───────────────────┘
```

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 详细的模块职责划分
- [架构设计](03_Architecture.md) - 组件关系和数据流
- [公共 API](04_Public_API.md) - 公共 API 接口说明
- [安全评审](08_Security_Review.md) - 权限和安全机制

---

*更新时间: 2026-02-06 15:08:55*
