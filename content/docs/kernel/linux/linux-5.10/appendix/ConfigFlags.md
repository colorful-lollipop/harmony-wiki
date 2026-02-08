# 关键配置选项

## 概述

本文档列出 OpenHarmony Linux Kernel 5.10 的关键配置选项（Kconfig），按功能分类整理。

## 1. 架构相关配置

### 1.1 ARM64 架构

```
CONFIG_ARM64=y                    # 启用 ARM64 支持
CONFIG_64BIT=y                    # 64 位内核
CONFIG_ARCH_ACTIONS=y             # 炬力平台
CONFIG_ARCH_SUNXI=y               # 全志平台
CONFIG_ARCH_HISI=y                # 海思平台
CONFIG_ARCH_MEDIATEK=y            # 联发科平台
CONFIG_ARCH_QCOM=y                # 高通平台
CONFIG_ARCH_ROCKCHIP=y            # 瑞芯微平台
```

### 1.2 通用架构选项

```
CONFIG_SMP=y                      # 对称多处理器支持
CONFIG_NR_CPUS=256                # 最大 CPU 数
CONFIG_PREEMPT=y                  # 抢占式内核
CONFIG_HZ=250                     # 内核时钟频率 (Hz)
CONFIG_SCHED_OMIT_FRAME_POINTER=y # 调度优化
```

## 2. 调度器配置

### 2.1 调度策略

```
CONFIG_CGROUP_SCHED=y             # CGroup 调度支持
CONFIG_FAIR_GROUP_SCHED=y         # CFS 组调度
CONFIG_RT_GROUP_SCHED=y           # 实时组调度
CONFIG_CFS_BANDWIDTH=y            # CFS 带宽控制
CONFIG_SCHED_AUTOGROUP=y          # 自动分组调度
```

### 2.2 OpenHarmony 调度增强

```
CONFIG_SCHED_RTG=y                # RTG (实时组) 调度
CONFIG_SCHED_WALT=y               # WALT 负载跟踪
CONFIG_SCHED_QHOP=y               # QoS 控制
```

## 3. 内存管理配置

### 3.1 基本内存管理

```
CONFIG_MMU=y                      # 内存管理单元支持
CONFIG_SPARSEMEM=y                # 稀疏内存模型
CONFIG_TRANSPARENT_HUGEPAGE=y     # 透明大页
CONFIG_COMPACTION=y               # 内存压缩
CONFIG_MIGRATION=y                # 页面迁移
CONFIG_KSM=y                      # 内核同页合并
```

### 3.2 内存控制组

```
CONFIG_CGROUPS=y                  # Control Groups
CONFIG_MEMCG=y                    # 内存控制组
CONFIG_MEMCG_SWAP=y               # Swap 控制
CONFIG_MEMCG_KMEM=y               # 内核内存控制
```

### 3.3 OpenHarmony 内存增强

```
CONFIG_HYPERHOLD=y                # HyperHold 内存扩展
CONFIG_HYPERHOLD_ZSWAPD=y         # ZSwap 守护进程
CONFIG_ZSMALLOC=y                 # ZSMalloc 压缩内存分配器
CONFIG_PURGEABLE=y                # Purgeable 内存支持
CONFIG_MEMTRACE_ASHMEM=y          # Ashmem 追踪
```

### 3.4 内存安全

```
CONFIG_KASAN=y                    # Kernel Address Sanitizer
CONFIG_KASAN_GENERIC=y            # 通用 KASAN
CONFIG_KASAN_STACK=y              # KASAN 栈检查
CONFIG_STACKPROTECTOR=y           # 栈溢出保护
CONFIG_STACKPROTECTOR_STRONG=y    # 强栈保护
```

## 4. 文件系统配置

### 4.1 虚拟文件系统

```
CONFIG_PROC_FS=y                  # /proc 文件系统
CONFIG_SYSFS=y                    # /sys 文件系统
CONFIG_TMPFS=y                    # tmpfs 文件系统
CONFIG_DEVTMPFS=y                 # devtmpfs
CONFIG_DEBUG_FS=y                 # debugfs
```

### 4.2 磁盘文件系统

```
CONFIG_EXT4_FS=y                  # Ext4 文件系统
CONFIG_EXT4_FS_POSIX_ACL=y        # Ext4 POSIX ACL
CONFIG_F2FS_FS=y                  # F2FS 闪存文件系统
CONFIG_F2FS_FS_SECURITY=y         # F2FS 安全支持
CONFIG_XFS_FS=y                   # XFS 文件系统
CONFIG_BTRFS_FS=y                 # Btrfs 文件系统
```

### 4.3 OpenHarmony 特有文件系统

```
CONFIG_HMDFS_FS=y                 # HMD 分布式文件系统
CONFIG_SHAREFS_FS=y               # ShareFS
CONFIG_F2FS_FS_COMPRESSION=y      # F2FS 压缩
```

## 5. 网络配置

### 5.1 基本网络

```
CONFIG_NET=y                      # 网络支持
CONFIG_INET=y                     # TCP/IP
CONFIG_IPV6=y                     # IPv6
CONFIG_NETFILTER=y                # Netfilter 框架
CONFIG_UNIX=y                     # Unix 域套接字
CONFIG_PACKET=y                   # 包套接字
```

### 5.2 Netfilter

```
CONFIG_NF_CONNTRACK=y             # 连接跟踪
CONFIG_NF_CONNTRACK_IPV4=y        # IPv4 连接跟踪
CONFIG_NF_CONNTRACK_IPV6=y        # IPv6 连接跟踪
CONFIG_IP_NF_IPTABLES=y           # IPTables
CONFIG_IP_NF_FILTER=y             # IPTables 过滤
CONFIG_IP_NF_NAT=y                # IPTables NAT
CONFIG_NETFILTER_XT_TARGET_REDIRECT=y
```

### 5.3 高级网络功能

```
CONFIG_BRIDGE=y                   # 网桥
CONFIG_VLAN_8021Q=y               # VLAN
CONFIG_VXLAN=y                    # VXLAN
CONFIG_NET_SCHED=y                # 流量控制
CONFIG_NET_CLS_CGROUP=y           # CGroup 分类
CONFIG_CGROUP_NET_PRIO=y          # CGroup 网络优先级
```

### 5.4 无线网络

```
CONFIG_CFG80211=y                 # 无线配置
CONFIG_MAC80211=y                 # IEEE 802.11
CONFIG_MAC80211_LEDS=y            # 无线 LED
CONFIG_RFKILL=y                   # 射频开关
```

## 6. 驱动配置

### 6.1 设备驱动框架

```
CONFIG_DEVICE_TREE=y              # 设备树支持
CONFIG_OF=y                       # Open Firmware
CONFIG_OF_OVERLAY=y               # 设备树覆盖层
CONFIG_GENERIC_IRQ_CHIP=y         # 通用中断芯片
```

### 6.2 总线支持

```
CONFIG_PCI=y                      # PCI 总线
CONFIG_PCI_MSI=y                  # PCI MSI
CONFIG_PCIEPORTBUS=y              # PCIe
CONFIG_USB=y                      # USB 支持
CONFIG_USB_SUPPORT=y
CONFIG_USB_XHCI_HCD=y             # xHCI 控制器
CONFIG_USB_EHCI_HCD=y             # EHCI 控制器
CONFIG_USB_OHCI_HCD=y             # OHCI 控制器
CONFIG_USB_STORAGE=y              # USB 存储
```

### 6.3 块设备

```
CONFIG_BLK_MQ_PCI=y               # PCI 多队列块层
CONFIG_BLK_MQ_virtio=y            # VirtIO 多队列
CONFIG_NVME_CORE=y                # NVMe 核心
CONFIG_BLK_DEV_NVME=y             # NVMe 块设备
CONFIG_BLK_DEV_LOOP=y             # Loop 设备
CONFIG_BLK_DEV_RAM=y              # RAM 磁盘
```

### 6.4 OpenHarmony 特有驱动

```
CONFIG_BLACKBOX=y                 # Blackbox 崩溃收集
CONFIG_HILOG=y                    # HiLog 日志系统
CONFIG_HIEVENT=y                  # HiEvent 事件上报
CONFIG_ZEROHUNG=y                 # ZeroHung 冻结检测
CONFIG_HUNGTASK=y                 # Hungtask 检测
CONFIG_HCK=y                      # 厂商钩子框架
```

## 7. 安全配置

### 7.1 安全模块

```
CONFIG_SECURITY=y                 # 启用安全模块
CONFIG_SECURITY_SELINUX=y         # SELinux
CONFIG_SECURITY_APPARMOR=y        # AppArmor
CONFIG_SECURITY_SMACK=y           # Smack
CONFIG_SECURITY_TOMOYO=y          # TOMOYO
CONFIG_SECURITY_YAMA=y            # Yama
CONFIG_SECURITY_LOADPIN=y         # LoadPin
CONFIG_SECURITY_LOCKDOWN_LSM=y    # 内核锁定
```

### 7.2 网络安全

```
CONFIG_SECURITY_NETWORK=y         # 网络安全 hooks
CONFIG_SECURITY_NETWORK_XFRM=y    # IPSec 安全
CONFIG_SECURITY_PATH=y            # 路径安全 hooks
CONFIG_INET_SECMARK=y             # 网络 Secmark
```

### 7.3 完整性

```
CONFIG_INTEGRITY=y                # 完整性子系统
CONFIG_INTEGRITY_SIGNATURE=y      # 完整性签名
CONFIG_INTEGRITY_ASYMMETRIC_KEYS=y
CONFIG_IMA=y                      # IMA 度量
CONFIG_IMA_APPRAISE=y             # IMA 评估
CONFIG_EVM=y                      # EVM 扩展验证
```

### 7.4 安全加固

```
CONFIG_FORTIFY_SOURCE=y           # 源码强化
CONFIG_GCC_PLUGINS=y              # GCC 插件
CONFIG_GCC_PLUGIN_STACKLEAK=y     # 栈泄漏防护
CONFIG_GCC_PLUGIN_STRUCTLEAK=y    # 结构体泄漏防护
CONFIG_HARDENED_USERCOPY=y        # 用户拷贝加固
CONFIG_INIT_STACK_ALL_ZERO=y      # 栈初始化清零
CONFIG_INIT_ON_ALLOC_DEFAULT_ON=y # 分配时清零
CONFIG_INIT_ON_FREE_DEFAULT_ON=y  # 释放时清零
```

## 8. 调试与跟踪配置

### 8.1 基本调试

```
CONFIG_DEBUG_KERNEL=y             # 内核调试
CONFIG_DEBUG_FS=y                 # debugfs
CONFIG_MAGIC_SYSRQ=y              # SysRq 魔术键
CONFIG_DEBUG_MEMORY_INIT=y        # 内存初始化调试
```

### 8.2 跟踪

```
CONFIG_FTRACE=y                   # Ftrace
CONFIG_FUNCTION_TRACER=y          # 函数跟踪
CONFIG_DYNAMIC_FTRACE=y           # 动态 ftrace
CONFIG_STACK_TRACER=y             # 栈跟踪
CONFIG_SCHED_TRACER=y             # 调度跟踪
CONFIG_BLK_DEV_IO_TRACE=y         # 块设备 IO 跟踪
```

### 8.3 锁调试

```
CONFIG_PROVE_LOCKING=y            # 锁验证
CONFIG_LOCK_STAT=y                # 锁统计
CONFIG_DEBUG_LOCK_ALLOC=y         # 锁分配调试
CONFIG_DEBUG_ATOMIC_SLEEP=y       # 原子睡眠调试
```

## 9. 虚拟化配置

```
CONFIG_VIRTUALIZATION=y           # 虚拟化支持
CONFIG_KVM=y                      # KVM 虚拟化
CONFIG_KVM_ARM_HOST=y             # ARM KVM
CONFIG_KVM_ARM_PMU=y              # ARM PMU 虚拟化
CONFIG_VHOST_NET=y                # vhost-net
CONFIG_VHOST_VSOCK=y              # vhost-vsock
```

## 10. 电源管理配置

```
CONFIG_PM=y                       # 电源管理
CONFIG_PM_SLEEP=y                 # 睡眠支持
CONFIG_PM_RUNTIME=y               # 运行时电源管理
CONFIG_CPU_FREQ=y                 # CPU 频率调节
CONFIG_CPU_FREQ_GOV_SCHEDUTIL=y   # schedutil 调节器
CONFIG_CPU_IDLE=y                 # CPU 空闲
CONFIG_SUSPEND=y                  # 挂起到内存
CONFIG_HIBERNATION=y              # 休眠到磁盘
```

## 11. 推荐配置组合

### 11.1 移动设备配置

```
# 架构
CONFIG_ARM64=y
CONFIG_PREEMPT=y
CONFIG_HZ=300

# 调度
CONFIG_CGROUP_SCHED=y
CONFIG_FAIR_GROUP_SCHED=y
CONFIG_SCHED_RTG=y
CONFIG_SCHED_WALT=y

# 内存
CONFIG_TRANSPARENT_HUGEPAGE=y
CONFIG_KSM=y
CONFIG_HYPERHOLD=y
CONFIG_ZSMALLOC=y

# 文件系统
CONFIG_F2FS_FS=y
CONFIG_EXT4_FS=y

# 网络
CONFIG_INET=y
CONFIG_IPV6=y
CONFIG_CFG80211=y
CONFIG_MAC80211=y

# 安全
CONFIG_SECURITY=y
CONFIG_SECURITY_SELINUX=y
CONFIG_SECURITY_APPARMOR=y
```

### 11.2 服务器配置

```
# 架构
CONFIG_X86_64=y
CONFIG_PREEMPT_NONE=y

# 内存
CONFIG_TRANSPARENT_HUGEPAGE=y
CONFIG_NUMA=y

# 网络
CONFIG_INET=y
CONFIG_IPV6=y
CONFIG_NETFILTER=y

# 虚拟化
CONFIG_KVM=y

# 安全
CONFIG_SECURITY=y
CONFIG_SECURITY_SELINUX=y
CONFIG_IMA=y
```

### 11.3 调试配置

```
# 调试
CONFIG_DEBUG_KERNEL=y
CONFIG_DEBUG_FS=y
CONFIG_MAGIC_SYSRQ=y

# 内存安全
CONFIG_KASAN=y
CONFIG_STACKPROTECTOR=y
CONFIG_HARDENED_USERCOPY=y

# 跟踪
CONFIG_FTRACE=y
CONFIG_FUNCTION_TRACER=y

# 锁调试
CONFIG_PROVE_LOCKING=y
```

---

*生成时间: 2026-02-06*

**注意**: 实际配置应根据具体硬件平台和应用场景调整。
