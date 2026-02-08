# 内部 API

> connectivity_cangjie_wrapper 模块接口、依赖方向与稳定性标注

## 概述

本文档描述项目内部的模块间接口，用于理解模块依赖关系和接口稳定性。

## 模块依赖方向

```
┌─────────────────────────────────────────────────────────────────┐
│                        模块依赖关系图                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    kit.ConnectivityKit (稳定)                    │
│                           │                                      │
│          ┌────────────────┼────────────────┐                    │
│          ▼                ▼                ▼                    │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│   │   ble       │  │  wifi_mgr   │  │  a2dp/hfp   │            │
│   │  (稳定)     │  │   (稳定)    │  │   (稳定)    │            │
│   └─────────────┘  └─────────────┘  └─────────────┘            │
│          │                │                │                    │
│          │                │                │                    │
│          ▼                ▼                ▼                    │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│   │ bluetooth   │  │  constant   │  │ base_profile│            │
│   │  (基础)     │  │  (常量)     │  │  (框架)     │            │
│   └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 内部模块接口

### bluetooth (基础模块)

**职责**: 提供基础日志和错误处理功能

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| `BLUETOOTH_LOG` | `HilogChannel` | 稳定 | 蓝牙日志通道 |
| `OPERATION_FAILED` | `Int32` | 稳定 | 操作失败错误码 |
| `checkRet(errorCode: Int32)` | `func` | 稳定 | 错误码检查函数 |
| `getErrorMsg(code: Int32)` | `func` | 稳定 | 获取错误消息 |
| `CallbackController` | `class` | 稳定 | 回调控制器 |

**依赖关系**:

```
bluetooth (基础)
├── cangjie_ark_interop:ohos.business_exception
├── cangjie_ark_interop:ohos.callback_invoke
├── cangjie_ark_interop:ohos.ffi
├── cangjie_ark_interop:ohos.labels
└── hiviewdfx_cangjie_wrapper:ohos.hilog
```

**文件位置**: `ohos/bluetooth/BUILD.gn`

### ble (BLE 模块)

**职责**: BLE 扫描、广播、GATT 服务

**内部依赖**:

```cj
import ohos.bluetooth.{ BLUETOOTH_LOG, OPERATION_FAILED, checkRet, getErrorMsg, CallbackController }
import ohos.business_exception.BusinessException
import ohos.callback_invoke.{ Callback1Argument, CallbackObject }
import ohos.ffi.{ Callback1Param, safeMalloc, cArr2cjArr }
```

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| `createGattServer()` | `func` | 稳定 | 创建 GATT 服务端 |
| `createGattClientDevice(deviceId: String)` | `func` | 稳定 | 创建 GATT 客户端 |
| `startBleScanning(filters, options)` | `func` | 稳定 | 开始 BLE 扫描 |
| `stopBleScanning()` | `func` | 稳定 | 停止 BLE 扫描 |
| `startAdvertising(setting, advData, advResponse)` | `func` | 稳定 | 开始广播 |
| `stopAdvertising()` | `func` | 稳定 | 停止广播 |
| `on(eventType, callback)` | `func` | 稳定 | 注册回调 |
| `off(eventType, callback)` | `func` | 稳定 | 取消回调 |
| `GattServer` | `class` | 稳定 | GATT 服务端类 |
| `GattClientDevice` | `class` | 稳定 | GATT 客户端类 |

**依赖关系**:

```
ble
├── ../:ohos.bluetooth (基础)
├── ../constant:ohos.bluetooth.constant (常量)
└── external: bluetooth:cj_bluetooth_ble_ffi (FFI)
```

**文件位置**: `ohos/bluetooth/ble/BUILD.gn`

### constant (常量模块)

**职责**: 定义蓝牙相关枚举常量

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| `ProfileConnectionState` | `enum` | 稳定 | Profile 连接状态 |
| [其他枚举] | `enum` | 稳定 | 其他常量 |

**依赖关系**:

```
constant
└── cangjie_ark_interop:ohos.labels (APILevel 注解)
```

**文件位置**: `ohos/bluetooth/constant/BUILD.gn`

### a2dp (A2DP 模块)

**职责**: 音频分发 Profile

**内部依赖**:

```cj
import ohos.bluetooth.{ BLUETOOTH_LOG, ... }
import ohos.business_exception.BusinessException
import ohos.callback_invoke.{ Callback1Argument, CallbackObject }
```

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| [TODO: 补充具体 API] | | 稳定 | |

**依赖关系**:

```
a2dp
├── ../:ohos.bluetooth (基础)
├── ../constant:ohos.bluetooth.constant (常量)
├── ../base_profile:ohos.bluetooth.base_profile (框架)
└── external: bluetooth:cj_bluetooth_a2dp_ffi (FFI)
```

**文件位置**: `ohos/bluetooth/a2dp/BUILD.gn`

### hfp (HFP 模块)

**职责**: 免提 Profile

**依赖关系**:

```
hfp
├── ../:ohos.bluetooth (基础)
├── ../constant:ohos.bluetooth.constant (常量)
├── ../base_profile:ohos.bluetooth.base_profile (框架)
└── external: bluetooth:cj_bluetooth_hfp_ffi (FFI)
```

**文件位置**: `ohos/bluetooth/hfp/BUILD.gn`

### base_profile (基础 Profile 框架)

**职责**: 提供 Profile 的基础框架接口

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| [TODO: 补充具体 API] | | 稳定 | |

**依赖关系**:

```
base_profile
└── ../constant:ohos.bluetooth.constant (常量)
```

**文件位置**: `ohos/bluetooth/base_profile/BUILD.gn`

### connection (连接管理模块)

**职责**: 蓝牙设备连接管理

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| [TODO: 补充具体 API] | | 稳定 | |

**依赖关系**:

```
connection
├── ../:ohos.bluetooth (基础)
└── external: bluetooth:cj_bluetooth_connection_ffi (FFI)
```

**文件位置**: `ohos/bluetooth/connection/BUILD.gn`

### wifi_manager (WLAN 模块)

**职责**: WLAN P2P 功能

**内部依赖**:

```cj
import ohos.business_exception.BusinessException
import ohos.callback_invoke.{ Callback1Argument, CallbackObject }
import ohos.ffi.{ SUCCESS_CODE, cArr2cjArr, Callback1Param }
import ohos.hilog.HilogChannel
```

**导出符号**:

| 符号 | 类型 | 稳定性 | 说明 |
|------|------|--------|------|
| `isWifiActive()` | `func` | 稳定 | 查询 WiFi 状态 |
| `getScanInfoList()` | `func` | 稳定 | 获取扫描结果 |
| `p2pConnect(config)` | `func` | 稳定 | P2P 连接 |
| `p2pCancelConnect()` | `func` | 稳定 | 取消 P2P 连接 |
| `startDiscoverDevices()` | `func` | 稳定 | 开始发现设备 |
| `stopDiscoverDevices()` | `func` | 稳定 | 停止发现设备 |
| `on(eventType, callback)` | `func` | 稳定 | 注册回调 |
| `off(eventType, callback)` | `func` | 稳定 | 取消回调 |

**依赖关系**:

```
wifi_manager
├── cangjie_ark_interop:ohos.business_exception
├── cangjie_ark_interop:ohos.callback_invoke
├── cangjie_ark_interop:ohos.ffi
├── cangjie_ark_interop:ohos.labels
├── hiviewdfx_cangjie_wrapper:ohos.hilog
└── external: wifi:cj_wifi_ffi (FFI)
```

**文件位置**: `ohos/wifi_manager/BUILD.gn`

## 稳定性标注

### 稳定性等级说明

| 等级 | 说明 | 变更策略 |
|------|------|----------|
| **稳定 (Stable)** | 公开 API，已正式发布 | 遵循语义化版本 |
| **实验性 (Experimental)** | 新增 API，可能变更 | 可能在后续版本中修改或移除 |
| **内部 (Internal)** | 仅供模块内部使用 | 随时可能变更 |

### 稳定性判断依据

1. **公开导出**: 通过 `public import` 或 `public func/class/enum` 导出 → 稳定
2. **private/内部符号**: 仅模块内使用 → 内部
3. **@APILevel 注解**: 有 `since` 版本标注 → 稳定

## 接口替换点

| 接口类型 | 替换点 | 说明 |
|----------|--------|------|
| **FFI 函数** | `external_deps` 中的 `cj_*_ffi` | 可替换底层实现 |
| **日志** | `ohos.hilog` | 可替换为其他日志系统 |
| **异常** | `BusinessException` | 遵循 cangjie_ark_interop 定义 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Project_Overview.md) | 项目定位与核心能力 |
| [目录结构](./02_Directory_Structure.md) | 模块划分与文件布局 |
| [架构说明](./03_Architecture.md) | 组件关系与数据流 |
| [N-API 参考](./04_N-API_Reference.md) | 对外 API 详细说明 |
| [构建系统](./06_Build_System.md) | GN Targets 与产物 |
