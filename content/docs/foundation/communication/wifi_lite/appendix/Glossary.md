# 术语表

## 缩略语

| 缩略语 | 全称 | 中文 | 首次出现 |
|--------|------|------|----------|
| Wi-Fi | Wireless Fidelity | 无线保真 | [概览](01_Overview.md) |
| SSID | Service Set Identifier | 服务集标识符 | [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h) |
| BSSID | Basic Service Set Identifier | 基本服务集标识符 | [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h) |
| RSSI | Received Signal Strength Indicator | 接收信号强度指示 | [wifi_scan_info.h](interfaces/wifiservice/wifi_scan_info.h) |
| WPA | Wi-Fi Protected Access | Wi-Fi 安全访问 | [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.h) |
| WPA2 | Wi-Fi Protected Access II | Wi-Fi 安全访问 II | [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.md) |
| MAC | Media Access Control | 媒体访问控制 | [wifi_device.h](interfaces/wifiservice/wifi_device.h) |
| IP | Internet Protocol | 网际协议 | [wifi_device.h](interfaces/wifiservice/wifi_device.h) |
| N-API | Native API | 原生 API | [README](README.md) |
| API | Application Programming Interface | 应用程序编程接口 | [API 参考](02_API_Reference.md) |
| GN | Generate Ninja | 生成 Ninja 构建文件工具 | [构建系统](03_Build_System.md) |
| IPC | Inter-Process Communication | 进程间通信 | - |
| SA | System Ability | 系统能力 | - |

## Wi-Fi 频段

| 频段 | 频率范围 | 特点 |
|------|----------|------|
| 2.4G | 2.4-2.5 GHz | 穿墙能力强，覆盖广 |
| 5G | 5.15-5.825 GHz | 速度快，干扰少，穿墙弱 |

**证据**: [wifi_device.h:240](interfaces/wifiservice/wifi_device.h#L240) 中的 `HOTSPOT_BAND_TYPE_5G` 和 `HOTSPOT_BAND_TYPE_2G` 宏

## 安全类型

| 类型值 | 安全协议 | 说明 |
|--------|----------|------|
| `WIFI_SECURITY_OPEN` | 开放 | 无密码 |
| `WIFI_SECURITY_WEP` | WEP | 已废弃，不安全 |
| `WIFI_SECURITY_PSK` | WPA/WPA2 | 个人级安全 |
| `WIFI_SECURITY_SAE` | WPA3 | 新一代安全 |

**证据**: [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h) 或 [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.h) 中的安全类型定义

## Wi-Fi 模式

| 模式 | 英文 | 功能 |
|------|------|------|
| Station | Station (STA) | 客户端模式，连接到热点 |
| Hotspot | Access Point (AP) | 热点模式，提供无线接入 |
| Monitor | Monitor | 监听模式，抓包分析 |

## 状态码

| 状态码 | 含义 |
|--------|------|
| `WIFI_STA_ACTIVE` | Station 已启用 |
| `WIFI_STA_NOT_ACTIVE` | Station 未启用 |
| `WIFI_HOTSPOT_ACTIVE` | Hotspot 已启用 |
| `WIFI_HOTSPOT_NOT_ACTIVE` | Hotspot 未启用 |

**证据**: [wifi_device.h:71-72](interfaces/wifiservice/wifi_device.h#L71-L72) 和 [wifi_hotspot.h:103-104](interfaces/wifiservice/wifi_hotspot.h#L103-L104)

## 错误码前缀

| 前缀 | 来源 | 说明 |
|------|------|------|
| `WIFI_SUCCESS` | 成功 | 操作成功 |
| `WIFI_ERROR_` | 错误 | 操作失败相关 |
| `WIFI_FAILED` | 失败 | 通用失败 |

**证据**: [wifi_error_code.h](interfaces/wifiservice/wifi_error_code.h)

---

## 术语索引

| 术语 | 解释 |
|------|------|
| 缓冲区溢出 | 向缓冲区写入超出其容量的数据 |
| 竞态条件 | 多线程访问共享资源时的执行顺序不确定 |
| 回调函数 | 由被调用方执行的函数指针 |
| 句柄 | 引用资源的抽象标识符 |

---

[返回 SUMMARY.md](SUMMARY.md)
