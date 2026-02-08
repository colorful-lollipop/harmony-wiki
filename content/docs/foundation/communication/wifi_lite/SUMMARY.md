# 文档导航

## 新人阅读路线

建议阅读顺序（按优先级）：

1. **[概览](01_Overview.md)** → 了解项目定位、核心能力
2. **[使用指南](05_Usage_Guide.md)** → 快速上手，掌握完整使用流程
3. **[API 参考](02_API_Reference.md)** → 掌握 Station/Hotspot 接口
4. **[构建系统](03_Build_System.md)** → 理解编译配置
5. **[安全评审](04_Security_Review.md)** → 了解安全注意事项

---

## 全站目录

### 核心文档

| 章节 | 标题 | 说明 |
|------|------|------|
| [README](README.md) | 文档说明 | 覆盖范围、更新方式 |
| [01_Overview](01_Overview.md) | 项目概览 | 定位、边界、模块职责 |
| [02_API_Reference](02_API_Reference.md) | API 参考 | Station/Hotspot 接口完整清单 |
| [03_Build_System](03_Build_System.md) | 构建系统 | GN 目标、编译产物 |
| [04_Security_Review](04_Security_Review.md) | 安全评审 | 攻击面、风险点、建议 |
| [05_Usage_Guide](05_Usage_Guide.md) | 使用指南 | 完整代码示例、最佳实践 |

### 附录

| 章节 | 标题 | 说明 |
|------|------|------|
| [附录：术语表](appendix/Glossary.md) | 术语定义 | 缩略语、全称 |
| [附录：错误码](appendix/ErrorCodes.md) | 错误码参考 | 完整错误码列表 |

---

## 快速索引

### Station 模式 API

| 函数 | 功能 | 行号 |
|------|------|------|
| `EnableWifi()` | 启用 Station 模式 | [wifi_device.h:57](interfaces/wifiservice/wifi_device.h#L57) |
| `DisableWifi()` | 禁用 Station 模式 | [wifi_device.h:66](interfaces/wifiservice/wifi_device.h#L66) |
| `Scan()` | 开始扫描 | [wifi_device.h:84](interfaces/wifiservice/wifi_device.h#L84) |
| `GetScanInfoList()` | 获取扫描结果 | [wifi_device.h:98](interfaces/wifiservice/wifi_device.h#L98) |
| `AddDeviceConfig()` | 添加网络配置 | [wifi_device.h:111](interfaces/wifiservice/wifi_device.h#L111) |
| `ConnectTo()` | 连接网络 | [wifi_device.h:169](interfaces/wifiservice/wifi_device.h#L169) |
| `GetLinkedInfo()` | 获取连接信息 | [wifi_device.h:198](interfaces/wifiservice/wifi_device.h#L198) |

### Hotspot 模式 API

| 函数 | 功能 | 行号 |
|------|------|------|
| `EnableHotspot()` | 启用热点 | [wifi_hotspot.h:63](interfaces/wifiservice/wifi_hotspot.h#L63) |
| `DisableHotspot()` | 禁用热点 | [wifi_hotspot.h:72](interfaces/wifiservice/wifi_hotspot.h#L72) |
| `SetHotspotConfig()` | 设置热点配置 | [wifi_hotspot.h:86](interfaces/wifiservice/wifi_hotspot.h#L86) |
| `GetStationList()` | 获取已连接站点 | [wifi_hotspot.h:121](interfaces/wifiservice/wifi_hotspot.h#L121) |

### 数据结构

| 结构体 | 用途 | 头文件 |
|--------|------|--------|
| `WifiDeviceConfig` | Wi-Fi 连接配置 | [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h) |
| `HotspotConfig` | 热点配置 | [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.h) |
| `WifiEvent` | 事件回调 | [wifi_event.h](interfaces/wifiservice/wifi_event.h) |
| `WifiScanInfo` | 扫描信息 | [wifi_scan_info.h](interfaces/wifiservice/wifi_scan_info.h) |
| `WifiLinkedInfo` | 连接信息 | [wifi_linked_info.h](interfaces/wifiservice/wifi_linked_info.md) |

---

## 相关链接

- **OpenHarmony 官方文档**: https://gitee.com/openharmony/docs
- **Wi-Fi Lite 子系统**: https://gitee.com/openharmony/docs/blob/master/docs-cn/readme/distributed-communication-subsystem.md
- **组件仓库**: `//foundation/communication/wifi_lite`
