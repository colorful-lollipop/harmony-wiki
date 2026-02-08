# Linux 内核 6.6 - 项目概览

## 目的

本文档介绍 OpenHarmony `kernel_linux_common_6.6` 项目的定位、边界、核心能力、运行环境和关键概念。

## 适用范围

- 读者目标：内核开发者、驱动开发者、安全审计人员、系统集成工程师
- 前置知识：C 语言、操作系统基础、Linux 命令行
- 核心版本：Linux 6.6

---

## 项目定位

### 仓库定义

**kernel_linux_common_6.6** 是 OpenHarmony 操作系统的 Linux 内核基础仓库。

**关键特征**:
- 基于 Linux 内核 6.6 原生代码
- 包含 OpenHarmony 特定适配（EPFS、HMDFS、Hyperhold、Vendor Hooks）
- 使用标准 Linux 设备驱动模型（非 HDF）
- 采用 Kbuild 构建系统（非 GN）

**代码证据**:
- 根目录 `README_OpenHarmony.md` 明确说明："仓用途：linux-6.6原生仓"
- OpenHarmony 组件位于：
  - `fs/epfs/` - Enhanced Proxy File System
  - `fs/hmdfs/` - Harmony Distributed File System
  - `drivers/hyperhold/` - 内存压缩驱动
  - `drivers/hck/` - Vendor hook 框架

### 项目边界

**包含**:
- Linux 内核核心代码（arch/, kernel/, mm/, fs/, net/, drivers/）
- 标准子系统（LSM 安全框架、驱动模型、网络协议栈）
- OpenHarmony 特定文件系统和驱动

**不包含**:
- HDF (Hardware Driver Framework) - OpenHarmony 驱动框架
- HCS (HDF Configuration Source) - 配置解析器
- 用户态工具（glibc、busybox 等）
- OpenHarmony 服务框架（SAMGR、Ability 等）

**原因分析**:
本仓库是 Linux 内核上游代码，OpenHarmony 的用户态服务在独立仓库中。内核层与用户态通过标准 Linux 接口（系统调用、procfs、sysfs、netlink）交互。

---

## 核心能力

### 硬件抽象

**支持架构** (证据: `arch/` 目录):
- arm64 (AArch64) - 主要目标架构
- x86_64 - 桌面/服务器
- riscv - RISC-V 处理器
- arm32 - 32 位 ARM
- powerpc - Power 架构
- 其他：mips, s390, sparc 等（25+ 架构）

**抽象层**:
- 设备树 (Device Tree) - 硬件描述
- ACPI (Advanced Configuration Power Interface) - x86 电源管理
- Platform Bus - 轻量级设备框架

### 进程管理

**能力**:
- 抢占式多任务调度
- 实时调度支持 (RT 调度器)
- 多核 SMP 支持
- CPU 隔离与热插拔

**证据**:
- `kernel/sched/core.c` - 主调度器 `schedule()` 函数
- `kernel/sched/rt.c` - 实时调度器
- `kernel/sched/rtg/` - WALT (Window-based Activity Level Tracker) RTG

### 内存管理

**能力**:
- 虚拟内存管理 (分页、共享内存、COW)
- Buddy 分配器（物理页面管理）
- SLAB/SLUB 分配器（对象缓存）
- 透明大页 (THP)
- 内存压缩 (zswap, Hyperhold)

**证据**:
- `mm/page_alloc.c` - Buddy 分配器 `alloc_pages()` 系列函数
- `mm/slub.c` - SLUB 分配器 `kmem_cache_create()`
- `mm/huge_memory.c` - 透明大页支持
- `mm/zswap.c` - zswap 压缩交换
- `drivers/hyperhold/` - OpenHarmony 扩展内存压缩

### 文件系统

**支持的文件系统** (证据: `fs/` 目录):
- **本地**: ext4, xfs, btrfs, f2fs, jfs, reiserfs
- **网络**: nfs, ceph, cifs (SMB)
- **虚拟**: procfs, sysfs, debugfs, tmpfs
- **特殊**: overlayfs, fuse, devpts
- **OpenHarmony**: EPFS, HMDFS

**VFS 层**:
- `fs/` - VFS (Virtual File System) 核心实现
- `include/linux/fs.h` - 文件系统接口定义

### 网络协议栈

**能力**:
- TCP/IP 协议栈 (IPv4, IPv6)
- Netfilter 防火墙框架
- WiFi/蓝牙支持
- 高性能 XDP (eXpress Data Path)

**证据**:
- `net/ipv4/` - IPv4 协议实现
- `net/ipv6/` - IPv6 协议实现
- `net/netfilter/` - iptables/nftables 框架
- `net/mac80211/` - WiFi MAC 层
- `net/bluetooth/` - 蓝牙协议栈
- `net/xdp/` - XDP 数据路径

### 安全机制

**能力**:
- LSM (Linux Security Modules) 框架
- Capability 基础权限模型
- SELinux / AppArmor / Smack 等策略模块
- IMA/EVM 完整性验证

**证据**:
- `security/security.c` - LSM 核心调度
- `security/selinux/` - SELinux 实现
- `security/apparmor/` - AppArmor 实现
- `include/linux/capability.h` - Capability 定义
- `security/integrity/ima/` - IMA 完整性度量

---

## 运行环境

### 典型部署场景

1. **OpenHarmony 手机/平板**:
   - 架构: arm64
   - 启动: U-Boot → 内核镜像 → init 进程
   - 根文件系统: ext4/f2fs + HMDFS 挂载

2. **OpenHarmony IoT 设备**:
   - 架构: arm32 / riscv
   - 资源受限: 使用轻量级配置
   - 设备树: `.dtb` 描述硬件

3. **开发/调试环境**:
   - 架构: x86_64
   - 模拟器: QEMU
   - 调试工具: ftrace, perf, gdb

### 内核配置空间

**配置类型**:
- `y` - 内置到内核镜像（vmlinux）
- `m` - 编译为模块（.ko 文件，可动态加载）
- `n` - 不编译

**证据**:
- `Kconfig` 文件定义配置选项
- `.config` 文件保存用户选择
- `include/generated/autoconf.h` 生成 C 宏

**典型配置**:
- `CONFIG_ARM64=y` - 目标架构
- `CONFIG_SMP=y` - 多核支持
- `CONFIG_PREEMPT=y` - 抢占式调度
- `CONFIG_MODULES=y` - 模块支持
- `CONFIG_NET=y` - 网络支持

---

## 关键概念

### 系统调用 (System Call)

**定义**: 用户态程序请求内核服务的标准接口。

**证据**:
- `arch/arm64/kernel/syscall.c` - ARM64 系统调用表
- `kernel/sys.c` - 通用系统调用实现

**示例**:
- `open()/close()` - 文件操作
- `read()/write()` - I/O 操作
- `socket()/bind()/connect()` - 网络操作
- `fork()/exec()` - 进程管理

### 内核模块 (Kernel Module)

**定义**: 可动态加载/卸载的内核代码组件（.ko 文件）。

**证据**:
- `kernel/module/main.c` - 模块加载核心
- `insmod` / `rmmod` - 用户态加载/卸载工具

**优势**:
- 不需要重启内核
- 按需加载减少内存占用

### 设备驱动 (Device Driver)

**定义**: 操作硬件设备的内核代码。

**类型**:
- 字符设备 (char) - 流式设备（键盘、鼠标）
- 块设备 (block) - 可随机访问（硬盘、闪存）
- 网络设备 (net) - 数据包接口（网卡）

**证据**:
- `drivers/char/` - 字符设备
- `drivers/block/` - 块设备
- `drivers/net/` - 网络设备

### LSM (Linux Security Modules)

**定义**: 内核安全钩子框架，允许多个安全策略模块共存。

**证据**:
- `include/linux/lsm_hooks.h` - Hook 列表定义
- `include/linux/lsm_hook_defs.h` - Hook 函数声明
- `security/security.c` - Hook 调度器

**工作原理**:
1. 内核在关键位置调用 `security_*()` hook
2. LSM 框架遍历注册的安全模块
3. 每个模块检查权限或强制策略
4. 决策返回给内核

### RCU (Read-Copy-Update)

**定义**: 无锁同步机制，允许多读者并发访问。

**证据**:
- `kernel/rcu/` - RCU 实现
- `include/linux/rcupdate.h` - RCU API

**关键 API**:
- `rcu_read_lock()` / `rcu_read_unlock()` - 读者临界区
- `synchronize_rcu()` - 等待所有读者完成

### 中断 (Interrupt)

**定义**: 硬件事件通知内核的异步机制。

**证据**:
- `kernel/irq/` - 中断管理
- `arch/*/kernel/irq.c` - 架构特定中断处理

**分类**:
- 硬件中断 (IRQ) - 外设事件（网卡、键盘）
- 软中断 (Softirq) - 延迟处理（网络 RX）
- Tasklet - 软中断的变体

### 工作队列 (Workqueue)

**定义**: 延迟执行内核任务的机制（进程上下文）。

**证据**:
- `kernel/workqueue.c` - 工作队列实现
- `kernel/sched/workqueue.c` - 调度器集成

**关键 API**:
- `schedule_work()` - 提交任务
- `flush_work()` - 等待任务完成

---

## OpenHarmony 特定扩展

### EPFS (Enhanced Proxy File System)

**位置**: `fs/epfs/`

**用途**: 增强型代理文件系统，用于跨进程文件代理访问。

**证据**:
- `fs/epfs/main.c:module_init(epfs_init)` - 模块入口

### HMDFS (Huawei Mobile Distributed File System)

**位置**: `fs/hmdfs/` (57 个文件)

**用途**: 分布式文件系统，支持跨设备文件共享。

**组件**:
- `inode.c` - Inode 管理
- `dentry.c` - 目录项缓存
- `client/` / `server/` - 客户端/服务端
- `transport/` - 传输层
- `auth/` - 认证模块

**证据**:
- `fs/hmdfs/Kconfig:config HMDFS_FS` - 配置选项

### Hyperhold

**位置**: `drivers/hyperhold/`

**用途**: 内存压缩驱动，扩展 zswap/zram 能力。

**证据**:
- `drivers/hyperhold/Kconfig:config HYPERHOLD` - 配置选项

### Vendor Hooks

**位置**: `drivers/hck/`, `include/linux/hck/`

**用途**: 允许厂商在关键位置插入自定义代码。

**证据**:
- `include/linux/hck/lite_vendor_hooks.h` - Hook 声明

---

## 相关跳转

- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构详解
- [02_Architecture.md](02_Architecture.md) - 架构与组件交互
- [05_Build_System.md](05_Build_System.md) - 如何构建内核
- [07_Security_Review.md](07_Security_Review.md) - 安全分析

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
