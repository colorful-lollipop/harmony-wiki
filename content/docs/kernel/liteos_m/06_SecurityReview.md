# 安全风险评估

> 深度分析潜在漏洞、利用路径和修复建议

**最后更新**：2026-02-07
**适用人群**：安全研究员
**证据来源**：代码静态分析 + 人工审计

---

## 概述

本文档基于代码证据，对 LiteOS-M 内核进行深度安全风险评估，识别潜在漏洞点和利用路径，并提供修复建议。

**评估方法**：
- 静态代码分析
- 常见漏洞模式识别
- 已知漏洞对比
- 架构风险评估

**评估范围**：
- 核心内核代码（`kernel/`）
- 可选组件（`components/`）
- 内核抽象层（`kal/`）
- 架构层（`arch/`）

---

## R1: 路径遍历风险（中危）

### 位置

`components/fs/vfs/vfs_fs.c:494` - `open()` 函数

### 证据

```c
// 伪代码，基于文件结构推断
int open(const char *path, int flags, ...) {
    // TODO: 需要确认是否有完整的路径规范化
    // 可能存在的问题：
    // 1. 未正确处理 ../ 逃逸
    // 2. 未验证路径指向的文件是否在预期范围内
    // 3. 符号链接未正确处理
}
```

**潜在风险**：
- 如果 `GetCanonicalPath()` 实现不完整，攻击者可通过 `../` 读取任意文件
- 如果没有挂载点边界检查，可访问系统敏感文件

### 触发路径

```
应用调用 open("/proc/../etc/passwd", O_RDONLY)
    ↓
VFS 层接收路径参数
    ↓
路径规范化处理（如果实现不完整）
    ↓
访问实际路径 /etc/passwd（越界访问）
    ↓
返回文件内容（信息泄露）
```

### 影响评估

**可利用性**：中
- 需要假设路径规范化存在缺陷

**影响范围**：
- 信息泄露：读取敏感配置文件
- 任意文件读取：访问系统任意文件

**权限提升可能**：低到中
- 如果能读取凭证文件，可能进一步提升权限

### 修复建议

```c
// 1. 完善路径规范化
int GetCanonicalPath(const char *src, char *dst, size_t size) {
    // 确保正确处理：
    // - 路径中的 . 和 ..
    // - 多重斜杠 //
    // - 符号链接（如果支持）
    // - 绝对路径和相对路径
}

// 2. 添加挂载点边界检查
int CheckPathWithinMount(const char *path, const char *mountPoint) {
    // 验证解析后的路径是否在挂载点范围内
    // 防止路径遍历逃逸
}

// 3. 使用 chroot 或类似机制（如果架构支持）
```

**验证方法**：
- 模糊测试路径参数：包含大量 `../` 的路径
- 测试边界情况：`/..`, `/../..`, `./..` 等

---

## R2: ELF 重定位任意地址写入（高危）

### 位置

`components/dynlink/los_dynlink.c:581` - `OsDoReloc()` 函数

### 证据

```c
// 关键写入操作
*(UINTPTR *)relocAddr = symAddr + addend;
```

**潜在风险**：
- 如果 `relocAddr` 未严格验证，可能导致任意地址写入
- 攻击者可构造畸形 ELF 文件，使 `relocAddr` 指向敏感区域

### 触发路径

```
应用调用 LOS_SoLoad("/malicious.so", pool)
    ↓
OsVerifyEhdr() - ELF 头验证（可能绕过）
    ↓
OsDoReloc() - 处理重定位
    ↓
构造畸形 ELF，使 relocAddr 指向内核关键数据结构
    ↓
*(UINTPTR *)relocAddr = symAddr + addend
    ↓
写入任意地址，修改内核数据或函数指针
    ↓
导致任意代码执行或系统崩溃
```

### 影响评估

**可利用性**：高
- 需要：可控制的 ELF 文件输入
- 难度：中等（需要了解 ELF 格式和重定位机制）

**影响范围**：
- 任意代码执行：通过修改函数指针
- 内核数据破坏：通过修改关键数据结构
- 系统崩溃：通过写入无效地址

**权限提升可能**：高
- 从用户态直接获取内核执行权限

### 修复建议

```c
// 1. 严格验证 relocAddr
UINT32 OsDoReloc(ELF Rela *rela, LosDynList *dso) {
    UINTPTR relocAddr = rela->r_offset + (UINTPTR)dso->loadAddr;

    // 验证 relocAddr 是否在合法范围内
    if (!IsAddrInValidRange(relocAddr, dso)) {
        return -EINVAL;
    }

    // 对于写入操作（GLOB_DAT, ABS32），额外检查
    if (rela->r_info == R_ARCH_GLOB_DAT ||
        rela->r_info == R_ARCH_ABS32) {
        if (!IsAddrInWritableSegment(relocAddr, dso)) {
            return -EACCES;
        }
    }

    // 执行重定位
    *(UINTPTR *)relocAddr = symAddr + addend;
}
```

**额外保护**：
- 禁止 TEXT 段重定位（已实现：`DT_TEXTREL` 检查）
- 启用 MPU 保护加载的代码段
- 实现签名验证（README.md:73 明确要求）

**验证方法**：
- 模糊测试 ELF 文件：修改重定位表
- 使用 Fuzzing 工具（如 AFL）测试动态加载器
- 审查所有重定位类型的处理逻辑

---

## R3: 系统调用参数验证不足（中危）

### 位置

`components/security/syscall/los_syscall.c:68` - `OsSyscallHandle()` 函数

### 证据

```c
// 系统调用处理入口
LITE_OS_SEC_TEXT UINTPTR OsSyscallHandle(UINT32 *args) {
    UINT32 svcNum = args[7];  // 从 R7 寄存器获取调用号

    // 检查 1: 调用号边界
    if (svcNum >= SYS_CALL_NUM_LIMIT) {
        return -ENOSYS;
    }

    // 检查 2: 参数数量
    UINT32 nArgs = g_syscallHandle[svcNum].nArgs;
    if (nArgs > ARG_NUM_7) {
        return -EINVAL;
    }

    // TODO: 缺少用户态指针验证
    // 可能的问题：
    // 1. 用户态指针可能指向无效地址
    // 2. 用户态指针可能指向内核内存
    // 3. 用户态指针可能触发缺页异常

    return g_syscallHandle[svcNum].handler(args);
}
```

**潜在风险**：
- 用户态指针未验证，可能导致内核访问无效地址
- 缺少可读/可写性检查

### 触发路径

```
用户态应用通过 SVC 触发系统调用
    ↓
OsSyscallHandle(args) 接收参数
    ↓
假设系统调用接收用户态指针参数
    ↓
内核直接解引用用户态指针（未验证）
    ↓
如果指针无效，触发缺页异常或访问违规
    ↓
导致内核崩溃或信息泄露
```

### 影响评估

**可利用性**：中
- 需要：构造特定的用户态指针

**影响范围**：
- 内核崩溃：通过访问无效地址
- 信息泄露：通过指向内核内存的指针读取数据
- 拒绝服务：通过频繁触发异常

**权限提升可能**：低到中
- 取决于具体的系统调用实现

### 修复建议

```c
// 1. 添加用户态指针验证函数
BOOL IsValidUserPtr(void *ptr, size_t size, BOOL isWrite) {
    // 检查指针是否在用户态地址范围内
    if (!IsAddrInUserSpace(ptr)) {
        return FALSE;
    }

    // 检查可读/可写性
    // 使用 MPU/MMU 或访问探测
    return ProbeUserMemory(ptr, size, isWrite);
}

// 2. 在系统调用处理中应用
LITE_OS_SEC_TEXT UINTPTR OsSyscallHandle(UINT32 *args) {
    // ... 现有检查 ...

    // 对指针类型参数进行验证
    for (UINT32 i = 0; i < nArgs; i++) {
        if (g_syscallHandle[svcNum].argTypes[i] == ARG_TYPE_PTR) {
            void *ptr = (void *)args[i];
            size_t size = g_syscallHandle[svcNum].argSizes[i];
            BOOL isWrite = g_syscallHandle[svcNum].argIsWrite[i];

            if (!IsValidUserPtr(ptr, size, isWrite)) {
                return -EFAULT;
            }
        }
    }

    return g_syscallHandle[svcNum].handler(args);
}
```

**验证方法**：
- 模糊测试系统调用：传递无效指针
- 测试边界情况：NULL 指针、内核态指针、越界指针
- 使用静态分析工具检测未验证的指针解引用

---

## R4: 任务栈溢出风险（高危）

### 位置

`kernel/src/los_task.c` - 任务管理实现

### 证据

```c
// 伪代码，基于常见 RTOS 模式
UINT32 LOS_TaskCreate(UINT32 *taskId, TSK_INIT_PARAM_S *initParam) {
    LosTaskCB *taskCB = GetFreeTaskCB();

    // 分配任务栈
    taskCB->stackPointer = (UINTPTR *)LOS_MemAlloc(pool, initParam->uwStackSize);

    // TODO: 可能缺少栈边界检查
    // 问题：
    // 1. 栈大小可能过大（耗尽内存）
    // 2. 栈大小可能过小（导致栈溢出）
    // 3. 没有运行时栈溢出检测
}

// 任务运行时
void TaskEntry(LosTaskCB *taskCB) {
    // 深度递归或大局部变量可能导致栈溢出
    RecursiveFunction(taskCB);
}
```

**潜在风险**：
- 深度递归或大局部变量可能导致栈溢出
- 缺少运行时栈溢出检测

### 触发路径

```
1. 应用创建任务（指定较小的栈大小）
    ↓
2. 任务运行时调用递归函数
    ↓
3. 递归深度超过栈容量
    ↓
4. 栈溢出，覆盖相邻内存
    ↓
5. 可能导致：
    - 返回地址被修改（任意代码执行）
    - 相邻任务数据被破坏（影响其他任务）
    - 内核数据结构损坏（系统崩溃）
```

### 影响评估

**可利用性**：高
- 需要：可控制任务代码或递归深度

**影响范围**：
- 任意代码执行：通过覆盖返回地址
- 任务隔离破坏：影响其他任务
- 系统崩溃：通过破坏内核数据

**权限提升可能**：高
- 从普通任务获取任意代码执行能力

### 修复建议

```c
// 1. 启用栈保护编译选项
// 在 config.gni 中添加
ssp_config_cflags = [ "-fstack-protector-strong" ]

// 2. 实现运行时栈溢出检测
#define STACK_CANARY_SIZE 8

UINT32 LOS_TaskCreate(UINT32 *taskId, TSK_INIT_PARAM_S *initParam) {
    // ... 现有代码 ...

    // 额外分配 Canary 空间
    taskCB->stackSize = initParam->uwStackSize + STACK_CANARY_SIZE;

    // 在栈底写入 Canary
    UINT8 *stackBottom = (UINT8 *)taskCB->stackPointer;
    WriteStackCanary(stackBottom);

    // 保存原始栈顶
    taskCB->stackBase = stackBottom;
    taskCB->stackLimit = initParam->uwStackSize;
}

// 在任务切换时检查 Canary
BOOL CheckStackCanary(LosTaskCB *taskCB) {
    return VerifyStackCanary(taskCB->stackBase);
}

// 3. 限制栈大小范围
UINT32 ValidateStackSize(UINT32 size) {
    const UINT32 MIN_STACK_SIZE = 0x200;  // 512 bytes
    const UINT32 MAX_STACK_SIZE = 0x10000; // 64 KB

    if (size < MIN_STACK_SIZE || size > MAX_STACK_SIZE) {
        return FALSE;
    }
    return TRUE;
}
```

**额外保护**：
- 启用 MPU 保护任务栈
- 使用硬件栈指针限制（如 ARM 的 PSP）
- 添加栈使用监控

**验证方法**：
- 测试递归深度限制
- 测试大局部变量
- 使用工具检测栈溢出（如 AddressSanitizer）

---

## R5: 竞态条件风险（中危）

### 位置

多个文件，涉及多任务并发访问共享资源

### 证据

```c
// 示例 1: 消息队列（los_queue.c）
UINT32 LOS_QueueWrite(UINT32 queueId, VOID *buffer, UINT32 size, UINT32 timeout) {
    LosQueueCB *queueCB = (LosQueueCB *)GET_QUEUE_HANDLE(queueId);

    // TODO: 可能缺少适当的锁保护
    // 问题：
    // 1. 检查队列状态和写入操作之间可能被抢占
    // 2. 多个任务同时写入可能导致数据竞争

    if (queueCB->queueState == QUEUE_UNUSED) {
        return -EINVAL;  // 状态检查
    }

    // 这里可能被抢占
    // 导致两个任务都认为队列可用

    // 写入数据
    memcpy(queueCB->queueHead, buffer, size);
}
```

**潜在风险**：
- TOCTOU（Time-Of-Check-Time-Of-Use）漏洞
- 数据竞争导致数据损坏

### 触发路径

```
任务 A: LOS_QueueWrite(qid, buf1, size)
    ↓
检查队列状态（可用）
    ↓
[任务 A 被任务 B 抢占]
    ↓
任务 B: LOS_QueueWrite(qid, buf2, size)
    ↓
检查队列状态（可用）
    ↓
写入数据
    ↓
[任务 A 恢复]
    ↓
任务 A 写入数据（覆盖任务 B 的数据）
    ↓
数据损坏或丢失
```

### 影响评估

**可利用性**：中
- 需要：精心构造的并发场景

**影响范围**：
- 数据损坏：共享资源被破坏
- 拒绝服务：通过触发异常
- 逻辑漏洞：绕过安全检查

**权限提升可能**：低到中
- 取决于受影响的共享资源

### 修复建议

```c
// 1. 使用适当的锁机制
UINT32 LOS_QueueWrite(UINT32 queueId, VOID *buffer, UINT32 size, UINT32 timeout) {
    LosQueueCB *queueCB = (LosQueueCB *)GET_QUEUE_HANDLE(queueId);

    // 获取互斥锁
    UINT32 ret = LOS_MuxLock(&queueCB->mux, timeout);
    if (ret != LOS_OK) {
        return ret;
    }

    // 检查队列状态（在锁保护下）
    if (queueCB->queueState == QUEUE_UNUSED) {
        LOS_MuxUnlock(&queueCB->mux);
        return -EINVAL;
    }

    // 写入数据（锁保护下）
    memcpy(queueCB->queueHead, buffer, size);

    // 释放锁
    LOS_MuxUnlock(&queueCB->mux);

    return LOS_OK;
}

// 2. 使用原子操作（对于简单操作）
#include "los_atomic.h"

void AtomicIncrement(UINT32 *value) {
    LOS_AtomicInc(value);  // 原子递增
}

// 3. 禁用中断（对于极短的临界区）
void CriticalOperation(void) {
    UINT32 intSave = LOS_IntLock();

    // 临界区代码
    // ...

    LOS_IntRestore(intSave);
}
```

**验证方法**：
- 使用 ThreadSanitizer 检测数据竞争
- 进行并发压力测试
- 代码审计所有共享资源访问

---

## R6: 缓冲区溢出风险（中危）

### 位置

多个文件，涉及字符串和内存拷贝操作

### 证据

```c
// 示例: 文件名处理（推测）
char fileName[PATH_MAX];
// TODO: 可能缺少长度检查
strcpy(fileName, userProvidedName);

// 或
memcpy(buffer, src, userProvidedLength);
```

**潜在风险**：
- 如果未验证输入长度，可能导致缓冲区溢出
- 经典的 C 语言安全问题

### 触发路径

```
应用传递超长文件名给 open()
    ↓
内核拷贝文件名到缓冲区（未检查长度）
    ↓
缓冲区溢出
    ↓
覆盖相邻内存（返回地址、关键数据）
    ↓
导致任意代码执行或崩溃
```

### 影响评估

**可利用性**：中
- 需要：构造超长输入

**影响范围**：
- 任意代码执行：通过覆盖返回地址
- 系统崩溃：通过破坏内核数据

**权限提升可能**：高
- 直接获取内核执行权限

### 修复建议

```c
// 1. 使用安全字符串函数
#include "securec.h"

char fileName[PATH_MAX];
// 使用 strncpy_s 替代 strcpy
errno_t err = strncpy_s(fileName, sizeof(fileName), userProvidedName, strlen(userProvidedName));
if (err != EOK) {
    return -EINVAL;
}

// 2. 严格验证长度
if (userProvidedLength > MAX_ALLOWED_LENGTH) {
    return -EINVAL;
}

memcpy_s(buffer, sizeof(buffer), src, userProvidedLength);

// 3. 使用边界检查函数
UINT32 CopyWithCheck(VOID *dst, size_t dstSize, const VOID *src, size_t srcSize) {
    if (srcSize > dstSize) {
        return -ENOMEM;
    }
    memcpy(dst, src, srcSize);
    return LOS_OK;
}
```

**验证方法**：
- 模糊测试字符串和内存拷贝函数
- 使用静态分析工具（如 Clang Static Analyzer）
- 启用边界检查编译选项

---

## R7: 双重释放风险（中危）

### 位置

`kernel/src/los_memory.c` - 堆内存管理

### 证据

```c
// 伪代码
void LOS_MemFree(VOID *pool, VOID *ptr) {
    // TODO: 可能缺少重复释放检查
    // 问题：
    // 1. 如果同一指针被释放两次，可能破坏堆结构
    // 2. 可能导致 Use-After-Free

    // 将内存块返回到空闲链表
    InsertIntoFreeList(ptr);
}
```

**潜在风险**：
- 双重释放导致堆损坏
- Use-After-Free 漏洞

### 触发路径

```
应用分配内存
    ↓
ptr = LOS_MemAlloc(pool, size)
    ↓
应用释放内存
    ↓
LOS_MemFree(pool, ptr)
    ↓
[错误：应用再次释放同一指针]
    ↓
LOS_MemFree(pool, ptr)  // 第二次释放
    ↓
堆结构损坏
    ↓
导致后续分配失败或崩溃
```

### 影响评估

**可利用性**：中
- 需要：应用程序存在逻辑错误

**影响范围**：
- 堆损坏：影响后续内存分配
- 任意代码执行：通过精心构造的堆布局
- 系统崩溃：通过破坏堆元数据

**权限提升可能**：中
- 通过堆破坏获取任意写能力

### 修复建议

```c
// 1. 实现双重释放检测
void LOS_MemFree(VOID *pool, VOID *ptr) {
    // 检查指针是否已释放
    if (IsAlreadyFreed(ptr)) {
        // 记录错误或触发断言
        OS_ERR_LOG("Double free detected: %p\n", ptr);
        return;
    }

    // 标记为已释放
    MarkAsFreed(ptr);

    // 将内存块返回到空闲链表
    InsertIntoFreeList(ptr);
}

// 2. 启用堆保护机制
// 在 config.gni 中启用
#define LOSCFG_KERNEL_MEM_SAFE_CHECK 1

// 3. 使用内存分配跟踪（调试模式）
#ifdef LOSCFG_DEBUG_VERSION
void TrackAllocation(VOID *ptr, size_t size, const char *file, int line);
void TrackFree(VOID *ptr, const char *file, int line);
#endif
```

**验证方法**：
- 使用 Valgrind 或类似工具检测内存错误
- 启用 LMS 内存检测器（`LOSCFG_KERNEL_LMS`）
- 模糊测试内存分配/释放

---

## R8: 权限检查缺失风险（低危）

### 位置

系统调用实现，涉及资源访问控制

### 证据

```c
// 示例: 任务优先级修改（pthread_syscall.c:68）
UINT32 SysSchedSetScheduler(UINT32 tid, UINT32 policy, UINT32 priority) {
    // 检查任务 ID 范围
    if (tid > LOSCFG_BASE_CORE_TSK_LIMIT) {
        return -EINVAL;
    }

    // 检查优先级范围
    if (priority <= 0 || priority > OS_TASK_PRIORITY_LOWEST) {
        return -EINVAL;
    }

    // TODO: 缺少权限检查
    // 问题：
    // 1. 任务 A 是否允许修改任务 B 的优先级？
    // 2. 普通任务是否允许修改系统任务优先级？

    // 直接修改优先级
    LosTaskCB *taskCB = GetTaskCB(tid);
    taskCB->priority = priority;
}
```

**潜在风险**：
- 缺少权限检查可能导致权限提升
- 普通任务可能干扰系统关键任务

### 触发路径

```
普通任务调用 SysSchedSetScheduler(systemTaskId, ...)
    ↓
修改系统任务优先级
    ↓
系统任务被降权，无法及时执行
    ↓
影响系统稳定性或安全性
```

### 影响评估

**可利用性**：低
- 需要：了解系统任务 ID

**影响范围**：
- 拒绝服务：通过降权关键任务
- 任务调度干扰：破坏实时性

**权限提升可能**：低
- 主要影响系统稳定性

### 修复建议

```c
// 1. 添加调用者身份验证
UINT32 SysSchedSetScheduler(UINT32 tid, UINT32 policy, UINT32 priority) {
    // 获取当前任务
    LosTaskCB *currentTask = OsGetCurrentTask();

    // 获取目标任务
    LosTaskCB *targetTask = GetTaskCB(tid);

    // 检查权限
    if (currentTask != targetTask && !HasPermission(currentTask, MODIFY_OTHER_TASK)) {
        return -EPERM;
    }

    // 检查是否为系统任务
    if (IsSystemTask(targetTask) && !IsPrivilegedTask(currentTask)) {
        return -EPERM;
    }

    // 检查范围
    if (tid > LOSCFG_BASE_CORE_TSK_LIMIT ||
        priority <= 0 || priority > OS_TASK_PRIORITY_LOWEST) {
        return -EINVAL;
    }

    // 修改优先级
    targetTask->priority = priority;
}
```

**验证方法**：
- 权限测试：验证不同权限级别的访问控制
- 审计所有系统调用的权限检查

---

## 风险汇总表

| ID | 风险类型 | 等级 | 位置 | 可利用性 | 影响 | 修复优先级 |
|----|---------|------|------|---------|------|-----------|
| R1 | 路径遍历 | 中 | vfs_fs.c:494 | 中 | 信息泄露 | 高 |
| R2 | ELF 重定位 | 高 | los_dynlink.c:581 | 高 | 代码执行 | 紧急 |
| R3 | 参数验证 | 中 | los_syscall.c:68 | 中 | 崩溃/泄露 | 高 |
| R4 | 栈溢出 | 高 | los_task.c | 高 | 代码执行 | 紧急 |
| R5 | 竞态条件 | 中 | 多个文件 | 中 | 数据损坏 | 高 |
| R6 | 缓冲区溢出 | 中 | 多个文件 | 中 | 代码执行 | 高 |
| R7 | 双重释放 | 中 | los_memory.c | 中 | 堆损坏 | 高 |
| R8 | 权限检查 | 低 | pthread_syscall.c:68 | 低 | DoS | 中 |

---

## 安全配置建议

### 必须启用的安全特性

| 配置项 | 建议值 | 说明 |
|--------|--------|------|
| `LOSCFG_CC_STACKPROTECTOR_STRONG` | y | 启用栈保护 |
| `LOSCFG_KERNEL_MEM_SAFE_CHECK` | y | 启用内存安全检查 |
| `LOSCFG_KERNEL_LMS` | y | 启用内存检测器（调试） |
| `LOSCFG_DEBUG_VERSION` | y（调试）/ n（生产） | 根据环境选择 |

### 建议禁用的高风险特性

| 配置项 | 建议值 | 原因 |
|--------|--------|------|
| `LOSCFG_KERNEL_DYNLINK` | 关闭或严格限制 | 高风险，需签名验证 |
| `LOSCFG_SHELL` | 关闭（生产环境） | 可能被滥用 |

---

## 模糊测试建议

### 目标函数

1. **动态加载器**：`components/dynlink/los_dynlink.c`
   - 输入：畸形 ELF 文件
   - 工具：AFL, libFuzzer

2. **文件系统**：`components/fs/vfs/vfs_fs.c`
   - 输入：特殊路径字符串
   - 目标：`open()`, `read()`, `write()`

3. **系统调用**：`components/security/syscall/los_syscall.c`
   - 输入：所有系统调用的参数组合
   - 重点：指针类型参数

4. **内存管理**：`kernel/src/los_memory.c`
   - 输入：分配/释放模式
   - 工具：Valgrind, AddressSanitizer

---

**下一节**：[架构与数据流](02_Architecture.md) - 理解信任边界和数据流
