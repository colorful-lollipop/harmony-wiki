# lwIP 简介

## 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | lwIP (lightweight IP) |
| **上游版本** | STABLE-2_2_1_RELEASE |
| **许可证** | BSD 3-Clause License |
| **上游地址** | https://savannah.nongnu.org/projects/lwip/ |
| **原始作者** | Adam Dunkels (Swedish Institute of Computer Science) |

## 原始功能概述

lwIP 是一个小型独立的 **TCP/IP 协议套件实现**，设计目标是：

- **资源占用低**: 仅需数十 KB RAM 和约 40 KB 代码 ROM
- **功能完整**: 支持完整的 TCP、UDP、IP 协议栈
- **可裁剪**: 模块化设计，可按需启用功能
- **可移植**: 支持裸机和多种 RTOS

### 支持协议

| 协议 | 说明 |
|------|------|
| **IPv4/IPv6** | 双栈支持，包含数据包转发 |
| **ICMP/ICMPv6** | 网络维护和调试 |
| **IGMP/MLD** | 组播流量管理 |
| **ND (Neighbor Discovery)** | IPv6 邻居发现和 Stateless 地址自动配置 |
| **DHCP/DHCPv6** | 动态主机配置协议 |
| **AutoIP/APIPA** | 零配置网络 (Zeroconf) |
| **UDP** | 用户数据报协议，含 UDP-Lite 扩展 |
| **TCP** | 传输控制协议，含拥塞控制、RTT 估计、快速恢复 |
| **DNS/mDNS** | 域名解析和组播 DNS |
| **PPPoS/PPPoE** | 串口/以太网上的点对点协议 |
| **6LoWPAN** | 低功耗无线个域网 (IEEE 802.15.4, BLE) |
| **SNMP** | 简单网络管理协议 (v2c/v3) |

### 应用层组件

- HTTP 服务器 (支持 HTTPS via altcp)
- MQTT 客户端
- SNTP 客户端
- TFTP 服务器
- SMTP 客户端
- NetBIOS 名称服务
- iPerf 服务器

## OpenHarmony 中的定位和作用

### 系统定位

在 OpenHarmony 中，lwIP 作为 **基础网络协议栈**，主要服务于：

1. **轻量级设备**: 适用于 mini/small 系统类型
2. **分布式软总线**: 为 DSoftBus 提供底层网络通信能力
3. **内核网络**: 为 LiteOS 内核提供 TCP/IP 支持
4. **物联网场景**: 为 IoT 设备提供网络连接能力

### OH 特有增强

OpenHarmony 对 lwIP 进行了三项重要增强：

| 特性 | 说明 | 应用场景 |
|------|------|----------|
| **网络容器** | 支持网络命名空间隔离 | 多应用网络隔离、容器化部署 |
| **低功耗模式** | 定时器空闲时允许 CPU 睡眠 | 电池供电设备、IoT 传感器 |
| **分布式网络** | 支持跨设备网络代理 | 分布式软总线、多设备协同 |

### 与标准 lwIP 的区别

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony lwIP                          │
├──────────────────────┬──────────────────────────────────────┤
│     标准 lwIP        │         OH 特有增强                  │
│  (上游 2.2.1)        │                                      │
├──────────────────────┼──────────────────────────────────────┤
│ • TCP/UDP/IP         │ • 网络容器隔离 (LOSCFG_NET_CONTAINER)│
│ • IPv4/IPv6          │ • 低功耗管理 (LWIP_LOWPOWER)         │
│ • Socket API         │ • 分布式网络 (LWIP_ENABLE_DISTRIBUTED_NET)│
│ • Raw API            │                                      │
│ • PPP/6LoWPAN        │                                      │
└──────────────────────┴──────────────────────────────────────┘
```

## 目录结构

```
lwip/
├── BUILD.gn              # OH GN 构建配置
├── lwip.gni              # GN 源文件列表定义
├── bundle.json           # OH 组件配置
├── OAT.xml               # OSS 审计配置
├── COPYING               # BSD 许可证
├── README.OpenSource     # 开源信息
│
├── src/                  # 核心源代码
│   ├── core/             # 核心协议实现
│   │   ├── ipv4/         # IPv4 协议
│   │   ├── ipv6/         # IPv6 协议
│   │   ├── distributed_net/  # OH: 分布式网络
│   │   ├── lowpower.c    # OH: 低功耗管理
│   │   └── net_group.c   # OH: 网络容器
│   ├── api/              # API 层 (Socket/Netconn)
│   ├── netif/            # 网络接口层
│   ├── apps/             # 应用层组件
│   └── include/lwip/     # 头文件
│       └── distributed_net/  # OH: 分布式网络头文件
│
├── contrib/              # 贡献代码和移植
│   └── ports/            # 各平台移植
├── doc/                  # 文档
└── test/                 # 测试代码
```

## 文档导航

- [Patch 详细分析](02_Patches.md) - OH 特有修改的详细分析
- [构建适配说明](03_Build_Integration.md) - BUILD.gn 和 GN 配置
- [依赖关系与使用](04_Usage_in_OH.md) - 谁在使用 lwIP 及使用方式
- [API/接口差异](05_API_Differences.md) - OH 特有 API 说明
