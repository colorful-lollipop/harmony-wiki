# 附录: 关键调用链

## 目的

本文档提供 Pasteboard 关键功能的调用链图，帮助理解代码执行流程。

## SetPasteData 调用链

```mermaid
sequenceDiagram
    participant App as JS Application
    participant NAPI as N-API Layer
    participant Client as PasteboardClient
    participant Loader as ServiceLoader
    participant Proxy as Service Proxy
    participant IPC as IPC/Binder
    participant Stub as Service Stub
    participant Service as PasteboardService
    participant Storage as Data Storage

    Note over App,Storage: === 1. JS API 调用 ===
    App->>NAPI: setData(pasteData)
    
    Note over NAPI: === 2. N-API 处理 ===
    NAPI->>NAPI: napi_get_cb_info() 获取参数
    NAPI->>NAPI: GetValue() 类型转换
    NAPI->>NAPI: PasteDataNapi::NewInstance()
    NAPI->>Client: PasteboardClient::GetInstance()
    NAPI->>Client: PasteboardClient::SetPasteData()
    
    Note over Client: === 3. 客户端处理 ===
    Client->>Client: GetPasteboardService()
    Client->>Loader: ServiceLoader::GetPasteboardServiceProxy()
    Loader->>Loader: LoadSystemAbility(SA_ID: 3701)
    Loader-->>Client: sptr<IPasteboardService>
    
    Client->>Client: Validate pasteData
    Client->>Client: Serialize to TLV
    Client->>Proxy: proxy->SetPasteData(data)
    
    Note over Proxy,IPC: === 4. IPC 传输 ===
    Proxy->>IPC: MessageParcel write
    Proxy->>IPC: SendRequest(SET_PASTE_DATA: 104)
    IPC->>Stub: OnRemoteRequest()
    
    Note over Stub: === 5. 服务端接收 ===
    Stub->>Stub: ReadInterfaceToken()
    Stub->>Stub: switch (code) case SET_PASTE_DATA
    Stub->>Service: PasteboardService::OnSetPasteData()
    
    Note over Service: === 6. 服务处理 ===
    Service->>Service: GetCallingTokenID()
    Service->>Service: VerifyPermission()
    Service->>Service: CheckShareOption()
    Service->>Service: Deserialize TLV
    Service->>Storage: Store pasteData
    Service->>Service: NotifyObservers()
    Service->>Service: ReportUEEvent()
    
    Note over Service,App: === 7. 返回结果 ===
    Service-->>Stub: Return int32_t
    Stub-->>IPC: Write result
    IPC-->>Proxy: Read result
    Proxy-->>Client: 
    Client-->>NAPI: 
    NAPI->>NAPI: Create Promise result
    NAPI-->>App: Promise resolved
```

**代码参考**:
- `interfaces/kits/napi/src/napi_systempasteboard.cpp:500`
- `framework/innerkits/src/pasteboard_client.cpp:300`
- `services/core/src/pasteboard_service.cpp:1267`

## GetPasteData 调用链

```mermaid
sequenceDiagram
    participant App as JS Application
    participant NAPI as N-API Layer
    participant Client as PasteboardClient
    participant Proxy as Service Proxy
    participant IPC as IPC/Binder
    participant Stub as Service Stub
    participant Service as PasteboardService
    participant Storage as Data Storage

    Note over App,Storage: === 1. JS API 调用 ===
    App->>NAPI: getData()
    
    Note over NAPI: === 2. N-API 处理 ===
    NAPI->>NAPI: AsyncCall::Call()
    NAPI->>Client: PasteboardClient::GetPasteData()
    
    Note over Client: === 3. 客户端处理 ===
    Client->>Client: GetPasteboardService()
    Client->>Client: Build Request
    Client->>Proxy: proxy->GetPasteData()
    
    Note over Proxy,IPC: === 4. IPC 传输 ===
    Proxy->>IPC: MessageParcel write
    Proxy->>IPC: SendRequest(GET_PASTE_DATA: 200)
    IPC->>Stub: OnRemoteRequest()
    
    Note over Stub: === 5. 服务端接收 ===
    Stub->>Stub: ReadInterfaceToken()
    Stub->>Stub: switch (code) case GET_PASTE_DATA
    Stub->>Service: PasteboardService::OnGetPasteData()
    
    Note over Service: === 6. 服务处理 ===
    Service->>Service: GetCallingTokenID()
    Service->>Service: GetCallingPid()
    Service->>Service: VerifyPermission()
    Service->>Service: CheckShareOption()
    Service->>Service: GetAppInfo()
    Service->>Storage: Retrieve data
    Service->>Service: GrantUriPermission()
    Service->>Service: Marshal response (TLV)
    
    Note over Service,App: === 7. 返回结果 ===
    Service-->>Stub: Return PasteData
    Stub-->>IPC: Write result (Ashmem/TLV)
    IPC-->>Proxy: Read result
    Proxy-->>Client: 
    Client->>Client: Deserialize TLV
    Client-->>NAPI: Return PasteData
    NAPI->>NAPI: Create PasteDataNapi object
    NAPI-->>App: Promise resolved with data
```

**代码参考**:
- `interfaces/kits/napi/src/napi_systempasteboard.cpp:300`
- `framework/innerkits/src/pasteboard_client.cpp:400`
- `services/core/src/pasteboard_service.cpp:831`

## 服务启动调用链

```mermaid
sequenceDiagram
    participant Init as Init Process
    participant SAMgr as SystemAbilityManager
    participant Loader as Loader
    participant PBS as PasteboardService
    participant Account as AccountManager
    participant DFX as DFX Reporter

    Note over Init,DFX: === 1. 系统初始化 ===
    Init->>Init: parse pasteboardservice.cfg
    Init->>SAMgr: Start SA 3701
    
    Note over SAMgr: === 2. SA 管理器 ===
    SAMgr->>SAMgr: Query SA profile (3701.json)
    SAMgr->>Loader: Load library
    
    Note over Loader: === 3. 动态加载 ===
    Loader->>Loader: dlopen libpasteboard_service.z.so
    Loader->>PBS: MakeAndRegisterAbility()
    
    Note over PBS: === 4. 服务构造 ===
    PBS->>PBS: PasteboardService()
    PBS->>PBS: SystemAbility(PASTEBOARD_SERVICE_ID: 3701)
    PBS->>SAMgr: Publish(this)
    
    SAMgr->>PBS: OnStart()
    
    Note over PBS: === 5. 服务启动 ===
    PBS->>PBS: SetMaxWorkThreadNum(32)
    PBS->>PBS: InitServiceHandler()
    PBS->>PBS: Loader::LoadUid()
    PBS->>PBS: moduleConfig_.Init()
    PBS->>PBS: Watch config changes
    PBS->>PBS: AddSysAbilityListener()
    PBS->>PBS: InitScreenStatus()
    PBS->>PBS: Init()
    
    Note over PBS: === 6. 子模块初始化 ===
    PBS->>Account: AccountManager::GetInstance()
    PBS->>DFX: Reporter::GetInstance()
    
    PBS->>SAMgr: state_ = STATE_RUNNING
```

**代码参考**:
- `services/core/src/pasteboard_service.cpp:116`
- `services/load/src/loader.cpp`
- `etc/init/pasteboardservice.cfg`

## 数据变化通知调用链

```mermaid
sequenceDiagram
    participant AppA as App A (Writer)
    participant NAPI as N-API Layer
    participant Client as PasteboardClient
    participant Proxy as Service Proxy
    participant IPC as IPC/Binder
    participant Service as PasteboardService
    participant Storage as Data Storage
    participant IPC2 as IPC/Binder
    participant Proxy2 as Observer Proxy
    participant Client2 as PasteboardClient
    participant AppB as App B (Observer)

    Note over AppA,AppB: === 1. 写入数据 ===
    AppA->>NAPI: setData(data)
    NAPI->>Client: SetPasteData()
    Client->>Proxy: IPC: SET_PASTE_DATA
    Proxy->>IPC: SendRequest(104)
    IPC->>Service: OnRemoteRequest()
    Service->>Storage: Store data
    Service->>Service: changeCount_++
    Service->>Service: NotifyObservers()
    
    Note over Service: === 2. 通知观察者 ===
    loop For each observer
        Service->>IPC2: sptr->OnPasteboardChanged()
        IPC2->>Proxy2: IPC callback
        Proxy2->>Client2: OnPasteboardChanged()
        Client2->>Client2: Notify callbacks
        Client2->>AppB: JS callback
    end
```

**代码参考**:
- `services/core/src/pasteboard_service.cpp:2950`
- `framework/innerkits/src/pasteboard_client.cpp:600`

## 分布式同步调用链

```mermaid
sequenceDiagram
    participant AppA as App A (Device A)
    participant ClientA as PasteboardClient A
    participant ServiceA as PasteboardService A
    participant DMA as DeviceManager A
    participant DMB as DeviceManager B
    participant ServiceB as PasteboardService B
    participant ClientB as PasteboardClient B
    participant AppB as App B (Device B)

    Note over AppA,AppB: === 1. 复制到剪贴板 ===
    AppA->>ClientA: setData(data)
    ClientA->>ServiceA: IPC: SET_PASTE_DATA
    ServiceA->>ServiceA: Mark local data
    ServiceA->>DMA: Register device status
    
    Note over DMA,DMB: === 2. 设备发现 ===
    DMA->>DMB: Device online notification
    DMB->>ServiceB: Update device list
    
    Note over AppB: === 3. 粘贴操作 ===
    AppB->>ClientB: getData()
    ClientB->>ServiceB: IPC: GET_PASTE_DATA
    ServiceB->>ServiceB: Detect remote data
    ServiceB->>DMB: Query source device
    DMB->>DMA: Get device info
    
    Note over ServiceA,ServiceB: === 4. P2P 数据传输 ===
    ServiceB->>ServiceA: Request data (P2P)
    ServiceA->>ServiceA: Check share option
    ServiceA->>ServiceA: Encrypt data
    ServiceA-->>ServiceB: Transfer encrypted data
    ServiceB->>ServiceB: Decrypt data
    ServiceB->>ServiceB: Cache locally
    ServiceB-->>ClientB: Return data
    ClientB-->>AppB: 
```

**代码参考**:
- `framework/framework/device/dm_adapter.cpp`
- `services/core/src/pasteboard_service.cpp:3030`

## TLV 序列化调用链

```mermaid
sequenceDiagram
    participant Data as PasteData
    participant Record as PasteDataRecord
    participant Entry as PasteDataEntry
    participant TLVW as TLVWriter
    participant Parcel as MessageParcel
    participant Ashmem as Ashmem

    Note over Data,Ashmem: === 1. 序列化 ===
    Data->>Data: Marshalling()
    Data->>Record: For each record
    Record->>Record: Marshalling()
    Record->>Entry: For each entry
    Entry->>Entry: Marshalling()
    Entry->>TLVW: TLVWriteable::Write()
    TLVW->>TLVW: Counting()
    TLVW->>TLVW: Encoding()
    TLVW->>Parcel: MessageParcel write
    
    alt Data > 32KB
        Parcel->>Ashmem: Create Ashmem
        Ashmem->>Ashmem: mmap()
        Ashmem->>Ashmem: memcpy_s()
    end
```

**代码参考**:
- `framework/innerkits/src/paste_data.cpp:696`
- `framework/tlv/tlv_writeable.cpp:107`
- `framework/tlv/message_parcel_warp.cpp:86`

## 相关链接

- [架构说明 → 01_Architecture.md](01_Architecture.md)
- [内部 API → 04_Inner_API.md](04_Inner_API.md)
- [N-API 参考 → 03_NAPI_Reference.md](03_NAPI_Reference.md)
