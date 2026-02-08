# 项目概览

## 项目定位

`multimedia/player_framework` 是 OpenHarmony 媒体子系统的核心组件，为开发者提供**音视频播放和录制**的完整解决方案。

## 核心能力

### 音频能力
- 音频播放（本地/流媒体）
- 音频录制（麦克风/系统音频）
- 音效管理
- 音频触觉反馈（Audio-Haptic）

### 视频能力
- 视频播放（支持多种格式）
- 视频录制（摄像头/屏幕）
- 视频转码

### 媒体辅助
- 元数据提取（AVMetadataExtractor）
- 封面帧生成（AVImageGenerator）
- 屏幕录制与捕获
- 媒体数据源管理

## 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (JS/CJ)                            │
├─────────────────────────────────────────────────────────────┤
│              frameworks/js (N-API)                           │
│   - audio_player    - video_player    - avplayer            │
│   - audio_recorder  - video_recorder  - avrecorder          │
│   - soundpool       - avmetadatahelper - avscreen_capture   │
│   - system_sound_manager  - audio_haptic                    │
├─────────────────────────────────────────────────────────────┤
│              frameworks/cj (FFI)                             │
│   - C/C++ 接口绑定供其他语言使用                              │
├─────────────────────────────────────────────────────────────┤
│              services (C/S 框架)                             │
│   - sa_media       - player        - recorder               │
│   - avcodec        - screen_capture - avmetadatahelper      │
├─────────────────────────────────────────────────────────────┤
│              histreamer 引擎                                 │
│   - 播放引擎      - 录制引擎       - 转码引擎                 │
│   - 元数据引擎    - LPP 低功耗引擎                           │
└─────────────────────────────────────────────────────────────┘
```

## 运行环境

- **操作系统**: OpenHarmony
- **系统服务**: Media Service (SA)
- **权限要求**: 根据功能不同，需要相应的权限

## 关键依赖

- IPC 框架 (IPCSkeleton, IRemoteObject)
- 权限管理 (AccessTokenKit)
- Surface 框架 (用于视频渲染)
- SAMgr (System Ability Manager)

## 代码证据

| 功能 | 源码位置 | 关键文件 |
|------|---------|---------|
| N-API 注册 | `frameworks/js/media/native_module_ohos_media.cpp:87` | `napi_module_register(&g_module)` |
| 权限检查 | `services/utils/media_permission.cpp` | `CheckMicPermission()`, `CheckReadMediaPermission()` |
| SA 服务 | `services/services/sa_media/` | `media_service_stub.cpp`, `media_server.cpp` |
| 播放引擎 | `services/engine/histreamer/player/` | `hiplayer_impl.cpp` |

## 相关模块

- [目录结构与模块职责](02_Directory_Structure.md)
- [N-API 接口总览](05_NAPI_Overview.md)
- [安全风险评审](15_Security_Review.md)
