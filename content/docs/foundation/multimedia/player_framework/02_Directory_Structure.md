# 目录结构与模块职责

## 顶层目录

```
player_framework/
├── interfaces/          # 对外接口层
│   ├── kits/           # 应用接口 (JS/C API)
│   └── inner_api/      # 系统内部件接口
├── frameworks/         # 客户端进程实现
│   ├── js/             # N-API 实现
│   ├── native/         # Native C++ 实现
│   ├── taihe/          # Taihe 引擎封装
│   └── cj/             # C/C++ FFI 绑定
├── services/           # 服务端实现
│   ├── services/       # C/S 框架服务
│   ├── engine/         # 引擎实现
│   ├── utils/          # 基础工具
│   ├── seccomp_policy/ # 安全策略
│   └── dfx/            # 诊断与调试
├── BUILD.gn            # 根构建入口
├── bundle.json         # 部件描述文件
└── config.gni          # 构建配置
```

## interfaces/ - 对外接口层

### kits/ - 应用接口

| 目录 | 职责 | 主要文件 |
|------|-----|---------|
| `js/` | JS N-API 头文件 | `native_module_ohos_media.h`, `audio_player_napi.h` |
| `c/` | C API 头文件 | 底层 C 接口定义 |

### inner_api/ - 内部 API

| 目录 | 职责 |
|------|-----|
| `native/` | Native 层内部接口 |

## frameworks/ - 客户端框架

### js/ - N-API 实现

| 模块目录 | 导出类 | 命名空间 |
|---------|-------|---------|
| `player/` | AudioPlayerNapi, VideoPlayerNapi | ohos.media |
| `avplayer/` | AVPlayerNapi | ohos.multimedia.avplayer |
| `recorder/` | AudioRecorderNapi, VideoRecorderNapi | ohos.media |
| `avrecorder/` | AVRecorderNapi | ohos.multimedia.avrecorder |
| `soundpool/` | SoundPoolNapi | ohos.multimedia.soundpool |
| `metadatahelper/` | AVMetadataExtractorNapi, AVImageGeneratorNapi | ohos.multimedia |
| `avscreen_capture/` | AVScreenCaptureNapi | ohos.multimedia.avscreen_capture |
| `screencapturemonitor/` | ScreenCaptureMonitorNapi | ohos.multimedia |
| `audio_haptic/` | AudioHapticManagerNapi | ohos.multimedia.audiohaptic |
| `system_sound_manager/` | SystemSoundManagerNapi | ohos.systemSoundManager |
| `mediasource/` | MediaSourceNapi | ohos.multimedia.mediasource |
| `media/` | 主入口模块 | multimedia.media |
| `common/` | 公共工具 | - |

### cj/ - FFI 绑定

支持 C/C++ 开发者使用框架功能：

- `avplayer/`
- `avrecorder/`
- `avscreen_capture/`
- `avtranscoder/`
- `audio_haptic/`
- `metadatahelper/`
- `soundpool/`

## services/ - 服务端

### services/ - C/S 框架

| 模块 | 职责 | 关键文件 |
|------|-----|---------|
| `sa_media/` | 媒体主进程管理 | `media_service_stub.cpp`, `media_server.cpp` |
| `player/` | 播放服务 | `player_service_stub.cpp`, `player_server.cpp` |
| `recorder/` | 录制服务 | `recorder_service_stub.cpp`, `recorder_server.cpp` |
| `screen_capture/` | 屏幕捕获 | `screen_capture_service_stub.cpp` |
| `avmetadatahelper/` | 元数据辅助 | `avmetadatahelper_service_stub.cpp` |
| `monitor/` | 监控服务 | `monitor_service_stub.cpp` |
| `factory/` | 工厂模式 | `engine_factory.cpp` |
| `common/` | 公共组件 | - |

### engine/ - 引擎实现

| 模块 | 职责 |
|------|-----|
| `histreamer/` | HiStreamer 播放引擎 |
| `histreamer/player/` | 播放引擎实现 |
| `histreamer/recorder/` | 录制引擎实现 |
| `histreamer/transcoder/` | 转码引擎实现 |
| `histreamer/avmetadatahelper/` | 元数据引擎 |
| `histreamer/lpp/` | 低功耗播放引擎 (LPP) |
| `histreamer/lpp/lpp_audio_streamer/` | LPP 音频流 |
| `histreamer/lpp/lpp_video_streamer/` | LPP 视频流 |
| `common/` | 公共引擎组件 |

### utils/ - 工具类

| 文件 | 职责 |
|------|-----|
| `media_permission.cpp` | 权限检查 |
| `media_dfx.cpp` | 诊断与调试 |
| `media_utils.cpp` | 通用工具 |

## 代码证据

- N-API 入口: `frameworks/js/media/native_module_ohos_media.cpp:71-88`
- SA 服务入口: `services/services/sa_media/ipc/media_service_stub.cpp:131`
- 权限检查: `services/utils/media_permission.cpp:31-68`
