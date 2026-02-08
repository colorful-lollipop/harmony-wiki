# 对外接口文档 (Interface)

## 1. API 总览

`midi_framework` 提供 18 个 Native C API，分为 4 类：

| 类别 | 数量 | API |
|------|------|-----|
| 客户端生命周期 | 2 | Create / Destroy |
| 设备管理 | 4 | GetCount / GetInfos / Open / Close |
| 端口管理 | 5 | GetCount / GetInfos / OpenInput / OpenOutput / Close |
| 数据传输 | 3 | Send / SendSysEx / Flush |

---

## 2. 客户端生命周期 API

### 2.1 OH_MIDIClientCreate

**功能**: 创建 MIDI 客户端实例，建立与服务器的 IPC 连接

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:58
OH_MIDIStatusCode OH_MIDIClientCreate(
    OH_MIDIClient **client,           // 输出: 客户端句柄
    OH_MIDICallbacks callbacks,        // 回调函数结构
    void *userData                     // 用户数据
);
```

**返回值**:
| 状态码 | 值 | 说明 |
|--------|-----|------|
| MIDI_STATUS_OK | 0 | 成功 |
| MIDI_STATUS_GENERIC_INVALID_ARGUMENT | 1 | client 为 nullptr |
| MIDI_STATUS_GENERIC_IPC_FAILURE | 2 | IPC 连接失败 |

**回调结构**:
```cpp
typedef struct {
    OH_OnMIDIDeviceChange onDeviceChange;  // 设备热插拔回调
    OH_OnMIDIError onError;                 // 错误回调
} OH_MIDICallbacks;
```

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:24-32`

---

### 2.2 OH_MIDIClientDestroy

**功能**: 销毁客户端，释放所有资源

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:69
OH_MIDIStatusCode OH_MIDIClientDestroy(OH_MIDIClient *client);
```

**返回值**:
| 状态码 | 说明 |
|--------|------|
| MIDI_STATUS_OK | 成功 |
| MIDI_STATUS_INVALID_CLIENT | 客户端无效 |
| MIDI_STATUS_GENERIC_IPC_FAILURE | IPC 失败 |

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:34-42`

---

## 3. 设备管理 API

### 3.1 OH_MIDIGetDeviceCount

**功能**: 获取当前连接的 MIDI 设备数量

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:84
OH_MIDIStatusCode OH_MIDIGetDeviceCount(
    OH_MIDIClient *client,
    size_t *count          // 输出: 设备数量
);
```

**返回值**:
| 状态码 | 说明 |
|--------|------|
| MIDI_STATUS_OK | 成功 |
| MIDI_STATUS_INVALID_CLIENT | 客户端无效 |
| MIDI_STATUS_GENERIC_INVALID_ARGUMENT | count 为 nullptr |

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:44-51`

---

### 3.2 OH_MIDIGetDeviceInfos

**功能**: 获取设备详细信息列表

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:107
OH_MIDIStatusCode OH_MIDIGetDeviceInfos(
    OH_MIDIClient *client,
    OH_MIDIDeviceInformation *infos,     // 用户分配缓冲区
    size_t capacity,                      // 缓冲区容量
    size_t *actualNumDevices              // 实际返回数量
);
```

**设备信息结构**:
```cpp
typedef struct {
    int64_t midiDeviceId;           // 设备唯一ID
    OH_MIDIDeviceType deviceType;   // USB=0, BLE=1
    OH_MIDIProtocol nativeProtocol; // PROTOCOL_1_0=1, PROTOCOL_2_0=2
    char productName[256];          // 产品名称
    char vendorName[256];           // 厂商名称
    char deviceAddress[64];         // 物理地址 (MAC/序列号)
} OH_MIDIDeviceInformation;
```

**竞态处理**: 设备数量可能在两次调用间变化，需检查 `actualNumDevices`

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:53-68`

---

### 3.3 OH_MIDIOpenDevice

**功能**: 打开指定的 USB MIDI 设备

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:125
OH_MIDIStatusCode OH_MIDIOpenDevice(
    OH_MIDIClient *client,
    int64_t deviceId,           // 设备ID (从 GetDeviceInfos 获取)
    OH_MIDIDevice **device      // 输出: 设备句柄
);
```

**返回值**:
| 状态码 | 说明 |
|--------|------|
| MIDI_STATUS_OK | 成功 |
| MIDI_STATUS_INVALID_CLIENT | 客户端无效 |
| MIDI_STATUS_DEVICE_ALREADY_OPEN | 设备已打开 |
| MIDI_STATUS_GENERIC_INVALID_ARGUMENT | device 为 nullptr 或 deviceId 无效 |

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:70-83`

---

### 3.4 OH_MIDIOpenBleDevice

**功能**: 异步打开 BLE MIDI 设备

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:147
OH_MIDIStatusCode OH_MIDIOpenBleDevice(
    OH_MIDIClient *client,
    const char *deviceAddr,              // MAC 地址 (如 "AA:BB:CC:DD:EE:FF")
    OH_MIDIOnDeviceOpened callback,       // 异步回调
    void *userData
);
```

**权限要求**: `ohos.permission.ACCESS_BLUETOOTH`

**回调定义**:
```cpp
typedef void (*OH_MIDIOnDeviceOpened)(
    void *userData,
    bool opened,                        // true: 成功, false: 失败
    OH_MIDIDevice *device,              // 成功时有效
    OH_MIDIDeviceInformation info
);
```

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:85-97`

---

### 3.5 OH_MIDICloseDevice

**功能**: 关闭设备

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:158
OH_MIDIStatusCode OH_MIDICloseDevice(OH_MIDIDevice *device);
```

**返回值**:
| 状态码 | 说明 |
|--------|------|
| MIDI_STATUS_OK | 成功 |
| MIDI_STATUS_INVALID_DEVICE_HANDLE | 设备句柄无效 |

**实现位置**: `frameworks/native/ohmidi/OHMidi.cpp:99-107`

---

## 4. 端口管理 API

### 4.1 OH_MIDIGetPortCount / OH_MIDIGetPortInfos

**功能**: 获取设备端口数量和端口信息

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:175
OH_MIDIStatusCode OH_MIDIGetPortCount(
    OH_MIDIClient *client,
    int64_t deviceId,
    size_t *count
);

// interfaces/kits/c/midi/native_midi.h:200
OH_MIDIStatusCode OH_MIDIGetPortInfos(
    OH_MIDIClient *client,
    int64_t deviceId,
    OH_MIDIPortInformation *infos,
    size_t capacity,
    size_t *actualNumPorts
);
```

**端口信息结构**:
```cpp
typedef struct {
    uint32_t portIndex;                 // 端口索引
    int64_t deviceId;                   // 所属设备ID
    OH_MIDIPortDirection direction;     // INPUT=0, OUTPUT=1
    char name[64];                      // 端口名称
} OH_MIDIPortInformation;
```

**实现位置**: `OHMidi.cpp:109-134`

---

### 4.2 OH_MIDIOpenInputPort

**功能**: 打开输入端口（接收数据）

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:224
OH_MIDIStatusCode OH_MIDIOpenInputPort(
    OH_MIDIDevice *device,
    OH_MIDIPortDescriptor descriptor,    // 端口描述符
    OH_OnMIDIReceived callback,           // 数据接收回调
    void *userData
);
```

**端口描述符**:
```cpp
typedef struct {
    uint32_t portIndex;                 // 端口索引
    OH_MIDIProtocol protocol;           // MIDI_PROTOCOL_1_0 / MIDI_PROTOCOL_2_0
} OH_MIDIPortDescriptor;
```

**回调定义**:
```cpp
typedef void (*OH_OnMIDIReceived)(
    void *userData,
    const OH_MIDIEvent *events,         // 事件数组
    size_t eventCount                   // 事件数量
);
```

**注意**: 回调运行在非 UI 线程，请勿直接操作 UI

**实现位置**: `OHMidi.cpp:136-147`

---

### 4.3 OH_MIDIOpenOutputPort

**功能**: 打开输出端口（发送数据）

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:240
OH_MIDIStatusCode OH_MIDIOpenOutputPort(
    OH_MIDIDevice *device,
    OH_MIDIPortDescriptor descriptor
);
```

**实现位置**: `OHMidi.cpp:149-157`

---

### 4.4 OH_MIDIClosePort

**功能**: 关闭指定端口

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:253
OH_MIDIStatusCode OH_MIDIClosePort(
    OH_MIDIDevice *device,
    uint32_t portIndex
);
```

**实现位置**: `OHMidi.cpp:159-167`

---

### 4.5 OH_MIDIFlushOutputPort

**功能**: 清空输出端口缓冲区

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:326
OH_MIDIStatusCode OH_MIDIFlushOutputPort(
    OH_MIDIDevice *device,
    uint32_t portIndex
);
```

**注意**: 此函数不发送 "All Notes Off" 消息，仅清空队列

**实现位置**: `OHMidi.cpp:187-191`

---

## 5. 数据传输 API

### 5.1 OH_MIDISend

**功能**: 发送 MIDI 事件（非阻塞、原子操作）

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:279
OH_MIDIStatusCode OH_MIDISend(
    OH_MIDIDevice *device,
    uint32_t portIndex,
    OH_MIDIEvent *events,        // 事件数组
    uint32_t eventCount,          // 事件数量
    uint32_t *eventsWritten       // 实际写入数量
);
```

**事件结构**:
```cpp
typedef struct {
    uint64_t timestamp;     // 纳秒级时间戳 (0=立即发送)
    size_t length;          // UMP 字数 (1-4)
    uint32_t *data;         // UMP 数据 (4字节对齐)
} OH_MIDIEvent;
```

**返回值**:
| 状态码 | 说明 |
|--------|------|
| MIDI_STATUS_OK | 全部事件发送成功 |
| MIDI_STATUS_WOULD_BLOCK | 缓冲区满，部分发送 |
| MIDI_STATUS_INVALID_DEVICE_HANDLE | 设备无效 |
| MIDI_STATUS_INVALID_PORT | 端口无效或未打开 |

**特性**:
- 非阻塞：缓冲区满时立即返回
- 原子性：每个事件要么完整写入，要么不写入
- 部分成功：检查 `eventsWritten` 确认

**示例**:
```cpp
// 发送 Note On (Channel 0, Note 60, Velocity 100)
uint32_t noteOn = 0x20903C64;  // [MT=2][Group=0][Status=9][Ch=0][Note=3C][Vel=64]
OH_MIDIEvent event = {0, 1, &noteOn};
uint32_t written = 0;
OH_MIDISend(device, 0, &event, 1, &written);
```

**实现位置**: `OHMidi.cpp:169-177`

---

### 5.2 OH_MIDISendSysEx

**功能**: 发送长 SysEx 消息（字节流到 UMP 辅助函数）

**声明**:
```cpp
// interfaces/kits/c/midi/native_midi.h:307
OH_MIDIStatusCode OH_MIDISendSysEx(
    OH_MIDIDevice *device,
    uint32_t portIndex,
    uint8_t *data,           // 原始字节流 (F0...F7)
    uint32_t byteSize        // 字节数
);
```

**注意**:
- **阻塞调用**：内部循环直到全部发送完成
- 自动分片为 Type 3 (64-bit Data Message) UMP 包
- 适用于 MIDI 1.0 风格的字节流数据

**实现位置**: `OHMidi.cpp:179-185`

---

## 6. 错误码汇总

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| MIDI_STATUS_OK | 0 | 操作成功 |
| MIDI_STATUS_GENERIC_INVALID_ARGUMENT | 1 | 无效参数（空指针等） |
| MIDI_STATUS_GENERIC_IPC_FAILURE | 2 | IPC 通信失败 |
| MIDI_STATUS_INSUFFICIENT_RESULT_SPACE | 3 | 缓冲区空间不足 |
| MIDI_STATUS_INVALID_CLIENT | 4 | 无效客户端句柄 |
| MIDI_STATUS_INVALID_DEVICE_HANDLE | 5 | 无效设备句柄 |
| MIDI_STATUS_INVALID_PORT | 6 | 无效端口索引 |
| MIDI_STATUS_WOULD_BLOCK | 7 | 发送缓冲区满（非阻塞） |
| MIDI_STATUS_TIMEOUT | 8 | 操作超时 |
| MIDI_STATUS_TOO_MANY_OPEN_DEVICES | 9 | 打开设备数超限 |
| MIDI_STATUS_TOO_MANY_OPEN_PORTS | 10 | 打开端口数超限 |
| MIDI_STATUS_DEVICE_ALREADY_OPEN | 11 | 设备已被此客户端打开 |
| MIDI_STATUS_PORT_ALREADY_OPEN | 12 | 端口已被此客户端打开 |
| MIDI_STATUS_SERVICE_DIED | 13 | MIDI 服务死亡/断开 |
| MIDI_STATUS_UNKNOWN_ERROR | -1 | 未知系统错误 |

---

## 7. IPC 接口定义

### 7.1 IMidiService.idl

```idl
interface IMidiService {
    [ipccode 0] void CreateMidiInServer(
        [in] IRemoteObject object,      // 回调 Stub
        [out] IRemoteObject client,     // 返回 IIpcMidiInServer proxy
        [out] unsigned int clientId
    );
}
```

### 7.2 IIpcMidiInServer.idl

```idl
interface IIpcMidiInServer {
    [ipccode 0] void GetDevices([out] List<OrderedMap<int, String>> devices);
    void GetDevicePorts([in] long deviceId, [out] List<OrderedMap<int, String>> ports);
    void OpenDevice([in] long deviceId);
    void OpenBleDevice([in] String address, [in] IRemoteObject object);
    void OpenInputPort([out] sharedptr<MidiSharedRing> buffer, [in] long deviceId, [in] unsigned int portIndex);
    void OpenOutputPort([out] sharedptr<MidiSharedRing> buffer, [in] long deviceId, [in] unsigned int portIndex);
    void CloseInputPort([in] long deviceId, [in] unsigned int portIndex);
    void CloseOutputPort([in] long deviceId, [in] unsigned int portIndex);
    void CloseDevice([in] long deviceId);
    void DestroyMidiClient();
}
```

**位置**: `services/idl/`

---

*下一章: [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析*
