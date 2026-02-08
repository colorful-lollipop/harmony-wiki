# 目录结构与模块职责

## 目的

本文档详细说明 Linux 内核 6.6 的目录结构，按职责分类每个主要目录，并提供关键文件和符号的引用。

## 适用范围

- 读者目标：内核开发者、驱动开发者、代码审查人员
- 核心版本：Linux 6.6

---

## 顶层目录结构（不含测试）

```
linux-6.6/
├── arch/              # 架构相关代码
├── block/             # 块设备层
├── certs/             # 证书管理
├── crypto/            # 加密 API
├── drivers/           # 设备驱动
├── fs/               # 文件系统
├── include/           # 头文件
├── init/              # 内核初始化
├── ipc/              # 进程间通信
├── kernel/           # 核心内核代码
├── lib/              # 通用库函数
├── mm/               # 内存管理
├── net/              # 网络子系统
├── samples/           # 示例代码
├── scripts/           # 构建脚本
├── security/          # 安全框架
├── sound/             # 音频子系统
├── tools/             # 用户态工具
├── usr/              # initramfs
└── virt/             # 虚拟化支持
```

---

## 按职责分类

### 硬件抽象层

#### arch/

**职责**: CPU 架构的硬件抽象，包括系统调用入口、内存管理、设备树支持。

**主要架构** (25+):
- `arm64/` - 64 位 ARM (AArch64) - OpenHarmony 主目标
- `arm/` - 32 位 ARM
- `x86/` - Intel/AMD x86/x86_64
- `riscv/` - RISC-V
- `powerpc/` - Power 架构
- 其他：mips, s390, sparc, alpha 等

**每个架构的子目录**:
```
arch/<arch>/
├── kernel/           # 架构特定内核代码（入口、信号、陷阱）
├── mm/               # 架构特定内存管理（页表、TLB）
├── boot/             # 启动代码
├── include/           # 架构头文件
├── lib/              # 架构库函数
└── configs/          # 默认配置
```

**关键文件**:
- `arch/arm64/kernel/syscall.c` - ARM64 系统调用表
- `arch/x86/entry/syscall_64.c` - x86_64 系统调用
- `arch/arm64/boot/Makefile` - 内核镜像生成

**证据**:
- 顶层 `arch/` 目录结构
- 各架构 `Makefile` 定义构建规则

---

### 核心子系统

#### kernel/

**职责**: 核心内核服务，不依赖特定子系统。

**子目录及职责**:

| 子目录 | 职责 | 关键文件/符号 |
|--------|--------|--------------|
| `sched/` | 进程调度 | `schedule()`, `fork()` |
| `irq/` | 中断管理 | `request_irq()`, `irq_domain_add()` |
| `time/` | 时间管理 | `do_gettimeofday()`, `hrtimer_start()` |
| `rcu/` | RCU 同步 | `synchronize_rcu()`, `rcu_read_lock()` |
| `trace/` | ftrace/tracing | `tracepoint()`, `trace_printk()` |
| `bpf/` | eBPF 子系统 | `bpf_prog_load()` |
| `module/` | 模块加载 | `load_module()`, `try_module_get()` |
| `cgroup/` | 控制组 | `cgroup_add_task()` |
| `power/` | 电源管理 | `pm_suspend()`, `device_wakeup_enable()` |
| `locking/` | 锁原语 | `mutex_lock()`, `rwsem_down_read()` |
| `signal.c` | 信号处理 | `do_signal()`, `send_signal()` |
| `sys.c` | 系统调用 | `SYSCALL_DEFINE*` 宏 |
| `fork.c` | 进程创建 | `do_fork()`, `copy_process()` |
| `exit.c` | 进程退出 | `do_exit()` |
| `pid.c` | PID 管理 | `alloc_pid()`, `free_pid()` |
| `cred.c` | 凭据管理 | `prepare_creds()`, `commit_creds()` |
| `workqueue.c` | 工作队列 | `schedule_work()`, `flush_work()` |

**OpenHarmony 特定**:
- `kernel/sched/rtg/` - WALT (Window-based Activity Level Tracker) RTG 调度器

**证据**:
- `kernel/sched/core.c` - 主调度器
- `kernel/bpf/syscall.c` - BPF 系统调用

---

#### mm/

**职责**: 物理和虚拟内存管理。

**关键文件**:

| 文件 | 职责 | 关键符号 |
|------|--------|---------|
| `page_alloc.c` | Buddy 页面分配器 | `alloc_pages()`, `__get_free_pages()` |
| `slub.c` | SLUB 对象分配器 | `kmem_cache_create()`, `kmem_cache_alloc()` |
| `vmalloc.c` | vmalloc 分配器 | `vmalloc()`, `vfree()` |
| `mmap.c` | 内存映射 | `do_mmap()`, `mmap_region()` |
| `memory.c` | 页面错误处理 | `handle_mm_fault()` |
| `rmap.c` | 反向映射 | `page_add_anon_rmap()` |
| `vmscan.c` | 页面回收（kswapd） | `kswapd_run()` |
| `swap*.c` | 交换管理 | `swap_readpage()`, `swap_writepage()` |
| `hugetlb.c` | 大页管理 | `hugetlb_alloc_page()` |
| `huge_memory.c` | 透明大页 | `transparent_hugepage_enabled()` |
| `ksm.c` | 页面合并 | `ksm_enter()` |
| `compaction.c` | 内存压缩 | `compact_zone()` |
| `memcontrol.c` | 内存 cgroup | `mem_cgroup_charge()` |
| `zswapd*.c` | OpenHarmony swap 守护进程 | `zswapd_init()` |

**子目录**:
- `damon/` - DAMON 数据访问监控
- `kasan/` - Kernel Address Sanitizer
- `kfence/` - Kernel Electric Fence
- `kmsan/` - Kernel Memory Sanitizer

**证据**:
- `mm/page_alloc.c:3864` - `__alloc_pages_nodemask()` 函数
- `mm/slub.c:3923` - `kmem_cache_alloc()` 函数

---

#### block/

**职责**: 块设备层，管理存储设备的 I/O 调度。

**关键文件**:
- `blk-core.c` - 块设备核心
- `blk-mq.c` - 多队列块层
- `genhd.c` - 磁盘管理
- `elevator.c` - I/O 调度器

**证据**:
- `block/blk-mq.c` - blk-mq 多队列实现

---

### 文件系统

#### fs/

**职责**: VFS (Virtual File System) 层和具体文件系统实现。

**VFS 核心**:
- `open.c` - 文件打开
- `read_write.c` - 读写操作
- `namei.c` - 路径查找
- `dcache.c` - Dentry 缓存
- `inode.c` - Inode 管理
- `file_table.c` - 文件表
- `mount.c` - 挂载管理

**支持的文件系统** (60+):

| 类别 | 文件系统 | 位置 |
|------|---------|------|
| 日志型 | ext4, xfs, jfs, reiserfs | `fs/ext4/`, `fs/xfs/` |
| 闪存优化 | f2fs, jffs2, ubifs | `fs/f2fs/`, `fs/ubifs/` |
| 网络型 | nfs, ceph, cifs, 9p | `fs/nfs/`, `fs/ceph/`, `fs/smb/` |
| 虚拟型 | procfs, sysfs, debugfs, tmpfs | `fs/proc/`, `fs/sysfs/` |
| 聚合型 | overlayfs, unionfs | `fs/overlayfs/` |
| 特殊 | fuse, devpts, hugetlbfs | `fs/fuse/`, `fs/devpts/` |

**OpenHarmony 特定**:
- `fs/epfs/` - Enhanced Proxy File System (10 文件)
  - 入口: `module_init(epfs_init)` 在 `main.c`
- `fs/hmdfs/` - Harmony Distributed File System (57 文件)
  - 组件: inode, dentry, client, server, transport, auth
  - 配置: `CONFIG_HMDFS_FS`
- `fs/sharefs/` - 共享文件系统

**证据**:
- `fs/epfs/main.c` - EPFS 初始化
- `fs/hmdfs/Kconfig` - HMDFS 配置选项

---

### 网络子系统

#### net/

**职责**: 网络协议栈，支持 TCP/IP、蓝牙、无线等。

**核心** (`net/core/`):
- `sock.c` - Socket 层
- `dev.c` - 网络设备接口
- `sk_buff.c` - SKB 管理
- `request_sock.c` - 请求队列管理

**协议**:

| 协议栈 | 位置 | 说明 |
|---------|------|------|
| IPv4 | `net/ipv4/` | TCP/IP 协议栈 |
| IPv6 | `net/ipv6/` | IPv6 支持 |
| Netfilter | `net/netfilter/` | 防火墙框架 |
| 蓝牙 | `net/bluetooth/` | Bluetooth 协议栈 |
| 无线 | `net/mac80211/`, `net/wireless/` | WiFi (802.11) |
| 802.11 | `net/mac802154/` | IEEE 802.15.4 |
| 套接字 | `net/unix/`, `net/netlink/` | Unix/Netlink |
| 高级 | `net/xfrm/`, `net/tls/` | IPsec, TLS |

**关键文件**:
- `net/ipv4/af_inet.c` - IPv4 socket 操作
- `net/ipv4/tcp.c` - TCP 实现
- `net/ipv4/udp.c` - UDP 实现
- `net/netfilter/core.c` - Netfilter 核心

**证据**:
- `net/ipv4/tcp.c:2340` - `tcp_v4_connect()` 函数
- `net/netfilter/core.c` - Hook 框架

---

### 驱动模型

#### drivers/base/

**职责**: 设备驱动核心框架，实现设备-驱动绑定。

**关键文件**:

| 文件 | 职责 | 关键符号 |
|------|--------|---------|
| `core.c` | 设备核心 | `device_register()`, `device_del()` |
| `driver.c` | 驱动核心 | `driver_register()`, `driver_unregister()` |
| `bus.c` | 总线管理 | `bus_register()`, `device_bind_driver()` |
| `dd.c` | 驱动探测 | `driver_probe_device()` |
| `class.c` | 设备类 | `class_register()`, `class_create()` |
| `platform.c` | 平台总线 | `platform_driver_register()` |
| `firmware.c` | 固件加载 | `request_firmware()` |

**关键结构** (证据: `include/linux/device.h`):
- `struct device` - 设备结构
- `struct device_driver` - 驱动结构
- `struct bus_type` - 总线类型
- `struct class` - 设备类

#### drivers/

**驱动类别**:

| 目录 | 说明 | 典型驱动 |
|------|------|---------|
| `char/` | 字符设备 | misc, tty, input |
| `block/` | 块设备 | loop, nbd |
| `net/` | 网络设备 | ethernet, wireless |
| `platform/` | 平台驱动 | GPIO, I2C, SPI |
| `pci/` | PCI 设备 | 各种 PCI 设备 |
| `usb/` | USB 设备 | USB 存储、HID |
| `i2c/` | I2C 设备 | 传感器、EEPROM |
| `spi/` | SPI 设备 | Flash、显示 |
| `gpio/` | GPIO 控制 | GPIO 控制器 |
| `clk/` | 时钟管理 | 时钟驱动 |
| `regulator/` | 电源管理 | 电压调节器 |
| `dma/` | DMA 控制 | DMA 引擎 |
| `media/` | 多媒体 | 摄像头、V4L2 |
| `gpu/` | GPU 驱动 | DRM、GPU |
| `staging/` | 实验性驱动 | 新驱动 |

**OpenHarmony 特定**:
- `drivers/hyperhold/` - 内存压缩驱动
  - 配置: `CONFIG_HYPERHOLD`
- `drivers/hck/` - Vendor hook 框架
  - 文件: `vendor_hooks.c`
- `drivers/accessibility/` - 辅助功能驱动

**证据**:
- `drivers/base/core.c` - 设备注册
- `drivers/hyperhold/Kconfig` - Hyperhold 配置

---

### 安全框架

#### security/

**职责**: LSM (Linux Security Modules) 框架和安全策略实现。

**核心文件**:

| 文件 | 职责 | 关键符号 |
|------|--------|---------|
| `security.c` | LSM 核心 | `security_inode_permission()` |
| `commoncap.c` | Capability 实现 | `cap_capable()` |
| `min_addr.c` | 最小地址限制 | `mmap_min_addr` |

**LSM 实现**:

| LSM | 位置 | 说明 |
|-----|--------|------|
| SELinux | `security/selinux/` | 强制访问控制 (MAC) |
| AppArmor | `security/apparmor/` | 基于配置文件的 MAC |
| Smack | `security/smack/` | 简化 MAC |
| Tomoyo | `security/tomoyo/` | 策略访问控制 |
| Landlock | `security/landlock/` | 非特权沙箱 |
| Yama | `security/yama/` | ptrace 限制 |
| LoadPin | `security/loadpin/` | 模块加载限制 |
| Lockdown | `security/lockdown/` | 内核锁定模式 |
| SafeSetID | `security/safesetid/` | setuid/setgid 限制 |
| BPF LSM | `security/bpf/` | BPF 安全策略 |

**完整性子系统**:
- `security/integrity/ima/` - IMA (Integrity Measurement Architecture)
- `security/integrity/evm/` - EVM (Extended Verification Module)

**证据**:
- `security/security.c` - LSM 调度器
- `include/linux/lsm_hooks.h` - Hook 定义

---

### 加密子系统

#### crypto/

**职责**: 加密算法 API 和实现。

**核心文件**:

| 文件 | 职责 | 关键符号 |
|------|--------|---------|
| `algapi.c` | 算法注册核心 | `crypto_register_alg()` |
| `skcipher.c` | 对称密码 API | `crypto_skcipher_*()` |
| `ahash.c` | 异步哈希 API | `crypto_ahash_*()` |
| `shash.c` | 同步哈希 API | `crypto_shash_*()` |
| `aead.c` | AEAD API | `crypto_aead_*()` |

**支持的算法** (100+):
- 对称: AES, SM4, ChaCha20
- 哈希: SHA-1, SHA-256, SHA-512, SM3
- 公钥: RSA, ECDSA, SM2
- 模式: GCM, CCM, XTS, CTR, CBC

**证据**:
- `crypto/algapi.c:387` - `crypto_register_alg()` 函数

---

### 进程间通信

#### ipc/

**职责**: System V IPC 和 POSIX IPC。

**关键文件**:
- `msg.c` - 消息队列
- `sem.c` - 信号量
- `shm.c` - 共享内存
- `mqueue.c` - POSIX 消息队列

**证据**:
- `ipc/msg.c` - 消息队列实现

---

### 初始化

#### init/

**职责**: 内核启动流程和 init 进程。

**关键文件**:
- `main.c` - 内核入口 `start_kernel()`
- `do_mounts.c` - 根文件系统挂载
- `do_mounts_initrd.c` - initramfs 处理

**证据**:
- `init/main.c:652` - `start_kernel()` 函数

---

### 工具和库

#### lib/

**职责**: 通用内核库函数。

**子目录**:
- `crc32.c` - CRC32 计算
- `bsearch.c` - 二分搜索
- `sort.c` - 排序算法
- `kobject.c` - Kobject 通用对象
- `test_kmod.c` - 模块测试

#### scripts/

**职责**: 构建辅助脚本和工具。

**关键目录**:
- `kconfig/` - Kconfig 配置工具
- `kbuild/` - Kbuild 构建工具
- `genksyms/` - 符号版本生成
- `mod/` - 模块工具
- `recordmcount.pl` - ftrace 计数

**证据**:
- `scripts/kconfig/conf.c` - 配置工具
- `scripts/Kbuild.include` - 构建定义

---

### 用户态工具

#### tools/

**职责**: 用户态工具，用于内核开发和调试。

**关键工具**:
- `perf/` - 性能分析工具
- `bpf/` - eBPF 工具链
- `testing/selftests/` - 内核自测试
- `virt/` - 虚拟化工具
- `tracing/` - 追踪工具

---

### 文档

#### Documentation/

**职责**: 内核官方文档。

**重要子目录**:
- `admin-guide/` - 管理员指南
- `driver-api/` - 驱动开发 API
- `filesystems/` - 文件系统文档
- `kbuild/` - 构建系统文档
- `networking/` - 网络协议栈文档
- `security/` - 安全文档

---

## OpenHarmony 特定组件汇总

| 组件 | 位置 | 说明 | 配置选项 |
|------|------|------|---------|
| EPFS | `fs/epfs/` | Enhanced Proxy File System | CONFIG_EPFS_FS |
| HMDFS | `fs/hmdfs/` | Harmony Distributed File System | CONFIG_HMDFS_FS |
| Hyperhold | `drivers/hyperhold/` | 内存压缩驱动 | CONFIG_HYPERHOLD |
| Vendor Hooks | `drivers/hck/`, `include/linux/hck/` | 厂商 hook 框架 | CONFIG_LITE_VENDOR_HOOKS |

**证据**:
- `README_OpenHarmony.md` - OpenHarmony 说明
- 各组件 `Kconfig` 文件

---

## 头文件结构

#### include/

**职责**: 公共头文件，定义内核 API。

**关键子目录**:

| 目录 | 说明 | 示例头文件 |
|------|------|-----------|
| `linux/` | 核心内核 API | `fs.h`, `sched.h`, `mm.h`, `net.h` |
| `asm/` | 架构特定头 | 链接到 `asm-<arch>/` |
| `uapi/linux/` | 用户态 API | `ioctl.h`, `socket.h`, `capability.h` |

**证据**:
- `include/linux/fs.h` - VFS 接口
- `include/linux/device.h` - 设备模型

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 架构与子系统交互
- [04_Internal_APIs.md](04_Internal_APIs.md) - 内部 API 详情

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
