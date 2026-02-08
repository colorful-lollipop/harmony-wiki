# 常见问题与调试

## 1. 构建问题

### 1.1 GN 构建失败

#### 问题表现

```
ERROR: Can't find ohos_kernel_build.gni
```

#### 解决方案

**检查步骤**:
1. 确认 OpenHarmony 构建环境已正确设置
2. 检查 `device_name` 参数是否正确传递
3. 验证 `ohos_kernel_build.gni` 模板路径

**命令示例**:

```bash
# 设置环境变量
export OHOS_ROOT=/path/to/openharmony
export DEVICE_NAME=rk3568

# 验证构建
./build.sh --product-name $DEVICE_NAME --build-target ko_build --ccache
```

### 1.2 Makefile 编译错误

#### 问题表现

```
make: *** No rule to make target 'CONFIG_XXX'
```

#### 解决方案

1. 确认对应 Kconfig 已启用
2. 检查 `.config` 文件包含配置
3. 运行 `make olddefconfig` 更新配置

```bash
# 启用配置
make menuconfig
# 导航到: Device Drivers → Character devices → TZDriver

# 保存配置
make savedefconfig

# 编译模块
make CONFIG_TZDRIVER=y modules
```

---

## 2. 运行时问题

### 2.1 模块加载失败

#### 问题表现

```
insmod: ERROR: could not insert module xxx.ko: Unknown symbol in module
```

#### 常见原因

| 原因 | 解决方案 |
|------|----------|
| 符号未导出 | 检查 `EXPORT_SYMBOL` |
| 依赖模块未加载 | 先加载依赖模块 |
| 版本不匹配 | 重新编译匹配内核版本 |
| 权限不足 | 使用 `sudo` 或 root 用户 |

#### 调试步骤

```bash
# 检查模块信息
modinfo xxx.ko

# 检查依赖
lsmod | grep module_name

# 检查 dmesg
dmesg | tail -50

# 检查系统日志
journalctl -k | grep module_name
```

### 2.2 SMC 调用失败

#### 问题表现

```
[  123.456] TZDRIVER: SMC failed with error -1
```

#### 调试方法

1. **启用详细日志**:

```bash
# 启用 tzdebug
mount -t debugfs none /sys/kernel/debug
echo "tm" > /sys/kernel/debug/tzdriver/debug_mask
cat /sys/kernel/debug/tzdriver/log
```

2. **检查 TEE 版本**:

```bash
cat /sys/kernel/debug/tzdriver/version
```

3. **常见原因**:

| 错误码 | 原因 | 解决方案 |
|--------|------|----------|
| -1 | 参数错误 | 检查 IOCTL 参数 |
| -2 | 权限不足 | 检查 SELinux 上下文 |
| -3 | TEE 未就绪 | 确认 TEE 驱动加载 |
| -4 | 超时 | 检查设备响应 |

---

## 3. 调试技术

### 3.1 debugfs 调试

#### TZDriver 调试节点

| 节点 | 路径 | 功能 |
|------|------|------|
| 版本信息 | `/sys/kernel/debug/tzdriver/version` | 驱动版本 |
| 日志 | `/sys/kernel/debug/tzdriver/log` | 运行日志 |
| 调试掩码 | `/sys/kernel/debug/tzdriver/debug_mask` | 调试开关 |

#### Memory Security 调试

| 节点 | 路径 | 功能 |
|------|------|------|
| JIT 区域 | `/sys/kernel/debug/jit_memory/` | JIT 区域列表 |
| 隐藏地址 | `/proc/self/maps` | 验证地址隐藏 |

### 3.2 内核日志

#### 查看内核日志

```bash
# 实时查看
dmesg -w | grep tzdriver

# 查看最近日志
dmesg | tail -100

# 保存日志
dmesg > /tmp/kernel.log
```

#### 日志级别

| 级别 | 值 | 说明 |
|------|-----|------|
| TLOGE | 0 | 错误 |
| TLOGW | 1 | 警告 |
| TLOGI | 2 | 信息 |
| TLOGD | 3 | 调试 |
| TLOGV | 4 | 详细 |

### 3.3 ftrace 跟踪

#### SMC 调用跟踪

```bash
# 挂载 debugfs
mount -t debugfs none /sys/kernel/debug

# 启用函数跟踪
echo function > /sys/kernel/debug/tracing/current_tracer

# 过滤 tzdriver
echo 'tzdriver_*' > /sys/kernel/debug/tracing/set_ftrace_filter

# 开始跟踪
echo 1 > /sys/kernel/debug/tracing/tracing_on

# 查看结果
cat /sys/kernel/debug/tracing/trace
```

---

## 4. 性能问题

### 4.1 SMC 延迟

#### 诊断方法

```bash
# 使用 perf 分析
perf record -g -a --call-graph dwarf -e 'sched:sched_switch' sleep 10
perf report
```

#### 优化建议

| 场景 | 建议 |
|------|------|
| 高频 SMC 调用 | 考虑批量处理 |
| 多核竞争 | 调整 `CONFIG_CPU_AFF_NR` |
| 内存分配 | 预分配 mailbox 内存 |

### 4.2 内存占用

#### 检查内存

```bash
# 查看模块内存使用
lsmod | grep module_name

# 查看 slab 缓存
slabtop | grep tzdriver

# 查看进程内存
cat /proc/<pid>/status | grep VmRSS
```

---

## 5. 安全问题

### 5.1 SELinux 拒绝

#### 问题表现

```
avc: denied { operation } for pid=xxx comm="xxx" ... scontext=xxx tcontext=xxx
```

#### 解决方案

1. **临时解决** (开发环境):

```bash
# 设置 SELinux 为宽容模式
setenforce 0

# 或针对特定域
semodule -i mymodule.pp
```

2. **永久解决**:

```bash
# 生成策略模块
ausearch -m avc -c xxx | audit2allow -M xxx

# 安装策略
semodule -i xxx.pp
```

### 5.2 权限问题

#### 检查权限

```bash
# 检查设备节点权限
ls -la /dev/tc_ns_client

# 检查用户组
groups <username>

# 添加用户到组
sudo usermod -aG <group> <username>
```

---

## 6. 常用命令速查

### 6.1 模块操作

| 命令 | 说明 |
|------|------|
| `lsmod` | 列出已加载模块 |
| `insmod <module.ko>` | 加载模块 |
| `rmmod <module_name>` | 卸载模块 |
| `modinfo <module.ko>` | 查看模块信息 |
| `modprobe <module_name>` | 智能加载模块 |

### 6.2 日志查看

| 命令 | 说明 |
|------|------|
| `dmesg` | 查看内核日志 |
| `journalctl -k` | 查看系统日志 |
| `cat /proc/kmsg` | 实时内核日志 |

### 6.3 调试工具

| 工具 | 用途 |
|------|------|
| `strace` | 系统调用跟踪 |
| `ltrace` | 库调用跟踪 |
| `perf` | 性能分析 |
| `ebpf` | 高级跟踪 |

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [modules/tzdriver.md](modules/tzdriver.md) | TZDriver 调试说明 |
| [02_Architecture.md](02_Architecture.md) | 架构设计 |
| [04_Security_Review.md](04_Security_Review.md) | 安全问题 |
