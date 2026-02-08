# 02_Directory_Structure - 目录结构与模块职责

> 本文档详细说明 Cast+ Stream 模块的目录结构、文件组织和各模块职责。

---

## 1. 顶层目录结构

```
/foundation/CastEngine/castengine_cast_plus_stream
├── include/                          # 公共接口头文件（对外暴露）
├── src/                              # 源码实现
│   ├── channel/                      # 通道管理模块
│   ├── mirror/                       # 镜像播放模块
│   ├── rtsp/                         # RTSP 协议模块
│   ├── stream/                       # 流媒体模块
│   └── utils/                        # 工具模块
├── BUILD.gn                          # 根构建文件
├── LICENSE                           # Apache 2.0 许可证
├── README.md                         # 英文说明
└── README_zh.md                      # 中文说明
```

---

## 2. 目录详细说明

### 2.1 include/ - 公共接口头文件

**路径**: `/foundation/CastEngine/castengine_cast_plus_stream/include/`

**职责**: 定义对外暴露的接口，供父级框架或其他模块使用

| 文件 | 说明 | 关键内容 |
|------|------|----------|
| `cast_session_common.h` | 公共数据结构 | `SessionState` 枚举、`CastRemoteDeviceInfo` 结构体 |
| `cast_session_enums.h` | 消息 ID 枚举 | `MessageId` 枚举定义 |
| `cast_session_impl_class.h` | 主类定义 | `CastSessionImpl` 类声明 |
| `cast_session_impl.h` | 内部类定义 | 状态类、Listener 类定义 |
| `cast_session_impl_stub.h` | IPC Stub 定义 | `CastSessionImplStub` 类 |
| `cast_session_listener_impl_proxy.h` | IPC Proxy 定义 | `CastSessionListenerImplProxy` 类 |

**依赖关系**:
```
include/*.h → 依赖父级框架头文件（i_cast_session_impl.h 等）
          → 被 src/ 下所有实现文件包含
```

### 2.2 src/ - 源码实现

#### 2.2.1 src/channel/ - 通道管理模块

**路径**: `/foundation/CastEngine/castengine_cast_plus_stream/src/channel/`

**职责**: 管理设备间通信通道，支持 SoftBus 和 TCP 两种传输方式

```
src/channel/
├── include/                          # 模块内部头文件
│   ├── channel.h                     # Channel 基类
│   ├── channel_info.h                # 通道信息枚举（LinkType, ModuleType）
│   ├── channel_listener.h            # 通道数据监听接口
│   ├── channel_manager.h             # ChannelManager 类
│   ├── channel_manager_listener.h    # 通道生命周期监听接口
│   ├── channel_request.h             # 通道请求参数
│   ├── connection.h                  # Connection 基类
│   └── connection_listener.h         # 连接状态监听接口
├── src/                              # 实现
│   ├── channel_manager.cpp           # 通道管理器实现
│   ├── softbus/                      # SoftBus 传输实现
│   │   ├── softbus_connection.cpp    # SoftBus 连接实现
│   │   ├── softbus_connection.h      # SoftBusConnection 类
│   │   ├── softbus_wrapper.cpp       # SoftBus SDK 包装
│   │   └── softbus_wrapper.h         # SoftBusWrapper 类
│   └── tcp/                          # TCP 传输实现
│       ├── tcp_connection.cpp        # TCP 连接实现
│       ├── tcp_connection.h          # TcpConnection 类
│       ├── tcp_socket.cpp            # Socket 包装
│       └── tcp_socket.h              # TcpSocket 类
└── BUILD.gn                          # 模块构建文件
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `ChannelManager` | `channel_manager.cpp` | 工厂模式创建连接，管理连接生命周期 |
| `SoftBusConnection` | `softbus_connection.cpp` | SoftBus 连接实现 |
| `TcpConnection` | `tcp_connection.cpp` | TCP 连接实现 |
| `SoftBusWrapper` | `softbus_wrapper.cpp` | SoftBus SDK 包装 |
| `TcpSocket` | `tcp_socket.cpp` | BSD Socket 包装 |

**依赖关系**:
```
channel → utils (加密、工具)
        → device_manager (设备发现)
        → dsoftbus (SoftBus SDK)
```

#### 2.2.2 src/rtsp/ - RTSP 协议模块

**路径**: `/foundation/CastEngine/castengine_cast_plus_stream/src/rtsp/`

**职责**: 实现 RTSP 协议，负责会话控制和参数协商

```
src/rtsp/
├── include/                          # 公共头文件
│   ├── i_rtsp_controller.h           # RTSP 控制器接口
│   ├── rtsp_basetype.h               # 基础类型和常量
│   ├── rtsp_listener.h               # RTSP 事件监听接口
│   └── rtsp_param_info.h             # 参数信息结构
├── src/                              # 实现
│   ├── rtsp_channel_manager.cpp      # RTSP 通道管理
│   ├── rtsp_channel_manager.h        # RtspChannelManager 类
│   ├── rtsp_controller.cpp           # RTSP 控制器实现
│   ├── rtsp_controller.h             # RtspController 类
│   ├── rtsp_listener_inner.h         # 内部监听接口
│   ├── rtsp_package.cpp              # RTSP 消息封装
│   ├── rtsp_package.h                # RtspEncap 类
│   ├── rtsp_param_info.cpp           # 参数信息实现
│   ├── rtsp_parse.cpp                # RTSP 消息解析
│   └── rtsp_parse.h                  # RtspParse 类
└── BUILD.gn                          # 模块构建文件
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `RtspController` | `rtsp_controller.cpp` | RTSP 状态机，请求/响应处理 |
| `RtspChannelManager` | `rtsp_channel_manager.cpp` | RTSP 通道管理，加密处理 |
| `RtspParse` | `rtsp_parse.cpp` | RTSP 消息解析 |
| `RtspEncap` | `rtsp_package.cpp` | RTSP 消息封装 |
| `ParamInfo` | `rtsp_param_info.cpp` | 参数信息存储 |

**RTSP 方法支持**:
| 方法 | 处理函数 | 说明 |
|------|----------|------|
| OPTIONS | `ProcessOptionRequest()` | 能力查询 |
| SETUP | `ProcessSetupRequest()` | 会话建立 |
| PLAY | `ProcessPlayRequest()` | 播放控制 |
| PAUSE | `ProcessPauseRequest()` | 暂停控制 |
| TEARDOWN | `ProcessTearDownRequest()` | 会话终止 |
| SET_PARAMETER | `ProcessSetParamRequest()` | 参数设置 |
| GET_PARAMETER | `ProcessGetParameterRequestM3()` | 参数获取 |
| ANNOUNCE | `ProcessAnnounceRequest()` | 加密协商 |

**依赖关系**:
```
rtsp → channel (通道传输)
     → utils (加密、工具)
     → openssl (AES 加密)
```

#### 2.2.3 src/stream/ - 流媒体模块

**路径**: `/foundation/CastEngine/castengine_cast_plus_stream/src/stream/`

**职责**: 实现流媒体播放控制，包括播放器和本地文件传输

```
src/stream/
├── include/                          # 公共头文件
│   ├── cast_stream_common.h          # 流媒体公共定义
│   ├── cast_stream_manager_client.h  # 客户端管理器
│   ├── cast_stream_manager_server.h  # 服务端管理器
│   ├── i_cast_stream_listener.h      # 流事件监听接口
│   ├── i_cast_stream_manager.h       # 管理器基类
│   ├── i_cast_stream_manager_client.h # 客户端接口
│   └── i_cast_stream_manager_server.h # 服务端接口
├── src/                              # 实现
│   ├── cast_stream_manager_client.cpp # 客户端实现
│   ├── cast_stream_manager_server.cpp # 服务端实现
│   ├── i_cast_stream_manager.cpp     # 基类实现
│   ├── local/                        # 本地文件传输
│   │   ├── include/
│   │   │   ├── cast_local_file_channel_client.h
│   │   │   ├── cast_local_file_channel_server.h
│   │   │   ├── i_cast_local_file_channel.h
│   │   │   ├── i_data_listener.h
│   │   │   └── local_data_source.h
│   │   └── src/
│   │       ├── cast_local_file_channel_client.cpp
│   │       ├── cast_local_file_channel_common.cpp
│   │       ├── cast_local_file_channel_server.cpp
│   │       └── local_data_source.cpp
│   └── player/                       # 播放器实现
│       ├── include/
│       │   ├── cast_stream_player.h
│       │   ├── cast_stream_player_common.h
│       │   ├── cast_stream_player_manager.h
│       │   ├── cast_stream_player_utils.h
│       │   ├── i_stream_player_impl.h
│       │   ├── remote_player_controller.h
│       │   ├── stream_player_impl_stub.h
│       │   └── stream_player_listener_impl_proxy.h
│       └── src/
│           ├── cast_stream_player.cpp
│           ├── cast_stream_player_manager.cpp
│           ├── cast_stream_player_utils.cpp
│           ├── remote_player_controller.cpp
│           ├── stream_player_impl_stub.cpp
│           └── stream_player_listener_impl_proxy.cpp
└── BUILD.gn                          # 模块构建文件
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `CastStreamManagerClient` | `cast_stream_manager_client.cpp` | Source 端管理器 |
| `CastStreamManagerServer` | `cast_stream_manager_server.cpp` | Sink 端管理器 |
| `CastStreamPlayer` | `cast_stream_player.cpp` | 播放器实现 |
| `CastStreamPlayerManager` | `cast_stream_player_manager.cpp` | 播放器管理 |
| `RemotePlayerController` | `remote_player_controller.cpp` | 远程播放控制 |
| `CastLocalFileChannelServer` | `cast_local_file_channel_server.cpp` | 文件服务 |
| `CastLocalFileChannelClient` | `cast_local_file_channel_client.cpp` | 文件客户端 |
| `LocalDataSource` | `local_data_source.cpp` | 本地数据源（带缓存） |

**依赖关系**:
```
stream → channel (通道传输)
       → utils (工具)
       → player_framework (系统播放器)
       → audio_framework (系统音频)
```

#### 2.2.4 src/mirror/ - 镜像播放模块

**路径**: `/foundation/CastEngine/castengine_cast_plus_stream/src/mirror/`

**职责**: 实现屏幕镜像功能，包括视频采集、编码和音频采集

```
src/mirror/
├── include/                          # 公共头文件
│   ├── mirror_player_impl.h          # 镜像播放器实现
│   └── mirror_player_impl_stub.h     # IPC Stub
├── src/                              # 实现
│   ├── mirror_player_impl.cpp        # 镜像播放器实现
│   └── mirror_player_impl_stub.cpp   # Stub 实现
└── BUILD.gn                          # 模块构建文件
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `MirrorPlayerImpl` | `mirror_player_impl.cpp` | 镜像播放控制 |
| `MirrorPlayerImplStub` | `mirror_player_impl_stub.cpp` | IPC 存根 |

**依赖关系**:
```
mirror → channel (通道传输)
       → rtsp (协议控制)
       → stream (流媒体)
       → utils (工具)
       → graphic_2d (图形渲染)
       → window_manager (窗口管理)
       → av_codec (音视频编解码)
```

#### 2.2.5 src/utils/ - 工具模块

**路径**: `/foundation/CastEngine/castengine_cast_plus_stream/src/utils/`

**职责**: 提供通用工具类，包括加密、状态机、消息队列等

```
src/utils/
├── include/                          # 公共头文件
│   ├── cast_timer.h                  # 定时器
│   ├── encrypt_decrypt.h             # 加密解密
│   ├── handler.h                     # 消息处理器
│   ├── message.h                     # 消息定义
│   ├── permission.h                  # 权限检查
│   ├── state_machine.h               # 状态机
│   └── utils.h                       # 通用工具
└── src/                              # 实现
    ├── cast_timer.cpp                # 定时器实现
    ├── encrypt_decrypt.cpp           # 加密实现
    ├── handler.cpp                   # 处理器实现
    ├── message.cpp                   # 消息实现
    ├── permission.cpp                # 权限实现
    ├── state_machine.cpp             # 状态机实现
    └── utils.cpp                     # 工具实现
└── BUILD.gn                          # 模块构建文件
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `EncryptDecrypt` | `encrypt_decrypt.cpp` | AES-128 加密解密 |
| `Permission` | `permission.cpp` | 权限检查 |
| `StateMachine` | `state_machine.cpp` | 通用状态机框架 |
| `State` | `state_machine.cpp` | 状态基类 |
| `Handler` | `handler.cpp` | 消息处理器 |
| `Message` | `message.cpp` | 消息封装 |
| `CastTimer` | `cast_timer.cpp` | 定时器 |

**依赖关系**:
```
utils → openssl (加密)
      → glib (Base64)
      → access_token (权限 SDK)
```

---

## 3. 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          模块依赖关系图                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              ┌─────────────┐                                │
│                              │    Root     │                                │
│                              │ cast_session│                                │
│                              └──────┬──────┘                                │
│                                     │                                        │
│         ┌───────────────────────────┼───────────────────────────┐           │
│         │                           │                           │           │
│         ▼                           ▼                           ▼           │
│  ┌─────────────┐            ┌─────────────┐            ┌─────────────┐     │
│  │   channel   │◄───────────│   utils     │───────────►│    rtsp     │     │
│  │  (通道管理)  │            │  (工具库)   │            │  (RTSP协议) │     │
│  └──────┬──────┘            └─────────────┘            └──────┬──────┘     │
│         │                                                     │            │
│         │            ┌─────────────┐                          │            │
│         └───────────►│   stream    │◄─────────────────────────┘            │
│                      │ (流媒体)    │                                       │
│                      └──────┬──────┘                                       │
│                             │                                              │
│                             ▼                                              │
│                      ┌─────────────┐                                       │
│                      │   mirror    │                                       │
│                      │  (镜像播放) │                                       │
│                      └─────────────┘                                       │
│                                                                              │
│  依赖方向说明: 箭头指向被依赖方                                             │
│  - channel 依赖 utils                                                      │
│  - rtsp 依赖 channel 和 utils                                              │
│  - stream 依赖 channel 和 utils                                            │
│  - mirror 依赖 channel、rtsp、stream、utils                                │
│  - root (cast_session) 依赖所有子模块                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 源文件清单

### 4.1 核心会话文件

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| `src/cast_session_impl.cpp` | ~1250 | 主会话实现 |
| `src/cast_session_impl_stub.cpp` | ~286 | IPC Stub 实现 |
| `src/cast_session_listener_impl_proxy.cpp` | ~103 | IPC Proxy 实现 |
| `src/cast_session_state.cpp` | ~600 | 状态机实现 |
| `src/cast_session_listeners.cpp` | ~408 | Listener 实现 |

### 4.2 通道模块文件

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| `src/channel/src/channel_manager.cpp` | ~400 | 通道管理器 |
| `src/channel/src/softbus/softbus_connection.cpp` | ~350 | SoftBus 连接 |
| `src/channel/src/softbus/softbus_wrapper.cpp` | ~200 | SoftBus 包装 |
| `src/channel/src/tcp/tcp_connection.cpp` | ~400 | TCP 连接 |
| `src/channel/src/tcp/tcp_socket.cpp` | ~250 | Socket 包装 |

### 4.3 RTSP 模块文件

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| `src/rtsp/src/rtsp_controller.cpp` | ~1350 | RTSP 控制器 |
| `src/rtsp/src/rtsp_channel_manager.cpp` | ~300 | RTSP 通道管理 |
| `src/rtsp/src/rtsp_package.cpp` | ~708 | RTSP 消息封装 |
| `src/rtsp/src/rtsp_parse.cpp` | ~200 | RTSP 消息解析 |
| `src/rtsp/src/rtsp_param_info.cpp` | ~150 | 参数信息 |

### 4.4 流媒体模块文件

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| `src/stream/src/cast_stream_manager_client.cpp` | ~400 | 客户端管理器 |
| `src/stream/src/cast_stream_manager_server.cpp` | ~450 | 服务端管理器 |
| `src/stream/src/player/src/cast_stream_player.cpp` | ~600 | 播放器实现 |
| `src/stream/src/player/src/cast_stream_player_manager.cpp` | ~400 | 播放器管理 |
| `src/stream/src/player/src/remote_player_controller.cpp` | ~350 | 远程控制器 |
| `src/stream/src/local/src/cast_local_file_channel_server.cpp` | ~400 | 文件服务 |
| `src/stream/src/local/src/cast_local_file_channel_client.cpp` | ~300 | 文件客户端 |
| `src/stream/src/local/src/local_data_source.cpp` | ~350 | 本地数据源 |

### 4.5 镜像模块文件

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| `src/mirror/src/mirror_player_impl.cpp` | ~500 | 镜像播放器 |
| `src/mirror/src/mirror_player_impl_stub.cpp` | ~200 | Stub 实现 |

### 4.6 工具模块文件

| 文件路径 | 行数 | 说明 |
|----------|------|------|
| `src/utils/src/encrypt_decrypt.cpp` | ~515 | 加密解密 |
| `src/utils/src/permission.cpp` | ~127 | 权限检查 |
| `src/utils/src/state_machine.cpp` | ~200 | 状态机 |
| `src/utils/src/handler.cpp` | ~150 | 消息处理器 |
| `src/utils/src/message.cpp` | ~100 | 消息实现 |
| `src/utils/src/utils.cpp` | ~200 | 通用工具 |
| `src/utils/src/cast_timer.cpp` | ~150 | 定时器 |

---

## 5. 相关文档

- [00_Overview.md](./00_Overview.md) - 项目概览
- [01_Project_Positioning.md](./01_Project_Positioning.md) - 项目定位
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [06_GN_Targets.md](./06_GN_Targets.md) - GN 构建目标
