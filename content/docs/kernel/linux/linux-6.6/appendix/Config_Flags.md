# 关键配置选项

## 目的

本文档汇总 Linux 内核 6.6 的关键 Kconfig 配置选项，包括架构、子系统、安全等。

## 适用范围

- 读者目标：内核配置工程师、系统集成工程师
- 核心版本：Linux 6.6

---

## 架构配置

### ARM64 配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_ARM64` | bool | y | 启用 64 位 ARM |
| `CONFIG_ARM64_VA_BITS_48` | bool | y | 48 位虚拟地址 |
| `CONFIG_ARM64_64K_PAGES` | bool | n | 64 KB 页面 |
| `CONFIG_SMP` | bool | y | 多核支持 |
| `CONFIG_NR_CPUS` | int | 8 | 最大 CPU 数 |
| `CONFIG_ARM64_SVE` | bool | y | 可伸缩向量扩展 |

**证据**: `arch/arm64/Kconfig`

### x86_64 配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_X86_64` | bool | y | 启用 64 位 x86 |
| `CONFIG_X86_LOCAL_APIC` | bool | y | 本地 APIC |
| `CONFIG_X86_IO_APIC` | bool | y | I/O APIC |
| `CONFIG_SMP` | bool | y | 多核支持 |

**证据**: `arch/x86/Kconfig`

---

## 进程管理配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_SCHED_DEBUG` | bool | n | 调度器调试 |
| `CONFIG_CGROUPS` | bool | y | 控制组支持 |
| `CONFIG_CGROUP_FREEZER` | bool | y | 冻结控制组 |
| `CONFIG_CGROUP_PIDS` | bool | y | PID 限制 |
| `CONFIG_RT_GROUP_SCHED` | bool | y | 实时组调度 |
| `CONFIG_PREEMPT` | bool | y | 抢占式调度 |
| `CONFIG_PREEMPT_VOLUNTARY` | bool | n | 自愿抢占 |
| `CONFIG_PREEMPT_RT` | bool | n | 完全抢占（实时） |

**证据**: `kernel/sched/Kconfig`

---

## 内存管理配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_MMU` | bool | y | MMU 支持 |
| `CONFIG_FLATMEM` | bool | n | 扁平内存模型 |
| `CONFIG_SPARSEMEM` | bool | y | 稀疏内存模型 |
| `CONFIG_SPARSEMEM_VMEMMAP` | bool | y | 虚拟内存映射 |
| `CONFIG_SLUB` | bool | y | SLUB 分配器 |
| `CONFIG_KASAN` | bool | n | 地址消毒器 |
| `CONFIG_KFENCE` | bool | n | 电动栅栏 |
| `CONFIG_ZONE_DEVICE` | bool | y | 设备内存区域 |
| `CONFIG_TRANSPARENT_HUGEPAGE` | bool | y | 透明大页 |
| `CONFIG_COMPACTION` | bool | y | 内存压缩 |
| `CONFIG_KSM` | bool | y | 页面合并 |

**证据**: `mm/Kconfig`

---

## 文件系统配置

### 通用文件系统

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_FILESYSTEMS` | bool | y | 文件系统支持 |
| `CONFIG_FS_POSIX_ACL` | bool | y | POSIX ACL |
| `CONFIG_QUOTA` | bool | y | 磁盘配额 |

### 具体文件系统

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_EXT4_FS` | tristate | m | Ext4 文件系统 |
| `CONFIG_EXT4_FS_POSIX_ACL` | bool | y | Ext4 ACL |
| `CONFIG_XFS_FS` | tristate | m | XFS 文件系统 |
| `CONFIG_XFS_QUOTA` | bool | y | XFS 配额 |
| `CONFIG_BTRFS_FS` | tristate | m | Btrfs 文件系统 |
| `CONFIG_F2FS_FS` | tristate | m | F2FS 文件系统 |
| `CONFIG_F2FS_FS_XATTR` | bool | y | F2FS 扩展属性 |

**证据**: `fs/Kconfig`, `fs/ext4/Kconfig`

### OpenHarmony 文件系统

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_EPFS_FS` | tristate | m | EPFS 文件系统 |
| `CONFIG_HMDFS_FS` | tristate | m | HMDFS 文件系统 |
| `CONFIG_HMDFS_FS_ENCRYPTION` | bool | y | HMDFS TLS 加密 |
| `CONFIG_SHARE_FS` | tristate | m | ShareFS 文件系统 |

**证据**: `fs/epfs/Kconfig`, `fs/hmdfs/Kconfig`

---

## 网络配置

### 通用网络

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_NET` | bool | y | 网络支持 |
| `CONFIG_INET` | bool | y | TCP/IP 协议栈 |
| `CONFIG_INET_UDP_DIAG` | bool | y | UDP 诊断 |
| `CONFIG_INET_TCP_DIAG` | bool | y | TCP 诊断 |
| `CONFIG_PACKET` | bool | y | AF_PACKET |
| `CONFIG_UNIX` | bool | y | Unix 域套接字 |

### 协议配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_IPV6` | bool | y | IPv6 支持 |
| `CONFIG_NETFILTER` | bool | y | Netfilter 框架 |
| `CONFIG_NF_TABLES` | bool | y | nftables |
| `CONFIG_IP_NF_IPTABLES` | bool | y | iptables |
| `CONFIG_BRIDGE` | bool | m | 网桥 |
| `CONFIG_VLAN_8021Q` | bool | m | VLAN |

**证据**: `net/Kconfig`, `net/ipv4/Kconfig`

---

## 安全配置

### 通用安全

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_SECURITY` | bool | y | LSM 框架 |
| `CONFIG_SECURITY_NETWORK` | bool | y | 网络安全钩子 |
| `CONFIG_KEYS` | bool | y | 密钥管理 |
| `CONFIG_HARDENED_USERCOPY` | bool | y | 用户态复制加强 |

### LSM 配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_SECURITY_SELINUX` | bool | n | SELinux |
| `CONFIG_SECURITY_SMACK` | bool | n | Smack |
| `CONFIG_SECURITY_APPARMOR` | bool | n | AppArmor |
| `CONFIG_SECURITY_TOMOYO` | bool | n | Tomoyo |
| `CONFIG_SECURITY_YAMA` | bool | y | Yama |
| `CONFIG_SECURITY_LOADPIN` | bool | n | LoadPin |
| `CONFIG_SECURITY_LOCKDOWN` | bool | n | 内核锁定 |

**证据**: `security/Kconfig`, `security/selinux/Kconfig`

### 完整性配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_INTEGRITY` | bool | n | 完整性子系统 |
| `CONFIG_IMA` | bool | n | IMA 完整性度量 |
| `CONFIG_EVM` | bool | n | EVM 扩展验证 |

**证据**: `security/integrity/Kconfig`

---

## 驱动配置

### 通用驱动

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_GPIOLIB` | bool | y | GPIO 库 |
| `CONFIG_I2C` | bool | y | I2C 总线 |
| `CONFIG_SPI` | bool | y | SPI 总线 |
| `CONFIG_PCI` | bool | y | PCI 总线 |
| `CONFIG_USB` | bool | y | USB 总线 |
| `CONFIG_MMC` | bool | y | MMC/SD 卡 |

### 设备类

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_INPUT` | bool | y | 输入设备 |
| `CONFIG_SOUND` | bool | y | 音频子系统 |
| `CONFIG_DRM` | bool | y | DRM 图形 |
| `CONFIG_HID` | bool | y | HID 设备 |

**证据**: `drivers/Kconfig`

### OpenHarmony 驱动

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_HYPERHOLD` | bool | n | Hyperhold 内存驱动 |
| `CONFIG_LITE_VENDOR_HOOKS` | bool | n | Vendor hooks |

**证据**: `drivers/hyperhold/Kconfig`, `drivers/hck/Kconfig`

---

## 调试配置

### 通用调试

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_DEBUG_KERNEL` | bool | y | 内核调试 |
| `CONFIG_DEBUG_INFO` | bool | n | 调试信息（符号） |
| `CONFIG_DEBUG_INFO_REDUCED` | bool | y | 精简调试信息 |
| `CONFIG_DEBUG_INFO_BTF` | bool | n | BPF 类型信息 |

### 跟踪

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_FTRACE` | bool | y | 函数跟踪 |
| `CONFIG_FUNCTION_TRACER` | bool | y | 函数跟踪器 |
| `CONFIG_FUNCTION_GRAPH_TRACER` | bool | y | 函数图跟踪 |
| `CONFIG_SCHED_TRACER` | bool | y | 调度跟踪 |

### 内存调试

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_SLUB_DEBUG` | bool | n | SLUB 调试 |
| `CONFIG_PAGE_POISONING` | bool | n | 页面投毒 |
| `CONFIG_DEBUG_OBJECTS` | bool | n | 对象调试 |
| `CONFIG_DEBUG_SG` | bool | n | SG 调试 |
| `CONFIG_DEBUG_CREDENTIALS` | bool | n | 凭据调试 |

**证据**: `kernel/Kconfig.debug`, `lib/Kconfig.debug`

---

## 性能配置

### 调度器

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_SCHED_WALT` | bool | n | WALT 调度器 |
| `CONFIG_SCHED_RTG` | bool | n | RTG 调度器 |
| `CONFIG_CPU_FREQ` | bool | y | CPU 频率调节 |
| `CONFIG_CPU_IDLE` | bool | y | CPU 空闲 |

**证据**: `kernel/sched/Kconfig`, `kernel/sched/rtg/Kconfig`

### 电源管理

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_PM` | bool | y | 电源管理 |
| `CONFIG_SUSPEND` | bool | y | 系统休眠 |
| `CONFIG_HIBERNATION` | bool | y | 系统休眠到磁盘 |
| `CONFIG_PM_DEBUG` | bool | n | 电源管理调试 |

**证据**: `kernel/power/Kconfig`

---

## 模块配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_MODULES` | bool | y | 模块支持 |
| `CONFIG_MODULE_UNLOAD` | bool | y | 模块卸载 |
| `CONFIG_MODVERSIONS` | bool | y | 模块符号版本 |
| `CONFIG_MODULE_SIG` | bool | n | 模块签名 |
| `CONFIG_MODULE_SIG_FORCE` | bool | n | 强制模块签名 |
| `CONFIG_MODULE_SIG_SHA512` | bool | n | SHA512 签名 |

**证据**: `kernel/module/Kconfig`

---

## Crypto 配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_CRYPTO` | bool | y | 加密 API |
| `CONFIG_CRYPTO_AES` | bool | y | AES 算法 |
| `CONFIG_CRYPTO_SHA256` | bool | y | SHA-256 哈希 |
| `CONFIG_CRYPTO_SHA512` | bool | y | SHA-512 哈希 |
| `CONFIG_CRYPTO_SM4` | bool | y | SM4 算法 |
| `CONFIG_CRYPTO_SM3` | bool | y | SM3 哈希 |

**证据**: `crypto/Kconfig`

---

## 虚拟化配置

| 选项 | 类型 | 默认值 | 说明 |
|------|------|---------|------|
| `CONFIG_VIRTUALIZATION` | bool | y | 虚拟化支持 |
| `CONFIG_KVM` | tristate | m | KVM 虚拟机 |
| `CONFIG_VHOST_NET` | tristate | m | vhost 网络 |
| `CONFIG_LGUEST` | tristate | n | Lguest |

**证据**: `virt/kvm/Kconfig`, `virt/lib/Kconfig`

---

## OpenHarmony 特定配置

### 完整列表

| 选项 | 位置 | 说明 |
|------|------|------|
| `CONFIG_EPFS_FS` | `fs/epfs/Kconfig` | EPFS 文件系统 |
| `CONFIG_HMDFS_FS` | `fs/hmdfs/Kconfig` | HMDFS 文件系统 |
| `CONFIG_HMDFS_FS_ENCRYPTION` | `fs/hmdfs/Kconfig` | HMDFS TLS 加密 |
| `CONFIG_SHARE_FS` | `fs/sharefs/Kconfig` | ShareFS 文件系统 |
| `CONFIG_HYPERHOLD` | `drivers/hyperhold/Kconfig` | Hyperhold 内存驱动 |
| `CONFIG_LITE_VENDOR_HOOKS` | `drivers/hck/Kconfig` | Vendor hooks |

**证据**: 各组件 `Kconfig` 文件

---

## 相关跳转

- [05_Build_System.md](05_Build_System.md) - Kbuild/Kconfig 详解
- [00_Overview.md](00_Overview.md) - 核心概念
- [07_Security_Review.md](07_Security_Review.md) - 安全配置

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
