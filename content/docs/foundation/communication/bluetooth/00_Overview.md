# 项目概览

> **目的**: 快速理解 OpenHarmony Bluetooth 模块的定位、能力边界和技术栈  
> **适用范围**: 新人入门、架构设计参考、API 选型

## 项目定位

### 核心定位

OpenHarmony **Bluetooth 模块**是 OpenHarmony 系统的蓝牙通信子系统核心框架，提供以下能力：

| 能力分类 | 具体功能 | 状态 |
|----------|----------|------|
| **BLE（低功耗蓝牙）** | 广播、扫描、GATT 客户端/服务端 | ✅ 完整 |
| **Classic BT（经典蓝牙）** | A2DP、HFP、HID、SPP 等 Profile | ✅ 完整 |
| **基础 GAP 操作** | 设备发现、配对管理、连接控制 | ✅ 完整 |
| **系统集成** | SA 1130 生命周期管理、权限控制 | ✅ 完整 |

### 技术约束

| 约束 | 说明 | 证据 |
|------|------|------|
| 必须使用 C 语言编译 | 模块核心代码为 C/C++ | `README.md: "The Bluetooth module must be compiled in C language"` |
| 系统能力 | `SystemCapability.Communication.Bluetooth.Core` | `bundle.json:line 41` |
| 支持系统类型 | Standard（标准系统） | `bundle.json:line 48` |

## 模块边界

### 对外边界（暴露给应用层）

```
应用层 (ArkTS/JS/C++)
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│                  对外 API 层                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │   N-API     │  │   C API     │  │   FFI       │      │
│  │ (ArkTS/JS)  │  │   (C/C++)   │  │  (Rust/JS)  │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
└─────────────────────────────────────────────────────────┘
```

### 内部边界（框架内部）

```
对外 API 层
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│                  内部框架层                               │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                    IPC 层                           │ │
│  │   Proxy → MessageParcel → SAMGR → SA 1130         │ │
│  └─────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              Profile 管理与适配层                    │ │
│  │   C Adapter → Profile Impl → IPC Interface        │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 外部依赖（不在本仓库）

| 依赖项 | 说明 | 用途 |
|--------|------|------|
| SAMGR | System Ability Manager | SA 1130 注册与 IPC 路由 |
| Bluetooth Service | 蓝牙协议栈服务（独立仓库） | SA 1130 服务端实现 |
| IPC Framework | 进程间通信框架 | Proxy/Stub 通信 |
| Ability Runtime | 能力运行时 | N-API 上下文管理 |

## 核心能力详解

### 1. BLE（Bluetooth Low Energy）

| 子模块 | 能力 | 关键类/接口 |
|--------|------|-------------|
| 广播 | 发起 BLE 广播、设置广播数据 | `BleAdvertiser` |
| 扫描 | BLE 设备发现、过滤 | `BleCentralManager` |
| GATT 客户端 | 发现服务、读写特征值 | `GattClient` |
| GATT 服务端 | 发布服务、接收读写请求 | `GattServer` |

### 2. Classic Bluetooth Profiles

| Profile | 能力 | 关键类/接口 |
|---------|------|-------------|
| A2DP | 音频流传输（源端/接收端） | `A2dpSource`, `A2dpSink` |
| HFP | 免提通话控制 | `HandsFreeUnit` (HF), `HandsFreeAudioGateway` (AG) |
| HID | 人体学设备连接 | `BluetoothHidHost`, `BluetoothHidDevice` |
| AVRCP | 音视频远程控制 | `AvrcpController`, `AvrcpTarget` |
| PAN | 个人局域网共享 | `BluetoothPan` |
| OPP | 对象推送 | `BluetoothOpp` |
| PBAP | 电话簿访问 | `BluetoothPbapPse` |
| MAP | 消息访问 | `BluetoothMapMse` |
| SPP | 串口协议 | `BluetoothSocket` |

### 3. 基础能力

| 能力 | 说明 | 关键 API |
|------|------|----------|
| 设备发现 | 扫描周围蓝牙设备 | `StartBtDiscovery()`, `CancelBtDiscovery()` |
| 配对管理 | 设备配对与取消配对 | `CreateBond()`, `RemovePair()` |
| 连接管理 | Profile 连接控制 | `Connect()`, `Disconnect()` |
| 本地设备信息 | 获取/设置本地名称、地址、类型 | `GetLocalName()`, `SetLocalDeviceClass()` |

## 运行环境

### 编译环境

| 环境 | 要求 |
|------|------|
| 构建系统 | GN (Generate Ninja) |
| 编译工具链 | LLVM/Clang, GCC |
| Python | 3.x（用于 GN 工具链） |

### 运行时依赖

| 依赖 | 最小版本 | 说明 |
|------|----------|------|
| OpenHarmony SDK | 4.0+ | N-API 运行支持 |
| SAMGR | - | 系统能力管理器 |
| Bluetooth Service | - | SA 1130 服务端 |

### 目标硬件

| 系统类型 | 支持情况 | 说明 |
|----------|----------|------|
| Standard（标准系统） | ✅ 支持 | 手机、平板、PC 等富设备 |
| Mini（轻量系统） | ⚠️ 部分 | 仅 BLE C API |
| Small（超轻系统） | ⚠️ 部分 | 仅 BLE C API |

## 关键概念

### System Ability ID 1130

蓝牙系统能力的唯一标识符，用于与蓝牙服务进程通信：

```
客户端框架 (本仓库)  ──IPC──>  SAMGR  ──路由──>  Bluetooth Service (SA 1130)
```

证据：`bluetooth_service_ipc_interface_code.h:22`
```cpp
/* SAID: 1130 */
```

### N-API 模块命名空间

| 模块 | JS 命名空间 | 注册文件 |
|------|-------------|----------|
| BLE | `bluetooth.ble` | `native_module_ble.cpp:70` |
| GAP | `bluetooth` / `bluetoothManager` | `native_module.cpp:97-99` |
| A2DP | `bluetooth.a2dp` | `native_module_a2dp.cpp:60` |
| HFP | `bluetooth.hfp` | `native_module_hfp.cpp:65` |
| ... | ... | ... |

### 线程模型

| 线程 | 职责 | 关键实现 |
|------|------|----------|
| 主线程 | N-API 调用入口、JS 回调 | Node.js N-API 事件循环 |
| FFRT 线程 | 异步任务执行 | `ffrt_inner.h` |
| IPC 线程 | 跨进程通信 | SAMGR 内部管理 |
| Profile 线程 | 蓝牙协议栈操作 | 各 Profile 实现 |

## 相关资源

| 资源 | 链接 |
|------|------|
| 项目源码 | `/Volumes/lexar/code/d/work/oh/foundation/communication/bluetooth` |
| Gitee 仓库 | https://gitee.com/openharmony/communication_bluetooth |
| OpenHarmony Docs | https://gitee.com/openharmony/docs |
| Bluetooth SIG | https://www.bluetooth.com/ |

---

**下一步**: [目录结构](01_Directory_Structure.md) → 了解代码组织方式
