# 01 - 架构说明

> Camera_Lite 组件架构详解

---

## 系统架构图

```mermaid
graph TB
    subgraph Application["应用层"]
        App[相机应用]
    end
    
    subgraph Framework["Framework层 (Client)"]
        CK[CameraKit]
        CM[CameraManager]
        CSC[CameraServiceClient]
        CDC[CameraDeviceClient]
        CI[CameraImpl]
        subgraph Callbacks["回调接口"]
            CSCb[CameraStateCallback]
            FSCb[FrameStateCallback]
            CDCb[CameraDeviceCallback]
        end
    end
    
    subgraph IPC["IPC层"]
        direction TB
        Binder[Binder IPC]
        Pass[Passthrough]
    end
    
    subgraph Service["Service层 (Server)"]
        CSrv[CameraServer]
        CSvc[CameraService]
        CD[CameraDevice]
        subgraph Assistants["助手类"]
            RA[RecordAssistant]
            PA[PreviewAssistant]
            CA[CaptureAssistant]
            CBA[CallbackAssistant]
        end
    end
    
    subgraph HAL["HAL层"]
        HCam[HalCamera]
        Codec[CodecInterface]
        Disp[DisplayLayer]
    end
    
    subgraph HW["硬件层"]
        CamHW[相机硬件]
        Venc[视频编码器]
        DispHW[显示硬件]
    end
    
    App --> CK
    CK --> CM
    CM --> CSC
    CM --> CI
    CI --> CDC
    CI --> CSCb
    CI --> FSCb
    CM --> CDCb
    
    CSC --> Binder
    CDC --> Binder
    Binder --> CSrv
    
    CSC -.-> Pass
    CDC -.-> Pass
    Pass -.-> CSvc
    
    CSrv --> CSvc
    CSvc --> CD
    CD --> RA
    CD --> PA
    CD --> CA
    CD --> CBA
    
    CD --> HCam
    RA --> Codec
    PA --> Disp
    CA --> Codec
    CBA --> HCam
    
    HCam --> CamHW
    Codec --> Venc
    Disp --> DispHW
```

---

## 运行模式

### Binder模式

**适用场景**: 多进程架构，客户端和服务端运行在不同进程

**工作流程**:

```mermaid
sequenceDiagram
    participant App as 应用
    participant Client as CameraServiceClient
    participant IPC as IPC Proxy
    participant Server as CameraServer
    participant Service as CameraService
    
    App->>+Client: GetCameraAbility(id)
    Client->>IPC: Invoke(GET_CAMERA_ABILITY)
    IPC->>Server: 跨进程调用
    Server->>Service: GetCameraAbility(id)
    Service-->>Server: CameraAbility*
    Server-->>IPC: Write reply
    IPC-->>Client: Callback with data
    Client-->>App: CameraAbility*
```

**关键代码**:
- 客户端: `frameworks/binder/src/camera_service_client.cpp`
- 服务端: `services/server/src/camera_server.cpp`
- IPC类型定义: `services/server/include/camera_type.h`

### Passthrough模式

**适用场景**: 单进程架构，追求更高性能

**工作流程**:

```mermaid
sequenceDiagram
    participant App as 应用
    participant Client as CameraServiceClient
    participant Service as CameraService
    
    App->>+Client: GetCameraAbility(id)
    Client->>Service: 直接调用
    Service-->>Client: CameraAbility*
    Client-->>App: CameraAbility*
```

**编译切换**:

在 `frameworks/BUILD.gn` 中通过条件控制:

```gn
if (enable_media_passthrough_mode == true) {
  # Passthrough模式: 包含服务和passthrough客户端
  sources += [
    "../services/impl/src/camera_device.cpp",
    "../services/impl/src/camera_service.cpp",
    "passthrough/src/camera_device_client.cpp",
    "passthrough/src/camera_service_client.cpp",
  ]
  defines = [ "ENABLE_PASSTHROUGH_MODE" ]
} else {
  # Binder模式: 仅binder客户端
  sources += [
    "binder/src/camera_device_client.cpp",
    "binder/src/camera_service_client.cpp",
  ]
}
```

---

## 类关系图

### 客户端类层次

```mermaid
classDiagram
    class CameraKit {
        +GetInstance() CameraKit*
        +GetCameraIds() list~string~
        +GetCameraAbility(string) CameraAbility*
        +CreateCamera(string, callback, handler)
        -CameraManager* cameraManager_
    }
    
    class CameraManager {
        <<abstract>>
        +GetInstance() CameraManager*
        +GetCameraIds()* list~string~
        +CreateCamera()*
    }
    
    class CameraManagerImpl {
        -CameraServiceClient* cameraServiceClient_
        -map~string,CameraImpl*~ cameraMapCache_
        -list~pair~callback,handler~~ deviceCbList_
    }
    
    class Camera {
        <<interface>>
        +GetCameraId()* string
        +Configure(config)*
        +TriggerLoopingCapture(fc)* int32_t
        +TriggerSingleCapture(fc)* int32_t
        +StopLoopingCapture(type)*
        +Release()*
    }
    
    class CameraImpl {
        -string id_
        -CameraAbility* ability_
        -CameraInfo* info_
        -CameraConfig* config_
        -CameraDeviceClient* deviceClient_
        +OnCreate(cameraId)
        +OnFrameFinished(ret, fc)
    }
    
    class CameraServiceClient {
        +GetInstance() CameraServiceClient*
        +InitCameraServiceClient(callback)
        +GetCameraIdList() list~string~
        +GetCameraAbility(string) CameraAbility*
        +CreateCamera(string)
    }
    
    class CameraDeviceClient {
        +GetInstance() CameraDeviceClient*
        +SetCameraConfig(cc) int32_t
        +TriggerLoopingCapture(fc) int32_t
        +TriggerSingleCapture(fc) int32_t
        +StopLoopingCapture(type)
        +Release()
    }
    
    CameraKit --> CameraManager
    CameraManager <|-- CameraManagerImpl
    CameraManagerImpl --> CameraServiceClient
    CameraManagerImpl --> CameraImpl
    CameraImpl --> CameraDeviceClient
    Camera <|-- CameraImpl
```

### 服务端类层次

```mermaid
classDiagram
    class CameraServer {
        +GetInstance() CameraServer*
        +CameraServerRequestHandle(funcId, origin, req, reply)
        +GetCameraAbility(req, reply)
        +CreateCamera(req, reply)
        -SvcIdentity sid_
    }
    
    class CameraService {
        +GetInstance() CameraService*
        +Initialize()
        +GetCameraAbility(string) CameraAbility*
        +CreateCamera(string) int32_t
        +GetCameraDevice(string) CameraDevice*
        -map~string,CameraDevice*~ deviceMap_
    }
    
    class CameraDevice {
        +Initialize() int32_t
        +TriggerLoopingCapture(fc, streamId) int32_t
        +TriggerSingleCapture(fc, streamId) int32_t
        +StopLoopingCapture(type)
        -RecordAssistant recordAssistant_
        -PreviewAssistant previewAssistant_
        -CaptureAssistant captureAssistant_
        -CallbackAssistant callbackAssistant_
    }
    
    class RecordAssistant {
        +SetFrameConfig(fc, streamId) int32_t
        +Start(streamId) int32_t
        +Stop() int32_t
    }
    
    class PreviewAssistant {
        +SetFrameConfig(fc, streamId) int32_t
        +Start(streamId) int32_t
        +Stop() int32_t
    }
    
    class CaptureAssistant {
        +SetFrameConfig(fc, streamId) int32_t
        +Start(streamId) int32_t
        +Stop() int32_t
    }
    
    class CallbackAssistant {
        +SetFrameConfig(fc, streamId) int32_t
        +Start(streamId) int32_t
        +Stop() int32_t
    }
    
    CameraServer --> CameraService
    CameraService --> CameraDevice
    CameraDevice --> RecordAssistant
    CameraDevice --> PreviewAssistant
    CameraDevice --> CaptureAssistant
    CameraDevice --> CallbackAssistant
```

---

## 数据流图

### 预览数据流 (Preview)

```mermaid
flowchart LR
    A[相机硬件] -->|原始数据| B[HAL层]
    B -->|YUV| C[显示层]
    C -->|渲染| D[屏幕]
```

**代码位置**: `services/impl/src/camera_device.cpp` 的 PreviewAssistant

### 录像数据流 (Record)

```mermaid
flowchart LR
    A[相机硬件] -->|原始数据| B[HAL层]
    B -->|YUV| C[视频编码器]
    C -->|H.264/H.265| D[Surface]
    D -->|消费| E[文件系统/网络]
```

**关键流程**:
1. `RecordAssistant::SetFrameConfig()` - 配置编码器
2. `RecordAssistant::Start()` - 启动编码
3. `OnVencBufferAvailble()` - 编码数据回调
4. `CopyCodecOutput()` - 数据拷贝到Surface

**代码位置**: `services/impl/src/camera_device.cpp` 第339-389行

### 拍照数据流 (Capture)

```mermaid
flowchart LR
    A[相机硬件] -->|原始数据| B[HAL层]
    B -->|YUV| C[Codec编码]
    C -->|JPEG/HEVC| D[Surface]
    D -->|消费| E[应用]
```

**代码位置**: `services/impl/src/camera_device.cpp` 的 CaptureAssistant::Start()

### 回调数据流 (Callback)

```mermaid
flowchart LR
    A[相机硬件] -->|原始数据| B[HAL层]
    B -->|YUV| C[CallbackAssistant]
    C -->|拷贝线程| D[Surface]
    D -->|消费| E[应用回调]
```

**关键流程**:
1. `StreamCopyProcess()` - 独立线程循环读取HAL buffer
2. `HalCameraDequeueBuf()` - 获取HAL层buffer
3. `FlushBuffer()` - 提交到Surface

**代码位置**: `services/impl/src/camera_device.cpp` 第753-805行

---

## 线程模型

```mermaid
graph TB
    subgraph MainThread["主线程"]
        API[API调用处理]
        IPC[IPC通信]
        CB[回调分发]
    end
    
    subgraph PreviewThread["预览线程 (可选)"]
        PT[YuvCopyProcess]
    end
    
    subgraph CallbackThread["回调线程"]
        CT[StreamCopyProcess]
    end
    
    subgraph CodecThreads["编解码线程 (Codec内部)"]
        VencT[视频编码线程]
    end
    
    API --> IPC
    IPC --> CB
    
    PT -.->|YUV数据| API
    CT -.->|原始数据| CB
    VencT -.->|编码数据| CB
```

### 线程职责

| 线程 | 职责 | 代码位置 |
|------|------|----------|
| 主线程 | API调用、IPC通信、回调处理 | 框架入口 |
| 预览线程 | YUV数据拷贝 (预留) | `camera_device.cpp:531` |
| 回调线程 | 原始帧数据拷贝 | `camera_device.cpp:753` |
| 编码线程 | 视频/图像编码 | Codec模块内部 |

**注意事项**:
- 回调通过 `EventHandler::Post()` 切换到主线程执行
- `StreamCopyProcess` 使用 `usleep(DELAY_TIME_ONE_FRAME)` 控制帧率

---

## 回调机制

### 异步回调流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant IPC as IPC层
    participant Server as CameraServer
    participant Handler as EventHandler
    
    Note over Client,Handler: 注册阶段
    Client->>IPC: CreateCamera (携带SvcIdentity)
    IPC->>Server: 传递回调对象
    Server->>Server: 保存 sid_
    
    Note over Client,Handler: 事件触发
    Server->>Server: 事件发生(如配置完成)
    Server->>IPC: SendRequest(sid_, ON_CAMERA_CONFIGURED)
    IPC->>Client: 回调 ClientCallback
    Client->>Handler: Post(回调Lambda)
    Handler->>Client: 执行回调(on线程)
```

### 回调类型对照表

| 回调类型 | 触发时机 | 处理位置 |
|----------|----------|----------|
| OnCreated | 相机创建成功 | `CameraImpl::OnCreate()` |
| OnCreateFailed | 相机创建失败 | `CameraImpl::OnCreateFailed()` |
| OnConfigured | 相机配置成功 | `CameraImpl::OnConfigured()` |
| OnReleased | 相机已释放 | `CameraImpl::Release()` |
| OnFrameFinished | 帧捕获完成 | `CameraImpl::OnFrameFinished()` |
| OnFrameError | 帧捕获错误 | `CameraImpl::OnFrameFinished()` |
| OnCameraStatus | 设备状态变化 | `CameraManagerImpl::OnCameraStatusChange()` |

---

## 状态机

### 助手类状态 (DeviceAssistant)

```
LOOP_IDLE → LOOP_READY → LOOP_LOOPING → LOOP_STOP
                ↑___________________________|
```

| 状态 | 说明 |
|------|------|
| LOOP_IDLE | 初始状态 |
| LOOP_READY | 配置完成，可启动 |
| LOOP_LOOPING | 运行中 |
| LOOP_STOP | 已停止 |
| LOOP_ERROR | 错误状态 |

**状态检查**:
```cpp
if (assistant->state_ == LOOP_IDLE || 
    assistant->state_ == LOOP_LOOPING || 
    assistant->state_ == LOOP_ERROR) {
    MEDIA_ERR_LOG("Device state is %d, cannot start looping capture.", assistant->state_);
    return MEDIA_ERR;
}
```

**证据**: `services/impl/src/camera_device.cpp:877-880`

---

## 关键时序

### 相机生命周期

```mermaid
sequenceDiagram
    participant App as 应用
    participant Kit as CameraKit
    participant Mgr as CameraManager
    participant Srv as CameraService
    participant Dev as CameraDevice
    
    App->>Kit: GetInstance()
    Kit->>Kit: CheckPermission()
    Kit-->>App: CameraKit*
    
    App->>Kit: GetCameraIds()
    Kit->>Mgr: GetCameraIds()
    Mgr->>Srv: GetCameraIdList()
    Srv->>Mgr: list<string>
    Mgr-->>App: list<string>
    
    App->>Kit: CreateCamera(id, callback, handler)
    Kit->>Mgr: CreateCamera()
    Mgr->>Srv: CreateCamera(id)
    Srv->>Srv: HalCameraDeviceOpen()
    Srv->>Dev: new CameraDevice()
    Srv->>Srv: OnCameraStatusChange(CREATED)
    Srv->>Mgr: 回调 OnCameraStatusChange
    Mgr->>Mgr: OnCreated()
    Mgr->>App: 回调 OnCreated(Camera)
    
    App->>App: Configure(config)
    App->>App: TriggerLoopingCapture(fc)
    App->>App: ... 使用相机 ...
    
    App->>App: Release()
    Dev->>Srv: StopLoopingCapture()
    Srv->>Srv: HalCameraDeviceClose()
```

---

## 下一步

- [API参考](02_API_Reference.md) - 查看完整API定义
- [内部接口](03_Inner_API.md) - 了解模块内部设计
- [安全风险](05_Security.md) - 审查安全问题
