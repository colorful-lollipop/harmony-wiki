# 关键调用链

## 1. 创建会话调用链

```
JS 应用
    │
    ▼
N-API: createAVSession()
  └─> napi_avsession_manager.cpp::CreateSession()
      │
      ▼
  Native: AVSessionManagerImpl::CreateSession()
      │
      ▼
  IPC Proxy: avsession_service_proxy.cpp::CreateSession()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_service_stub.cpp::HandleCreateSessionInner()
      │
      ├─> 权限检查: PermissionChecker::CheckPermission()
      │   └─> IPCSkeleton::GetCallingTokenID()
      │       └─> AccessTokenKit::VerifyAccessToken()
      │
      ▼
  Service: avsession_service.cpp::CreateSessionInner()
      │
      ▼
  Session Item: avsession_item.cpp::Init()
      │
      ▼
  返回 sessionId
```

**入口**: `frameworks/js/napi/session/src/napi_avsession_manager.cpp:CreateSession()`  
**关键检查**: `utils/src/permission_checker.cpp:CheckPermission()`

---

## 2. 发送控制命令调用链

```
播控中心 (Controller 应用)
    │
    ▼
N-API: sendControlCommand()
  └─> napi_avsession_controller.cpp::SendControlCommand()
      │
      ▼
  Native: AVSessionControllerProxy::SendControlCommand()
      │
      ▼
  IPC Proxy: avsession_controller_proxy.cpp::SendControlCommand()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_controller_stub.cpp::HandleSendControlCommand()
      │
      ├─> 权限检查: PermissionChecker::CheckPermission()
      │   └─> AccessTokenKit::VerifyAccessToken()
      │
      ├─> 归属验证: AVControllerItem::GetOwnerPid()
      │
      ▼
  Service: avsession_service.cpp::ProcessControllerCommand()
      │
      ▼
  Session Item: avsession_item.cpp::OnCommand()
      │
      ├─> 回调分发: AVSessionCallbackClient::OnPlay()
      │   └─> EventHandler::PostTask()
      │
      ▼
  返回给媒体应用 (JS Callback)
```

**入口**: `frameworks/js/napi/session/src/napi_avsession_controller.cpp:SendControlCommand()`  
**关键检查**: `services/session/ipc/stub/avsession_controller_stub.cpp:HandleSendControlCommand()`

---

## 3. 元数据更新调用链

```
媒体应用
    │
    ▼
N-API: setAVMetadata()
  └─> napi_avsession.cpp::SetAVMetaData()
      │
      ▼
  Native: AVSessionProxy::SetAVMetaData()
      │
      ▼
  IPC Proxy: avsession_proxy.cpp::SetAVMetaData()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_stub.cpp::HandleSetAVMetaData()
      │
      ├─> 权限检查: CheckPermission(CHECK_MEDIA_RESOURCES_PERMISSION)
      │
      ├─> 归属验证: AVSessionItem::GetOwnerPid()
      │
      ▼
  Session Item: avsession_item.cpp::SetMetaData()
      │
      ├─> 数据验证: AVMetaData::IsValid()
      │
      ├─> 状态更新: SetState()
      │
      ▼
  广播通知: NotifyMetaDataChange()
      │
      └─> 所有订阅的 Controller 收到 OnMetaDataChange()
```

**入口**: `frameworks/js/napi/session/src/napi_avsession.cpp:SetAVMetaData()`  
**数据验证**: `interfaces/inner_api/native/session/include/avmeta_data.cpp:IsValid()`

---

## 4. 播放状态同步调用链

```
媒体应用
    │
    ▼
N-API: setAVPlaybackState()
  └─> napi_avsession.cpp::SetAVPlaybackState()
      │
      ▼
  Native: AVSessionProxy::SetAVPlaybackState()
      │
      ▼
  IPC Proxy: avsession_proxy.cpp::SetAVPlaybackState()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_stub.cpp::HandleSetAVPlaybackState()
      │
      ├─> 权限检查
      │
      ├─> 归属验证
      │
      ▼
  Session Item: avsession_item.cpp::SetPlaybackState()
      │
      ├─> 状态验证: AVPlaybackState::IsValid()
      │
      ├─> 状态持久化 (可选)
      │
      ▼
  广播通知: NotifyPlaybackStateChange()
      │
      ├─> 播控中心 Controller 收到 OnPlaybackStateChange()
      │
      └─> 分布式会话同步 (RemoteSessionSource)
```

**入口**: `frameworks/js/napi/session/src/napi_avsession.cpp:SetAVPlaybackState()`  
**状态验证**: `interfaces/inner_api/native/session/include/avplayback_state.h:IsValid()`

---

## 5. 设备发现调用链

```
应用
    │
    ▼
N-API: startCastDeviceDiscovery()
  └─> napi_avsession_manager.cpp::StartCastDiscovery()
      │
      ▼
  Native: AVSessionManagerImpl::StartCastDiscovery()
      │
      ▼
  IPC Proxy: avsession_service_proxy.cpp::StartCastDiscovery()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_service_stub.cpp::HandleStartCastDiscovery()
      │
      ├─> 权限检查: CHECK_SYSTEM_PERMISSION
      │
      ▼
  Service: avsession_service.cpp::StartCastDiscoveryInner()
      │
      ├─> 设备管理: DeviceManagerAdapter
      │
      ├─> 软总线发现: SoftbusSessionManager
      │
      ▼
  广播事件: NotifyDeviceAvailable()
      │
      └─> 监听器收到 on('deviceAvailable')
```

**入口**: `frameworks/js/napi/session/src/napi_avsession_manager.cpp:StartCastDiscovery()`  
**权限检查**: `services/session/ipc/stub/avsession_service_stub.cpp:HandleStartCastDiscovery()`

---

## 6. 音频投送调用链

```
应用
    │
    ▼
N-API: castAudio()
  └─> napi_avsession_manager.cpp::CastAudio()
      │
      ▼
  Native: AVSessionManagerImpl::CastAudio()
      │
      ▼
  IPC Proxy: avsession_service_proxy.cpp::CastAudio()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_service_stub.cpp::HandleCastAudio()
      │
      ├─> 权限检查: CHECK_MEDIA_RESOURCES_PERMISSION
      │
      ├─> 设备验证: OutputDeviceInfo 验证
      │
      ▼
  Service: avsession_service.cpp::CastAudioInner()
      │
      ├─> Audio Framework: AudioAdapter
      │
      ├─> Device Manager: 远端设备选择
      │
      ├─> Softbus: 建立音频通道
      │
      ▼
  开始音频流传输
```

**入口**: `frameworks/js/napi/session/src/napi_avsession_manager.cpp:CastAudio()`  
**设备验证**: `interfaces/inner_api/native/session/include/avsession_descriptor.h`

---

## 7. 会话销毁调用链

```
应用
    │
    ▼
N-API: destroy()
  └─> napi_avsession.cpp::Destroy()
      │
      ▼
  Native: AVSessionProxy::Destroy()
      │
      ▼
  IPC Proxy: avsession_proxy.cpp::Destroy()
      │
      ▼ (IPC 跨进程)
      │
      ▼
  IPC Stub: avsession_stub.cpp::HandleDestroy()
      │
      ├─> 权限检查
      │
      ├─> 归属验证
      │
      ▼
  Session Item: avsession_item.cpp::Destroy()
      │
      ├─> 设置销毁状态: SetDestroyed()
      │
      ├─> 清理控制器: DestroyController()
      │
      ├─> 移除监听器: RemoveListener()
      │
      ├─> 广播销毁事件: NotifySessionDestroy()
      │
      ├─> 注销回调: UnregisterCallback()
      │
      └─> 释放资源: ReleaseResources()
```

**入口**: `frameworks/js/napi/session/src/napi_avsession.cpp:Destroy()`  
**清理逻辑**: `services/session/server/avsession_item.cpp:Destroy()`

---

## 8. 跨进程回调分发

```
服务端事件发生 (如 OnPlay)
    │
    ▼
Session Item: avsession_item.cpp::OnPlay()
    │
    ▼
Callback Proxy: avsession_callback_proxy.cpp
    │
    ├─> IPC 调用: SendRequest()
    │   │
    │   ▼ (IPC 跨进程)
    │       │
    │       ▼
    │   Stub: avsession_callback_stub.cpp
    │       │
    │       ▼
    │   Event Handler: PostTask()
    │       │
    │       ▼
    │   Native Callback: AVSessionCallbackClient::OnPlay()
    │       │
    │       ▼
    │   N-API: napi_avsession_callback.cpp::OnPlay()
    │       │
    │       ▼
    └──> JS Callback
```

**回调注册**: `frameworks/js/napi/session/src/napi_avsession.cpp:RegisterCallback()`  
**事件分发**: `utils/src/avsession_event_handler.cpp:AVSessionPostTask()`
