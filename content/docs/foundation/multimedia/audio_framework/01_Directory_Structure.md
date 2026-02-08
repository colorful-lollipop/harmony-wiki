# Audio Framework - 目录结构与模块职责

## 整体架构

```
audio_framework/
├── interfaces/              # 📜 API 接口层 (纯头文件)
├── frameworks/              # 🔧 框架层 (客户端实现)
├── services/               # ⚙️ 服务层 (系统服务)
├── sa_profile/             # 🔔 SA 配置
└── plugins/                # 🔨 构建插件
```

## 1. interfaces/ - API 接口定义

**职责**: 定义所有公共和内部 API（纯头文件，无实现）

```
interfaces/
└── inner_api/native/           # 内部 Native API (C++)
    ├── audiocommon/include/    # 公共类型定义 (AudioInfo, errors, descriptors)
    ├── audiocapturer/include/  # AudioCapturer 接口
    ├── audiorenderer/include/  # AudioRenderer/TonePlayer 接口
    ├── audiomanager/include/   # 所有管理器接口
    │   ├── audio_system_manager.h
    │   ├── audio_routing_manager.h
    │   ├── audio_volume_manager.h
    │   ├── audio_session_manager.h
    │   ├── audio_effect_manager.h
    │   ├── audio_interrupt_manager.h
    │   └── audio_stream_manager.h
    ├── audiosasdk/include/      # SA SDK 接口
    ├── audiosuite/include/     # Audio Suite 接口
    ├── audioloopback/include/  # 回环接口
    └── toneplayer/include/     # 音调播放器接口
```

**证据来源**: `bundle.json:137-209`

## 2. frameworks/ - 客户端框架实现

**职责**: 提供客户端 API 实现，支持多种语言绑定

```
frameworks/
├── js/napi/                      # JavaScript/TypeScript N-API
│   ├── common/                   # 公共工具 (async, enums, errors, DFX)
│   ├── audiomanager/             # AudioManager + 25+ callback 类型
│   ├── audiorenderer/            # AudioRenderer + callbacks
│   ├── audiocapturer/            # AudioCapturer + callbacks
│   ├── audioloopback/            # 回环功能
│   ├── toneplayer/               # 音调播放
│   └── asrcontroller/            # ASR 控制器
│
├── native/                       # Native C/C++ 实现
│   ├── ohaudio/                 # OH Audio C API (NDK)
│   ├── ohaudiosuite/            # Audio Suite C++ API
│   ├── audiorenderer/           # 渲染器实现
│   ├── audiocapturer/           # 采集器实现
│   ├── audioeffect/             # 音效链管理
│   ├── audiopolicy/            # 策略客户端
│   ├── hdiadapter_new/          # HDI 硬件抽象层
│   │   ├── adapter/            # 设备管理器
│   │   ├── sink/               # 渲染 sinks (bluetooth, fast, offload)
│   │   └── source/             # 采集 sources
│   ├── pulseaudio/             # PulseAudio 集成
│   ├── opensles/               # OpenSL ES 兼容
│   └── bluetoothclient/        # 蓝牙管理
│
├── cj/                          # Cangjie 语言绑定
└── taihe/                       # Taihe 语言绑定
```

## 3. services/ - 系统服务实现

**职责**: 运行在系统进程中的后台服务，处理核心逻辑

### 3.1 audio_policy/ - 音频策略服务

**职责**: 策略决策中心（路由、音量、焦点、设备选择）

```
services/audio_policy/
├── client/                      # 策略客户端
├── server/
│   ├── domain/                  # 业务域
│   │   ├── device/             # 设备管理 (A2DP, PnP, VA)
│   │   ├── effect/            # 音效/空间化
│   │   ├── interrupt/          # 焦点管理
│   │   ├── pipe/              # 管道配置
│   │   ├── router/            # 路由策略
│   │   ├── session/           # 会话管理
│   │   ├── stream/            # 流集合
│   │   ├── tone/              # 音调管理
│   │   ├── volume/             # 音量管理
│   │   └── zone/              # 多区域音频
│   └── infra/                  # 基础设施
│       ├── config/            # XML 配置解析
│       ├── datashare/          # 设置存储
│       └── window/             # 窗口集成
└── idl/                        # IPC 接口定义
```

### 3.2 audio_service/ - 音频核心服务

**职责**: 低层音频流管理、HAL 交互

```
services/audio_service/
├── client/                      # 服务客户端
├── server/                      # 服务实现
│   ├── include/                 # 服务器头文件
│   │   ├── AudioServer
│   │   ├── RendererInServer
│   │   ├── CapturerInServer
│   │   └── PlaybackEngine
│   └── src/
│       ├── audio_server.cpp     # 主入口
│       ├── renderer/capturer
│       └── PA adapter
└── idl/                        # IPC 接口定义
```

### 3.3 audio_engine/ - 高性能音频引擎 (HPAE)

**职责**: 下一代高性能音频引擎，节点图处理架构

```
services/audio_engine/
├── manager/                     # 各管理器
├── node/                        # 处理节点 (sink/input/output)
├── plugin/                      # 处理插件 (resample, channel_converter)
└── monitor/                     # 运行时监控
```

### 3.4 audio_suite/ - 音频处理套件

**职责**: 高层音频处理（EQ、噪声消除、空间音频）

```
services/audio_suite/
├── client/
│   ├── manager/                 # 套件引擎
│   ├── node/                    # 处理节点
│   └── utils/                   # 算法接口
└── config/                      # 能力配置
```

## 4. sa_profile/ - System Ability 配置

**职责**: SA 注册配置

```
sa_profile/
├── audio_policy.json            # AudioPolicyServer SA 配置
└── pulseaudio.json              # AudioServer SA 配置
```

**SA IDs**:
- `3001`: AUDIO_DISTRIBUTED_SERVICE_ID → AudioServer
- `3009`: AUDIO_POLICY_SERVICE_ID → AudioPolicyServer

## 模块边界总结

| 层次 | 运行位置 | 职责 |
|------|----------|------|
| interfaces/ | N/A | API 契约定义 |
| frameworks/native | 客户端进程 | Native C/C++ API |
| frameworks/js/napi | 客户端进程 | JS 绑定 |
| services/audio_policy | system_service | 策略决策 |
| services/audio_service | audio_service | 流管理 |
| services/audio_engine | audio_service | 高性能引擎 |
| services/audio_suite | 客户端进程 | 音频处理套件 |

## 相关文档

- [架构设计](02_Architecture.md)
- [N-API 接口](03_NAPI.md)
- [构建系统](04_Build.md)
