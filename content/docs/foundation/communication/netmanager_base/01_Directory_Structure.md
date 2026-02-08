# 目录结构与模块职责

## 1. 顶层目录结构

根据代码证据，netmanager_base 仓库采用分层架构设计：

```
foundation/communication/netmanager_base/
├── bpf/                          # BPF (Berkeley Packet Filter) 相关
├── common/                       # 公共代码 (ANI/Rust 相关)
├── frameworks/                   # 框架层实现
├── interfaces/                   # 接口定义层
├── resource/                     # 资源文件
├── sa_profile/                   # System Ability 配置文件
├── services/                     # 服务层实现 (IPC 服务端)
├── utils/                        # 工具类库
├── wiki/                         # 本文档
├── figures/                      # 架构图
├── bundle.json                   # 组件配置
└── netmanager_base_config.gni    # GN 构建配置
```

---

## 2. 生产代码目录详解

### 2.1 BPF 层 (`bpf/`)

```
bpf/
├── bpf_syscall_wrapper/         # BPF 系统调用封装
├── bpf_loader/                  # BPF 程序加载器
├── bpf_reader/                  # BPF 数据读取
└── bpf_progs/                   # BPF 程序源码
```

**职责**: 提供 eBPF 程序的基础设施，用于高性能网络数据包过滤和流量统计。

---

### 2.2 公共代码层 (`common/`)

```
common/
├── ani_rs/                      # Rust ANI 绑定
├── ani_rs_macros/              # Rust 过程宏
├── ani_sys/                    # ANI 系统绑定
└── ani_test/                   # ANI 测试工具 (忽略)
```

**职责**: 支持 Rust 语言实现的 ANI (Ark Native Interface) 接口，用于高性能场景。

**关键文件**:
- `ani_rs/src/ani/serialize.rs` - ANI 序列化
- `ani_rs/src/ani/deserialize.rs` - ANI 反序列化

---

### 2.3 框架层 (`frameworks/`)

```
frameworks/
├── cj/connection/              # Cangjie FFI 接口
├── ets/ani/                    # ArkTS/ETS ANI 接口
│   ├── connection/             # 网络连接模块
│   └── statistics/             # 流量统计模块
├── js/napi/                    # JavaScript N-API
│   ├── connection/             # @ohos.net.connection
│   ├── network/                # @ohos.net.network
│   ├── netpolicy/              # @ohos.net.policy
│   └── netstats/               # @ohos.net.statistics
└── native/                     # Native C++ 客户端
    ├── netconnclient/
    ├── netpolicyclient/
    ├── netstatsclient/
    └── netmanagernative/
```

**职责**: 提供多语言 API 封装，将底层 C++ 服务封装为 JS/TS/ArkTS/C 可调用的接口。

**关键模块**:

| 模块 | 路径 | 说明 |
|------|------|------|
| connection N-API | `js/napi/connection/` | 网络连接 JS API |
| network N-API | `js/napi/network/` | 网络基础 JS API |
| policy N-API | `js/napi/netpolicy/` | 网络策略 JS API |
| statistics N-API | `js/napi/netstats/` | 流量统计 JS API |

---

### 2.4 接口层 (`interfaces/`)

```
interfaces/
├── innerkits/                   # 内部 API (C++ 接口)
│   ├── include/                # 公共头文件
│   ├── netconnclient/          # 网络连接客户端接口
│   ├── netpolicyclient/        # 网络策略客户端接口
│   ├── netstatsclient/         # 网络统计客户端接口
│   └── netmanagernative/       # 原生网络接口
└── kits/c/netconnclient/       # C 语言接口 (NDK)
```

**职责**: 定义对外暴露的 API 接口，包括内部 kits (C++) 和 C API (NDK)。

**关键文件**:

| 接口 | 路径 | 说明 |
|------|------|------|
| INetConnService | `innerkits/netconnclient/include/proxy/i_net_conn_service.h` | 连接服务接口定义 |
| NetConnClient | `innerkits/netconnclient/include/net_conn_client.h` | 连接客户端类 |
| INetPolicyService | `innerkits/netpolicyclient/include/i_net_policy_service.h` | 策略服务接口定义 |
| NetPolicyClient | `innerkits/netpolicyclient/include/net_policy_client.h` | 策略客户端类 |

---

### 2.5 服务层 (`services/`)

```
services/
├── common/                      # 服务公共代码
├── etc/init/                    # 初始化配置文件
├── netconnmanager/             # 网络连接管理服务
├── netmanagernative/           # 本地网络管理服务
├── netpolicymanager/           # 网络策略管理服务
├── netstatsmanager/            # 流量统计管理服务
└── netsyscontroller/           # 网络系统控制器
```

**职责**: 实现核心网络管理功能，作为 System Ability 提供 IPC 服务。

#### 2.5.1 NetConnManager (`services/netconnmanager/`)

```
netconnmanager/
├── include/
│   ├── net_conn_service.h              # 服务主类
│   ├── net_supplier.h                  # 网络供应商
│   ├── network.h                       # 网络对象
│   ├── net_activate.h                  # 网络激活
│   ├── net_monitor.h                   # 网络监测
│   ├── net_http_probe.h                # HTTP 探测
│   └── ...
└── src/
    ├── net_conn_service.cpp            # 服务实现
    ├── net_supplier.cpp
    ├── network.cpp
    └── ...
```

**核心职责**:
- 网络生命周期管理 (注册/注销/激活)
- 网络选择策略 (WiFi/蜂窝/以太网优先级)
- HTTP 代理配置 (全局/应用级别)
- DNS 解析管理
- 网络连通性探测
- PAC 代理自动配置

**关键类**:
- `NetConnService` - 服务主类，继承 SystemAbility
- `NetSupplier` - 网络供应商抽象
- `Network` - 网络实例封装
- `NetActivate` - 网络激活请求

#### 2.5.2 NetPolicyManager (`services/netpolicymanager/`)

```
netpolicymanager/
├── include/
│   ├── net_policy_service.h            # 服务主类
│   ├── net_policy_core.h               # 策略核心
│   ├── net_policy_firewall.h           # 防火墙规则
│   └── ...
└── src/
    ├── net_policy_service.cpp
    └── ...
```

**核心职责**:
- 应用网络策略 (按 UID)
- 后台网络策略
- 流量配额管理
- 防火墙规则管理
- 设备空闲模式策略

**关键类**:
- `NetPolicyService` - 服务主类
- `NetPolicyCore` - 策略核心引擎
- `NetPolicyFirewall` - 防火墙规则管理

#### 2.5.3 NetStatsManager (`services/netstatsmanager/`)

```
netstatsmanager/
├── include/
│   ├── net_stats_service.h             # 服务主类
│   ├── net_stats_cached.h              # 缓存管理
│   ├── net_stats_history.h             # 历史数据
│   └── ...
└── src/
    └── ...
```

**核心职责**:
- 实时流量统计
- 历史流量数据管理
- 流量告警通知
- 数据库存储与查询

**关键类**:
- `NetStatsService` - 服务主类
- `NetStatsCached` - 缓存管理器
- `NetStatsHistory` - 历史数据管理

#### 2.5.4 NetManagerNative (`services/netmanagernative/`)

```
netmanagernative/
├── include/
│   ├── netsys/
│   │   ├── netsys_native_service.h     # Native 服务主类
│   │   ├── dns_manager.h               # DNS 管理
│   │   └── ...
│   └── manager/
│       ├── conn_manager.h              # 连接管理
│       ├── route_manager.h             # 路由管理
│       ├── interface_manager.h         # 接口管理
│       ├── firewall_manager.h          # 防火墙管理
│       ├── vpn_manager.h               # VPN 管理
│       └── ...
├── bpf/                                # BPF 实现
└── fwmarkclient/                       # Fwmark 客户端
```

**核心职责**:
- 底层网络操作 (netlink 通信)
- DNS 解析服务
- iptables/nftables 防火墙规则
- 路由表管理
- VPN 隧道管理
- BPF/eBPF 程序管理
- 网络接口管理

**关键类**:
- `NetsysNativeService` - Native 服务主类
- `DnsManager` - DNS 管理
- `FirewallManager` - 防火墙管理
- `RouteManager` - 路由管理
- `InterfaceManager` - 接口管理
- `VpnManager` - VPN 管理

#### 2.5.5 NetsysController (`services/netsyscontroller/`)

```
netsyscontroller/
├── include/
│   ├── netsys_controller.h             # 控制器主类
│   └── netsys_native_client.h          # Native 客户端
└── src/
    └── ...
```

**核心职责**:
- 作为上层服务与 Native 层的桥梁
- 封装底层网络操作
- 提供单例访问模式

**关键类**:
- `NetsysController` - 控制器单例
- `NetsysNativeClient` - Native 服务客户端

---

### 2.6 工具层 (`utils/`)

```
utils/
├── bundle_utils/                # Bundle 工具
├── common_utils/                # 通用工具
├── data_share/                  # 数据共享
└── napi_utils/                  # N-API 工具
```

**职责**: 提供各模块共享的通用工具函数。

**关键文件**:

| 工具 | 路径 | 说明 |
|------|------|------|
| 权限检查 | `common_utils/src/netmanager_base_permission.cpp` | AccessToken 权限验证 |
| 日志 | `common_utils/include/netmanager_base_log.h` | HiLog 封装 |
| 事件上报 | `common_utils/src/event_report.cpp` | HiSysEvent 封装 |
| NAPI 工具 | `napi_utils/src/napi_utils.cpp` | N-API 辅助函数 |

---

## 3. 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              应用层                                      │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │
│  │   JS/TS    │ │  ArkTS     │ │   C++      │ │     C      │           │
│  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘           │
└────────┼──────────────┼──────────────┼──────────────┼──────────────────┘
         │              │              │              │
         ▼              ▼              ▼              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           框架接口层                                      │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  frameworks/js/napi/  │  frameworks/ets/ani/  │  interfaces/...   │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           内部接口层 (InnerKits)                          │
│  ┌─────────────────┬─────────────────┬─────────────────┐              │
│  │ net_conn_manager│ net_policy_manager│ net_stats_manager│              │
│  │     _if         │       _if       │       _if        │              │
│  └────────┬────────┴────────┬────────┴────────┬────────┘              │
└───────────┼─────────────────┼─────────────────┼─────────────────────────┘
            │                 │                 │
            ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            服务层 (Services)                              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                     │
│  │ NetConnMgr   │ │ NetPolicyMgr │ │ NetStatsMgr  │                     │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘                     │
│         │                │                │                              │
│         └────────────────┴────────────────┘                              │
│                          │                                              │
│                   ┌──────┴──────┐                                       │
│                   │NetsysCtrl   │                                       │
│                   └──────┬──────┘                                       │
│                          │                                              │
│                   ┌──────┴──────┐                                       │
│                   │NetMgrNative │                                       │
│                   └──────┬──────┘                                       │
└──────────────────────────┼──────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            系统内核层                                     │
│                   Kernel / Netlink / BPF / iptables                     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 关键文件索引

### 4.1 服务主类

| 服务 | 头文件 | 实现文件 |
|------|--------|----------|
| NetConnService | `services/netconnmanager/include/net_conn_service.h` | `services/netconnmanager/src/net_conn_service.cpp` |
| NetPolicyService | `services/netpolicymanager/include/net_policy_service.h` | `services/netpolicymanager/src/net_policy_service.cpp` |
| NetStatsService | `services/netstatsmanager/include/net_stats_service.h` | `services/netstatsmanager/src/net_stats_service.cpp` |
| NetsysNativeService | `services/netmanagernative/include/netsys/netsys_native_service.h` | `services/netmanagernative/src/netsys_native_service.cpp` |

### 4.2 接口定义

| 接口 | 文件 |
|------|------|
| INetConnService | `interfaces/innerkits/netconnclient/include/proxy/i_net_conn_service.h` |
| INetPolicyService | `interfaces/innerkits/netpolicyclient/include/i_net_policy_service.h` |
| INetsysService | `interfaces/innerkits/netmanagernative/include/i_netsys_service.h` |

### 4.3 N-API 模块

| 模块 | 入口文件 |
|------|----------|
| connection | `frameworks/js/napi/connection/connection_module/src/connection_module.cpp` |
| network | `frameworks/js/napi/network/network_module/src/network_module.cpp` |
| netpolicy | `frameworks/js/napi/netpolicy/src/netpolicy_module.cpp` |
| netstats | `frameworks/js/napi/netstats/src/statistics_module.cpp` |

---

## 5. 目录命名规范

| 目录模式 | 用途 |
|----------|------|
| `include/` | 头文件目录 |
| `src/` | 源文件目录 |
| `stub/` | IPC Stub 实现 (服务端) |
| `proxy/` | IPC Proxy 实现 (客户端) |
| `async_context/` | N-API 异步上下文 |
| `async_work/` | N-API 异步工作 |
| `observer/` | 观察者模式实现 |
| `options/` | 配置选项类 |

---

*生成时间: 2025-02-06*
