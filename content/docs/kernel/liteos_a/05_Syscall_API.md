# 系统调用接口

## 概述

系统调用 (syscall) 是用户态与内核态交互的唯一接口。LiteOS-A 通过 `syscall/` 目录实现 POSIX 兼容的系统调用接口。

**代码位置**: `syscall/`

## 系统调用入口

### 关键文件

| 文件 | 职责 |
|------|------|
| `syscall_lookup.h` | syscall 查找表宏定义 |
| `syscall_pub.h` | 用户态/内核态拷贝宏 |
| `los_syscall.c` | syscall 入口框架 |
| `los_syscall.h` | syscall 头文件 |

### syscall 调用流程

```
用户态                   内核态
   │                       │
   ├─→ libc wrapper        │
   │                       │
   ├─→ SVC #0 (软中断)      │
   │                       ├─→ los_syscall.c:OsArmAarch64SyscallHandle
   │                       │         或
   │                       ├─→ los_syscall.c:OsArm32SyscallHandle
   │                       │
   │                       ├─→ 根据 nr 查表 (syscall_lookup.h)
   │                       │
   │                       ├─→ 调用具体处理函数
   │                       │
   └─← 返回结果            ─┘
```

## Syscall 分类

### 1. 文件系统 Syscall

**证据**: `syscall_lookup.h:34-137`

| Syscall | 处理器函数 | 参数 | 说明 |
|---------|-----------|------|------|
| `read` | `SysRead` | fd, buf, count | 读取文件 |
| `write` | `SysWrite` | fd, buf, count | 写入文件 |
| `open` | `SysOpen` | path, flags, mode | 打开文件 |
| `close` | `SysClose` | fd | 关闭文件 |
| `creat` | `SysCreat` | path, mode | 创建文件 |
| `unlink` | `SysUnlink` | path | 删除文件 |
| `link` | `SysLink` | oldpath, newpath | 创建硬链接 |
| `symlink` | `SysSymlink` | target, linkpath | 创建软链接 |
| `readlink` | `SysReadlink` | path, buf, bufsz | 读取软链接 |
| `mkdir` | `SysMkdir` | path, mode | 创建目录 |
| `rmdir` | `SysRmdir` | path | 删除目录 |
| `chdir` | `SysChdir` | path | 切换目录 |
| `fchdir` | `SysFchdir` | fd | FD切换目录 |
| `getcwd` | `SysGetcwd` | buf, size | 获取当前目录 |
| `rename` | `SysRename` | oldpath, newpath | 重命名 |
| `chmod` | `SysChmod` | path, mode | 修改权限 |
| `fchmod` | `SysFchmod` | fd, mode | FD修改权限 |
| `chown` | `SysChown` | path, owner, group | 修改所有者 |
| `fchown` | `SysFchown` | fd, owner, group | FD修改所有者 |
| `stat` | `SysStat` | path, statbuf | 获取文件状态 |
| `fstat` | `SysFstat` | fd, statbuf | FD获取状态 |
| `lstat` | `SysLstat` | path, statbuf | 获取链接状态 |
| `access` | `SysAccess` | path, mode | 检查访问权限 |
| `lseek` | `SysLseek` | fd, offset, whence | 文件偏移 |
| `ioctl` | `SysIoctl` | fd, request, ... | IO控制 |
| `fcntl` | `SysFcntl` | fd, cmd, ... | 文件控制 |
| `dup` | `SysDup` | fd | 复制FD |
| `dup2` | `SysDup2` | fd1, fd2 | 指定复制FD |
| `pipe` | `SysPipe` | pipefd | 创建管道 |
| `select` | `SysSelect` | nfds, readfds, ... | 多路IO |
| `poll` | `SysPoll` | fds, nfds, timeout | 多路IO |
| `mount` | `SysMount` | source, target, ... | 挂载文件系统 |
| `umount` | `SysUmount` | target | 卸载文件系统 |
| `statfs` | `SysStatfs` | path, buf | 文件系统状态 |
| `getdents` | `SysGetdents64` | fd, dirp, count | 读取目录项 |
| `mmap` | `SysMmap` | addr, len, prot, ... | 内存映射 |
| `munmap` | `SysMunmap` | addr, len | 解除映射 |
| `mprotect` | `SysMprotect` | addr, len, prot | 内存保护 |

### 2. 进程/线程 Syscall

**证据**: `syscall_lookup.h:139-224`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `exit` | `SysThreadExit` | 线程退出 |
| `exit_group` | `SysUserExitGroup` | 进程退出 |
| `fork` | `SysFork` | 创建进程 |
| `vfork` | `SysVfork` | 创建进程(vfork) |
| `clone` | `SysClone` | 创建线程 |
| `execve` | `SysExecve` | 执行程序 |
| `wait4` | `SysWait` | 等待进程 |
| `waitid` | `SysWaitid` | 等待指定进程 |
| `getpid` | `SysGetPID` | 获取进程ID |
| `getppid` | `SysGetPPID` | 获取父进程ID |
| `gettid` | `SysGetTid` | 获取线程ID |
| `kill` | `SysKill` | 发送信号 |
| `tkill` | `SysPthreadKill` | 发送信号给线程 |
| `pause` | `SysPause` | 等待信号 |
| `brk` | `SysBrk` | 设置进程break |
| `unshare` | `SysUnshare` | 取消共享 |
| `setns` | `SysSetns` | 设置命名空间 |

### 3. 调度 Syscall

**证据**: `syscall_lookup.h:171-180`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `sched_yield` | `SysSchedYield` | 让出CPU |
| `sched_setparam` | `SysSchedSetParam` | 设置调度参数 |
| `sched_getparam` | `SysSchedGetParam` | 获取调度参数 |
| `sched_setscheduler` | `SysSchedSetScheduler` | 设置调度策略 |
| `sched_getscheduler` | `SysSchedGetScheduler` | 获取调度策略 |
| `sched_setaffinity` | `SysSchedSetAffinity` | 设置CPU亲和性 |
| `sched_getaffinity` | `SysSchedGetAffinity` | 获取CPU亲和性 |
| `sched_rr_get_interval` | `SysSchedRRGetInterval` | 获取时间片 |

### 4. 内存管理 Syscall

**证据**: `syscall_lookup.h:182`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `mmap2` | `SysMmap` | 内存映射 |
| `mremap` | `SysMremap` | 重映射 |
| `munmap` | `SysMunmap` | 解除映射 |
| `mprotect` | `SysMprotect` | 内存保护 |

### 5. 时间/定时器 Syscall

**证据**: `syscall_lookup.h:224-238`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `timer_create` | `SysTimerCreate` | 创建定时器 |
| `timer_settime` | `SysTimerSettime` | 设置定时器 |
| `timer_gettime` | `SysTimerGettime` | 获取定时器时间 |
| `timer_getoverrun` | `SysTimerGetoverrun` | 获取超时次数 |
| `timer_delete` | `SysTimerDelete` | 删除定时器 |
| `clock_gettime` | `SysClockGettime` | 获取时钟时间 |
| `clock_settime` | `SysClockSettime` | 设置时钟时间 |
| `clock_getres` | `SysClockGetres` | 获取时钟分辨率 |
| `nanosleep` | `SysNanoSleep` | 纳秒睡眠 |

### 6. 信号 Syscall

**证据**: `syscall_lookup.h:185-189`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `rt_sigaction` | `SysSigAction` | 设置信号处理 |
| `rt_sigprocmask` | `SysSigprocMask` | 设置信号掩码 |
| `rt_sigpending` | `SysSigPending` | 待处理信号 |
| `rt_sigtimedwait` | `SysSigTimedWait` | 限时等待信号 |
| `rt_sigsuspend` | `SysSigSuspend` | 暂停等待信号 |

### 7. 用户/组 ID Syscall

**证据**: `syscall_lookup.h:191-214`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `getuid` | `SysGetUserID` | 获取真实用户ID |
| `getgid` | `SysGetGroupID` | 获取真实组ID |
| `geteuid` | `SysGetEffUserID` | 获取有效用户ID |
| `getegid` | `SysGetEffGID` | 获取有效组ID |
| `setuid` | `SysSetUserID` | 设置用户ID |
| `setgid` | `SysSetGroupID` | 设置组ID |
| `setreuid` | `SysSetRealEffUserID` | 设置真实/有效UID |
| `setregid` | `SysSetRealEffGroupID` | 设置真实/有效GID |
| `setresuid` | `SysSetRealEffSaveUserID` | 设置所有用户ID |
| `setresgid` | `SysSetRealEffSaveGroupID` | 设置所有组ID |
| `getresuid` | `SysGetRealEffSaveUserID` | 获取所有用户ID |
| `getresgid` | `SysGetRealEffSaveGroupID` | 获取所有组ID |

### 8. 网络 Socket Syscall

**证据**: `syscall_lookup.h:240-257`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `socket` | `SysSocket` | 创建socket |
| `bind` | `SysBind` | 绑定地址 |
| `connect` | `SysConnect` | 连接 |
| `listen` | `SysListen` | 监听 |
| `accept` | `SysAccept` | 接受连接 |
| `send` | `SysSend` | 发送数据 |
| `sendto` | `SysSendTo` | 发送数据(带地址) |
| `recv` | `SysRecv` | 接收数据 |
| `recvfrom` | `SysRecvFrom` | 接收数据(带地址) |
| `shutdown` | `SysShutdown` | 关闭socket |
| `setsockopt` | `SysSetSockOpt` | 设置选项 |
| `getsockopt` | `SysGetSockOpt` | 获取选项 |
| `getsockname` | `SysGetSockName` | 获取本地地址 |
| `getpeername` | `SysGetPeerName` | 获取远端地址 |

### 9. 共享内存 Syscall

**证据**: `syscall_lookup.h:259-264`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `shmget` | `SysShmGet` | 创建/获取共享内存 |
| `shmat` | `SysShmAt` | 附加共享内存 |
| `shmdt` | `SysShmDt` | 分离共享内存 |
| `shmctl` | `SysShmCtl` | 控制共享内存 |

### 10. 消息队列 Syscall

**证据**: `syscall_lookup.h:233-238`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `mq_open` | `SysMqOpen` | 打开消息队列 |
| `mq_unlink` | `SysMqUnlink` | 删除消息队列 |
| `mq_send` | `SysMqTimedSend` | 发送消息 |
| `mq_receive` | `SysMqTimedReceive` | 接收消息 |
| `mq_notify` | `SysMqNotify` | 通知 |
| `mq_getattr` | `SysMqGetSetAttr` | 获取属性 |

### 11. 能力 (Capability) Syscall

**证据**: `syscall_lookup.h:195-198`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `ohos_capget` | `SysCapGet` | 获取能力 |
| `ohos_capset` | `SysCapSet` | 设置能力 |

### 12. LiteOS 特有 Syscall

**证据**: `syscall_lookup.h:273-280`

| Syscall | 处理器函数 | 说明 |
|---------|-----------|------|
| `pthread_set_detach` | `SysUserThreadSetDetach` | 设置线程分离 |
| `pthread_join` | `SysThreadJoin` | 等待线程 |
| `pthread_deatch` | `SysUserThreadDetach` | 分离线程 |
| `create_user_thread` | `SysCreateUserThread` | 创建用户线程 |
| `getrusage` | `SysGetrusage` | 获取资源使用 |
| `sysconf` | `SysSysconf` | 获取系统配置 |
| `ugetrlimit` | `SysUgetrlimit` | 获取资源限制 |
| `setrlimit` | `SysSetrlimit` | 设置资源限制 |

## 用户态/内核态数据传输

**证据**: `syscall_pub.h`

### 关键宏定义

| 宏 | 用途 |
|----|------|
| `CHECK_ASPACE()` | 检查用户指针地址空间 |
| `DUP_FROM_USER()` | 从用户态拷贝数据 |
| `DUP_TO_USER()` | 拷贝数据到用户态 |
| `CPY_FROM_USER()` | 复制用户态指针指向的数据 |
| `CPY_TO_USER()` | 复制数据到用户态指针 |

### 使用示例

```c
// 检查用户指针
CHECK_ASPACE(userPtr, size, return -EFAULT);

// 从用户态拷贝
DUP_FROM_USER(userBuf, size, return -ENOMEM);

// 处理完成后拷贝回用户态
DUP_TO_USER(userBuf, size);
```

## 相关文档

- [架构说明](/02_Architecture.md)
- [内核模块详解](/04_Kernel_Modules.md)
- [安全评审](/06_Security_Review.md)
