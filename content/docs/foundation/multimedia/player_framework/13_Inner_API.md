# Inner API 参考

## 服务端内部接口

### IPlayerService

```cpp
// 文件: services/services/player/ipc/player_service_stub.cpp
class IPlayerService {
public:
    virtual ~IPlayerService() = default;
    virtual int32_t SetSource(const std::string& url) = 0;
    virtual int32_t Play() = 0;
    virtual int32_t Pause() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Reset() = 0;
    virtual int32_t SetVolume(float leftVolume, float rightVolume) = 0;
    virtual int64_t GetCurrentTime() = 0;
    virtual int64_t GetDuration() = 0;
    virtual int32_t Seek(int32_t msec) = 0;
};
```

### IRecorderService

```cpp
// 文件: services/services/recorder/ipc/recorder_service_stub.cpp
class IRecorderService {
public:
    virtual ~IRecorderService() = default;
    virtual int32_t SetSource(const std::string& url) = 0;
    virtual int32_t SetVideoEncoder(int32_t quality, int32_t encoder) = 0;
    virtual int32_t SetVideoSize(int32_t width, int32_t height) = 0;
    virtual int32_t SetVideoFrameRate(int32_t frameRate) = 0;
    virtual int32_t SetAudioEncoder(int32_t encoder) = 0;
    virtual int32_t SetAudioSampleRate(int32_t sampleRate) = 0;
    virtual int32_t Start() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Pause() = 0;
    virtual int32_t Resume() = 0;
    virtual int32_t Release() = 0;
};
```

## 引擎接口

### IHiStreamer

```cpp
// 文件: services/engine/histreamer/player/hiplayer_impl.cpp
class HiPlayerImpl {
public:
    virtual int32_t Init() = 0;
    virtual int32_t Prepare() = 0;
    virtual int32_t Play() = 0;
    virtual int32_t Pause() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Seek(int64_t ms) = 0;
    virtual int32_t SetVolume(float volume) = 0;
    virtual int64_t GetCurrentPosition() = 0;
    virtual int64_t GetDuration() = 0;
    virtual void SetCallback(const std::shared_ptr<PlayerCallback>& callback) = 0;
};
```

## 客户端内部接口

### IMediaService

```cpp
// 文件: services/services/sa_media/ipc/media_service_proxy.cpp
class IMediaService {
public:
    virtual sptr<IRemoteObject> GetSubSystemAbility(
        MediaSystemAbility subSystemId,
        const sptr<IRemoteObject>& listener) = 0;
    virtual sptr<IRemoteObject> GetSubSystemAbilityWithTimeOut(
        MediaSystemAbility subSystemId,
        const sptr<IRemoteObject>& listener,
        uint32_t timeoutMs) = 0;
};
```

## 稳定性标注

### 稳定接口 (Stable)

| 接口 | 位置 | 描述 |
|------|-----|------|
| IPlayerService | services/services/player/ | 播放服务核心接口 |
| IRecorderService | services/services/recorder/ | 录制服务核心接口 |
| IMediaService | services/services/sa_media/ | SA 管理接口 |

### 不稳定接口 (Unstable)

| 接口 | 位置 | 描述 |
|------|-----|------|
| 引擎内部接口 | services/engine/histreamer/ | 引擎实现细节，可能变更 |
| 回调接口 | services/xxx/callback/ | 内部回调协议 |

## 相关文档

- [架构说明](03_Architecture.md)
- [引擎实现](14_Engine_Implementation.md)
