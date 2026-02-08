# GN 构建系统

## 目的

本文档说明 Net Manager Ext 的 GN（Generate Ninja）构建系统配置、关键 Targets、依赖关系和编译选项。

---

## GN 配置文件层级

```
netmanager_ext/
├── BUILD.gn                      # 根构建入口
├── bundle.json                    # 组件元数据
└── netmanager_ext_config.gni       # 构建配置变量
    ├── *.gni                     # 子模块配置
    └── modules/*/BUILD.gn         # 各模块构建文件
```

---

## 根 BUILD.gn

### 文件位置

`/Volumes/lexar/code/d/work/oh/foundation/communication/netmanager_ext/BUILD.gn`

### 关键构建组（Groups）

| 组名 | Target 类型 | 行号 | 包含 Targets | 条件 |
|------|-----------|-------|-------------|-------|
| `common_ext_packages` | group | 16 | net_event_report | `netmanager_ext_feature_ethernet` |
| `ethernet_packages` | group | 25 | ethernet (napi), ethernet_manager_if, ethernet_interfaces, net_event_report | `netmanager_ext_feature_ethernet` |
| `share_packages` | group | 38 | sharing (napi), net_tether_manager_if, network_share_config | `netmanager_ext_feature_share` |
| `mdns_packages` | group | 50 | mdns (napi), mdns_manager_if, mdns_manager | `netmanager_ext_feature_mdns` |
| `netfirewall_packages` | group | 61 | netfirewall (napi), netfirewall_manager_if, netfirewall_default_rule, netfirewall_manager | `netmanager_ext_feature_net_firewall` |
| `vpn_packages` | group | 72 | vpn (napi), net_vpn_manager_if, net_vpn_manager | `netmanager_ext_feature_vpn` |
| `vpn_ext_packages` | group | 82 | vpnextensionability, vpnextensionability_napi, vpnextensioncontext_napi, vpnextension, vpn_extension_module | `netmanager_ext_feature_vpnext` |
| `wearable_distributed_net_packages` | group | 95 | wearable_distributed_net_manager_if, wearable_distributed_net_link_info, wearable_distributed_net_manager | `netmanager_ext_feature_wearable_distributed_net` |
| `networkslice_packages` | group | 105 | networkslice_manager_if, networkslice_manager | `netmanager_ext_feature_networkslice` |

**代码证据**：BUILD.gn:16-112

### Service 构建目标

| Target 名称 | 位置 | 类型 | 用途 |
|-----------|-------|------|------|
| `mdnsmanager` | services/mdnsmanager/BUILD.gn | ohos_shared_library | MDNS SA |
| `netfirewall_manager` | services/netfirewallmanager/BUILD.gn | ohos_shared_library | Firewall SA |
| `networkslice_manager` | services/networkslicemanager/BUILD.gn | ohos_shared_library | Network Slice SA |
| `wearable_distributed_net_manager` | services/wearabledistributednetmanager/BUILD.gn | ohos_shared_library | Wearable Distributed Net SA |

---

## netmanager_ext_config.gni

### 文件位置

`/Volumes/lexar/code/d/work/oh/foundation/communication/netmanager_ext/netmanager_ext_config.gni`

### 路径变量

| 变量名 | 值 | 用途 |
|---------|-----|------|
| `NETMANAGER_EXT_ROOT` | `$SUBSYSTEM_DIR/netmanager_ext` | 项目根目录 |
| `ETHERNETMANAGER_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/ethernetmanager` | 以太网服务源码 |
| `NETWORKSHAREMANAGER_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/networksharemanager` | 共享服务源码 |
| `MDNSMANAGER_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/mdnsmanager` | MDNS 服务源码 |
| `VPNMANAGER_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/vpnmanager` | VPN 服务源码 |
| `NETFIREWALLMANAGER_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/netfirewallmanager` | 防火墙服务源码 |
| `WEARABLE_DISTRIBUTED_NET_MANAGER_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/wearabledistributednetmanager` | 可穿戴分布式网络源码 |
| `NETWORKSLICE_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/services/networkslicemanager` | 网络切片源码 |
| `ETHERNETMANAGER_INNERKITS_SOURCE_DIR` | `$NETMANAGER_EXT_ROOT/frameworks/native/ethernetclient` | 以太网客户端源码 |

**代码证据**：netmanager_ext_config.gni:14-51

### 特性开关（Feature Flags）

| 特性开关 | 默认值 | 说明 | 行号 |
|---------|--------|------|------|
| `netmanager_ext_feature_ethernet` | true | 以太网模块 | 62 |
| `netmanager_ext_feature_share` | true | 网络共享模块 | 63 |
| `netmanager_ext_feature_mdns` | true | MDNS 模块 | 64 |
| `netmanager_ext_feature_net_firewall` | false | 防火墙模块 | 65 |
| `netmanager_ext_feature_sysvpn` | false | 系统 VPN | 66 |
| `netmanager_ext_feature_vpn` | true | VPN 模块 | 67 |
| `netmanager_ext_feature_vpn_for_user0` | false | User0 VPN | 68 |
| `netmanager_ext_feature_vpnext` | true | VPN 扩展 | 69 |
| `netmanager_ext_feature_wearable_distributed_net` | false | 可穿戴分布式网络 | 71 |
| `netmanager_ext_feature_networkslice` | false | 网络切片 | 76 |
| `netmanager_ext_extensible_authentication` | false | 可扩展认证（EAP） | 75 |
| `netmanager_ext_share_traffic_limit_enable` | false | 共享流量限制 | 74 |
| `netmanager_ext_share_notification_enable` | false | 共享通知 | 73 |
| `netmanager_ext_feature_coverage` | false | 代码覆盖率 | 61 |

**代码证据**：netmanager_ext_config.gni:57-77

### 编译标志

| 标志类型 | 内容 | 用途 |
|---------|------|------|
| `common_cflags` | `-D_FORTIFY_SOURCE=2`, `-fdata-sections`, `-ffunction-sections`, `-Os`, `-O2` | 通用编译优化 |
| `memory_optimization_cflags` | `-fvisibility=hidden` | C 代码可见性优化 |
| `memory_optimization_cflags_cc` | `-fvisibility=hidden`, `-fvisibility-inlines-hidden` | C++ 代码可见性优化 |
| `memory_optimization_ldflags` | `-Wl,--exclude-libs=ALL`, `-Wl,--gc-sections` | 链接优化 |

**代码证据**：netmanager_ext_config.gni:81-101

---

## 关键 N-API Targets

### Ethernet N-API

**Target 定义**：`frameworks/js/napi/ethernet/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `ethernet` | ohos_shared_library | libnet_ethernet.z.so | 以太网 N-API 模块 |

**源码**：
- `ethernet_module.cpp`
- `ethernet_async_work.cpp`
- `*_context.cpp`（各类 Context）

**依赖**：
- `//foundation/communication/netmanager_base:net_conn_manager_if`
- `//foundation/communication/netmanager_base:net_exec`（待确认）

---

### Sharing N-API

**Target 定义**：`frameworks/js/napi/sharing/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `sharing` | ohos_shared_library | libnet_sharing.z.so | 网络共享 N-API 模块 |

**源码**：
- `netshare_module.cpp`
- `netshare_async_work.cpp`
- `netshare_exec.cpp`
- `*_context.cpp`

---

### MDNS N-API

**Target 定义**：`frameworks/js/napi/mdns/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `mdns` | ohos_shared_library | libnet_mdns.z.so | MDNS N-API 模块 |

---

### VPN N-API

**Target 定义**：`frameworks/js/napi/vpn/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `vpn` | ohos_shared_library | libnet_vpn.z.so | VPN N-API 模块 |

---

### NetFirewall N-API

**Target 定义**：`frameworks/js/napi/netfirewall/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `netfirewall` | ohos_shared_library | libnet_netfirewall.z.so | 防火墙 N-API 模块 |

---

## 关键 Inner Kit Targets

### Ethernet Inner Kit

**Target 定义**：`interfaces/innerkits/ethernetclient/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `ethernet_manager_if` | ohos_shared_library | libethernet_manager_if.z.so | 以太网客户端 Proxy |

**源码**：
- `src/proxy/*.cpp` - IPC Proxy 实现

**依赖**：
- `//foundation/communication/netmanager_ext:ethernet_manager`（服务）

---

### Sharing Inner Kit

**Target 定义**：`interfaces/innerkits/netshareclient/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `net_tether_manager_if` | ohos_shared_library | libnet_tether_manager_if.z.so | 网络共享客户端 Proxy |

---

### Firewall Inner Kit

**Target 定义**：`interfaces/innerkits/netfirewallclient/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `netfirewall_manager_if` | ohos_shared_library | libnetfirewall_manager_if.z.so | 防火墙客户端 Proxy |

---

## 关键 Service Targets

### Ethernet Service

**Target 定义**：`services/ethernetmanager/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `ethernet_manager` | ohos_shared_library | libethernet_manager.z.so | 以太网 SA（System Ability） |
| `ethernet_interfaces` | source_set | - | 以太网接口头文件 |

**SA 配置**：`sa_profile/ethernet_manager_profile.json`（待确认路径）

---

### MDNS Service

**Target 定义**：`services/mdnsmanager/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `mdns_manager` | ohos_shared_library | libmdns_manager.z.so | MDNS SA |

**SA ID**：1161
**进程**：mdnsmanager
**配置文件**：sa_profile/1161.json

---

### NetFirewall Service

**Target 定义**：`services/netfirewallmanager/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `netfirewall_manager` | ohos_shared_library | libnetfirewall_manager.z.so | 防火墙 SA |
| `netfirewall_default_rule` | ohos_executable | netfirewall_default_rule | 默认规则工具 |
| `netfirewall_policy_manager` | ohos_shared_library | libnetfirewall_policy_manager.z.so | 策略管理 |

**SA ID**：8300
**进程**：netmanager
**配置文件**：sa_profile/8300.json

---

### NetworkSlice Service

**Target 定义**：`services/networkslicemanager/BUILD.gn`

| 子 Target | 类型 | 输出文件 | 用途 |
|----------|------|----------|------|
| `networkslice_manager` | ohos_shared_library | libnetworkslice_manager.z.so | 网络切片 SA |

**SA ID**：8301
**进程**：netmanager
**配置文件**：sa_profile/8301.json

---

## 依赖关系

### 外部组件依赖（来自 bundle.json）

| 依赖组件 | 用途 | bundle.json 行号 |
|---------|------|---------------|
| `bounds_checking_function` | 内存安全检查 | bundle.json:53 |
| `ipc` | IPC 通信 | bundle.json:54 |
| `safwk` | SA 框架 | bundle.json:55 |
| `napi` | N-API 桥接 | bundle.json:56 |
| `dhcp` | DHCP 客户端 | bundle.json:57 |
| `hilog` | 日志系统 | bundle.json:58 |
| `netmanager_base` | 基础网络服务 | bundle.json:59 |
| `eventhandler` | 事件循环 | bundle.json:60 |
| `bluetooth` | 蓝牙支持 | bundle.json:61 |
| `hisysevent` | 系统事件 | bundle.json:62 |
| `access_token` | 权限令牌 | bundle.json:71 |
| `samgr` | SA 管理器 | bundle.json:65 |
| `usb_manager` | USB 管理 | bundle.json:66 |
| `drivers_interface_usb` | USB 驱动接口 | bundle.json:67,71 |
| `wifi` | WiFi 支持 | bundle.json:68 |
| `bundle_framework` | Bundle 框架 | bundle.json:69 |
| `ability_runtime` | Ability 运行时 | bundle.json:70 |
| `cJSON` | JSON 解析 | bundle.json:72 |
| `common_event_service` | 公共事件服务 | bundle.json:73 |
| `hitrace` | 性能跟踪 | bundle.json:74 |
| `window_manager` | 窗口管理（VPN 对话框） | bundle.json:75 |
| `openssl` | 加密支持 | bundle.json:84 |

**代码证据**：bundle.json:51-97

### 内部依赖

各模块通过 IPC 调用其他模块的 SA：

| 调用方 | 被调用方 | 依赖类型 |
|-------|---------|---------|
| VPN Manager | Ethernet Manager | IPC（获取网络状态） |
| Sharing Manager | Ethernet Manager | IPC（获取接口信息） |

---

## 条件编译

### 基于特性的条件编译

```gn
if (netmanager_ext_feature_ethernet) {
  # 以太网相关 targets
}

if (netmanager_ext_feature_vpn) {
  # VPN 相关 targets
}
```

**代码证据**：BUILD.gn:18-22, 27-35 等

### 基于系统组件的条件编译

```gn
if (defined(global_parts_info.communication_wifi) &&
    global_parts_info.communication_wifi) {
  communication_wifi_switch_enable = true
}
```

**代码证据**：netmanager_ext_config.gni:103-107

---

## 构建产物映射

### Target → 产物映射

| Target 类型 | 产物后缀 | 示例 |
|-----------|----------|-------|
| `ohos_shared_library` | `.z.so` | libnet_ethernet.z.so |
| `ohos_static_library` | `.a` | libnetmanager_base.a |
| `ohos_executable` | 无 | netfirewall_default_rule |
| `source_set` | 无 | 头文件集合 |

---

## 相关跳转

- [编译产物](07_Build_Artifacts.md) - 产物清单与安装路径
- [项目定位](01_Project_Positioning.md) - 特性开关说明
- [目录结构](02_Directory_Structure.md) - 文件组织
