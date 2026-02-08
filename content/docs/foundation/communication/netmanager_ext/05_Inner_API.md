# 内部 API 模块接口

## 目的

本文档说明 Net Manager Ext 内部模块间的 API 接口、依赖关系、稳定性说明，帮助开发者进行模块间协作开发。

---

## Inner Kit 概览

每个模块的 Inner Kit 包括：
- **客户端 Proxy**：调用远程 SA
- **服务端 Stub**：处理来自 Proxy 的请求
- **接口定义**：IPC 接口（I*Service）
- **数据结构**：Parcel 序列化对象

### Inner Kit 列表（来自 bundle.json）

| Inner Kit | 产物名 | 对应服务 | bundle.json 行号 |
|-----------|--------|---------|---------------|
| net_tether_manager_if | libnet_tether_manager_if.z.so | Network Sharing | bundle.json:130-140 |
| ethernet_manager_if | libethernet_manager_if.z.so | Ethernet | bundle.json:142-151 |
| mdns_manager_if | libmdns_manager_if.z.so | MDNS | bundle.json:153-162 |
| vpn_extension_module | libvpn_extension_module.z.so | VPN Extension | bundle.json:164-171 |
| net_vpn_manager_if | libnet_vpn_manager_if.z.so | VPN | bundle.json:173-182 |
| netfirewall_manager_if | libnetfirewall_manager_if.z.so | Net Firewall | bundle.json:184-193 |
| wearable_distributed_net_manager_if | libwearable_distributed_net_manager_if.z.so | Wearable Distributed Net | bundle.json:195-204 |
| networkslice_manager_if | libnetworkslice_manager_if.z.so | Network Slice | bundle.json:206-212 |

---

## Ethernet Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/ethernetclient/include/ethernet_client.h` - 客户端接口
- `interfaces/innerkits/ethernetclient/include/interface_configuration.h` - 配置数据结构
- `interfaces/innerkits/ethernetclient/include/ethernet_device_info.h` - 设备信息

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/ethernetclient/src/proxy/ethernet_proxy.cpp`

**Stub 位置**：
- `services/ethernetmanager/src/ethernet_stub.cpp`

### 关键方法

| 方法名 | 说明 | 权限要求 |
|-------|------|----------|
| `SetIfaceConfig` | 设置接口配置 | CONNECTIVITY_INTERNAL |
| `GetIfaceConfig` | 获取接口配置 | GET_NETWORK_INFO |
| `IsIfaceActive` | 检查接口激活状态 | GET_NETWORK_INFO |
| `GetAllActiveIfaces` | 获取所有激活接口 | GET_NETWORK_INFO |
| `GetMacAddress` | 获取 MAC 地址 | GET_ETHERNET_LOCAL_MAC |
| `RegisterIfacesStateChanged` | 注册状态变化回调 | GET_NETWORK_INFO |

**代码证据**：
- 服务实现：services/ethernetmanager/src/ethernet_service.cpp:211-527

### 回调接口

**头文件**：
- `interfaces/innerkits/ethernetclient/include/proxy/interface_state_callback.h`

**Stub 实现**：
- `frameworks/native/ethernetclient/src/proxy/interface_state_callback_stub.cpp`

---

## Network Sharing Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/netshareclient/include/networkshare_client.h` - 客户端接口
- `interfaces/innerkits/netshareclient/include/networkshare_constants.h` - 常量定义

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/netshareclient/src/proxy/sharing_proxy.cpp`（待确认具体路径）

**Stub 位置**：
- `services/networksharemanager/src/networkshare_stub.cpp`（可能存在）

### 关键方法

| 方法名 | 说明 | 权限要求 |
|-------|------|----------|
| `StartSharing` | 开启共享 | CONNECTIVITY_INTERNAL |
| `StopSharing` | 停止共享 | CONNECTIVITY_INTERNAL |
| `IsSharing` | 获取共享状态 | 无 |
| `IsSharingSupported` | 检查共享支持 | 无 |
| `GetStats*Bytes` | 获取流量统计 | 无 |

**代码证据**：
- 服务实现：services/networksharemanager/src/networkshare_service.cpp:172-560
- 权限检查：networkshare_service.cpp:172, 184, 201, 230, 282, ...

### 回调接口

**头文件**：
- `interfaces/innerkits/netshareclient/include/proxy/ipccallback/sharing_event_callback.h`

**Stub/Proxy 实现**：
- `frameworks/native/netshareclient/src/proxy/ipccallback/sharing_event_callback_stub.cpp`
- `frameworks/native/netshareclient/src/proxy/ipccallback/sharing_event_callback_proxy.cpp`

**事件类型**：
- `EVENT_SHARE_STATE_CHANGE` - 共享状态改变
- `EVENT_IFACE_SHARE_STATE_CHANGE` - 接口共享状态改变
- `EVENT_SHARE_UPSTREAM_CHANGE` - 上行网卡改变

**代码证据**：
- Observer 包装：frameworks/js/napi/sharing/src/netshare_observer_wrapper.cpp

---

## MDNS Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/mdnsclient/include/mdns_client.h`

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/mdnsclient/src/proxy/mdns_proxy.cpp`（待确认）

**Stub 位置**：
- `services/mdnsmanager/src/mdns_stub.cpp`（待确认）

### System Ability

**SA ID**：1161
**进程**：mdnsmanager
**配置文件**：sa_profile/1161.json

**代码证据**：
- SA 配置：sa_profile/1161.json:4-6

---

## VPN Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/netvpnclient/include/networkvpn_client.h`

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/netvpnclient/src/proxy/vpn_proxy.cpp`（待确认）

**Stub 位置**：
- `services/vpnmanager/src/networkvpn_stub.cpp`（待确认）

### 关键方法

| 方法名 | 说明 | 权限要求 |
|-------|------|----------|
| `SetUp` | 建立连接 | MANAGE_VPN |
| `Destroy` | 断开连接 | MANAGE_VPN |
| `Prepare` | 准备连接 | GET_NETWORK_INFO |

**代码证据**：
- 权限检查：services/vpnmanager/src/networkvpn_service.cpp:674

---

## Net Firewall Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/netfirewallclient/include/netfirewall_client.h`
- `interfaces/innerkits/netfirewallclient/include/netfirewall_common.h` - 公共定义
- `interfaces/innerkits/netfirewallclient/include/i_netfirewall_service.h` - 服务接口

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/netfirewallclient/src/netfirewall_proxy.cpp`

**Stub 位置**：
- `services/netfirewallmanager/src/netfirewall_stub.cpp`

### 关键方法

| 方法名 | 说明 | 权限要求 |
|-------|------|----------|
| `AddFirewallRule` | 添加防火墙规则 | MANAGE_NET_FIREWALL |
| `DeleteFirewallRule` | 删除防火墙规则 | MANAGE_NET_FIREWALL |
| `GetFirewallRules` | 获取规则列表 | GET_NET_FIREWALL |

**代码证据**：
- 权限常量：services/netfirewallmanager/src/netfirewall_stub.cpp:33
- `PERMISSION_MANAGE_NET_FIREWALL = "ohos.permission.MANAGE_NET_FIREWALL"`
- `PERMISSION_GET_NET_FIREWALL = "ohos.permission.GET_NET_FIREWALL"`

### System Ability

**SA ID**：8300
**进程**：netmanager
**配置文件**：sa_profile/8300.json

**代码证据**：
- SA 配置：sa_profile/8300.json:4-6

---

## Network Slice Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/networksliceclient/include/networkslice_client.h`

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/networksliceclient/src/proxy/networkslice_proxy.cpp`

**Stub 位置**：
- `services/networkslicemanager/src/networkslice_stub.cpp`

### System Ability

**SA ID**：8301
**进程**：netmanager
**配置文件**：sa_profile/8301.json
**run-on-create**：true

**代码证据**：
- SA 配置：sa_profile/8301.json:4-6

---

## Wearable Distributed Net Inner API

### 接口头文件

**主要头文件**：
- `interfaces/innerkits/wearabledistributednetclient/include/wearable_distributed_net_client.h`

### IPC 接口定义

**Proxy 位置**：
- `frameworks/native/wearabledistributednetclient/src/proxy/wearable_distributed_net_proxy.cpp`（待确认）

**Stub 位置**：
- `services/wearabledistributednetmanager/src/wearable_distributed_net_stub.cpp`（待确认）

### 关键方法

| 方法名 | 说明 | 权限要求 |
|-------|------|----------|
| `SetupNet` | 建立网络 | CONNECTIVITY_INTERNAL |
| `TearDownNet` | 断开网络 | CONNECTIVITY_INTERNAL |

**代码证据**：
- 权限检查：services/wearabledistributednetmanager/src/wearable_distributed_net_service.cpp:56, 67, 77, 87

---

## 模块依赖方向

### 依赖图谱

```
┌─────────────────────────────────────────────────────────┐
│           netmanager_base (外部依赖)               │
│  连接管理、策略管理、流量管理                      │
└──────────────────┬──────────────────────────────┘
                 │ 依赖
                 ↓
┌─────────────────────────────────────────────────────────┐
│        netmanager_ext Inner Kits                 │
│  ┌──────┬──────┬──────┬──────┬──────┐   │
│  │Ethernet│Sharing│ MDNS │ VPN  │Firewall│   │
│  └──────┴──────┴──────┴──────┴──────┘   │
└──────────────────┬──────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────┐
│     netmanager_ext Services (SA)                 │
│  ┌──────┬──────┬──────┬──────┬──────┐   │
│  │Ethernet│Sharing│ MDNS │ VPN  │Firewall│   │
│  └──────┴──────┴──────┴──────┴──────┘   │
└──────────────────┬──────────────────────────────┘
                 │
                 ↓ 系统调用
┌─────────────────────────────────────────────────────────┐
│     netstack / drivers / kernel                   │
└─────────────────────────────────────────────────────────┘
```

**依赖说明**：
- ✅ Inner Kits → SA 服务（单向依赖）
- ✅ SA 服务 → netmanager_base（外部依赖）
- ✅ SA 服务 → netstack（外部依赖）
- ❌ 无循环依赖

**代码证据**：bundle.json:51-97 - deps.components

---

## 接口稳定性说明

### 稳定接口（公共 API）

**定义依据**：
1. 在 `interfaces/innerkits/*/include/` 中定义
2. 在 `bundle.json` 的 `inner_kits` 中声明
3. 有明确的版本控制

**示例**：
- `IEthernetService` @ ethernet_client.h
- `INetworkShareService` @ networkshare_client.h
- `INetFirewallService` @ i_netfirewall_service.h

**变更控制**：需要跨模块协调，避免破坏现有客户端

### 内部实现接口（不稳定）

**定义依据**：
1. 仅在 `services/*/src/` 中使用
2. 服务内部类名（如 `EthernetService`, `NetworkShareService`）
3. 未在 `interfaces/innerkits/` 中暴露

**示例**：
- `EthernetService` 类的具体方法
- `NetworkShareStateMachine` 内部状态机

**变更自由度**：可随意修改，不影响外部模块

---

## 数据流向

### 请求流向（JS → 内核）

```
JavaScript App
  ↓ (N-API 调用)
N-API Module (frameworks/js/napi/)
  ↓ (异步工作)
IPC Proxy (frameworks/native/)
  ↓ (HDI/HBinder)
SA Stub (services/*/)
  ↓ (权限检查 + 业务逻辑)
netmanager_base / netstack
  ↓ (系统调用)
Kernel / Drivers
```

### 事件流向（内核 → JS）

```
Kernel / Drivers
  ↑ (网络事件)
netmanager_base / netstack
  ↑ (回调)
SA Service (services/*/)
  ↑ (IPC 回调)
IPC Callback Stub (frameworks/native/)
  ↑ (事件通知)
N-API Observer (frameworks/js/napi/)
  ↑ (JavaScript 回调)
JavaScript App
```

**代码证据**：
- 事件监听：frameworks/js/napi/sharing/src/netshare_observer_wrapper.cpp
- IPC 回调：frameworks/native/netshareclient/src/proxy/ipccallback/sharing_event_callback_stub.cpp

---

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 权限模型
- [架构说明](03_Architecture.md) - 调用链与数据流
- [JS API 文档](04_JS_API.md) - 对外接口
- [GN Build 文档](06_GN_Build.md) - 构建配置
