# 攻击面分析 (AttackSurface)

## 1. 攻击面概述

本文档识别 midi_framework 的所有外部输入入口、敏感操作和信任边界，为安全研究提供完整攻击面视图。

---

## 2. 外部输入清单

### 2.1 N-API 参数输入

**攻击面**: 应用通过 Native API 传入的参数

| API | 参数 | 风险点 | 代码位置 |
|-----|------|--------|----------|
| `OH_MIDIClientCreate` | `client` (出参) | 空指针解引用 | `OHMidi.cpp:26` |
| `OH_MIDIOpenBleDevice` | `deviceAddr` | 字符串长度、格式验证 | `OHMidi.cpp:90` |
| `OH_MIDIGetDeviceInfos` | `infos`, `capacity` | 缓冲区溢出 | `OHMidi.cpp:60` |
| `OH_MIDIGetPortInfos` | `infos`, `capacity` | 缓冲区溢出 | `OHMidi.cpp:127` |
| `OH_MIDISend` | `events`, `eventCount` | 数组越界、数据长度 | `OHMidi.cpp:173` |
| `OH_MIDISendSysEx` | `data`, `byteSize` | 大数据包、长度欺骗 | `OHMidi.cpp:181` |

**输入验证点**:
```cpp
// frameworks/native/ohmidi/OHMidi.cpp:26
CHECK_AND_RETURN_RET_LOG(client != nullptr, MIDI_STATUS_GENERIC_INVALID_ARGUMENT, ...)

// frameworks/native/ohmidi/OHMidi.cpp:90
CHECK_AND_RETURN_RET_LOG(deviceAddr != nullptr && callback != nullptr, ...)
```

---

### 2.2 IPC 数据输入

**攻击面**: 跨进程通信数据

| IPC 接口 | 输入参数 | 风险点 | 代码位置 |
|----------|----------|--------|----------|
| `CreateMidiInServer` | `object` (IRemoteObject) | 死亡通知对象伪造 | `midi_service_controller.cpp:130` |
| `OpenDevice` | `deviceId` | 非法设备 ID | `midi_service_controller.cpp:185` |
| `OpenBleDevice` | `address` | MAC 地址格式、长度 | `midi_service_controller.cpp:215` |
| `OpenInputPort` | `deviceId`, `portIndex` | 索引越界 | `midi_service_controller.cpp:323` |

**IPC 参数验证**:
```cpp
// services/server/src/midi_service_controller.cpp:130
CHECK_AND_RETURN_RET_LOG(object, MIDI_STATUS_UNKNOWN_ERROR, "object is nullptr");

// services/server/src/midi_service_controller.cpp:329
CHECK_AND_RETURN_RET_LOG(clients_.find(clientId) != clients_.end(),
    MIDI_STATUS_INVALID_CLIENT, "Client not found");
```

---

### 2.3 USB 设备输入

**攻击面**: 物理 USB MIDI 设备

| 输入源 | 攻击向量 | 风险等级 |
|--------|----------|----------|
| USB 描述符 | 恶意 VID/PID、端点配置 | **HIGH** |
| USB 热插拔 | 设备模拟、BadUSB | **HIGH** |
| MIDI 数据流 | SysEx 攻击、缓冲区溢出 | **MEDIUM** |
| 设备名称 | 字符串注入 | LOW |

**代码位置**:
- USB 设备发现: `services/server/src/midi_device_usb.cpp:43`
- HDI 接口调用: `services/server/src/midi_device_usb.cpp:62`

---

### 2.4 BLE 设备输入

**攻击面**: 蓝牙 LE MIDI 设备

| 输入源 | 攻击向量 | 风险等级 |
|--------|----------|----------|
| BLE 广播包 | 设备伪装、MITM | **HIGH** |
| GATT 数据 | 畸形 UMP 包、超长数据 | **MEDIUM** |
| MAC 地址 | 地址欺骗 | MEDIUM |
| 设备名称 | 字符串注入 | LOW |

**BLE UUID**:
```cpp
// services/server/src/midi_device_ble.cpp:41-42
static constexpr const char *MIDI_SERVICE_UUID = "03B80E5A-EDE8-4B33-A751-6CE34EC4C700";
static constexpr const char *MIDI_CHAR_UUID = "7772E5DB-3868-4112-A1A9-F2669D106BF3";
```

**代码位置**:
- MAC 地址解析: `services/server/src/midi_device_ble.cpp:180-220`
- UMP 转换: `services/server/src/midi_device_ble.cpp:52-111`

---

### 2.5 共享内存输入

**攻击面**: 客户端映射的共享内存区域

| 输入源 | 攻击向量 | 风险等级 |
|--------|----------|----------|
| File Descriptor | 伪造 FD、stdin/stdout 注入 | **MEDIUM** |
| 内存数据 | 竞态条件、数据篡改 | **MEDIUM** |
| 控制头 | 位置指针伪造 | **HIGH** |

**保护机制**:
```cpp
// services/common/src/midi_shared_ring.cpp:196-205
int minfd = 2; // ignore stdout, stdin and stderr
CHECK_AND_RETURN_RET_LOG(fd > minfd, nullptr, "CreateFromRemote failed: invalid fd");

// 验证实际文件大小
off_t actualSize = lseek(fd, 0, SEEK_END);
CHECK_AND_RETURN_RET_LOG((actualSize == (off_t)size) && size != 0, nullptr, ...);
```

---

### 2.6 配置文件输入

**攻击面**: 系统配置和权限

| 文件 | 风险点 |
|------|--------|
| `sa_profile/midi_server.json` | SAID 篡改、权限提升 |
| `sa_profile/midi_server.cfg` | 资源限制绕过 |
| `/system/etc/ueventd.config` | 设备节点权限配置 |

---

## 3. 敏感操作清单

### 3.1 权限相关操作

| 操作 | 权限要求 | 代码位置 |
|------|----------|----------|
| BLE 设备访问 | `ohos.permission.ACCESS_BLUETOOTH` | `native_midi.h:134` |
| USB 设备节点访问 | `/dev/snd/midiC*D*` (0660 system audio) | README_zh.md:359 |
| 服务拉起 | `SystemAbilityManager.LoadSystemAbility` | 系统框架层 |

**注意**: 服务层代码中**未发现**显式的权限校验调用，依赖 IPC 机制隐式校验。

---

### 3.2 内存操作

| 操作 | 风险 | 代码位置 | 防护 |
|------|------|----------|------|
| 共享内存 mmap | 内存映射攻击 | `midi_shared_ring.cpp:128` | FD 验证、大小检查 |
| memcpy_s | 缓冲区拷贝 | `midi_shared_ring.cpp:598` | 安全版本 |
| memset_s | 内存初始化 | `midi_client.cpp:49` | 安全版本 |
| strncpy_s | 字符串拷贝 | `midi_client.cpp:66` | 带长度限制 |

---

### 3.3 系统调用

| 调用 | 用途 | 代码位置 |
|------|------|----------|
| `mmap` | 共享内存映射 | `midi_shared_ring.cpp:128` |
| `futex` | 用户态同步 | `futex_tool.cpp` |
| `epoll` | IO 多路复用 | `midi_device_connection.cpp` |
| `timerfd` | 定时器 | `midi_device_connection.cpp` |

---

### 3.4 IPC 操作

| 操作 | 风险 | 代码位置 |
|------|------|----------|
| AddDeathRecipient | 死亡通知伪造 | `midi_service_controller.cpp:155` |
| ReadFileDescriptor | FD 传递攻击 | `midi_shared_ring.cpp:199` |
| Unmarshalling | 序列化攻击 | `midi_shared_ring.cpp:220-221` |

---

## 4. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           不可信区域 (Untrusted)                              │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   第三方 App     │    │   USB MIDI 设备  │    │   BLE MIDI 设备          │  │
│  │   (任意代码)     │    │   (物理接触)     │    │   (无线连接)             │  │
│  └────────┬────────┘    └────────┬────────┘    └────────────┬────────────┘  │
└───────────┼──────────────────────┼──────────────────────────┼────────────────┘
            │                      │                          │
            ▼                      ▼                          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           半可信区域 (Semi-Trusted)                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                    Native Framework (应用进程)                          │ │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐   │ │
│  │   │   OHMidi    │  │MidiClient   │  │     MidiSharedRing          │   │ │
│  │   │   C API     │  │   Private   │  │   (Client View)             │   │ │
│  │   └──────┬──────┘  └──────┬──────┘  └─────────────┬───────────────┘   │ │
│  └──────────┼────────────────┼───────────────────────┼───────────────────┘ │
└─────────────┼────────────────┼───────────────────────┼─────────────────────┘
              │                │                       │
              │ IPC (Binder)   │                       │ Shared Memory
              ▼                ▼                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           可信区域 (Trusted)                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                    System Service (系统服务进程)                        │ │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │ │
│  │  │MidiServiceCtrl  │  │MidiDeviceManager│  │     MidiSharedRing      │  │ │
│  │  │   (单例)        │  │                 │  │   (Server View, R/W)    │  │ │
│  │  └────────┬────────┘  └────────┬────────┘  └─────────────────────────┘  │ │
│  │           │                    │                                        │ │
│  │           ▼                    ▼                                        │ │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │ │
│  │  │   USB Driver    │  │   BLE Driver    │  │    HDI Interface        │  │ │
│  │  │   (HDI::Midi)   │  │  (GATT Client)  │  │                         │  │ │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

**边界说明:**
1. **App ↔ Framework**: Native API 调用边界，参数需要验证
2. **Framework ↔ Service**: IPC Binder 边界，IPC 数据需要验证
3. **Service ↔ Driver**: HDI/HAL 边界，硬件数据需要解析验证

---

## 5. 攻击路径示例

### 5.1 USB BadUSB 攻击路径

```
攻击者准备恶意 USB 设备
        ↓
插入 OpenHarmony 设备
        ↓
USB 驱动识别设备
        ↓
midi_framework 注册设备
        ↓
应用打开设备端口
        ↓
恶意设备发送畸形 MIDI/SysEx 数据
        ↓
服务端解析数据 → 缓冲区溢出/UAF
        ↓
代码执行或权限提升
```

**缓解措施**:
- VID/PID 白名单机制
- 输入数据严格边界检查
- 使用 `memcpy_s` 等安全函数

---

### 5.2 BLE MITM 攻击路径

```
攻击者在 BLE MIDI 设备和手机之间
        ↓
拦截 GATT 连接
        ↓
转发并篡改 UMP 数据
        ↓
注入畸形 UMP 包
        ↓
服务端 UMP 解析器崩溃
        ↓
拒绝服务或内存损坏
```

**缓解措施**:
- 强制使用 LE Secure Connections
- UMP 包长度验证
- 状态机完整性检查

---

### 5.3 IPC 伪造攻击路径

```
恶意应用伪造 IRemoteObject
        ↓
调用 CreateMidiInServer
        ↓
传递恶意 DeathRecipient
        ↓
触发 OnRemoteDied 回调
        ↓
执行恶意代码或触发 UAF
```

**缓解措施**:
- IPC 对象来源验证
- 死亡回调限制执行上下文
- 使用 weak_ptr 防止悬空

---

## 6. 风险矩阵

| 攻击面 | 威胁等级 | 利用难度 | 影响范围 | 缓解状态 |
|--------|----------|----------|----------|----------|
| USB 物理接入 | **HIGH** | Medium | 系统级 | 部分缓解 |
| BLE 无线接入 | **HIGH** | High | 服务级 | 部分缓解 |
| IPC 数据伪造 | **MEDIUM** | High | 服务级 | 已缓解 |
| 共享内存篡改 | **MEDIUM** | High | 进程级 | 已缓解 |
| N-API 参数 | LOW | Low | 应用级 | 已缓解 |
| 配置文件 | LOW | High | 系统级 | 依赖系统 |

**图例**:
- 威胁等级: HIGH(红色) / MEDIUM(黄色) / LOW(绿色)
- 缓解状态: 已缓解 / 部分缓解 / 未缓解

---

*下一章: [06_SecurityReview.md](06_SecurityReview.md) - 安全风险评估*
