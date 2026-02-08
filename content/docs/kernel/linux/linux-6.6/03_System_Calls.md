# 系统调用接口

## 目的

本文档梳理 Linux 内核 6.6 的系统调用接口，包括分类、关键实现、参数校验和错误处理。

## 适用范围

- 读者目标：内核开发者、安全审计人员、系统调用开发者
- 核心版本：Linux 6.6

---

## 系统调用概述

**定义**: 系统调用是用户态程序请求内核服务的标准接口。

**调用方式**:
- `syscall` 指令 (x86_64: `syscall`, ARM64: `svc #0`)
- 通过 glibc 封装库调用（更易用）

**证据**:
- `arch/arm64/kernel/syscall.c` - ARM64 系统调用表
- `arch/x86/entry/syscall_64.c` - x86_64 系统调用表

---

## 系统调用表

### ARM64 系统调用表

**位置**: `arch/arm64/kernel/syscall.c`

**定义方式**:
```c
#define __SYSCALL(nr, sym) [nr] = sym,
const sys_call_ptr_t sys_call_table[__NR_syscalls] = {
    [0 ... __NR_syscalls - 1] = __arm64_sys_ni_syscall,
#include <asm/unistd.h>
};
#undef __SYSCALL
```

**证据**:
- `arch/arm64/kernel/syscall.c:56` - sys_call_table 定义

### x86_64 系统调用表

**位置**: `arch/x86/entry/syscall_64.c`

**证据**:
- `arch/x86/entry/syscall_64.c:17` - sys_call_table 定义

---

## 系统调用分类

### 1. 文件系统调用

**主要调用**:

| 系统调用 | 编号 | 内核实现 | 功能 |
|----------|------|---------|------|
| `open` | 2 | `ksys_open()` | 打开文件 |
| `close` | 3 | `ksys_close()` | 关闭文件 |
| `read` | 0 | `ksys_read()` | 读取数据 |
| `write` | 1 | `ksys_write()` | 写入数据 |
| `lseek` | 8 | `ksys_lseek()` | 文件指针定位 |
| `stat` | 4 | `ksys_newstat()` | 获取文件状态 |
| `fstat` | 5 | `ksys_newfstat()` | 获取文件状态 |
| `access` | 21 | `ksys_access()` | 检查访问权限 |
| `mkdir` | 83 | `ksys_mkdir()` | 创建目录 |
| `rmdir` | 84 | `ksys_rmdir()` | 删除目录 |
| `unlink` | 87 | `ksys_unlink()` | 删除文件 |
| `rename` | 82 | `ksys_renameat2()` | 重命名 |

**内核入口** (证据: `fs/open.c`):
- `ksys_open()` - `do_sys_openat2()` 的封装
- `ksys_read()` - `vfs_read()` 的封装
- `ksys_write()` - `vfs_write()` 的封装

**参数校验**:
- 用户态指针验证（`copy_from_user()`, `copy_to_user()`）
- 权限检查（`inode_permission()`）
- 路径遍历检查

**错误码**:
- `EBADF` - 无效文件描述符
- `EACCES` - 权限不足
- `ENOENT` - 文件不存在
- `ENOMEM` - 内存不足

---

### 2. 进程管理调用

**主要调用**:

| 系统调用 | 编号 | 内核实现 | 功能 |
|----------|------|---------|------|
| `fork` | 57 | `kernel_clone()` | 创建进程 |
| `vfork` | 58 | `kernel_clone()` | 创建进程（共享内存） |
| `execve` | 59 | `do_execve()` | 执行程序 |
| `exit` | 60 | `do_exit()` | 退出进程 |
| `wait4` | 61 | `kernel_wait4()` | 等待子进程 |
| `getpid` | 39 | `sys_getpid()` | 获取进程 ID |
| `getppid` | 110 | `sys_getppid()` | 获取父进程 ID |
| `kill` | 62 | `ksys_kill()` | 发送信号 |
| `getpriority` | 140 | `sys_getpriority()` | 获取优先级 |
| `setpriority` | 141 | `sys_setpriority()` | 设置优先级 |

**内核入口** (证据: `kernel/fork.c`, `kernel/exec.c`):
- `kernel_clone()` - `fork()`, `vfork()`, `clone()` 的实现
- `do_execve()` - `execve()` 的实现

**权限检查**:
- `capable(CAP_SYS_NICE)` - 修改其他进程优先级
- `capable(CAP_KILL)` - 杀死其他进程

---

### 3. 网络调用

**主要调用**:

| 系统调用 | 编号 | 内核实现 | 功能 |
|----------|------|---------|------|
| `socket` | 41 | `__sys_socket()` | 创建 socket |
| `bind` | 49 | `__sys_bind()` | 绑定地址 |
| `listen` | 50 | `__sys_listen()` | 监听连接 |
| `accept` | 43 | `__sys_accept4()` | 接受连接 |
| `connect` | 42 | `__sys_connect()` | 连接服务器 |
| `send` | 44 | `__sys_sendto()` | 发送数据 |
| `recv` | 45 | `__sys_recvfrom()` | 接收数据 |
| `shutdown` | 48 | `__sys_shutdown()` | 关闭连接 |

**内核入口** (证据: `net/socket.c`):
- `__sys_socket()` - Socket 创建
- `__sys_bind()` - 地址绑定
- `__sys_connect()` - 连接

**参数校验**:
- 用户态地址验证
- 端口范围检查
- LSM hook: `security_socket_create()`, `security_socket_bind()`

---

### 4. IPC 调用

**主要调用**:

| 系统调用 | 编号 | 内核实现 | 功能 |
|----------|------|---------|------|
| `shmget` | 29 | `ksys_shmget()` | 创建共享内存 |
| `shmat` | 30 | `ksys_shmat()` | 附加共享内存 |
| `shmdt` | 67 | `ksys_shmdt()` | 分离共享内存 |
| `msgsnd` | 46 | `ksys_msgsnd()` | 发送消息 |
| `msgrcv` | 47 | `ksys_msgrcv()` | 接收消息 |
| `semop` | 37 | `ksys_semop()` | 信号量操作 |

**内核入口** (证据: `ipc/msg.c`, `ipc/shm.c`, `ipc/sem.c`):
- `ksys_shmget()` - 共享内存创建
- `ksys_msgsnd()` - 消息发送

---

### 5. 内存管理调用

**主要调用**:

| 系统调用 | 编号 | 内核实现 | 功能 |
|----------|------|---------|------|
| `mmap` | 9 | `ksys_mmap_pgoff()` | 内存映射 |
| `munmap` | 11 | `ksys_munmap()` | 取消映射 |
| `mprotect` | 10 | `ksys_mprotect()` | 修改权限 |
| `brk` | 12 | `ksys_brk()` | 修改堆大小 |
| `madvise` | 28 | `ksys_madvise()` | 内存使用建议 |
| `mbind` | 237 | `ksys_mbind()` | 内存绑定 |

**内核入口** (证据: `mm/mmap.c`, `mm/mprotect.c`):
- `ksys_mmap_pgoff()` - `mmap()` 实现
- `ksys_munmap()` - `munmap()` 实现

---

### 6. 信号处理

**主要调用**:

| 系统调用 | 编号 | 内核实现 | 功能 |
|----------|------|---------|------|
| `signal` | 48 | `do_signal()` | 信号处理 |
| `sigaction` | 13 | `do_sigaction()` | 设置信号处理 |
| `sigprocmask` | 14 | `do_sigprocmask()` | 信号掩码 |
| `pause` | 34 | `do_sigtimedwait()` | 等待信号 |
| `kill` | 62 | `ksys_kill()` | 发送信号 |

**内核入口** (证据: `kernel/signal.c`):
- `do_sigaction()` - 信号处理设置
- `ksys_kill()` - 发送信号

---

## 参数校验策略

### 用户态指针验证

**目的**: 防止内核访问用户态非法地址。

**关键 API**:
```c
copy_from_user(to, from, n);   // 从用户态复制
copy_to_user(to, from, n);     // 复制到用户态
get_user(x, ptr);              // 获取单个值
put_user(x, ptr);              // 设置单个值
```

**证据**:
- `include/linux/uaccess.h` - 用户态访问 API

### 权限检查

**Capability 检查**:
```c
if (capable(CAP_SYS_ADMIN))
    // 允许操作
```

**证据**:
- `include/linux/capability.h` - Capability 定义
- `kernel/capability.c` - 检查实现

**LSM 检查**:
```c
ret = security_inode_permission(inode, mask);
if (ret)
    return ret;
```

**证据**:
- `security/security.c` - LSM 调度
- `include/linux/lsm_hook_defs.h` - Hook 定义

---

## 错误码规范

**常见错误码** (`include/uapi/asm-generic/errno.h`):

| 错误码 | 数值 | 含义 | 触发条件 |
|--------|------|------|---------|
| `EPERM` | 1 | 操作不允许 | 权限不足 |
| `ENOENT` | 2 | 文件不存在 | 路径无效 |
| `EACCES` | 13 | 权限不足 | LSM/文件权限检查失败 |
| `EFAULT` | 14 | 错误地址 | 用户态指针无效 |
| `EINVAL` | 22 | 无效参数 | 参数不合法 |
| `ENOMEM` | 12 | 内存不足 | `kmalloc()` 失败 |
| `EBADF` | 9 | 错误描述符 | 文件描述符无效 |

---

## 系统调用实现示例

### open() 系统调用

**调用链**:
```
用户态 open()
    ↓
do_sys_openat2() [fs/open.c]
    ↓
getname_flags() [获取路径]
    ↓
do_filp_open() [打开文件]
    ↓
path_openat() [路径解析]
    ↓
do_open() [实际打开]
    ↓
security_inode_permission() [LSM 检查]
    ↓
inode_permission() [权限检查]
    ↓
vfs_open() [VFS 打开]
    ↓
file->f_op->open() [文件系统打开]
```

**代码位置**: `fs/open.c:437`

**关键代码片段**:
```c
long do_sys_openat2(int dfd, const char __user *filename,
                  struct open_how *how)
{
    struct open_flags op;
    int fd;

    // 获取文件名并验证
    struct filename *tmp = getname_flags(filename, how->flags);
    if (IS_ERR(tmp))
        return PTR_ERR(tmp);

    // 解析打开标志
    fd = get_open_how_flags(how, &op);

    // 打开文件
    fd = do_filp_open(dfd, tmp, &op);
    putname(tmp);

    return fd;
}
```

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 核心概念
- [02_Architecture.md](02_Architecture.md) - 架构与数据流
- [04_Internal_APIs.md](04_Internal_APIs.md) - 内部 API
- [07_Security_Review.md](07_Security_Review.md) - 安全分析

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
