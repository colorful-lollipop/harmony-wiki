# 常见问题与定位路径

## 目的

本文档汇总 Linux 内核 6.6 构建和运行中的常见问题及定位路径。

## 适用范围

- 读者目标：内核开发者、系统集成工程师
- 核心版本：Linux 6.6

---

## 构建问题

### 1. 配置错误

**症状**:
```
make: *** No rule to make target 'defconfig'.  Stop.
```

**原因**: 架构未指定

**定位**:
```bash
# 检查当前架构
echo $ARCH

# 检查 Makefile 中的目标
grep -n "defconfig" Makefile
```

**解决方案**:
```bash
# 指定架构
make ARCH=arm64 defconfig

# 或设置环境变量
export ARCH=arm64
make defconfig
```

**证据**:
- `Makefile:500` - 架构相关目标

---

### 2. 交叉编译失败

**症状**:
```
aarch64-linux-gnu-gcc: command not found
```

**原因**: 交叉编译器未安装或路径错误

**定位**:
```bash
# 检查编译器
which $CROSS_COMPILEgcc

# 检查环境变量
env | grep CROSS_COMPILE
```

**解决方案**:
```bash
# 安装交叉编译器
sudo apt install gcc-aarch64-linux-gnu

# 设置前缀
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

---

### 3. 模块编译失败

**症状**:
```
ERROR: "module_name" undefined!
```

**原因**: 导出符号缺失

**定位**:
```bash
# 查找符号
grep -r "EXPORT_SYMBOL.*symbol_name" .

# 查找引用
grep -r "symbol_name" module_dir/
```

**解决方案**:
```bash
# 添加导出（在符号定义处）
EXPORT_SYMBOL(symbol_name);

# 或修改模块依赖
echo "MODULE_DEPENDENCIES += dependency_module" >> Makefile
```

**证据**:
- `scripts/mod/modpost.c` - 模块后处理

---

### 4. 链接失败

**症状**:
```
undefined reference to `symbol_name'
```

**原因**: 符号未定义或未导出

**定位**:
```bash
# 查看 System.map
grep symbol_name System.map

# 查看模块符号
modinfo module_name | grep vermagic
```

**解决方案**:
```bash
# 重新编译依赖模块
make drivers/<dep_dir>/clean
make drivers/<dep_dir>/

# 或内置到内核
obj-y += dep_dir/
```

---

## 运行时问题

### 1. 内核启动失败

**症状**:
```
Kernel panic - not syncing: VFS: Unable to mount root fs
```

**原因**: 根文件系统未编译或驱动缺失

**定位**:
```bash
# 查看内核日志
dmesg | grep -i "ext4\|root\|panic"

# 检查加载的模块
lsmod | grep ext4
```

**解决方案**:
```bash
# 重新配置内核
make ARCH=arm64 menuconfig
# 启用文件系统
-> File systems -> <*> Ext4 filesystem

# 重新编译
make -j$(nproc)
```

**证据**:
- `init/do_mounts.c` - 根文件系统挂载

---

### 2. 模块加载失败

**症状**:
```
insmod: ERROR: could not insert module
dmesg: module: Unknown symbol
```

**原因**: 符号版本不匹配或依赖缺失

**定位**:
```bash
# 查看模块信息
modinfo module_name

# 查看依赖
modprobe --show-depends module_name

# 查看内核符号
grep symbol_name /proc/kallsyms
```

**解决方案**:
```bash
# 先加载依赖模块
modprobe dependency_module

# 或检查内核版本匹配
uname -r
modinfo module_name | grep vermagic
```

---

### 3. 驱动探测失败

**症状**:
```
dmesg: driver: probe of <device> failed with error -22
```

**原因**: 设备树不匹配或驱动 Bug

**定位**:
```bash
# 查看设备树
dtc -I dtb -O dts -o temp.dts <device>.dtb
cat temp.dts | grep -A 10 device_name

# 查看驱动日志
dmesg | grep -i "probe\|driver"
```

**解决方案**:
```bash
# 修改设备树
# 或更新驱动匹配表
static const struct of_device_id driver_of_match[] = {
    { .compatible = "vendor,device", },
    {}
};
```

**证据**:
- `drivers/base/dd.c` - 驱动探测

---

## 性能问题

### 1. 高 CPU 使用

**症状**:
```
top 显示内核线程占用大量 CPU
```

**定位**:
```bash
# 查看内核线程
top -H

# 查看 CPU 使用
perf top

# 查看调度延迟
perf sched record -e sched:sched_switch -a -- sleep 1
```

**解决方案**:
```bash
# 检查 kswapd（内存回收）
cat /proc/vmstat | grep pgscan

# 检查软中断
cat /proc/softirqs

# 优化调度器
echo performance > /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

**证据**:
- `kernel/sched/core.c` - 调度器

---

### 2. 内存泄漏

**症状**:
```
系统内存持续增长
```

**定位**:
```bash
# 查看内存信息
cat /proc/meminfo
cat /proc/vmallocinfo

# 使用 KASAN
make CONFIG_KASAN=y
# 查看 KASAN 报告
dmesg | grep KASAN
```

**解决方案**:
```bash
# 启用 slab debug
cat /proc/slabinfo

# 或使用 kmemleak
echo scan > /sys/kernel/debug/kmemleak
cat /sys/kernel/debug/kmemleak
```

**证据**:
- `mm/kasan/` - KASAN 实现
- `mm/kmemleak.c` - kmemleak

---

## 调试方法

### 1. Ftrace

**目的**: 函数跟踪

**使用**:
```bash
# 启用跟踪
echo function > /sys/kernel/debug/tracing/current_tracer
echo 1 > /sys/kernel/debug/tracing/tracing_on

# 查看跟踪
cat /sys/kernel/debug/tracing/trace

# 查看可用函数
cat /sys/kernel/debug/tracing/available_filter_functions
```

**证据**:
- `kernel/trace/trace.c` - ftrace 实现

### 2. Perf

**目的**: 性能分析

**使用**:
```bash
# 记录 CPU 事件
perf record -e cycles,instructions -a -- sleep 5

# 分析
perf report

# 查看热点函数
perf top
```

**证据**:
- `kernel/events/core.c` - perf 实现

### 3. Kgdb

**目的**: 内核调试

**配置**:
```bash
# 启用 Kgdb
CONFIG_KGDB=y
CONFIG_KGDB_SERIAL_CONSOLE=y
```

**使用**:
```bash
# 在另一终端
echo g > /proc/sysrq-trigger

# 连接 gdb
gdb vmlinux /proc/kcore
```

**证据**:
- `kernel/debug/debug_core.c` - kgdb 实现

---

## 日志分析

### 内核日志位置

| 日志类型 | 位置 |
|---------|------|
| 运行时日志 | `dmesg` 或 `/var/log/kern.log` |
| Oops/Panic | `/var/log/dmesg` 或串口输出 |
| 系统日志 | `/var/log/syslog` |

### 常见关键字

| 关键字 | 含义 |
|--------|------|
| `BUG:` | 内核 bug |
| `WARNING:` | 警告 |
| `call trace:` | 堆栈跟踪 |
| `general protection fault:` | 内存访问错误 |
| `Unable to handle kernel paging request` | 页面错误 |

---

## OpenHarmony 特定问题

### 1. EPFS 挂载失败

**症状**:
```
mount: unknown filesystem type 'epfs'
```

**定位**:
```bash
# 检查模块
lsmod | grep epfs

# 查看内核配置
grep CONFIG_EPFS_FS .config
```

**解决方案**:
```bash
# 加载模块
insmod epfs.ko

# 或重新编译内核
make CONFIG_EPFS_FS=y
```

**证据**:
- `fs/epfs/main.c` - EPFS 初始化

### 2. HMDFS 连接失败

**症状**:
```
HMDFS: unable to connect to remote device
```

**定位**:
```bash
# 查看网络连接
netstat -an | grep hmdfs_port

# 查看日志
dmesg | grep HMDFS
```

**解决方案**:
```bash
# 检查网络配置
# 检查防火墙
iptables -L -n | grep hmdfs_port

# 检查 TLS 证书
cat /etc/hmdfs/cert.pem
```

**证据**:
- `fs/hmdfs/transport/` - 传输层

---

## 相关跳转

- [05_Build_System.md](05_Build_System.md) - 构建系统
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物
- [02_Architecture.md](02_Architecture.md) - 架构原理

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
