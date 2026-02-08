# 内部 API

## 概述

内部 API 是模块间的接口定义，包括：
- 类继承关系
- 纯虚接口
- 回调接口
- 数据结构

## 核心接口定义

### 1. 数据传输接口

```cpp
// services/audiotransport/interface/iaudio_data_transport.h
class IAudioDataTransport {
public:
    virtual int32_t SetUp(const AudioParam &localParam, 
                          const AudioParam &remoteParam,
                          const std::shared_ptr<IAudioDataTransCallback> &callback) = 0;
    virtual int32_t Start() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Release() = 0;
    virtual int32_t FeedAudioData(const std::shared_ptr<AudioData> &audioData) = 0;
    virtual int32_t InitEngine(const std::string &ip, int32_t port) = 0;
    virtual int32_t SendMessage(const AudioEvent &event) = 0;
};
```

### 2. 数据传输回调

```cpp
// services/audiotransport/interface/iaudio_datatrans_callback.h
class IAudioDataTransCallback {
public:
    virtual void OnEngineTransDataAvailable(const std::shared_ptr<AudioData> &audioData) = 0;
    virtual void OnEngineTransEvent(const AudioEvent &event) = 0;
    virtual void OnEngineTransMessage(const AudioEvent &event) = 0;
};
```

### 3. 控制传输接口

```cpp
// services/audiotransport/interface/iaudio_ctrl_transport.h
class IAudioCtrlTransport {
public:
    virtual int32_t SetUp(const std::shared_ptr<IAudioCtrlTransCallback> &callback) = 0;
    virtual int32_t Start() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Release() = 0;
    virtual bool IsOpened() = 0;
    virtual int32_t SendAudioEvent(const AudioEvent &event) = 0;
};
```

### 4. 处理器接口

```cpp
// services/audioprocessor/interface/iaudio_processor.h
class IAudioProcessor {
public:
    virtual int32_t ConfigureAudioProcessor(const AudioParam &localParam,
                                             const AudioParam &remoteParam) = 0;
    virtual int32_t StartAudioProcessor() = 0;
    virtual int32_t StopAudioProcessor() = 0;
    virtual int32_t FeedAudioProcessor(const std::shared_ptr<AudioData> &audioData) = 0;
    virtual int32_t ReleaseAudioProcessor() = 0;
};
```

### 5. 事件回调接口

```cpp
// services/common/audioeventcallback/iaudio_event_callback.h
class IAudioEventCallback {
public:
    virtual void NotifyEvent(const AudioEvent &event) = 0;
};
```

### 6. HDI 回调接口

```cpp
// services/audiohdiproxy/include/idaudio_hdi_callback.h
class IDAudioHdiCallback {
public:
    virtual int32_t OnReadData(const std::string &devId, int32_t dhId,
                               const std::vector<char> &data) = 0;
    virtual int32_t OnWriteData(const std::string &devId, int32_t dhId,
                                std::vector<char> &data) = 0;
    virtual int32_t OnNotifyEvent(const std::string &devId, int32_t dhId,
                                  const AudioEvent &event) = 0;
};
```

## 核心数据结构

### AudioData

```cpp
// services/common/audiodata/include/audio_data.h
class AudioData {
public:
    size_t Capacity();      // 缓冲区容量
    size_t Size();          // 有效数据大小
    uint8_t *Data();        // 数据指针
    int64_t GetPts();       // 时间戳
    void SetPts(int64_t pts);
    
    // 元数据
    int32_t GetInt32Value(const std::string &key);
    void SetInt32Value(const std::string &key, int32_t value);
    
private:
    std::vector<uint8_t> data_;
    int64_t pts_ = 0;
    std::map<std::string, int32_t> int32Map_;
    std::map<std::string, int64_t> int64Map_;
    std::map<std::string, std::string> stringMap_;
};
```

### AudioParam

```cpp
// services/common/audioparam/audio_param.h
struct AudioCommonParam {
    int32_t sampleRate = 0;      // 采样率
    int32_t channelMask = 0;     // 声道配置
    int32_t bitFormat = 0;       // 位格式
    std::string codecType;       // 编解码类型
    int32_t frameSize = 0;       // 帧大小
};

struct AudioCaptureOptions {
    int32_t sourceType = 0;
    int32_t capturerFlags = 0;
};

struct AudioRenderOptions {
    int32_t contentType = 0;
    int32_t streamUsage = 0;
    int32_t renderFlags = 0;
};

struct AudioParam {
    AudioCommonParam comParam;
    AudioCaptureOptions captureOpts;
    AudioRenderOptions renderOpts;
};
```

### AudioEvent

```cpp
// services/common/audioparam/audio_event.h
struct AudioEvent {
    uint32_t type;           // 事件类型
    std::string content;     // 事件内容（JSON）
};

// 事件类型枚举
enum AudioEventType : uint32_t {
    // 扬声器事件
    OPEN_SPEAKER = 11,
    CLOSE_SPEAKER = 12,
    SPEAKER_OPENED = 13,
    SPEAKER_CLOSED = 14,
    
    // 麦克风事件
    OPEN_MIC = 21,
    CLOSE_MIC = 22,
    MIC_OPENED = 23,
    MIC_CLOSED = 24,
    
    // 控制事件
    VOLUME_SET = 31,
    VOLUME_CHANGE = 33,
    AUDIO_FOCUS_CHANGE = 41,
    AUDIO_RENDER_STATE_CHANGE = 42,
    
    // 参数事件
    SET_PARAM = 51,
    SEND_PARAM = 52,
    
    // MMAP 事件
    MMAP_SPK_START = 81,
    MMAP_SPK_STOP = 82,
    MMAP_MIC_START = 83,
    MMAP_MIC_STOP = 84,
};
```

## 核心类层次

### Source 端设备层次

```
DAudioIODev (基类)
    │
    ├── DSpeakerDev (扬声器设备)
    │       ├── OnWriteData()     // HDF 写入回调
    │       ├── OnEnqueueData()   // 数据入队
    │       └── EnqueueThread()   // 入队线程
    │
    └── DMicDev (麦克风设备)
            ├── OnReadData()      // HDF 读取回调
            ├── OnMicDataReceived() // 数据接收
            └── EnqueueThread()   // 入队线程
```

### Sink 端客户端层次

```
ISpkClient / IMicClient (接口)
    │
    ├── DSpeakerClient (扬声器客户端)
    │       ├── SetUp()           // 设置
    │       ├── StartRender()     // 开始渲染
    │       ├── OnEngineTransDataAvailable() // 接收数据
    │       └── PlayThread()      // 播放线程
    │
    └── DMicClient (麦克风客户端)
            ├── SetUp()           // 设置
            ├── StartCapture()    // 开始采集
            ├── OnReadData()      // 读取回调
            └── CaptureThread()   // 采集线程
```

## 模块间调用关系

### 数据流调用链

```
播放流程:
HDF -> DSpeakerDev::OnWriteData() -> EnqueueThread() -> AVTransSenderTransport::FeedAudioData() -> 软总线

录音流程:
软总线 -> AVTransReceiverTransport::OnEngineDataAvailable() -> DMicDev::OnMicDataReceived() -> HDF
```

### 控制流调用链

```
事件通知:
DAudioSourceDev::HandleOpenDSpeaker() -> DaudioSourceCtrlTrans::SendAudioEvent() -> 软总线 -> DaudioSinkCtrlTrans::OnChannelEvent() -> DAudioSinkDev::HandleOpenDSpeaker()
```

## 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `IAudioDataTransport` | 稳定 | 传输层核心接口 |
| `IAudioCtrlTransport` | 稳定 | 控制传输核心接口 |
| `IAudioProcessor` | 实验性 | 处理器接口，可能扩展 |
| `IAudioEventCallback` | 稳定 | 事件回调接口 |
| `IDAudioHdiCallback` | 稳定 | HDI 回调接口 |

## 可替换点

### 1. 传输层替换

可通过实现 `IAudioDataTransport` 接口替换传输实现：

```cpp
class CustomTransport : public IAudioDataTransport {
    // 自定义传输实现
};
```

### 2. 处理器替换

可通过实现 `IAudioProcessor` 接口添加音频处理：

```cpp
class EffectProcessor : public IAudioProcessor {
    // 自定义音效处理
};
```

### 3. HDI 回调替换

可通过实现 `IDAudioHdiCallback` 接口自定义 HDI 交互：

```cpp
class CustomHdiCallback : public IDAudioHdiCallback {
    // 自定义 HDF 回调处理
};
```

---

*文档生成时间: 2025-02-06*
