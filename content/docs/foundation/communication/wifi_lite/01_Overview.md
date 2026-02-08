# 项目概览

## 项目定位

`wifi_lite` 是 OpenHarmony 分布式通信子系统的轻量级 Wi-Fi 接口库，为第三方开发者提供 Wi-Fi station（客户端）和 hotspot（热点）模式的 C 语言编程接口。

### 核心能力

| 能力 | 描述 | 相关文件 |
|------|------|----------|
| Station 模式 | Wi-Fi 客户端连接管理 | [wifi_device.h](interfaces/wifiservice/wifi_device.h) |
| Hotspot 模式 | Wi-Fi 热点创建与管理 | [wifi_hotspot.h](interfaces/wifiservice/wifi_hotspot.h) |
| 事件回调 | Wi-Fi 状态变化通知 | [wifi_event.h](interfaces/wifiservice/wifi_event.h) |
| 扫描发现 | 周边 Wi-Fi 热点探测 | [wifi_scan_info.h](interfaces/wifiservice/wifi_scan_info.h) |

### 适用范围

- **适配系统**: OpenHarmony Mini（轻量级设备）
- **接口类型**: C 语言原生接口（非 N-API）
- **依赖层级**: 上层应用 → wifi_lite 接口 → Wi-Fi 服务实现

## 模块职责

### interfaces/wifiservice/

本目录包含所有对外接口定义，无实现代码：

| 文件 | 职责 | 依赖 |
|------|------|------|
| [wifi_device.h](interfaces/wifiservice/wifi_device.h) | Station 模式 API | wifi_event.h, wifi_scan_info.h, wifi_error_code.h |
| [wifi_hotspot.h](interfaces/wifiservice/wifi_hotspot.h) | Hotspot 模式 API | wifi_device.h, wifi_hotspot_config.h |
| [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h) | Station 连接配置结构 | - |
| [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.h) | Hotspot 配置结构 | - |
| [wifi_event.h](interfaces/wifiservice/wifi_event.h) | 事件回调函数指针 | - |
| [wifi_scan_info.h](interfaces/wifiservice/wifi_scan_info.h) | 扫描结果数据结构 | station_info.h |
| [wifi_linked_info.h](interfaces/wifiservice/wifi_linked_info.h) | 连接状态信息 | - |
| [station_info.h](interfaces/wifiservice/station_info.h) | 站点信息结构 | - |
| [wifi_error_code.h](interfaces/wifiservice/wifi_error_code.h) | 错误码枚举定义 | - |

### 架构位置

```
┌─────────────────────────────────────────────────────────┐
│                    应用层 (ArkTS/JS)                      │
├─────────────────────────────────────────────────────────┤
│                 N-API 绑定层 (如有)                        │
├─────────────────────────────────────────────────────────┤
│              wifi_lite C 接口层 (本仓库)                    │
├─────────────────────────────────────────────────────────┤
│              Wi-Fi 服务实现层 (kernel/service)             │
├─────────────────────────────────────────────────────────┤
│                   Wi-Fi 硬件抽象层                         │
└─────────────────────────────────────────────────────────┘
```

**注意**: 本仓库仅包含第三层「C 接口定义」，实际功能实现在其他模块中。

## 运行环境

| 环境 | 要求 |
|------|------|
| 操作系统 | OpenHarmony Mini 系统 |
| 硬件 | 支持 Wi-Fi 的开发板/设备 |
| 编译系统 | GN + Ninja |
| 构建配置 | `//foundation/communication/wifi_lite:wifi` |

## 关键概念

### Station 模式

设备作为 Wi-Fi 客户端连接到无线网络：

- **启用**: `EnableWifi()` → 开启 Station 功能
- **扫描**: `Scan()` / `AdvanceScan()` → 发现周边热点
- **连接**: `AddDeviceConfig()` + `ConnectTo()` → 配置并连接
- **断开**: `Disconnect()` → 断开当前连接
- **状态**: `GetLinkedInfo()` → 查询连接状态

### Hotspot 模式

设备作为 Wi-Fi 热点供其他设备连接：

- **配置**: `SetHotspotConfig()` → 设置 SSID、安全类型、密钥
- **启用**: `EnableHotspot()` → 开启热点
- **管理**: `GetStationList()` → 查看已连接设备
- **断开**: `DisassociateSta()` → 强制断开指定设备

### 事件回调

通过 `RegisterWifiEvent()` 注册回调函数，监听以下事件：

- `WifiEventStateChangedCallback`: Wi-Fi 开关状态变化
- `WifiConnectionChangedCallback`: 连接状态变化
- `WifiScanStateChangedCallback`: 扫描完成通知

## 目录结构

```
wifi_lite/
├── interfaces/                    # 对外接口目录
│   └── wifiservice/             # Wi-Fi 服务接口
│       ├── wifi_device.h        # Station API (274 行)
│       ├── wifi_hotspot.h       # Hotspot API (153 行)
│       ├── wifi_device_config.h # 连接配置
│       ├── wifi_hotspot_config.h # 热点配置
│       ├── wifi_event.h         # 事件回调
│       ├── wifi_scan_info.h     # 扫描信息
│       ├── wifi_linked_info.h   # 连接信息
│       ├── station_info.h       # 站点信息
│       └── wifi_error_code.h    # 错误码
├── BUILD.gn                      # GN 构建配置
├── bundle.json                   # 组件描述
├── README.md                     # 项目说明
└── LICENSE                       # Apache 2.0 许可证
```

## 版本信息

| 属性 | 值 |
|------|------|
| 项目版本 | 3.1.0 |
| 子系统 | communication |
| 组件名 | wifi_lite |
| 适配系统 | mini |

---

[返回 SUMMARY.md](SUMMARY.md) | [API 参考](02_API_Reference.md)
