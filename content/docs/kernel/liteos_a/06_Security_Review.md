# 安全风险评审

## 概述

本文档基于代码分析，对 LiteOS-A 内核进行安全风险评估，识别潜在的攻击面和可被利用点。

**评审范围**: `kernel/liteos_a` 核心模块 (不含测试)

## 攻击面分析

### 1. 系统调用入口
**风险等级**: 中

- **位置**: `syscall/`
- **攻击面**: 用户态通过 syscall 进入内核的唯一入口
- **威胁**: 恶意构造 syscall 参数可能导致：
  - 内核态内存越界访问
  - 提权操作
  - 拒绝服务

### 2. 文件系统操作
**风险等级**: 中

- **位置**: `fs/`, `syscall/fs_syscall.c`
- **攻击面**: 文件读写、挂载、设备节点操作
- **威胁**:
  - 路径遍历 (`..` 注入)
  - 符号链接攻击
  - 设备文件滥用

### 3. 进程/线程管理
**风险等级**: 低

- **位置**: `kernel/base/core/los_process.c`, `los_task.c`
- **攻击面**: 进程创建、调度、信号发送
- **威胁**: 资源耗尽 (fork 炸弹)

### 4. 网络栈
**风险等级**: 低

- **位置**: `net/` (lwIP)
- **攻击面**: Socket 操作
- **威胁**: 网络协议栈漏洞 (依赖第三方代码)

### 5. 设备驱动
**风险等级**: 低

- **位置**: `drivers/`, `bsd/`
- **攻击面**: 字符设备、块设备
- **威胁**: 直接内存访问

## 可被利用点分析

### 1. 用户指针校验不完整

**证据**: `syscall/syscall_pub.h:48-66`

```c
#define CHECK_ASPACE(ptr, len, ...) \
    do { \
        if (ptr != NULL && len != 0) { \
            if (!LOS_IsUserAddressRange((VADDR_T)(UINTPTR)ptr, len)) { \
                set_errno(EFAULT); \
                __VA_ARGS__; \
                return -get_errno(); \
            } \
```

**问题**: `CHECK_ASPACE` 宏只检查地址范围，不检查指针是否有效 (valid)

**触发条件**: 用户传入已释放或未映射的指针

**影响**: 内核访问无效地址可能导致:
- 内核崩溃 (NULL ptr dereference)
- 信息泄露 (读取任意内核内存)

**修复建议**:
```c
// 增加额外的有效性检查
if (LOS_IsUserAddressRange(ptr, len)) {
    if (!IsMappedMemory(OsCurrProcessGet()->vmSpace, ptr)) {
        set_errno(EFAULT);
        return -EFAULT;
    }
}
```

### 2. 整数溢出风险

**证据**: `syscall/fs_syscall.c` (多处涉及 size_t 计算)

**问题**: 在计算缓冲区大小时可能发生整数溢出

**触发条件**: 恶意构造参数使 size + offset 溢出

**影响**: 缓冲区越界读写

**修复建议**:
```c
// 使用安全整数运算
if (len > SIZE_MAX - offset) {
    return -EINVAL;
}
size_t total = len + offset;
```

### 3. 路径遍历漏洞

**证据**: `fs/vfs/`

**问题**: 部分文件操作未充分验证路径中的 `..`

**触发条件**: 通过路径遍历访问受限文件

**影响**: 绕过沙箱访问敏感文件

**修复建议**:
```c
// 在 VFS 层规范化路径
char *normalized = NormalizePath(path);
if (strstr(normalized, "/..")) {
    return -EPERM;
}
```

### 4. 竞态条件 (TOCTOU)

**证据**: `fs_syscall.c` 中的文件操作

**问题**: 检查时间与使用时间之间的竞态 (Time-of-check to time-of-use)

**触发条件**:
1. 检查文件权限
2. 内核调度切换
3. 攻击者替换文件
4. 继续操作

**影响**: 权限绕过

**修复建议**:
```c
// 使用原子操作或文件描述符
int fd = openat(AT_FDCWD, path, O_NOFOLLOW);
if (fd < 0) return fd;
```

### 5. 资源耗尽 (DoS)

**证据**: `kernel/base/ipc/`, `kernel/base/mem/`

**问题**: 关键资源 (消息队列、内存) 没有全局限制

**触发条件**: 恶意程序持续申请资源

**影响**:
- 内存耗尽
- 文件描述符耗尽
- IPC 资源耗尽

**修复建议**:
```c
// 添加全局资源限制
static atomic_t g_queueCount = ATOMIC_INIT(0);
if (atomic_read(&g_queueCount) >= MAX_QUEUES) {
    return -ENOMEM;
}
atomic_inc(&g_queueCount);
```

### 6. 内核栈溢出风险

**证据**: `kernel/base/core/los_task.c`

**问题**: 大型局部变量可能导致栈溢出

**触发条件**: 任务栈设置过小且使用大型局部变量

**影响**: 内核崩溃

**修复建议**:
```c
// 避免在栈上分配大对象
static char largeBuffer[LARGE_SIZE];  // 使用静态分配
```

### 7. 能力检查缺失

**证据**: `security/cap/`

**问题**: 部分特权操作缺少 capability 检查

**触发条件**: 普通进程执行需要特权的操作

**影响**: 提权

**修复建议**:
```c
// 对特权操作添加 capability 检查
if (!CapCheck(CAP_SYS_ADMIN)) {
    return -EPERM;
}
```

## 信任边界

```
+------------------------+
|    User Space          | ←── 不可信边界
|------------------------|
|   Shell / Init         |
+------------------------+
         ↓↑ (syscall)
+------------------------+
|    Kernel Space        | ←── 信任边界
|   Kernel Code         |
+------------------------+
         ↓↑ (HAL/HDF)
+------------------------+
|    Hardware            |
+------------------------+
```

## 已有的安全机制

| 机制 | 位置 | 说明 |
|------|------|------|
| **用户态指针校验** | `syscall_pub.h` | `CHECK_ASPACE` 宏 |
| **地址空间隔离** | `kernel/base/vm/` | 用户/内核空间分离 |
| **Capability** | `security/cap/` | 细粒度权限控制 |
| **虚拟ID映射** | `security/vid/` | UID/GID 映射 |
| **栈保护** | `BUILD.gn` | `-fstack-protector-*` |
| **只读数据段** | 链接脚本 | `.rodata` 只读 |

## 安全建议优先级

| 优先级 | 问题 | 建议 |
|--------|------|------|
| **高** | 指针校验不完整 | 增加内存映射有效性检查 |
| **高** | 整数溢出 | 使用安全整数运算 |
| **中** | 路径遍历 | VFS 层路径规范化 |
| **中** | TOCTOU 竞态 | 使用原子文件操作 |
| **低** | 资源限制 | 添加全局资源计数 |
| **低** | 能力检查 | 补充特权操作检查 |

## 第三方依赖安全

| 依赖 | 版本 | 已知漏洞 | 备注 |
|------|------|----------|------|
| lwIP | - | 需关注 | 定期更新 |
| NuttX | - | 需关注 | 定期更新 |
| musl | - | 较少 | 安全审计较好 |

## 评审局限性

1. **未覆盖测试代码**: 本评审不引用任何测试代码
2. **静态分析为主**: 未进行动态 fuzz 测试
3. **依赖第三方**: lwIP/NuttX 的安全问题需查看其安全公告
4. **边界情况**: 可能存在未考虑到的边界条件

## 相关文档

- [架构说明](/02_Architecture.md)
- [内核模块详解](/04_Kernel_Modules.md)
- [系统调用接口](/05_Syscall_API.md)
