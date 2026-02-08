# 关键调用链

## 目的与适用范围

本文档描述分布式屏幕的关键调用链，用于代码理解和问题定位。

---

## 调用链 1: 设备使能流程 (Source端)

### 调用图

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         设备使能调用链 (Source端)                                    │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   DH Framework                                                                       │
│        │                                                                            │
│        │ RegisterDistributedHardware(devId, dhId, param, reqId)                     │
│        ▼                                                                            │
│   DScreenSourceHandler                                                               │
│        │                                                                            │
│        │ RegisterDistributedHardware()                                               │
│        ▼                                                                            │
│   DScreenSourceProxy ─────────IPC────────► DScreenSourceStub                        │
│        │                                      │                                    │
│        │                                      │ OnRemoteRequest(REGISTER_DISTRIBUTED_HARDWARE)
│        │                                      ▼                                    │
│        │                                DScreenSourceStub                           │
│        │                                      │                                    │
│        │                                      │ RegisterDistributedHardwareInner()  │
│        │                                      │  - HasEnableDHPermission() ✓        │
│        │                                      ▼                                    │
│        │                                DScreenSourceService                        │
│        │                                      │                                    │
│        │                                      │ RegisterDistributedHardware()       │
│        │                                      ▼                                    │
│        │                                DScreenManager                               │
│        │                                      │                                    │
│        │                                      │ EnableDistributedScreen()           │
│        │                                      ▼                                    │
│        │                                DScreen                                      │
│        │                                      │                                    │
│        │                                      │ Enable(screenInfo, version)         │
│        │                                      │  - CreateTrans()                    │
│        │                                      │  - SetupImageSourceProcessor()      │
│        │                                      │  - Start()                          │
│        │                                      ▼                                    │
│        │                                ScreenSourceTrans                            │
│        │                                      │                                    │
│        │                                      │ SetUp()                             │
│        │                                      │  - CreateSoftBusSession()           │
│        │                                      │  - NegotiateCodec()                 │
│        │                                      ▼                                    │
│        │                                SoftbusAdapter                               │
│        │                                      │                                    │
│        │                                      │ CreateSoftBusSession()              │
│        │                                      │  - CheckSrcPermission() ✓           │
│        │                                      ▼                                    │
│        └───────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 关键代码路径

**入口**: `interfaces/innerkits/native_cpp/screen_source/src/dscreen_source_handler.cpp`

**IPC调用**: `interfaces/innerkits/native_cpp/screen_source/src/dscreen_source_proxy.cpp`

**Stub处理**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_stub.cpp`

**服务实现**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_service.cpp`

**屏幕管理**: `services/screenservice/sourceservice/dscreenmgr/2.0/src/dscreen_manager.cpp`

---

## 调用链 2: 屏幕数据传输流程

### 调用图

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         屏幕数据传输调用链                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Source端 (主控)                                                        Sink端 (被控) │
│                                                                                     │
│   [Graphic System]                                                                 │
│        │                                                                            │
│        │ OnBufferAvailable                                                         │
│        ▼                                                                            │
│   ImageSourceEncoder                                                                │
│        │                                                                            │
│        │ OnOutputBufferAvailable                                                   │
│        ▼                                                                            │
│   ScreenSourceTrans                                                                 │
│        │                                                                            │
│        │ FeedChannelData() ───FFRT异步──► SendData()                               │
│        │                                                                            │
│        ▼                                                                            │
│   ScreenDataChannelImpl                                                             │
│        │                                                                            │
│        │ SendData()                                                                │
│        ▼                                                                            │
│   SoftbusAdapter                                                                    │
│        │                                                                            │
│        │ SendBytes() ──────────────────────────────────────────────────────────────►│
│        │                           软总线传输                                       │
│        │                                                                            │
│        │ ◄─────────────────────────────────────────────────────────────────────────│
│        │                           对端接收                                        │
│        ▼                                                                            │
│                                                              ScreenDataChannelImpl  │
│                                                                      │              │
│                                                                      │ OnDataReceived│
│                                                                      ▼              │
│                                                              ScreenSinkTrans        │
│                                                                      │              │
│                                                                      │ FeedData()   │
│                                                                      ▼              │
│                                                              ImageSinkDecoder       │
│                                                                      │              │
│                                                                      │ Decode()     │
│                                                                      ▼              │
│                                                              [Display Window]       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 关键代码路径

**编码器**: `services/screentransport/screensourceprocessor/encoder/src/image_source_encoder.cpp`

**Source传输**: `services/screentransport/screensourcetrans/src/screen_source_trans.cpp`

**数据通道**: `services/screentransport/screendatachannel/src/screen_data_channel_impl.cpp`

**软总线适配**: `services/softbusadapter/src/softbus_adapter.cpp`

**Sink传输**: `services/screentransport/screensinktrans/src/screen_sink_trans.cpp`

**解码器**: `services/screentransport/screensinkprocessor/decoder/src/image_sink_decoder.cpp`

---

## 调用链 3: IPC通信流程

### 调用图

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         IPC通信调用链                                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Client App                                                                         │
│        │                                                                            │
│        │ InitSource(params, callback)                                               │
│        ▼                                                                            │
│   DScreenSourceHandler                                                               │
│        │                                                                            │
│        │ GetInstance()                                                              │
│        │ InitSource()                                                               │
│        ▼                                                                            │
│   DScreenSourceProxy                                                                 │
│        │                                                                            │
│        │ InitSource() ─────────────────────────────────────────────────────────────►│
│        │      │                          IPC (Binder)                               │
│        │      │                                                                    │
│        │      │ ┌─────────────────────────────────────────────────────────────────┐│
│        │      │ │  1. SendRequest(INIT_SOURCE)                                    ││
│        │      │ │  2. data.WriteInterfaceToken()                                  ││
│        │      │ │  3. data.WriteString(params)                                    ││
│        │      │ │  4. data.WriteRemoteObject(callback)                            ││
│        │      │ │  5. remote_->SendRequest()                                      ││
│        │      │ └─────────────────────────────────────────────────────────────────┘│
│        │      │                                                                    │
│        │ ◄────┘                                                                    │
│        │                           服务端处理                                       │
│        │      ┌─────────────────────────────────────────────────────────────────┐  │
│        │      │                                                                 │  │
│        │      ▼                                                                 │  │
│        │   DScreenSourceStub ──OnRemoteRequest()                                │  │
│        │          │                                                             │  │
│        │          │ 1. ReadInterfaceToken() 验证                                 │  │
│        │          │ 2. HasEnableDHPermission() 权限检查                          │  │
│        │          │ 3. switch(code) -> InitSourceInner()                         │  │
│        │          ▼                                                             │  │
│        │   DScreenSourceService ──InitSource()                                  │  │
│        │          │                                                             │  │
│        │          │ 1. Init()                                                   │  │
│        │          │ 2. RegisterCallback()                                       │  │
│        │          ▼                                                             │  │
│        │   DScreenManager ──Init()                                              │  │
│        │          │                                                             │  │
│        │          ▼                                                             │  │
│        │   reply.WriteInt32(result)                                             │  │
│        │                                                                 │  │
│        │      └─────────────────────────────────────────────────────────────────┘  │
│        │                                                                            │
│        ▼                                                                            │
│   返回结果给Client                                                                   │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 关键代码路径

**Handler**: `interfaces/innerkits/native_cpp/screen_source/src/dscreen_source_handler.cpp`

**Proxy**: `interfaces/innerkits/native_cpp/screen_source/src/dscreen_source_proxy.cpp`

**Stub**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_stub.cpp`

**Service**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_service.cpp`

---

## 调用链 4: 回调通知流程

### 调用图

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         回调通知调用链                                               │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Source Service                                                                     │
│        │                                                                            │
│        │ 使能成功/失败                                                               │
│        ▼                                                                            │
│   DScreenSourceCallbackProxy ─────────IPC────────► DScreenSourceCallbackStub        │
│        │                                              │ (Client侧)                 │
│        │                                              │                            │
│        │                                              ▼                            │
│        │                                        应用回调接口                        │
│        │                                              │                            │
│        │                                              │ OnNotifyRegResult()        │
│        │                                              │ OnNotifyUnregResult()      │
│        │                                              ▼                            │
│        │                                        应用处理结果                        │
│        │                                                                            │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 关键代码路径

**CallbackProxy**: `services/screenservice/sourceservice/dscreenservice/src/callback/dscreen_source_callback_proxy.cpp`

**Callback定义**: `interfaces/innerkits/native_cpp/screen_source/include/idscreen_source_callback.h`

---

## 相关跳转

- [对外接口](../03_Interfaces.md) - 接口定义
- [内部接口](../04_Inner_APIs.md) - 内部模块接口
- [架构设计](../01_Architecture.md) - 架构说明