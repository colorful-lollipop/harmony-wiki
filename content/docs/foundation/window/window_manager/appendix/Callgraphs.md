# 附录：关键调用链

## 窗口创建调用链

```
应用层 (JS)
    │
    ▼
window.create(ctx, config)
    │
    ▼
N-API 层 (C++)
    │
    ├── JsWindowManager::Create()
    │   └── [interfaces/kits/napi/window_runtime/window_manager_napi/js_window_manager.cpp]
    │
    ▼
Client 层 (C++)
    │
    ├── Window::Create()
    │   └── [interfaces/innerkits/wm/window.h]
    │
    ├── WindowManager::CreateWindow()
    │   └── [wm/src/window_manager.cpp]
    │
    ├── WindowAdapter::CreateWindow()
    │   └── [wm/src/window_adapter.cpp]
    │
    ▼
IPC 层
    │
    ├── WindowManagerProxy::CreateWindow()
    │   └── [wmserver/src/zidl/window_manager_proxy.cpp]
    │
    ├── IPC: TRANS_ID_CREATE_WINDOW
    │
    ▼
Server 层 (C++)
    │
    ├── WindowManagerStub::OnRemoteRequest()
    │   └── [wmserver/src/zidl/window_manager_stub.cpp]
    │
    ├── WindowManagerService::CreateWindow()
    │   └── [wmserver/src/window_manager_service.cpp]
    │
    ├── WindowController::CreateWindow()
    │   └── [wmserver/src/window_controller.cpp]
    │
    ├── WindowRoot::SaveWindowNode()
    │   └── [wmserver/src/window_root.cpp]
    │
    └── WindowNode::Create()
        └── [wmserver/src/window_node.cpp]
```

## 显示信息获取调用链

```
应用层 (JS)
    │
    ▼
display.getDefaultDisplay()
    │
    ▼
N-API 层 (C++)
    │
    ├── JsDisplayManager::GetDefaultDisplay()
    │   └── [interfaces/kits/napi/display_runtime/js_display_manager.cpp]
    │
    ▼
Client 层 (C++)
    │
    ├── DisplayManager::GetDefaultDisplay()
    │   └── [interfaces/innerkits/dm/display_manager.h]
    │
    ├── DisplayManagerAdapter::GetDefaultDisplay()
    │   └── [dm/src/display_manager_adapter.cpp]
    │
    ▼
IPC 层
    │
    ├── ScreenSessionManagerProxy::GetDefaultDisplayInfo()
    │   └── [window_scene/screen_session_manager/src/zidl/screen_session_manager_proxy.cpp]
    │
    ├── IPC: TRANS_ID_GET_DEFAULT_DISPLAY_INFO
    │
    ▼
Server 层 (C++)
    │
    ├── ScreenSessionManagerStub::OnRemoteRequest()
    │   └── [window_scene/screen_session_manager/src/zidl/screen_session_manager_stub.cpp]
    │
    ├── ScreenSessionManager::GetDefaultDisplayInfo()
    │   └── [window_scene/screen_session_manager/src/screen_session_manager.cpp]
    │
    └── AbstractDisplay::GetDisplayInfo()
        └── [dmserver/src/abstract_display.cpp]
```

## 截图调用链

```
应用层 (JS)
    │
    ▼
screenshot.save()
    │
    ▼
N-API 层 (C++)
    │
    ├── save::MainFunc()
    │   └── [interfaces/kits/napi/screenshot/native_screenshot_module.cpp]
    │
    ▼
Client 层 (C++)
    │
    ├── Screenshot::Save()
    │   └── [snapshot/src/screenshot.cpp]
    │
    ▼
IPC 层
    │
    ├── ScreenSessionManagerProxy::GetScreenSnapshot()
    │
    ├── IPC: TRANS_ID_GET_SCREEN_SNAPSHOT
    │
    ▼
Server 层 (C++)
    │
    ├── ScreenSessionManagerStub::OnRemoteRequest()
    │
    ├── ScreenSessionManager::GetScreenSnapshot()
    │   └── [window_scene/screen_session_manager/src/screen_session_manager.cpp]
    │
    ├── Permission Check: CheckScreenCapturePermission()
    │   └── [window_scene/screen_session_manager/src/screen_session_manager.cpp:2883]
    │
    └── RenderService::GetScreenCapture()
        └── [graphic_graphic_2d]
```

## 场景会话创建调用链 (Scene Board)

```
应用层 (JS)
    │
    ▼
sceneSessionManager.requestSceneSession()
    │
    ▼
N-API 层 (C++)
    │
    ├── JsSceneSessionManager::RequestSceneSession()
    │   └── [window_scene/interfaces/kits/napi/scene_session_manager/js_scene_session_manager.cpp]
    │
    ▼
Client 层 (C++)
    │
    ├── SessionManager::RequestSceneSession()
    │   └── [window_scene/session_manager/src/session_manager.cpp]
    │
    ├── SceneSessionManagerProxy::CreateAndConnectSpecificSession()
    │   └── [window_scene/session_manager/src/zidl/scene_session_manager_proxy.cpp]
    │
    ▼
IPC 层
    │
    ├── IPC: TRANS_ID_CREATE_AND_CONNECT_SPECIFIC_SESSION
    │
    ▼
Server 层 (C++)
    │
    ├── SceneSessionManagerStub::OnRemoteRequest()
    │   └── [window_scene/session_manager/src/zidl/scene_session_manager_stub.cpp]
    │
    ├── SceneSessionManager::CreateAndConnectSpecificSession()
    │   └── [window_scene/session_manager/src/scene_session_manager.cpp]
    │
    ├── Permission Check: IsCallingBundleNameValid()
    │   └── [window_scene/common/src/session_permission.cpp]
    │
    ├── SceneSession::Create()
    │   └── [window_scene/session/host/src/scene_session.cpp]
    │
    └── Session::Connect()
        └── [window_scene/session/host/src/session.cpp]
```

## 输入事件分发调用链

```
硬件层
    │
    ▼
输入设备驱动
    │
    ▼
Multimodal Input Service
    │
    ├── libmmi-client.so
    │
    ▼
Window Manager
    │
    ├── InputTransferStation::OnInputEvent()
    │   └── [wm/src/input_transfer_station.cpp]
    │
    ├── WindowInputChannel::SendInputEvent()
    │   └── [wm/src/window_input_channel.cpp]
    │
    ▼
应用层
    │
    ├── Window::ConsumeInputEvent()
    │   └── [wm/src/window.cpp]
    │
    └── OnTouchEvent() callback
        └── 应用回调
```

## 屏幕旋转调用链

```
传感器层
    │
    ▼
加速度传感器检测方向变化
    │
    ▼
Display Manager Service
    │
    ├── ScreenRotationController::HandleSensorEvent()
    │   └── [dmserver/src/screen_rotation_controller.cpp]
    │
    ├── DisplayManagerService::SetOrientation()
    │   └── [dmserver/src/display_manager_service.cpp]
    │
    ▼
Screen Session Manager
    │
    ├── ScreenSessionManager::SetOrientation()
    │   └── [window_scene/screen_session_manager/src/screen_session_manager.cpp]
    │
    ├── ScreenSession::SetRotation()
    │   └── [window_scene/session/screen/src/screen_session.cpp]
    │
    ▼
Client 通知
    │
    ├── DisplayManagerAgent::OnDisplayChange()
    │   └── [dm/src/zidl/display_manager_agent_stub.cpp]
    │
    ▼
应用层
    │
    └── display.on('change') 回调
```

## 相关文档

- [架构说明](02_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
- [内部 API](05_Inner_API.md)
