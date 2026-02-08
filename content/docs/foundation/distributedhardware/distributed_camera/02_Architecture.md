# 架构说明

## 整体架构图

```mermaid
graph TB
    subgraph "应用层 (不可信)"
        APP["HAP 应用"]
    end

    subgraph "分布式硬件框架"
        DHFWK["distributed_hardware_fwk"]
    end

    subgraph "分布式相机服务 (dcamera 进程)"
        subgraph "SDK 层"
            SOURCE_SDK["Source SDK\n(camera_source)"]
            SINK_SDK["Sink SDK\n(camera_sink)"]
        end

        subgraph "SA 服务层"
            SOURCE_SA["Source SA (4803)\n分布式相机源端服务"]
            SINK_SA["Sink SA (4804)\n分布式相机被控端服务"]
        end

        subgraph "内部模块"
            CHANNEL["Channel\n软总线通道"]
            DATA_PROC["Data Process\n数据处理"]
            CAM_OP["Camera Operator\n相机操作"]
        end
    end

    subgraph "底层依赖"
        CAMERA_FW["Camera Framework"]
        SOFTBUS["SoftBus"]
        HDF["HDF Layer"]
    end

    APP --> DHFWK
    DHFWK --> SOURCE_SDK
    DHFWK --> SINK_SDK
    SOURCE_SDK --> SOURCE_SA
    SINK_SDK --> SINK_SA
    SOURCE_SA --> CHANNEL
    SINK_SA --> CHANNEL
    CHANNEL <--> SOFTBUS
    SOURCE_SA --> DATA_PROC
    SINK_SA --> DATA_PROC
    DATA_PROC --> CAM_OP
    CAM_OP --> CAMERA_FW
    SOURCE_SA --> HDF
```

## 数据流

### 主控端 → 被控端 (控制流)

```mermaid
sequenceDiagram
    participant App as 应用层
    participant FW as DHFWK
    participant SrcSDK as Source SDK
    participant SrcSA as Source SA
    participant Channel as Channel
    participant SinkSA as Sink SA
    participant Op as CameraOp

    App->>FW: 启用分布式相机
    FW->>SrcSDK: InitSource()
    SrcSDK->>SrcSA: IPC 调用
    SrcSA->>SrcSA: 权限校验
    SrcSA->>Channel: 建立会话
    Channel->>SinkSA: 软总线连接
    SinkSA->>Op: 打开本地相机
    Op-->>SinkSA: 返回流数据
    SinkSA->>Channel: 发送图像
    Channel-->>SrcSA: 接收图像
    SrcSA->>SrcSA: 数据处理
    SrcSA-->>FW: 返回流
    FW-->>App: 预览/拍照/录像
```

### 被控端 → 主控端 (数据流)

```mermaid
sequenceDiagram
    participant Op as CameraOp
    participant SinkSA as Sink SA
    participant Channel as Channel
    participant SrcSA as Source SA
    participant DataProc as DataProcess

    Op->>SinkSA: 采集预览帧
    SinkSA->>SinkSA: 数据封装
    SinkSA->>Channel: 发送数据
    Channel->>SrcSA: 接收数据
    SrcSA->>DataProc: 处理数据
    DataProc->>DataProc: 解码/缩放/格式转换
    DataProc-->>SrcSA: 处理后数据
    SrcSA-->>FW: 输出流
```

---

## 线程模型

### Source Service 线程模型

```
┌─────────────────────────────────────────────────────────────────┐
│                        Source Service                            │
├─────────────────────────────────────────────────────────────────┤
│  主线程 (EventHandler)                                          │
│  ├── SA 生命周期管理 (OnStart/OnStop)                           │
│  ├── IPC 请求分发 (OnRemoteRequest)                             │
│  └── 状态机事件处理                                              │
├─────────────────────────────────────────────────────────────────┤
│  IPC 调用线程 (来自 Proxy 的调用线程)                             │
│  └── 直接在调用线程执行 Inner 方法                               │
├─────────────────────────────────────────────────────────────────┤
│  数据处理线程 (FFRT Task)                                       │
│  ├── 视频帧解码                                                  │
│  ├── 图像缩放                                                    │
│  └── 色彩空间转换                                                │
├─────────────────────────────────────────────────────────────────┤
│  回调通知线程 (业务回调)                                          │
│  └── 在注册的 EventHandler 上 post 任务                         │
└─────────────────────────────────────────────────────────────────┘
```

### Sink Service 线程模型

```
┌─────────────────────────────────────────────────────────────────┐
│                        Sink Service                              │
├─────────────────────────────────────────────────────────────────┤
│  主线程 (EventHandler)                                          │
│  ├── SA 生命周期管理                                             │
│  ├── IPC 请求分发                                               │
│  └── 访问控制监听                                               │
├─────────────────────────────────────────────────────────────────┤
│  数据发送线程 (FFRT Task)                                       │
│  └── 编码后数据通过 Channel 发送                                  │
├─────────────────────────────────────────────────────────────────┤
│  相机回调线程 (Camera Framework)                                  │
│  └── 预览/拍照/录像回调                                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 状态机模型

### Source 状态机

```
                    ┌────────────────────────────────────────┐
                    │         Init (初始化)                 │
                    └────────────────┬───────────────────────┘
                                     │ InitSource()
                                     ▼
                    ┌────────────────────────────────────────┐
                    │        Regist (已注册)                 │◄─────────────┐
                    │   RegisterDistributedHardware()        │              │
                    └────────────────┬───────────────────────┘              │
                                     │ UnregisterDistributedHardware()      │
                                     ▼                                    │
                    ┌────────────────────────────────────────┐              │
                    │         Open (已打开)                   │              │
                    │       OpenDistributedCamera()          │              │
                    └────────────────┬───────────────────────┘              │
                                     │ CloseDistributedCamera()            │
                                     ▼                                    │
                    ┌────────────────────────────────────────┐              │
                    │      ConfigStream (已配置流)            │──────────────┘
                    │      ConfigDistributedStreams()         │
                    └────────────────┬───────────────────────┘
                                     │ ReleaseSource()
                                     ▼
                    ┌────────────────────────────────────────┐
                    │         Release (已释放)                 │
                    └────────────────────────────────────────┘
```

### Sink 状态机

```
                    ┌────────────────────────────────────────┐
                    │         Init (初始化)                   │
                    └────────────────┬───────────────────────┘
                                     │ InitSink()
                                     ▼
                    ┌────────────────────────────────────────┐
                    │        Ready (就绪)                     │◄─────────────┐
                    │  SubscribeLocalHardware()               │              │
                    └────────────────┬───────────────────────┘              │
                                     │ UnsubscribeLocalHardware()          │
                                     ▼                                    │
                    ┌────────────────────────────────────────┐              │
                    │         Open (已打开)                   │──────────────┤
                    │       OpenChannel()                    │              │
                    └────────────────┬───────────────────────┘              │
                                     │ CloseChannel()                      │
                                     ▼                                    │
                    ┌────────────────────────────────────────┐              │
                    │      Capture (采集中)                   │──────────────┘
                    │       StartCapture()                   │
                    └────────────────┬───────────────────────┘
                                     │ StopCapture() / ReleaseSink()
                                     ▼
                    ┌────────────────────────────────────────┐
                    │         Release (已释放)                 │
                    └────────────────────────────────────────┘
```

---

## 时序图：设备上线流程

```mermaid
sequenceDiagram
    participant DM as DeviceManager
    participant FW as DHFWK
    participant SrcSA as Source SA
    participant HDF as HDF Layer
    participant CamFW as Camera FW

    DM->>FW: 设备上线通知
    FW->>SrcSA: RegisterDistributedHardware()
    SrcSA->>SrcSA: 权限校验
    SrcSA->>HDF: 加载虚拟相机驱动
    HDF->>CamFW: 注册虚拟 Camera
    CamFW-->>HDF: 驱动注册成功
    HDF-->>SrcSA: 回调结果
    SrcSA-->>FW: 返回结果
    FW-->>DM: 使能成功
```

## 时序图：主控端预览流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant FW as DHFWK
    participant SrcSDK as Source SDK
    participant SrcSA as Source SA
    participant Channel as Channel
    participant SinkSA as Sink SA
    participant Op as CameraOp
    participant CamFW as Camera FW

    App->>FW: GetDistributedCameras()
    FW->>SrcSDK: GetSourceHardwareHandler()
    SrcSDK->>SrcSA: IPC: RegisterDistributedHardware()
    
    SrcSA->>Channel: OpenSession()
    Channel->>SinkSA: 建立连接
    SinkSA->>Op: SubscribeLocalHardware()
    Op->>CamFW: 打开相机
    
    loop 预览帧
        CamFW->>Op: OnFrameAvailable()
        Op->>SinkSA: 发送帧数据
        SinkSA->>Channel: SendData()
        Channel-->>SrcSA: 接收数据
        SrcSA->>SrcSA: 解码/缩放
        SrcSA-->>FW: 输出预览流
        FW-->>App: 预览回调
    end
```

---

## 模块协作模式

### 1. IPC 调用模式

```
Client (Proxy)                          Server (Stub)
    │                                      │
    │───── IPC Request ──────────────────►│
    │     (binder transact)               │
    │                                      │
    │                                      │─► Validate Token
    │                                      │─► Check Permission
    │                                      │─► Execute Inner()
    │                                      │
    │◄──── IPC Response ───────────────────│
    │     (return value)                   │
```

### 2. 回调模式

```
Server (Callback Proxy)              Client (Callback Stub)
    │                                      │
    │───── OnXXX() ──────────────────────►│
    │                                      │─► Process Event
    │◄──── ACK ───────────────────────────│
```

### 3. 异步事件模式

```
Event Source                         Event Handler
    │                                      │
    │───── Post Task ────────────────────►│
    │     (EventHandler)                  │
    │                                      │─► Process Async
```

---

## 安全边界

| 边界 | 描述 | 防护措施 |
|------|------|----------|
| **IPC 边界** | Proxy ↔ Stub | InterfaceToken 校验 + AccessToken 权限校验 |
| **进程边界** | dcamera 进程 | SELinux (u:r:dcamera:s0) + uid/gid 隔离 |
| **网络边界** | 软总线传输 | 数据包校验 + 传输层加密 |
| **权限边界** | API 访问 | ENABLE_DISTRIBUTED_HARDWARE / ACCESS_DISTRIBUTED_HARDWARE |
