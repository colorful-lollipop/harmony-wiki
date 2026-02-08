# 内部 API (Inner API)

## 目的

本文档详细说明 `device_status` 模块的内部 API 接口定义，包括模块间接口、依赖方向和稳定性标注。

---

## Inner API 分类

本模块的 Inner API 按照职责分为以下几类：

| 分类 | 说明 | 文件位置 |
|--------|------|----------|
| **服务接口** | Idevicestatus、IIntention、IContext | `services/` 目录 |
| **插件接口** | IPlugin、IPluginManager | `intention/prototype/` |
| **协调接口** | ICooperate、IDragManager | `intention/` |
| **回调接口** | 各种 IRemote 回调接口 | `interfaces/innerkits/include/` |
| **客户端接口** | IClient、各种 Manager | `frameworks/native/` |
| **适配器接口** | IInputAdapter、IDSoftbusAdapter | `intention/adapters/` |

---

## 核心服务接口

### 1. Idevicestatus（设备状态服务接口）

**位置**: `services/communication/base/i_devicestatus.h`

**接口定义**：

```cpp
class Idevicestatus : public IRemoteBroker {
public:
    // 订阅设备状态变化
    virtual void Subscribe(Type type,
                        ActivityEvent event,
                        ReportLatencyNs latency,
                        sptr<IRemoteDevStaCallback> callback) = 0;

    // 取消订阅
    virtual void Unsubscribe(Type type,
                          ActivityEvent event,
                          sptr<IRemoteDevStaCallback> callback) = 0;

    // 获取设备状态缓存
    virtual Data GetCache(const Type &type) = 0;

    // 分配 Socket FD（用于插件）
    virtual int32_t AllocSocketFd(const std::string &programName,
                                     int32_t moduleType,
                                     int32_t &socketFd,
                                     int32_t &tokenType) = 0;

    // 检查服务是否运行
    virtual bool IsRunning() const {
        return true;
    }

    // 接口描述符
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.msdp.Idevicestatus");
};
```

**实现类**: `DeviceStatusService` (继承自 `DeviceStatusSrvStub`)

**调用方**：
- IntentionService（内部）
- 客户端（通过 IPC）

---

### 2. IContext（中央上下文接口）

**位置**: `intention/prototype/include/i_context.h`

**接口定义**：

```cpp
class IContext {
public:
    // 获取任务委托器
    virtual IDelegateTasks& GetDelegateTasks() = 0;

    // 获取设备管理器
    virtual IDeviceManager& GetDeviceManager() = 0;

    // 获取定时器管理器
    virtual ITimerManager& GetTimerManager() = 0;

    // 获取拖拽管理器
    virtual IDragManager& GetDragManager() = 0;

    // 获取 DDM 适配器
    virtual IDDMAdapter& GetDDM() = 0;

    // 获取 DSoftBus 适配器
    virtual IDSoftbus& GetDSoftbus() = 0;

    // 获取输入适配器
    virtual IInputAdapter& GetInput() = 0;

    // 获取插件管理器
    virtual IPluginManager& GetPluginManager() = 0;

    // 获取 Socket 会话管理器
    virtual ISocketSessionManager& GetSocketSessionManager() = 0;
};
```

**实现类**: `DeviceStatusService`

**依赖方向**：
- DeviceStatusService 依赖所有子系统
- 子系统通过 IContext 接口访问服务能力

---

## 插件系统接口

### 1. IPlugin（插件接口）

**位置**: `intention/prototype/include/i_plugin.h`

**接口定义**：

```cpp
class IPlugin {
public:
    // 启用插件
    virtual int32_t Enable(Callback callback) = 0;

    // 禁用插件
    virtual void Disable() = 0;

    // 启动插件（指定监控 ID）
    virtual void Start(const std::string &option, Callback callback) = 0;

    // 停止插件
    virtual void Stop() = 0;

    // 添加监听器
    virtual void AddWatch(WatchType type, Callback callback) = 0;

    // 移除监听器
    virtual void RemoveWatch(WatchType type) = 0;

    // 设置参数
    virtual void SetParam(ParamType type, const ParamValue &value) = 0;

    // 获取参数
    virtual void GetParam(ParamType type, ParamValue &value, Callback callback) = 0;

    // 控制
    virtual void Control(ControlType type, const std::string &option) = 0;
};
```

**实现类**：
- `CooperatePlugin` - 协同插件
- `MotionDragPlugin` - 运动拖拽插件

**生命周期**：
- `Enable()` → `Start()` → ... → `Stop()` → `Disable()`

---

### 2. IPluginManager（插件管理器接口）

**位置**: `intention/scheduler/plugin_manager/include/i_plugin_manager.h`

**接口定义**：

```cpp
class IPluginManager {
public:
    // 加载插件
    virtual int32_t LoadPlugin(const std::string &pluginName, sptr<IPlugin> &plugin) = 0;

    // 卸载插件
    virtual int32_t UnloadPlugin(const std::string &pluginName) = 0;

    // 检查插件是否已加载
    virtual bool IsPluginLoaded(const std::string &pluginName) = 0;

    // 遍历所有插件
    virtual void ForEachPlugin(std::function<void(sptr<IPlugin>)> callback) = 0;
};
```

**实现类**: `PluginManager` (在 `intention/scheduler/plugin_manager/`)

---

## 协调接口

### 1. ICooperate（协同接口）

**位置**: `intention/prototype/include/i_cooperate.h`

**接口定义**：

```cpp
class ICooperate {
public:
    // 启动协同
    virtual void OnStartCooperate(StartCooperateData &data) = 0;

    // 停止协同
    virtual void OnStopCooperate() = 0;
};
```

**实现类**：
- `CooperateContext` (实现 IContext)
- `CooperateIn/CooperateOut/CooperateFree` (实现 ICooperate)

**调用关系**：
- CooperatePlugin 调用 `OnStartCooperate()` 和 `OnStopCooperate()`
- CooperateClient 通过 Intention IPC 调用

---

### 2. IDragManager（拖拽管理器接口）

**位置**: `intention/prototype/include/i_drag_manager.h`

**接口定义**：

```cpp
class IDragManager {
public:
    // 获取拖拽数据摘要
    virtual DragData GetDataSummary(int32_t monitorId) = 0;

    // 设置拖拽窗口可见性
    virtual int32_t SetDragWindowVisible(bool isVisible, const std::string &packageName) = 0;

    // 添加拖拽阴影
    virtual int32_t AddShadow(uint8_t shadowPixel, const std::string &packageName) = 0;

    // 移除拖拽阴影
    virtual int32_t RemoveShadow(int32_t shadowId, const std::string &packageName) = 0;

    // 通知拖拽结果
    virtual void NotifyDragResult(DragResult result, const std::string &packageName) = 0;
};
```

**实现类**：
- `DragManager` (在 `services/interaction/drag/`)

---

## 设备管理接口

### 1. IDeviceManager（设备管理器接口）

**位置**: `intention/services/device_manager/include/i_device_manager.h`

**接口定义**：

```cpp
class IDeviceManager {
public:
    // 枚举设备
    virtual void EnumerateDevice(ObserverPtr observer) = 0;

    // 添加设备观察者
    virtual int32_t AddObserver(ObserverPtr observer) = 0;

    // 移除设备观察者
    virtual void RemoveObserver(int32_t observerId) = 0;

    // 获取设备状态
    virtual void GetDeviceState(const std::string &networkId, DeviceStateCallback &callback) = 0;
};
```

**实现类**: `DeviceManager` (在 `intention/services/device_manager/`)

---

## 回调接口

### 1. IRemoteDevStaCallback（静止状态回调）

**位置**: `interfaces/innerkits/include/iremote_dev_sta_callback.h`

**接口定义**：

```cpp
class IRemoteDevStaCallback : public IRemoteBroker {
public:
    // 设备状态变化回调
    virtual void OnDeviceStatusChanged(const Data &devicestatusData) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.msdp.IRemoteDevStaCallback");
};
```

### 2. IRemoteBoomerangCallback（Boomerang 回调）

**位置**: `interfaces/innerkits/include/iremote_boomerang_callback.h`

**接口定义**：

```cpp
class IRemoteBoomerangCallback : public IRemoteBroker {
public:
    // Boomerang 事件回调
    virtual void OnBoomerangEvent(const BoomerangData &data) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.msdp.IRemoteBoomerangCallback");
};
```

### 3. IRemoteOnScreenCallback（屏幕感知回调）

**位置**: `interfaces/innerkits/include/iremote_on_screen_callback.h`

**接口定义**：

```cpp
class IRemoteOnScreenCallback : public IRemoteBroker {
public:
    // 屏幕感知事件回调
    virtual void OnOnScreenEvent(const OnScreenEventData &data) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.msdp.IRemoteOnScreenCallback");
};
```

---

## 适配器接口

### 1. IInputAdapter（输入适配器）

**位置**: `intention/adapters/input_adapter/include/i_input_adapter.h`

**接口定义**：

```cpp
class IInputAdapter {
public:
    // 注入观察者
    virtual void InjectObserver(ObserverPtr observer) = 0;

    // 移除观察者
    virtual void RemoveObserver(ObserverPtr observer) = 0;

    // 开始注入
    virtual int32_t Start() = 0;

    // 停止注入
    virtual void Stop() = 0;
};
```

**实现类**: `InputAdapter` (在 `intention/adapters/input_adapter/`)

### 2. IDSoftbusAdapter（DSoftBus 适配器）

**位置**: `intention/adapters/dsoftbus_adapter/include/i_dsoftbus_adapter.h`

**接口定义**：

```cpp
class IDSoftbusAdapter {
public:
    // 启用 DSoftBus
    virtual int32_t Enable() = 0;

    // 禁用 DSoftBus
    virtual void Disable() = 0;

    // 打开会话
    virtual int32_t OpenSession(const std::string &networkId) = 0;

    // 关闭会话
    virtual void CloseSession(const std::string &networkId) = 0;

    // 发送数据包
    virtual int32_t SendPacket(const std::string &networkId, NetPacket &packet) = 0;

    // 广播数据包
    virtual int32_t BroadcastPacket(NetPacket &packet) = 0;

    // 检查会话是否存在
    virtual bool HasSessionExisted(const std::string &networkId) = 0;

    // 获取本地网络 ID
    static std::string GetLocalNetworkId();
};
```

**实现类**: `DSoftbusAdapter` (在 `intention/adapters/dsoftbus_adapter/`)

---

## 客户端接口

### 1. IClient（客户端接口）

**位置**: `interfaces/innerkits/include/i_client.h`

**接口定义**：

```cpp
class IClient {
public:
    // 连接到设备
    virtual void Connect(const std::string &udid, ClientCallback callback) = 0;

    // 断开连接
    virtual void Disconnect(const std::string &udid) = 0;

    // 发送数据
    virtual void SendData(const std::string &udid, const void *data, uint32_t dataLen) = 0;

    // 接收数据
    virtual void OnData(const std::string &udid, const void *data, uint32_t dataLen) = 0;

    // 获取状态
    virtual void GetState(ClientCallback callback) = 0;
};
```

---

## 依赖方向

### 稳定性标注

| 接口 | 稳定性 | 判据 |
|--------|---------|----------|
| **Idevicestatus** | **稳定** | 公共服务接口，由 DeviceStatusService 实现 |
| **IContext** | **稳定** | 中央上下文接口，定义清晰的服务契约 |
| **IPlugin** | **稳定** | 插件接口，定义标准的插件生命周期 |
| **IPluginManager** | **稳定** | 插件管理器接口，插件系统的核心 |
| **IInputAdapter** | **稳定** | 输入适配器，抽象 MMI 子系统 |
| **IDSoftbusAdapter** | **稳定** | DSoftBus 适配器，封装分布式总线 API |
| **IClient** | **稳定** | Socket 客户端接口，定义简单的数据传输契约 |

| 回调接口 | **稳定** | 所有 IRemote 回调接口，继承 IRemoteBroker |
| 实现 Manager 类 | **稳定** | 各种 Manager 类（DragManager、DeviceManager 等） |

### 不稳定接口

| 接口 | 稳定性 | 原因 |
|--------|---------|----------|
| **内部私有接口** | **不稳定** | 如 DeviceStatusService 内部使用的 private 接口，可能随需求变化 |
| **实验性功能** | **不稳定** | Rust 模块相关接口（`device_status_rust_enabled = false` 时） |
| **Legacy 接口** | **不稳定** | Coordination 模块相关接口，保留向后兼容 |

---

## 接口层次关系

```
应用层 (N-API/ETS)
    ↓
[Inner Kits / Frameworks]
    ↓
┌─────────────────────────────────────────────────┐
│  Inner API 定义层                        │
│  (interfaces/innerkits/include/)          │
├────────────────────────────────────────────────────┤
│  服务接口                                  │
│  (Idevicestatus, IContext)               │
├────────────────────────────────────────────────────┤
│  插件接口                                  │
│  (IPlugin, IPluginManager)               │
├────────────────────────────────────────────────────┤
│  适配器接口                                  │
│  (IInputAdapter, IDSoftbusAdapter)          │
├────────────────────────────────────────────────────┤
│  客户接口                                  │
│  (IClient)                                 │
└─────────────────────────────────────────────────────┘
                    ↓
              [实现层]
              (services/, intention/, frameworks/)
```

---

## 使用建议

### 添加新功能

1. **服务层功能**：在 IContext 中添加新的 getter 方法
2. **新插件**：实现 IPlugin 接口并通过 PluginManager 注册
3. **新适配器**：实现相应的适配器接口（IInputAdapter 等）

### 扩展现有接口

1. **遵循命名规范**：使用 `I` 前缀
2. **定义描述符**：使用 `DECLARE_INTERFACE_DESCRIPTOR` 宏
3. **最小依赖**：尽量减少接口间的循环依赖

---

## 相关跳转

- **[01_Overview](01_Overview.md)** - 项目概览
- **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构详解
- **[03_Architecture](03_Architecture.md)** - 架构设计
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考
- **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标
- **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物说明

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
