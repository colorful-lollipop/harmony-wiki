# 引擎实现

## HiStreamer 引擎

### 概述

HiStreamer 是 player_framework 的核心播放引擎，基于 GStreamer 架构实现。

### 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     HiStreamer 引擎                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Source     │  │  Demuxer     │  │  Decoder    │              │
│  │  (数据源)    │──▶│  (解复用)   │──▶│  (解码器)   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                              │                  │
│                                              ▼                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Sink       │  │  Renderer    │  │  Encoder    │              │
│  │  (输出)     │◀──│  (渲染器)   │◀──│  (编码器)   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

### 核心文件

| 文件 | 职责 |
|------|-----|
| `services/engine/histreamer/player/hiplayer_impl.cpp` | 播放引擎主实现 |
| `services/engine/histreamer/recorder/recorder_impl.cpp` | 录制引擎主实现 |
| `services/engine/histreamer/factory/engine_factory.cpp` | 引擎工厂 |

## LPP 低功耗引擎

### 概述

LPP (Low Power Player) 是针对低功耗场景优化的音频播放引擎。

### 特点

- 降低 CPU 占用
- 减少功耗
- 支持音频流播放

### 模块

| 模块 | 职责 |
|------|-----|
| `lpp_audio_streamer/` | 音频流处理 |
| `lpp_video_streamer/` | 视频流处理 |
| `lpp_engine_manager/` | 引擎管理 |

## 引擎工厂

```cpp
// 文件: services/services/factory/engine_factory_repo.cpp
class EngineFactoryRepo {
public:
    static std::shared_ptr<PlayerEngine> CreatePlayerEngine();
    static std::shared_ptr<RecorderEngine> CreateRecorderEngine();
    static std::shared_ptr<MetadataEngine> CreateMetadataEngine();
};
```

## 引擎生命周期

```
Create() → Init() → Prepare() → Start() → Stop() → Release()
```

### 状态转换

```
┌─────────┐    Init     ┌─────────┐    Prepare   ┌─────────┐
│ CREATED │─────────────▶│ INITING │─────────────▶│ PREPARED│
└─────────┘              └─────────┘              └─────────┘
       │                       │                       │
       │ Release               │ Start                │ Stop
       ▼                       ▼                       ▼
┌─────────┐              ┌─────────┐              ┌─────────┐
│  RELEASED│◀─────────────│  RUNNING│◀─────────────│  PAUSED │
└─────────┘              └─────────┘              └─────────┘
```

## 相关文档

- [系统架构](03_Architecture.md)
- [Inner API 参考](13_Inner_API.md)
