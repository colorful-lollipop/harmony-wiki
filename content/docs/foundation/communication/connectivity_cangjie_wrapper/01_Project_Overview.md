# 项目概览

> connectivity_cangjie_wrapper 项目定位、核心能力与运行环境

## 项目定位

### 定位声明

`connectivity_cangjie_wrapper` 是 OpenHarmony **communication 子系统**中的一个组件，专注于为 **Cangjie 语言**应用提供蓝牙和 WLAN 服务的 **API 封装层**。

```
┌─────────────────────────────────────────────────────────────────┐
│                    项目在 OpenHarmony 中的位置                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  应用层 (Cangjie 应用)                                           │
│         ↓                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │        connectivity_cangjie_wrapper (本项目)              │   │
│  │     提供 Cangjie API 封装供应用调用                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│         ↓                                                        │
│  ┌──────────────────┐    ┌──────────────────┐                  │
│  │ communication_   │    │ communication_   │                  │
│  │ bluetooth        │    │ wifi             │                  │
│  │ (蓝牙原生组件)   │    │ (WiFi 原生组件)  │                  │
│  └──────────────────┘    └──────────────────┘                  │
│         ↓                                                        │
│  硬件抽象层 (蓝牙芯片 / WiFi 芯片)                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 项目边界

| 包含内容 | 不包含内容 |
|----------|------------|
| Cangjie API 定义与封装 | 蓝牙/WiFi 底层实现 |
| 参数校验与错误处理 | 蓝牙芯片驱动 |
| 回调机制封装 | WiFi 固件交互 |
| Kit 层统一出口 | 硬件特定优化 |

## 核心能力

### 蓝牙服务

| 能力 | 模块 | 功能描述 |
|------|------|----------|
| **BLE 扫描** | `ohos.bluetooth.ble` | 发现周围低功耗蓝牙设备 |
| **BLE 广播** | `ohos.bluetooth.ble` | 作为外设发送广播 |
| **GATT 服务** | `ohos.bluetooth.ble` | 作为服务端或客户端提供 GATT 服务 |
| **A2DP** | `ohos.bluetooth.a2dp` | 高质量音频流分发（耳机、音箱） |
| **HFP** | `ohos.bluetooth.hfp` | 免提通话控制（车载、耳机） |
| **连接管理** | `ohos.bluetooth.connection` | 设备配对与连接状态管理 |
| **常量定义** | `ohos.bluetooth.constant` | 连接状态、Profile 类型等枚举 |

### WLAN 服务

| 能力 | 模块 | 功能描述 |
|------|------|----------|
| **P2P 发现** | `ohos.wifi_manager` | 发现周围 P2P 设备 |
| **P2P 连接** | `ohos.wifi_manager` | 建立设备间直连 |
| **P2P 回调** | `ohos.wifi_manager` | 监听扫描状态变化 |

## 运行环境

### 设备类型

| 设备类型 | 支持状态 | 说明 |
|----------|----------|------|
| **标准设备** | ✅ 支持 | Phone、Tablet、TV 等 |
| 轻量设备 | ❌ 不支持 | Wearable、Lite 等 |

### 系统要求

| 要求 | 详情 |
|------|------|
| **OpenHarmony 版本** | API 22 及以上 |
| **系统能力** | `SystemCapability.Communication.Bluetooth.Core` |
| | `SystemCapability.Communication.WiFi.STA` |
| | `SystemCapability.Communication.WiFi.P2P` |

### 依赖组件

| 组件 | 用途 | 来源 |
|------|------|------|
| `cangjie_ark_interop` | C 语言互操作接口、异常类 | arkcompiler_cangjie_ark_interop |
| `hiviewdfx_cangjie_wrapper` | 日志接口 | hiviewdfx_cangjie_wrapper |
| `bluetooth` | 蓝牙 FFI 底层接口 | communication_bluetooth |
| `wifi` | WiFi FFI 底层接口 | communication_wifi |

## 版本信息

| 信息 | 值 |
|------|-----|
| **bundle.json 版本** | 6.1 |
| **ROM 占用** | 约 1100KB |
| **RAM 占用** | 约 1176KB |
| **许可证** | Apache License 2.0 |
| **当前状态** | Beta |

## 关键概念

### Cangjie API 注解

项目中的 Cangjie API 使用 `@!APILevel` 注解标注，示例：

```cj
@!APILevel[
    since: "22",
    permission: "ohos.permission.ACCESS_BLUETOOTH",
    syscap: "SystemCapability.Communication.Bluetooth.Core",
    throwexception: true,
    workerthread: true
]
public func startBleScanning(...): Unit { ... }
```

### 注解属性说明

| 属性 | 必填 | 说明 |
|------|------|------|
| `since` | 是 | API 引入的 API 版本号 |
| `syscap` | 是 | 所需的系统能力标识 |
| `permission` | 否 | 调用所需权限 |
| `throwexception` | 否 | 是否抛出 BusinessException |
| `workerthread` | 否 | 是否在工作线程执行 |

### 错误码体系

| 错误码 | 含义 | 处理建议 |
|--------|------|----------|
| 201 | 权限拒绝 | 检查并申请所需权限 |
| 801 | 能力不支持 | 检查设备是否支持 |
| 2900001 | 服务停止 | 等待服务重启 |
| 2900003 | 蓝牙禁用 | 引导用户开启蓝牙 |
| 2900099 | 操作失败 | 检查参数和网络状态 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [目录结构](./02_Directory_Structure.md) | 模块划分与文件布局 |
| [架构说明](./03_Architecture.md) | 组件图与数据流 |
| [N-API 参考](./04_N-API_Reference.md) | API 详细说明 |
| [安全评审](./07_Security_Review.md) | 安全使用指南 |
