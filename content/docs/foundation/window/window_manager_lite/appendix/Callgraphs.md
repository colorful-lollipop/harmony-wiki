# 附录：调用链图谱

## 窗口创建调用链

### 客户端调用链

```
App (UI Framework)
    │
    ▼
IWindowsManager::GetInstance()
    │
    ▼
LiteProxyWindowsManager::Init()
    │
    ▼
IWindowsManager::CreateWindow(config)
    │
    ▼
LiteProxyWindowsManager::CreateWindow(config)
    │
    ▼
new LiteProxyWindow(LiteWMRequestor::CreateWindow(config))
    │
    ▼
LiteProxyWindow::Init()
    │
    ▼
LiteProxyWindow 构造完成，返回 IWindow*
```

### IPC 调用链

```
LiteProxyWindow (App 进程)
    │
    ▼
LiteWinRequestor 构造
    │
    ▼
LiteWinRequestor::GenericSurface(reply)
    │
    ▼
IPC Framework: SendRequest()
    │
    ▼ IPC 跨进程
    ▼
LiteWMS::Invoke(funcId, origin, req, reply)
    │
    ▼
LiteWMS::WMSRequestHandle(funcId, req, reply)
    │
    ├── case LiteWMS_CreateWindow:
    │   │
    │   ▼
    │   LiteWMS::CreateWindow(req, reply)
    │       │
    │       ▼
    │       GetCallingPid()
    │       │
    │       ▼
    │       LiteWM::CreateWindow(config, pid)
    │           │
    │           ├── CheckWinIdIsAvailable()
    │           ├── new LiteWindow(config)
    │           ├── CreateSurface()
    │           └── PushFront to winList_
    │
    └── WriteInt32(reply, windowId)
```

## 窗口操作调用链

### Show 操作

```
LiteProxyWindow::Show()
    │
    ▼
LiteWinRequestor::Show()
    │
    ▼
IPC: SendRequest(LiteWMS_Show, id)
    │
    ▼
LiteWMS::Show(req, reply)
    │
    ▼
LiteWM::Show(id)
    │
    ├── GetWindowById(id)
    ├── SetIsShow(true)
    └── UpdateWindowRegion(window, rect)
```

### Resize 操作

```
LiteProxyWindow::Resize(width, height)
    │
    ▼
LiteWinRequestor::Resize(width, height)
    │
    ▼
IPC: SendRequest(LiteWMS_Resize, id, width, height)
    │
    ▼
LiteWMS::Resize(req, reply)
    │
    ▼
LiteWM::Resize(id, width, height)
    │
    ├── GetWindowById(id)
    ├── window->Resize(width, height)
    └── UpdateWindowRegion(window, rect)
```

### GetSurface 操作

```
LiteProxyWindow::GetSurface()
    │
    ▼
LiteProxyWindow::surface_ (已缓存)
    │
    └── 返回 LiteProxySurface
```

## 输入事件分发调用链

```
RawEvent (HDI 驱动)
    │
    ▼
InputEventHub::EventCallback(pkgs, count, devIndex)
    │
    ├── 转换为 RawEvent
    └── 放入 eventQueue_
    │
    ▼
InputManagerService::Distribute() [独立线程]
    │
    ├── 从 eventQueue_ 读取
    └── InputEventDistributer::Distribute(events, size)
            │
            └── LiteWM::OnRawEvent(rawEvent)
                    │
                    ├── GetLayerRotateType() [可选旋转]
                    ├── SetMousePosition() [鼠标事件]
                    └── FindTargetWindow(event)
                            │
                            ├── 查找模态窗口
                            └── 坐标/事件类型匹配
                                    │
                                    └── SetEventData(window, event)
```

## 截图操作调用链

```
App 调用 Screenshot()
    │
    ▼
LiteProxyWindowsManager::Screenshot()
    │
    ▼
IPC: SendRequest(LiteWMS_Screenshot)
    │
    ▼
LiteWMS::Screenshot(req, reply)
    │
    ├── GetCallingUid()
    ├── CheckPermission(uid, "ohos.permission.WRITE_MEDIA_IMAGES")
    │   ├── 失败 → WriteInt32(LiteWMS_EUNKNOWN) → 返回
    │   └── 成功 → 继续
    │
    ├── SurfaceImpl::GenericSurfaceByIpcIo(*req)
    │
    └── LiteWM::OnScreenshot(surface)
            │
            └── MainTaskHandler() 中执行
                    │
                    ├── needScreenshot_ = true
                    └── ProcessUpdates() 中执行
                            │
                            ├── 遍历窗口
                            ├── memcpy_s(dstBuffer, srcLayerData)
                            └── FlushBuffer()
```

## 服务初始化调用链

### WMS 服务初始化

```
系统启动
    │
    ▼
SYSEX_SERVICE_INIT(Init)
    │
    ▼
samgr_wms.cpp:Init()
    │
    ├── SAMGR_GetInstance()->RegisterService(&g_example)
    │   │
    │   └── 注册 WMS 服务
    │       ├── GetName() → "WMS"
    │       ├── Initialize() → 设置 identity
    │       ├── MessageHandle()
    │       ├── GetTaskConfig() → {LEVEL_HIGH, ...}
    │       └── Invoke() → WMSRequestHandle
    │
    └── SAMGR_GetInstance()->RegisterDefaultFeatureApi("WMS", ...)
```

### IMS 服务初始化

```
系统启动
    │
    ▼
SYSEX_SERVICE_INIT(Init)
    │
    ▼
samgr_ims.cpp:Init()
    │
    ├── SAMGR_GetInstance()->RegisterService(&g_example)
    │   │
    │   └── 注册 IMS 服务
    │
    └── SAMGR_GetInstance()->RegisterDefaultFeatureApi("IMS", ...)
```

### LiteWM 初始化

```
LiteWM 构造
    │
    ├── InitMutex() → 初始化互斥锁
    │
    ├── InputManagerService::GetInstance()
    │   └── GetDistributer()->AddRawEventListener(this)
    │
    ├── InitMouseCursor()
    │
    └── GetDevSurfaceData() → layerData_
```

## 窗口销毁调用链

```
App 调用 window->Destroy()
    │
    ▼
LiteProxyWindow::Destroy()
    │
    ▼
LiteWinRequestor::~LiteWinRequestor()
    │
    ▼
IPC: SendRequest(LiteWMS_RemoveWindow, id)
    │
    ▼
LiteWMS::RemoveWindow(req, reply)
    │
    ▼
LiteWM::RemoveWindow(id)
    │
    ├── GetWindowNodeById(id)
    ├── winList_.Remove(node)
    ├── AddUpdateRegion(rect)
    └── delete window
```

## 客户端死亡通知

```
客户端进程崩溃
    │
    ▼
IPC Framework 检测到死亡
    │
    ▼
DeathCallback(arg)
    │
    ├── arg->pid = 崩溃客户端 PID
    │
    └── LiteWM::OnClientDeathNotify(pid)
            │
            ├── 遍历 winList_
            ├── 匹配 PID
            └── RemoveWindow() + delete
```

## 关键数据结构传递

### LiteWinConfig 传递

```
App (LiteWinConfig)
    │
    ▼ IPC IpcIo 序列化
    │
    ▼
LiteWMS::CreateWindow(req)
    │
    ├── ReadRawData(req, sizeof(LiteWinConfig))
    │
    └── LiteWM::CreateWindow(config, pid)
```

### Surface 传递

```
Surface 对象 (lite_wm.cpp)
    │
    ├── surface_ (当前 Buffer)
    ├── backBuf_ (后备 Buffer)
    │
    └── IPC: IpcObjectStub 传递
            ├── Handle: IPC_INVALID_HANDLE
            ├── Token: SERVICE_TYPE_ANONYMOUS
            └── Cookie: objectStub 指针
```

### RawEvent 传递

```
InputEventHub (RawEvent)
    │
    ├── eventQueue_ [队列]
    │
    └── InputManagerService::Distribute()
            │
            └── InputEventDistributer::Distribute()
                    │
                    └── LiteWM::OnRawEvent(event)
```
