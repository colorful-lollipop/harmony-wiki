# 内部实现细节 (Internals)

## 1. 核心类职责

### 1.1 客户端核心类

```
┌─────────────────────────────────────────────────────────────┐
│                    MidiClientPrivate                        │
│  职责: 管理客户端生命周期、设备列表、IPC 连接                  │
├─────────────────────────────────────────────────────────────┤
│  - ipcService_: MidiServiceClient (IPC 代理)                 │
│  - deviceInfos_: 设备列表缓存                                │
│  - callback_: 热插拔/错误回调                                │
├─────────────────────────────────────────────────────────────┤
│  关键方法:                                                   │
│  - Init()              建立 IPC 连接                         │
│  - GetDevices()        获取设备列表                          │
│  - OpenDevice()        打开设备                              │
│  - DestroyMidiClient() 清理资源                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    MidiDevicePrivate                        │
│  职责: 管理设备端口、数据传输                                 │
├─────────────────────────────────────────────────────────────┤
│  - ipcService_: weak_ptr 到 MidiServiceInterface             │
│  - deviceId_: 设备 ID                                        │
│  - inputPorts_: 输入端口映射                                 │
│  - outputPorts_: 输出端口映射                                │
├─────────────────────────────────────────────────────────────┤
│  关键方法:                                                   │
│  - OpenInputPort()     创建输入端口+接收线程                 │
│  - OpenOutputPort()    创建输出端口+共享内存                 │
│  - Send()              发送 MIDI 数据                        │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 服务端核心类

```
┌─────────────────────────────────────────────────────────────┐
│                  MidiServiceController                      │
│  职责: 全局状态管理、客户端生命周期、设备管理                  │
├─────────────────────────────────────────────────────────────┤
│  - deviceManager_: 设备管理器                                │
│  - clients_: 客户端映射 (clientId -> MidiInServer)           │
│  - deviceClientContexts_: 设备上下文映射                      │
│  - lock_: 互斥锁保护                                         │
├─────────────────────────────────────────────────────────────┤
│  关键方法:                                                   │
│  - CreateMidiInServer() 创建客户端会话                       │
│  - OpenDevice()         打开设备                             │
│  - DestroyMidiClient()  销毁客户端                           │
│  - ScheduleUnloadTask() 启动自动卸载                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 共享内存实现详解

### 2.1 内存布局

```
┌──────────────────────────────────────────────────────────────┐
│                    共享内存区域 (mmap)                        │
├──────────────────────────────────────────────────────────────┤
│  ControlHeader (64 bytes)                                     │
│  ├─ readPosition:  atomic<uint32_t>  读位置                  │
│  ├─ writePosition: atomic<uint32_t>  写位置                  │
│  ├─ capacity:       uint32_t         容量                    │
│  ├─ futexObj:       atomic<uint32_t> futex 对象              │
│  └─ flags:          uint32_t         标志位                  │
├──────────────────────────────────────────────────────────────┤
│  Ring Buffer Data                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ ShmMidiEventHeader │ Payload (UMP data) │ Padding      │ │
│  │ (16 bytes)         │ (variable)         │ (align 4)    │ │
│  └─────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ ShmMidiEventHeader │ Payload                           │ │
│  └─────────────────────────────────────────────────────────┘ │
│  ...                                                          │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 写入流程

```cpp
// services/common/src/midi_shared_ring.cpp:594-700
MidiStatusCode TryWriteEvents(...) {
    // 1. 验证参数
    ValidateWriteArgs(events, eventCount);
    
    // 2. 读取当前位置
    uint32_t writeIndex = controler_->writePosition.load();
    uint32_t readIndex = controler_->readPosition.load();
    
    // 3. 检查空间
    for each event {
        if (space insufficient) {
            *eventsWritten = writtenCount;
            return MidiStatusCode::WOULD_BLOCK;
        }
        
        // 4. 写入事件
        WriteEvent(writeIndex, event);
        
        // 5. 更新写位置
        controler_->writePosition.store(writeIndex);
    }
    
    // 6. 通知消费者
    WakeFutex();
    return MidiStatusCode::OK;
}
```

### 2.3 Futex 同步机制

```cpp
// services/common/include/futex_tool.h
enum class FutexCode { OK, TIMEOUT, INTERRUPTED };

class FutexTool {
public:
    // 等待条件满足或超时
    static FutexCode WaitFor(std::atomic<uint32_t> &futex, 
                             uint32_t expected, 
                             int64_t timeoutNs);
    // 唤醒等待者
    static void Wake(std::atomic<uint32_t> &futex, uint32_t wakeVal);
};
```

**使用模式**:
```cpp
// 生产者 (服务端写入数据)
ring.WriteEvent(...);
ring.WakeFutex(IS_READY);  // 唤醒消费者

// 消费者 (客户端读取数据)
futex.WaitFor(controler->futexObj, IS_READY, timeout);
ring.DrainToBatch(events, ...);
```

---

## 3. UMP 协议处理

### 3.1 UMP 包格式

```
Universal MIDI Packet (UMP):
┌─────────────────────────────────────────────────────────────┐
│ Message Type (MT) │ 包内容                                  │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0x0 (Utility)  │ 32-bit: JR Timestamp                    │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0x1 (System)   │ 32-bit: System Common/Real Time         │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0x2 (MIDI 1.0) │ 32-bit: Channel Voice Message           │
│                   │ [4b MT][4b Group][4b Status][4b Ch]     │
│                   │ [8b Data1][8b Data2]                    │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0x3 (Data)     │ 64-bit: SysEx/Data Message              │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0x4 (MIDI 2.0) │ 64-bit: Channel Voice Message           │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0xD (Flex)     │ 128-bit: Flex Data                      │
├───────────────────┼─────────────────────────────────────────┤
│ MT=0xF (Stream)   │ 128-bit: UMP Stream/Endpoint Discovery  │
└───────────────────┴─────────────────────────────────────────┘
```

### 3.2 MIDI 1.0 → UMP 转换

**代码位置**: `services/common/src/ump_processor.cpp`

```cpp
// MIDI 1.0 Note On: [0x90][Note][Velocity]
// 转换为 UMP Type 2:
uint32_t ump = (0x2 << 28)      // MT = 2 (MIDI 1.0 Channel Voice)
             | (0x0 << 24)      // Group = 0
             | (0x9 << 20)      // Status = Note On
             | (channel << 16)  // Channel
             | (note << 8)      // Note number
             | velocity;        // Velocity
```

---

## 4. 设备驱动抽象

### 4.1 驱动接口

```cpp
// services/server/include/midi_device_driver.h
class MidiDeviceDriver {
public:
    virtual ~MidiDeviceDriver() = default;
    
    // 设备管理
    virtual std::vector<DeviceInformation> GetRegisteredDevices() = 0;
    virtual int32_t OpenDevice(int64_t deviceId) = 0;
    virtual int32_t CloseDevice(int64_t deviceId) = 0;
    
    // 端口管理
    virtual int32_t OpenInputPort(int64_t deviceId, uint32_t portIndex, 
                                   UmpInputCallback cb) = 0;
    virtual int32_t CloseInputPort(int64_t deviceId, uint32_t portIndex) = 0;
    virtual int32_t OpenOutputPort(int64_t deviceId, uint32_t portIndex) = 0;
    virtual int32_t CloseOutputPort(int64_t deviceId, uint32_t portIndex) = 0;
    
    // 数据发送 (输出)
    virtual int32_t HanleUmpInput(int64_t deviceId, uint32_t portIndex,
                                   std::vector<MidiEventInner> &list) = 0;
};
```

### 4.2 USB 驱动实现

```cpp
// services/server/include/midi_device_usb.h
class UsbMidiTransportDeviceDriver : public MidiDeviceDriver {
private:
    sptr<HDI::Midi::V1_0::IMidiInterface> midiHdi_;  // HDI 接口
public:
    // 通过 HDI 调用内核驱动
    int32_t OpenDevice(int64_t deviceId) override {
        return midiHdi_->OpenDevice(deviceId);
    }
    
    int32_t HanleUmpInput(int64_t deviceId, uint32_t portIndex,
                           std::vector<MidiEventInner> &list) override {
        // 转换 UMP 为 HDI 消息格式
        std::vector<MidiMessage> messages = ConvertToHdi(list);
        return midiHdi_->SendMidiMessages(deviceId, portIndex, messages);
    }
};
```

### 4.3 BLE 驱动实现

```cpp
// services/server/include/midi_device_ble.h
class BleMidiTransportDeviceDriver : public MidiDeviceDriver {
private:
    std::unordered_map<int32_t, DeviceCtx> devices_;
    static constexpr const char *MIDI_SERVICE_UUID = 
        "03B80E5A-EDE8-4B33-A751-6CE34EC4C700";
    static constexpr const char *MIDI_CHAR_UUID = 
        "7772E5DB-3868-4112-A1A9-F2669D106BF3";
    
public:
    // UMP ↔ MIDI 1.0 Byte Stream 转换
    static void ConvertUmpToMidi1(const uint32_t* umpData, 
                                   size_t count, 
                                   std::vector<uint8_t>& midi1Bytes);
    static void ConvertMidi1ToUmp(const uint8_t* midiData,
                                   size_t count,
                                   std::vector<uint32_t>& umpWords);
};
```

---

## 5. 资源生命周期

### 5.1 客户端资源

```
创建: OH_MIDIClientCreate()
    ├── 分配 MidiClientPrivate
    ├── 建立 IPC 连接
    ├── 注册死亡通知
    └── 返回客户端句柄

使用期间:
    ├── 设备打开 → 分配 MidiDevicePrivate
    ├── 端口打开 → 分配 MidiInputPort/MidiOutputPort
    │   └── 创建共享内存 ring buffer
    └── 数据收发 → 读写共享内存

销毁: OH_MIDIClientDestroy()
    ├── 关闭所有端口
    ├── 关闭所有设备
    ├── IPC DestroyMidiClient()
    └── 释放内存
```

### 5.2 服务端资源

```
创建: CreateMidiInServer()
    ├── 分配 MidiInServer
    ├── 注册死亡通知
    ├── 加入 clients_ 映射
    └── 取消自动卸载定时器

使用期间:
    ├── 设备打开 → 创建设备上下文
    ├── 端口打开 → 创建设备连接 + 客户端连接
    │   ├── Input: 广播到所有客户端 rings
    │   └── Output: 每个客户端一个 ring
    └── 客户端死亡 → 死亡通知回调清理

销毁: DestroyMidiClient()
    ├── 关闭客户端关联的所有端口
    ├── 关闭客户端关联的所有设备
    ├── 从 clients_ 移除
    └── 无客户端时启动卸载定时器
```

---

## 6. 内部 API 契约

### 6.1 稳定接口

以下接口在版本迭代中保持稳定:

| 接口 | 位置 | 稳定性 |
|------|------|--------|
| `OH_MIDI*` C API | `native_midi.h` | **Stable** |
| `IMidiService.idl` | `services/idl/` | **Stable** |
| `IIpcMidiInServer.idl` | `services/idl/` | **Stable** |

### 6.2 内部实现细节

以下实现可能随版本变化:

| 组件 | 说明 |
|------|------|
| `MidiSharedRing` 布局 | 共享内存结构可能优化 |
| `UmpProcessor` 算法 | UMP 转换实现可能改进 |
| 自动卸载策略 | 超时时间可能调整 |
| 线程模型 | 可能引入线程池 |

---

*文档结束*
