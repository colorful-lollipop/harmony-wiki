# 内部 API

## 目的

本文档介绍 media_lite 项目的内部 API，包括模块接口、依赖方向、稳定性和可替换点。

## 适用范围

- 适用于框架开发者
- 适用于系统开发者
- 包含 Player 和 Recorder 模块的 C++ 接口

---

## Player 模块

### 接口定义

**位置**：`interfaces/kits/player_lite/player.h`（行 37-424）

**类名**：`OHOS::Media::Player`

**枚举类型**

#### PlayerSeekMode

| 值 | 说明 |
|-----|------|
| PLAYER_SEEK_PREVIOUS_SYNC | 跳转到前一个同步帧 |
| PLAYER_SEEK_NEXT_SYNC | 跳转到后一个同步帧 |
| PLAYER_SEEK_CLOSEST_SYNC | 跳转到最近的同步帧 |
| PLAYER_SEEK_CLOSEST | 跳转到最近的帧 |

#### PlayerStates

| 值 | 说明 |
|-----|------|
| PLAYER_STATE_ERROR | 0 - 错误 |
| PLAYER_IDLE | 1 - 空闲 |
| PLAYER_INITIALIZED | 2 - 已初始化 |
| PLAYER_PREPARING | 4 - 准备中 |
| PLAYER_PREPARED | 8 - 已准备 |
| PLAYER_STARTED | 16 - 已开始 |
| PLAYER_PAUSED | 32 - 已暂停 |
| PLAYER_STOPPED | 64 - 已停止 |
| PLAYER_PLAYBACK_COMPLETE | 128 - 播放完成 |

### 核心方法

#### 设置类方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| SetSource(const Source&) | int32_t | 设置播放源（URI、FD、流） |
| Prepare() | int32_t | 准备播放环境，必须在 SetSource 后调用 |
| SetVideoSurface(Surface*) | int32_t | 设置视频渲染 Surface |

#### 播放控制方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| Play() | int32_t | 开始或恢复播放 |
| Pause() | int32_t | 暂停播放 |
| Stop() | int32_t | 停止播放 |
| Rewind(int64_t mSeconds, int32_t mode) | int32_t | 跳转到指定位置 |

#### 属性方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| IsPlaying() | bool | 检查是否正在播放 |
| SetVolume(float leftVolume, float rightVolume) | int32_t | 设置音量（0-300） |
| EnableSingleLooping(bool loop) | int32_t | 设置循环播放 |
| IsSingleLooping() | bool | 检查是否循环播放 |

#### 查询方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| GetCurrentTime(int64_t &time) | int32_t | 获取当前播放位置（毫秒） |
| GetDuration(int64_t &duration) | int32_t | 获取总时长（毫秒） |
| GetVideoWidth(int32_t &videoWidth) | int32_t | 获取视频宽度 |
| GetVideoHeight(int32_t &videoHeight) | int32_t | 获取视频高度 |
| GetPlayerState(int32_t &state) | int32_t | 获取播放器状态 |

#### 扩展方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| SetPlaybackSpeed(float speed) | int32_t | 设置播放速度（支持：-128/-64/-32/-16/-8/-4/-2/1/2/4/8/16/32/64/128） |
| GetPlaybackSpeed(float &speed) | int32_t | 获取播放速度 |
| SetAudioStreamType(int32_t type) | int32_t | 设置音频流类型 |
| GetAudioStreamType(int32_t &type) | void | 获取音频流类型 |
| SetParameter(const Format &params) | int32_t | 通过 Format 设置扩展参数（当前版本不支持） |

#### 资源管理方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| Reset() | int32_t | 重置播放器到初始状态 |
| Release() | int32_t | 释放播放器资源 |
| SetPlayerCallback(const shared_ptr<PlayerCallback> &cb) | void | 注册回调 |

### PlayerCallback 回调接口

**位置**：`interfaces/kits/player_lite/player.h`（行 101-160）

| 方法 | 签名 | 说明 |
|------|--------|------|
| OnPlaybackComplete() | void | 播放完成时调用 |
| OnError(int32_t errorType, int32_t errorCode) | void | 发生错误时调用 |
| OnInfo(int type, int extra) | void | 接收播放信息时调用 |
| OnVideoSizeChanged(int width, int height) | void | 视频尺寸变化时调用 |
| OnRewindToComplete() | void | 跳转完成时调用 |

---

## Recorder 模块

### 接口定义

**位置**：`interfaces/kits/recorder_lite/recorder.h`（行 37-632）

**类名**：`OHOS::Media::Recorder`

**枚举类型**

#### VideoSourceType

| 值 | 说明 |
|-----|------|
| VIDEO_SOURCE_SURFACE_YUV | 0 - YUV 视频数据通过 Surface |
| VIDEO_SOURCE_SURFACE_RGB | 1 - RGB 视频数据通过 Surface |
| VIDEO_SOURCE_SURFACE_ES | 2 - 编码视频数据通过 Surface |

#### OutputFormatType

| 值 | 说明 |
|-----|------|
| FORMAT_DEFAULT | 0 - 默认格式 |
| FORMAT_MPEG_4 | 1 - MPEG4 格式 |
| FORMAT_TS | 2 - TS 格式 |

### 核心方法

#### 视频源设置方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| SetVideoSource(VideoSourceType source, int32_t &sourceId) | int32_t | 设置视频源类型和 ID |
| SetVideoEncoder(int32_t sourceId, VideoCodecFormat encoder) | int32_t | 设置视频编码器 |
| SetVideoSize(int32_t sourceId, int32_t width, int32_t height) | int32_t | 设置视频分辨率 |
| SetVideoFrameRate(int32_t sourceId, int32_t frameRate) | int32_t | 设置视频帧率 |
| SetVideoEncodingBitRate(int32_t sourceId, int32_t rate) | int32_t | 设置视频编码比特率 |
| SetCaptureRate(int32_t sourceId, double fps) | int32_t | 设置视频捕获速率 |
| GetSurface(int32_t sourceId) | shared_ptr<Surface> | 获取视频输入 Surface |

#### 音频源设置方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| SetAudioSource(AudioSourceType source, int32_t &sourceId) | int32_t | 设置音频源类型和 ID |
| SetAudioEncoder(int32_t sourceId, AudioCodecFormat encoder) | int32_t | 设置音频编码器 |
| SetAudioSampleRate(int32_t sourceId, int32_t rate) | int32_t | 设置音频采样率 |
| SetAudioChannels(int32_t sourceId, int32_t num) | int32_t | 设置音频通道数 |
| SetAudioEncodingBitRate(int32_t sourceId, int32_t bitRate) | int32_t | 设置音频编码比特率 |

#### 输出控制方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| SetOutputFormat(OutputFormatType format) | int32_t | 设置输出文件格式 |
| SetOutputPath(const string &path) | int32_t | 设置输出文件路径 |
| SetOutputFile(int32_t fd) | int32_t | 设置输出文件描述符 |
| SetMaxDuration(int32_t duration) | int32_t | 设置最大录制时长（秒） |
| SetMaxFileSize(int64_t size) | int32_t | 设置最大文件大小（字节） |
| SetNextOutputFile(int32_t fd) | int32_t | 设置下一个输出文件 FD |

#### 录制控制方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| Prepare() | int32_t | 准备录制环境，必须在设置源后调用 |
| Start() | int32_t | 开始录制 |
| Pause() | int32_t | 暂停录制 |
| Resume() | int32_t | 恢复录制 |
| Stop(bool block) | int32_t | 停止录制（block：true 等待缓存处理完，false 立即停止） |

#### 资源管理方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| Reset() | int32_t | 重置录制器，可重新设置源和参数 |
| Release() | int32_t | 释放录制器资源 |
| SetRecorderCallback(const shared_ptr<RecorderCallback> &callback) | int32_t | 注册录制回调 |

#### 扩展方法

| 方法 | 签名 | 返回值 | 说明 |
|------|--------|--------|------|
| SetDataSource(DataSourceType dataType, int32_t &sourceId) | int32_t | 设置数据源类型 |
| SetFileSplitDuration(FileSplitType type, int64_t timestamp, uint32_t duration) | int32_t | 手动分割视频 |
| SetParameter(int32_t sourceId, const Format &format) | int32_t | 设置扩展参数 |

### RecorderCallback 回调接口

**位置**：`interfaces/kits/recorder_lite/recorder.h`（行 126-209）

#### RecorderInfoType（信息类型）

| 值 | 说明 |
|-----|------|
| RECORDER_INFO_MAX_DURATION_APPROACHING | 0 - 达到最大时长阈值 |
| RECORDER_INFO_MAX_FILESIZE_APPROACHING | 1 - 达到最大文件大小阈值 |
| RECORDER_INFO_MAX_DURATION_REACHED | 2 - 达到最大时长 |
| RECORDER_INFO_MAX_FILESIZE_REACHED | 3 - 达到最大文件大小 |
| RECORDER_INFO_NEXT_OUTPUT_FILE_STARTED | 4 - 下一个输出文件开始 |
| RECORDER_INFO_FILE_SPLIT_FINISHED | 5 - 文件分割完成 |
| RECORDER_INFO_FILE_START_TIME_MS | 6 - 文件开始时间 |
| RECORDER_INFO_NEXT_FILE_FD_NOT_SET | 7 - 需要下一个文件 FD |
| RECORDER_INFO_NO_FRAME_DATA | 8 - 无帧数据 |

#### RecorderErrorType（错误类型）

| 值 | 说明 |
|-----|------|
| RECORDER_ERROR_CREATE_FILE_FAIL | 0 - 创建文件失败 |
| RECORDER_ERROR_WRITE_FILE_FAIL | 1 - 写入文件失败 |
| RECORDER_ERROR_CLOSE_FILE_FAIL | 2 - 关闭文件失败 |
| RECORDER_ERROR_READ_DATA_ERROR | 3 - 读取数据失败或超时 |
| RECORDER_ERROR_INTERNAL_OPERATION_FAIL | 4 - 内部操作失败 |
| RECORDER_ERROR_UNKNOWN | 5 - 未知错误 |

#### 回调方法

| 方法 | 签名 | 说明 |
|------|--------|------|
| OnError(int32_t errorType, int32_t errorCode) | void | 发生错误时调用 |
| OnInfo(int32_t type, int32_t extra) | void | 接收录制信息时调用 |

---

## 依赖方向

### Player 模块依赖

```
Player (interfaces/kits)
    ↓
    (Binder 模式)
        ↓
    samgr_lite / ipc_lite
        ↓
    player_impl (services)
    ↓
    hardware_media_sdk / histreamer

    (Passthrough 模式)
        ↓
    player_impl (services) [直接链接]
```

**依赖组件**：
- `permission_lite` - 权限检查
- `surface_lite` - 视频 Surface
- `media_utils_lite` - 公共工具和类型定义
- `samgr_lite` - 服务管理（Binder 模式）
- `ipc_lite` - 进程间通信（Binder 模式）

### Recorder 模块依赖

```
Recorder (interfaces/kits)
    ↓
    (Binder 模式)
        ↓
        samgr_lite / ipc_lite
        ↓
    recorder_impl (services)
        ↓
    hardware_media_sdk / audio_capturer_impl

    (Passthrough 模式)
        ↓
    recorder_impl (services) [直接链接]
```

**依赖组件**：
- `permission_lite` - 权限检查
- `surface_lite` - 视频 Surface
- `media_utils_lite` - 公共工具和类型定义
- `samgr_lite` - 服务管理（Binder 模式）
- `ipc_lite` - 进程间通信（Binder 模式）
- `audio_lite` - 音频捕获（Passthrough 模式）
- `camera_lite` - 相机输入（Passthrough 模式）

---

## 模块接口

### 可替换点

| 模块 | 可替换点 | 说明 |
|------|----------|------|
| Player | player_impl | 播放器核心实现，可替换为其他播放引擎 |
| Recorder | recorder_impl | 录音器核心实现，可替换为其他录制引擎 |

### 稳定接口

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| Player C++ 接口 | 稳定 | 定义在 `interfaces/kits/player_lite/player.h` |
| Recorder C++ 接口 | 稳定 | 定义在 `interfaces/kits/recorder_lite/recorder.h` |
| PlayerCallback | 稳定 | 虚基类，用于回调 |
| RecorderCallback | 稳定 | 虚基类，用于回调 |

### 内部实现接口（不稳定）

| 模块 | 说明 |
|------|------|
| PlayerClient | IPC 客户端实现，可能随 IPC 机制变化 |
| PlayerServer | 服务端实现，内部细节可能变化 |
| PlayerImpl | 内部实现类，依赖外部播放引擎 |
| RecorderClient | IPC 客户端实现，可能随 IPC 机制变化 |
| RecorderServer | 服务端实现，内部细节可能变化 |
| RecorderImpl | 内部实现类，依赖外部硬件 SDK |

---

## 相关跳转

- [项目概览](00_Overview.md)
- [目录结构与模块职责](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [对外 JSI 接口](03_JSI_Interfaces.md)
- [GN Targets](05_GN_Targets.md)
