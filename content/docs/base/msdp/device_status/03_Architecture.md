# 架构说明

## 目的

本文档详细说明 `device_status` 模块的架构设计，包括组件关系、数据流、线程模型和关键时序。

---

## 总体架构

### 架构概览图

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (ArkTS/ETS/JS)        │
├────────────────────────────────────────────────────────────────────┤
│                  JS N-API / ETS API                   │
│                  (frameworks/js/napi/, frameworks/ets/)           │
├────────────────────────────────────────────────────────────────────┤
│                 Native Client Library                   │
│             (libdevicestatus_client.z.so)              │
├────────────────────────────────────────────────────────────────────┤
│              Intention Client / Socket               │
│              (intention/frameworks/client/,               │
│              intention/ipc/socket/)                       │
└────────────────────────────────────────────────────────────────────┘
                    │ IPC (Binder / Unix Socket)
                    ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Device Status Service                    │
│                  (SystemAbility SA 2902)                      │
├────────────────────────────────────────────────────────────────────┤
│                        IContext                        │
│                   (中央上下文接口)                      │
├────────────┬─────────────┬─────────────┬──────────────┤
│  IDelegate │  IDevice │  ITimer │   IDrag     │  ISocket     │
│   Tasks    │ Manager │   Manager │   Manager     │  Session Mgr │
├────────────┴─────────────┴─────────────┴──────────────┤
│                    IntentionService                       │
│                  (业务逻辑编排)                           │
├────────────┬─────────────┬─────────────┬──────────────┤
│ Cooperate │   Drag     │ Stationary │ Boomerang │  OnScreen    │
│  Server  │  Server    │   Server    │  Server     │  Server      │
└────────────┴─────────────┴─────────────┴───────────────┘
         │
         ↓ Socket/IPC
┌─────────────────────────────────────────────────────────────────┐
│                   基础设施层                         │
├────────────┬─────────────┬─────────────┬──────────────┤
│  Adapters │  IPC 基础 │  调度    │  设备管理 │  插件管理 │
└────────────┴─────────────┴─────────────┴───────────────┘
```

---

## 核心组件

### 1. DeviceStatusService（主服务）

**位置**: `services/native/src/devicestatus_service.cpp`

**职责**：
- 作为 SystemAbility（SA 2902）向 SAMGR 注册
- 实现 IContext 接口，聚合所有子系统
- 初始化和管理工作线程
- 管理服务生命周期（OnStart/OnStop）
- 发布 IntentionService 到 SAMGR

**继承关系**：
```
DeviceStatusService
├── SystemAbility (SA 框架基类)
├── IContext (中央上下文接口)
├── StreamServer (Socket 流服务器)
└── DeviceStatusSrvStub (IPC 请求处理器)
```

**初始化流程**：
```
OnStart()
  ├── Set worker thread ID
  ├── Init() - 初始化所有子系统
  │   ├── DeviceStatusManager (设备状态)
  │   ├── Epoll 创建 (事件循环)
  │   ├── DelegateTasks (任务委托)
  │   ├── TimerManager (定时器)
  │   ├── DeviceManager (设备管理)
  │   ├── DragManager (拖拽)
  │   └── Dumper (调试)
  ├── Publish IntentionService
  ├── Enable SocketSessionManager
  ├── Enable DeviceManager
  └── Start worker thread
```

**工作线程事件循环**：
```
OnThread()
  ├── Epoll wait
  └── 根据事件类型分发
      ├── EPOLL_EVENT_SOCKET → SocketSessionManager
      ├── EPOLL_EVENT_ETASK → DelegateTasks
      ├── EPOLL_EVENT_TIMER → TimerManager
      └── EPOLL_EVENT_DEVICE_MGR → DeviceManager
```

---

### 2. IntentionService（意图服务）

**位置**: `intention/services/intention_service/`

**职责**：
- 高层级业务逻辑编排
- 通过 IPC 向客户端提供功能
- 管理所有插件的生命周期（启用/禁用）
- 实现 IContext 接口供 DeviceStatusService 聚合

**管理器**：
- **PluginManager**：插件生命周期管理（加载/卸载插件）
- **TaskScheduler**：异步/同步任务调度
- **DeviceManager**：输入设备热插拔监控
- **SocketSessionManager**：Socket 会话管理

---

### 3. 功能插件服务器

#### 3.1 CooperateServer（协同服务）

**位置**: `intention/cooperate/server/`

**职责**：
- 跨设备鼠标键盘输入共享
- 协同状态管理（激活、去激活、断开）
- 通过 DSoftBus 与远程设备通信
- 实现状态机（CooperateFree、CooperateIn、CooperateOut）

**状态转换**：
```
Free (空闲)
  ├── OnStartWithOptions → Active
  ├── OnStart → Active
  └── OnStop → Free

Active (协同中)
  ├── DSoftbusHandler::OnStartCooperate → Coordinating
  ├── OnStop → Free
  └── 连接断开 → Free
```

#### 3.2 DragServer（拖拽服务）

**位置**: `intention/drag/server/`

**职责**：
- 拖拽操作启动和停止
- 拖拽数据管理
- 拖拽动画渲染
- 预览动画管理
- 跨设备拖拽支持

**关键组件**：
- `DragManager` - 拖拽管理器
- `DragDrawing` - 拖拽绘制
- `DragDataManager` - 数据管理
- `DragSmoothProcessor` - 平滑处理
- `DragVSyncStation` - VSync 同步

#### 3.3 StationaryServer（静止服务）

**位置**: `intention/stationary/server/`

**职责**：
- 设备静止状态检测
- 传感器数据采集和处理
- 状态事件分发
- 回调管理

#### 3.4 BoomerangServer（元数据服务）

**位置**: `intention/boomerang/server/`

**职责**：
- 图片元数据编码和解码
- 元数据绑定事件分发
- 屏幕内容获取
- 一步绑定支持

#### 3.5 OnScreenServer（屏幕感知服务）

**位置**: `intention/onscreen/server/`

**职责**：
- 控制事件发送
- 页面内容获取
- 屏幕感知能力触发（AI 算法、OCR 等）
- 感知回调管理
- 白名单应用验证

---

### 4. 基础设施层

#### 4.1 Epoll 事件循环

**位置**: `intention/common/epoll/`

**职责**：
- 提供 Unix Epoll 事件循环
- 支持多种事件源注册
- 高性能 I/O 多路复用

**事件类型**：
```cpp
enum EpollEventType {
    EPOLL_EVENT_SOCKET,    // Socket 事件
    EPOLL_EVENT_ETASK,     // 委托任务
    EPOLL_EVENT_TIMER,      // 定时器过期
    EPOLL_EVENT_DEVICE_MGR  // 设备管理器
};
```

#### 4.2 DelegateTasks（任务委托）

**位置**: `services/delegate_task/`

**职责**：
- 提供跨线程任务投递机制
- 线程安全保证
- 任务队列管理

**使用场景**：
```
任何线程 → DelegateTasks::PostTask()
    ↓
    投递到服务线程的工作队列
    ↓
    在 Epoll 事件循环中执行
```

#### 4.3 SocketSessionManager（会话管理）

**位置**: `intention/ipc/socket/`

**职责**：
- Unix Domain Socket 连接管理
- 会话生命周期（创建、销毁、心跳）
- 高性能数据传输

#### 4.4 Adapters（适配器层）

**InputAdapter**：
- 抽象 MMI（多模态输入）子系统
- 提供统一的输入事件接口

**DSoftbusAdapter**：
- 封装 Distributed SoftBus API
- 支持设备发现和跨设备通信

**DDMAdapter**：
- 封装 Distributed Device Manager API
- 设备状态和能力查询

**CommonEventAdapter**：
- 封装 Common Event Service API
- 通用事件订阅

---

## 数据流

### 拖拽数据流

```
应用启动拖拽
    ↓ (N-API)
DragManager::OnStartDrag()
    ↓
    验证权限和状态
    ↓
    创建 DragDrawing 实例
    ↓
    发送拖拽开始事件
    ↓
    跨设备：通过 Intention IPC → Remote DragServer
    ↓
    渲染拖拽窗口和阴影
    ↓
    处理拖拽事件（移动、释放）
    ↓
    OnStopDrag()
    ↓
    清理资源
```

### 屏幕感知数据流

```
应用注册屏幕感知
    ↓ (N-API)
OnScreenManager::OnScreenAwareness()
    ↓
    验证权限（白名单检查）
    ↓
    触发感知能力（AI 算法、OCR）
    ↓
    获取页面内容
    ↓
    通过回调返回数据
    ↓
    应用处理内容
```

### 协同数据流

```
主设备启动协同
    ↓ (N-API)
CooperateClient::Start()
    ↓
    验证权限
    ↓
    通过 Intention IPC → CooperateServer
    ↓
    建立跨设备连接（DSoftBus）
    ↓
    同步输入事件
    │   └──> 鼠标移动、点击、滚轮
    │       └──> 键盘输入
    ↓
    远程设备接收输入
    ↓
    停止协同
```

---

## 线程模型

### 服务端线程

**DeviceStatusService 线程**：
```
主线程
├── OnStart() - 初始化
├── OnStop() - 关闭
└── OnThread() - 事件循环（Epoll）
    ├── Socket 连接处理
    ├── 任务执行（DelegateTasks）
    ├── 定时器过期（TimerManager）
    └── 设备管理（DeviceManager）
```

**线程安全**：
- `DelegateTasks` 提供线程安全的任务投递
- 使用互斥锁保护共享数据
- 原子操作保证原子性

### 事件线程

N-API 回调使用 `napi_send_event` 将事件投递到 ArkTS 事件线程：
```cpp
napi_send_event(env, task, napi_eprio_immediate);
```

---

## 关键时序

### 拖拽启动时序

```
应用
    ↓
[N-API] DragContext::on('drag')
    ↓
调用 JS 拖拽监听器
    ↓
调用 [Native] InteractionManager::StartDrag()
    ↓
[IPC] IntentionService::StartDrag()
    ↓
[IPC] DragServer::OnStartDrag()
    ↓
创建 DragDrawing 实例
    ↓
开始动画渲染
    ↓
拖拽进行中...
    ↓
[IPC] 应用调用 GetDragDataSummary()
    ↓
返回拖拽数据摘要
    ↓
应用释放或移动
    ↓
[N-API] DragContext::off('drag')
    ↓
[IPC] InteractionManager::StopDrag()
    ↓
[IPC] IntentionService::StopDrag()
    ↓
[IPC] DragServer::OnStopDrag()
    ↓
结束动画
    ↓
清理资源
```

### 协同启动时序

```
主设备
    ↓
[N-API] CooperateContext::enable()
    ↓
调用 JS 协同监听器
    ↓
调用 [Native] CooperateClient::Enable()
    ↓
[IPC] IntentionService::PrepareCooperation()
    ↓
[IPC] CooperateServer::OnPrepare()
    ↓
准备协同资源
    ↓
建立 DSoftBus 连接
    ↓
[IPC] IntentionService::ActivateCooperation()
    ↓
[IPC] CooperateServer::OnActivate()
    ↓
激活协同
    ↓
开始输入同步
    ↓
从设备
    ↓
[N-API] CooperateContext::start()
    ↓
建立连接
    ↓
开始接收输入
    ↓
协同进行中...
    ↓
主设备停止
    ↓
[N-API] CooperateContext::stop()
    ↓
[IPC] IntentionService::DeactivateCooperation()
    ↓
[IPC] CooperateServer::OnDeactivate()
    ↓
断开连接
    ↓
清理资源
```

---

## 组件交互关系

### DeviceStatusService 的依赖注入

通过 IContext 接口，DeviceStatusService 向子系统提供以下能力：

```cpp
class DeviceStatusService : public IContext {
public:
    virtual IDelegateTasks& GetDelegateTasks() override;
    virtual IDeviceManager& GetDeviceManager() override;
    virtual ITimerManager& GetTimerManager() override;
    virtual IDragManager& GetDragManager() override;
    virtual IDDMAdapter& GetDDM() override;
    virtual IPluginManager& GetPluginManager() override;
    virtual ISocketSessionManager& GetSocketSessionManager() override;
    virtual IInputAdapter& GetInput() override;
    virtual IDSoftbus& GetDSoftbus() override;
};
```

### 插件与服务的交互

插件通过 IPlugin 接口与服务交互：

```cpp
class IPlugin {
public:
    virtual int32_t Enable(Callback callback) = 0;
    virtual void Disable() = 0;
    virtual void Start(const std::string& option, callback) = 0;
    virtual void Stop() = 0;
    virtual void AddWatch(WatchType type, callback) = 0;
    virtual void RemoveWatch(WatchType type) = 0;
    virtual void SetParam(ParamType type, const ParamValue& value) = 0;
    virtual void GetParam(ParamType type, ParamValue& value, callback) = 0;
    virtual void Control(ControType type, const std::string& option) = 0;
};
```

---

## 性能优化

### 1. 事件分发优化

- **Epoll 多路复用**：单一线程处理多种事件源
- **批量任务处理**：DelegateTasks 支持批量任务投递
- **异步回调**：避免阻塞服务线程

### 2. Socket 通信优化

- **Unix Domain Socket**：低延迟数据传输
- **会话复用**：SocketSessionManager 管理连接池
- **心跳机制**：保持连接活跃

### 3. 内存管理

- **共享数据结构**：使用 Sequenceable 数据类型共享内存
- **引用计数**：智能指针管理生命周期
- **对象池**：减少动态分配

---

## 相关跳转

- **[01_Overview](01_Overview.md)** - 项目概览和核心能力
- **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构详解
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考文档
- **[05_Inner_API](05_Inner_API.md)** - 内部 API 接口定义
- **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标梳理
- **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物说明
- **[08_Security_Review](08_Security_Review.md)** - 安全风险评审
- **[09_FAQ](09_FAQ.md)** - 常见问题

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
