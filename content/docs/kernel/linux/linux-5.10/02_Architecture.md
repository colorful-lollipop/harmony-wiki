# 目录结构与模块职责

## 目录结构概览

```
linux-5.10/
├── arch/           # 体系架构相关代码
├── block/          # 块设备层
├── certs/          # 证书管理
├── crypto/         # 加密API
├── Documentation/  # 内核文档
├── drivers/        # 设备驱动
├── fs/             # 文件系统
├── include/        # 头文件
├── init/           # 初始化代码
├── ipc/            # 进程间通信
├── kernel/         # 核心子系统
├── lib/            # 库函数
├── mm/             # 内存管理
├── net/            # 网络协议栈
├── samples/        # 示例代码
├── scripts/        # 构建脚本
├── security/       # 安全模块
├── sound/          # 音频子系统
├── tools/          # 工具
├── usr/            # initramfs
└── virt/           # 虚拟化
```

## 各目录详细职责

### 1. arch/ - 体系架构支持

**职责**: 各 CPU 架构的硬件抽象层

| 子目录 | 架构 |
|--------|------|
| arm64/ | ARM64 (AArch64) |
| arm/ | ARM (32-bit) |
| x86/ | x86/x86_64 |
| riscv/ | RISC-V |
| powerpc/ | PowerPC |
| mips/ | MIPS |

**关键文件**:
- `arch/arm64/kernel/` - ARM64 内核核心代码
- `arch/arm64/boot/dts/` - 设备树源文件
- `arch/arm64/configs/` - 默认配置文件
- `arch/arm64/include/asm/` - 架构特定头文件

### 2. block/ - 块设备层

**职责**: 块设备 I/O 调度与管理

| 关键文件 | 功能 |
|----------|------|
| `blk-core.c` | 块设备核心 |
| `blk-mq.c` | Multi-queue 块层 |
| `elevator.c` | I/O 调度器框架 |
| `bfq-iosched.c` | BFQ 调度器 |
| `mq-deadline.c` | Deadline 调度器 |

### 3. crypto/ - 加密子系统

**职责**: 加密算法和密钥管理

| 子目录 | 功能 |
|--------|------|
| `api.c` | 加密 API |
| `sha*_generic.c` | SHA 算法 |
| `aes*.c` | AES 算法 |
| `rsa.c` | RSA 算法 |

### 4. drivers/ - 设备驱动

**职责**: 各类硬件设备驱动

| 子目录 | 驱动类型 |
|--------|----------|
| `base/` | 驱动框架核心 |
| `char/` | 字符设备 |
| `block/` | 块设备驱动 |
| `net/` | 网络设备 |
| `usb/` | USB 驱动 |
| `pci/` | PCI 驱动 |
| `gpio/` | GPIO 驱动 |
| `i2c/` | I2C 总线 |
| `spi/` | SPI 总线 |
| `tty/` | 串口驱动 |
| `input/` | 输入设备 |
| `gpu/` | 图形驱动 |
| `media/` | 媒体设备 |
| `staging/` | 暂存驱动 |

### 5. fs/ - 文件系统

**职责**: 虚拟文件系统和具体文件系统实现

| 子目录 | 文件系统 |
|--------|----------|
| `proc/` | /proc 文件系统 |
| `sysfs/` | /sys 文件系统 |
| `devpts/` | 伪终端 |
| `ext4/` | Ext4 文件系统 |
| `jbd2/` | Ext4 日志 |
| `f2fs/` | F2FS 闪存文件系统 |
| `fat/` | FAT/NTFS 基础 |
| `ntfs/` | NTFS 支持 |
| `cifs/` | SMB/CIFS |
| `nfs/` | NFS |
| `9p/` | 9P 协议 |
| `debugfs/` | 调试文件系统 |

### 6. include/ - 头文件

**职责**: 内核公共头文件

| 子目录 | 内容 |
|--------|------|
| `linux/` | 核心头文件 |
| `uapi/` | 用户空间 API 头文件 |
| `asm-generic/` | 通用汇编头 |

### 7. init/ - 初始化

**职责**: 内核启动初始化

| 关键文件 | 功能 |
|----------|------|
| `main.c` | 内核入口 start_kernel() |
| `initramfs.c` | initramfs 支持 |

### 8. ipc/ - 进程间通信

**职责**: IPC 机制实现

| 关键文件 | 功能 |
|----------|------|
| `sem.c` | 信号量 |
| `msg.c` | 消息队列 |
| `shm.c` | 共享内存 |
| `mqueue.c` | POSIX 消息队列 |

### 9. kernel/ - 核心子系统

**职责**: 内核核心功能

| 子目录/文件 | 功能 |
|-------------|------|
| `sched/` | 进程调度器 |
| `time/` | 时间管理 |
| `irq/` | 中断管理 |
| `power/` | 电源管理 |
| `fork.c` | 进程创建 |
| `exit.c` | 进程退出 |
| `signal.c` | 信号处理 |
| `sys.c` | 系统调用实现 |
| `ptrace.c` | 进程跟踪 |
| `module.c` | 模块加载 |
| `kexec.c` | 快速重启 |
| `sys_ni.c` | 未实现系统调用 |

### 10. lib/ - 库函数

**职责**: 内核通用库

| 关键文件 | 功能 |
|----------|------|
| `string.c` | 字符串操作 |
| `vsprintf.c` | 格式化输出 |
| `kstrtox.c` | 字符串转数字 |
| `radix-tree.c` | 基数树 |
| `rbtree.c` | 红黑树 |
| `list_sort.c` | 链表排序 |

### 11. mm/ - 内存管理

**职责**: 虚拟内存和物理内存管理

| 关键文件 | 功能 |
|----------|------|
| `memory.c` | 内存映射核心 |
| `page_alloc.c` | 页面分配 |
| `slab.c` | Slab 分配器 |
| `vmalloc.c` | 虚拟内存分配 |
| `mmap.c` | mmap 实现 |
| `mprotect.c` | 内存保护 |
| `ksm.c` | 内存合并 |
| `zsmalloc.c` | 压缩内存分配 |

### 12. net/ - 网络协议栈

**职责**: 网络协议实现

| 子目录 | 协议/功能 |
|--------|-----------|
| `core/` | 网络核心 |
| `ipv4/` | IPv4 协议 |
| `ipv6/` | IPv6 协议 |
| `tcp.c` | TCP 协议 |
| `udp.c` | UDP 协议 |
| `socket.c` | Socket API |
| `netlink/` | Netlink |
| `packet/` | 原始套接字 |
| `unix/` | Unix 域套接字 |
| `netfilter/` | 防火墙框架 |
| `bridge/` | 网桥 |
| `mac80211/` | WiFi |

### 13. security/ - 安全模块

**职责**: 安全框架和 LSM

| 子目录 | 安全模块 |
|--------|----------|
| `selinux/` | SELinux |
| `apparmor/` | AppArmor |
| `smack/` | Smack |
| `tomoyo/` | TOMOYO |
| `yama/` | Yama |
| `integrity/` | IMA/EVM |
| `keys/` | 密钥管理 |

### 14. sound/ - 音频子系统

**职责**: ALSA 音频框架

| 子目录 | 功能 |
|--------|------|
| `core/` | ALSA 核心 |
| `drivers/` | 音频驱动 |
| `soc/` | ASoC (SoC 音频) |

### 15. virt/ - 虚拟化

**职责**: 虚拟化支持

| 子目录 | 功能 |
|--------|------|
| `kvm/` | KVM 虚拟化 |
| `mmio/` | MMIO 支持 |

### 16. scripts/ - 构建脚本

**职责**: 内核构建工具

| 关键脚本 | 功能 |
|----------|------|
| `kconfig/` | 配置系统 |
| `checkpatch.pl` | 补丁检查 |
| `mod/` | 模块工具 |
| `sign-file` | 模块签名 |
| `ohos-check-dir.sh` | OpenHarmony 目录检查 |

## 关键子系统入口

| 子系统 | 主要源文件 | 配置文件 |
|--------|-----------|----------|
| 进程调度 | `kernel/sched/core.c` | `kernel/sched/rtg/Kconfig` |
| 内存管理 | `mm/memory.c`, `mm/page_alloc.c` | `mm/Kconfig` |
| VFS | `fs/*.c` | `fs/Kconfig` |
| 网络核心 | `net/core/*.c` | `net/Kconfig` |
| 块层 | `block/blk-core.c` | `block/Kconfig` |
| 安全框架 | `security/security.c` | `security/Kconfig` |
| 设备驱动框架 | `drivers/base/*.c` | `drivers/Kconfig` |
| 加密 API | `crypto/api.c` | `crypto/Kconfig` |

---

*生成时间: 2026-02-06*
