# 01_Overview - 项目概述

本文档描述 audio_lite 项目的定位、能力、运行环境和关键概念。

## 1.1 项目定位

### 1.1.1 基本信息

| 属性 | 值 |
|------|-----|
| 组件名 | `@ohos/audio_lite` |
| 子系统 | `multimedia` |
| 版本 | 3.1 |
| License | Apache License 2.0 |
| 适配系统 | `mini`, `small`（轻量系统） |
| ROM 占用 | 约 59 KB |

**证据**：`bundle.json:3-20`

```json
{
    "name": "@ohos/audio_lite",
    "description": "Audio encoder and decoder for small system.",
    "version": "3.1",
    "subsystem": "multimedia",
    "adapted_system_type": ["mini", "small"],
    "rom": "59KB"
}
```

### 1.1.2 在子系统中的位置

```
multimedia 子系统
├── audio_lite      ← 本项目（轻量级音频采集）
├── media_lite      ← 标准版媒体框架
├── camera_lite     ← 轻量级相机
└── media_utils_lite ← 媒体工具库
```

**证据**：`README.md:14-16`

### 1.1.3 核心能力

audio_lite 是 OpenHarmony 轻量系统的音频采集组件，提供以下核心能力：

| 能力 | 说明 | 支持模式 |
|------|------|----------|
| 音频采集 | 从麦克风等音频源采集原始 PCM 数据 | 双模式 |
| 参数配置 | 支持采样率、声道、位宽、码率等参数配置 | 双模式 |
| 时间戳 | 提供音频帧的时间戳信息 | 双模式 |
| 音频编码 | 支持 AAC 等格式编码 | 双模式 |
| 双通信模式 | Binder IPC（跨进程）/ Passthrough（本地直连） | 编译开关控制 |

## 1.2 项目边界

### 1.2.1 包含范围

```
audio_lite/
├── frameworks/     ✓ 框架层（AudioCapturer、客户端实现）
├── interfaces/     ✓ 对外 C++ API
├── services/       ✓ 服务端、实现层
└── test/           ✗ 测试代码（不纳入 Wiki）
```

### 1.2.2 不包含范围

| 组件 | 说明 | 所属仓库 |
|------|------|----------|
| Audio HAL | 音频硬件抽象层 | `drivers/peripheral/audio` |
| Codec 库 | 音频编码库 | `device/soc/hisilicon/...` |
| JS/N-API 绑定 | JavaScript 接口 | 本项目无（纯 C++） |
| 标准版 audio | 完整音频框架 | `multimedia_audio` |

### 1.2.3 对外依赖

| 依赖项 | 版本/来源 | 用途 |
|--------|----------|------|
| `media_utils_lite` | 内置子系统 | 公共媒体类型定义 |
| `samgr_lite` | 系统组件 | 系统能力管理、IPC 框架 |
| `surface_lite` | 图形子系统 | 共享内存缓冲区 |
| `pms_client` | 安全子系统 | 权限管理（Binder 模式） |
| `ipc_single` | 通信子系统 | IPC 通信 |
| `bounds_checking_function` | 第三方 | 安全字符串函数 |

**证据**：`frameworks/BUILD.gn:45-48`, `services/BUILD.gn:37-45`

## 1.3 运行环境

### 1.3.1 系统要求

| 要求 | 说明 |
|------|------|
| 操作系统 | OpenHarmony（mini/small 系统） |
| 编译工具 | GN + Ninja |
| C++ 标准 | C++11 或更高 |
| 系统服务 | Samgr_lite 运行态 |

### 1.3.2 硬件要求

| 硬件 | 要求 | 说明 |
|------|------|------|
| 音频编解码器 | Codec 驱动 | 通过 dlopen 动态加载 |
| 音频采集设备 | Audio Capture 驱动 | HAL 层抽象 |
| 内存 | ~59 KB ROM | bundle.json 定义 |

### 1.3.3 构建环境

```bash
# 设置开发板
hb set

# 构建 audio_lite
hb build audio_lite
```

## 1.4 关键概念

### 1.4.1 双通信模式

audio_lite 支持两种客户端-服务端通信模式，由编译开关控制：

| 模式 | 开关 | 适用场景 |
|------|------|----------|
| **Binder IPC 模式** | `enable_media_passthrough_mode = false` | 多进程隔离、安全控制 |
| **Passthrough 模式** | `enable_media_passthrough_mode = true` | 单进程部署、性能优先 |

**证据**：`frameworks/BUILD.gn:20-49`

```gn
if (enable_media_passthrough_mode == true) {
    # Passthrough: 直接链接实现
    sources += [ "passthrough/audio_capturer_client.cpp" ]
    deps += [ "//foundation/multimedia/audio_lite/services:audio_capturer_impl" ]
} else {
    # Binder: IPC 通信
    sources += [ "binder/audio_capturer_client.cpp" ]
    deps += [ "//foundation/systemabilitymgr/samgr_lite/samgr:samgr" ]
}
```

### 1.4.2 状态机

AudioCapturer 实现严格的状态机控制：

```
INITIALIZED ──SetCapturerInfo()──► PREPARED
    │                                │
    │                                ▼
    │◄────────────────────────── RECORDING
    │      Stop()                       │
    │           │                       │
    │           │                       │ Read()
    │           │                       │
    │           ▼                       │
    │◄─────── STOPPED ──────────────────┘
    │
    │ Release()
    ▼
RELEASED
```

**证据**：`interfaces/kits/audio_capturer.h:109-120`

### 1.4.3 Surface 共享内存

Binder IPC 模式下使用 Surface 进行音频数据传输：

| 组件 | 作用 |
|------|------|
| Surface | 共享内存缓冲区管理 |
| SurfaceBuffer | 单个数据帧缓冲区 |
| IpcIo | IPC 序列化（控制命令） |

**证据**：`frameworks/binder/audio_capturer_client.cpp:102-117`

```cpp
int32_t AudioCapturer::AudioCapturerClient::InitSurface()
{
    Surface *surface = Surface::CreateSurface();
    surface->RegisterConsumerListener(*this);  // 注册数据就绪回调
    surface->SetWidthAndHeight(SURFACE_WIDTH, SURFACE_HEIGHT);
    surface->SetQueueSize(SURFACE_QUEUE_SIZE);
    surface->SetSize(SURFACE_SIZE);
    return 0;
}
```

### 1.4.4 服务名称

| 服务名 | 定义位置 | 说明 |
|--------|----------|------|
| `AudioCapServer` | `audio_capturer_server.h` | Samgr 注册的服务名称 |

**证据**：`services/server/include/audio_capturer_server.h`

## 1.5 相关资源

### 1.5.1 代码仓库

| 仓库 | 说明 |
|------|------|
| [audio_lite](../../README.md) | 本项目 |
| [media_lite](https://gitee.com/openharmony/multimedia_media_lite) | 标准版媒体框架 |
| [media_utils_lite](https://gitee.com/openharmony/multimedia_media_utils_lite) | 媒体工具库 |
| [camera_lite](https://gitee.com/openharmony/multimedia_camera_lite) | 轻量级相机 |

### 1.5.2 外部文档

| 文档 | 说明 |
|------|------|
| [Multimedia 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/multimedia.md) | 多媒体子系统概述 |
| OpenHarmony API 文档 | JS/C++ 接口说明 |

---

**上一章**：[README](README.md) | **下一章**：[02_Architecture](02_Architecture.md)
