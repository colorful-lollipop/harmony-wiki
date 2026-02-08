# 安全风险评审

## 概述

本文档对 OpenHarmony Linux Kernel 5.10 进行安全风险评审，识别攻击面、信任边界和潜在的可利用点。

## 1. 安全架构概览

### 1.1 LSM (Linux Security Modules) 框架

**核心文件**: `security/security.c`

Linux 内核采用 LSM 框架支持多种安全模块堆叠。OpenHarmony 默认可能启用的模块：

```
LSM 堆栈顺序:
1. lockdown     - 内核锁定模式
2. yama         - ptrace 限制
3. loadpin      - 模块加载位置限制
4. safesetid    - setuid 安全
5. integrity    - IMA/EVM 完整性
6. selinux      - SELinux (可选)
7. smack        - Smack (可选)
8. tomoyo       - TOMOYO (可选)
9. apparmor     - AppArmor (可选)
10. bpf         - BPF LSM
```

### 1.2 安全模块配置

**配置文件**: `security/Kconfig`

| 配置项 | 说明 |
|--------|------|
| `CONFIG_SECURITY` | 启用安全模块框架 |
| `CONFIG_SECURITY_SELINUX` | SELinux 支持 |
| `CONFIG_SECURITY_APPARMOR` | AppArmor 支持 |
| `CONFIG_SECURITY_NETWORK` | 网络安全 hooks |
| `CONFIG_SECURITY_PATH` | 路径访问控制 hooks |
| `CONFIG_SECURITY_WRITABLE_HOOKS` | 允许运行时修改 hooks |

## 2. 攻击面分析

### 2.1 系统调用攻击面

**入口**: 400+ 系统调用，位于 `kernel/` 和 `arch/*/kernel/`

| 攻击向量 | 风险等级 | 代码位置 |
|----------|----------|----------|
| 指针参数 | 高 | 所有 SYSCALL_DEFINE* |
| 长度参数 | 高 | copy_from/to_user 调用点 |
| 文件描述符 | 中 | 文件操作系统调用 |
| 权限检查绕过 | 高 | capable() 调用点 |

**关键系统调用风险点**:
- `kernel/sys.c` - 系统信息/控制 (setpriority, prctl)
- `kernel/fork.c` - 进程创建 (clone, fork)
- `kernel/signal.c` - 信号处理 (kill, rt_sigqueueinfo)
- `kernel/module.c` - 模块加载 (init_module, finit_module)
- `kernel/bpf/syscall.c` - BPF 操作 (bpf)
- `mm/mmap.c` - 内存映射 (mmap)

### 2.2 Proc/Sysfs 攻击面

**入口**: `fs/proc/`, `fs/sysfs/`

| 接口 | 风险 | 位置 |
|------|------|------|
| `/proc/kmsg` | 信息泄露 | `fs/proc/kmsg.c` |
| `/proc/[pid]/mem` | 内存访问 | `fs/proc/base.c` |
| `/proc/sys/kernel/*` | 配置篡改 | `kernel/sysctl.c` |
| `/sys/kernel/debug/*` | 调试信息泄露 | `kernel/debug/` |

### 2.3 Ioctl 攻击面

**入口**: 遍布 `drivers/`, `fs/`

| 子系统 | 风险 | 位置 |
|--------|------|------|
| 块设备 | 设备控制 | `block/ioctl.c` |
| 字符设备 | 驱动特定 | `drivers/char/` |
| GPU/DRM | 图形内存 | `drivers/gpu/drm/` |
| 网络设备 | 网卡配置 | `drivers/net/` |

### 2.4 网络攻击面

**入口**: `net/`, `drivers/net/`

| 协议 | 风险 | 位置 |
|------|------|------|
| IPv4/IPv6 | 协议解析 | `net/ipv4/`, `net/ipv6/` |
| TCP/UDP | 协议状态机 | `net/ipv4/tcp.c`, `udp.c` |
| Netfilter | 规则处理 | `net/netfilter/` |
| BPF | 程序验证 | `kernel/bpf/` |

### 2.5 模块加载攻击面

**入口**: `kernel/module.c`

| 风险 | 描述 | 代码 |
|------|------|------|
| 未签名模块 | 加载恶意代码 | `kernel/module.c:4189` (init_module) |
| 符号劫持 | 覆盖内核符号 | `kernel/module.c` |
| 模块参数 | 命令注入 | 模块初始化函数 |

### 2.6 OpenHarmony 特有攻击面

**入口**: `drivers/staging/`, `include/dfx/`

| 组件 | 风险 | 位置 |
|------|------|------|
| HiLog | 日志注入 | `drivers/staging/hilog/hilog.c` |
| Blackbox | 崩溃信息泄露 | `drivers/staging/blackbox/` |
| HMDFS | 分布式 FS 漏洞 | `fs/hmdfs/` |
| HyperHold | 内存管理 | `drivers/hyperhold/` |
| HCK | 厂商钩子 | `drivers/hck/vendor_hooks.c` |

## 3. 信任边界

### 3.1 用户空间/内核空间边界

```
+-------------------------+
|     User Space          |
|  (OpenHarmony Apps)     |
+-------------------------+
|  System Call Interface  |  <-- 信任边界
+-------------------------+
|     Kernel Space        |
|   (This Repository)     |
+-------------------------+
|    Hardware Layer       |
+-------------------------+
```

**边界检查点**:
- `copy_from_user()` / `copy_to_user()` - 用户空间拷贝
- `access_ok()` - 地址有效性检查
- `capable()` - 权限检查

### 3.2 特权边界

| 特权级别 | 代码位置 | 检查函数 |
|----------|----------|----------|
| 非特权用户 | 所有系统调用 | `capable(CAP_*)` |
| 特权进程 | `kernel/capability.c` | `capable()`, `ns_capable()` |
| Root 用户 | `kernel/sys.c` | `uid_eq()`, `gid_eq()` |

### 3.3 模块边界

| 边界类型 | 控制机制 |
|----------|----------|
| 内核/模块 | 符号导出表 `EXPORT_SYMBOL()` |
| 模块加载 | `CONFIG_MODULE_SIG`, `CONFIG_MODULE_FORCE_LOAD` |
| 模块参数 | `module_param()` 定义 |

## 4. 可被利用点分析

### 4.1 高危风险点

#### 风险 1: 系统调用参数验证不足

**证据**: `kernel/sys.c:2354` (prctl)
```c
SYSCALL_DEFINE5(prctl, int, option, unsigned long, arg2, unsigned long, arg3,
                unsigned long, arg4, unsigned long, arg5)
```

**风险**: 某些 prctl 选项可能未充分验证参数

**影响**: 本地权限提升、信息泄露

**修复建议**:
- 审查所有 prctl 选项的参数验证
- 添加边界检查
- 启用 `CONFIG_SECCOMP`

#### 风险 2: 模块加载缺乏强制签名

**证据**: `kernel/module.c:4189` (init_module)
```c
SYSCALL_DEFINE3(init_module, void __user *, umod, unsigned long, len,
                const char __user *, uargs)
```

**风险**: 如果未启用 `CONFIG_MODULE_SIG_FORCE`，可加载未签名模块

**影响**: 加载恶意内核模块

**修复建议**:
- 启用 `CONFIG_MODULE_SIG=y`
- 启用 `CONFIG_MODULE_SIG_FORCE=y`
- 配置模块签名密钥

#### 风险 3: BPF 程序验证器绕过

**证据**: `kernel/bpf/syscall.c:4404` (bpf)
```c
SYSCALL_DEFINE3(bpf, int, cmd, union bpf_attr __user *, uattr, unsigned int, size)
```

**风险**: BPF 验证器复杂，可能存在绕过

**影响**: 内核内存损坏、权限提升

**修复建议**:
- 启用 `CONFIG_BPF_JIT_ALWAYS_ON`
- 限制 BPF 程序大小
- 启用 unprivileged BPF 限制

#### 风险 4: 用户空间拷贝溢出

**证据**: `mm/usercopy.c`
```c
// hardened usercopy 检查
```

**风险**: 未启用 `CONFIG_HARDENED_USERCOPY` 时可能发生拷贝溢出

**影响**: 内存损坏、信息泄露

**修复建议**:
- 启用 `CONFIG_HARDENED_USERCOPY=y`
- 启用 `CONFIG_HARDENED_USERCOPY_FALLBACK` 用于调试

#### 风险 5: 文件系统路径遍历

**证据**: `fs/namei.c`
```c
// 路径解析和权限检查
```

**风险**: 符号链接攻击、路径遍历

**影响**: 未授权文件访问

**修复建议**:
- 启用 `CONFIG_SECURITY_PATH=y`
- 使用 `O_NOFOLLOW` 标志
- 验证路径规范化

### 4.2 中危风险点

#### 风险 6: HiLog 日志注入

**证据**: `drivers/staging/hilog/hilog.c`
```c
// 日志设备 /dev/hilog
```

**风险**: 恶意进程可能注入伪造日志

**影响**: 日志污染、审计绕过

**修复建议**:
- 验证日志来源
- 实施日志完整性检查
- 限制日志写入权限

#### 风险 7: Blackbox 信息泄露

**证据**: `drivers/staging/blackbox/blackbox_core.c`
```c
// 崩溃信息收集
```

**风险**: 崩溃日志可能包含敏感信息

**影响**: 信息泄露

**修复建议**:
- 过滤敏感信息
- 限制 blackbox 访问权限
- 加密存储崩溃日志

#### 风险 8: HCK 钩子滥用

**证据**: `drivers/hck/vendor_hooks.c`
```c
// 厂商钩子框架
```

**风险**: 厂商钩子可能被滥用修改内核行为

**影响**: 内核完整性破坏

**修复建议**:
- 审查所有钩子注册
- 启用完整性度量
- 限制钩子可修改的范围

## 5. 安全配置建议

### 5.1 推荐安全配置

```
# 安全模块
CONFIG_SECURITY=y
CONFIG_SECURITY_SELINUX=y
CONFIG_SECURITY_APPARMOR=y
CONFIG_SECURITY_NETWORK=y
CONFIG_SECURITY_PATH=y

# 模块签名
CONFIG_MODULE_SIG=y
CONFIG_MODULE_SIG_FORCE=y
CONFIG_MODULE_SIG_SHA256=y

# 内存安全
CONFIG_KASAN=y
CONFIG_STACKPROTECTOR=y
CONFIG_HARDENED_USERCOPY=y
CONFIG_INIT_STACK_ALL_ZERO=y

# 加固
CONFIG_FORTIFY_SOURCE=y
CONFIG_SECURITY_DMESG_RESTRICT=y
CONFIG_DEBUG_KERNEL=n

# Seccomp
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y

# BPF 安全
CONFIG_BPF_JIT_ALWAYS_ON=y
CONFIG_UNPRIVILEGED_BPF_DISABLED=y
```

### 5.2 OpenHarmony 特有安全建议

1. **启用 HiLog 来源验证**
2. **限制 Blackbox 访问**
3. **审查 HCK 钩子**
4. **启用 HMDFS 访问控制**

## 6. 安全监控与审计

### 6.1 审计机制

**文件**: `kernel/audit.c`

| 事件类型 | 描述 |
|----------|------|
| `AUDIT_SYSCALL` | 系统调用审计 |
| `AUDIT_PATH` | 文件访问审计 |
| `AUDIT_CONFIG_CHANGE` | 配置变更 |

### 6.2 安全日志

**HiSysEvent**: `drivers/staging/hisysevent/`
- 系统事件记录
- 安全事件上报

**HiLog**: `drivers/staging/hilog/`
- 内核日志输出
- 日志级别控制

## 7. CVE 处理流程

### 7.1 OpenHarmony 补丁格式

参考 `README` 文件：

```
ohos inclusion
CVE: $cve-id or NA
issue: $issue-id or NA
bugzilla: $bug-id or NA

changelog

Signed-off-by: $name <$email>
```

### 7.2 安全更新检查

- 上游 Linux Stable: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-5.10.y
- OpenHarmony Issue: https://gitee.com/openharmony/kernel_linux/issues

## 8. 安全测试建议

### 8.1 静态分析

```bash
# 使用 checkpatch.pl 检查补丁
./scripts/checkpatch.pl patch.diff

# 使用 sparse 进行静态分析
make C=1

# 使用 clang-analyzer
scan-build make
```

### 8.2 动态测试

```bash
# 启用 KASAN
CONFIG_KASAN=y

# 启用 KFENCE
CONFIG_KFENCE=y

# 使用 syzkaller 进行模糊测试
```

### 8.3 模糊测试目标

| 目标 | 类型 |
|------|------|
| 系统调用 | syscall fuzzing |
| ioctl | ioctl fuzzing |
| 文件系统 | filesystem fuzzing |
| 网络协议 | network fuzzing |
| BPF | BPF verifier fuzzing |

---

*生成时间: 2026-02-06*

**注意**: 本文档基于代码分析，实际部署时还需结合具体配置和硬件环境进行全面安全评估。
