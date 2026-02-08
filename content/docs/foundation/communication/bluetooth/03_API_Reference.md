# API 参考

> **目的**: 提供 Bluetooth 模块所有对外 API 的完整参考  
> **适用范围**: 应用开发、API 迁移、接口审查

## API 总览

### 按能力分类

| 能力分类 | N-API 模块 | C API | 主要类/接口 |
|----------|------------|-------|-------------|
| **蓝牙开关控制** | `bluetooth` | `oh_bt_gap.h` | `BluetoothHost` |
| **BLE 广播** | `bluetooth.ble` | `oh_bt_gatt.h` | `BleAdvertiser` |
| **BLE 扫描** | `bluetooth.ble` | `oh_bt_gatt.h` | `BleCentralManager` |
| **GATT 客户端** | `bluetooth.ble` | `oh_bt_gatt_client.h` | `GattClient` |
| **GATT 服务端** | `bluetooth.ble` | `oh_bt_gatt_server.h` | `GattServer` |
| **A2DP 音频** | `bluetooth.a2dp` | - | `A2dpSource`, `A2dpSink` |
| **HFP 免提** | `bluetooth.hfp` | - | `HandsFreeUnit`, `HandsFreeAudioGateway` |
| **HID 设备** | `bluetooth.hid` | - | `BluetoothHidHost`, `BluetoothHidDevice` |
| **AVRCP 遥控** | `bluetooth.a2dp` | - | `AvrcpController`, `AvrcpTarget` |
| **PAN 网络** | `bluetooth.pan` | - | `BluetoothPan` |
| **OPP 推送** | `bluetooth.opp` | - | `BluetoothOpp` |
| **PBAP 电话簿** | `bluetooth.pbap` | - | `BluetoothPbapPse` |
| **MAP 消息** | `bluetooth.map` | - | `BluetoothMapMse` |
| **SPP 串口** | `bluetooth.socket` | `oh_bt_spp.h` | `BluetoothSocket` |
| **设备连接** | `bluetooth.connection` | - | `BluetoothConnection` |
| **权限管理** | `bluetooth.access` | - | `BluetoothAccess` |

### 按系统分类

| 系统类型 | 支持的 API | 说明 |
|----------|------------|------|
| **Standard** | N-API + C API | 完整功能集 |
| **Mini** | 部分 C API | 仅 BLE 相关 |
| **Small** | 部分 C API | 仅 BLE 相关 |

## N-API 模块清单

### 模块注册点

| 模块 | 注册文件 | 注册行号 | JS 命名空间 |
|------|----------|----------|-------------|
| BLE | `native_module_ble.cpp` | 70 | `bluetooth.ble` |
| GAP/Host | `native_module.cpp` | 109 | `bluetooth` / `bluetoothManager` |
| A2DP | `native_module_a2dp.cpp` | 60 | `bluetooth.a2dp` |
| HFP | `native_module_hfp.cpp` | 65 | `bluetooth.hfp` |
| HID | `native_module_hid.cpp` | 65 | `bluetooth.hid` |
| Connection | `native_module_connection.cpp` | 60 | `bluetooth.connection` |
| Access | `native_module_access.cpp` | 63 | `bluetooth.access` |
| Socket | `native_module_socket.cpp` | 60 | `bluetooth.socket` |
| PAN | `native_module_pan.cpp` | 62 | `bluetooth.pan` |
| BaseProfile | `native_module_base_profile.cpp` | 61 | `bluetooth.baseProfile` |
| Constant | `native_module_constant.cpp` | 61 | `bluetooth.constant` |
| Common | `module_common.cpp` | 61 | `bluetooth.common` |
| OPP | `native_module_opp.cpp` | - | `bluetooth.opp` |
| PBAP | `native_module_pbap.cpp` | - | `bluetooth.pbap` |
| MAP | `native_module_map.cpp` | - | `bluetooth.map` |
| AudioManager | `native_module_audio_manager.cpp` | - | `bluetooth.audioManager` |

### N-API 注册模式

所有 N-API 模块遵循统一注册模式：

**证据**: `native_module.cpp:92-110`
```cpp
static napi_module bluetoothModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = NULL,
    .nm_register_func = Init,
#ifdef ENABLE_NAPI_BLUETOOTH_MANAGER
    .nm_modname = "bluetoothManager",
#else
    .nm_modname = "bluetooth",
#endif
    .nm_priv = ((void *)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    HILOGI("Register bluetoothModule nm_modname:%{public}s", bluetoothModule.nm_modname);
    napi_module_register(&bluetoothModule);
}
```

## 核心 API 示例

### 蓝牙开关控制

| API | 说明 | 同步/异步 | 实现文件 |
|-----|------|----------|----------|
| `enableBluetooth()` | 开启经典蓝牙 | 异步 (Promise) | `native_module.cpp:63` |
| `disableBluetooth()` | 关闭经典蓝牙 | 异步 (Promise) | `native_module.cpp:63` |
| `enableBluetooth()` | 开启 BLE | 异步 (Promise) | `native_module.cpp:63` |
| `disableBluetooth()` | 关闭 BLE | 异步 (Promise) | `native_module.cpp:63` |
| `getState()` | 获取蓝牙状态 | 同步 | `native_module.cpp:75` |

### BLE 广播

| API | 说明 | 同步/异步 | 实现文件 |
|-----|------|----------|----------|
| `startAdvertising()` | 启动 BLE 广播 | 异步 (Callback) | `native_module_ble.cpp` |
| `stopAdvertising()` | 停止 BLE 广播 | 异步 (Callback) | `native_module_ble.cpp` |
| `setAdvertisingData()` | 设置广播数据 | 同步 | `native_module_ble.cpp` |
| `setScanResponseData()` | 设置扫描响应 | 同步 | `native_module_ble.cpp` |

### BLE 扫描

| API | 说明 | 同步/异步 | 实现文件 |
|-----|------|----------|----------|
| `startBLEScan()` | 启动 BLE 扫描 | 异步 (Callback) | `native_module_ble.cpp` |
| `stopBLEScan()` | 停止 BLE 扫描 | 异步 (Callback) | `native_module_ble.cpp` |
| `setBLEScanParams()` | 设置扫描参数 | 同步 | `native_module_ble.cpp` |

### GATT 客户端

| API | 说明 | 同步/异步 | 实现文件 |
|-----|------|----------|----------|
| `connect()` | 连接 GATT 设备 | 异步 (Callback) | `native_module_ble.cpp` |
| `disconnect()` | 断开连接 | 异步 (Callback) | `native_module_ble.cpp` |
| `discoverServices()` | 发现服务 | 异步 (Promise) | `native_module_ble.cpp` |
| `readCharacteristicValue()` | 读取特征值 | 异步 (Callback) | `native_module_ble.cpp` |
| `writeCharacteristicValue()` | 写入特征值 | 异步 (Callback) | `native_module_ble.cpp` |
| `setNotifyCharacteristic()` | 设置通知 | 异步 (Callback) | `native_module_ble.cpp` |

### GATT 服务端

| API | 说明 | 同步/异步 | 实现文件 |
|-----|------|----------|----------|
| `addService()` | 添加服务 | 同步 | `native_module_ble.cpp` |
| `addCharacteristic()` | 添加特征值 | 同步 | `native_module_ble.cpp` |
| `addDescriptor()` | 添加描述符 | 同步 | `native_module_ble.cpp` |
| `startService()` | 启动服务 | 同步 | `native_module_ble.cpp` |
| `notifyCharacteristicValue()` | 发送通知 | 同步 | `native_module_ble.cpp` |

## 错误码定义

### 通用错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | `BluetoothErrorCode::SUCCESS` | 成功 |
| -1 | `BluetoothErrorCode::FAIL` | 失败 |
| 401 | `BluetoothErrorCode::PARAM_ERROR` | 参数错误 |
| 10001 | `BluetoothErrorCode::NOT_INIT` | 未初始化 |
| 10002 | `BluetoothErrorCode::NOT_ENABLED` | 未开启 |
| 10003 | `BluetoothErrorCode::INVALID_PARAM` | 无效参数 |
| 10004 | `BluetoothErrorCode::PROFILE_NOT_INIT` | Profile 未初始化 |
| 10005 | `BluetoothErrorCode::PROFILE_NOT_CONNECTED` | Profile 未连接 |
| 10006 | `BluetoothErrorCode::DEVICE_NOT_FOUND` | 设备未找到 |
| 10007 | `BluetoothErrorCode::CONNECTION_FAILED` | 连接失败 |
| 10008 | `BluetoothErrorCode::AUTH_REJECTED` | 认证被拒 |
| 10009 | `BluetoothErrorCode::AUTH_TIMEOUT` | 认证超时 |

**证据**: `bluetooth_errorcode.h`

### Profile 专用错误码

各 Profile 可能有额外的专用错误码，定义在对应的接口头文件中。

## 权限要求

### 权限声明

使用蓝牙 API 需要在应用的配置文件中声明权限：

| 权限 | 用途 | 必选/可选 |
|------|------|----------|
| `ohos.permission.USE_BLUETOOTH` | 使用蓝牙基础功能 | 必选 |
| `ohos.permission.ACCESS_BLUETOOTH` | 访问蓝牙设备信息 | 必选 |
| `ohos.permission.DISCOVER_BLUETOOTH` | 发现附近蓝牙设备 | 可选 |
| `ohos.permission.CONNECT_BLUETOOTH` | 连接已配对设备 | 可选 |
| `ohos.permission.MANAGE_BLUETOOTH` | 管理蓝牙（配对/取消配对） | 可选 |

### 权限检查位置

权限检查在 N-API 层进行：

**证据**: `native_module_*.cpp` 中的参数校验

## C API 参考

### C API 头文件

| 头文件 | 说明 | 适用范围 |
|--------|------|----------|
| `oh_bt_gap.h` | GAP 基础 API | Mini/Small/Standard |
| `oh_bt_gatt.h` | GATT 通用 API | Mini/Small/Standard |
| `oh_bt_gatt_client.h` | GATT 客户端 API | Mini/Small/Standard |
| `oh_bt_gatt_server.h` | GATT 服务端 API | Mini/Small/Standard |
| `oh_bt_spp.h` | SPP Socket API | Standard |
| `oh_bt_def.h` | 类型定义与常量 | Mini/Small/Standard |

### C API 示例

**蓝牙开关** (`oh_bt_gap.h`):
```c
// 开启经典蓝牙
bool EnableBt(void);

// 关闭经典蓝牙
bool DisableBt(void);

// 开启 BLE
bool EnableBle(void);

// 关闭 BLE
bool DisableBle(void);

// 获取蓝牙状态
int GetBtState();

// 检查 BLE 是否开启
bool IsBleEnabled();
```

**GATT 服务端** (`oh_bt_gatt_server.h`):
```c
// 初始化 BT 协议栈
int InitBtStack(void);

// 开启 BT 协议栈
int EnableBtStack(void);

// 注册 GATT 服务器
int BleGattsRegister(BtUuid appUuid);

// 添加服务
int BleGattsAddService(int serverId, BtUuid srvcUuid, bool isPrimary, int number);

// 添加特征值
int BleGattsAddCharacteristic(int serverId, int srvcHandle, BtUuid characUuid, 
                              int properties, int permissions);

// 添加描述符
int BleGattsAddDescriptor(int serverId, int srvcHandle, BtUuid descUuid, int permissions);

// 启动服务
int BleGattsStartService(int serverId, int srvcHandle);
```

---

**子章节**: 
- [N-API 详细参考](03_NAPI_Reference.md) → 各 N-API 模块完整 API 清单
- [C API 参考](03_CAPI_Reference.md) → C API 详细说明

**下一步**: [构建系统](04_Build_System.md) → 了解编译配置与产物
