# Audio Lite

OpenHarmony 轻量级音频采集组件。

## Overview

| 属性 | 值 |
|------|-----|
| 组件名 | @ohos/audio_lite |
| 子系统 | multimedia |
| 适配系统 | mini, small |
| ROM | 59KB |
| License | Apache-2.0 |

## Directory Structure

```
foundation/multimedia/audio_lite
├── frameworks/          # Framework 层（对外 API）
│   ├── audio_capturer.cpp           # AudioCapturer 实现
│   ├── binder/                       # Binder IPC 客户端
│   │   └── audio_capturer_client.cpp
│   └── passthrough/                  # Passthrough 模式客户端
│       └── audio_capturer_client.cpp
├── interfaces/         # 对外接口
│   └── kits/
│       └── audio_capturer.h          # C++ API 头文件
├── services/           # 服务层
│   ├── impl/                        # 实现层
│   │   ├── audio_capturer_impl.cpp
│   │   ├── audio_encoder/
│   │   │   └── audio_encoder.cpp
│   │   └── audio_source/
│   │       └── audio_source.cpp
│   └── server/                      # SA Server
│       ├── audio_capturer_server.cpp
│       └── audio_capturer_samgr.cpp
└── test/               # 测试（不纳入文档）
```

## Key Capabilities

- **Audio Capture**: 音频采集功能
- **Dual Mode**: 支持 Passthrough 和 Binder IPC 两种通信模式
- **Encoding**: 支持音频编码（通过 AudioEncoder）

## Build

```bash
hb set   # 选择开发板
hb build audio_lite
```

## Important Notes

- **No N-API**: 本项目不包含 Node.js N-API 绑定，是纯 C++ framework
- **Samgr Service**: 通过系统能力管理器 (Samgr) 提供服务
- **IPC Communication**: 使用 OpenHarmony IPC 框架进行进程间通信
