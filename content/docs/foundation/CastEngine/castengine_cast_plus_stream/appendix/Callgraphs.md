# Appendix: Callgraphs - 关键调用链

> 本文档提供 Cast+ Stream 模块的关键调用链，展示从入口到核心逻辑的完整调用路径。

---

## 1. 会话生命周期调用链

### 1.1 会话创建与初始化

```
[调用链] 应用创建投屏会话

ICastSessionImpl::AddDevice() [IPC]
└── CastSessionImplStub::DoAddDeviceTask()
    ├── Permission::CheckMirrorPermission() / CheckStreamPermission()
    │   └── CheckPermission() [当前为桩函数]
    │
    └── CastSessionImpl::AddDevice()
        ├── Permission::CheckPidPermission()
        │   ├── IPCSkeleton::GetCallingPid()
        │   └── 检查 PID 白名单
        │
        ├── CastSessionImpl::FindRemoteDevice()
        │   └── 在 remoteDeviceList_ 中查找
        │
        ├── CastSessionImpl::ProcessConnect() [异步消息]
        │   └── Handler::SendMessage(MSG_CONNECT)
        │       └── StateMachine::HandleMessage()
        │           └── DisconnectedState::HandleMessage(MSG_CONNECT)
        │               └── CastSessionImpl::ProcessConnect()
        │                   ├── ChannelManager::CreateChannel(RTSP)
        │                   │   ├── ChannelManager::GetConnection()
        │                   │   │   └── 创建 SoftBusConnection / TcpConnection
        │                   │   └── Connection::StartConnection() / StartListen()
        │                   │       └── SoftBusWrapper::OpenSession() / TcpSocket::Connect()
        │                   │
        │                   └── IRtspController::Start()
        │                       └── RtspController::Start()
        │                           └── RtspChannelManager::StartSession()
        │                               └── 初始化 RTSP 状态机
        │
        └── CastSessionImpl::ChangeDeviceState(CONNECTING)
            └── 通知监听器 OnDeviceState()
```

### 1.2 RTSP 握手流程

```
[调用链] M1-M6 RTSP 握手

# M1: OPTIONS
RtspController::OnRequest(OPTIONS)
└── RtspController::ProcessOptionRequest()
    └── 返回能力列表

# M2: OPTIONS Response
RtspController::OnResponse()
└── RtspController::ProcessCommonResponse()
    └── 更新等待状态

# M3: GET_PARAMETER
RtspController::OnRequest(GET_PARAMETER)
└── RtspController::ProcessGetParameterRequestM3()
    └── 返回本地能力参数
        ├── ProcessVideoInfo()
        ├── ProcessAudioInfo()
        └── ProcessUibc()

# M4: SET_PARAMETER
RtspController::OnRequest(SET_PARAMETER)
└── RtspController::ProcessSetParamRequest()
    └── 解析并存储对方参数
        ├── ParseVideoInfo()
        ├── ParseAudioInfo()
        └── ParseUibc()

# M5: SET_PARAMETER (触发 SETUP)
RtspController::OnRequest(SET_PARAMETER)
└── RtspController::ProcessSetParamRequest()
    └── 检测到 his_trigger_method=SETUP
        └── RtspController::NotifyTrigger(TRIGGER_SETUP)
            └── CastSessionImpl::RtspListenerImpl::NotifyTrigger()
                └── 创建媒体通道

# M6: SETUP
RtspController::OnRequest(SETUP)
└── RtspController::ProcessSetupRequest()
    ├── 协商媒体端口
    └── 发送 SETUP 响应
        └── RtspController::SendSetupResponse()
```

### 1.3 播放控制调用链

```
[调用链] 开始播放

ICastSessionImpl::Play() [IPC]
└── CastSessionImpl::Play()
    ├── Permission::CheckPidPermission()
    └── CastSessionImpl::ProcessPlay() [异步消息]
        └── Handler::SendMessage(MSG_PLAY)
            └── StateMachine::HandleMessage()
                └── PausedState::HandleMessage(MSG_PLAY)
                    └── CastSessionImpl::ProcessPlay()
                        ├── IRtspController::Action(PLAY)
                        │   └── RtspController::Action()
                        │       └── 发送 SET_PARAMETER trigger=PLAY
                        │           └── RtspChannelManager::SendData()
                        │               └── EncryptDecrypt::EncryptData() [加密]
                        │                   └── AES128Encry()
                        │
                        └── CastSessionImpl::TransferTo(PLAYING)
                            └── PlayingState::Enter()
                                └── 通知监听器 OnDeviceState(PLAYING)
```

### 1.4 会话断开调用链

```
[调用链] 断开连接

ICastSessionImpl::RemoveDevice() [IPC]
└── CastSessionImplStub::DoRemoveDeviceTask()
    ├── Permission::CheckMirrorPermission() / CheckStreamPermission()
    └── CastSessionImpl::RemoveDevice()
        ├── CastSessionImpl::ProcessDisconnect() [异步消息]
        │   └── Handler::SendMessage(MSG_DISCONNECT)
        │       └── StateMachine::HandleMessage()
        │           └── [任意状态]::HandleMessage(MSG_DISCONNECT)
        │               └── CastSessionImpl::ProcessDisconnect()
        │                   ├── IRtspController::Action(TEARDOWN)
        │                   │   └── RtspController::Action()
        │                   │       └── 发送 TEARDOWN
        │                   │
        │                   ├── ChannelManager::DestroyAllChannels()
        │                   │   └── Connection::CloseConnection()
        │                   │       └── SoftBusWrapper::CloseSession() / TcpSocket::Close()
        │                   │
        │                   └── CastSessionImpl::TransferTo(DISCONNECTED)
        │                       └── DisconnectedState::Enter()
        │
        └── CastSessionImpl::RemoveRemoteDevice()
            └── 从 remoteDeviceList_ 移除
```

---

## 2. 镜像投屏调用链

### 2.1 镜像播放器创建

```
[调用链] 创建镜像播放器

ICastSessionImpl::CreateMirrorPlayer() [IPC]
└── CastSessionImplStub::DoCreateMirrorPlayer()
    ├── Permission::CheckMirrorPermission()
    └── CastSessionImpl::CreateMirrorPlayer()
        ├── Permission::CheckPidPermission()
        ├── CastSessionImpl::MirrorPlayerGetter()
        │   └── 创建 MirrorPlayerImpl
        │       └── MirrorPlayerImpl::Init()
        │           └── 初始化镜像播放参数
        │
        └── 返回 MirrorPlayerImplStub 对象
            └── new MirrorPlayerImplStub(mirrorPlayerImpl)
```

### 2.2 镜像播放启动

```
[调用链] 启动镜像播放

IMirrorPlayerImpl::Play() [IPC]
└── MirrorPlayerImplStub::DoPlayTask()
    └── MirrorPlayerImpl::Play()
        └── CastSessionImpl::Play()
            └── [同 1.3 播放控制调用链]
                └── 启动屏幕采集
                    ├── 视频采集线程
                    │   └── 采集屏幕帧
                    │       └── 编码 (H.264/H.265)
                    │           └── Channel::SendStream()
                    │               └── SoftBusConnection::SendStream()
                    │                   └── SoftBusWrapper::SendStream()
                    │
                    └── 音频采集线程
                        └── 采集系统音频
                            └── 编码 (AAC)
                                └── Channel::SendStream()
```

### 2.3 输入事件回传

```
[调用链] Sink 端输入事件回传

Sink 设备触摸/按键
└── 系统输入事件
    └── UIBC 通道
        └── CastSessionImpl::RtspListenerImpl::NotifyEventChange()
            └── CastSessionImpl::OnRemoteCtrlEvent()
                └── CastSessionListenerImplProxy::OnRemoteCtrlEvent() [IPC]
                    └── 发送到 Source 端应用
```

---

## 3. 流媒体调用链

### 3.1 流媒体播放器创建

```
[调用链] 创建流媒体播放器

ICastSessionImpl::CreateStreamPlayer() [IPC]
└── CastSessionImplStub::DoCreateStreamPlayer()
    ├── Permission::CheckStreamPermission()
    └── CastSessionImpl::CreateStreamPlayer()
        ├── Permission::CheckPidPermission()
        ├── CastSessionImpl::CreateStreamPlayerManager()
        │   └── 创建 CastStreamManagerClient (Source) / Server (Sink)
        │
        └── 创建 CastStreamPlayer
            └── CastStreamPlayer::Init()
                └── 初始化播放器参数
```

### 3.2 本地文件传输

```
[调用链] 本地文件投射

Source 端:
CastLocalFileChannelServer::AddLocalFileInfo()
└── 注册文件 fd 到 fileInfoMap_

Sink 端请求文件:
CastLocalFileChannelClient::RequestByteData()
└── 发送 HTTP Range 请求
    └── Channel::SendBytes()
        └── 网络传输
            └── Source 端接收
                └── CastLocalFileChannelServer::OnDataReceived()
                    └── CastLocalFileChannelServer::ProcessRequestData()
                        ├── 解析 HTTP 请求
                        ├── 查找文件 fd
                        ├── ReadFileDataByFd() [读取文件数据]
                        └── ResponseFileDataRequest() [发送响应]
                            └── Channel::SendBytes()

Sink 端接收数据:
CastLocalFileChannelClient::OnDataReceived()
└── 写入缓存
    └── LocalDataSource::ReadAt() [播放器读取]
        └── 从缓存读取数据
            └── Media::Player::SetSource(dataSource)
```

### 3.3 播放控制同步

```
[调用链] Source 控制 Sink 播放

Source 端:
IStreamPlayerIpc::Play() [IPC]
└── StreamPlayerImplStub::DoPlayTask()
    └── CastStreamPlayer::Play()
        └── RemotePlayerController::NotifyPeerPlay()
            └── CastStreamManagerClient::SendActionToPeers()
                └── 发送 JSON 动作: {"action": "play"}
                    └── Channel::SendBytes()

Sink 端接收:
Channel::OnDataReceived()
└── CastStreamManagerServer::OnDataReceived()
    └── CastStreamManagerServer::ProcessReceivedData()
        └── 解析 JSON 动作
            └── CastStreamManagerServer::ProcessActionPlay()
                └── CastStreamPlayerManager::Play()
                    └── Media::Player::Play()
                        └── 开始播放

Sink 端状态回调:
Media::Player::OnStateChanged()
└── CastStreamPlayerManager::OnPlayerStatusChanged()
    └── CastStreamManagerServer::NotifyPeerPlayerStatusChanged()
        └── 发送状态到 Source
            └── Source 端更新 UI
```

---

## 4. 通道管理调用链

### 4.1 通道创建

```
[调用链] 创建媒体通道

CastSessionImpl::SetupMedia()
└── ChannelManager::CreateChannel()
    ├── ChannelManager::IsRequestValid() [参数校验]
    │
    ├── ChannelManager::GetConnection(linkType)
    │   └── 根据 linkType 返回 Connection
    │       ├── SOFT_BUS → SoftBusConnection
    │       ├── TCP → TcpConnection
    │       └── VTP → TcpConnection
    │
    └── Connection::StartConnection() / StartListen() [根据角色]
        ├── SoftBusConnection::StartConnection()
        │   └── SoftBusWrapper::OpenSession()
        │       └── OpenSession() [SoftBus SDK]
        │
        └── TcpConnection::StartListen() [Server]
            └── TcpSocket::Bind()
                └── TcpSocket::Listen()
                    └── accept() [等待连接]
```

### 4.2 数据发送

```
[调用链] 发送媒体数据

编码器输出数据
└── Channel::SendStream()
    └── SoftBusConnection::SendStream()
        ├── 数据加密 [可选]
        │   └── EncryptDecrypt::EncryptData()
        │       └── AES128Encry()
        │
        └── SoftBusWrapper::SendStream()
            └── SendStream() [SoftBus SDK]

或 TCP:
Channel::SendBytes()
└── TcpConnection::SendBytes()
    └── TcpSocket::Send()
        └── send() [系统调用]
```

### 4.3 数据接收

```
[调用链] 接收媒体数据

SoftBus 回调:
SoftBusConnection::OnConnectionStreamReceived() [静态回调]
└── SoftBusConnection::instance->OnStreamReceived()
    └── IChannelListener::OnStreamReceived() [通知监听器]
        └── 播放器解码渲染

或 TCP:
TcpConnection::ReceiveThread()
└── TcpSocket::Recv()
    ├── 读取 4 字节长度头
    ├── 长度校验
    ├── 读取 payload
    ├── 数据解密 [可选]
    │   └── EncryptDecrypt::DecryptData()
    │       └── AES128Decrypt()
    │
    └── IChannelListener::OnDataReceived() [通知监听器]
```

---

## 5. 加密调用链

### 5.1 会话密钥协商

```
[调用链] RTSP 加密协商

Source 端发送 ANNOUNCE:
RtspController::Action(ANNOUNCE)
└── 发送支持的加密算法列表
    └── EncryptDecrypt::GetEncryptInfo() → "aes128ctr"

Sink 端接收 ANNOUNCE:
RtspController::OnRequest(ANNOUNCE)
└── RtspController::ProcessAnnounceRequest()
    ├── ParseCipherItem() [解析算法]
    ├── EncryptDecrypt::GetEncryptMatch() [匹配算法]
    └── RtspController::SetNegAlgorithmId() [设置协商结果]

密钥分发:
CastSessionImpl::ProcessSetUpSuccess()
└── 交换 sessionKey [16 字节]
    └── 存储到 CastInnerRemoteDevice.sessionKey[]
```

### 5.2 数据加密

```
[调用链] 加密发送数据

RtspChannelManager::SendData()
└── EncryptDecrypt::EncryptData()
    ├── 生成随机 IV
    │   └── EncryptDecrypt::GetAESIv()
    │       └── RAND_bytes() [OpenSSL]
    │
    └── AES128Encry()
        ├── EVP_CIPHER_CTX_new() [创建上下文]
        ├── EVP_EncryptInit_ex() [初始化]
        ├── EVP_EncryptUpdate() [加密数据]
        ├── EVP_EncryptFinal_ex() [ finalize ]
        └── 输出: IV + ciphertext
```

### 5.3 数据解密

```
[调用链] 解密接收数据

RtspChannelManager::OnDataReceived()
└── EncryptDecrypt::DecryptData()
    ├── 提取 IV (前 16 字节)
    └── AES128Decrypt()
        ├── EVP_CIPHER_CTX_new()
        ├── EVP_DecryptInit_ex()
        ├── EVP_DecryptUpdate()
        ├── EVP_DecryptFinal_ex()
        └── 输出: plaintext
```

---

## 6. 相关文档

- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [05_Internal_API.md](./05_Internal_API.md) - 内部 API
