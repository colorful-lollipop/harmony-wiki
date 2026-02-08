# 项目定位与核心能力

## 目的

本文档说明 Net Manager Ext 项目的定位、边界、核心能力、运行环境和关键概念，帮助新人快速理解项目整体。

---

## 项目定位

### 基本信息

| 属性 | 值 | 来源 |
|------|-----|------|
| **组件名称** | netmanager_ext | bundle.json:2 |
| **子系统** | communication | bundle.json:21 |
| **版本** | 4.0 | bundle.json:3 |
| **许可证** | Apache License 2.0 | bundle.json:8 |
| **代码仓库** | https://gitee.com/openharmony/communication_netmanager_ext | bundle.json:7 |

### 项目性质

Net Manager Ext 是 OpenHarmony **网络管理子系统的扩展模块**，提供可裁剪的网络管理能力：

**核心服务**（归档在 `netmanager_base`，不在本仓库）：
- 连接管理
- 策略管理
- 流量管理

**扩展服务**（本仓库）：
- 以太网连接（Ethernet）
- 网络共享（Network Sharing）
- 多播 DNS（mDNS）
- 虚拟专用网络（VPN）
- 网络防火墙（Net Firewall）
- 可穿戴分布式网络（Wearable Distributed Net）
- 网络切片（Network Slice）

**证据**：README_zh.md:16-17

---

## 系统能力（SystemCapabilities）

| 系统能力 ID | 说明 | 对应模块 | bundle.json |
|--------------|------|---------|------------|
| SystemCapability.Communication.NetManager.Ethernet | 以太网管理 | ethernet | bundle.json:23 |
| SystemCapability.Communication.NetManager.NetSharing | 网络共享 | sharing | bundle.json:24 |
| SystemCapability.Communication.NetManager.MDNS | 多播 DNS | mdns | bundle.json:25 |
| SystemCapability.Communication.NetManager.Vpn | VPN 管理 | vpn | bundle.json:26 |
| SystemCapability.Communication.NetManager.NetFirewall | 网络防火墙 | netfirewall | bundle.json:27 |
| SystemCapability.Communication.NetManager.Eap | EAP 认证（已禁用） | - | bundle.json:28 (值为 false) |

---

## 特性开关（Feature Flags）

所有特性在 `netmanager_ext_config.gni:57-77` 中定义，支持按需裁剪：

| 特性开关 | 默认值 | 模块 | 说明 |
|---------|--------|------|------|
| `netmanager_ext_feature_ethernet` | true | ethernet | 以太网管理 |
| `netmanager_ext_feature_share` | true | sharing | 网络共享 |
| `netmanager_ext_feature_mdns` | true | mdns | mDNS 服务发现 |
| `netmanager_ext_feature_vpn` | true | vpn | VPN 管理 |
| `netmanager_ext_feature_vpnext` | true | vpn_ext | VPN 扩展能力 |
| `netmanager_ext_feature_net_firewall` | false | netfirewall | 网络防火墙 |
| `netmanager_ext_feature_sysvpn` | false | vpn | 系统 VPN |
| `netmanager_ext_feature_wearable_distributed_net` | false | wearable_distributed | 可穿戴分布式网络 |
| `netmanager_ext_feature_networkslice` | false | networkslice | 网络切片 |
| `netmanager_ext_feature_vpn_for_user0` | false | vpn | User0 VPN |
| `netmanager_ext_extensible_authentication` | false | ethernet | 可扩展认证（EAP） |
| `netmanager_ext_share_traffic_limit_enable` | false | sharing | 共享流量限制 |
| `netmanager_ext_share_notification_enable` | false | sharing | 共享通知 |
| `netmanager_ext_feature_coverage` | false | - | 代码覆盖率 |

---

## 运行环境

### 进程模型

根据 `sa_profile/*.json` 配置，服务运行在以下进程：

| 进程名 | System Ability | 产物文件 | 来源 |
|---------|---------------|----------|------|
| netmanager | 8300 (NetFirewall) | libnetfirewall_manager.z.so | sa_profile/8300.json:5 |
| netmanager | 8301 (NetworkSlice) | libnetworkslice_manager.z.so | sa_profile/8301.json:5 |
| mdnsmanager | 1161 (mDNS) | libmdns_manager.z.so | sa_profile/1161.json:5 |
| netmanager | 8400 (Wearable Distributed Net) | libwearable_distributed_net_manager.z.so | sa_profile/8400.json:5 |

**注**：部分 SA（如 VPN、Sharing、Ethernet）可能运行在其他进程，需要进一步确认 `netmanager_base` 进程。

### 系统依赖

| 依赖组件 | 用途 | bundle.json |
|---------|------|------------|
| napi | N-API 绑定 | bundle.json:56 |
| ipc | IPC 通信 | bundle.json:54 |
| safwk/samgr | SA 框架 | bundle.json:55,65 |
| access_token | 权令牌验证 | bundle.json:71 |
| hilog | 日志输出 | bundle.json:58 |
| hisysevent | 系统事件 | bundle.json:62 |
| netmanager_base | 基础网络服务 | bundle.json:59 |
| eventhandler | 事件循环 | bundle.json:60 |
| wifi/bluetooth | 网络硬件 | bundle.json:68,61 |
| drivers_interface_usb | USB 管理 | bundle.json:67,71 |

完整依赖列表见 [GN Build 文档](06_GN_Build.md)。

---

## 关键概念

### N-API（Node-API）

OpenHarmony 的 JavaScript 与 Native C++ 桥接层，允许 JS 应用调用 C++ 服务。

- **注册方式**：`napi_module_register`（构造函数自动调用）
- **模块命名**：`net.ethernet`, `net.sharing` 等
- **示例**：ethernet_module.cpp:159-162

### System Ability（SA）

OpenHarmony 的跨进程服务框架，提供系统级服务能力。

- **SA ID**：16-bit 整数（如 8300, 1161, 8301, 8400）
- **IPC 机制**：HDI/HBinder
- **Proxy/Stub 模式**：客户端使用 Proxy，服务端实现 Stub

### 权限模型

所有敏感操作都需要权限验证（通过 `NetManagerPermission::CheckPermission`）：

| 权限 | 用途 | 代码位置 |
|-------|------|---------|
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | ethernet_service.cpp:241,256 |
| `ohos.permission.GET_ETHERNET_LOCAL_MAC` | 获取 MAC 地址 | ethernet_service.cpp:211 |
| `ohos.permission.CONNECTIVITY_INTERNAL` | 系统内部连接操作 | ethernet_service.cpp:226,284 |
| `ohos.permission.MANAGE_VPN` | 管理 VPN | networkvpn_service.cpp:674 |
| `ohos.permission.MANAGE_NET_FIREWALL` | 管理防火墙 | netfirewall_stub.cpp:33 |
| `ohos.permission.MANAGE_ENTERPRISE_WIFI_CONNECTION` | 企业 WiFi | ethernet_service.cpp:451 |

**证据**：services/*/src/*_service.cpp 中的权限检查调用

---

## 项目边界

### 包含内容

✅ N-API 模块（JS → C++ 绑定）
✅ System Ability 服务实现（C++）
✅ IPC Proxy/Stub（进程间通信）
✅ 网络配置与状态管理
✅ 安全权限验证

### 不包含内容

❌ 基础网络服务（连接、策略、流量）→ `netmanager_base` 仓库
❌ 网络协议栈实现 → `netstack` 仓库
❌ 硬件驱动 → `drivers_interface_*` 仓库
❌ 测试代码（本仓库 `test/` 目录）

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 代码组织详解
- [架构说明](03_Architecture.md) - 架构图与数据流
- [JS API 文档](04_JS_API.md) - 完整接口清单
- [安全风险评审](08_Security_Review.md) - 攻击面分析
