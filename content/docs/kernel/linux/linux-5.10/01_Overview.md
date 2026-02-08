# 项目定位与边界

## 项目概述

**OpenHarmony Linux Kernel 5.10** 是 OpenHarmony 操作系统的内核层，提供硬件抽象、资源管理、安全隔离等核心能力。

## 项目定位

### 1.1 在 OpenHarmony 架构中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用层                        │
├─────────────────────────────────────────────────────────────┤
│                    OpenHarmony 框架层                        │
│         (ACE, ARK, Ability Framework, ...)                  │
├─────────────────────────────────────────────────────────────┤
│                    OpenHarmony 服务层                        │
│              (系统服务, 硬件服务框架)                         │
├─────────────────────────────────────────────────────────────┤
│  本仓库: OpenHarmony Linux Kernel 5.10                      │
│  ┌──────────────┬──────────────┬─────────────────────────┐  │
│  │ 进程调度      │ 内存管理      │ 文件系统                 │  │
│  │ 网络协议栈    │ 设备驱动      │ 安全模块                 │  │
│  └──────────────┴──────────────┴─────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                      硬件抽象层 (HAL)                        │
├─────────────────────────────────────────────────────────────┤
│                      硬件平台                                │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 核心职责

| 职责 | 说明 | 代码位置 |
|------|------|----------|
| **进程管理** | 进程创建、调度、销毁 | `kernel/`, `init/` |
| **内存管理** | 物理/虚拟内存分配、页表管理 | `mm/` |
| **文件系统** | VFS、具体文件系统实现 | `fs/` |
| **网络协议栈** | TCP/IP、网络设备、过滤 | `net/` |
| **设备驱动** | 硬件设备抽象和控制 | `drivers/` |
| **安全** | 访问控制、安全模块 | `security/` |
| **中断/时钟** | 硬件中断、定时器管理 | `kernel/irq/`, `kernel/time/` |

### 1.3 与 OpenHarmony 用户空间的关系

**接口边界**:
- 系统调用接口
- proc 文件系统
- sysfs 文件系统
- 设备文件接口
- Netlink 接口

## 项目边界

### 2.1 包含在本仓库

| 组件类型 | 范围 | 示例 |
|----------|------|------|
| **内核核心** | 所有核心子系统 | 调度、内存、文件系统 |
| **硬件支持** | 多架构支持 | ARM64, ARM, x86, RISC-V |
| **设备驱动** | 通用驱动框架和具体驱动 | GPU, 网卡, 存储 |
| **网络协议** | 完整 TCP/IP 协议栈 | IPv4/IPv6, TCP, UDP |
| **文件系统** | VFS 和具体实现 | Ext4, F2FS, procfs |
| **安全模块** | LSM 框架和实现 | SELinux, AppArmor |
| **加密子系统** | 算法和密钥管理 | crypto/, security/keys/ |

### 2.2 不包含在本仓库

| 组件 | 归属 | 说明 |
|------|------|------|
| **用户空间程序** | 基础系统 | init, shell, 工具 |
| **OpenHarmony 框架** | 上层框架 | ACE, ARK, Ability |
| **HAL 服务** | 硬件服务层 | 用户态 HAL 守护进程 |
| **应用层** | 应用生态 | 第三方应用 |
| **测试代码** | 测试仓库 | test/, tests/ 目录 |

### 2.3 OpenHarmony 特有组件

本仓库包含的 OpenHarmony 特有功能：

| 组件 | 功能 | 位置 |
|------|------|------|
| **DFX 框架** | 诊断和追踪 | `include/dfx/` |
| **Blackbox** | 崩溃日志收集 | `drivers/staging/blackbox/` |
| **HiLog** | 日志系统 | `drivers/staging/hilog/` |
| **HiEvent** | 事件上报 | `drivers/staging/hievent/` |
| **ZeroHung** | 冻结检测 | `drivers/staging/zerohung/` |
| **Hungtask** | 任务挂起检测 | `drivers/staging/hungtask/` |
| **HMDFS** | 分布式文件系统 | `fs/hmdfs/` |
| **ShareFS** | 共享文件系统 | `fs/sharefs/` |
| **HyperHold** | 内存扩展 | `drivers/hyperhold/` |
| **HCK** | 厂商钩子框架 | `drivers/hck/` |

## 核心能力

### 3.1 标准 Linux 能力

#### 进程管理
- **调度策略**: CFS (完全公平调度)、实时调度、Deadline 调度
- **调度类**: Stop, Deadline, Real-Time, Fair, Idle
- **OpenHarmony 增强**: WALT (窗口辅助负载跟踪)、RTG (实时组调度)

**关键代码**: `kernel/sched/`

#### 内存管理
- **页式内存管理**: 4KB/64KB 页面, 大页支持
- **内存分配器**: Buddy 系统, SLAB/SLUB/SLOB
- **虚拟内存**: mmap, vmalloc, 页表管理
- **内存回收**: kswapd, OOM 杀手

**OpenHarmony 增强**:
- Purgeable 内存 (可回收内存)
- HyperHold (ZRAM 扩展)
- ZSwapd 优化
- RSS 阈值控制

**关键代码**: `mm/`, `drivers/hyperhold/`

#### 文件系统
- **VFS**: 虚拟文件系统层
- **支持的文件系统**: Ext4, F2FS, XFS, Btrfs, NFS, CIFS
- **虚拟文件系统**: procfs, sysfs, devpts, tmpfs
- **OpenHarmony 特有**: HMDFS, ShareFS

**关键代码**: `fs/`

#### 网络协议栈
- **协议支持**: IPv4, IPv6, TCP, UDP, SCTP, DCCP
- **功能特性**: Netfilter, Traffic Control, Multipath TCP
- **无线支持**: WiFi (mac80211), Bluetooth
- **虚拟化**: VXLAN, GRE, GENEVE

**关键代码**: `net/`

#### 设备驱动框架
- **总线类型**: Platform, PCI, USB, I2C, SPI, SDIO
- **设备类型**: 字符设备、块设备、网络设备
- **设备树**: Device Tree 支持

**关键代码**: `drivers/`

### 3.2 OpenHarmony 特有能力

#### DFX (Design For X) 框架
提供系统级诊断和追踪能力：

| 组件 | 功能 | 位置 |
|------|------|------|
| **HiSysEvent** | 系统事件上报 | `drivers/staging/hisysevent/` |
| **HiLog** | 统一日志输出 | `drivers/staging/hilog/` |
| **Blackbox** | 崩溃信息收集 | `drivers/staging/blackbox/` |
| **ZeroHung** | 系统冻结检测 | `drivers/staging/zerohung/` |
| **Hungtask** | 任务挂起检测 | `drivers/staging/hungtask/` |

#### 内存管理增强
| 功能 | 描述 | 代码 |
|------|------|------|
| **HyperHold** | 内存压缩和交换扩展 | `drivers/hyperhold/` |
| **Purgeable** | 可清除内存管理 | `mm/purgeable.c` |
| **ZSwapd** | ZSwap 守护进程优化 | `mm/zswapd.c` |
| **Memtrace** | 内存使用追踪 | `mm/memtrace_ashmem.c` |

#### 文件系统扩展
| 文件系统 | 描述 | 代码 |
|----------|------|------|
| **HMDFS** | 分布式文件系统，支持跨设备文件共享 | `fs/hmdfs/` |
| **ShareFS** | 共享文件系统 | `fs/sharefs/` |

#### 厂商钩子框架 (HCK)
允许厂商在不修改内核核心代码的情况下添加功能：
- **位置**: `drivers/hck/`
- **机制**: 通过预定义的 hook 点扩展内核行为

## 运行环境

### 4.1 支持的硬件架构

| 架构 | 主要用途 | 代码位置 |
|------|----------|----------|
| **ARM64 (AArch64)** | 移动设备、服务器 | `arch/arm64/` |
| **ARM (32-bit)** | 嵌入式、IoT | `arch/arm/` |
| **x86_64** | 开发主机、模拟器 | `arch/x86/` |
| **RISC-V** | 新兴架构 | `arch/riscv/` |

### 4.2 典型硬件平台

**ARM64 平台支持** (部分):
- HiSilicon (海思)
- Qualcomm (高通)
- MediaTek (联发科)
- Rockchip (瑞芯微)
- Samsung (三星)
- Broadcom (博通)

**设备树支持**:
- ARM64: 34+ 供应商目录
- ARM: 2000+ DTS/DTBI 文件

### 4.3 最小系统要求

| 资源 | 最小需求 | 推荐配置 |
|------|----------|----------|
| **内存** | 16 MB (嵌入式) | 512 MB+ |
| **存储** | 4 MB (内核) | 32 MB+ (含模块) |
| **CPU** | ARMv7 / x86 | ARM64 / x86_64 |

## 关键概念

### 5.1 内核空间 vs 用户空间

```
┌─────────────────────────────────┐
│         用户空间                 │
│  - 应用程序代码                  │
│  - 受限的内存访问                │
│  - 通过系统调用访问内核          │
├─────────────────────────────────┤
│       系统调用接口               │
├─────────────────────────────────┤
│         内核空间                 │
│  - 内核代码                      │
│  - 完全硬件访问                  │
│  - 所有内存可访问                │
└─────────────────────────────────┘
```

### 5.2 模块化设计

内核组件可分为三种类型：

| 类型 | 配置选项 | 说明 |
|------|----------|------|
| **Built-in** | `CONFIG_XXX=y` | 静态编译进内核镜像 |
| **Module** | `CONFIG_XXX=m` | 动态加载的模块 (.ko) |
| **Disabled** | `# CONFIG_XXX is not set` | 不包含该功能 |

### 5.3 配置系统 (Kconfig)

配置层次结构：
```
Kconfig (根配置)
├── init/Kconfig          (基本初始化)
├── kernel/Kconfig        (核心功能)
├── mm/Kconfig            (内存管理)
├── net/Kconfig           (网络)
├── drivers/Kconfig       (驱动)
├── fs/Kconfig            (文件系统)
├── security/Kconfig      (安全)
├── crypto/Kconfig        (加密)
├── lib/Kconfig           (库)
└── vendor/Kconfig        (OpenHarmony 扩展)
```

### 5.4 设备树 (Device Tree)

硬件描述机制：
- **源文件**: `.dts` (设备树源), `.dtsi` (包含文件)
- **二进制**: `.dtb` (设备树二进制)
- **位置**: `arch/*/boot/dts/`
- **作用**: 描述硬件配置，实现内核与硬件解耦

---

*生成时间: 2026-02-06*
