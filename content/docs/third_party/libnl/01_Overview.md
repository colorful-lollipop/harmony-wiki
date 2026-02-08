# 01 - 原始库简介

## 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | libnl (Netlink Library) |
| **上游版本** | 3.11.0 |
| **许可证** | LGPL V2.1 |
| **上游地址** | https://github.com/thom311/libnl |
| **官方文档** | https://www.infradead.org/~tgr/libnl/ |

## 原始功能

libnl 是一个提供基于 **netlink 协议** 的 Linux 内核接口 API 的库。Netlink 是 Linux 内核与用户空间进程之间通信的标准接口，主要用于网络配置。

### 主要功能模块

| 模块 | 功能 |
|------|------|
| **Route** | 路由、链路、邻居表、地址、规则、TC（流量控制）管理 |
| **Generic Netlink** | 通用 netlink 协议支持 |
| **Netfilter** | 连接跟踪、日志、队列管理 |
| **XFRM** | IPsec 安全关联（SA）、安全策略（SP）管理 |
| **IDIAG** | TCP 连接诊断 |
| **CLI** | 命令行接口工具 |

### 核心 API 类别

- **Socket 管理**: `nl_socket_alloc()`, `nl_connect()`
- **消息操作**: `nlmsg_alloc()`, `nl_send_sync()`
- **地址操作**: `nl_addr_build()`, `nl_addr2str()`
- **缓存管理**: `nl_cache_alloc()`, `nl_cache_refill()`
- **对象操作**: `nl_object_priv()`, `nl_object_clone()`

## 在 OpenHarmony 中的定位

### 使用场景

libnl 在 OpenHarmony 中主要用于 **WLAN 子系统** 和 **网络管理**：

1. **WLAN 驱动接口**: 
   - 配置网络接口（`rtnl_link_*`）
   - 管理 IP 地址（`rtnl_addr_*`）
   - 操作路由表（`rtnl_route_*`）
   - 管理邻居表（`rtnl_neigh_*`）

2. **Wi-Fi 连接管理**:
   - wpa_supplicant 使用 libnl 监控网络接口状态
   - 配置无线网卡参数

3. **流量控制（TC）**:
   - 配置 QoS 策略
   - 管理队列规则（qdisc）

### 系统架构位置

```
┌─────────────────────────────────────────┐
│           应用层 (WLAN 设置等)           │
├─────────────────────────────────────────┤
│           框架层 (Wi-Fi 服务)            │
├─────────────────────────────────────────┤
│     HDI 层 (drivers_peripheral/wlan)     │
├─────────────────────────────────────────┤
│    ┌──────────────┐    ┌──────────┐    │
│    │ wpa_supplicant│    │ wlan_client│  │
│    └──────┬───────┘    └────┬─────┘    │
│           │                 │          │
│           └────────┬────────┘          │
│                    ▼                    │
│              ┌──────────┐              │
│              │  libnl   │              │
│              └────┬─────┘              │
├───────────────────┼─────────────────────┤
│              Netlink Socket             │
├───────────────────┼─────────────────────┤
│              Linux Kernel               │
│    (Network Stack, Wireless Stack)      │
└─────────────────────────────────────────┘
```

## 版本演进

| OH 版本 | libnl 版本 | 说明 |
|---------|-----------|------|
| 早期 | 3.7.0 | 初始集成版本 |
| 当前 | 3.11.0 | 升级版本，支持更多内核特性 |

## 与上游的差异

OpenHarmony 版本的 libnl 与上游主要差异：

1. **编译适配**: 移除 `typeof` 等 GCC 扩展，适配 OH 编译环境
2. **内核版本适配**: 使用宏控制新内核特性的使用
3. **安全加固**: 修复了多个安全漏洞（空指针、缓冲区溢出等）
4. **构建系统集成**: 使用 BUILD.gn 替代 autotools

详细差异请参见 [02_Patches.md](02_Patches.md)。
