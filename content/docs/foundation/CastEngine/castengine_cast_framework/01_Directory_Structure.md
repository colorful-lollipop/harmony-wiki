# 目录结构与模块职责

> 文档版本: 1.0.0
> 最后更新: 2026-02-06

## 目的

本文档详细说明 CastEngine 框架的目录结构和各模块的职责边界，帮助开发者快速定位和理解代码组织。

## 完整目录树（不含测试）

```
/foundation/CastEngine/castengine_cast_framework/
├── client/                              # 客户端实现
│   ├── BUILD.gn                          # 客户端构建配置
│   ├── include/                           # 客户端内部头文件
│   │   ├── cast_service_listener_impl_stub.h
│   │   ├── cast_session.h
│   │   ├── cast_session_impl_proxy.h
│   │   ├── cast_session_listener_impl_stub.h
│   │   ├── cast_session_manager.h
│   │   ├── cast_session_manager_adaptor.h
│   │   ├── cast_session_manager_service_proxy.h
│   │   ├── mirror_player.h
│   │   ├── mirror_player_impl_proxy.h
│   │   ├── stream_player.h
│   │   ├── stream_player_impl_proxy.h
│   │   └── stream_player_listener_impl_stub.h
│   └── src/                               # 客户端实现源代码
│       ├── cast_engine_service_load_callback.cpp
│       ├── cast_service_listener_impl_stub.cpp
│       ├── cast_session.cpp
│       ├── cast_session_impl_proxy.cpp
│       ├── cast_session_listener_impl_stub.cpp
│       ├── cast_session_manager.cpp
│       ├── cast_session_manager_adaptor.cpp
│       ├── cast_session_manager_service_proxy.cpp
│       ├── mirror_player.cpp
│       ├── mirror_player_impl_proxy.cpp
│       ├── stream_player.cpp
│       ├── stream_player_impl_proxy.cpp
│       └── stream_player_listener_impl_stub.cpp
│
├── common/                              # 公共代码
│   ├── BUILD.gn                          # 公共代码构建配置
│   ├── include/private/                    # 私有头文件
│   │   ├── cast_stub_helper.h
│   │   ├── cast_meta_node_constant.h
│   │   ├── i_stream_player_listener_impl.h
│   │   ├── i_cast_session_impl.h
│   │   ├── i_mirror_player_impl.h
│   │   ├── i_stream_player_ipc.h
│   │   ├── cast_engine_log.h
│   │   ├── i_cast_session_listener_impl.h
│   │   ├── cast_service_common.h
│   │   ├── i_cast_session_manager_service.h
│   │   ├── cast_engine_common_helper.h
│   │   ├── cast_engine_dfx.h
│   │   ├── radar_constants.h
│   │   └── cast_stub_helper.h
│   └── src/                              # 公共源代码
│       ├── cast_engine_common_helper.cpp
│       └── cast_engine_dfx.cpp
│
├── etc/                                  # 系统配置
│   └── init/                              # init 配置
│       ├── BUILD.gn
│       └── cast_engine_service.cfg      # 服务启动配置
│
├── interfaces/                            # 接口层
│   ├── inner_api/                        # 内部 C++ API（对内）对
│   │   ├── BUILD.gn
│   │   └── include/                       # 公共接口头文件
│   │       ├── cast_data_source.h
│   │       ├── cast_engine_common.h
│   │       ├── cast_engine_errors.h
│   │       ├── cast_session_manager.h
│   │       ├── cast_shared_memory.h
│   │       ├── cast_shared_memory_base.h
│   │       ├── cast_shared_memory_ipc.h
│   │       ├── i_cast_session.h
│   │       ├── i_cast_session_manager_adaptor.h
│   │       ├── i_cast_session_manager_listener.h
│   │       ├── i_mirror_player.h
│   │       ├── i_stream_player.h
│   │       └── oh_remote_control_event.h
│   │
│   └── kits/js/                          # N-API（对上）对
│       ├── BUILD.gn
│       ├── include/
│       └── src/                             # N-API 实现源代码
│           ├── napi_async_work.cpp       # 异步工作队列
│           ├── napi_callback.cpp          # 回调处理
│           ├── napi_cast_session.cpp      # 会话 N-API
│           ├── napi_cast_session_listener.cpp
│           ├── napi_cast_session_manager.cpp
│           ├── napi_cast_session_manager_listener.cpp
│           ├── napi_castengine_enum.cpp   # 枚举导出
│           ├── napi_castengine_utils.cpp   # 工具函数
│           ├── napi_mirror_player.cpp     # 镜像播放器 N-API
│           ├── napi_stream_player.cpp     # 流播放器 N-API
│           ├── napi_stream_player_listener.cpp
│           └── native_module.cpp          # 模块注册入口
│
├── sa_profile/                           # System Ability 配置
│   ├── BUILD.gn
│   └── 5526.json                       # SA 配置（SA ID 5526）
│
├── service/                              # 服务端实现
│   ├── BUILD.gn                          # 服务端构建配置
│   ├── include/                           # 服务端头文件
│   │   ├── cast_session_manager_service.h
│   │   ├── cast_session_manager_service_stub.h
│   │   └── cast_service_listener_impl_proxy.h
│   │
│   └── src/                              # 服务端源代码
│       ├── cast_service_listener_impl_proxy.cpp
│       ├── cast_session_manager_service.cpp   # SA 主服务
│       ├── cast_session_manager_service_stub.cpp
│       │
│       ├── device_manager/                  # 设备管理模块
│       │   ├── BUILD.gn
│       │   ├── include/
│       │   │   ├── cast_device_data_manager.h
│       │   │   ├── connection_manager.h
│       │   │   ├── connection_manager_listener.h
│       │   │   └── discovery_manager.h
│       │   └── src/
│       │       ├── cast_device_data_manager.cpp
│       │       ├── connection_manager.cpp
│       │       └── discovery_manager.cpp
│       │
│       └── session/                      # 会话管理模块
│           ├── BUILD.gn
│           ├── include/
│           │   ├── cast_session_impl_stub.h
│           │   ├── cast_session_impl_class.h
│           │   ├── cast_session_impl.h
│           │   ├── cast_session_listener_impl_proxy.h
│           │   ├── cast_session_enums.h
│           │   └── cast_session_common.h
│           └── src/
│               ├── cast_session_impl.cpp
│               ├── cast_session_impl_stub.cpp
│               ├── cast_session_listener_impl_proxy.cpp
│               ├── cast_session_listeners.cpp
│               └── cast_session_state.cpp
│
│               ├── channel/                # 通道管理
│               │   ├── BUILD.gn
│               │   ├── include/
│               │   └── src/
│               │       ├── channel_manager.cpp
│               │       ├── softbus/
│               │       │   ├── softbus_connection.cpp
│               │       │   └── softbus_wrapper.cpp
│               │       └── tcp/
│               │           ├── tcp_connection.cpp
│               │           └── tcp_socket.cpp
│               │
│               ├── mirror/                 # 镜像播放器
│               │   ├── BUILD.gn
│               │   ├── include/
│               │   └── src/
│               │       ├── mirror_player_impl.cpp
│               │       └── mirror_player_impl_stub.cpp
│               │
│               ├── rtsp/                   # RTSP 协议
│               │   ├── BUILD.gn
│               │   ├── include/
│               │   │   ├── rtsp_basetype.h
│               │   │   ├── i_rtsp_controller.h
│               │   │   ├── rtsp_param_info.h
│               │   │   └── rtsp_listener.h
│               │   └── src/
│               │       ├── rtsp_channel_manager.cpp
│               │       ├── rtsp_controller.cpp
│               │       ├── rtsp_package.cpp
│               │       ├── rtsp_param_info.cpp
│               │       └── rtsp_parse.cpp
│               │
│               ├── stream/                 # 流播放器
│               │   ├── BUILD.gn
│               │   ├── include/
│               │   │   └── ...
│               │   └── src/
│               │       ├── cast_stream_manager_client.cpp
│               │       ├── cast_stream_manager_server.cpp
│               │       ├── i_cast_stream_manager.cpp
│               │       ├── local/
│               │       │   ├── cast_local_file_channel_client.cpp
│               │       │   ├── cast_local_file_channel_common.cpp
│               │       │   ├── cast_local_file_channel_server.cpp
│               │       │   └── local_data_source.cpp
│               │       └── player/
│               │           ├── cast_stream_player.cpp
│               │           ├── cast_stream_player_manager.cpp
│               │           ├── cast_stream_player_utils.cpp
│               │           ├── remote_player_controller.cpp
│               │           ├── stream_player_impl_stub.cpp
│               │           └── stream_player_listener_impl_proxy.cpp
│               │
│               └── utils/                  # 工具类
│                   ├── BUILD.gn
│                   ├── include/
│                   │   ├── cast_timer.h
│                   │   ├── encrypt_decrypt.h
│                   │   ├── handler.h
│                   │   ├── message.h
│                   │   ├── permission.h
│                   │   ├── state_machine.h
│                   │   └── utils.h
│                   └── src/
│                       ├── cast_timer.cpp
│                       ├── encrypt_decrypt.cpp
│                       ├── handler.cpp
│                       ├── message.cpp
│                       ├── permission.cpp
│                       ├── state_machine.cpp
│                       └── utils.cpp
│
├── BUILD.gn                             # 根构建配置
├── bundle.json                          # 组件描述和依赖
├── cast_engine.gni                      # GN 变量定义
├── hisysevent.yaml                     # HiSysEvent 配置
├── LICENSE                              # Apache License 2.0
├── README.md                           # 项目说明（英文）
└── README_zh.md                         # 项目说明（中文）
```

## 模块职责说明

### 1. client/ - 客户端模块

**职责**: 提供客户端侧的实现，包括 IPC 代理和对象封装。

**主要功能**:
- IPC 客户端代理（Proxy 类）
- CastSession、StreamPlayer、MirrorPlayer 的客户端封装
- IPC 连接管理和回调处理
- 服务可用性监听

**关键文件**:
- `cast_session_manager_service_proxy.cpp:1-300` - SA 服务代理
- `cast_session_impl_proxy.cpp:1-380` - 会话代理
- `stream_player_impl_proxy.cpp:1-270` - 流播放器代理
- `mirror_player_impl_proxy.cpp:1-220` - 镜像播放器代理

**依赖**:
- IPC 框架
- Samgr（System Ability Manager）
- 公共代码库

### 2. common/ - 公共代码模块

**职责**: 提供跨模块共享的通用工具和辅助类。

**主要功能**:
- 日志系统（统一日志接口）
- 性能监控和 DFX（Device eXperience Framework）
- 工具类（字符串处理、时间处理等）
- 共享内存管理基础

**关键文件**:
- `cast_engine_log.h` - 日志接口定义
- `cast_engine_dfx.cpp` - DFX 实现
- `cast_engine_common_helper.cpp` - 通用辅助函数

### 3. interfaces/ - 接口层

#### 3.1 interfaces/inner_api/ - 内部 C++ API

**职责**: 为框架内部（客户端/服务端）提供稳定的 C++ 接口。

**目标用户**:
- 其他 OpenHarmony 框架需要集成 CastEngine
- 系统级组件开发

**稳定性**: 这些接口被认为是稳定的内部 API，可以依赖。

**主要接口**:

| 接口 | 说明 |
|-----|------|
| ICastSession | 投屏会话接口 |
| IStreamPlayer | 流播放器接口 |
| IMirrorPlayer | 镜像播放器接口 |
| ICastSessionManager | 会话管理器接口 |
| ICastSessionManagerListener | 会话管理器监听器 |
| CastDataSource | 数据源接口 |

#### 3.2 interfaces/kits/js/ - N-API 层

**职责**: 提供 JavaScript 应用可调用的原生 API 接口。

**目标用户**:
- OpenHarmony 应用开发者
- ArkTS/TypeScript 开发者

**模块名**: `cast`
**入口文件**: `native_module.cpp:56-59`

### 4. service/ - 服务端模块

**职责**: 实现 CastEngine 的 System Ability 服务，提供核心投屏能力。

**主要功能**:
- SA 服务注册和生命周期管理
- 设备发现和连接管理
- 投屏会话管理
- 协议适配（集成外部协议库）
- 播放器管理（镜像和流）
- 权限验证

**子模块**:

#### 4.1 device_manager/ - 设备管理器

**职责**: 管理投屏设备的发现、连接和数据管理。

**关键组件**:

| 类 | 职责 |
|----|------|
| DiscoveryManager | 设备发现（WiFi Display/DLNA 等）|
| ConnectionManager | 设备连接管理 |
| CastDeviceDataManager | 设备数据存储和管理 |

**证据**: `service/src/device_manager/src/discovery_manager.cpp`

#### 4.2 session/ - 会话管理器

**职责**: 管理投屏会话的生命周期、状态和交互。

**关键组件**:

| 类 | 职责 |
|----|------|
| CastSessionImpl | 会话实现 |
| CastSessionState | 会话状态机 |
| ChannelManager | 通道管理（TCP/SoftBus）|

**子模块**:

##### channel/ - 通道管理

**职责**: 管理设备间的数据传输通道。

**实现**:
- SoftBus 通道（分布式通信）
- TCP 通道（标准网络通信）
- 通道包装和抽象层

**证据**: `service/src/session/src/channel/src/channel_manager.cpp`

##### mirror/ - 镜像播放器

**职责**: 实现屏幕镜像投屏功能。

**实现**:
- 屏幕捕获
- 视频编码
- 数据传输
- 远程控制事件处理

**证据**: `service/src/session/src/mirror/src/mirror_player_impl.cpp`

##### stream/ - 流播放器

**职责**: 实现媒体流播放功能。

**实现**:
- 媒体文件解析
- 播放控制
- 播放列表管理
- 远程控制器
- 播放器状态管理

**子模块**:
- `local/` - 本地文件播放
- `player/` - 播放器核心

**证据**: `service/src/session/src/stream/src/player/src/cast_stream_player.cpp`

##### rtsp/ - RTSP 协议

**职责**: 实现 RTSP（Real Time Streaming Protocol）协议支持。

**实现**:
- RTSP 消息封装
- RTSP 参数处理
- RTSP 消息解析
- RTSP 通道管理

**证据**: `service/src/session/src/rtsp/src/rtsp_controller.cpp`

##### utils/ - 工具类

**职责**: 提供会话模块需要的通用工具和服务。

**关键功能**:

| 工具 | 说明 |
|-----|------|
| Permission | 权限检查（MIRROR/STREAM 权限）|
| EncryptDecrypt | 加密解密功能 |
| StateMachine | 状态机实现 |
| CastTimer | 定时器管理 |
| Handler | 事件处理器 |

**权限实现**: `service/src/session/src/utils/src/permission.cpp:55-83`

**权限定义**:
- `ohos.permission.ACCESS_CAST_ENGINE_MIRROR` - 镜像投屏权限
- `ohos.permission.ACCESS_CAST_ENGINE_STREAM` - 流播放权限

### 5. sa_profile/ - SA 配置

**职责**: 定义 System Ability 的配置信息。

**文件**: `5526.json`

**配置内容**:
```json
{
    "process": "cast_engine_service",
    "systemability": [{
        "name": 5526,
        "libpath": "libcast_engine_service.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**说明**:
- SA ID: 5526
- 延迟启动（run-on-create: false）
- 非分布式 SA
- dump 级别 1

### 6. etc/init/ - 初始化配置

**职责**: 提供服务启动配置。

**文件**: `cast_engine_service.cfg`

**用途**: 定义服务的启动参数、权限和依赖。

## 模块依赖关系

### 依赖方向图

```
N-API 层 (cast)
    │
    │ depends on
    ▼
内部 API 层 (cast_engine_client.z.so)
    │
    │ depends on
    ▼
公共代码 (cast_engine_common_sources)
    ▲
    │ depends on
    │
客户端 ─────┼───── 服务端 (SA)
            │
            │ depends on
            ▼
公共代码 + 设备管理 + 会话管理
```

### 关键依赖链

#### N-API → 内部 API

```
interfaces/kits/js/src/*.cpp
    │
    ├─> #include interfaces/inner_api/include/*.h
    │
    └─> 调用内部 API 接口
```

#### 客户端 → 服务端 (IPC)

```
client/src/*_proxy.cpp
    │
    ├─> 使用 Samgr 获取 SA
    │
    ├─> 调用 IPC 接口
    │
    └─> service/src/*_stub.cpp
```

#### 服务端 → 公共代码

```
service/src/*/*.cpp
    │
    ├─> #include common/include/private/*.h
    │
    └─> 使用公共工具（日志、DFX、工具）
```

## 代码组织原则

### 1. 分层架构

- **接口层** (interfaces/): 定义对外的 API（N-API 和内部 API）
- **客户端层** (client/): 提供客户端实现和 IPC 代理
- **服务层** (service/): 提供 System Ability 服务和核心实现
- **公共层** (common/): 提供共享工具和基础功能

### 2. 模块化设计

- 每个功能模块有独立的 BUILD.gn
- 模块间通过明确接口通信
- 支持独立测试和复用

### 3. 适配器模式

- 协议适配器（DLNA/WiFi Display/Cast+Stream）作为外部模块
- 通过统一接口集成
- 易于扩展新协议

### 4. IPC 通信模式

- Proxy-Stub 模式用于客户端-服务端通信
- 基于 OpenHarmony IPC 框架
- 支持跨进程调用

## 代码定位指南

### 我想使用 CastEngine 功能开发应用

→ 阅读 [N-API 文档](03_N-API.md)

### 我想深入理解内部实现

→ 阅读 [架构文档](02_Architecture.md) 和 [内部 API 文档](04_Internal_API.md)

### 我想修改构建系统

→ 阅读 [GN Targets 文档](05_GN_Targets.md)

### 我想定位特定功能的实现

**设备发现**:
- `service/src/device_manager/src/discovery_manager.cpp`

**会话管理**:
- `service/src/cast_session_manager_service.cpp`
- `service/src/session/src/cast_session_impl.cpp`

**权限检查**:
- `service/src/session/src/utils/src/permission.cpp`

**IPC Stub**:
- `service/src/cast_session_manager_service_stub.cpp`
- `service/src/session/src/cast_session_impl_stub.cpp`

### 我想理解错误处理

→ 查阅 [错误码定义](../_work/NOTES.md#错误码定义)

---

**返回**: [SUMMARY.md](SUMMARY.md)
