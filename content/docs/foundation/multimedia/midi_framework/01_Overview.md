# 项目概览 (Overview)

## 1. 一句话定义

`midi_framework` 是 OpenHarmony 的多媒体 MIDI 系统服务，为应用提供 USB 和 BLE MIDI 设备的统一管理能力，以及基于 UMP 协议的高性能低延迟数据传输。

---

## 2. 能力边界

### 2.1 能做什么

| 能力 | 说明 |
|------|------|
| 设备发现 | 自动发现 USB MIDI 热插拔、BLE MIDI 扫描连接 |
| 设备管理 | 多客户端并发访问，设备状态生命周期管理 |
| 数据传输 | UMP 格式数据收发，支持 MIDI 1.0/2.0 |
| 协议转换 | 内部自动处理 UMP ↔ MIDI 1.0 字节流转换 |
| 高性能 IO | 共享内存零拷贝，futex 高效同步 |

### 2.2 不能做什么

| 限制 | 说明 |
|------|------|
| 无音频合成 | 本框架仅处理 MIDI 控制数据，不包含音源合成 |
| 无文件解析 | 不支持 MIDI 文件（.mid）的读取和播放 |
| 无 JS API | 仅提供 Native C API，未封装 ArkTS/JS 接口 |
| 无网络 MIDI | 不支持 RTP-MIDI 等网络传输协议 |

---

## 3. 运行环境

### 3.1 软件依赖

**必需组件** (`bundle.json`):
```json
[
    "usb_manager",          // USB 设备热插拔检测
    "bluetooth",            // BLE GATT 连接
    "samgr",                // 系统能力管理（服务拉起）
    "safwk",                // SA 框架
    "ipc",                  // IPC 通信
    "drivers_interface_midi" // HDI 驱动接口
]
```

### 3.2 硬件要求

**USB MIDI:**
- 开发设备支持 USB Host 模式
- 内核开启 ALSA 支持 (`CONFIG_SND_USB_AUDIO`, `CONFIG_SND_RAWMIDI`)
- `midi_server` 对 `/dev/snd/midiC*D*` 有读写权限

**BLE MIDI:**
- 开发设备支持 Bluetooth Low Energy
- 应用需申请权限 `ohos.permission.ACCESS_BLUETOOTH`

### 3.3 SystemCapability

应用需检查系统能力:
```cpp
@syscap SystemCapability.Multimedia.Audio.MIDI
```

---

## 4. 架构分层

```
┌─────────────────────────────────────────┐
│           应用层 (Application)           │
│     MIDI APP (DAW, 教学软件等)           │
└─────────────────┬───────────────────────┘
                  │ Native API
┌─────────────────▼───────────────────────┐
│           框架层 (Framework)             │
│   OHMIDI → MidiClient → MidiDevice      │
└─────────────────┬───────────────────────┘
                  │ IPC (IIpcMidiInServer)
┌─────────────────▼───────────────────────┐
│         系统服务层 (System Service)       │
│   MidiServer → MidiServiceController    │
│   → MidiDeviceManager → USB/BLE Driver  │
└─────────────────┬───────────────────────┘
                  │ HDI / Bluetooth API
┌─────────────────▼───────────────────────┐
│           驱动层 (Driver)                │
│   USB: ALSA HDI  /  BLE: GATT Client    │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│           外设 (Hardware)                │
│   USB MIDI 键盘  /  BLE MIDI 控制器      │
└─────────────────────────────────────────┘
```

详细架构见 [02_Architecture.md](02_Architecture.md)

---

## 5. 快速开始

### 5.1 最小示例

```cpp
#include <native_midi.h>
#include <iostream>

// 1. 创建客户端
OH_MIDIClient *client = nullptr;
OH_MIDICallbacks callbacks = {nullptr, nullptr};
OH_MIDIClientCreate(&client, callbacks, nullptr);

// 2. 获取设备列表
size_t count = 0;
OH_MIDIGetDeviceCount(client, &count);
std::vector<OH_MIDIDeviceInformation> devices(count);
OH_MIDIGetDeviceInfos(client, devices.data(), count, &count);

// 3. 打开设备
OH_MIDIDevice *device = nullptr;
OH_MIDIOpenDevice(client, devices[0].midiDeviceId, &device);

// 4. 打开输出端口
OH_MIDIPortDescriptor desc = {0, MIDI_PROTOCOL_1_0};
OH_MIDIOpenOutputPort(device, desc);

// 5. 发送 MIDI Note On
uint32_t noteOn = 0x20903C64;  // Note On, Ch0, Note 60, Vel 100
OH_MIDIEvent event = {0, 1, &noteOn};
uint32_t written = 0;
OH_MIDISend(device, 0, &event, 1, &written);

// 6. 清理
OH_MIDICloseDevice(device);
OH_MIDIClientDestroy(client);
```

完整示例见 [README_zh.md](../README_zh.md) 第 214-338 行

---

## 6. 服务生命周期

### 6.1 按需启动

```
App 调用 OH_MIDIClientCreate()
        ↓
查询 SAMgr MIDI 服务 (SAID: 3014)
        ↓
服务未启动 → SAMgr 拉起 midi_server 进程
        ↓
服务初始化，建立 IPC 连接
```

### 6.2 自动退出

```
最后一个客户端销毁
        ↓
启动 60 秒卸载定时器
        ↓
无新连接 → 调用 UnloadSystemAbility(3014)
        ↓
服务进程退出
```

代码位置: `services/server/src/midi_service_controller.cpp:98-125`

---

## 7. 相关资源

### 7.1 官方文档
- [OpenHarmony MIDI 开发指南](https://gitcode.com/openharmony/docs)
- [MIDI 2.0 UMP 规范](https://midi.org/midi-2-0-specifications)
- [BLE MIDI 规范](https://midi.org/midi-over-bluetooth-low-energy-ble-midi)

### 7.2 相关仓库
- [drivers_peripheral](https://gitcode.com/openharmony/drivers_peripheral) - MIDI HDI 驱动实现
- [drivers_interface](https://gitcode.com/openharmony/drivers_interface) - HDI 接口定义
- [alsa-libs](https://gitcode.com/openharmony/third_party_alsa-lib) - ALSA 库

---

*下一章: [02_Architecture.md](02_Architecture.md) - 架构与数据流分析*
