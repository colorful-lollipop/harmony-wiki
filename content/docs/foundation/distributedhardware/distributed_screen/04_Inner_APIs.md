# 内部接口

## 目的与适用范围

本文档描述分布式屏幕内部模块间的接口定义和依赖关系。

---

## 模块接口概览

### 模块依赖图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           内部模块接口依赖关系                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌───────────────────────────────────────────────────────────────────┐    │
│   │                     Screen Service Layer                           │    │
│   │  ┌─────────────────┐        ┌─────────────────┐                   │    │
│   │  │ DScreenSource   │        │ DScreenSink     │                   │    │
│   │  │ Service         │        │ Service         │                   │    │
│   │  └────────┬────────┘        └────────┬────────┘                   │    │
│   │           │                          │                            │    │
│   │           ▼                          ▼                            │    │
│   │  ┌─────────────────┐        ┌─────────────────┐                   │    │
│   │  │ DScreenManager  │        │ ScreenRegionMgr │                   │    │
│   │  │ (v1.0/v2.0)     │        │ (v1.0/v2.0)     │                   │    │
│   │  └────────┬────────┘        └────────┬────────┘                   │    │
│   └───────────┼──────────────────────────┼───────────────────────────┘    │
│               │                          │                                 │
│               ▼                          ▼                                 │
│   ┌───────────────────────────────────────────────────────────────────┐    │
│   │                     Transport Layer                                │    │
│   │  ┌─────────────────┐        ┌─────────────────┐                   │    │
│   │  │ ScreenSource    │◄──────►│ ScreenSink      │                   │    │
│   │  │ Trans           │        │ Trans           │                   │    │
│   │  └────────┬────────┘        └────────┬────────┘                   │    │
│   │           │                          │                            │    │
│   │           ▼                          ▼                            │    │
│   │  ┌─────────────────┐        ┌─────────────────┐                   │    │
│   │  │ ImageSource     │        │ ImageSink       │                   │    │
│   │  │ Processor       │        │ Processor       │                   │    │
│   │  └────────┬────────┘        └────────┬────────┘                   │    │
│   └───────────┼──────────────────────────┼───────────────────────────┘    │
│               │                          │                                 │
│               └──────────┬───────────────┘                                 │
│                          ▼                                                 │
│   ┌───────────────────────────────────────────────────────────────────┐    │
│   │                     SoftBus Adapter Layer                          │    │
│   │                    (SoftbusAdapter)                                │    │
│   │                                                                   │    │
│   │  - CreateSoftBusSession()   - SendData()                         │    │
│   │  - RegisterSoftBusListener() - CloseSoftBusSession()             │    │
│   └───────────────────────────────────────────────────────────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Service层内部接口

### DScreenManager 接口

**文件**: `services/screenservice/sourceservice/dscreenmgr/2.0/include/dscreen_manager.h`

```cpp
class DScreenManager {
public:
    static DScreenManager *GetInstance();
    
    // 初始化/释放
    int32_t Init();
    void UnInit();
    
    // 屏幕管理
    int32_t EnableDistributedScreen(const std::string &devId, 
                                    const std::string &dhId,
                                    const EnableParam &param);
    int32_t DisableDistributedScreen(const std::string &devId, 
                                     const std::string &dhId);
    
    // 查询
    std::shared_ptr<DScreen> FindDScreenById(const std::string &screenId);
    std::map<std::string, std::shared_ptr<DScreen>> &GetAllDScreens();
    
    // 回调注册
    void RegisterDScreenCallback(const std::shared_ptr<DScreenCallback> &callback);
    void UnRegisterDScreenCallback();
};
```

**职责**: 管理所有分布式屏幕实例，负责屏幕的使能、去使能和查询。

---

### DScreen 接口

**文件**: `services/screenservice/sourceservice/dscreenmgr/2.0/include/dscreen.h`

```cpp
class DScreen : public DScreenCallback {
public:
    DScreen(const std::string &devId, const std::string &screenId);
    ~DScreen();
    
    // 使能/去使能
    int32_t Enable(const std::string &screenInfo, const std::string &version);
    int32_t Disable();
    
    // 连接管理
    int32_t Connect();
    int32_t Disconnect();
    int32_t AddScreen();
    int32_t RemoveScreen();
    
    // 获取器
    std::string GetDevId() const;
    std::string GetScreenId() const;
    DScreenState GetState() const;
    std::shared_ptr<ScreenSourceTrans> GetScreenSourceTrans();
};
```

**状态机**:
```
DISABLED -> ENABLING -> ENABLED -> CONNECTING -> CONNECTED -> DISCONNECTING -> DISABLED
```

---

### ScreenRegionManager 接口

**文件**: `services/screenservice/sinkservice/screenregionmgr/2.0/include/screenregionmgr.h`

```cpp
class ScreenRegionManager {
public:
    static ScreenRegionManager *GetInstance();
    
    // 初始化/释放
    int32_t Init();
    void UnInit();
    
    // 区域管理
    int32_t CreateDScreenRegion(const std::string &screenId, 
                                const std::string &screenInfo);
    int32_t ReleaseDScreenRegion(const std::string &screenId);
    
    // 查询
    std::shared_ptr<ScreenRegion> FindScreenRegionById(const std::string &screenId);
    std::map<std::string, std::shared_ptr<ScreenRegion>> &GetAllScreenRegions();
};
```

**职责**: 管理被控端的屏幕显示区域。

---

### ScreenRegion 接口

**文件**: `services/screenservice/sinkservice/screenregionmgr/2.0/include/screenregion.h`

```cpp
class ScreenRegion : public std::enable_shared_from_this<ScreenRegion> {
public:
    ScreenRegion(const std::string &screenId);
    ~ScreenRegion();
    
    // 设置
    int32_t SetUp(const std::string &screenInfo);
    int32_t Start();
    int32_t Stop();
    
    // 窗口管理
    int32_t ShowWindow();
    int32_t HideWindow();
    int32_t MoveWindow(int32_t startX, int32_t startY);
    
    // 获取器
    std::string GetScreenId() const;
    std::shared_ptr<ScreenSinkTrans> GetScreenSinkTrans();
};
```

---

## Transport层内部接口

### ScreenSourceTrans 接口

**文件**: `services/screentransport/screensourcetrans/include/screen_source_trans.h`

```cpp
class ScreenSourceTrans : public IScreenChannelListener,
                          public IImageProcessorListener {
public:
    ScreenSourceTrans();
    ~ScreenSourceTrans();
    
    // 初始化/释放
    int32_t SetUp(const VideoParam &localParam, const VideoParam &remoteParam,
                  const std::string &remoteDevId);
    int32_t Release();
    
    // 启动/停止
    int32_t Start();
    int32_t Stop();
    
    // 数据输入
    void OnImageProcessDone(const std::shared_ptr<DataBuffer> &data);
    int32_t FeedChannelData(const std::shared_ptr<DataBuffer> &data);
    
    // 会话回调
    void OnSessionOpened() override;
    void OnSessionClosed() override;
    void OnDataReceived(const std::shared_ptr<DataBuffer> &data) override;
};
```

**职责**: 主控端传输管理，负责编码后的数据传输。

---

### ScreenSinkTrans 接口

**文件**: `services/screentransport/screensinktrans/include/screen_sink_trans.h`

```cpp
class ScreenSinkTrans : public IScreenChannelListener {
public:
    ScreenSinkTrans();
    ~ScreenSinkTrans();
    
    // 初始化/释放
    int32_t SetUp(const VideoParam &localParam, const VideoParam &remoteParam);
    int32_t Release();
    
    // 启动/停止
    int32_t Start();
    int32_t Stop();
    
    // 数据接收
    void OnDataReceived(const std::shared_ptr<DataBuffer> &data) override;
    void OnSessionOpened() override;
    void OnSessionClosed() override;
};
```

**职责**: 被控端传输管理，负责接收数据并送入解码器。

---

### ImageSourceEncoder 接口

**文件**: `services/screentransport/screensourceprocessor/encoder/include/image_source_encoder.h`

```cpp
class ImageSourceEncoder : public Media::MediaCodecCallback {
public:
    ImageSourceEncoder();
    ~ImageSourceEncoder();
    
    // 配置
    int32_t ConfigureEncoder(const VideoParam &param);
    int32_t ReleaseEncoder();
    
    // 编解码器回调
    void OnInputBufferAvailable(uint32_t index, std::shared_ptr<Media::AVBuffer> buffer) override;
    void OnOutputBufferAvailable(uint32_t index, std::shared_ptr<Media::AVBuffer> buffer) override;
    void OnOutputFormatChanged(const Media::Format &format) override;
    void OnError(Media::AVCodecErrorType errorType, int32_t errorCode) override;
};
```

**职责**: 视频编码器管理，支持H264/H265/MPEG4编码。

---

### ImageSinkDecoder 接口

**文件**: `services/screentransport/screensinkprocessor/decoder/include/image_sink_decoder.h`

```cpp
class ImageSinkDecoder : public Media::MediaCodecCallback {
public:
    ImageSinkDecoder();
    ~ImageSinkDecoder();
    
    // 配置
    int32_t ConfigureDecoder(const VideoParam &param);
    int32_t ReleaseDecoder();
    
    // 数据输入
    int32_t InputScreenData(const std::shared_ptr<DataBuffer> &data);
    
    // 编解码器回调
    void OnInputBufferAvailable(uint32_t index, std::shared_ptr<Media::AVBuffer> buffer) override;
    void OnOutputBufferAvailable(uint32_t index, std::shared_ptr<Media::AVBuffer> buffer) override;
    void OnOutputFormatChanged(const Media::Format &format) override;
    void OnError(Media::AVCodecErrorType errorType, int32_t errorCode) override;
};
```

**职责**: 视频解码器管理。

---

## SoftBus Adapter层接口

### SoftbusAdapter 接口

**文件**: `services/softbusadapter/include/softbus_adapter.h`

```cpp
class SoftbusAdapter {
public:
    static SoftbusAdapter *GetInstance();
    
    // 初始化/释放
    int32_t InitSoftbusAdapter();
    void ReleaseSoftbusAdapter();
    
    // 会话管理
    int32_t CreateSoftBusSession(const std::string &sessionName,
                                 const std::string &peerDeviceId);
    int32_t CloseSoftBusSession(int32_t sessionId);
    int32_t GetSessionIdByDeviceId(const std::string &deviceId);
    
    // 数据传输
    int32_t SendData(int32_t sessionId, const std::shared_ptr<DataBuffer> &data);
    int32_t SendBytes(int32_t sessionId, const std::shared_ptr<DataBuffer> &data);
    
    // 监听器管理
    int32_t RegisterSoftBusListener(const std::shared_ptr<ISoftbusListener> &listener,
                                    const std::string &sessionName);
    void UnRegisterSoftBusListener(const std::string &sessionName);
    
    // 权限检查
    bool CheckSrcPermission(const std::string &sinkNetworkId);
};
```

**职责**: 封装软总线传输接口，提供统一的设备发现、会话管理和数据传输能力。

---

### ISoftbusListener 接口

**文件**: `services/softbusadapter/include/isoftbus_listener.h`

```cpp
class ISoftbusListener {
public:
    virtual ~ISoftbusListener() = default;
    
    // 会话事件
    virtual void OnSessionOpened(int32_t sessionId) = 0;
    virtual void OnSessionClosed(int32_t sessionId) = 0;
    
    // 数据接收
    virtual void OnDataReceived(int32_t sessionId, 
                                const std::shared_ptr<DataBuffer> &data) = 0;
    
    // 事件通知
    virtual void OnStreamReceived(int32_t sessionId, 
                                  const std::shared_ptr<DataBuffer> &data) {}
};
```

---

### SoftBusPermissionCheck 接口

**文件**: `services/softbusadapter/include/softbus_permission_check.h`

```cpp
class SoftBusPermissionCheck {
public:
    // 权限检查
    static bool CheckSrcPermission(const std::string &sinkNetworkId);
    static bool CheckSinkPermission(const AccountInfo &callerAccountInfo);
    
    // 账户信息
    static bool GetLocalAccountInfo(AccountInfo &localAccountInfo);
    static int32_t GetCurrentUserId();
    
    // 访问信息设置
    static bool SetAccessInfoToSocket(const int32_t sessionId);
};
```

**职责**: 提供软总线连接的权限检查和同账号验证。

---

## ScreenClient层接口

### ScreenClient 接口

**文件**: `services/screenclient/include/screen_client.h`

```cpp
class ScreenClient {
public:
    static ScreenClient &GetInstance();
    
    // 窗口管理
    int32_t AddWindow(const std::string &screenId, int32_t windowId);
    int32_t RemoveWindow(int32_t windowId);
    int32_t ShowWindow(int32_t windowId);
    int32_t HideWindow(int32_t windowId);
    int32_t MoveWindow(int32_t windowId, int32_t startX, int32_t startY);
    
    // Surface获取
    sptr<Surface> GetSurface(int32_t windowId);
    
    // 窗口查询
    int32_t GetWindowId(const std::string &screenId);
};
```

**职责**: 管理代理显示窗口，提供Surface用于解码器渲染。

---

## 接口稳定性标注

| 接口层 | 稳定性 | 说明 |
|--------|--------|------|
| **SDK层** (`IDScreenSource`/`IDScreenSink`) | **稳定** | 对外公开接口，向后兼容 |
| **Service层** (`DScreenManager`/`ScreenRegionManager`) | **内部** | 服务内部使用，可能变动 |
| **Transport层** (`ScreenSourceTrans`等) | **内部** | 服务内部使用，可能变动 |
| **SoftBus层** (`SoftbusAdapter`) | **内部** | 服务内部使用，可能变动 |

---

## 相关跳转

- [对外接口](03_Interfaces.md) - SDK接口定义
- [架构设计](01_Architecture.md) - 架构说明
- [安全风险](07_Security.md) - 安全分析