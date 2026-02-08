# 目录结构与模块职责

## 目的

本文档详细说明 `device_status` 模块的目录结构、各模块的职责以及模块间的依赖关系。

---

## 目录树

```
/Volumes/lexar/code/d/work/oh/base/msdp/device_status/
├── frameworks/              # 框架代码
│   ├── native/             # Native 客户端代码
│   ├── js/                 # JS N-API 绑定
│   │   └── napi/           # N-API 模块
│   │       ├── boomerang/       # 元数据绑定模块
│   │       ├── device_status/   # 设备状态 v1 模块
│   │       ├── distance_measurement/ # 距离测量模块
│   │       ├── interaction/     # 交互模块
│   │       │   ├── drag/         # 拖拽交互
│   │       │   ├── cooperate/    # 输入设备协同
│   │       │   └── coordination/ # 协同（legacy）
│   │       ├── motion/          # 运动感知模块
│   │       ├── onscreen/        # 屏幕感知模块
│   │       ├── underage_model/  # 用户年龄模型
│   │       └── src/            # 通用 N-API 基础
│   └── ets/                # ArkTS ETS 模块
│       ├── devicestatus/       # 设备状态 ETS
│       ├── drag/               # 拖拽 ETS
│       ├── coordination/       # 协同 ETS
│       ├── motion/             # 运动 ETS
│       ├── metadataBinding/     # 元数据绑定 ETS
│       └── onScreen/           # 屏幕感知 ETS
├── interfaces/             # 对外接口存放目录
│   └── innerkits/          # Inner API
│       ├── include/           # 公共头文件
│       │   ├── stationary_data.h
│       │   ├── drag_data.h
│       │   ├── drag_data_util.h
│       │   ├── coordination_message.h
│       │   └── interaction/include/
│       │       ├── i_drag_listener.h
│       │       ├── i_coordination_listener.h
│       │       ├── i_cooperate_listener.h
│       │       ├── i_event_listener.h
│       │       ├── i_start_drag_listener.h
│       │       └── i_subscript_listener.h
│       ├── devicestatus_algorithm.h
│       ├── on_screen_data.h
│       ├── on_screen_algorithm.h
│       └── boomerang_data.h
├── services/               # 服务的代码目录
│   ├── native/             # Device Status 服务
│   │   ├── include/        # 服务头文件
│   │   └── src/            # 服务实现
│   ├── interaction/drag/       # 交互拖拽服务
│   ├── drag_auth/           # 拖拽鉴权
│   ├── communication/         # IPC 通信层
│   └── boomerang/           # Boomerang 服务
├── intention/              # Intention 框架（插件系统）
│   ├── prototype/           # 核心接口定义
│   ├── services/             # 意图服务
│   │   ├── intention_service/  # 主意图服务
│   │   └── device_manager/    # 设备管理服务
│   ├── scheduler/             # 任务调度
│   │   ├── task_scheduler/    # 异步/同步任务
│   │   ├── timer_manager/      # 定时器管理
│   │   └── plugin_manager/     # 插件生命周期管理
│   ├── stationary/         # 静止状态
│   │   ├── server/           # 服务端
│   │   ├── client/           # 客户端
│   │   └── data/             # 数据处理
│   ├── onscreen/           # 屏幕感知
│   │   ├── server/
│   │   └── client/
│   ├── drag/               # 拖拽意图
│   │   ├── server/
│   │   └── client/
│   ├── cooperate/          # 协同意图
│   │   ├── server/
│   │   ├── client/
│   │   └── plugin/          # 状态机实现
│   ├── boomerang/          # Boomerang
│   │   ├── server/
│   │   └── client/
│   ├── common/             # 公共基础设施
│   │   ├── epoll/            # Epoll 事件循环
│   │   ├── channel/          # 通信通道
│   │   └── common_event_adapter/ # 通用事件适配
│   ├── adapters/           # 外部系统适配器
│   │   ├── input_adapter/     # MMI 输入适配
│   │   ├── dsoftbus_adapter/ # DSoftBus 适配
│   │   └── ddm_adapter/      # DDM 适配
│   ├── ipc/                # IPC 通信
│   │   ├── socket/           # Socket 会话管理
│   │   ├── tunnel/           # IPC 隧道
│   │   └── sequenceable_types/ # 序列化类型
│   └── frameworks/         # 框架层
│       └── client/         # IntentionManager 客户端
├── rust/                   # Rust 实现模块
│   ├── modules/            # Rust 功能模块
│   │   ├── basic/            # 基础设备状态
│   │   ├── coordination/     # 协同
│   │   ├── drag/             # 拖拽
│   │   └── scheduler/        # 调度器
│   ├── frameworks/          # Rust 框架
│   ├── services/            # Rust 服务
│   ├── subsystem/           # Rust 子系统集成
│   │   ├── input/            # Input 子系统
│   │   ├── dsoftbus/         # DSoftBus 子系统
│   │   ├── device_profile/   # Device Profile 子系统
│   │   └── distributed_hardware/ # Distributed Hardware 子系统
│   └── data/               # Rust 数据结构
├── libs/                   # 算法库
│   ├── interface/           # 算法接口
│   ├── src/                # 算法实现
│   └── include/            # 算法头文件
├── utils/                  # 工具库
│   ├── common/             # 公共工具
│   ├── ipc/                # IPC 工具
│   ├── json_parser/         # JSON 解析
│   └── custom_config/       # 配置解析
├── tools/                  # 开发工具
│   └── vdev/               # 虚拟设备工具
├── sa_profile/             # SA 配置
│   ├── 2902.json           # C++ SA 配置
│   └── 2902_rust.json      # Rust SA 配置
├── etc/                    # 资源文件
│   └── drag_icon/          # 拖拽图标
├── wiki/                   # Wiki 文档（本目录）
├── _work/                  # Wiki 工作区
│   ├── NOTES.md            # 工作笔记
│   └── PLAN.md            # 任务计划
└── test/                   # 测试代码（不作为业务证据）
```

---

## 模块职责详解

### 1. Frameworks（框架层）

#### 1.1 Native 客户端 (`frameworks/native/`)

**职责**：提供 C++ 客户端库，供系统应用和 Native 服务使用。

**主要组件**：
- `DeviceStatusClient` - 设备状态客户端
- `StationaryManager` - 静止状态管理器
- `OnScreenManager` - 屏幕感知管理器
- `BoomerangManager` - Boomerang 管理器
- `InteractionManager` - 交互管理器
- `StreamServer` - Socket 流服务器

**关键文件**：
```
frameworks/native/src/
├── client.cpp                              # 客户端主入口
├── devicestatus_client.cpp              # 设备状态客户端实现
├── stationary_manager.cpp              # 静止状态管理器
├── on_screen_manager.cpp                # 屏幕感知管理器
├── boomerang_manager.cpp               # Boomerang 管理器
├── interaction_manager.cpp              # 交互管理器
├── stream_server.cpp                   # Socket 流服务器
└── event_handler/include/              # 事件处理
```

#### 1.2 JS N-API (`frameworks/js/napi/`)

**职责**：提供 JavaScript (Node-API) 绑定，供 JS/ETS 应用调用。

**N-API 模块列表**（共 10 个）：
1. `stationary` - 设备静止状态
2. `device_status` - 设备状态 v1
3. `motion` - 运动感知
4. `distance_measurement` - 距离测量
5. `onscreen` - 屏幕感知
6. `screen_event` - 屏幕事件
7. `underage_model` - 用户年龄模型
8. `boomerang` - 元数据绑定
9. `draginteraction` - 拖拽交互
10. `inputdevicecooperate` - 输入设备协同
11. `cooperate` - 设备协同（legacy）

**关键文件**：
```
frameworks/js/napi/src/                          # 通用 N-API
├── devicestatus_napi.cpp                      # 设备状态 N-API（legacy）
├── devicestatus_event.cpp                   # 事件处理
├── devicestatus_napi_error.cpp            # 错误处理
├── devicestatus_device_status_napi.cpp     # 设备状态 v1
├── boomerang/                                # Boomerang 模块
│   ├── boomerang_napi.cpp                # Boomerang N-API
│   ├── boomerang_event.cpp                # Boomerang 事件
│   └── boomerang_napi_error.cpp         # Boomerang 错误处理
├── motion/                                   # 运动模块
│   ├── motion_napi.cpp                     # 运动 N-API
│   ├── motion_event_napi.cpp               # 运动事件
│   └── motion_napi_error.cpp              # 运动错误处理
├── onscreen/                                 # 屏幕感知模块
│   ├── on_screen_napi.cpp                # 屏幕感知 N-API
│   ├── screen_event_napi.cpp              # 屏幕事件 N-API
│   └── on_screen_napi_error.cpp          # 屏幕错误处理
├── distance_measurement/                       # 距离测量模块
│   ├── distance_measurement_napi.cpp         # 距离测量 N-API
│   ├── distance_measurement_event_napi.cpp   # 距离事件
│   └── distance_measurement_napi_error.cpp  # 距离错误处理
├── underage_model/                           # 用户年龄模型
│   ├── underage_model_napi.cpp              # 用户状态 N-API
│   ├── underage_model_napi_event.cpp       # 用户状态事件
│   └── underage_model_napi_error.cpp       # 用户状态错误处理
└── interaction/                              # 交互模块
    ├── drag/                             # 拖拽
    │   ├── native_register_module.cpp       # 拖拽 N-API 注册
    │   ├── js_drag_context.cpp             # 拖拽上下文
    │   └── js_drag_manager.cpp            # 拖拽管理器
    ├── cooperate/                          # 协同
    │   ├── native_register_module.cpp       # 协同 N-API 注册
    │   ├── js_cooperate_context.cpp          # 协同上下文
    │   └── js_cooperate_manager.cpp           # 协同管理器
    └── coordination/                       # 协同（legacy）
        └── native_register_module.cpp       # 协同 N-API 注册
```

#### 1.3 ETS/ANI (`frameworks/ets/`)

**职责**：提供 ArkTS/ANI 绑定，供 ETS 应用使用。

**主要模块**：
- `devicestatus` - 设备状态 ANI
- `drag` - 拖拽 ANI
- `coordination` - 协同 ANI
- `motion` - 运动 ANI
- `metadataBinding` - 元数据绑定 ANI
- `onScreen` - 屏幕感知 ANI

**产物**：
- 共享库 (`.so`)
- Ark Bytecode (`.abc`)

---

### 2. Services（服务层）

#### 2.1 Device Status 服务 (`services/native/`)

**职责**：实现主 SystemAbility 服务（SA 2902），管理所有设备状态感知功能。

**主要组件**：
- `DeviceStatusService` - 主服务类
- `DeviceStatusManager` - 设备状态管理器
- `DeviceStatusSrvStub` - IPC 请求处理器
- `StreamServer` - Socket 流服务器
- `DeviceStatusDumper` - 调试信息转储器

**关键文件**：
```
services/native/src/
├── devicestatus_service.cpp                 # 主服务实现
├── devicestatus_manager.cpp                # 设备状态管理
├── devicestatus_srv_stub.cpp              # IPC Stub
├── devicestatus_msdp_client_impl.cpp       # MSDP 客户端实现
├── stream_server.cpp                     # Socket 流服务器
├── devicestatus_dumper.cpp               # 调试转储
├── devicestatus_hisysevent.cpp            # Hisysevent 上报
└── devicestatus_napi_manager.cpp           # N-API 管理器
```

#### 2.2 Interaction 服务 (`services/interaction/drag/`)

**职责**：实现拖拽交互功能，包括拖拽启动、停止、预览、动画等。

**主要组件**：
- `DragManager` - 拖拽管理器
- `DragServer` - 拖拽服务器
- `DragDrawing` - 拖拽绘制和动画
- `DragDataManager` - 拖拽数据管理
- `DragAuth` - 拖拽鉴权

**关键文件**：
```
services/interaction/drag/src/
├── drag_manager.cpp                        # 拖拽管理器
├── drag_server.cpp                         # 拖拽服务器
├── drag_drawing.cpp                       # 拖拽绘制
├── drag_data_manager.cpp                  # 拖拽数据管理
├── drag_smooth_processor.cpp             # 拖拽平滑处理
├── event_hub.cpp                          # 事件枢纽
├── display_change_event_listener.cpp       # 显示变化监听
├── app_state_observer.cpp                # 应用状态观察
├── collaboration_service_status_change.cpp  # 协同服务变化
├── drag_vsync_station.cpp                # VSync 站点
├── drag_hisysevent.cpp                   # Hisysevent 上报
├── state_change_notify.cpp                # 状态变化通知
├── drag_smooth_processor.cpp             # 拖拽平滑处理
└── pull_throw_listener.cpp                  # 投掷监听
```

#### 2.3 通信层 (`services/communication/`)

**职责**：提供 IPC 通信基础设施。

**主要组件**：
- `Idevicestatus` - 设备状态服务接口
- `DeviceStatusSrvStub` - 服务端 IPC Stub
- 各种序列化数据类型

---

### 3. Intention 框架（插件系统）

#### 3.1 核心接口 (`intention/prototype/`)

**职责**：定义所有插件系统的核心接口。

**主要接口**：
- `IContext` - 中央上下文接口，聚合所有子系统
- `IPlugin` - 插件接口，定义插件生命周期
- `IPluginManager` - 插件管理器接口
- `ICooperate` - 跨设备协同接口
- `IDragManager` - 拖拽管理器接口
- `IMotionDrag` - 运动拖拽接口
- `IDeviceManager` - 设备管理器接口
- `ITimerManager` - 定时器管理器接口
- `ISocketSessionManager` - Socket 会话管理器接口
- `IDSoftbusAdapter` - DSoftBus 适配器接口
- `IDDMAdapter` - Distributed Device Manager 适配器接口
- `ICommonEventAdapter` - Common Event Service 适配器接口
- `IInputAdapter` - MMI 输入适配器接口

#### 3.2 意图服务 (`intention/services/intention_service/`)

**职责**：高层级业务逻辑服务，协调所有插件和服务。

**主要组件**：
- `IntentionService` - 主意图服务类
- 各种插件实例管理

#### 3.3 功能插件 (`intention/` 各功能目录)

**Cooperate** - 跨设备协同：
- `server/` - CooperateServer 实现
- `client/` - CooperateClient 实现
- `plugin/` - 状态机实现（CooperateFree、CooperateIn、CooperateOut）

**Drag** - 拖拽意图：
- `server/` - DragServer 实现
- `client/` - DragClient 实现

**Stationary** - 静止状态：
- `server/` - StationaryServer 实现
- `client/` - StationaryClient 实现
- `data/` - 数据处理

**Boomerang** - 元数据绑定：
- `server/` - BoomerangServer 实现
- `client/` - BoomerangClient 实现

**OnScreen** - 屏幕感知：
- `server/` - OnScreenServer 实现
- `client/` - OnScreenClient 实现

#### 3.4 基础设施 (`intention/common/` & `intention/ipc/`)

**主要组件**：
- `Epoll` - Epoll 事件循环实现
- `SocketSessionManager` - Socket 会话管理
- `SocketConnection` / `SocketSession` - Socket 连接管理
- `Channel` - 通信通道
- `Tunnel` - IPC 隧道

#### 3.5 调度系统 (`intention/scheduler/`)

**主要组件**：
- `TaskScheduler` - 异步/同步任务调度器
- `TimerManager` - 定时器管理器
- `PluginManager` - 插件生命周期管理

#### 3.6 适配器 (`intention/adapters/`)

**主要适配器**：
- `InputAdapter` - MMI 输入适配器
- `DSoftbusAdapter` - Distributed SoftBus 适配器
- `DDMAdapter` - Distributed Device Manager 适配器
- `CommonEventAdapter` - Common Event Service 适配器

---

### 4. Rust 模块

**职责**：提供 Rust 实现的替代方案（实验性，默认禁用）。

**主要模块**：
- `modules/` - Rust 功能模块
  - `basic/` - 基础设备状态
  - `coordination/` - 协同
  - `drag/` - 拖拽
  - `scheduler/` - 调度器
- `frameworks/` - Rust 框架
- `services/` - Rust 服务
- `subsystem/` - 子系统集成
  - `input/` - Input 子系统绑定
  - `dsoftbus/` - DSoftBus 子系统绑定
  - `device_profile/` - Device Profile 子系统绑定
  - `distributed_hardware/` - Distributed Hardware 子系统绑定
- `data/` - Rust 数据结构

---

### 5. Utils（工具库）

**职责**：提供公共工具和基础设施。

**主要工具**：
- `common/` - 公共工具（错误码、日志、配置等）
- `ipc/` - IPC 工具
- `json_parser/` - JSON 解析工具
- `custom_config/` - 配置解析工具

---

### 6. Libs（算法库）

**职责**：提供 MSDP 算法库实现。

**主要组件**：
- `algorithm/` - 算法实现（绝对静止、水平、垂直、相对静止）
- `devicestatus_msdp_mock.cpp` - Mock 实现（用于测试）

---

### 7. Interfaces（接口层）

**职责**：定义 Inner API，供系统内部使用。

**主要接口**：
- `Idevicestatus` - 设备状态服务接口
- `IRemoteDevStaCallback` - 静止状态回调接口
- `IRemoteBoomerangCallback` - Boomerang 回调接口
- `IRemoteOnScreenCallback` - 屏幕感知回调接口
- `IDragListener` - 拖拽监听接口
- `ICoordinationListener` - 协同监听接口
- `ICooperateListener` - 协同监听接口
- `IEventListener` - 通用事件监听接口
- `IStartDragListener` - 启动拖拽监听接口
- `ISubscriptListener` - 脚本样式监听接口
- `IHotAreaListener` - 热区监听接口

---

## 模块依赖关系

### 依赖方向

```
应用层
    ↓
[N-API / ETS] → Inner Kits
    ↓
Native Client → Intention Client
    ↓
Socket Client → IPC
    ↓
└────────── Device Status Service (SA 2902) ──────┘
                    ↓
        ┌─────────────────────────────────────┐
        │   Intention Service               │
        │  (业务逻辑编排)               │
        ├──────────┬───────────────────┤
        │  Drag  │ Cooperate │ Stationary  │
        │ Server │ Server    │  Server     │
        └──────────┴───────────────────┘
                    ↓
        ┌─────────────────────────────────────┐
        │  基础设施层                   │
        ├──────────┬───────────────────┤
        │ Adapters│ IPC 基础 │  调度 │  设备管理 │
        └──────────┴───────────────────┘
```

### 模块边界

| 模块 | 边界定义 | 稳定性 |
|--------|-----------|---------|
| **Intention 框架** | `intention/` 目录 | 稳定 - 插件化架构，内部模块间通过接口交互 |
| **Services** | `services/` 目录 | 稳定 - 作为 System Ability 运行，通过 SA 框架管理生命周期 |
| **Frameworks** | `frameworks/` 目录 | 稳定 - 提供客户端 API，不直接访问硬件 |
| **Rust 模块** | `rust/` 目录 | 稳定 - 独立实现，可选的替代方案 |

---

## 模块间通信

### 1. IPC 通信

- **Binder IPC**：用于客户端与主服务的通信
- **Unix Domain Socket**：用于 Intention 框架内的高性能数据传输

### 2. Socket 通信

- **SocketServer**：管理 Unix Domain Socket 连接
- **SocketSessionManager**：管理 Socket 会话生命周期

### 3. DSoftBus 通信

- **IDSoftbusAdapter**：封装 DSoftBus API
- 支持设备发现和跨设备通信

---

## 相关跳转

- **[03_Architecture](03_Architecture.md)** - 完整架构设计说明，包含组件图、数据流
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考文档
- **[05_Inner_API](05_Inner_API.md)** - 内部 API 接口定义

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
