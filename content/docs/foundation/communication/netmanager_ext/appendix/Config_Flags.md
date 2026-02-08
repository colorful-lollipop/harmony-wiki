# 配置标志（附录）

## 目的

本文档列出 Net Manager Ext 的关键编译宏、Feature Flags 和配置选项，帮助开发者进行条件编译和功能裁剪。

---

## Feature Flags（特性开关）

### 以太网相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_ethernet` | true | 以太网模块编译 | netmanager_ext_config.gni:62 |
| `netmanager_ext_extensible_authentication` | false | 可扩展认证（EAP） | netmanager_ext_config.gni:75 |

**效果**：
- `true`：编译以太网 N-API、SA、Inner Kit
- `false`：跳过所有以太网相关代码

**相关 Targets**：
- `ethernet_packages` (BUILD.gn:25)
- `common_ext_packages` (BUILD.gn:18-22)

---

### 网络共享相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_share` | true | 网络共享模块 | netmanager_ext_config.gni:63 |
| `netmanager_ext_share_traffic_limit_enable` | false | 流量限制功能 | netmanager_ext_config.gni:74 |
| `netmanager_ext_share_notification_enable` | false | 共享通知 | netmanager_ext_config.gni:73 |

**效果**：
- `true`：编译共享 N-API、SA、流量管理
- `false`：禁用网络共享功能

**相关 Targets**：
- `share_packages` (BUILD.gn:38-48)

---

### MDNS 相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_mdns` | true | mDNS 服务发现 | netmanager_ext_config.gni:64 |

**效果**：
- `true`：编译 MDNS N-API、SA（SA ID: 1161）
- `false`：禁用 mDNS 功能

**相关 Targets**：
- `mdns_packages` (BUILD.gn:50-59)

---

### VPN 相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_vpn` | true | VPN 基础功能 | netmanager_ext_config.gni:67 |
| `netmanager_ext_feature_sysvpn` | false | 系统 VPN | netmanager_ext_config.gni:66 |
| `netmanager_ext_feature_vpnext` | true | VPN 扩展能力 | netmanager_ext_config.gni:69 |
| `netmanager_ext_feature_vpn_for_user0` | false | User0 VPN | netmanager_ext_config.gni:68 |

**效果**：
- `netmanager_ext_feature_vpn`：控制 VPN N-API 和 SA
- `netmanager_ext_feature_sysvpn`：控制系统级 VPN
- `netmanager_ext_feature_vpnext`：控制 VPN Extension Ability
- `netmanager_ext_feature_vpn_for_user0`：User0 VPN 支持

**相关 Targets**：
- `vpn_packages` (BUILD.gn:72-80)
- `vpn_ext_packages` (BUILD.gn:82-93)

---

### 网络防火墙相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_net_firewall` | false | 网络防火墙 | netmanager_ext_config.gni:65 |

**效果**：
- `true`：编译防火墙 N-API、SA（SA ID: 8300）
- `false`：禁用防火墙功能（默认）

**相关 Targets**：
- `netfirewall_packages` (BUILD.gn:61-70)

---

### 网络切片相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_networkslice` | false | 网络切片管理 | netmanager_ext_config.gni:76 |

**效果**：
- `true`：编译网络切片 SA（SA ID: 8301）
- `false`：禁用网络切片功能（默认）

**相关 Targets**：
- `networkslice_packages` (BUILD.gn:105-112)

---

### 可穿戴分布式网络相关

| 宏/变量 | 默认值 | 控制内容 | 位置 |
|----------|--------|---------|------|
| `netmanager_ext_feature_wearable_distributed_net` | false | 可穿戴分布式网络 | netmanager_ext_config.gni:71 |

**效果**：
- `true`：编译 Wearable SA（SA ID: 8400）
- `false`：禁用可穿戴分布式网络（默认）

**相关 Targets**：
- `wearable_distributed_net_packages` (BUILD.gn:95-103)

---

## 编译配置宏

### 优化级别

| 宏组 | 内容 | 用途 | 位置 |
|-------|------|------|------|
| `common_cflags` | `-Os -O2` | 代码优化 | netmanager_ext_config.gni:81-87 |
| `memory_optimization_cflags` | `-fvisibility=hidden` | 符号可见性 | netmanager_ext_config.gni:89-91 |
| `memory_optimization_ldflags` | `-Wl,--gc-sections` | 链接优化 | netmanager_ext_config.gni:98-101 |

**Fortify**：
```cpp
#define _FORTIFY_SOURCE=2
```
- 编译时插入运行时检查
- 检测缓冲区溢出、格式化字符串漏洞

---

### 系统依赖检测

| 变量 | 依赖组件 | 检测逻辑 | 位置 |
|------|---------|---------|------|
| `communication_wifi_switch_enable` | `wifi` | netmanager_ext_config.gni:104-107 |
| `communication_bluetooth_switch_enable` | `bluetooth` | netmanager_ext_config.gni:109-113 |
| `usb_manager_enable` | `usb_usb_manager` | netmanager_ext_config.gni:115-118 |
| `battery_manager_switch_enable` | `powermgr_battery_manager` | netmanager_ext_config.gni:120-124 |

**检测逻辑**：
```gn
if (defined(global_parts_info.communication_wifi) &&
    global_parts_info.communication_wifi) {
  communication_wifi_switch_enable = true
}
```

---

## SA ID 配置

### System Ability ID 分配

| SA | ID | 进程名 | run-on-create | 配置文件 |
|----|-----|---------|--------------|----------|
| MDNS | 1161 | mdnsmanager | true | sa_profile/1161.json |
| NetFirewall | 8300 | netmanager | false | sa_profile/8300.json |
| NetworkSlice | 8301 | netmanager | true | sa_profile/8301.json |
| Wearable Distributed Net | 8400 | netmanager | false | sa_profile/8400.json |

**说明**：
- SA ID 由系统分配，16-bit 整数
- `run-on-create: true`：系统启动时立即启动
- `run-on-create: false`：首次访问时启动（懒加载）

---

## N-API 模块名

### JS 命名空间映射

| N-API 模块文件 | JavaScript 模块名 | 导出对象 |
|--------------|---------------|---------|
| ethernet_module.cpp | net.ethernet | {setIfaceConfig, getIfaceConfig, ...} |
| netshare_module.cpp | net.sharing | {startSharing, stopSharing, ...} |
| mdns_module.cpp | net.mdns | {addLocalService, ...} |
| vpn_module.cpp | net.vpn | {setUp, destroy, ...} |
| netfirewall_module.cpp | net.netfirewall | {addFirewallRule, ...} |

**代码证据**：
- ethernet_module.cpp:154 - `.nm_modname = "net.ethernet"`
- netshare_module.cpp:38 - `.nm_modname = "net.sharing"`

---

## 系统能力（Syscap）

### 定义方式

**位置**：`bundle.json:22-28`

```json
"syscap": [
  "SystemCapability.Communication.NetManager.Ethernet",
  "SystemCapability.Communication.NetManager.NetSharing",
  "SystemCapability.Communication.NetManager.MDNS",
  "SystemCapability.Communication.NetManager.Vpn",
  "SystemCapability.Communication.NetManager.NetFirewall",
  "SystemCapability.Communication.NetManager.Eap = false"
]
```

**用途**：
- 应用在 `module.json5` 中声明需要的能力
- 系统根据能力决定是否安装本模块

---

## 调试选项

### 日志级别

| 宏 | 值 | 效果 | 位置 |
|-----|-----|------|------|
| `enable_netmgr_ext_debug` | true | 启用详细日志 | netmanager_ext_config.gni:60 |
| `use_js_debug` | false | JavaScript 调试（未使用）| netmanager_ext_config.gni:56 |

**日志宏**：
- `NETMGR_EXT_LOGD()` - DEBUG 级别
- `NETMGR_EXT_LOGI()` - INFO 级别
- `NETMGR_EXT_LOGE()` - ERROR 级别

**使用方式**：
```cpp
NETMGR_EXT_LOGI("SetIfaceConfig: iface=%s", iface);
NETMGR_EXT_LOGE("Permission denied: uid=%d", uid);
```

---

## 权限字符串

### 权限定义

| 权限字符串 | 用途 | 授予对象 | 位置 |
|-----------|------|---------|------|
| `ohos.permission.GET_NETWORK_INFO` | 读取网络信息 | 普通应用 | bundle.json 依赖 |
| `ohos.permission.GET_ETHERNET_LOCAL_MAC` | 读取 MAC 地址 | 系统应用 | ethernet_service.cpp:211 |
| `ohos.permission.CONNECTIVITY_INTERNAL` | 内部连接管理 | 系统应用 | ethernet_service.cpp:226 |
| `ohos.permission.MANAGE_VPN` | 管理 VPN | 系统应用 | networkvpn_service.cpp:674 |
| `ohos.permission.MANAGE_NET_FIREWALL` | 管理防火墙 | 系统应用 | netfirewall_stub.cpp:33 |
| `ohos.permission.MANAGE_ENTERPRISE_WIFI_CONNECTION` | 管理 WiFi | 系统应用 | ethernet_service.cpp:451 |

**权限检查代码**：
```cpp
namespace Permission {
constexpr const char* GET_NETWORK_INFO = "ohos.permission.GET_NETWORK_INFO";
constexpr const char* CONNECTIVITY_INTERNAL = "ohos.permission.CONNECTIVITY_INTERNAL";
constexpr const char* MANAGE_VPN = "ohos.permission.MANAGE_VPN";
}
```

---

## 编译产物配置

### 共享库类型

| Target 类型 | 产物后缀 | 安装路径 |
|-----------|----------|----------|
| `ohos_shared_library` | `.z.so` | /system/lib64/ |
| `ohos_static_library` | `.a` | 链接到依赖 |

### 可执行文件

| Target 类型 | 产物后缀 | 安装路径 |
|-----------|----------|----------|
| `ohos_executable` | 无 | /system/bin/ |

---

## 运行时参数

### 内核参数

| 参数 | 用途 | 示例 |
|------|------|------|
| `debug.netmanager_ext` | 调试开关 | `param set debug.netmanager_ext 1` |
| `persist.debug.netmanager_ext` | 持久调试 | `param set persist.debug.netmanager_ext 1` |

### 限制参数

| 参数 | 用途 | 默认值 |
|------|------|--------|
| 最大共享会话数 | 限制并发共享 | 10 |
| 最大防火墙规则数 | 限制规则数量 | 100 |

---

## 自定义配置

### 构建配置文件

**位置**：项目根目录

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 根构建入口 |
| `bundle.json` | 组件元数据 |
| `netmanager_ext_config.gni` | GN 配置变量 |
| `OAT.xml` | 开放审计工具配置 |

**修改方式**：
```bash
# 编辑配置文件后重新构建
vi netmanager_ext_config.gni
./build.sh --product-name <product>
```

---

## 相关跳转

- [GN Build 文档](06_GN_Build.md) - 构建系统详解
- [编译产物](07_Build_Artifacts.md) - 产物清单
- [项目定位](01_Project_Positioning.md) - 特性开关说明
