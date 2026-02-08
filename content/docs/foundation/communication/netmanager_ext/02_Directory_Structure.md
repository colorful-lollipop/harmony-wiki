# 目录结构与模块职责

## 目的

本文档说明 Net Manager Ext 的完整目录结构、各目录/文件的职责划分，帮助开发者快速定位代码。

---

## 顶层目录概览

```
netmanager_ext/
├── figures/                          # README 文档使用的图片资源
├── frameworks/                        # 框架层（N-API 绑定 + Native 实现）
│   ├── js/                          # JavaScript 接口实现（N-API 模块）
│   │   └── napi/                   # 各模块 N-API 实现
│   ├── native/                      # Native C++ 实现框架
│   └── ets/                        # ArkTS/ETS 接口实现
├── interfaces/                       # 接口定义
│   ├── innerkits/                   # 部件间内部接口（IPC Proxy/Stub）
│   └── kits/                       # 应用层接口定义（如需）
├── sa_profile/                      # System Ability 配置文件
├── services/                        # 核心服务实现（SA）
├── utils/                           # 通用工具类
├── resource/                        # 资源文件
├── test/                           # 单元测试（本文档忽略）
├── BUILD.gn                        # GN 构建入口
├── bundle.json                      # 组件元数据配置
└── netmanager_ext_config.gni         # GN 构建配置
```

---

## 目录职责详解

### figures/

**职责**：存放 README 文档使用的架构图和说明图

**关键文件**：
- `net_manager_arch_zh.png` - 网络管理架构图（README_zh.md:20）

**代码证据**：README_zh.md:20

---

### frameworks/js/

**职责**：实现 N-API 模块，将 JavaScript 调用桥接到 C++ 服务

**子目录结构**：

```
frameworks/js/napi/
├── ethernet/                      # 以太网 N-API 模块
├── sharing/                       # 网络共享 N-API 模块
├── mdns/                         # mDNS N-API 模块
├── vpn/                          # VPN N-API 模块
├── netfirewall/                   # 防火墙 N-API 模块
├── extensionability/               # VPN Extension Ability N-API
├── extensioncontext/               # VPN Extension Context N-API
└── vpnext/                       # VPN Extension N-API
```

**关键文件**：

| 模块 | N-API 模块文件 | 注册函数 | 模块名 |
|------|--------------|---------|--------|
| ethernet | ethernet_module.cpp | RegisterEthernetInterface | net.ethernet |
| sharing | netshare_module.cpp | InitNetShareModule | net.sharing |
| mdns | mdns_module.cpp | InitMdnsModule | net.mdns |
| vpn | vpn_module.cpp | RegisterVpnInterface | net.vpn |
| netfirewall | netfirewall_module.cpp | RegisterNetFirewallInterface | net.netfirewall |

**代码证据**：
- ethernet_module.cpp:149-157 - N-API 模块定义
- netshare_module.cpp:206-214 - N-API 模块定义

**职责边界**：
- ✅ JS 接口绑定（`napi_define_properties`）
- ✅ 参数解析与校验
- ✅ 异步工作队列（`napi_create_threadsafe_function`）
- ✅ 错误码映射
- ❌ 核心业务逻辑 → 委托给 `services/` 中的 SA

---

### frameworks/native/

**职责**：Native C++ 框架实现，包括 IPC Proxy 和通用工具

**子目录结构**：

```
frameworks/native/
├── ethernetclient/                # 以太网客户端（IPC Proxy）
├── netshareclient/              # 网络共享客户端
├── mdnsclient/                # mDNS 客户端
├── netvpnclient/              # VPN 客户端
├── netfirewallclient/          # 防火墙客户端
├── vpnextension/              # VPN 扩展客户端
├── wearabledistributednetclient/  # 可穿戴分布式网络客户端
└── networksliceclient/         # 网络切片客户端
```

**代码证据**：interfaces/innerkits/*/src/proxy/*.cpp

**职责边界**：
- ✅ IPC Proxy 实现（调用远程 SA）
- ✅ 回调 Stub/Proxy（服务端回调客户端）
- ❌ SA 核心实现 → 在 `services/` 中

---

### interfaces/innerkits/

**职责**：定义部件间接口头文件，供模块内部使用

**子目录**：

```
interfaces/innerkits/
├── ethernetclient/                # {ethernet}_client.h, proxy/*.h
├── netshareclient/              # {networkshare}_client.h, proxy/*.h
├── mdnsclient/                # mdns_client.h
├── netvpnclient/              # {networkvpn}_client.h
├── netfirewallclient/          # netfirewall_client.h
├── vpnextension/              # vpn_extension_module_loader.h
├── wearabledistributednetclient/  # wearable_distributed_net_client.h
└── networksliceclient/         # networkslice_client.h
```

**代码证据**：bundle.json:129-212 - inner_kits 定义

**职责边界**：
- ✅ 接口头文件定义
- ✅ IPC 接口（I*Service）
- ✅ 数据结构定义（Parcel 序列化）
- ❌ 实现代码 → 在 `frameworks/native/` 和 `services/` 中

---

### sa_profile/

**职责**：System Ability 注册配置，定义 SA ID、进程归属、启动时机

**关键文件**：

| SA ID | 服务名 | 配置文件 | 进程 | run-on-create |
|-------|---------|---------|--------|--------------|
| 1161 | MDNS Service | 1161.json | mdnsmanager | true |
| 8300 | Net Firewall | 8300.json | netmanager | false |
| 8301 | Network Slice | 8301.json | netmanager | true |
| 8400 | Wearable Distributed Net | 8400.json | netmanager | false |

**代码证据**：
- sa_profile/1161.json:2-12 - MDNS SA 配置
- sa_profile/8300.json:2-12 - Firewall SA 配置
- sa_profile/8301.json:2-14 - Network Slice SA 配置
- sa_profile/8400.json:2-14 - Wearable Distributed Net SA 配置

**职责边界**：
- ✅ SA 元数据配置
- ❌ SA 实现代码 → 在 `services/` 中

---

### services/

**职责**：实现核心业务逻辑，作为 System Ability 提供服务

**子目录结构**：

```
services/
├── ethernetmanager/                # 以太网管理服务
│   ├── src/
│   │   ├── ethernet_service.cpp      # SA 主服务类
│   │   ├── ethernet_stub.cpp      # IPC Stub 实现
│   │   ├── ethernet_dhcp_controller.cpp
│   │   └── ...
│   └── BUILD.gn
├── networksharemanager/          # 网络共享服务
│   ├── src/
│   │   ├── networkshare_service.cpp
│   │   ├── networkshare_main_statemachine.cpp
│   │   ├── networkshare_tracker.cpp
│   │   └── ...
│   └── BUILD.gn
├── mdnsmanager/                # mDNS 服务
│   ├── src/
│   │   ├── mdns_service.cpp
│   │   ├── mdns_protocol_impl.cpp
│   │   └── ...
│   └── BUILD.gn
├── vpnmanager/                 # VPN 服务
│   ├── src/
│   │   ├── networkvpn_service.cpp
│   │   ├── virtual_vpn_ctl.cpp
│   │   ├── l2tp_vpn_ctl.cpp
│   │   └── ...
│   └── BUILD.gn
├── netfirewallmanager/          # 防火墙服务
│   ├── src/
│   │   ├── netfirewall_service.cpp
│   │   ├── netfirewall_stub.cpp
│   │   └── ...
│   └── BUILD.gn
├── networkslicemanager/         # 网络切片服务
│   ├── src/
│   │   ├── networkslice_service.cpp
│   │   ├── networkslice_stub.cpp
│   │   └── ...
│   └── BUILD.gn
├── wearabledistributednetmanager/ # 可穿戴分布式网络服务
│   ├── src/
│   │   ├── wearable_distributed_net_service.cpp
│   │   └── ...
│   └── BUILD.gn
└── etc/init/                    # 启动配置
    ├── ethernet.cfg
    ├── mdnsmanager.rc
    ├── vpnmanager.cfg
    └── ...
```

**代码证据**：
- services/ethernetmanager/src/ethernet_service.cpp:211-527 - Ethernet SA 实现
- services/networksharemanager/src/networkshare_service.cpp:172-560 - Sharing SA 实现
- services/vpnmanager/src/networkvpn_service.cpp:674 - VPN 权限检查
- services/netfirewallmanager/src/netfirewall_service.cpp:30 - Firewall SA 实现

**职责边界**：
- ✅ System Ability 实现
- ✅ 业务逻辑实现
- ✅ 权限验证（`NetManagerPermission::CheckPermission`）
- ✅ 网络状态管理
- ❌ JS 接口绑定 → 在 `frameworks/js/napi/` 中

---

### utils/

**职责**：提供跨模块的通用工具类

**子目录**：

```
utils/
└── event_report/                  # 网络事件上报工具
    ├── event_report_center.cpp
    └── event_report_center.h
```

**代码证据**：utils/BUILD.gn - event_report 目标

**职责边界**：
- ✅ 通用工具类
- ✅ 事件上报中心
- ❌ 业务逻辑 → 在各 `services/` 模块中

---

### resource/

**职责**：资源文件（如配置模板、图标等）

**代码证据**：resource/BUILD.gn

---

### BUILD.gn

**职责**：根目录 GN 构建入口，定义顶层构建组

**关键构建组**：

| 组名 | Target 类型 | 包含内容 | BUILD.gn 行号 |
|------|-----------|---------|-------------|
| `common_ext_packages` | group | 公共包 | 16-23 |
| `ethernet_packages` | group | 以太网相关 | 25-36 |
| `share_packages` | group | 网络共享相关 | 38-48 |
| `mdns_packages` | group | mDNS 相关 | 50-59 |
| `netfirewall_packages` | group | 防火墙相关 | 61-70 |
| `vpn_packages` | group | VPN 相关 | 72-80 |
| `vpn_ext_packages` | group | VPN 扩展相关 | 82-93 |
| `wearable_distributed_net_packages` | group | 可穿戴分布式网络 | 95-103 |
| `networkslice_packages` | group | 网络切片 | 105-112 |

**代码证据**：BUILD.gn:16-112

---

### bundle.json

**职责**：OpenHarmony 组件元数据配置

**关键内容**：
- 组件信息（名称、版本、子系统）
- 系统能力定义（syscap）
- 特性开关列表（features）
- 依赖组件（deps.components）
- 内部 Kit 定义（build.inner_kits）
- 构建组（build.group_type）

**代码证据**：bundle.json:1-218

---

### netmanager_ext_config.gni

**职责**：GN 构建配置文件，定义编译标志和路径

**关键内容**：
- 源码路径变量（如 `ETHERNETMANAGER_SOURCE_DIR`）
- 特性开关声明（`declare_args`）
- 编译标志（`common_cflags`, `memory_optimization_cflags`）

**代码证据**：netmanager_ext_config.gni:14-127

---

## 模块职责总结

| 模块 | N-API | SA 实现 | IPC Proxy | 职责 |
|------|--------|----------|-----------|------|
| ethernet | frameworks/js/napi/ethernet | services/ethernetmanager | frameworks/native/ethernetclient | 以太网连接配置与状态管理 |
| sharing | frameworks/js/napi/sharing | services/networksharemanager | frameworks/native/netshareclient | WiFi/蓝牙/USB 网络共享 |
| mdns | frameworks/js/napi/mdns | services/mdnsmanager | frameworks/native/mdnsclient | 多播 DNS 服务发现 |
| vpn | frameworks/js/napi/vpn | services/vpnmanager | frameworks/native/netvpnclient | VPN 连接管理 |
| netfirewall | frameworks/js/napi/netfirewall | services/netfirewallmanager | frameworks/native/netfirewallclient | 网络防火墙规则管理 |
| wearable_distributed | - | services/wearabledistributednetmanager | frameworks/native/wearabledistributednetclient | 可穿戴设备分布式网络 |
| networkslice | - | services/networkslicemanager | frameworks/native/networksliceclient | 网络切片管理 |

---

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 项目边界与核心能力
- [架构说明](03_Architecture.md) - 架构图与模块依赖
- [GN Build 文档](06_GN_Build.md) - 构建系统详解
