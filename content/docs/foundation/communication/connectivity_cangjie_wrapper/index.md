# connectivity_cangjie_wrapper

> OpenHarmony 蓝牙与 WLAN Cangjie API 封装层

## 项目简介

`connectivity_cangjie_wrapper` 是 OpenHarmony 平台上 **Cangjie 语言** 的蓝牙和 WLAN API 封装层，为应用开发者提供便捷的蓝牙服务与 WLAN 服务能力接口。

### 核心能力

| 能力类别 | 功能 | 说明 |
|----------|------|------|
| **蓝牙 BLE** | 设备扫描、广播、GATT 服务 | 低功耗蓝牙完整功能 |
| **蓝牙 A2DP** | 音频流传输 | 高质量多媒体音频分发 |
| **蓝牙 HFP** | 免提通话控制 | 蓝牙设备控制通话功能 |
| **蓝牙连接** | 设备配对与连接 | 基础连接管理 |
| **WLAN P2P** | 点对点直连 | 无需 AP 的设备互联 |

### 技术特点

- **Beta 功能**: 当前处于 Beta 阶段
- **标准设备**: 仅支持 OpenHarmony 标准设备
- **API 版本**: 从 API 22 开始支持
- **系统能力**: 依赖 `SystemCapability.Communication.Bluetooth.Core` 和 `SystemCapability.Communication.WiFi.*`

## 快速开始

### 权限申请

```cj
// 在 module.json5 中申请权限
"requestPermissions": [
  {
    "name": "ohos.permission.ACCESS_BLUETOOTH",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inUse"
    }
  },
  {
    "name": "ohos.permission.GET_WIFI_INFO",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inUse"
    }
  }
]
```

### 导入模块

```cj
import ohos.bluetooth.ble.*
import ohos.wifi_manager.*
```

### 基本使用示例

**BLE 扫描**：

```cj
// 开始扫描
startBleScanning([], None)

// 监听扫描结果
on(BleDeviceFind) { results ->
    for (result in results) {
        print("Found device: ${result.deviceId}")
    }
}
```

**WLAN P2P 连接**：

```cj
// 发现设备
startDiscoverDevices()

// 连接设备
let config = WifiP2pConfig()
p2pConnect(config)
```

## 文档导航

| 主题 | 链接 |
|------|------|
| 项目详细说明 | [项目概览](./01_Project_Overview.md) |
| 模块结构 | [目录结构](./02_Directory_Structure.md) |
| 架构设计 | [架构说明](./03_Architecture.md) |
| API 参考 | [N-API 参考](./04_N-API_Reference.md) |
| 构建系统 | [构建系统](./06_Build_System.md) |
| 安全使用 | [安全评审](./07_Security_Review.md) |
| 常见问题 | [FAQ 与排错](./08_FAQ_Troubleshooting.md) |

## 相关资源

- **API 文档**: [Cangjie ConnectivityKit API](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_en/apis/ConnectivityKit)
- **开发指南**: [蓝牙服务开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/connectivity/bluetooth/cj-bluetooth-overview.md)
- **源码仓库**: [gitee.com/openharmony/connectivity_cangjie_wrapper](https://gitee.com/openharmony/connectivity_cangjie_wrapper)

## 限制与约束

当前版本存在以下限制：

- 蓝牙服务暂不提供 Bluetooth socket、HID、PAN、PBAP、MAP Profile 相关功能
- WLAN 服务暂不提供 STA 模式和 AP 模式相关功能
- 仅支持标准设备，不支持轻量设备

---

*了解更多信息，请阅读 [项目概览](./01_Project_Overview.md)*
