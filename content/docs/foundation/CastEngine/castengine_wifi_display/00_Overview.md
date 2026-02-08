# 项目概览

## 项目定位

**castengine_wifi_display** 是 OpenHarmony 投播子系统的核心部件，部件别名为 **Sharing**（媒体分享之意）。

## 核心能力

该部件拥有**流媒体协议接入**、**媒体预览**、**媒体转分发**能力，受投播管理服务管理和调用，是音视频投播子系统重要的流媒体能力部件。

### WFD Source（主投端）

- 主投端发送器，用于投屏 Source 端业务
- 可发送多路屏幕镜像流到不同设备
- 设备作为 Source 端时不能同时作为 Sink 端

### WFD Sink（被投端）

- 被投端接收器，用于投屏 Sink 端业务
- 可接收多个设备的投屏流
- 允许多个设备同时投屏
- 支持多路音频独立控制（播放/静音）

## 运行环境

| 组件 | 要求 |
|------|------|
| 子系统 | castplus |
| 系统类型 | standard（标准设备） |
| 权限级别 | system_basic |
| 用户身份 | audio |

## 项目版本

- **版本号**: 3.2
- **许可证**: Apache License 2.0
- **发布类型**: code-segment

## 依赖关系

### 系统组件依赖

| 组件 | 用途 |
|------|------|
| ipc | 进程间通信 |
| safwk | System Ability Framework |
| media_foundation | 媒体基础框架 |
| av_codec | 音视频编解码 |
| audio_framework | 音频框架 |
| player_framework | 播放框架 |
| camera_framework | 相机框架 |
| wifi | WiFi 功能 |
| hilog | 日志系统 |
| bundle_framework | 包管理框架 |
| napi | Native API |
| samgr | Service and Ability Manager |
| graphic_2d / graphic_surface | 图形子系统 |
| drivers_peripheral_display | 显示驱动 |

### 第三方依赖

| 依赖 | 用途 |
|------|------|
| cJSON | JSON 解析 |
| jsoncpp | JSON 解析 |
| openssl | 加密库 |
| ffmpeg | 媒体处理 |

## 相关文档

- [目录结构与模块职责](01_Directory_Structure.md)
- [逻辑架构](02_Architecture.md)
- [N-API 接口](03_N-API_Interface.md)
- [安全风险评审](08_Security_Review.md)
