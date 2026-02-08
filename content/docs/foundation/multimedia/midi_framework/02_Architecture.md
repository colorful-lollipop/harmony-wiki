# 架构与数据流 (Architecture)

## 1. 组件架构图

```mermaid
graph TB
    subgraph App["应用层"]
        A[MIDI App]
    end

    subgraph Framework["框架层 (frameworks/native)"]
        B[OHMIDI.cpp]
        C[MidiClientPrivate]
        D[MidiDevicePrivate]
        E[MidiInputPort]
        F[MidiOutputPort]
        G[MidiSharedRing]
    end

    subgraph IPC["IPC 层 (services/idl)"]
        H[IIpcMidiInServer Proxy]
        I[IMidiCallback Stub]
    end

    subgraph Service["服务层 (services/server)"]
        J[MidiServer]
        K[MidiServiceController]
        L[MidiInServer]
        M[MidiDeviceManager]
        N[DeviceClientContext]
    end

    subgraph Driver["驱动层"]
        O[UsbMidiDriver]
        P[BleMidiDriver]
    end

    A -->|Native API| B
    B --> C
    C --> D
    D --> E
    D --> F
    E --> G
    F --> G
    C --> H
    H -->|IPC| J
    J --> K
    K --> L
    K --> M
    K --> N
    M --> O
    M --> P
    L -->|Callback| I
    I --> C
```

---

## 2. 关键交互流程

### 2.1 服务按需启动与生命周期

```mermaid
sequenceDiagram
    participant App as MIDI App
    participant Client as MidiClientPrivate
    participant SAMgr as SystemAbilityManager
    participant Server as MidiServer
    participant Ctrl as MidiServiceController

    App->>Client: OH_MIDIClientCreate()
    Client->>SAMgr: LoadSystemAbility(3014)
    alt 服务未启动
        SAMgr->>Server: 启动 midi_server 进程
        Server->>Server: OnStart()
        Server->>Ctrl: GetInstance()->Init()
        Ctrl->>Ctrl: deviceManager_->Init()
    end
    SAMgr-->>Client: 返回 IRemoteObject
    Client->>Server: CreateMidiInServer(callback)
    Server->>Ctrl: CreateMidiInServer()
    Ctrl->>Ctrl: 生成 clientId
    Ctrl->>Ctrl: AddDeathRecipient()
    Ctrl-->>Client: 返回 clientId
```

**代码位置:**
- `services/server/src/midi_service_controller.cpp:127-159`
- `services/server/src/midi_service_controller.cpp:98-125`

---

### 2.2 USB 设备发现与打开

```mermaid
sequenceDiagram
    participant App as MIDI App
    participant Client as MidiClientPrivate
    participant Server as MidiInServer
    participant Ctrl as MidiServiceController
    participant Mgr as MidiDeviceManager
    participant USB as UsbMidiDriver
    participant HDI as MIDI HDI

    Note over USB,HDI: 设备热插拔
    HDI->>USB: USB 设备插入事件
    USB->>Mgr: 注册设备
    Mgr->>Ctrl: NotifyDeviceChange(ADD)
    Ctrl->>Client: NotifyDeviceChange()
    Client->>App: onDeviceChange callback

    App->>Client: OH_MIDIOpenDevice()
    Client->>Server: OpenDevice(deviceId)
    Server->>Ctrl: OpenDevice(clientId, deviceId)
    Ctrl->>Ctrl: 验证 clientId 存在
    Ctrl->>Ctrl: 创建设备上下文
    Ctrl->>Mgr: OpenDevice(deviceId)
    Mgr->>USB: OpenDevice()
    USB->>HDI: OpenDevice()
    HDI-->>USB: OK
    USB-->>Mgr: OK
    Mgr-->>Ctrl: OK
    Ctrl-->>Client: OK
```

**代码位置:**
- `services/server/src/midi_service_controller.cpp:185-213`
- `services/server/src/midi_device_mananger.cpp`

---

### 2.3 BLE 设备异步打开

```mermaid
sequenceDiagram
    participant App as MIDI App
    participant Client as MidiClientPrivate
    participant Server as MidiInServer
    participant Ctrl as MidiServiceController
    participant Mgr as MidiDeviceManager
    participant BLE as BleMidiDriver
    participant BT as 蓝牙服务

    App->>Client: OH_MIDIOpenBleDevice(addr, callback)
    Client->>Server: OpenBleDevice(address)
    Server->>Ctrl: OpenBleDevice(clientId, address)
    Ctrl->>Ctrl: 检查 pending/active 列表
    Ctrl->>Mgr: OpenBleDevice(address, completeCallback)
    Mgr->>BLE: OpenDevice()
    BLE->>BT: GATT Connect
    BT-->>BLE: 连接成功
    BLE->>BLE: HandleBleOpenComplete()
    BLE->>Mgr: completeCallback(success, deviceId)
    Mgr->>Ctrl: HandleBleOpenComplete()
    Ctrl->>Ctrl: 创建设备上下文
    Ctrl->>Client: NotifyDeviceOpened()
    Client->>App: OH_MIDIOnDeviceOpened callback
```

**代码位置:**
- `services/server/src/midi_service_controller.cpp:215-321`

---

### 2.4 端口数据收发 (共享内存)

```mermaid
sequenceDiagram
    participant App as MIDI App
    participant OutPort as MidiOutputPort
    participant Ring as MidiSharedRing
    participant Conn as DeviceConnectionForOutput
    participant Driver as USB/BLE Driver
    participant Device as MIDI Device

    Note right of App: 发送数据 (App → Device)
    App->>OutPort: OH_MIDISend(events)
    OutPort->>Ring: TryWriteEvents()
    Ring->>Ring: 写入共享内存
    Ring->>Ring: WakeFutex()
    Conn->>Ring: epoll_wait() 唤醒
    Conn->>Ring: PeekNext()
    Conn->>Conn: ProcessMessages()
    Conn->>Driver: HanleUmpInput()
    Driver->>Device: 发送 MIDI 数据

    Note right of App: 接收数据 (Device → App)
    Device->>Driver: MIDI 数据输入
    Driver->>Conn: UmpInputCallback
    Conn->>Ring: TryWriteEvent() [每个客户端]
    Conn->>Ring: NotifyConsumer()
    App->>Ring: WaitFor() [futex]
    Ring-->>App: 唤醒
    App->>Ring: DrainToBatch()
    App->>App: onMIDIReceived callback
```

**代码位置:**
- `services/common/include/midi_shared_ring.h`
- `services/server/include/midi_device_connection.h`

---

## 3. 线程模型

### 3.1 客户端线程

| 线程 | 来源 | 职责 |
|------|------|------|
| **主线程** | App | API 调用、事件处理 |
| **ReceiverThread** | `MidiInputPort` | futex 等待，接收服务端数据 |
| **CallbackThread** | `MidiDevicePrivate` | 分发设备热插拔回调 |

### 3.2 服务端线程

| 线程 | 来源 | 职责 |
|------|------|------|
| **主线程** | `midi_server` | IPC 请求处理 |
| **OutputWorker** | `DeviceConnectionForOutput` | epoll 事件循环，发送数据到设备 |
| **UnloadThread** | `MidiServiceController` | 延迟卸载定时器 |
| **DeathNotify** | IPC 框架 | 客户端死亡通知处理 |

### 3.3 同步机制

```
共享内存同步:
┌─────────────────────────────────────┐
│  ControlHeader (64 bytes)           │
│  ├─ readPosition: atomic<uint32>   │
│  ├─ writePosition: atomic<uint32>  │
│  ├─ capacity: uint32                │
│  └─ futexObj: atomic<uint32>       │
├─────────────────────────────────────┤
│  Ring Buffer Data                   │
│  [ShmMidiEventHeader + payload]...  │
└─────────────────────────────────────┘

写入方: 更新 writePosition → WakeFutex()
读取方: WaitFor(futex) → 读取 → 更新 readPosition
```

**代码位置:** `services/common/include/midi_shared_ring.h:34-52`

---

## 4. 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                    应用进程 (低权限)                          │
│  ┌─────────────┐  ┌─────────────────────────────────────┐   │
│  │  MIDI App   │  │  libohmidi.so (Native Framework)    │   │
│  │  (Sandbox)  │  │  - MidiClientPrivate                │   │
│  └──────┬──────┘  │  - MidiDevicePrivate                │   │
│         │         │  - MidiSharedRing (Client View)     │   │
└─────────┼─────────┴─────────────────────────────────────┘───┘
          │ IPC (Binder)
┌─────────▼───────────────────────────────────────────────────┐
│                  服务进程 (system 权限)                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  midi_server                                          │   │
│  │  - MidiServiceController (全局状态管理)                │   │
│  │  - MidiDeviceManager (设备枚举和驱动管理)              │   │
│  │  - MidiSharedRing (Server View, 内存分配)             │   │
│  │  - 访问 USB/BLE 驱动、/dev/snd/* 设备节点             │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
          │ HDI / HAL
┌─────────▼───────────────────────────────────────────────────┐
│                     内核/驱动层                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  USB Driver  │  │  BLE Driver  │  │  ALSA/HDI    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

**关键边界:**
1. **App ↔ Framework**: Native API 调用
2. **Framework ↔ Service**: IPC Binder 通信
3. **Service ↔ Driver**: HDI 接口 / 蓝牙服务

---

## 5. 关键数据结构

### 5.1 设备信息

```cpp
// interfaces/kits/c/midi/native_midi_base.h:241-274
struct OH_MIDIDeviceInformation {
    int64_t midiDeviceId;           // 设备唯一ID
    OH_MIDIDeviceType deviceType;   // USB=0, BLE=1
    OH_MIDIProtocol nativeProtocol; // PROTOCOL_1_0 / PROTOCOL_2_0
    char productName[256];          // 产品名称
    char vendorName[256];           // 厂商名称
    char deviceAddress[64];         // 物理地址
};
```

### 5.2 MIDI 事件

```cpp
// interfaces/kits/c/midi/native_midi_base.h:215-234
struct OH_MIDIEvent {
    uint64_t timestamp;  // 纳秒级时间戳
    size_t length;       // UMP 字数 (1-4)
    uint32_t *data;      // UMP 数据指针
};
```

### 5.3 共享内存事件头

```cpp
// services/common/include/midi_shared_ring.h:47-51
struct ShmMidiEventHeader {
    uint64_t timestamp;
    uint32_t length;     // payload 长度
    uint32_t flags;      // SHM_EVENT_FLAG_WRAP 等
};
```

---

*下一章: [03_CodeMap.md](03_CodeMap.md) - 目录结构与代码地图*
