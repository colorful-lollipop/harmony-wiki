# 03_Architecture - 架构设计与数据流

## 概述

本文档描述 multimodalinput_input 子系统的整体架构设计、组件关系和数据流。

## 代码证据

**核心架构文件**:
- `service/module_loader/include/mmi_service.h` - 主服务定义
- `service/event_handler/include/input_event_handler.h` - 事件处理器
- `intention/ipc/tunnel/include/i_intention.h` - IPC 接口定义

---

## 3.1 系统架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              应用层 (Applications)                            │
├─────────────────────────────────────────────────────────────────────────────┤
│  JS/ArkTS 应用                                                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │ 输入事件监听     │  │ 设备状态监控     │  │ 事件注入        │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
│           │                    │                    │                       │
└───────────┼────────────────────┼────────────────────┼───────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           N-API 绑定层 (frameworks/napi)                     │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │ inputEventClient│  │  inputDevice    │  │  inputMonitor   │             │
│  │  事件注入模块    │  │  设备管理模块    │  │  事件监控模块    │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
│           │                    │                    │                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │    pointer      │  │ inputConsumer   │  │ infraredEmitter │             │
│  │  指针配置模块    │  │  按键消费模块    │  │  红外控制模块    │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
└───────────┼────────────────────┼────────────────────┼───────────────────────┘
            │                    │                    │
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        客户端代理层 (frameworks/proxy)                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    InputManagerImpl                                   │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │   │
│  │  │ EventHandler    │  │ DeviceManager   │  │ 订阅管理        │       │   │
│  │  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘       │   │
│  └───────────┼────────────────────┼────────────────────┼────────────────┘   │
│              │                    │                    │                      │
└──────────────┼────────────────────┼────────────────────┼────────────────────┘
               │                    │                    │
               │ IRemoteProxy       │                    │
               ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         IPC 通信层 (intention/ipc)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │ ConnectManager  │  │ IntentionProxy  │  │ UDS Socket      │             │
│  │  连接管理        │  │  意图代理        │  │  Unix域套接字    │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
│           │                    │                    │                       │
└───────────┼────────────────────┼────────────────────┼───────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          服务层 (MMIService SA: 3101)                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     MMIService                                        │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │   │
│  │  │ SystemAbility   │  │ UDSServer       │  │ConnectStub     │       │   │
│  │  │  系统能力基类    │  │  套接字服务      │  │  连接存根       │       │   │
│  │  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘       │   │
│  └───────────┼────────────────────┼────────────────────┼────────────────┘   │
│              │                    │                    │                      │
└──────────────┼────────────────────┼────────────────────┼────────────────────┘
               │                    │                    │
               ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      事件处理责任链 (InputEventHandler)                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│   │EventNormalize│─▶│ EventFilter  │─▶│Interceptor  │─▶│  KeyCommand  │  │
│   │   Handler    │  │   Handler    │  │   Handler   │  │   Handler    │  │
│   └──────────────┘  └──────────────┘  └──────────────┘  └──────┬───────┘  │
│         │                                    │                     │           │
│         ▼                                    ▼                     ▼           │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│   │KeySubscriber │─▶│EventMonitor  │─▶│EventDispatch│─▶│   Client     │  │
│   │   Handler    │  │   Handler    │  │   Handler   │  │   (应用)     │  │
│   └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         设备抽象层 (libinput adapter)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │   libinput      │  │   HID 驱动      │  │   输入设备      │             │
│  │   事件采集      │  │   硬件交互      │  │   触摸屏/鼠标   │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
└───────────┼────────────────────┼────────────────────┼───────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           硬件层 (Hardware)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  触摸屏    │  鼠标    │  键盘    │  触摸板    │  游戏手柄    │  红外       │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3.2 系统能力 (SA) 配置

### 3.2.1 SA 定义

**配置文件**: `sa_profile/3101.json`

```json
{
  "process": "multimodalinput",
  "systemability": [
    {
      "name": 3101,
      "libpath": "libmmi-server.z.so",
      "run-on-create": true,
      "distributed": false,
      "dump_level": 1
    }
  ]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| SA ID | 3101 | 系统能力标识 |
| libpath | `libmmi-server.z.so` | 服务实现库 |
| run-on-create | true | 随需创建时启动 |
| distributed | false | 非分布式服务 |
| dump_level | 1 | 支持 dump 能力 |

### 3.2.2 服务启动流程

```
1. SAMGR 检测到 SA 3001 请求
         │
         ▼
2. 加载 libmmi-server.z.so
         │
         ▼
3. MMIService 构造函数
         │
         ▼
4. OnStart() 被调用
   ├── InitLibinputService()  ← 初始化 libinput
   ├── InitService()          ← 初始化 epoll + socket
   └── AddSystemAbilityListener() ← 监听其他 SA
```

---

## 3.3 IPC 通信机制

### 3.3.1 IPC 组件

| 组件 | 类型 | 用途 |
|------|------|------|
| `IRemoteStub` | 服务端存根 | 处理客户端 IPC 请求 |
| `IRemoteProxy` | 客户端代理 | 向服务端发送请求 |
| `MessageParcel` | 数据打包 | 跨进程数据传输 |
| `UDSServer` | Unix 域套接字 | 高效事件传输 |

### 3.3.2 核心 IPC 接口

**IIntention 接口** (`intention/ipc/tunnel/include/i_intention.h`):

```cpp
class IIntention : public IRemoteBroker {
    virtual int32_t Enable(Intention intention, ...) = 0;
    virtual int32_t Disable(Intention intention, ...) = 0;
    virtual int32_t Start(Intention intention, ...) = 0;
    virtual int32_t Stop(Intention intention, ...) = 0;
    virtual int32_t AddWatch(Intention intention, ...) = 0;
    virtual int32_t RemoveWatch(Intention intention, ...) = 0;
    virtual int32_t SetParam(Intention intention, ...) = 0;
    virtual int32_t GetParam(Intention intention, ...) = 0;
    virtual int32_t Control(Intention intention, ...) = 0;
};
```

### 3.3.3 连接管理器

**文件**: `service/connect_manager/`

| 组件 | 功能 |
|------|------|
| `MultimodalInputConnectStub` | IPC 服务端 |
| `MultimodalInputConnectProxy` | IPC 客户端 |
| `ConnectManager` | 连接状态管理 |

---

## 3.4 事件处理责任链

### 3.4.1 Handler 列表

| Handler | 职责 | 是否可选 |
|---------|------|---------|
| `EventNormalizeHandler` | 事件归一化 | 核心必需 |
| `EventFilterHandler` | 事件过滤 | 可选 (feature) |
| `EventInterceptorHandler` | 事件拦截 | 可选 (feature) |
| `KeyCommandHandler` | 按键命令 | 可选 (feature) |
| `KeySubscriberHandler` | 按键订阅 | 可选 (feature) |
| `EventMonitorHandler` | 事件监控 | 可选 (feature) |
| `EventDispatchHandler` | 事件分发 | 核心必需 |

### 3.4.2 事件流处理

```
原始输入事件 (libinput)
        │
        ▼
┌───────────────────┐
│ EventNormalizeHandler │
│  (libinput → MMI) │ ← 事件格式转换
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  EventFilterHandler│  ← 按条件过滤
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│InterceptorHandler │  ← 预拦截
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  KeyCommandHandler │  ← 快捷键匹配
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│KeySubscriberHandler│ ← 按键订阅回调
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ EventMonitorHandler│ ← 监控回调
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│EventDispatchHandler│ → UDS Socket → 客户端
└───────────────────┘
```

---

## 3.5 线程模型

### 3.5.1 线程划分

| 线程 | 职责 | 数量 |
|------|------|------|
| Main Thread | 服务主循环 | 1 |
| libinput Thread | 设备事件读取 | 1 |
| Event Handler Thread | 事件处理 | 1-N |
| IPC Thread | IPC 请求处理 | 1 |

### 3.5.2 线程安全

- **InputManagerImpl**: 单例模式，线程安全
- **EventHandler**: 串行处理事件
- **UDSServer**: 异步 I/O (epoll)

---

## 3.6 数据流示例

### 3.6.1 触摸事件流

```
1. 触摸硬件产生中断
         │
         ▼
2. libinput 读取事件 → EventNormalizeHandler
         │
         ▼
3. 归一化为 PointerEvent
         │
         ▼
4. 责任链处理 (过滤 → 拦截 → 分发)
         │
         ▼
5. EventDispatchHandler 通过 UDS 发送
         │
         ▼
6. 客户端 InputManager 接收
         │
         ▼
7. N-API 回调通知 JS 层
```

### 3.6.2 事件注入流

```
1. JS 调用 injectEvent()
         │
         ▼
2. N-API 层参数校验
         │
         ▼
3. InputManagerImpl → ConnectProxy
         │
         ▼
4. IPC 调用 MMIService
         │
         ▼
5. 权限检查 (PermissionHelper)
         │
         ▼
6. 注入事件到处理链
         │
         ▼
7. 分发到目标窗口
```

---

## 3.7 意图服务 (Intention)

### 3.7.1 意图类型

| Intention | 功能 | 实现类 |
|-----------|------|--------|
| Cooperate | 跨设备协作 | `CooperateServer` |
| Drag | 拖拽操作 | `DragServer` |
| Socket | 套接字通信 | `SocketServer` |

### 3.7.2 跨设备协作流程

```
设备 A 应用                         设备 B
    │                                 │
    │──── CooperateRequest ─────────▶│
    │                                 │
    ◀─── Event Transfer ─────────────│
    │                                 │
    ◀─── Sync Complete ──────────────│
```

---

## 3.8 关键组件位置

| 组件 | 路径 |
|------|------|
| MMIService | `service/module_loader/src/mmi_service.cpp` |
| InputEventHandler | `service/event_handler/src/input_event_handler.cpp` |
| EventNormalizeHandler | `service/event_handler/src/event_normalize_handler.cpp` |
| InputManager | `interfaces/native/innerkits/proxy/include/input_manager.h` |
| InputManagerImpl | `frameworks/proxy/event_handler/include/input_manager_impl.h` |
| UDSServer | `intention/ipc/socket/include/uds_server.h` |
| PermissionHelper | `service/permission_helper/include/permission_helper.h` |
