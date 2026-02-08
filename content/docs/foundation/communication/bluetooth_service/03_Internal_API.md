# 内部 API

## 文档信息

- **目的**: 介绍模块间接口定义、依赖关系和接口稳定性
- **适用范围**: 开发者理解模块交互、架构师评估接口设计
- **关键结论**:
  1. 接口定义清晰分层
  2. 使用智能指针管理生命周期
  3. Observer 模式用于事件通知
  4. 部分接口标记为 BLUETOOTH_API（公开）
- **相关文档**: [01_Directory_Structure](01_Directory_Structure.md), [02_Architecture](02_Architecture.md)

---

## 接口稳定性说明

### 接口分类

| 稳定性 | 标记 | 说明 |
|---------|------|------|
| **公开** | `BLUETOOTH_API` | 可被外部组件调用 |
| **内部** | 无标记 | 仅组件内部使用 |
| **实验性** | 推断 | 可能变更，需谨慎使用 |

**判断依据**:
- 头文件位置 (`include/` vs `src/`)
- 宏标记 (`BLUETOOTH_API`)
- 注释说明

---

## IPC 层接口

### BluetoothHostStub

**文件**: `services/bluetooth/ipc/include/bluetooth_host_stub.h`

**职责**: IPC 通信 Stub，定义 IPC 接口

**关键方法**: 与 `IBluetoothHost` 一致（来自蓝牙框架）

**实现类**: `BluetoothHostServer` (`services/bluetooth/server/include/bluetooth_host_server.h:33`)

### Profile Stub 列表

| Profile | Stub 类 | 文件 |
|---------|----------|------|
| Host | `BluetoothHostStub` | `ipc/include/bluetooth_host_stub.h` |
| GATT Server | `BluetoothGattServerStub` | `ipc/include/bluetooth_gatt_server_stub.h` |
| GATT Client | `BluetoothGattClientStub` | `ipc/include/bluetooth_gatt_client_stub.h` |
| BLE Advertiser | `BluetoothBleAdvertiserStub` | `ipc/include/bluetooth_ble_advertiser_stub.h` |
| BLE Central Manager | `BluetoothBleCentralManagerStub` | `ipc/include/bluetooth_ble_central_manager_stub.h` |
| A2DP Source | `BluetoothA2dpSourceStub` | `ipc/include/bluetooth_a2dp_source_stub.h` |
| A2DP Sink | `BluetoothA2dpSinkStub` | `ipc/include/bluetooth_a2dp_sink_stub.h` |
| AVRCP CT | `BluetoothAvrcpCtStub` | `ipc/include/bluetooth_avrcp_ct_stub.h` |
| AVRCP TG | `BluetoothAvrcpTgStub` | `ipc/include/bluetooth_avrcp_tg_stub.h` |
| HFP AG | `BluetoothHfpAgStub` | `ipc/include/bluetooth_hfp_ag_stub.h` |
| HFP HF | `BluetoothHfpHfStub` | `ipc/include/bluetooth_hfp_hf_stub.h` |
| HID Host | `BluetoothHidHostStub` | `ipc/include/bluetooth_hid_host_stub.h` |
| PAN | `BluetoothPanStub` | `ipc/include/bluetooth_pan_stub.h` |
| Socket | `BluetoothSocketStub` | `ipc/include/bluetooth_socket_stub.h` |

**证据**: `services/bluetooth/ipc/include/*.h`

---

## Service 层接口

### IAdapterManager

**文件**: `services/bluetooth/service/include/interface_adapter_manager.h:103`

**稳定性**: 公开（`BLUETOOTH_API` 标记）

**职责**: 适配器管理器接口，管理 Classic 和 BLE 适配器

**关键方法**:

```cpp
class BLUETOOTH_API IAdapterManager {
public:
    static IAdapterManager *GetInstance();
    virtual void Reset() const = 0;
    virtual bool Start() = 0;
    virtual void Stop() const = 0;
    virtual bool Enable(const BTTransport transport) const = 0;
    virtual bool Disable(const BTTransport transport) const = 0;
    virtual BTStateID GetState(const BTTransport transport) const = 0;
    virtual bool RegisterStateObserver(IAdapterStateObserver &observer) const = 0;
    virtual std::shared_ptr<IAdapterClassic> GetClassicAdapterInterface() const = 0;
    virtual std::shared_ptr<IAdapterBle> GetBleAdapterInterface() const = 0;
};
```

### IAdapterClassic

**文件**: `services/bluetooth/service/include/interface_adapter_classic.h`

**稳定性**: 内部

**职责**: Classic 适配器接口

### IAdapterBle

**文件**: `services/bluetooth/service/include/interface_adapter_ble.h`

**稳定性**: 内部

**职责**: BLE 适配器接口

### Profile 接口

| 接口 | 文件 | 稳定性 |
|------|------|---------|
| `IProfile` | `interface_profile.h` | 内部 |
| `IProfileA2dpSrc` | `interface_profile_a2dp_src.h` | 内部 |
| `IProfileGattServer` | `interface_profile_gatt_server.h` | 内部 |
| `IProfileGattClient` | `interface_profile_gatt_client.h` | 内部 |
| `IProfileSocket` | `interface_profile_socket.h` | 内部 |

**证据**: `services/bluetooth/service/include/interface_*.h`

---

## Observer 接口

### IAdapterStateObserver

**文件**: `services/bluetooth/service/include/interface_adapter_manager.h:47`

**职责**: 适配器状态变化观察者

**回调**:
```cpp
virtual void OnStateChange(const BTTransport transport, const BTStateID state) = 0;
```

### ISystemStateObserver

**文件**: `services/bluetooth/service/include/interface_adapter_manager.h:79`

**职责**: 系统状态变化观察者

**回调**:
```cpp
virtual void OnSystemStateChange(const BTSystemState state) = 0;
```

### 其他 Observer

| Observer | 用途 | 文件 |
|----------|------|------|
| `IBluetoothHostObserver` | Host 事件 | `bluetooth_host_stub.h` |
| `IBluetoothBlePeripheralObserver` | BLE 外设事件 | `bluetooth_host_stub.h` |
| `IBluetoothRemoteDeviceObserver` | 远程设备事件 | `bluetooth_host_stub.h` |

**证据**: `services/bluetooth/ipc/include/*_observer_proxy.h`

---

## 权限接口

### PermissionManager

**文件**: `services/bluetooth/service/src/permission/permission_manager.h:25`

**稳定性**: 内部

**职责**: 权限管理器

**关键方法**:

```cpp
class PermissionManager {
public:
    static std::string GetCallingName();
    static std::string GetCallingName(const uint32_t& tokenId);
    static bool IsSystemHap();
    static bool IsSystemHap(const uint64_t& fullTokenId);
};
```

**证据**: `services/bluetooth/service/src/permission/permission_manager.h:25-36`

### AuthCenter

**文件**: `services/bluetooth/service/src/permission/auth_center.h`

**稳定性**: 内部

**职责**: 认证中心，具体权限检查

---

## Stack 层接口

### 推断接口

Stack 层主要通过 C API 或内部类接口暴露：

| 模块 | 推断接口 |
|------|----------|
| HCI | `HciInterface` |
| L2CAP | `L2capInterface` |
| GAP | `GapInterface` |
| SMP | `SmpInterface` |

**证据**: `services/bluetooth/stack/include/` (头文件列表)

---

## 依赖关系

### 层次依赖

```
Server 层
    ↓ 依赖
Service 层
    ↓ 依赖
Stack 层
    ↓ 依赖
Hardware 层
```

### Server → Service 依赖

**证据**: `services/bluetooth/server/BUILD.gn:29`

```gn
include_dirs = [
  "include",
  "//foundation/communication/bluetooth_service/services/bluetooth/service/include",
  # ...
]
```

### Service → Stack 依赖

**证据**: `services/bluetooth/service/BUILD.gn:343`

```gn
deps = [
  "$PART_DIR/external:btdummy",
  "$PART_DIR/stack:btstack",
]
```

### Stack → Hardware 依赖

**证据**: 推断（协议栈通过 HDI 调用硬件）

---

## 可替换点

### Profile 插件化

**设计**: 每个 Profile 独立编译

**Feature Flags 控制**:
- `bluetooth_service_a2dp_source_feature`
- `bluetooth_service_hfp_ag_feature`
- 等

**替换方式**: 通过 Feature flags 启用/禁用

### HDI 接口抽象

**设计**: `services/bluetooth/hardware/` 封装 HDI

**替换方式**: 实现不同的 HDI HAL

---

## 接口版本控制

### 推断版本管理

**基于 OpenHarmony 机制**:
- 接口版本通过 HDI 版本控制
- SA ID 固定（1130）
- IPC 接口通过 HIDL/IPC 机制兼容

---

## 总结

内部 API 设计清晰，具有以下特点：

1. **层次化**: Server → Service → Stack → Hardware
2. **Observer 模式**: 大量使用观察者模式通知事件
3. **智能指针**: 使用 `std::shared_ptr` 管理生命周期
4. **Feature Flags**: Profile 独立配置
5. **权限检查**: 统一的权限管理机制

**接口稳定性**:
- 公开接口: `BLUETOOTH_API` 标记
- 内部接口: 无标记，可能变更

**相关文档**:
- 架构详解: [02_Architecture](02_Architecture.md)
- 目录结构: [01_Directory_Structure](01_Directory_Structure.md)
- 构建系统: [04_GN_Targets](04_GN_Targets.md)
