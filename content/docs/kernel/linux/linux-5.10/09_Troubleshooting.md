# 问题定位与调试

## 概述

本文档提供 OpenHarmony Linux Kernel 5.10 的常见问题定位方法和调试技术。

## 1. 常见问题分类

### 1.1 启动问题

| 问题 | 症状 | 定位方法 |
|------|------|----------|
| 内核崩溃 (Panic) | 启动时停止，显示 panic 信息 | 查看串口日志/Blackbox |
| 启动挂起 | 启动过程中停止响应 | hungtask 检测、看门狗 |
| 设备识别失败 | 设备未初始化 | dmesg、设备树检查 |
| 模块加载失败 | insmod/modprobe 失败 | dmesg、modprobe 输出 |

### 1.2 运行时问题

| 问题 | 症状 | 定位方法 |
|------|------|----------|
| 系统卡顿 | 响应延迟或无响应 | ZeroHung、负载分析 |
| OOM (内存不足) | 进程被杀死 | dmesg、/proc/meminfo |
| 死锁 | 进程挂起 | hungtask 检测 |
| 性能下降 | CPU/内存使用率高 | perf、ftrace |

### 1.3 驱动问题

| 问题 | 症状 | 定位方法 |
|------|------|----------|
| 设备无响应 | 设备节点不可访问 | lsmod、dmesg |
| 数据传输错误 | IO 错误 | 驱动日志、DMA 检查 |
| 中断丢失 | 设备状态不更新 | /proc/interrupts |

## 2. 调试工具与方法

### 2.1 内核日志

**HiLog 系统**

| 组件 | 路径 | 用途 |
|------|------|------|
| HiLog 设备 | `/dev/hilog` | 内核日志输出 |
| HiEvent | 内核内部 | 事件上报 |
| Blackbox | `/data/blackbox/` | 崩溃日志收集 |

**日志级别**:
```c
// kernel/printk/printk.c
#define KERN_EMERG    "<0>"  /* 系统不可用 */
#define KERN_ALERT    "<1>"  /* 必须立即处理 */
#define KERN_CRIT     "<2>"  /* 临界条件 */
#define KERN_ERR      "<3>"  /* 错误 */
#define KERN_WARNING  "<4>"  /* 警告 */
#define KERN_NOTICE   "<5>"  /* 正常但重要 */
#define KERN_INFO     "<6>"  /* 信息 */
#define KERN_DEBUG    "<7>"  /* 调试 */
```

**查看日志**:
```bash
# 通过 dmesg
dmesg
dmesg | tail -100
dmesg -w  # 实时查看

# 通过 /proc/kmsg
cat /proc/kmsg

# 通过 HiLog
cat /dev/hilog

# 通过 syslog
journalctl -k
```

### 2.2 proc 文件系统

| 文件 | 用途 |
|------|------|
| `/proc/cpuinfo` | CPU 信息 |
| `/proc/meminfo` | 内存使用情况 |
| `/proc/vmstat` | 虚拟内存统计 |
| `/proc/slabinfo` | Slab 分配器信息 |
| `/proc/buddyinfo` | 内存碎片信息 |
| `/proc/zoneinfo` | 内存区域信息 |
| `/proc/interrupts` | 中断统计 |
| `/proc/softirqs` | 软中断统计 |
| `/proc/loadavg` | 负载平均值 |
| `/proc/uptime` | 运行时间 |
| `/proc/[pid]/status` | 进程状态 |
| `/proc/[pid]/maps` | 进程内存映射 |
| `/proc/[pid]/stack` | 进程栈跟踪 |

### 2.3 Sysctl 调试

**运行时参数调整**:
```bash
# 查看所有参数
sysctl -a

# 查看特定参数
sysctl kernel.printk
sysctl vm.swappiness

# 修改参数
sysctl -w kernel.printk="7 4 1 7"
echo 1 > /proc/sys/kernel/sysrq

# 常用调试参数
# 启用所有 SysRq
sysctl kernel.sysrq=1

# 增加日志缓冲区大小
sysctl kernel.printk_ratelimit_burst=100
```

**调试相关 Sysctl**:
```
kernel.panic              # panic 超时自动重启
kernel.panic_on_oops      # oops 时 panic
kernel.hung_task_timeout  # hungtask 检测超时
vm.panic_on_oom          # OOM 时 panic
kernel.softlockup_panic   # softlockup 时 panic
kernel.hardlockup_panic   # hardlockup 时 panic
```

### 2.4 Ftrace

**启用 Ftrace**:
```bash
# 挂载 debugfs
mount -t debugfs none /sys/kernel/debug

# 查看可用跟踪器
cat /sys/kernel/debug/tracing/available_tracers

# 启用函数跟踪
echo function > /sys/kernel/debug/tracing/current_tracer
echo 1 > /sys/kernel/debug/tracing/tracing_on

# 查看跟踪输出
cat /sys/kernel/debug/tracing/trace
```

**常用 Ftrace 功能**:
```bash
# 函数图跟踪
echo function_graph > /sys/kernel/debug/tracing/current_tracer

# 过滤特定函数
echo schedule > /sys/kernel/debug/tracing/set_ftrace_filter

# 事件跟踪
echo 1 > /sys/kernel/debug/tracing/events/sched/sched_switch/enable

# 跟踪特定 PID
echo $$ > /sys/kernel/debug/tracing/set_ftrace_pid
```

### 2.5 Perf

**安装和使用**:
```bash
# 编译 perf
cd tools/perf
make

# 基本用法
perf top              # 实时性能分析
perf record -a -g     # 记录系统调用图
perf report           # 查看报告
perf stat -a          # 统计系统事件

# 查看热点函数
perf top -p $(pidof process)

# 跟踪特定事件
perf record -e 'sched:*' -a
perf script
```

### 2.6 KASAN

**启用 KASAN**:
```
CONFIG_KASAN=y
CONFIG_KASAN_GENERIC=y
CONFIG_KASAN_INLINE=y
```

**KASAN 输出分析**:
```
==================================================================
BUG: KASAN: use-after-free in faulty_function+0x123/0x456
Read of size 8 at addr ffff888123456789 by task test/1234
CPU: 0 PID: 1234 Comm: test Not tainted 5.10.210
Call Trace:
 faulty_function+0x123/0x456
 caller_function+0xabc/0xdef
```

### 2.7 SysRq

**启用 SysRq**:
```bash
echo 1 > /proc/sys/kernel/sysrq
```

**常用 SysRq 命令**:
```bash
# 通过 /proc/sysrq-trigger
echo m > /proc/sysrq-trigger  # 显示内存信息
echo t > /proc/sysrq-trigger  # 显示任务状态
echo p > /proc/sysrq-trigger  # 显示寄存器
echo w > /proc/sysrq-trigger  # 显示阻塞任务
echo c > /proc/sysrq-trigger  # 触发 panic
echo s > /proc/sysrq-trigger  # 同步文件系统
echo u > /proc/sysrq-trigger  # 重新挂载只读
echo b > /proc/sysrq-trigger  # 立即重启
```

## 3. 崩溃分析

### 3.1 Blackbox 崩溃收集

**Blackbox 系统** (`drivers/staging/blackbox/`)

| 事件类型 | 描述 |
|----------|------|
| `EVENT_PANIC` | 内核 panic |
| `EVENT_OOPS` | 内核 oops |
| `EVENT_SYS_WATCHDOG` | 看门狗超时 |
| `EVENT_HUNGTASK` | 任务挂起 |

**崩溃日志位置**:
```bash
# Blackbox 日志通常在
/data/blackbox/
/var/log/blackbox/
```

### 3.2 Oops/Panic 分析

**典型 Oops 格式**:
```
Unable to handle kernel paging request at virtual address 0000000000000000
Mem abort info:
  ESR = 0x96000005
  EC = 0x25: DABT (current EL), IL = 32 bits
  SET = 0, FnV = 0
  EA = 0, S1PTW = 0
Data abort info:
  ISV = 0, ISS = 0x00000005
  CM = 0, WnR = 0
user pgtable: 4k pages, 39-bit VAs, pgdp=0000000001234000
[0000000000000000] pgd=0000000000000000, p4d=0000000000000000, pud=0000000000000000
Internal error: Oops: 96000005 [#1] PREEMPT SMP
Modules linked in: test_module
CPU: 0 PID: 1234 Comm: test Not tainted 5.10.210
Hardware name: Model
pstate: 60400005 (nZCv daif +PAN -UAO -TCO BTYPE=--)
pc : faulty_function+0x123/0x456
lr : caller_function+0xabc/0xdef
sp : ffff800012345678
x29: ffff800012345678 x28: ffff888123456789
...
Call trace:
 faulty_function+0x123/0x456
 caller_function+0xabc/0xdef
```

**分析要点**:
1. **错误类型**: Unable to handle kernel paging request / NULL pointer dereference
2. **发生位置**: pc (程序计数器) 指向的函数和偏移
3. **调用栈**: Call trace 显示调用链
4. **寄存器状态**: pstate, 通用寄存器

**使用 addr2line**:
```bash
# 找到地址对应的代码位置
aarch64-linux-gnu-addr2line -e vmlinux -a 0xffffff8008123456

# 查看函数反汇编
aarch64-linux-gnu-objdump -d vmlinux | grep -A 20 faulty_function
```

### 3.3 堆栈分析

**获取进程堆栈**:
```bash
# 通过 /proc
cat /proc/[pid]/stack

# 通过 SysRq
echo t > /proc/sysrq-trigger
dmesg | grep -A 20 "[pid]"

# 通过 ftrace
echo 1 > /sys/kernel/debug/tracing/options/stacktrace
echo function > /sys/kernel/debug/tracing/current_tracer
```

## 4. 性能问题定位

### 4.1 CPU 性能

**查看 CPU 使用**:
```bash
# 实时查看
top
htop

# 查看详细统计
cat /proc/stat
cat /proc/loadavg

# perf 分析
perf top
perf record -g -a sleep 10
perf report
```

### 4.2 内存性能

**内存分析**:
```bash
# 查看内存使用
cat /proc/meminfo
cat /proc/slabinfo

# 查看内存碎片
cat /proc/buddyinfo

# 查看 OOM 分数
cat /proc/[pid]/oom_score
cat /proc/[pid]/oom_score_adj
```

### 4.3 IO 性能

**块设备分析**:
```bash
# 查看 IO 统计
cat /proc/diskstats

# iostat（需安装）
iostat -x 1

# blktrace（需安装）
blktrace -d /dev/sda -o -
```

## 5. 网络问题定位

### 5.1 网络统计

```bash
# 查看接口统计
cat /proc/net/dev

# 查看连接跟踪
cat /proc/net/nf_conntrack

# 查看路由表
ip route
ip neigh

# 查看套接字
ss -s
ss -tlnp
```

### 5.2 网络抓包

```bash
# tcpdump
tcpdump -i eth0 -w capture.pcap

# 通过 ftrace
echo 1 > /sys/kernel/debug/tracing/events/skb/
```

## 6. OpenHarmony 特有调试

### 6.1 ZeroHung 冻结检测

**功能**: 检测系统/应用冻结

**启用**:
```bash
# 检查 ZeroHung 状态
cat /proc/sys/kernel/hung_task_timeout_secs
```

### 6.2 Hungtask 检测

**配置**:
```bash
# 启用 hungtask
echo 1 > /proc/sys/kernel/hung_task_panic

# 设置超时（秒）
echo 120 > /proc/sys/kernel/hung_task_timeout_secs
```

### 6.3 HiLog 使用

**读取日志**:
```bash
# 通过设备节点
cat /dev/hilog

# 日志级别过滤
echo 6 > /proc/sys/kernel/printk
```

## 7. 常见问题解决方案

### 7.1 启动失败

**问题**: 内核启动时 panic

**排查步骤**:
1. 检查串口日志最后的 panic 信息
2. 查看调用栈确定故障位置
3. 检查配置选项是否正确
4. 验证设备树兼容性
5. 检查模块依赖关系

**解决方案**:
```bash
# 启用 earlyprintk
earlyprintk=serial,ttyS0,115200

# 启用详细启动信息
debug

# 禁用某些驱动进行测试
module_blacklist=driver_name
```

### 7.2 模块加载失败

**问题**: insmod/modprobe 失败

**排查**:
```bash
# 查看失败原因
dmesg | tail -20

# 检查符号依赖
cat /proc/kallsyms | grep missing_symbol

# 检查模块信息
modinfo module.ko

# 检查版本匹配
modprobe --dump-modconfig module.ko
```

### 7.3 内存泄漏

**排查**:
```bash
# 查看 slab 分配器
cat /proc/slabinfo | sort -k2 -n | tail

# 使用 kmemleak（需启用 CONFIG_DEBUG_KMEMLEAK）
echo scan > /sys/kernel/debug/kmemleak
cat /sys/kernel/debug/kmemleak

# 查看进程内存
cat /proc/[pid]/status | grep -E 'VmRSS|VmSize'
```

### 7.4 死锁检测

**启用 lockdep**:
```
CONFIG_LOCKDEP=y
CONFIG_LOCK_STAT=y
CONFIG_DEBUG_LOCK_ALLOC=y
```

**查看 lockdep**:
```bash
cat /proc/lockdep
cat /proc/lockdep_stats
```

## 8. 调试内核配置

### 8.1 推荐调试配置

```
# 基本调试
CONFIG_DEBUG_KERNEL=y
CONFIG_DEBUG_FS=y
CONFIG_MAGIC_SYSRQ=y

# 内存调试
CONFIG_DEBUG_PAGEALLOC=y
CONFIG_DEBUG_SLAB=y
CONFIG_DEBUG_KMEMLEAK=y

# 锁调试
CONFIG_PROVE_LOCKING=y
CONFIG_LOCK_STAT=y
CONFIG_DEBUG_LOCK_ALLOC=y

# 调度调试
CONFIG_SCHED_DEBUG=y
CONFIG_SCHEDSTATS=y

# 跟踪
CONFIG_FTRACE=y
CONFIG_DYNAMIC_FTRACE=y
CONFIG_FUNCTION_TRACER=y
CONFIG_STACK_TRACER=y

# 错误检测
CONFIG_KASAN=y
CONFIG_UBSAN=y
CONFIG_STACKPROTECTOR=y
```

### 8.2 生产环境配置

```
# 最小调试开销
CONFIG_DEBUG_FS=y
CONFIG_MAGIC_SYSRQ=y
CONFIG_FTRACE=y
CONFIG_KASAN=n
CONFIG_LOCKDEP=n
```

---

*生成时间: 2026-02-06*
