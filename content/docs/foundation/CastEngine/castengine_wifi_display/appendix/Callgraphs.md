# 关键调用链

## N-API 创建流程

```
用户应用 (JS)
       │
       ▼
import { SharingWfd } from '@ohos.multimedia.sharingwfd'
       │
       ▼
SharingWfd.createSink() / SharingWfd.createSource()
       │
       ▼
┌──────────────────────────────────────────────────────────────┐
│  frameworks/kitsimpl/js/wfd/native_module_ohos_wfd.cpp:45-48 │
│  napi_module_register(&g_module)                             │
│                                                              │
│  g_module.nm_register_func = Export                          │
└──────────────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────────┐
│  WfdSinkNapi::CreateSink() / WfdSourceNapi::CreateSource()  │
│  位置: wfd_napi_sink.cpp / wfd_napi_source.cpp              │
└──────────────────────────────────────────────────────────────┘
       │
       ├──► WfdSinkFactory::CreateSink(type, key)
       │         │
       │         ▼
       │     WfdSinkImpl 实例化
       │         │
       │         ├──► AbilityManagerClient::GetInstance()
       │         │         │
       │         │         ▼
       │         │     获取 Ability 运行信息
       │         │
       │         ├──► DmKit::InitDeviceManager()
       │         │         │
       │         │         ▼
       │         │     初始化设备管理
       │         │
       │         └──► WfdSink::SetListener(listener)
       │
       └──► WfdSourceFactory::CreateSource(type, key)
                 │
                 ▼
             WfdSourceImpl 实例化
                 │
                 ├──► AbilityManagerClient::GetInstance()
                 ├──► DmKit::InitDeviceManager()
                 └──► WfdSource::SetListener(listener)
```

## WFD Sink 业务流程

```
WfdSinkImpl.Start()
       │
       ▼
Interaction::Connect(key)
       │
       ├──► IPC/RPC 连接
       │
       ▼
ContextMgr::CreateContext()
       │
       ├──► 创建业务容器
       │
       ▼
Agent::CreateSinkAgent()
       │
       ├──► 创建 Sink Agent
       │
       ▼
Session::StartNegotiation()
       │
       ├──► RTSP 协议交互
       │
       ▼
Mediachannel::CreateChannel()
       │
       ├──► 创建媒体通道
       │
       ▼
Codec::InitDecoder()
       │
       ├──► 初始化解码器
       │
       ▼
Network::StartListen()
       │
       ├──► 启动 RTP 接收
       │
       ▼
Surface::RegisterBufferProducer()
       │
       └──► 注册渲染 Surface
```

## WFD Source 业务流程

```
WfdSourceImpl.StartDiscovery()
       │
       ├──► DmKit::GetTrustedDevicesInfo()
       │         │
       │         └──► 扫描 WiFi 设备
       │
       ▼
Agent::StartSrcAgent()
       │
       ├──► 创建 Source Agent
       │
       ▼
Session::SendM2()
       │
       ├──► RTSP M2 消息
       │
       ▼
Mediachannel::CreateProducer()
       │
       ├──► ScreenCapture::StartCapture()
       │         │
       │         ├──► 屏幕采集
       │         └──► 视频编码
       │
       ├──► Codec::InitEncoder()
       │         │
       │         └──► H.264/H.265 编码
       │
       └──► Network::SendRTP()
                 │
                 └──► 发送 RTP 流
```

## 事件流

```
┌──────────────────────────────────────────────────────────────┐
│                        事件流向                               │
│                                                              │
│  WFD 设备 ──► Network (RTP/RTSP)                           │
│                  │                                           │
│                  ├──► Protocol (解析)                       │
│                  │         │                                  │
│                  │         ▼                                 │
│                  │     EventScheduler (事件分发)             │
│                  │         │                                  │
│                  │         ├──► Agent (业务处理)            │
│                  │         │         │                        │
│                  │         │         └──► Session            │
│                  │         │                                    │
│                  │         └──► ContextMgr                    │
│                  │                  │                         │
│                  │                  └──► Mediachannel        │
│                  │                                              │
│                  └──► IWfdEventListener (回调通知)            │
│                             │                                  │
│                             ▼                                  │
│                       N-API 层 (JS 回调)                       │
└──────────────────────────────────────────────────────────────┘
```
