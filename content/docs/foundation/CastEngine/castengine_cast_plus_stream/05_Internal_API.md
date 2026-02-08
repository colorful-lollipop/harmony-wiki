# 05_Internal_API - 内部 API

> 本文档详细说明 Cast+ Stream 模块的内部模块接口、依赖方向和稳定性评估。

---

## 1. 内部接口概述

### 1.1 接口分类

| 接口类型 | 说明 | 稳定性 |
|----------|------|--------|
| **Public Headers** | `include/` 目录下的头文件，对外暴露 | 高 |
| **Module Headers** | `src/*/include/` 目录下的头文件，模块间共享 | 中 |
| **Private Headers** | `src/*/` 目录下的头文件，模块内部使用 | 低 |
| **Internal Classes** | 实现文件中的内部类 | 低 |

### 1.2 接口稳定性说明

| 稳定性级别 | 说明 | 变更策略 |
|------------|------|----------|
| **高** | 对外 API，保持向后兼容 | 仅添加，不修改/删除 |
| **中** | 模块间接口，谨慎变更 | 变更需同步修改依赖方 |
| **低** | 内部实现细节，可自由变更 | 按需变更 |

---

## 2. 核心模块接口

### 2.1 CastSessionImpl 接口

**文件**: `include/cast_session_impl_class.h`

**稳定性**: 高（对外暴露）

```cpp
class CastSessionImpl : public StateMachine,
    public CastSessionImplStub,
    public std::enable_shared_from_this<CastSessionImpl> {
public:
    // 构造函数
    CastSessionImpl(const CastSessionProperty &property, const CastLocalDevice &localDevice);
    
    // 初始化
    bool Init();
    
    // 设备管理
    int32_t AddDevice(const CastRemoteDevice &remoteDevice);
    int32_t RemoveDevice(const std::string &deviceId);
    
    // 认证
    int32_t StartAuth(const AuthInfo &authInfo);
    
    // 查询接口
    int32_t GetSessionId(std::string &sessionId);
    int32_t GetDeviceState(const std::string &deviceId, DeviceState &deviceState);
    
    // 属性设置
    int32_t SetSessionProperty(const CastSessionProperty &property);
    
    // 播放器创建
    int32_t CreateMirrorPlayer(sptr<IMirrorPlayerImpl> &mirrorPlayer);
    int32_t CreateStreamPlayer(sptr<IStreamPlayerIpc> &streamPlayer);
    
    // 事件通知
    int32_t NotifyEvent(EventId eventId, std::string &jsonParam);
    int32_t SetCastMode(CastMode mode, std::string &jsonParam);
    
    // 生命周期
    void Stop();
    int32_t Release();
    
    // 内部方法（供 Listener 调用）
    int32_t Play(const std::string &deviceId);
    int32_t Pause(const std::string &deviceId);
    int32_t SetSurface(sptr<IBufferProducer> producer);
    int32_t DeliverInputEvent(const OHRemoteControlEvent &event);
};
```

### 2.2 ChannelManager 接口

**文件**: `src/channel/include/channel_manager.h`

**稳定性**: 中（模块间接口）

```cpp
class ChannelManager {
public:
    // 创建通道
    int CreateChannel(ChannelRequest &channelRequest, 
                      std::shared_ptr<IChannelListener> channelListener);
    int CreateChannel(ChannelRequest &channelRequest,
                      std::shared_ptr<IChannelListener> channelListener,
                      ChannelFileSchema channelFileSchema);
    
    // 销毁通道
    void DestroyChannel(const Channel &channel);
    void DestroyChannel(ModuleType moduleType);
    void DestroyAllChannels();
    
    // 查询接口
    std::shared_ptr<Connection> GetConnection(ChannelLinkType linkType);
    std::shared_ptr<Connection> GetConnectionByModuleType(ModuleType moduleType);
    
    // 设置监听器
    void SetChannelManagerListener(std::shared_ptr<IChannelManagerListener> listener);
    
    // 请求验证
    bool IsRequestValid(const ChannelRequest &channelRequest);
};
```

### 2.3 IRtspController 接口

**文件**: `src/rtsp/include/i_rtsp_controller.h`

**稳定性**: 中（模块间接口）

```cpp
class IRtspController {
public:
    // 生命周期
    virtual bool Start(int sessionId, const uint8_t *sessionKey, int keyLen) = 0;
    virtual void Stop() = 0;
    
    // 动作控制
    virtual bool Action(RtspActionType action, const ParamInfo &param) = 0;
    
    // 参数设置
    virtual void SetParamInfo(ParamInfo &paramInfo) = 0;
    virtual void SetEndType(EndType endType) = 0;
    virtual void SetLocalDeviceInfo(const CastLocalDevice &localDevice) = 0;
    
    // 事件监听
    virtual void SetRtspListener(std::shared_ptr<IRtspListener> listener) = 0;
};
```

### 2.4 ICastStreamManager 接口

**文件**: `src/stream/include/i_cast_stream_manager.h`

**稳定性**: 中（模块间接口）

```cpp
class ICastStreamManager {
public:
    // 动作发送
    virtual bool SendActionToPeers(int action, const std::string &param) = 0;
    
    // 事件监听
    virtual void SetStreamListener(std::shared_ptr<ICastStreamListener> listener) = 0;
    
    // 渲染准备
    virtual void OnRenderReady(bool isReady) = 0;
    
    // 事件处理
    virtual void OnEvent(EventId eventId, const std::string &data) = 0;
};
```

---

## 3. 模块依赖关系

### 3.1 依赖图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            模块依赖关系图                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   依赖方向: A ──► B 表示 A 依赖 B（A 使用 B 的接口）                         │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         cast_session                                │   │
│   │                        (根目标/聚合)                                │   │
│   └─────────────────────────────┬───────────────────────────────────────┘   │
│                                 │                                            │
│     ┌───────────────────────────┼───────────────────────────┐               │
│     │                           │                           │               │
│     ▼                           ▼                           ▼               │
│  ┌─────────────┐          ┌─────────────┐          ┌─────────────┐         │
│  │   channel   │◄─────────│   utils     │─────────►│    rtsp     │         │
│  │  (通道管理)  │          │  (工具库)   │          │  (RTSP协议) │         │
│  └──────┬──────┘          └──────┬──────┘          └──────┬──────┘         │
│         │                        │                        │               │
│         │                        ▼                        │               │
│         │                 ┌─────────────┐                 │               │
│         │                 │   stream    │◄────────────────┘               │
│         │                 │  (流媒体)   │                                 │
│         │                 └──────┬──────┘                                 │
│         │                        │                                        │
│         │                        ▼                                        │
│         │                 ┌─────────────┐                                 │
│         └────────────────►│   mirror    │                                 │
│                           │  (镜像播放) │                                 │
│                           └─────────────┘                                 │
│                                                                              │
│   依赖详情:                                                                  │
│   ─────────────────────────────────────────────────────────────────────    │
│   channel ──► utils: 使用加密、工具函数                                      │
│   rtsp ──► channel: 使用通道传输 RTSP 消息                                   │
│   rtsp ──► utils: 使用加密模块                                               │
│   stream ──► channel: 使用通道传输媒体数据                                   │
│   stream ──► utils: 使用工具函数                                             │
│   mirror ──► channel: 使用通道传输音视频                                     │
│   mirror ──► rtsp: 使用 RTSP 协议控制                                        │
│   mirror ──► stream: 使用流媒体功能                                          │
│   mirror ──► utils: 使用工具函数                                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 依赖矩阵

| 模块 | utils | channel | rtsp | stream | mirror |
|------|-------|---------|------|--------|--------|
| **utils** | - | - | - | - | - |
| **channel** | ✅ | - | - | - | - |
| **rtsp** | ✅ | ✅ | - | - | - |
| **stream** | ✅ | ✅ | - | - | - |
| **mirror** | ✅ | ✅ | ✅ | ✅ | - |
| **root** | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 4. 接口调用链

### 4.1 会话建立调用链

```
CastSessionImpl::AddDevice()
    ├── ChannelManager::CreateChannel() [RTSP]
    │   ├── ChannelManager::GetConnection()
    │   │   └── 创建 SoftBusConnection 或 TcpConnection
    │   └── Connection::StartConnection() / StartListen()
    │
    ├── IRtspController::Start()
    │   └── RtspController::Start()
    │       └── RtspChannelManager::StartSession()
    │
    └── CastSessionImpl::ProcessConnect()
        └── 状态机: DISCONNECTED → CONNECTING
```

### 4.2 播放控制调用链

```
CastSessionImpl::Play()
    ├── IRtspController::Action(PLAY)
    │   └── RtspController::Action()
    │       └── 发送 RTSP PLAY 请求
    │
    └── CastSessionImpl::ProcessPlay()
        └── 状态机: PAUSED → PLAYING
```

### 4.3 镜像播放调用链

```
MirrorPlayerImpl::Play()
    ├── CastSessionImpl::Play()
    │   └── [同上: 播放控制调用链]
    │
    └── 启动屏幕采集
        ├── 视频采集 → 编码 → Channel::SendStream()
        └── 音频采集 → 编码 → Channel::SendStream()
```

### 4.4 流媒体播放调用链

```
CastStreamPlayer::Play()
    ├── RemotePlayerController::NotifyPeerPlay()
    │   └── CastStreamManagerClient::SendActionToPeers()
    │       └── Channel::SendBytes() [JSON 动作]
    │
    └── CastStreamManagerServer::ProcessActionPlay()
        └── CastStreamPlayerManager::Play()
            └── Media::Player::Play()
```

---

## 5. 关键数据结构

### 5.1 会话相关

```cpp
// 会话状态
enum class SessionState : uint8_t {
    DEFAULT,
    DISCONNECTED,
    CONNECTING,
    CONNECTED,
    PLAYING,
    PAUSED,
    DISCONNECTING,
    STREAM,
    AUTHING,
    SESSION_STATE_MAX,
};

// 设备信息
struct CastRemoteDeviceInfo {
    CastInnerRemoteDevice remoteDevice;
    DeviceState deviceState;
};

// 会话属性
struct CastSessionProperty {
    CastMode castMode;           // MIRROR_CAST 或 STREAM_CAST
    ProtocolType protocolType;   // 协议类型
    // ... 其他属性
};
```

### 5.2 通道相关

```cpp
// 通道链接类型
enum class ChannelLinkType : uint8_t {
    SOFT_BUS = 0,
    TCP = 1,
    VTP = 2,
};

// 模块类型
enum class ModuleType : uint8_t {
    AUTH = 0,
    RTSP,
    VIDEO,
    AUDIO,
    RTCP,
    REMOTE_CONTROL,
    STREAM,
    UI_FILES,
    UI_BYTES,
    MODULE_TYPE_MAX,
};

// 通道请求
struct ChannelRequest {
    std::string remoteDeviceId;
    ChannelLinkType linkType;
    ModuleType moduleType;
    EndType endType;
    int port;
    // ... 其他参数
};
```

### 5.3 RTSP 相关

```cpp
// RTSP 动作类型
enum class RtspActionType : uint8_t {
    RTSP_ACTION_PLAY = 0,
    RTSP_ACTION_PAUSE,
    RTSP_ACTION_TEARDOWN,
    // ...
};

// 参数信息
class ParamInfo {
public:
    VideoProperty videoProperty;
    AudioProperty audioProperty;
    UibcProperty uibcProperty;
    // ... 其他参数
};
```

---

## 6. 可替换点

### 6.1 可替换组件

| 组件 | 当前实现 | 可替换为 | 接口 |
|------|----------|----------|------|
| **传输层** | SoftBus/TCP | 自定义协议 | `Connection` |
| **加密算法** | AES-128-CTR/GCM | 其他算法 | `EncryptDecrypt` |
| **状态机** | 自定义 | 其他状态机框架 | `StateMachine` |
| **播放器** | Media::Player | 其他播放器 | `IStreamPlayerImpl` |

### 6.2 扩展点

| 扩展点 | 扩展方式 | 示例 |
|--------|----------|------|
| **RTSP 方法** | 继承 `RtspController` | 添加自定义 RTSP 方法 |
| **通道类型** | 继承 `Connection` | 添加 WebSocket 通道 |
| **事件处理** | 实现 `ICastSessionListener` | 自定义事件处理 |

---

## 7. 相关文档

- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [04_External_API.md](./04_External_API.md) - 对外 API
- [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 关键调用链
