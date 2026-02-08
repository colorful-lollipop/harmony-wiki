# 对外接口文档

## 概述

OpenHarmony Linux Kernel 5.10 提供多种用户空间接口，包括系统调用、虚拟文件系统、设备文件等。本文档详细列出所有用户空间可见的接口。

## 1. 系统调用接口

### 1.1 系统调用入口

| 架构 | 入口文件 |
|------|----------|
| ARM64 | `arch/arm64/kernel/syscall.c` |
| x86_64 | `arch/x86/entry/syscall_64.c` |
| x86_32 | `arch/x86/entry/syscall_32.c` |
| RISC-V | `arch/riscv/kernel/syscall_table.c` |

### 1.2 系统调用定义

**核心定义文件**：
- `kernel/sys_ni.c` - 未实现系统调用占位符
- `include/linux/syscalls.h` - 系统调用声明头文件
- `arch/x86/entry/syscalls/syscall_64.tbl` - x86_64 系统调用号表
- `arch/arm/tools/syscall.tbl` - ARM 系统调用号表

### 1.3 主要系统调用类别

| 类别 | 系统调用 | 实现文件 |
|------|----------|----------|
| **进程控制** | fork, vfork, clone, clone3 | `kernel/fork.c` |
| | exit, exit_group | `kernel/exit.c` |
| | wait4, waitid, waitpid | `kernel/exit.c` |
| | execve, execveat | `fs/exec.c` |
| **进程管理** | nice | `kernel/sched/core.c:5272` |
| | sched_setscheduler | `kernel/sched/core.c:5905` |
| | sched_setparam, sched_getparam | `kernel/sched/core.c:5920,6001` |
| | sched_setaffinity | `kernel/sched/core.c:6258` |
| | sched_getaffinity | `kernel/sched/core.c:6320` |
| | setpriority, getpriority | `kernel/sys.c:198,268` |
| **文件系统** | open, openat | `fs/open.c` |
| | close | `fs/open.c` |
| | read, write | `fs/read_write.c` |
| | lseek | `fs/read_write.c` |
| | mmap, munmap | `mm/mmap.c` |
| | mprotect | `mm/mprotect.c` |
| | ioctl | `fs/ioctl.c` |
| **网络** | socket, socketpair | `net/socket.c` |
| | bind, listen, accept, connect | `net/socket.c` |
| | sendto, recvfrom, sendmsg, recvmsg | `net/socket.c` |
| | setsockopt, getsockopt | `net/socket.c` |
| **信号** | kill, tkill, tgkill | `kernel/signal.c:3651,3831,3847` |
| | sigaction, rt_sigaction | `kernel/signal.c:4245,4317` |
| | sigprocmask, rt_sigprocmask | `kernel/signal.c:3027,4197` |
| | sigpending, rt_sigpending | `kernel/signal.c:3099,4158` |
| **IPC** | shmget, shmat, shmdt | `ipc/shm.c` |
| | semget, semop, semctl | `ipc/sem.c` |
| | msgget, msgsnd, msgrcv, msgctl | `ipc/msg.c` |
| **时间** | nanosleep | `kernel/time/hrtimer.c:2043` |
| | clock_gettime, clock_settime | `kernel/time/posix-stubs.c:93,60` |
| | gettimeofday, settimeofday | `kernel/time/time.c:140,199` |
| | timer_create, timer_settime, timer_delete | `kernel/time/posix-timers.c:582,947,1013` |
| **模块** | init_module | `kernel/module.c:4189` |
| | delete_module | `kernel/module.c:981` |
| | finit_module | `kernel/module.c:4209` |
| **系统信息** | uname | `kernel/sys.c:1265` |
| | sysinfo | `kernel/sys.c:2697` |
| | getrlimit, setrlimit | `kernel/sys.c:1411,1683` |
| **其他** | reboot | `kernel/reboot.c:311` |
| | kexec_load, kexec_file_load | `kernel/kexec.c:247`, `kernel/kexec_file.c:329` |
| | bpf | `kernel/bpf/syscall.c:4404` |

## 2. Proc 文件系统接口

### 2.1 核心实现

| 文件 | 功能 |
|------|------|
| `fs/proc/root.c` | proc 文件系统根目录初始化 |
| `fs/proc/base.c` | `/proc/[pid]` 目录内容 |
| `fs/proc/generic.c` | proc 通用操作 |
| `fs/proc/inode.c` | proc inode 操作 |

### 2.2 标准 proc 文件

| proc 文件 | 实现文件 | 内容描述 |
|-----------|----------|----------|
| `/proc/cpuinfo` | `fs/proc/cpuinfo.c` | CPU 信息 |
| `/proc/meminfo` | `fs/proc/meminfo.c` | 内存信息 |
| `/proc/stat` | `fs/proc/stat.c` | 系统统计 |
| `/proc/version` | `fs/proc/version.c` | 内核版本 |
| `/proc/uptime` | `fs/proc/uptime.c` | 运行时间 |
| `/proc/loadavg` | `fs/proc/loadavg.c` | 平均负载 |
| `/proc/cmdline` | `fs/proc/cmdline.c` | 启动参数 |
| `/proc/devices` | `fs/proc/devices.c` | 设备列表 |
| `/proc/interrupts` | `fs/proc/interrupts.c` | 中断统计 |
| `/proc/kmsg` | `fs/proc/kmsg.c` | 内核消息 |
| `/proc/softirqs` | `fs/proc/softirqs.c` | 软中断统计 |

### 2.3 进程特定 proc 文件

| 文件 | 路径格式 | 描述 |
|------|----------|------|
| `/proc/[pid]/status` | `/proc/%d/status` | 进程状态 |
| `/proc/[pid]/stat` | `/proc/%d/stat` | 进程统计 |
| `/proc/[pid]/cmdline` | `/proc/%d/cmdline` | 命令行参数 |
| `/proc/[pid]/environ` | `/proc/%d/environ` | 环境变量 |
| `/proc/[pid]/maps` | `/proc/%d/maps` | 内存映射 |
| `/proc/[pid]/smaps` | `/proc/%d/smaps` | 详细内存统计 |
| `/proc/[pid]/fd/` | `/proc/%d/fd/` | 打开的文件描述符 |
| `/proc/[pid]/ns/` | `/proc/%d/ns/` | 命名空间 |
| `/proc/[pid]/exe` | `/proc/%d/exe` | 可执行文件符号链接 |
| `/proc/[pid]/cwd` | `/proc/%d/cwd` | 当前工作目录符号链接 |

### 2.4 Sysctl 接口

| 文件 | 功能 |
|------|------|
| `/proc/sys/kernel/*` | 内核参数 |
| `/proc/sys/vm/*` | 内存管理参数 |
| `/proc/sys/net/*` | 网络参数 |
| `/proc/sys/fs/*` | 文件系统参数 |
| `/proc/sys/debug/*` | 调试参数 |

**实现**: `fs/proc/proc_sysctl.c`, `kernel/sysctl.c`

## 3. Sysfs 接口

### 3.1 核心实现

| 文件 | 功能 |
|------|------|
| `fs/sysfs/file.c` | sysfs 文件操作（show/store） |
| `fs/sysfs/dir.c` | sysfs 目录操作 |
| `fs/sysfs/symlink.c` | sysfs 符号链接 |
| `fs/sysfs/mount.c` | sysfs 挂载点 |

### 3.2 标准 sysfs 目录

| 路径 | 内容 |
|------|------|
| `/sys/devices/` | 设备层次结构 |
| `/sys/class/` | 设备类（按类型组织） |
| `/sys/bus/` | 总线类型 |
| `/sys/module/` | 加载的内核模块 |
| `/sys/kernel/` | 内核对象和参数 |
| `/sys/fs/` | 文件系统信息 |
| `/sys/power/` | 电源管理 |

### 3.3 用户空间访问模式

```c
// 通过 sysfs 属性读取/写入内核数据
// show 方法：内核 -> 用户空间
ssize_t show(struct device *dev, struct device_attribute *attr, char *buf);

// store 方法：用户空间 -> 内核
ssize_t store(struct device *dev, struct device_attribute *attr, 
              const char *buf, size_t count);
```

## 4. 设备文件接口

### 4.1 字符设备

**核心实现**: `fs/char_dev.c`

**关键 API**:
- `register_chrdev_region()` - 注册字符设备号
- `alloc_chrdev_region()` - 动态分配字符设备号
- `register_chrdev()` - 注册字符设备
- `cdev_add()` / `cdev_init()` - cdev 操作

**标准字符设备**:
- `/dev/null` - 空设备
- `/dev/zero` - 零设备
- `/dev/random`, `/dev/urandom` - 随机数
- `/dev/tty*` - 终端设备
- `/dev/console` - 控制台

### 4.2 块设备

**核心实现**: `fs/block_dev.c`, `include/linux/genhd.h`

**关键 API**:
- `register_blkdev()` - 注册块设备
- `blkdev_get_by_path()` - 通过路径获取块设备
- `blkdev_get_by_dev()` - 通过设备号获取块设备

### 4.3 OpenHarmony 特有设备

| 设备 | 路径 | 主设备号 | 描述 |
|------|------|----------|------|
| HiLog | `/dev/hilog` | 245 | OpenHarmony 日志设备 |
| Blackbox | 内核内部 | N/A | 崩溃日志收集 |

**HiLog 实现**: `drivers/staging/hilog/hilog.c`
- 支持读写操作
- 环形缓冲区机制
- 用于内核日志输出

## 5. Netlink 接口

### 5.1 核心实现

| 文件 | 功能 |
|------|------|
| `net/netlink/af_netlink.c` | netlink 协议核心实现 |
| `net/netlink/genetlink.c` | Generic Netlink 实现 |
| `net/netlink/diag.c` | netlink 诊断接口 |
| `include/uapi/linux/netlink.h` | 用户空间头文件 |
| `include/uapi/linux/genetlink.h` | Generic Netlink 头文件 |

### 5.2 Netlink 协议类型

| 协议 | 值 | 用途 |
|------|-----|------|
| `NETLINK_ROUTE` | 0 | 路由/网络配置 |
| `NETLINK_USERSOCK` | 2 | 用户空间套接字 |
| `NETLINK_FIREWALL` | 3 | 防火墙 |
| `NETLINK_SOCK_DIAG` | 4 | 套接字诊断 |
| `NETLINK_NFLOG` | 5 | netfilter 日志 |
| `NETLINK_XFRM` | 6 | IPsec |
| `NETLINK_NETFILTER` | 12 | netfilter |
| `NETLINK_GENERIC` | 16 | Generic Netlink |

## 6. Ioctl 接口

### 6.1 通用 ioctl 处理

**核心实现**: `fs/ioctl.c`

### 6.2 主要 ioctl 接口位置

| 子系统 | 位置 | 描述 |
|--------|------|------|
| **块设备** | `block/ioctl.c` | BLK*, HDIO_* |
| **NVMe** | `drivers/nvme/host/core.c` | NVME_IOCTL_* |
| **DRM/GPU** | `drivers/gpu/drm/` | DRM_IOCTL_* |
| **SCSI** | `drivers/scsi/scsi_ioctl.c` | SCSI_IOCTL_* |
| **Watchdog** | `drivers/watchdog/` | WDIOC_* |
| **RTC** | `drivers/rtc/` | RTC_IOCTL_* |
| **Input** | `drivers/input/` | EVIOC_* |
| **V4L2** | `drivers/media/` | VIDIOC_* |
| **ALSA** | `sound/core/` | SNDRV_CTL_IOCTL_* |

### 6.3 UAPI 头文件

用户空间 ioctl 定义位于 `include/uapi/linux/`:
- `fs.h` - 文件系统 ioctl
- `ioctl.h` - 通用 ioctl
- `socket.h` - 套接字 ioctl
- `input.h` - 输入子系统
- `fb.h` - 帧缓冲
- `v4l2-controls.h`, `v4l2-common.h` - 视频4Linux2
- `sound/*.h` - ALSA 音频
- `mtd/*.h` - MTD 设备

## 7. OpenHarmony 特有接口扩展

### 7.1 ELF 格式扩展

**定义文件**: `include/uapi/linux/elf.h`

```c
#define PT_OHOS_RANDOMDATA  0x6788fc60  /* ohos-specific segment */
#define PT_OHOS_RANDOMDATA_SIZE_LIMIT  1024 * 128
```

**加载器支持**: `fs/binfmt_elf.c`
- 支持 PT_OHOS_RANDOMDATA 段类型
- 用于 OpenHarmony 随机数据传递

### 7.2 Blackbox 系统接口

**实现**: `drivers/staging/blackbox/blackbox_core.c`

**API 函数**:
- `bbox_register_module_ops()` - 注册模块操作
- `bbox_notify_error()` - 通知错误事件

**支持的错误事件类型**:
```c
EVENT_SYSREBOOT      // 系统重启
EVENT_LONGPRESS      // 长按电源键
EVENT_COMBINATIONKEY // 组合键
EVENT_SUBSYSREBOOT   // 子系统重启
EVENT_POWEROFF       // 关机
EVENT_PANIC          // 内核 panic
EVENT_OOPS           // 内核 oops
EVENT_SYS_WATCHDOG   // 看门狗超时
EVENT_HUNGTASK       // 任务挂起
```

### 7.3 HiEvent 事件系统

**实现**: `drivers/staging/hievent/hievent_driver.c`
- 事件上报驱动
- 支持系统事件记录和上报

### 7.4 ZeroHung 冻结检测

**实现**: `drivers/staging/zerohung/`
- 系统冻结检测模块
- 自动检测和报告系统无响应

## 8. 调用链示例

### 8.1 系统调用执行流程

```
用户空间
    ↓
_syscall() / svc #0  (ARM64)
    ↓
el0_sync_handler  (arch/arm64/kernel/entry-common.c)
    ↓
el0_svc
    ↓
do_el0_svc
    ↓
invoke_syscall    (arch/arm64/kernel/syscall.c)
    ↓
sys_call_table[scno]  (系统调用表)
    ↓
SYSCALL_DEFINE*(...)  (具体实现，如 kernel/sys.c)
```

### 8.2 proc 文件读取流程

```
用户空间: read("/proc/cpuinfo")
    ↓
sys_read()
    ↓
vfs_read()
    ↓
proc_reg_read()  (fs/proc/inode.c)
    ↓
proc_cpuinfo_show()  (fs/proc/cpuinfo.c)
    ↓
seq_printf()  // 输出 CPU 信息
```

### 8.3 设备文件操作

```
用户空间: open("/dev/hilog")
    ↓
sys_open()
    ↓
chrdev_open()  (fs/char_dev.c)
    ↓
hilog_open()  (drivers/staging/hilog/hilog.c)
    ↓
检查权限 → 分配资源
```

---

*生成时间: 2026-02-06*
