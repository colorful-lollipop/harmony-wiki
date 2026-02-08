# NetManager Base - 项目概览

## 1. 项目定位

**NetManager Base** 是 OpenHarmony 通信子系统的核心基础组件，负责提供设备网络管理的底层能力支持。

### 1.1 所属子系统

| 属性 | 值 |
|------|-----|
| **子系统** | Communication (通信) |
| **组件名** | netmanager_base |
| **SysCap** | SystemCapability.Communication.NetManager.Core |
| **版本** | 3.1.0 |

### 1.2 核心职责

根据代码证据 [`bundle.json:1-5`](../../bundle.json):

```json
{
    "name": "@ohos/netmanager_base",
    "version": "3.1.0",
    "description": "net manager service"
}
```

本项目提供以下核心能力：

1. **网络连接管理 (NetConnService)** - 管理网络生命周期、网络选择、HTTP 代理
2. **网络策略管理 (NetPolicyService)** - 应用网络策略、防火墙规则、流量配额
3. **流量统计管理 (NetStatsService)** - 网络流量统计、历史数据、告警通知
4. **本地网络服务 (NetManagerNative)** - 底层网络操作、DNS、iptables、BPF

---

## 2. 核心能力

### 2.1 功能模块

```
┌─────────────────────────────────────────────────────────────────────┐
│                        NetManager Base                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   连接管理    │  │   策略管理    │  │   统计管理    │              │
│  │ NetConnSvc   │  │ NetPolicySvc │  │ NetStatsSvc  │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                 │                       │
│         └─────────────────┴─────────────────┘                       │
│                           │                                         │
│                    ┌──────┴──────┐                                 │
│                    │ NetsysCtrl  │  ← 网络系统控制器                │
│                    └──────┬──────┘                                 │
│                           │                                         │
│                    ┌──────┴──────┐                                 │
│                    │  NetMgrNatv │  ← 本地网络服务                  │
│                    │   (DNS/    │    (BPF/iptables/路由)           │
│                    │  Firewall) │                                   │
│                    └─────────────┘                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 特性开关 (Feature Flags)

根据代码证据 [`netmanager_base_config.gni:46-64`](../../netmanager_base_config.gni):

| 特性 | 配置项 | 默认 | 说明 |
|------|--------|------|------|
| 网络防火墙 | `netmanager_base_enable_feature_net_firewall` | false | BPF 防火墙功能 |
| 可穿戴分布式网络 | `netmanager_base_enable_feature_wearable_distributed_net` | false | 可穿戴设备组网 |
| 系统 VPN | `netmanager_base_enable_feature_sysvpn` | false | 系统级 VPN |
| Hosts 管理 | `netmanager_base_enable_feature_hosts` | false | 自定义 hosts |
| 公共 DNS | `netmanager_base_enable_public_dns_server` | false | 公共 DNS 服务器 |
| 流量统计 | `netmanager_base_enable_traffic_statistic` | false | eBPF 流量统计 |
| PAC 代理 | `netmanager_base_enable_pac_proxy` | false | 代理自动配置 |
| 应用冻结 | `netmanager_base_enable_set_app_frozened` | false | 应用网络冻结 |

---

## 3. 运行环境

### 3.1 系统类型

根据代码证据 [`bundle.json:41-43`](../../bundle.json):

```json
"adapted_system_type": [
    "standard"
]
```

- **适配系统**: OpenHarmony Standard 系统
- **不支持**: Lite、Mini 系统

### 3.2 资源占用

根据代码证据 [`bundle.json:44-45`](../../bundle.json):

| 资源 | 占用 |
|------|------|
| ROM | 4.5MB |
| RAM | 10MB |

### 3.3 进程模型

```
┌─────────────────────────────────────────────┐
│           netmanager 进程                   │
│  ┌─────────────┐ ┌─────────────┐           │
│  │NetConnSvc   │ │NetPolicySvc │ ...       │
│  │  (SA 1151)  │ │  (SA 1152)  │            │
│  └─────────────┘ └─────────────┘           │
│                                             │
│  SA ID 分配:                               │
│  - 1151: NetConnService                    │
│  - 1152: NetPolicyService                  │
│  - 1153: NetStatsService                   │
│  - 1158: NetsysNativeService               │
│  - 1154-1157: 扩展功能                     │
└─────────────────────────────────────────────┘
```

---

## 4. 关键概念

### 4.1 网络术语

| 术语 | 说明 | 代码定义 |
|------|------|----------|
| **netId** | 网络标识符 | `int32_t`，见 `net_handle.h` |
| **supplierId** | 网络供应商 ID | `uint32_t`，WiFi/蜂窝网络等 |
| **NetCap** | 网络能力 | 枚举，如 `NET_CAPABILITY_INTERNET` |
| **NetBearType** | 网络承载类型 | 枚举，如 `BEARER_WIFI`、`BEARER_CELLULAR` |
| **NetSpecifier** | 网络规格 | 描述网络需求的结构体 |

### 4.2 System Ability (SA)

本项目使用 OpenHarmony SA 框架实现 IPC 服务：

- 服务继承 `SystemAbility` 类
- 通过 `DECLARE_SYSTEM_ABILITY` 宏声明
- 在 `sa_profile/` 目录配置 SA ID

示例代码 [`net_conn_service.h:63-67`](../../services/netconnmanager/include/net_conn_service.h):

```cpp
class NetConnService : public SystemAbility,
                       public INetActivateCallback,
                       public NetConnServiceStub,
                       public std::enable_shared_from_this<NetConnService> {
    DECLARE_SYSTEM_ABILITY(NetConnService)
```

### 4.3 IPC 通信

| 层级 | 实现 | 说明 |
|------|------|------|
| 接口定义 | `I*Service` | 服务接口 (如 `INetConnService`) |
| Stub | `*ServiceStub` | 服务端存根 |
| Proxy | `*ServiceProxy` | 客户端代理 |
| 通信机制 | Binder | OpenHarmony IPC |

---

## 5. 对外接口

### 5.1 JavaScript API

| 模块名 | 命名空间 | 说明 |
|--------|----------|------|
| `@ohos.net.connection` | `ohos.net.connection` | 网络连接管理 |
| `@ohos.net.policy` | `ohos.net.policy` | 网络策略管理 |
| `@ohos.net.statistics` | `ohos.net.statistics` | 流量统计查询 |
| `@ohos.net.network` | `ohos.net.network` | 网络基础能力 |

详见 [N-API 接口文档](03_NAPI_JS_API.md)

### 5.2 C++ 内部 API

| 库名 | 路径 | 说明 |
|------|------|------|
| `libnet_conn_manager_if.z.so` | `interfaces/innerkits/netconnclient/` | 连接管理客户端 |
| `libnet_policy_manager_if.z.so` | `interfaces/innerkits/netpolicyclient/` | 策略管理客户端 |
| `libnet_stats_manager_if.z.so` | `interfaces/innerkits/netstatsclient/` | 统计管理客户端 |
| `libnet_native_manager_if.z.so` | `interfaces/innerkits/netmanagernative/` | 原生服务客户端 |

详见 [内部 API 文档](04_Inner_API.md)

### 5.3 C API (NDK)

| 库名 | 路径 | 说明 |
|------|------|------|
| `libnet_connection.so` | `interfaces/kits/c/netconnclient/` | 连接管理 C 接口 |

---

## 6. 架构图

### 6.1 整体架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              应用层 (Apps)                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │   JS/TS应用   │  │   C++应用    │  │    C应用     │  │   ArkTS应用      │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘ │
└─────────┼─────────────────┼─────────────────┼────────────────────┼──────────┘
          │                 │                 │                    │
          └─────────────────┴─────────┬───────┴────────────────────┘
                                      ▼
                          ┌─────────────────────┐
                          │   框架接口层         │
                          │  (Framework Layer)  │
                          └──────────┬──────────┘
                                     │
          ┌──────────────────────────┼──────────────────────────┐
          │                          │                          │
          ▼                          ▼                          ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  连接管理服务     │    │  策略管理服务     │    │  统计管理服务     │
│ NetConnService   │    │ NetPolicyService │    │ NetStatsService  │
│    (SA 1151)     │    │    (SA 1152)     │    │    (SA 1153)     │
└────────┬─────────┘    └────────┬─────────┘    └────────┬─────────┘
         │                       │                       │
         │    ┌──────────────────┴───────────────────────┘
         │    │
         │    │         ┌─────────────────────┐
         │    │         │ NetsysController    │  ← 控制器
         │    │         └───────────┬─────────┘
         │    │                     │
         │    └──────────────▶┌─────┴───────────────────┐
         │                     │ NetManagerNative        │
         │                     │  (NetsysNativeService)  │
         │                     │    - DNS 管理           │
         │                     │    - 防火墙/iptables    │
         │                     │    - 路由管理           │
         │                     │    - BPF/eBPF           │
         │                     └─────────────┬───────────┘
         │                                   │
         └───────────────────────────────────┘
                                           ▼
                          ┌─────────────────────────────┐
                          │    Kernel / Netlink / BPF   │
                          └─────────────────────────────┘
```

### 6.2 数据流向

```
1. 网络连接管理流:
   App → NetConnClient → NetConnService → NetsysController → NetManagerNative → Kernel

2. 网络策略控制流:
   Settings → NetPolicyClient → NetPolicyService → NetsysController → iptables/BPF

3. 流量统计流:
   Kernel BPF → NetManagerNative → NetsysController → NetStatsService → Database

4. DNS解析流:
   App → DnsClient → NetManagerNative(DnsManager) → DNS Servers
```

---

## 7. 相关仓库

| 仓库 | 说明 |
|------|------|
| [communication_netmanager_base](https://gitee.com/openharmony/communication_netmanager_base) | 本仓库，基础网络管理 |
| [communication_netmanager_ext](https://gitee.com/openharmony/communication_netmanager_ext) | 扩展功能（VPN、Tethering、Ethernet） |
| [communication_netstack](https://gitee.com/openharmony/communication_netstack) | 网络协议栈 |

---

## 8. 文档索引

| 主题 | 文档 | 关键文件 |
|------|------|----------|
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) | 全仓库 |
| 架构设计 | [02_Architecture.md](02_Architecture.md) | `services/` |
| N-API 接口 | [03_NAPI_JS_API.md](03_NAPI_JS_API.md) | `frameworks/js/napi/` |
| 内部 API | [04_Inner_API.md](04_Inner_API.md) | `interfaces/innerkits/` |
| GN 构建 | [05_GN_Build.md](05_GN_Build.md) | `BUILD.gn` |
| 编译产物 | [06_Build_Artifacts.md](06_Build_Artifacts.md) | `bundle.json` |
| 安全风险 | [07_Security_Review.md](07_Security_Review.md) | `utils/common_utils/` |
| 问题定位 | [08_Troubleshooting.md](08_Troubleshooting.md) | 日志、配置 |

---

*生成时间: 2025-02-06*
