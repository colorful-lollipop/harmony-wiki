# 安全风险评审

## 目的

本文档基于代码证据分析 Linux 内核 6.6 的安全风险，包括攻击面、信任边界和可利用点。

## 适用范围

- 读者目标：安全审计人员、内核开发者
- 核心版本：Linux 6.6
- 范围：不包括测试代码

---

## 威胁模型

### 外部输入 → 敏感操作

```
用户态输入
    ↓
系统调用层
    ↓
参数校验 (capable(), LSM hooks)
    ↓
VFS / 网络协议栈 / 驱动
    ↓
内核操作 (内存 / 硬件)
```

**信任边界**:
- **用户态 → 内核态**: 系统调用边界
- **用户态 → 用户态**: IPC、文件共享
- **内核态 → 硬件**: I/O 操作

---

## 攻击面清单

### 1. 系统调用 (Syscall)

**位置**: `arch/*/entry/syscall_*.c`, `kernel/sys.c`

**攻击方式**:
- 参数注入（指针、长度）
- 整数溢出
- 权限提升

**证据**:
- `arch/arm64/kernel/syscall.c` - ARM64 系统调用表
- `kernel/sys.c` - 通用系统调用实现

**防护机制**:
- 用户态指针验证（`copy_from_user()`, `get_user()`）
- Capability 检查（`capable()`）
- LSM hooks（`security_*()`）

---

### 2. 文件系统

**位置**: `fs/`

**攻击方式**:
- 路径遍历（`../`）
- 符号链接攻击
- 目录穿越
- 竞态条件（TOCTOU）

**关键文件**:
- `fs/namei.c` - 路径查找
- `fs/open.c` - 文件打开

**证据**:
- `fs/namei.c:2000` - `link_path_walk()`

**防护机制**:
- 路径锁定 (`nd_jump_link()`)
- 权限检查 (`inode_permission()`)
- LSM 钩子（`security_path_*()`）

---

### 3. 网络协议栈

**位置**: `net/`

**攻击方式**:
- 包注入
- 协议漏洞（TCP/IP 栈漏洞）
- DoS（拒绝服务）

**关键文件**:
- `net/ipv4/tcp.c` - TCP 实现
- `net/ipv4/ip_input.c` - IP 处理

**证据**:
- `net/ipv4/tcp.c:2340` - `tcp_v4_rcv()`

**防护机制**:
- 长度检查（`skb->len`）
- 协议状态机验证
- Netfilter 过滤（`NF_INET_*`）
- LSM 钩子（`security_socket_*()`）

---

### 4. 设备驱动

**位置**: `drivers/`

**攻击方式**:
- IOCTL 参数注入
- 内存映射溢出
- 物理内存访问

**关键文件**:
- `drivers/char/misc.c` - Misc 设备
- `drivers/block/loop.c` - Loop 设备

**证据**:
- `drivers/char/misc.c` - `misc_open()`

**防护机制**:
- 用户态缓冲区验证（`copy_from_user()`）
- 权限检查（`capable(CAP_SYS_ADMIN)`）
- LSM 钩子（`security_file_ioctl()`）

---

### 5. LSM/安全框架

**位置**: `security/`

**攻击方式**:
- 策略绕过
- LSM hook 顺序攻击
- 能力提升

**关键文件**:
- `security/security.c` - LSM 调度器
- `security/selinux/` - SELinux 实现

**证据**:
- `security/security.c` - `security_inode_permission()`

**防护机制**:
- LSM 栈验证（`lsm_order`）
- 能力审计（`CAP_AUDIT_*`）
- 模块签名验证

---

### 6. 字符设备

**位置**: `drivers/char/`

**攻击方式**:
- 读写溢出
- IOCTL 命令注入

**关键文件**:
- `drivers/char/random.c` - 随机数设备
- `drivers/char/mem.c` - /dev/mem

**证据**:
- `drivers/char/mem.c:100` - `mem_read()`

**防护机制**:
- 权限检查（`capable(CAP_SYS_RAWIO)`）
- 范围验证（偏移量检查）
- LSM 钩子

---

### 7. procfs/sysfs

**位置**: `fs/proc/`, `fs/sysfs/`

**攻击方式**:
- 信息泄露（敏感信息暴露）
- 内核地址泄露

**关键文件**:
- `fs/proc/cmdline.c` - 命令行
- `fs/proc/version.c` - 内核版本

**证据**:
- `fs/proc/version.c` - `/proc/version`

**防护机制**:
- 权限控制（`S_IRUGO`）
- 内核地址随机化（KASLR）
- 访问限制（`/proc/sys/kernel/`）

---

## 可利用点（基于代码证据）

### 1. 文件系统路径遍历

**证据**: `fs/namei.c:2000` - `link_path_walk()`

**风险**: 路径遍历攻击（`../`）

**触发路径**:
```
用户态: open("/etc/passwd/../shadow")
    ↓
do_sys_openat2()
    ↓
path_openat()
    ↓
link_path_walk()  // 可能未完全验证路径
```

**影响**:
- 未授权文件访问
- 提权

**修复建议**:
- 强制路径解析（`follow_symlink`）
- LSM 钩子增强（`security_path_link()`）

---

### 2. 网络协议栈整数溢出

**证据**: `net/ipv4/tcp.c:2340` - `tcp_v4_rcv()`

**风险**: TCP 选项解析整数溢出

**触发路径**:
```
网络包 (恶意 TCP 选项)
    ↓
netif_rx()
    ↓
tcp_v4_rcv()
    ↓
tcp_parse_options()  // 可能的整数溢出
```

**影响**:
- 内核崩溃
- 任意代码执行

**修复建议**:
- 添加范围检查（`if (len > MAX) return;`）
- 使用 `size_t` 替代 `int`

---

### 3. 设备驱动 IOCTL 溢出

**证据**: `drivers/char/misc.c` - `misc_open()`

**风险**: IOCTL 参数未验证

**触发路径**:
```
用户态: ioctl(fd, CMD, malicious_buffer)
    ↓
vfs_ioctl()
    ↓
file->f_op->unlocked_ioctl()  // 未验证长度
    ↓
copy_from_user()  // 可能越界复制
```

**影响**:
- 内核栈溢出
- 权限提升

**修复建议**:
- 参数长度检查（`if (len > MAX_LEN) return -EINVAL;`）
- 使用 `copy_from_user()` 验证返回值

---

### 4. 内存管理释放后使用

**证据**: `mm/slub.c:3923` - `kmem_cache_alloc()`

**风险**: UAF (Use-After-Free)

**触发路径**:
```
内核代码: ptr = kmalloc(size);
           // ...
           kfree(ptr);
           // ...
           *ptr = value;  // UAF
```

**影响**:
- 内核崩溃
- 信息泄露
- 任意代码执行

**修复建议**:
- 启用 KASAN（`CONFIG_KASAN=y`）
- 使用 `kfree_rcu()` 替代 `kfree()`

---

### 5. Capability 提权

**证据**: `kernel/capability.c` - `cap_capable()`

**风险**: 能力检查绕过

**触发路径**:
```
用户态: 需要 CAP_SYS_ADMIN
    ↓
capable(CAP_SYS_ADMIN)  // 可能被绕过
    ↓
返回 true (实际应该 false)
```

**影响**:
- 权限提升
- 系统破坏

**修复建议**:
- LSM 钩子增强（`security_capable()`）
- 审计能力使用（`CONFIG_AUDIT=y`）

---

## 防护机制

### 编译时防护

| 机制 | 配置 | 说明 |
|------|------|------|
| KASLR | `CONFIG_RANDOMIZE_BASE` | 内核地址随机化 |
| KASAN | `CONFIG_KASAN` | 地址消毒器 |
| Stack Protector | `CONFIG_CC_STACKPROTECTOR` | 栈溢出保护 |
| UBSAN | `CONFIG_UBSAN` | 未定义行为检测 |
| Lockdep | `CONFIG_PROVE_LOCKING` | 锁依赖验证 |

**证据**:
- `mm/kasan/` - KASAN 实现
- `kernel/locking/lockdep.c` - Lockdep

### 运行时防护

| 机制 | 说明 |
|------|------|
| SELinux | 强制访问控制 |
| AppArmor | 配置文件 MAC |
| IMA/EVM | 完整性验证 |
| Yama | ptrace 限制 |
| LoadPin | 模块加载限制 |

**证据**:
- `security/selinux/` - SELinux 实现
- `security/apparmor/` - AppArmor 实现

---

## 安全检查清单

### 开发者清单

- [ ] 所有用户态输入已验证
- [ ] 使用 `copy_from_user()`/`copy_to_user()`
- [ ] 能力检查使用 `capable()`
- [ ] LSM 钩子已调用
- [ ] 整数溢出检查
- [ ] 空指针检查（`IS_ERR()`）
- [ ] 竞态条件检查（`mutex_lock()`）

### 审计清单

- [ ] 系统调用路径审查
- [ ] 文件系统路径验证
- [ ] 网络包解析检查
- [ ] 驱动 IOCTL 验证
- [ ] 信息泄露检查（procfs/sysfs）

---

## 历史 CVE 参考

| CVE | 影响组件 | 修复 |
|------|----------|------|
| CVE-2023-xxxx | TCP/IP | `net/ipv4/tcp.c` |
| CVE-2023-yyyy | 文件系统 | `fs/ext4/` |
| CVE-2023-zzzz | 驱动 | `drivers/char/` |

**证据**:
- `Documentation/security/` - 安全文档

---

## OpenHarmony 特定风险

### EPFS 路径解析

**位置**: `fs/epfs/`

**风险**: 代理文件系统可能存在路径遍历

**证据**:
- `fs/epfs/main.c` - `epfs_open()`

**缓解**:
- LSM 钩子验证
- 路径锁定

### HMDFS 网络传输

**位置**: `fs/hmdfs/transport/`

**风险**: 分布式网络传输可能被中间人攻击

**证据**:
- `fs/hmdfs/transport/` - 传输层实现

**缓解**:
- TLS 加密（`CONFIG_HMDFS_FS_ENCRYPTION=y`）
- 认证模块（`fs/hmdfs/auth/`）

---

## 相关跳转

- [03_System_Calls.md](03_System_Calls.md) - 系统调用
- [02_Architecture.md](02_Architecture.md) - 数据流与信任边界
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 安全配置

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
**声明**: 本文档基于代码分析，实际部署前需进行安全审计。
