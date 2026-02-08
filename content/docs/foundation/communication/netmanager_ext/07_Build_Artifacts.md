# 编译产物文档

## 目的

本文档说明 Net Manager Ext 的编译产物清单、安装路径、运行时加载关系，帮助开发者理解构建输出。

---

## 产物总览

### 产物类型与后缀

| Target 类型 | 产物后缀 | 安装位置 |
|-----------|----------|----------|
| `ohos_shared_library` | `.z.so` | `/system/lib64/` 或 `/vendor/lib64/` |
| `ohos_static_library` | `.a` | 链接到依赖模块 |
| `ohos_executable` | 无 | `/system/bin/` 或 `/vendor/bin/` |
| `ohos_hap` | `.hap` | 应用安装包（如适用） |

---

## N-API 模块产物

### 产物清单

| 模块名 | 产物文件 | 安装路径 | Target 定义 | 用途 |
|-------|----------|----------|-----------|------|
| Ethernet | libnet_ethernet.z.so | /system/lib64/ | frameworks/js/napi/ethernet/BUILD.gn | @ohos.net.ethernet |
| Sharing | libnet_sharing.z.so | /system/lib64/ | frameworks/js/napi/sharing/BUILD.gn | @ohos.net.sharing |
| MDNS | libnet_mdns.z.so | /system/lib64/ | frameworks/js/napi/mdns/BUILD.gn | @ohos.net.mdns |
| VPN | libnet_vpn.z.so | /system/lib64/ | frameworks/js/napi/vpn/BUILD.gn | @ohos.net.vpn |
| NetFirewall | libnet_netfirewall.z.so | /system/lib64/ | frameworks/js/napi/netfirewall/BUILD.gn | @ohos.net.netfirewall |

**代码证据**：bundle.json:130-212 - inner_kits 定义

### 加载方式

N-API 模块由 Ace/NArkTS 运行时在 JS 应用启动时加载：

```
应用启动
  ↓
import ethernet from '@ohos.net.ethernet'
  ↓
运行时加载 libnet_ethernet.z.so
  ↓
调用 napi_module_register
  ↓
暴露 JS 接口
```

---

## Inner Kit 产物

### 产物清单

| Inner Kit | 产物文件 | 安装路径 | Target 定义 | 对应 SA |
|-----------|----------|----------|-----------|---------|
| ethernet_manager_if | libethernet_manager_if.z.so | /system/lib64/ | interfaces/innerkits/ethernetclient/BUILD.gn | Ethernet SA |
| net_tether_manager_if | libnet_tether_manager_if.z.so | /system/lib64/ | interfaces/innerkits/netshareclient/BUILD.gn | Sharing SA |
| mdns_manager_if | libmdns_manager_if.z.so | /system/lib64/ | interfaces/innerkits/mdnsclient/BUILD.gn | MDNS SA |
| net_vpn_manager_if | libnet_vpn_manager_if.z.so | /system/lib64/ | interfaces/innerkits/netvpnclient/BUILD.gn | VPN SA |
| netfirewall_manager_if | libnetfirewall_manager_if.z.so | /system/lib64/ | interfaces/innerkits/netfirewallclient/BUILD.gn | Firewall SA |
| wearable_distributed_net_manager_if | libwearable_distributed_net_manager_if.z.so | /system/lib64/ | interfaces/innerkits/wearabledistributednetclient/BUILD.gn | Wearable Net SA |
| networkslice_manager_if | libnetworkslice_manager_if.z.so | /system/lib64/ | interfaces/innerkits/networksliceclient/BUILD.gn | Network Slice SA |
| vpn_extension_module | libvpn_extension_module.z.so | /system/lib64/ | interfaces/innerkits/vpnextension/BUILD.gn | VPN Extension |

**代码证据**：bundle.json:129-212 - inner_kits 定义

### 加载方式

Inner Kit 由依赖模块在启动时动态加载：

```
System Ability 启动
  ↓
dlopen("libethernet_manager_if.z.so")
  ↓
查找符号：GetProxyInstance()
  ↓
创建 IPC Proxy 实例
  ↓
通过 HDI/HBinder 调用 SA
```

---

## System Ability 产物

### 产物清单

| SA | SA ID | 产物文件 | 进程名 | 运行时机 | 配置文件 |
|----|-------|----------|--------|----------|----------|
| MDNS | 1161 | libmdns_manager.z.so | mdnsmanager | run-on-create: true | sa_profile/1161.json |
| NetFirewall | 8300 | libnetfirewall_manager.z.so | netmanager | run-on-create: false | sa_profile/8300.json |
| NetworkSlice | 8301 | libnetworkslice_manager.z.so | netmanager | run-on-create: true | sa_profile/8301.json |
| Wearable Distributed Net | 8400 | libwearable_distributed_net_manager.z.so | netmanager | run-on-create: false | sa_profile/8400.json |

**代码证据**：
- sa_profile/1161.json:2-12
- sa_profile/8300.json:2-12
- sa_profile/8301.json:2-14
- sa_profile/8400.json:2-14

### SA 加载流程

```
系统启动
  ↓
读取 sa_profile/*.json
  ↓
根据 SA ID 注册到 SAMGR
  ↓
根据 run-on-create 决定启动时机
  ↓
启动进程（如 mdnsmanager）
  ↓
dlopen("libmdns_manager.z.so")
  ↓
调用 SA 的 OnStart()
  ↓
SA 准备服务
```

---

## 可执行文件产物

### 产物清单

| 可执行文件 | 用途 | Target 定义 | 安装路径 |
|-----------|------|-----------|----------|
| netfirewall_default_rule | 设置默认防火墙规则 | services/netfirewallmanager/BUILD.gn | /system/bin/ |

**代码证据**：BUILD.gn - netfirewall_packages 包含该目标

### 加载方式

可执行文件由 init 进程在系统启动时调用：

```
系统启动
  ↓
init 进程读取 init rc 文件
  ↓
执行 netfirewall_default_rule
  ↓
配置 iptables/nftables 规则
```

---

## 配置文件产物

### SA Profile 文件

| 文件 | 用途 | 安装位置 |
|------|------|----------|
| 1161.json | MDNS SA 配置 | /etc/profile/ |
| 8300.json | NetFirewall SA 配置 | /etc/profile/ |
| 8301.json | NetworkSlice SA 配置 | /etc/profile/ |
| 8400.json | Wearable Distributed Net SA 配置 | /etc/profile/ |

**代码证据**：sa_profile/BUILD.gn: 构建这些配置文件

### Init 配置文件

| 文件 | 用途 | 安装位置 |
|------|------|----------|
| mdnsmanager.rc | MDNS 进程启动配置 | /etc/init/ |
| vpnmanager.cfg | VPN 服务配置 | /etc/init/ |
| ethernet.cfg | 以太网服务配置 | /etc/init/ |
| mdnsmanager_trust | MDNS 信任配置 | /etc/ |

**代码证据**：
- services/etc/init/BUILD.gn: 定义这些配置文件
- BUILD.gn:110, 111 - 在 service_group 中包含

---

## 运行时依赖关系

### 启动顺序

```
1. Init 进程
   ↓
2. SAMGR (System Ability Manager)
   ↓
3. netmanager 基础进程
   ├─ NetFirewall (8300, lazy)
   ├─ NetworkSlice (8301, immediate)
   └─ Wearable Distributed Net (8400, lazy)
   ↓
4. mdnsmanager 进程
   └─ MDNS (1161, immediate)
   ↓
5. N-API 模块加载（按需）
   └─ 应用启动时加载 libnet_*.z.so
```

### IPC 通信关系

```
┌─────────────────┐
│  N-API 进程   │  ← libnet_ethernet.z.so
└────────┬────────┘
         │ IPC
         ↓
┌─────────────────┐
│  netmanager 进程 │  ← SA 进程
│  ├─ Firewall (8300)
│  ├─ NetworkSlice (8301)
│  └─ Wearable Net (8400)
└─────────────────┘
         │ IPC
         ↓
┌─────────────────┐
│ mdnsmanager 进程│  ← SA 进程
│  └─ MDNS (1161)
└─────────────────┘
```

---

## 产物大小估算

### ROM 占用

| 组件 | ROM 大小 | 说明 |
|-------|---------|------|
| 基础框架 | ~2MB | bundle.json:49 |
| 以太网模块 | ~200KB | 估算 |
| 共享模块 | ~300KB | 估算 |
| VPN 模块 | ~500KB | 估算 |
| MDNS 模块 | ~150KB | 估算 |
| 防火墙模块 | ~100KB | 估算 |

**注**：实际大小取决于编译优化级别和特性开关。

**代码证据**：bundle.json:49 - `"rom": "2MB"`

### RAM 占用

| 组件 | RAM 大小 | 说明 |
|-------|---------|------|
| 基础运行时 | ~500KB | bundle.json:50 |
| SA 服务（活跃）| ~100-300KB/SA | 取决于连接数 |

**代码证据**：bundle.json:50 - `"ram": "500KB"`

---

## 产物验证

### 查看已安装产物

```bash
# 查看 N-API 模块
ls -l /system/lib64/libnet_*.z.so

# 查看 SA 库
ls -l /system/lib64/lib*_manager.z.so
ls -l /system/lib64/libmdns_manager.z.so

# 查看 SA 配置
ls -l /etc/profile/*.json

# 查看 Init 配置
ls -l /etc/init/*.{rc,cfg}
```

### 检查 SA 注册状态

```bash
# 查看已注册的 SA
sa_list | grep -E "8300|8301|8400|1161"
```

---

## 相关跳转

- [GN Build 文档](06_GN_Build.md) - 构建配置与 Targets
- [目录结构](02_Directory_Structure.md) - 文件组织
- [安全风险评审](08_Security_Review.md) - 产物安全分析
