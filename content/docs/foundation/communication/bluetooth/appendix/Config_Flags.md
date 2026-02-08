# 配置开关

> **目的**: 记录 Bluetooth 模块中的关键编译开关与运行时配置  
> **适用范围**: 功能调试、性能优化、定制化编译

## 编译时配置

### GN 可配置参数

| 参数 | 类型 | 默认值 | 文件位置 | 说明 |
|------|------|--------|----------|------|
| `bluetooth_service_resourceschedule` | `bool` | `true` | `bluetooth.gni:15` | 是否启用资源调度服务 |
| `bluetooth_kia_enable` | `bool` | `false` | `bluetooth.gni:16` | 是否启用 KIA 功能 |

**证据**: `bluetooth.gni:14-22`
```gn
declare_args() {
  bluetooth_service_resourceschedule = true
  bluetooth_kia_enable = false

  if (defined(global_parts_info) &&
      !defined(global_parts_info.resourceschedule_resource_schedule_service)) {
    bluetooth_service_resourceschedule = false
  }
}
```

### 条件编译宏

| 宏定义 | 用途 | 使用位置 |
|--------|------|----------|
| `ENABLE_NAPI_BLUETOOTH_MANAGER` | 启用蓝牙管理器 N-API | `native_module.cpp:96` |

**证据**: `native_module.cpp:92-100`
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
```

### Feature Flags 使用场景

#### KIA (Keep It Alive) 功能

当 `bluetooth_kia_enable = true` 时启用：

```cpp
// 可能的用途：
// - 保持蓝牙连接活跃
// - 阻止系统休眠
// - 电源管理相关
```

#### 资源调度服务

当 `bluetooth_service_resourceschedule = true` 时启用：

```cpp
// 可能的用途：
// - 蓝牙资源调度
// - 电量优化
// - 与系统资源管理器交互
```

---

## 运行时配置

### 系统参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `debug.bt.verbose` | `bool` | `false` | 启用详细蓝牙日志 |
| `debug.bt.hci` | `bool` | `false` | 启用 HCI 日志 |

**使用方法**:
```bash
# 设置参数
hdc shell setparam debug.bt.verbose true

# 查看参数
hdc shell getparam debug.bt.verbose
```

### 日志级别

| 级别 | 值 | 说明 |
|------|-----|------|
| DEBUG | 0 | 调试信息 |
| INFO | 1 | 一般信息 |
| WARN | 2 | 警告 |
| ERROR | 3 | 错误 |
| FATAL | 4 | 致命错误 |

### SA 加载超时

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `LOAD_SA_TIMEOUT_MS` | `4000` | SA 加载超时（毫秒） |

**证据**: `bluetooth_host.cpp:50`
```cpp
namespace {
constexpr int32_t LOAD_SA_TIMEOUT_MS = 4000;
}
```

---

## 蓝牙扫描参数

### BLE 扫描模式

| 模式 | 说明 |
|------|------|
| `SCAN_MODE_NONE` | 不扫描 |
| `SCAN_MODE_CONNECTABLE` | 可连接扫描 |
| `SCAN_MODE_GENERAL_DISCOVERABLE` | 一般可发现 |
| `SCAN_MODE_LIMITED_DISCOVERABLE` | 有限可发现 |
| `SCAN_MODE_BOTH` | 两者皆可 |

### BLE 扫描间隔/窗口

| 参数 | 范围 | 说明 |
|------|------|------|
| `interval` | 0x0004 - 0x4000 | 扫描间隔（单位：0.625ms） |
| `window` | 0x0004 - 0x4000 | 扫描窗口（单位：0.625ms） |

**计算公式**:
```
实际间隔 = interval * 0.625 ms
实际窗口 = window * 0.625 ms
```

**示例**:
```typescript
// 间隔 100ms，窗口 50ms
bluetoothBLE.startBLEScan({
  interval: 160,  // 100ms / 0.625 = 160
  window: 80       // 50ms / 0.625 = 80
});
```

---

## GATT 属性权限

| 权限 | 值 | 说明 |
|------|-----|------|
| `PERMISSION_READ` | 0x01 | 可读 |
| `PERMISSION_WRITE` | 0x02 | 可写 |
| `PERMISSION_READ_ENCRYPTED` | 0x04 | 加密读 |
| `PERMISSION_WRITE_ENCRYPTED` | 0x08 | 加密写 |
| `PERMISSION_READ_AUTHEN` | 0x10 | 认证读 |
| `PERMISSION_WRITE_AUTHEN` | 0x20 | 认证写 |

**组合示例**:
```cpp
// 可读可写
int permissions = PERMISSION_READ | PERMISSION_WRITE;

// 加密读写
int permissions = PERMISSION_READ_ENCRYPTED | PERMISSION_WRITE_ENCRYPTED;
```

---

## Profile 连接状态

### 连接状态码

| 状态 | 值 | 说明 |
|------|-----|------|
| `CONNECT_STATE_DISCONNECTED` | 0 | 断开 |
| `CONNECT_STATE_CONNECTING` | 1 | 连接中 |
| `CONNECT_STATE_CONNECTED` | 2 | 已连接 |
| `CONNECT_STATE_DISCONNECTING` | 3 | 断开中 |

### Profile 连接状态

| Profile | 状态码前缀 |
|---------|-----------|
| A2DP | `0x01` |
| HFP | `0x02` |
| HSP | `0x04` |
| PAN | `0x08` |
| PBAP | `0x10` |
| MAP | `0x20` |

---

## Bluetooth 状态码

| 状态 | 值 | 说明 |
|------|-----|------|
| `STATE_OFF` | 0 | 关闭 |
| `STATE_TURNING_ON` | 1 | 开启中 |
| `STATE_ON` | 2 | 开启 |
| `STATE_TURNING_OFF` | 3 | 关闭中 |
| `STATE_BLE_TURNING_ON` | 4 | BLE 开启中 |
| `STATE_BLE_ON` | 5 | BLE 开启 |
| `STATE_BLE_TURNING_OFF` | 6 | BLE 关闭中 |

---

## 蓝牙传输类型

| 类型 | 值 | 说明 |
|------|-----|------|
| `BT_TRANSPORT_BREDR` | 0 | 经典蓝牙 |
| `BT_TRANSPORT_LE` | 1 | 低功耗蓝牙 |
| `BT_TRANSPORT_AUTO` | 2 | 自动选择 |

---

## 系统能力

### SystemCapability

| Capability | 说明 |
|------------|------|
| `SystemCapability.Communication.Bluetooth.Core` | 蓝牙核心能力 |
| `SystemCapability.Communication.Bluetooth.Lite` | 蓝牙轻量能力 |

**证据**: `bundle.json:41-42`
```json
"syscap": [
  "SystemCapability.Communication.Bluetooth.Core",
  "SystemCapability.Communication.Bluetooth.Lite"
]
```

---

## HID 设备类型

| 类型 | 值 | 说明 |
|------|-----|------|
| `HID_DEVICE_TYPE_MOUSE` | 1 | 鼠标 |
| `HID_DEVICE_TYPE_KEYBOARD` | 2 | 键盘 |
| `HID_DEVICE_TYPE_GAMEPAD` | 4 | 游戏手柄 |
| `HID_DEVICE_TYPE_REMOTE` | 8 | 遥控器 |
| `HID_DEVICE_TYPE_OTHER` | 16 | 其他 |

---

## A2DP 编解码器

| 编解码器 | 值 | 说明 |
|----------|-----|------|
| `A2DP_CODEC_SBC` | 0x00 | SBC |
| `A2DP_CODEC_AAC` | 0x02 | AAC |
| `A2DP_CODEC_LDAC` | 0x06 | LDAC |
| `A2DP_CODEC_APTX` | 0x04 | aptX |
| `A2DP_CODEC_APTX_HD` | 0x05 | aptX HD |

---

[返回 SUMMARY](../SUMMARY.md)
