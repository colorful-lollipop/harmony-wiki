# 关键调用链

## 1. Source SDK 初始化调用链

```
GetSourceHardwareHandler()
    │
    ▼
dcamera_source_handler.cpp
    │
    ├──► InitSource(params, callback)
    │       │
    │       ├──► DistributedCameraSourceProxy::InitSource()
    │       │       │
    │       │       └──► IPC: InitSourceInner()
    │       │               │
    │       │               └──► SourceStub::InitSourceInner()
    │       │                       │
    │       │                       ├──► HasEnableDHPermission()
    │       │                       └──► DistributedCameraSourceService::InitSource()
    │       │                               │
    │       │                               └──► DCameraSourceStateMachine::Init()
    │
    └──► callback 回调
            │
            └──► IDCameraSourceCallback::OnNotifyRegResult()
```

---

## 2. Sink SDK 初始化调用链

```
GetSinkHardwareHandler()
    │
    ▼
dcamera_sink_handler.cpp
    │
    ├──► InitSink(params, callback)
    │       │
    │       ├──► DistributedCameraSinkProxy::InitSink()
    │       │       │
    │       │       └──► IPC: InitSinkInner()
    │       │               │
    │       │               └──► SinkStub::InitSinkInner()
    │       │                       │
    │       │                       ├──► HasEnableDHPermission()
    │       │                       └──► DistributedCameraSinkService::InitSink()
    │       │                               │
    │       │                               └──► 启动相机会话
    │
    └──► callback 回调
            │
            └──► IDCameraSinkCallback::OnNotifyResourceInfo()
```

---

## 3. 设备注册调用链

```
RegisterDistributedHardware(devId, dhId, reqId, param)
    │
    ▼
DistributedCameraSourceProxy::RegisterDistributedHardware()
    │
    └──► IPC: RegisterDistributedHardwareInner()
            │
            └──► SourceStub::RegisterDistributedHardwareInner()
                    │
                    ├──► HasEnableDHPermission()
                    ├──► InterfaceToken 校验
                    │
                    └──► DistributedCameraSourceService::RegisterDistributedHardware()
                            │
                            ├──► DCameraSourceStateMachine::TransitState(Regist)
                            │       │
                            │       └──► DCameraSourceRegistState::Enter()
                            │               │
                            │               ├──► RegisterToDHDF()
                            │               │       │
                            │               │       └──► HDF: RegisterProvider()
                            │               │               │
                            │               │               └──► Camera Framework: 注册虚拟相机
                            │               │
                            │               └──► callback.OnNotifyRegResult()
                            │                       │
                            │                       └──► Proxy: OnNotifyRegResult()
                            │
                            └──► tokenId_ = GetFirstCallerTokenID()
```

---

## 4. 预览数据流调用链 (Source → Sink)

```
Camera Framework
    │
    ├──► DCameraSourceDataProcess::PutInputBuffer()
    │       │
    │       └──► DCameraPipelineSource::ProcessData()
    │               │
    │               ├──► 编码处理
    │               │       │
    │               │       └──► EncodeVideoCallback::OnOutputBufferAvailable()
    │               │
    │               └──► Channel: SendData()
    │                       │
    │                       └──► DCameraSoftbusSession::SendBytes()
    │                               │
    │                               └──► SoftBus: 发送数据
    │
    ▼
SoftBus
    │
    └──► DCameraSoftbusAdapter::OnSourceBytesReceived()
            │
            └──► DCameraSoftbusSession::OnBytesReceived()
                    │
                    ├──► 数据包校验
                    │       │
                    │       └──► CheckUnPackBuffer()
                    │
                    └──► DCameraSinkDataProcess::PutInputBuffer()
                            │
                            └──► 传递给 Sink 回调
                                    │
                                    └──► IDCameraSinkCallback::OnFrameAvailable()
```

---

## 5. 权限校验调用链

```
IPC 调用进入 Stub
    │
    ▼
OnRemoteRequest(requestCode, data, reply)
    │
    ├──► ReadInterfaceToken()
    │       │
    │       └──► 校验 descriptor
    │
    ├──► 根据 requestCode 分发
    │       │
    │       └──► 调用对应的 *Inner() 方法
    │
    └──► *Inner() 方法
            │
            ├──► HasEnableDHPermission() / HasAccessDHPermission()
            │       │
            │       ├──► IPCSkeleton::GetCallingTokenID()
            │       │       │
            │       │       └──► 获取调用者 TokenID
            │       │
            │       └──► AccessTokenKit::VerifyAccessToken()
            │               │
            │               └──► 返回 PERMISSION_GRANTED/DENIED
            │
            ├──► (校验失败) 返回 DCAMERA_BAD_VALUE
            │
            └──► (校验成功) 执行业务逻辑
```

---

## 6. 通道建立调用链

```
Sink: OpenChannel(dhId, openInfo)
    │
    ▼
DistributedCameraSinkProxy::OpenChannel()
    │
    └──► IPC: OpenChannelInner()
            │
            └──► SinkStub::OpenChannelInner()
                    │
                    ├──► HasAccessDHPermission()
                    │
                    └──► DistributedCameraSinkService::OpenChannel()
                            │
                            ├──► OpenSoftbusSession()
                            │       │
                            │       ├──► DCameraChannelSinkImpl::OpenSession()
                            │       │       │
                            │       │       └──► DCameraSoftbusAdapter::OpenSession()
                            │       │               │
                            │       │               └──► SoftBus: CreateSession()
                            │       │
                            │       └──► OnSessionOpened()
                            │               │
                            │               └──► DCameraSoftbusSession::OnSessionOpened()
                            │                       │
                            │                       └──► 触发 ChannelNeg()
                            │
                            └──► 回调 OnChannelOpened()
```

---

## 7. Token 传递调用链

```
IPC 调用进入 (TokenID 获取)
    │
    ▼
IPCSkeleton::GetCallingTokenID()
    │
    └──► 返回 AccessTokenID
            │
            ▼
Stub: GetFirstCallerTokenID()
    │
    └──► 保存 tokenId_ = callingToken
            │
            ▼
Controller: SetTokenId(tokenId)
    │
    └──► 传递 TokenID 到内部模块
            │
            ▼
后续操作中使用 TokenID
    │
    ├──► 访问控制检查
    ├──► 审计日志记录
    └──► 权限验证
```
