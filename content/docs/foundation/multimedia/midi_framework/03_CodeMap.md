# 目录结构与代码地图 (CodeMap)

## 1. 顶层目录结构

```
foundation/multimedia/midi_framework/
├── bundle.json              # 部件描述与编译配置
├── config.gni               # GN 编译变量定义
├── figures/                 # 架构图资源
│   ├── zh-cn_image_midi_framework.png
│   ├── zh-cn_image_midi_framework_life_cycle.png
│   ├── zh-cn_image_midi_framework_device_manage.png
│   └── zh-cn_image_midi_framework_data_transfer.png
├── frameworks/              # 框架层实现
│   └── native/
│       ├── midi/            # 客户端核心逻辑
│       ├── midiutils/       # 基础工具库
│       └── ohmidi/          # C API 入口实现
├── interfaces/              # 接口定义
│   ├── inner_api/           # C++ 内部 API
│   ├── kits/c/midi/         # C Native API
│   └── *.h                  # 公共头文件
├── sa_profile/              # 系统服务配置
│   ├── midi_server.json     # SA 配置 (SAID: 3014)
│   └── midi_server.cfg      # 进程启动配置
├── services/                # 服务层实现
│   ├── common/              # 共享内存、UMP 处理
│   ├── idl/                 # IPC 接口定义
│   └── server/              # 服务端核心逻辑
└── test/                    # 测试代码 (本文档忽略)
```

---

## 2. 框架层 (frameworks/native)

### 2.1 ohmidi - C API 实现

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `frameworks/native/ohmidi/OHMidi.cpp` | 所有 OH_MIDI* 函数实现 | `OH_MIDIClientCreate`, `OH_MIDISend` 等 18 个 API |

**函数映射示例:**
```cpp
// OHMIDI.cpp:28
OH_MIDIStatusCode OH_MIDIClientCreate(OH_MIDIClient **client, ...) {
    return OHOS::MIDI::MidiClient::CreateMidiClient(...);
}
```

### 2.2 midi - 客户端核心

| 文件 | 职责 | 关键类/函数 |
|------|------|-------------|
| `include/midi_client.h` | C++ API 抽象类定义 | `class MidiClient`, `class MidiDevice` |
| `include/midi_client_private.h` | 私有类定义 | `MidiClientPrivate`, `MidiDevicePrivate`, `MidiInputPort`, `MidiOutputPort` |
| `include/midi_service_client.h` | IPC 客户端 | `MidiServiceClient` |
| `include/midi_service_interface.h` | 服务接口抽象 | `MidiServiceInterface` |
| `src/midi_client.cpp` | 客户端实现 | 设备/端口管理、数据收发 |
| `src/midi_service_client.cpp` | IPC 实现 | 服务发现、共享内存映射 |

**代码导航:**
- 设备打开: `src/midi_client.cpp:103-156`
- 输入端口接收线程: `src/midi_client.cpp:252-289`
- 数据发送: `src/midi_client.cpp:390-412`

### 2.3 midiutils - 工具库

| 文件 | 职责 |
|------|------|
| `include/midi_utils.h` | 工具函数声明 |
| `include/midi_log.h` | 日志宏定义 |
| `src/midi_utils.cpp` | 字符串处理、地址加密等 |

---

## 3. 接口层 (interfaces)

### 3.1 Native C API

| 文件 | 职责 | 代码量 |
|------|------|--------|
| `interfaces/kits/c/midi/native_midi.h` | 18 个 API 声明 | 332 行 |
| `interfaces/kits/c/midi/native_midi_base.h` | 数据结构、枚举、回调 | 426 行 |

**关键定义:**
- 错误码: `enum OH_MIDIStatusCode` (行 51-133)
- 设备信息: `struct OH_MIDIDeviceInformation` (行 241-274)
- 事件结构: `struct OH_MIDIEvent` (行 215-234)
- 回调类型: `OH_OnMIDIReceived`, `OH_OnMIDIDeviceChange` (行 353-420)

### 3.2 C++ 内部 API

| 文件 | 职责 |
|------|------|
| `interfaces/inner_api/native/midi_client.h` | `MidiClient`/`MidiDevice` 抽象类 |
| `interfaces/midi_info.h` | 内部数据结构 (DeviceInformation, MidiEvent 等) |
| `interfaces/midi_log.h` | 日志标签定义 |
| `interfaces/midi_service_death_recipent.h` | 死亡通知接收器 |

---

## 4. 服务层 (services)

### 4.1 idl - IPC 接口定义

| 文件 | 职责 | 关键接口 |
|------|------|----------|
| `services/idl/IMidiService.idl` | 服务工厂 | `CreateMidiInServer()` |
| `services/idl/IIpcMidiInServer.idl` | 主 IPC 接口 | `GetDevices`, `OpenDevice`, `OpenInputPort` 等 10 个方法 |
| `services/idl/IMidiCallback.idl` | 回调接口 | `NotifyDeviceChange`, `NotifyError` |
| `services/idl/IMidiDeviceOpenCallback.idl` | BLE 回调 | `NotifyDeviceOpened` |

### 4.2 server - 服务端实现

#### 核心控制器

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_service_controller.h` | 服务控制器 | `MidiServiceController` 单例 |
| `src/midi_service_controller.cpp` | 控制器实现 | 客户端管理、设备生命周期、自动卸载 |

**代码位置速查:**
```
客户端创建:    midi_service_controller.cpp:127-159
设备打开:      midi_service_controller.cpp:185-213
BLE设备打开:   midi_service_controller.cpp:215-321
输入端口打开:  midi_service_controller.cpp:323-362
自动卸载:      midi_service_controller.cpp:98-125
```

#### 服务入口

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_server.h` | 服务入口 | `MidiServer : public SystemAbility` |
| `src/midi_server.cpp` | 启动逻辑 | `OnStart()`, `OnStop()` |

#### 客户端会话

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_in_server.h` | 客户端会话 | `MidiInServer : public IpcMidiInServerStub` |
| `src/midi_in_server.cpp` | IPC 处理 | 转发请求到 MidiServiceController |

#### 设备管理

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_device_mananger.h` | 设备管理器 | `MidiDeviceManager` |
| `src/midi_device_mananger.cpp` | 设备枚举、驱动管理 | USB/BLE 设备列表维护 |

**代码位置速查:**
```
设备列表获取:  midi_device_mananger.cpp:47-84
设备打开:      midi_device_mananger.cpp:86-109
```

#### 设备驱动

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_device_driver.h` | 驱动抽象基类 | `class MidiDeviceDriver` |
| `include/midi_device_usb.h` | USB 驱动 | `UsbMidiTransportDeviceDriver` |
| `src/midi_device_usb.cpp` | USB 实现 | HDI 接口调用 |
| `include/midi_device_ble.h` | BLE 驱动 | `BleMidiTransportDeviceDriver` |
| `src/midi_device_ble.cpp` | BLE 实现 | GATT 连接、UMP 转换 |

**代码位置速查:**
```
USB 设备列表:  midi_device_usb.cpp:43-60
USB 打开设备:  midi_device_usb.cpp:62-77
BLE MAC 解析:  midi_device_ble.cpp:180-220
BLE UMP 转换:  midi_device_ble.cpp:52-111
```

#### 连接管理

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_device_connection.h` | 设备连接 | `DeviceConnectionForInput`, `DeviceConnectionForOutput` |
| `src/midi_device_connection.cpp` | 数据路由 | 输入广播、输出发送调度 |
| `include/midi_client_connection.h` | 客户端连接 | `ClientConnectionInServer` |
| `src/midi_client_connection.cpp` | 共享内存管理 | 每个客户端的 ring buffer |

**代码位置速查:**
```
输入数据广播:  midi_device_connection.cpp:107-145
输出调度循环:  midi_device_connection.cpp:201-280
```

#### 回调处理

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_listener_callback.h` | 回调适配器 | `MidiListenerCallback` |
| `src/midi_listener_callback.cpp` | 转发到客户端 | `NotifyDeviceChange`, `NotifyError` |

### 4.3 common - 共享组件

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `include/midi_shared_memory.h` | 共享内存抽象 | `MidiSharedMemory` |
| `include/midi_shared_ring.h` | 无锁队列 | `MidiSharedRing`, `ControlHeader`, `ShmMidiEventHeader` |
| `src/midi_shared_ring.cpp` | 队列实现 | `TryWriteEvents`, `DrainToBatch` |
| `include/futex_tool.h` | 用户态同步 | `FutexTool`, `FutexCode` |
| `src/futex_tool.cpp` | futex 封装 | futex_wait/wake |
| `include/ump_processor.h` | UMP 转换 | `UmpProcessor` |
| `src/ump_processor.cpp` | MIDI 1.0 → UMP | Byte stream 解析 |
| `include/ump_packet.h` | UMP 包 | `UmpPacket` |
| `src/ump_packet.cpp` | 包实现 | 128-bit 数据处理 |

**代码位置速查:**
```
共享内存创建:  midi_shared_ring.cpp:50-130
事件写入:      midi_shared_ring.cpp:594-700
事件读取:      midi_shared_ring.cpp:702-800
UMP 转换:      ump_processor.cpp:48-168
```

---

## 5. 配置文件

| 文件 | 职责 | 关键配置 |
|------|------|----------|
| `bundle.json` | 部件配置 | SAID: 3014, 依赖组件列表 |
| `sa_profile/midi_server.json` | 服务配置 | 进程名: midi_server, 库: libmidi_service.z.so |
| `sa_profile/midi_server.cfg` | 进程配置 | 启动参数、资源限制 |
| `config.gni` | 编译变量 | `midi_framework_root` |

---

## 6. 代码导航图

### 6.1 完整调用链示例

```
App 调用 OH_MIDISend()
    ↓
frameworks/native/ohmidi/OHMidi.cpp:169
    ↓ (C API → C++)
frameworks/native/midi/src/midi_client.cpp:390
    ↓ (Client → IPC)
frameworks/native/midi/src/midi_service_client.cpp:156
    ↓ (IPC 传输)
services/server/src/midi_in_server.cpp:60
    ↓ (Server 处理)
services/server/src/midi_service_controller.cpp:364
    ↓ (端口连接)
services/server/src/midi_device_connection.cpp:201
    ↓ (写入共享内存)
services/common/src/midi_shared_ring.cpp:594
    ↓ (驱动发送)
services/server/src/midi_device_usb.cpp:105
    ↓ (HDI 调用)
drivers_peripheral MIDI HDI
```

### 6.2 关键问题定位指南

| 问题类型 | 定位文件 | 查找关键词 |
|----------|----------|------------|
| 服务启动失败 | `services/server/src/midi_server.cpp` | `OnStart`, `SystemAbility` |
| 客户端连接失败 | `services/server/src/midi_service_controller.cpp:127` | `CreateMidiInServer` |
| 设备列表为空 | `services/server/src/midi_device_mananger.cpp:47` | `GetDevices` |
| 数据发送失败 | `services/common/src/midi_shared_ring.cpp:594` | `TryWriteEvents` |
| 数据接收失败 | `frameworks/native/midi/src/midi_client.cpp:252` | `ReceiverThread` |
| BLE 连接失败 | `services/server/src/midi_device_ble.cpp` | `OpenDevice` |
| 服务异常退出 | `services/server/src/midi_service_controller.cpp:98` | `ScheduleUnloadTask` |

---

*下一章: [04_Interface.md](04_Interface.md) - 对外接口文档*
