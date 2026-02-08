# Inner API 接口文档

## 头文件清单

### 核心接口 (34 个头文件)

**接口路径**: `interfaces/inner_api/native/session/include/`

| 文件 | 说明 | 关键类/接口 |
|------|------|-------------|
| `av_session.h` | 会话主接口 | AVSession (抽象类) |
| `avsession_manager.h` | 会话管理器 | AVSessionManager |
| `avsession_controller.h` | 控制器接口 | AVSessionController |
| `avplayback_state.h` | 播放状态 | AVPlaybackState |
| `avmeta_data.h` | 元数据 | AVMetaData |
| `avcontrol_command.h` | 控制命令 | AVControlCommand |
| `avsession_info.h` | 回调定义 | AVSessionCallback, AVControllerCallback |
| `avsession_descriptor.h` | 会话描述符 | AVSessionDescriptor |
| `avcast_controller.h` | 投屏控制器 | AVCastController |
| `avqueue_item.h` | 播放队列项 | AVQueueItem |
| `avqueue_info.h` | 队列信息 | AVQueueInfo |
| `avcall_state.h` | 通话状态 | AVCallState |
| `avcall_meta_data.h` | 通话元数据 | AVCallMetaData |
| `avcast_control_command.h` | 投屏控制命令 | AVCastControlCommand |
| `avcast_player_state.h` | 投屏播放状态 | AVCastPlayerState |
| `avsession_pixel_map.h` | 像素图 | AVSessionPixelMap |
| `avmedia_description.h` | 媒体描述 | AVMediaDescription |
| `avsession_errors.h` | 错误码 | AVSESSION_SUCCESS 等 |

---

## AVSession 接口

**头文件**: `av_session.h`

### 抽象类定义

```cpp
namespace OHOS::AVSession {
class AVSession {
public:
    enum {
        SESSION_TYPE_INVALID = -1,
        SESSION_TYPE_AUDIO = 0,
        SESSION_TYPE_VIDEO = 1,
        SESSION_TYPE_VOICE_CALL = 2,
        SESSION_TYPE_VIDEO_CALL = 3,
        SESSION_TYPE_PHOTO = 4
    };

    // 获取会话信息
    virtual std::string GetSessionId() = 0;
    virtual std::string GetSessionType() = 0;

    // 元数据操作
    virtual int32_t GetAVMetaData(AVMetaData& meta) = 0;
    virtual int32_t SetAVMetaData(const AVMetaData& meta) = 0;

    // 播放状态
    virtual int32_t GetAVPlaybackState(AVPlaybackState& state) = 0;
    virtual int32_t SetAVPlaybackState(const AVPlaybackState& state) = 0;

    // 队列操作
    virtual int32_t GetAVQueueItems(std::vector<AVQueueItem>& items) = 0;
    virtual int32_t SetAVQueueItems(const std::vector<AVQueueItem>& items) = 0;

    // 控制器
    virtual int32_t GetController(std::shared_ptr<AVSessionController>& controller) = 0;

    // 会话生命周期
    virtual int32_t RegisterCallback(const std::shared_ptr<AVSessionCallback>& callback) = 0;
    virtual int32_t Activate() = 0;
    virtual int32_t Deactivate() = 0;
    virtual int32_t Destroy() = 0;
};
}
```

### 实现类

| 类名 | 位置 | 说明 |
|------|------|------|
| `AVSessionProxy` | `services/session/ipc/proxy/avsession_proxy.cpp` | IPC 客户端代理 |
| `AVSessionItem` | `services/session/server/avsession_item.cpp` | 服务端会话实现 |

---

## AVSessionManager 接口

**头文件**: `avsession_manager.h`

### 单例模式

```cpp
namespace OHOS::AVSession {
class AVSessionManager {
public:
    static AVSessionManager& GetInstance();

    // 会话管理
    virtual std::shared_ptr<AVSession> CreateSession(
        const std::string& tag, int32_t type,
        const AppExecFwk::ElementName& elementName) = 0;

    virtual int32_t CreateSession(const std::string& tag, int32_t type,
        const AppExecFwk::ElementName& elementName,
        std::shared_ptr<AVSession>& session) = 0;

    // 查询
    virtual int32_t GetAllSessionDescriptors(
        std::vector<AVSessionDescriptor>& descriptors) = 0;
    virtual int32_t GetSessionDescriptors(int32_t category,
        std::vector<AVSessionDescriptor>& descriptors) = 0;
    virtual int32_t GetActivatedSessionDescriptors(
        std::vector<AVSessionDescriptor>& activatedSessions) = 0;

    // 控制器
    virtual int32_t CreateController(const std::string& sessionId,
        std::shared_ptr<AVSessionController>& controller) = 0;

    // 监听
    virtual int32_t RegisterSessionListener(
        const std::shared_ptr<SessionListener>& listener) = 0;

    // 系统控制
    virtual int32_t SendSystemAVKeyEvent(const MMI::KeyEvent& keyEvent) = 0;
    virtual int32_t SendSystemControlCommand(const AVControlCommand& command) = 0;

    // 投播
    virtual int32_t CastAudio(const SessionToken& token,
        const AudioStandard::AudioDeviceDescriptor& descriptor) = 0;
    virtual int32_t StartCastDiscovery() = 0;
    virtual int32_t StopCastDiscovery() = 0;
    virtual int32_t StartCast(const SessionToken& sessionToken,
        const OutputDeviceInfo& outputDeviceInfo) = 0;
    virtual int32_t StopCast(const SessionToken& sessionToken) = 0;
};
}
```

### 实现类

| 类名 | 位置 | 说明 |
|------|------|------|
| `AVSessionManagerImpl` | `frameworks/native/session/src/avsession_manager_impl.cpp` | 客户端实现 |

---

## AVSessionController 接口

**头文件**: `avsession_controller.h`

### 抽象类定义

```cpp
namespace OHOS::AVSession {
class AVSessionController {
public:
    // 状态获取
    virtual int32_t GetAVPlaybackState(AVPlaybackState& state) = 0;
    virtual int32_t GetAVMetaData(AVMetaData& data) = 0;
    virtual bool IsSessionActive() = 0;

    // 控制
    virtual int32_t SendAVKeyEvent(const MMI::KeyEvent& keyEvent) = 0;
    virtual int32_t SendControlCommand(const AVControlCommand& command) = 0;
    virtual int32_t SendCommonCommand(const std::string& command,
        const AAFwk::WantParams& args) = 0;

    // 查询
    virtual int32_t GetValidCommands(
        std::vector<int32_t>& commands) = 0;
    virtual int32_t GetLaunchAbility(
        AbilityRuntime::WantAgent::WantAgent& ability) = 0;
    virtual std::string GetSessionId() = 0;

    // 监听
    virtual int32_t RegisterCallback(
        const std::shared_ptr<AVControllerCallback>& callback) = 0;
    virtual int32_t Destroy() = 0;
};
}
```

### 实现类

| 类名 | 位置 | 说明 |
|------|------|------|
| `AVSessionControllerProxy` | `services/session/ipc/proxy/avsession_controller_proxy.cpp` | IPC 客户端代理 |
| `AVControllerItem` | `services/session/server/avcontroller_item.cpp` | 服务端控制器实现 |

---

## 数据结构

### AVPlaybackState

**头文件**: `avplayback_state.h`

```cpp
class AVPlaybackState : public Parcelable {
public:
    // 状态字段
    int32_t state_;           // 播放状态
    double speed_;             // 播放速度
    int64_t position_;        // 播放位置 (ms)
    int64_t bufferedTime_;    // 缓冲时间
    int32_t loopMode_;        // 循环模式
    bool isFavorite_;         // 是否收藏
    int32_t activeItemId_;    // 当前项 ID
    int32_t volume_;          // 音量
    int32_t maxVolume_;       // 最大音量
    bool isMuted_;            // 是否静音
    int64_t duration_;        // 总时长
    int32_t videoWidth_;      // 视频宽度
    int32_t videoHeight_;     // 视频高度
    AAFwk::WantParams extras_; // 扩展信息
};
```

### AVMetaData

**头文件**: `avmeta_data.h`

```cpp
class AVMetaData : public Parcelable {
public:
    std::string assetId_;           // 资源 ID
    std::string title_;             // 标题
    std::string artist_;            // 艺术家
    std::string album_;             // 专辑
    std::string author_;            // 作者
    std::string writer_;            // 词作者
    std::string composer_;          // 曲作者
    int64_t duration_;              // 时长
    std::shared_ptr<AVSessionPixelMap> mediaImage_; // 封面
    std::string lyric_;             // 歌词
    std::string appName_;           // 应用名
    // ... 更多字段
};
```

### AVControlCommand

**头文件**: `avcontrol_command.h`

```cpp
class AVControlCommand : public Parcelable {
public:
    enum Command {
        SESSION_CMD_INVALID = -1,
        SESSION_CMD_PLAY = 0,
        SESSION_CMD_PAUSE = 1,
        SESSION_CMD_STOP = 2,
        SESSION_CMD_PLAY_NEXT = 3,
        SESSION_CMD_PLAY_PREVIOUS = 4,
        SESSION_CMD_FAST_FORWARD = 5,
        SESSION_CMD_REWIND = 6,
        SESSION_CMD_SEEK = 7,
        SESSION_CMD_SET_SPEED = 8,
        SESSION_CMD_SET_LOOP_MODE = 9,
        SESSION_CMD_TOGGLE_FAVORITE = 10,
        // ... 更多命令
    };

    int32_t command_;
    double speed_;
    int64_t forwardTime_;
    int64_t rewindTime_;
    int64_t seekTime_;
    int32_t loopMode_;
    std::string assetId_;
};
```

---

## IPC 接口定义

### IAVSessionService

**文件**: `services/session/ipc/base/iavsession_service.h`

| 方法 | 功能 |
|------|------|
| `GetSession` | 获取会话 |
| `CreateSession` | 创建会话 |
| `GetAllSessionDescriptors` | 获取所有会话描述 |
| `GetSessionDescriptors` | 按类别获取会话 |
| `GetHistoricalSessionDescriptors` | 获取历史会话 |
| `CreateController` | 创建控制器 |
| `SendSystemAVKeyEvent` | 发送系统按键 |
| `SendSystemControlCommand` | 发送系统命令 |
| `CastAudio` | 音频投送 |
| `StartCastDiscovery` | 开始设备发现 |
| `StopCastDiscovery` | 停止设备发现 |
| `StartCast` | 开始投屏 |
| `StopCast` | 停止投屏 |

### IAVSession

**文件**: `services/session/ipc/base/iav_session.h`

| 方法 | 功能 |
|------|------|
| `GetSessionId` | 获取会话 ID |
| `GetSessionType` | 获取会话类型 |
| `GetAVMetaData` / `SetAVMetaData` | 元数据操作 |
| `GetAVPlaybackState` / `SetAVPlaybackState` | 播放状态操作 |
| `GetAVQueueItems` / `SetAVQueueItems` | 队列操作 |
| `RegisterCallback` | 注册回调 |
| `Activate` / `Deactivate` | 激活/停用 |
| `AddSupportCommand` | 添加支持命令 |
| `Destroy` | 销毁会话 |

---

## 回调机制

### AVSessionCallback (服务端回调)

**头文件**: `avsession_info.h`

```cpp
class AVSessionCallback {
public:
    virtual void OnPlay(const AVControlCommand& cmd) = 0;
    virtual void OnPause() = 0;
    virtual void OnStop() = 0;
    virtual void OnPlayNext(const AVControlCommand& cmd) = 0;
    virtual void OnPlayPrevious(const AVControlCommand& cmd) = 0;
    virtual void OnFastForward(int64_t time, const AVControlCommand& cmd) = 0;
    virtual void OnRewind(int64_t time, const AVControlCommand& cmd) = 0;
    virtual void OnSeek(int64_t time) = 0;
    virtual void OnSetSpeed(double speed) = 0;
    virtual void OnSetLoopMode(int32_t loopMode) = 0;
    virtual void OnToggleFavorite(const std::string& mediaId) = 0;
    virtual void OnMediaKeyEvent(const MMI::KeyEvent& keyEvent) = 0;
    virtual void OnOutputDeviceChange(int32_t connectionState,
        const OutputDeviceInfo& outputDeviceInfo) = 0;
    virtual void OnCommonCommand(const std::string& commonCommand,
        const AAFwk::WantParams& args) = 0;
    virtual void OnSkipToQueueItem(int32_t itemId) = 0;
    virtual void OnAVCallAnswer() = 0;
    virtual void OnAVCallHangUp() = 0;
    virtual void OnAVCallToggleCallMute() = 0;
};
```

### AVControllerCallback (客户端回调)

```cpp
class AVControllerCallback {
public:
    virtual void OnAVCallMetaDataChange(const AVCallMetaData& meta) = 0;
    virtual void OnAVCallStateChange(const AVCallState& state) = 0;
    virtual void OnSessionDestroy() = 0;
    virtual void OnPlaybackStateChange(const AVPlaybackState& state) = 0;
    virtual void OnMetaDataChange(const AVMetaData& meta) = 0;
    virtual void OnActiveStateChange(bool isActive) = 0;
    virtual void OnValidCommandChange(
        const std::vector<int32_t>& commands) = 0;
    virtual void OnQueueItemsChange(
        const std::vector<AVQueueItem>& items) = 0;
    virtual void OnExtrasChange(const AAFwk::WantParams& extras) = 0;
};
```

### SessionListener (全局监听)

```cpp
class SessionListener {
public:
    virtual void OnSessionCreate(const AVSessionDescriptor& descriptor) = 0;
    virtual void OnSessionRelease(const AVSessionDescriptor& descriptor) = 0;
    virtual void OnTopSessionChange(const AVSessionDescriptor& descriptor) = 0;
    virtual void OnAudioSessionChecked(const int32_t streamId) = 0;
    virtual void OnDeviceAvailable(const OutputDeviceInfo& info) = 0;
    virtual void OnDeviceOffline(const std::string& deviceId) = 0;
    virtual void OnRemoteDistributedSessionChange(
        const std::string& sessionId, const int32_t type) = 0;
};
```

---

## 错误码定义

**头文件**: `avsession_errors.h`

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | `AVSESSION_SUCCESS` | 成功 |
| 401 | `ERR_PARAM_INVALID` | 参数错误 |
| 6600101 | `ERR_SESSION_NOT_EXIST` | 会话不存在 |
| 6600102 | `ERR_SESSION_IS_ACTIVATED` | 会话已激活 |
| 6600103 | `ERR_SESSION_IS_NOT_ACTIVATED` | 会话未激活 |
| 6600104 | `ERR_SESSION_IS_DESTROYED` | 会话已销毁 |
| 6600105 | `ERR_CONTROLLER_NOT_EXIST` | 控制器不存在 |
| 6600106 | `ERR_CONTROLLER_IS_DESTROYED` | 控制器已销毁 |
| 6600107 | `ERR_SERVICE_NOT_FOUND` | 服务未找到 |
| 6600108 | `ERR_IPC_FAILED` | IPC 失败 |
