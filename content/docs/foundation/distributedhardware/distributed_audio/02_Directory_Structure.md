# 目录结构与模块职责

## 顶层目录结构

```
distributed_audio/
├── audiohandler/          # 音频处理器（硬件信息上报）
├── common/                # 公共模块
├── figures/               # 文档图片
├── interfaces/            # 对外接口
├── sa_profile/            # SA 配置文件
├── services/              # 核心服务模块
└── wiki/                  # 本文档
```

## 详细目录树

### 1. audiohandler/ - 音频处理器

**职责**: 硬件信息上报、设备状态变化通知

```
audiohandler/
├── include/
│   └── daudio_handler.h          # 主处理器头文件
└── src/
    └── daudio_handler.cpp        # 硬件信息处理实现

构建目标: distributed_audio_handler
```

**关键类**: `DAudioHandler` - 由分布式硬件框架加载，处理硬件能力上报

---

### 2. common/ - 公共模块

**职责**: 常量定义、工具函数、DFX 支持

```
common/
├── dfx_utils/               # DFX 工具（诊断/反馈/扩展）
│   ├── include/
│   │   ├── daudio_hidumper.h     # HiDumper 集成
│   │   ├── daudio_hisysevent.h   # 系统事件
│   │   ├── daudio_hitrace.h      # 跟踪
│   │   └── daudio_radar.h        # 性能雷达
│   └── src/
├── include/
│   ├── audio_types.h             # 音频类型定义
│   ├── daudio_constants.h        # 常量定义（SA ID 等）
│   ├── daudio_errorcode.h        # 错误码
│   ├── daudio_ipc_interface_code.h  # IPC 命令码
│   ├── daudio_latency_test.h     # 延迟测试
│   ├── daudio_log.h              # 日志工具
│   ├── daudio_ringbuffer.h       # 环形缓冲区
│   └── daudio_util.h             # 通用工具
└── src/
    ├── daudio_latency_test.cpp
    ├── daudio_ringbuffer.cpp
    └── daudio_util.cpp
```

**关键文件**:
- `daudio_constants.h:26` - SA ID 定义（4805/4806）
- `daudio_ipc_interface_code.h` - IPC 命令码枚举
- `daudio_util.cpp` - JSON 参数检查、设备 ID 校验

---

### 3. interfaces/ - 对外接口

**职责**: 提供 Inner Kits（Native C++ SDK）

```
interfaces/inner_kits/native_cpp/
├── audio_source/            # Source 端 SDK
│   ├── include/
│   │   ├── idaudio_source.h           # Source 接口定义
│   │   ├── idaudio_ipc_callback.h     # 回调接口定义
│   │   ├── daudio_source_handler.h    # Source 处理器
│   │   ├── daudio_source_proxy.h      # IPC 代理
│   │   ├── daudio_source_stub.h       # IPC Stub（在服务中）
│   │   ├── daudio_ipc_callback_stub.h # 回调 Stub
│   │   └── daudio_source_load_callback.h  # SA 加载回调
│   └── src/
│       ├── daudio_source_handler.cpp
│       ├── daudio_source_proxy.cpp
│       ├── daudio_ipc_callback.cpp
│       └── daudio_source_load_callback.cpp
│
└── audio_sink/              # Sink 端 SDK
    ├── include/
    │   ├── idaudio_sink.h               # Sink 接口定义
    │   ├── idaudio_sink_ipc_callback.h  # 回调接口定义
    │   ├── daudio_sink_handler.h        # Sink 处理器
    │   ├── daudio_sink_proxy.h          # IPC 代理
    │   └── daudio_sink_ipc_callback_stub.h  # 回调 Stub
    └── src/
        ├── daudio_sink_handler.cpp
        ├── daudio_sink_proxy.cpp
        └── daudio_sink_ipc_callback.cpp

构建目标: distributed_audio_source_sdk, distributed_audio_sink_sdk
```

**关键类**:
- `DAudioSourceHandler` - Source 端单例处理器
- `DAudioSinkHandler` - Sink 端单例处理器
- `DAudioSourceProxy/DAudioSinkProxy` - IPC 代理类

---

### 4. sa_profile/ - SA 配置文件

**职责**: 系统能力服务配置

```
sa_profile/
├── BUILD.gn
├── daudio.cfg              # 进程配置（权限、SELinux）
├── 4805.json               # Source SA 配置
├── 4806.json               # Sink SA 配置
└── common/
    ├── 4805.json           # Source 通用配置
    └── 4806.json           # Sink 通用配置
```

**关键配置**:
- `4805.json` - SA 4805 配置（libdistributed_audio_source.z.so）
- `4806.json` - SA 4806 配置（libdistributed_audio_sink.z.so）
- `daudio.cfg` - 进程权限配置

---

### 5. services/ - 核心服务模块

```
services/
├── audioclient/           # 音频客户端（Sink 端本地音频）
├── audiocontrol/          # 音频控制管理
├── audiohdiproxy/         # HDF 代理
├── audiomanager/          # 音频管理器（核心）
├── audioprocessor/        # 音频处理（编解码）
├── audiotransport/        # 音频传输
└── common/                # 服务公共模块
```

#### 5.1 audioclient/ - 音频客户端

**职责**: Sink 端与本地音频框架交互

```
audioclient/
├── interface/
│   ├── ispk_client.h           # 扬声器客户端接口
│   └── imic_client.h           # 麦克风客户端接口
├── micclient/
│   └── include/
│       └── dmic_client.h       # 麦克风客户端实现
└── spkclient/
    └── include/
        └── dspeaker_client.h   # 扬声器客户端实现
```

**关键类**:
- `DSpeakerClient` - 扬声器客户端（播放远端音频）
- `DMicClient` - 麦克风客户端（采集音频发送到远端）

#### 5.2 audiocontrol/ - 音频控制

**职责**: 控制通道管理（音量、焦点、媒体键）

```
audiocontrol/
├── controlsource/
│   └── include/
│       └── daudio_source_dev_ctrl_mgr.h  # Source 控制管理
└── controlsink/
    └── include/
        └── daudio_sink_dev_ctrl_mgr.h    # Sink 控制管理
```

**关键类**:
- `DAudioSourceDevCtrlMgr` - Source 端控制管理
- `DAudioSinkDevCtrlMgr` - Sink 端控制管理

#### 5.3 audiohdiproxy/ - HDF 代理

**职责**: 与 HDF 驱动层交互

```
audiohdiproxy/
└── include/
    ├── daudio_hdi_handler.h        # HDI 处理器（单例）
    ├── daudio_manager_callback.h   # 管理器回调
    └── idaudio_hdi_callback.h      # HDI 回调接口
```

**关键类**:
- `DAudioHdiHandler` - HDF 交互单例
- `DAudioManagerCallback` - HDF 回调实现

#### 5.4 audiomanager/ - 音频管理器（核心）

**职责**: 核心服务实现（SA 服务）

```
audiomanager/
├── managersource/           # Source 端设备管理
│   └── include/
│       ├── daudio_source_manager.h       # Source 管理器（单例）
│       ├── daudio_source_dev.h           # Source 设备
│       ├── daudio_io_dev.h               # IO 设备基类
│       ├── dspeaker_dev.h                # 扬声器设备
│       ├── dmic_dev.h                    # 麦克风设备
│       └── daudio_echo_cannel_manager.h  # 回声消除管理
├── managersink/             # Sink 端设备管理
│   └── include/
│       ├── daudio_sink_manager.h         # Sink 管理器（单例）
│       └── daudio_sink_dev.h             # Sink 设备
├── servicesource/           # Source 服务（SA 4805）
│   └── include/
│       ├── daudio_source_service.h       # Source 服务
│       ├── daudio_source_stub.h          # IPC Stub
│       └── daudio_ipc_callback_proxy.h   # 回调代理
└── servicesink/             # Sink 服务（SA 4806）
    └── include/
        ├── daudio_sink_service.h         # Sink 服务
        ├── daudio_sink_stub.h            # IPC Stub
        └── daudio_sink_ipc_callback_proxy.h  # 回调代理

构建目标: distributed_audio_source, distributed_audio_sink
```

**关键类**:
- `DAudioSourceService` - Source 系统能力服务（SA 4805）
- `DAudioSinkService` - Sink 系统能力服务（SA 4806）
- `DAudioSourceManager` - Source 管理器单例
- `DAudioSinkManager` - Sink 管理器单例
- `DAudioSourceDev/DAudioSinkDev` - 设备管理

#### 5.5 audioprocessor/ - 音频处理

**职责**: 音频数据处理（编解码等）

```
audioprocessor/
├── interface/
│   ├── iaudio_processor.h           # 处理器接口
│   └── iaudio_processor_callback.h  # 处理器回调
└── directprocessor/
    └── include/
        └── audio_direct_processor.h   # 直通处理器
```

**关键类**:
- `IAudioProcessor` - 处理器接口
- `AudioDirectProcessor` - 直通处理器（无处理）

#### 5.6 audiotransport/ - 音频传输

**职责**: 音频数据传输（通过软总线）

```
audiotransport/
├── interface/
│   ├── iaudio_data_transport.h      # 数据传输接口
│   ├── iaudio_datatrans_callback.h  # 数据回调
│   ├── iaudio_ctrl_transport.h      # 控制传输接口
│   └── iaudio_ctrltrans_callback.h  # 控制回调
├── audioctrltransport/              # 控制通道
│   └── include/
│       ├── daudio_source_ctrl_trans.h
│       └── daudio_sink_ctrl_trans.h
├── receiverengine/                  # 接收引擎（解码）
│   └── include/
│       ├── av_receiver_engine_adapter.h
│       └── av_receiver_engine_transport.h
└── senderengine/                    # 发送引擎（编码）
    └── include/
        ├── av_sender_engine_adapter.h
        └── av_sender_engine_transport.h

构建目标: distributed_audio_decode_transport, distributed_audio_encode_transport
```

**关键类**:
- `AVTransSenderTransport` - 发送传输
- `AVTransReceiverTransport` - 接收传输
- `DaudioSourceCtrlTrans/DaudioSinkCtrlTrans` - 控制传输

#### 5.7 common/ - 服务公共模块

**职责**: 共享数据结构和参数定义

```
services/common/
├── audiodata/
│   └── include/
│       └── audio_data.h        # 音频数据结构
├── audioparam/
│   ├── audio_event.h           # 事件类型定义
│   ├── audio_param.h           # 音频参数
│   └── audio_status.h          # 状态枚举
└── audioeventcallback/
    └── iaudio_event_callback.h # 事件回调接口

构建目标: distributed_audio_utils（包含公共代码）
```

**关键结构**:
- `AudioData` - 音频数据缓冲区
- `AudioEvent` - 事件结构
- `AudioParam` - 音频参数

## 模块依赖关系

```
                        ┌──────────────────┐
                        │   audiomanager   │
                        │  (Source/Sink)   │
                        └────────┬─────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌───────────────┐    ┌──────────────────┐    ┌───────────────┐
│ audiohdiproxy │    │  audiotransport  │    │ audiocontrol  │
│               │    │ (sender/receiver)│    │               │
└───────────────┘    └────────┬─────────┘    └───────────────┘
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
           ▼                  ▼                  ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │audioprocessor│  │audioclient   │  │    common    │
    │              │  │(spk/mic)     │  │              │
    └──────────────┘  └──────────────┘  └──────────────┘
```

## 关键文件索引

| 文件 | 说明 |
|------|------|
| `common/include/daudio_constants.h` | SA ID、错误码等常量 |
| `common/include/daudio_ipc_interface_code.h` | IPC 命令码 |
| `interfaces/inner_kits/*/include/idaudio_*.h` | 对外接口定义 |
| `services/audiomanager/servicesource/include/daudio_source_service.h` | Source 服务 |
| `services/audiomanager/servicesink/include/daudio_sink_service.h` | Sink 服务 |
| `services/audiomanager/managersource/include/daudio_source_manager.h` | Source 管理器 |
| `services/audiomanager/managersink/include/daudio_sink_manager.h` | Sink 管理器 |
| `sa_profile/4805.json` | Source SA 配置 |
| `sa_profile/4806.json` | Sink SA 配置 |

---

*文档生成时间: 2025-02-06*
