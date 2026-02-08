# iptables 概述

## 库基本信息

| 属性 | 详情 |
|-----|------|
| **名称** | iptables |
| **上游版本** | 1.8.11 (README.OpenSource) / 1.8.7 (bundle.json) |
| **许可证** | GPL V2.0 |
| **上游地址** | https://netfilter.org/ |
| **上游 Git** | https://git.netfilter.org/iptables/ |
| **OH 组件名** | @ohos/iptables |
| **OH 子系统** | thirdparty → communication (netmanager_base) |

**注意**: README.OpenSource 和 bundle.json 中记录的版本号存在差异（1.8.11 vs 1.8.7），建议核实实际基于的上游版本。

## 原始库功能简介

iptables 是 Linux 系统上用户空间的**数据包过滤和 NAT 管理工具**，用于配置 Linux 内核 netfilter 框架的规则集。

### 核心功能

1. **数据包过滤 (Packet Filtering)**
   - 基于 IP 地址、端口、协议等条件过滤网络流量
   - 支持 INPUT、OUTPUT、FORWARD 三条主要链
   - 支持自定义链

2. **网络地址转换 (NAT)**
   - SNAT (源地址转换)
   - DNAT (目的地址转换)
   - MASQUERADE (动态源地址转换)

3. **连接追踪 (Connection Tracking)**
   - 有状态防火墙支持
   - 关联连接自动允许

4. **IPv6 支持**
   - 通过 ip6tables 工具支持 IPv6 协议栈
   - 统一的配置语法

### 组件架构

```
iptables (用户空间)
    │
    ├── iptables / ip6tables (命令行工具)
    ├── iptables-save / ip6tables-save (规则保存)
    ├── iptables-restore / ip6tables-restore (规则恢复)
    │
    ├── libiptc (控制库)
    │   ├── libip4tc (IPv4 控制)
    │   └── libip6tc (IPv6 控制)
    │
    ├── libxtables (xtables 共享库)
    │
    └── extensions (扩展模块)
        ├── libxt_* (通用扩展)
        ├── libipt_* (IPv4 扩展)
        └── libip6t_* (IPv6 扩展)
            │
            ▼
    netfilter (内核空间)
```

## iptables 在 OpenHarmony 中的作用

### 子系统归属

根据 BUILD.gn 中的配置：
- **part_name**: `netmanager_base`
- **subsystem_name**: `communication`

这表明 iptables 在 OpenHarmony 中主要服务于**网络管理基础服务**，属于**通信子系统**的一部分。

### 典型使用场景

#### 1. 系统级网络策略
OpenHarmony 设备可能需要：
- **防火墙规则**: 控制系统级网络访问权限
- **端口过滤**: 限制特定端口的入站/出站流量
- **IP 黑白名单**: 基于 IP 地址的访问控制

#### 2. 网络共享功能
- **热点共享**: 通过 NAT 实现设备热点功能
- **网络桥接**: 多网卡间的流量转发
- **连接共享**: 将设备的网络连接共享给其他设备

#### 3. 应用网络管理
- **应用隔离**: 不同应用的网络访问隔离
- **流量控制**: 基于应用的网络流量管理
- **安全沙箱**: 受限应用的网络访问控制

#### 4. 开发调试支持
- **网络调试**: 开发阶段的网络抓包和流量分析
- **规则测试**: 网络策略的验证和测试

### 在 OH 中的定位

```
OpenHarmony 网络栈
    │
    ├── 应用层 (Apps)
    │       │
    │       ▼
    ├── 网络管理框架 (Net Manager)
    │       │
    │       ├── 策略管理 (Policy)
    │       ├── 连接管理 (Connectivity)
    │       │
    │       ▼
    ├── 网络服务 (Services)
    │       │
    │       ├── netmanager_base (使用 iptables)
    │       │       │
    │       │       ▼
    │       ├── iptables / ip6tables (本库)
    │       │       │
    │       │       ▼
    │       └── netfilter (内核)
    │
    └── 其他网络工具
```

## OH 适配概述

### 适配类型: 轻度适配

iptables 在 OpenHarmony 中的适配属于**轻度适配**，主要工作集中在构建系统层面：

| 适配维度 | 状态 | 说明 |
|---------|------|------|
| 源码修改 | 最小 | 仅 1 个 Patch (musl 兼容性) |
| 构建系统 | 完整 | 完整的 GN 构建配置 |
| 功能扩展 | 无 | 无 OH 特有功能扩展 |
| API 变更 | 无 | 保持上游 API 不变 |

### 主要适配内容

1. **musl libc 兼容性** (0001-musl-build-fix.patch)
   - 修改头文件包含路径
   - 支持 OH 标准系统的 musl 基础

2. **GN 构建系统**
   - 4 个 BUILD.gn 文件覆盖完整构建流程
   - 静态链接配置
   - 代码生成脚本 (genInit.py)

3. **Patch 自动应用**
   - install.sh 脚本在构建时自动应用 Patch

### 与上游差异总结

| 特性 | 上游 | OH 版本 |
|-----|------|---------|
| 构建系统 | autotools | GN + Ninja |
| 库类型 | 支持共享库 | 仅静态库 |
| 扩展加载 | 动态加载 | 静态链接 |
| C 库依赖 | glibc | musl |
| 特有代码 | 无 | 仅构建相关 |

## 文档导航

- [下一章: Patch 详细分析](./02_Patches.md) - 深入分析 musl 兼容性 Patch
- [构建适配说明](./03_Build_Integration.md) - GN 构建配置详解
- [OH 中的使用](./04_Usage_in_OH.md) - 依赖关系和使用场景
- [安全风险分析](./06_Security.md) - CVE 和升级建议
