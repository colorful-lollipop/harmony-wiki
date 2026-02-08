# appendix/Callgraphs - 关键调用链

## 概述

本文档描述 multimodalinput_input 子系统的关键调用链，便于理解代码执行路径。

---

## 7.1 事件注入调用链

### 7.1.1 JS → 服务端完整调用链

```
JS injectEvent()
    │
    ▼
N-API Bridge (js_register_module.cpp)
    │
    ├── 参数校验
    ├── 参数转换 (JS → C++)
    │
    ▼
InputManagerImpl::InjectInputEvent()
    │
    ▼
ConnectManagerProxy::SendRequest()
    │
    ▼
IPC 调用 (MessageParcel)
    │
    ▼
MMIService::OnRemoteRequest()
    │
    ▼
权限检查 (PermissionHelper::CheckInjectPermission)
    │
    ▼
事件处理链 (InputEventHandler)
    │
    ▼
EventDispatchHandler
    │
    ▼
UDS Socket → 客户端
```

### 7.1.2 关键代码路径

| 步骤 | 文件 | 函数 |
|------|------|------|
| JS 调用 | `js_register_module.cpp:169` | `InjectEvent()` |
| 参数校验 | `js_register_module.cpp:176-200` | `InjectEvent()` |
| 参数转换 | `js_register_module.cpp:127-166` | `GetInjectionEventData()` |
| IPC 发送 | `input_manager_impl.h` | `InjectInputEvent()` |
| 权限检查 | `permission_helper.cpp:62` | `CheckInjectPermission()` |
| 事件分发 | `event_dispatch_handler.cpp` | `HandleEvent()` |

---

## 7.2 设备查询调用链

```
JS getDeviceList()
    │
    ▼
N-API Bridge (js_input_device_context.cpp)
    │
    ▼
InputManagerImpl::GetDeviceList()
    │
    ▼
DeviceManager::GetDeviceList()
    │
    ▼
返回设备信息列表
```

---

## 7.3 IPC 通信调用链

### 7.3.1 请求发送 (客户端)

```
应用层
    │
    ▼
InputManager API
    │
    ▼
InputManagerImpl 方法
    │
    ▼
MultimodalInputConnectProxy::SendRequest()
    │
    ▼
MessageParcel 打包
    │
    ▼
IRemoteProxy::SendRequest()
    │
    ▼
Binder Driver (内核)
```

### 7.3.2 请求处理 (服务端)

```
Binder Driver (内核)
    │
    ▼
MultimodalInputConnectStub::OnRemoteRequest()
    │
    ▼
MMIService::OnRemoteRequest()
    │
    ▼
业务处理
    │
    ▼
MessageParcel 响应打包
    │
    ▼
返回给客户端
```

---

## 7.4 事件处理调用链

```
硬件事件 (libinput)
    │
    ▼
LibinputAdapter::OnEvent()
    │
    ▼
InputEventHandler::OnEvent()
    │
    ├── EventNormalizeHandler::HandleEvent()
    │       │
    │       └── libinput 事件 → MMI 事件
    │
    ├── EventFilterHandler::HandleEvent()
    │       │
    │       └── 条件过滤
    │
    ├── EventInterceptorHandler::HandleEvent()
    │       │
    │       └── 拦截判断
    │
    ├── EventDispatchHandler::HandleEvent()
    │       │
    │       └── UDS 发送
    │
    └── 返回处理结果
```

---

## 7.5 跨设备协作调用链

```
设备 A 应用 (发起协作)
    │
    ▼
CooperateServer::Enable()
    │
    ├── IPC 通知设备 B
    │
    ├── 建立 Socket 连接
    │
    └── 返回协作会话 ID
         │
         ▼
设备 B 收到协作请求
    │
    ├── 用户确认
    │
    ├── 建立双向通道
    │
    └── 事件同步开始
         │
         ▼
协作事件转发
    │
    ├── 鼠标跨屏
    │
    ├── 键盘跨屏
    │
    └── 剪贴板共享
```
