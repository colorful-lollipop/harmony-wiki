# UniProton 攻击面分析

**文档版本**: 1.0  
**更新日期**: 2026-02-07  
**分析范围**: UniProton RTOS 内核 (kernel/uniproton)

---

## 1. 概述

### 1.1 系统安全模型

UniProton 是一个**裸机实时操作系统 (RTOS)**，具有以下安全特征：

```
┌─────────────────────────────────────────────────────────┐
│                   UniProton 安全模型                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  信任边界:                                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │  硬件层 (唯一信任根)                              │   │
│  │  ├── MPU/MMU (可选)                              │   │
│  │  └── 特权模式                                     │   │
│  ├─────────────────────────────────────────────────┤   │
│  │  UniProton 内核 (完全特权)                       │   │
│  │  ├── 任务调度器                                   │   │
│  │  ├── 中断处理                                     │   │
│  │  ├── 内存管理                                     │   │
│  │  └── IPC 机制                                     │   │
│  ├─────────────────────────────────────────────────┤   │
│  │  用户任务 (共享地址空间)                          │   │
│  │  ├── 任务 A (优先级 0-62)                        │   │
│  │  ├── 任务 B (优先级 0-62)                        │   │
│  │  └── IDLE (优先级 63)                           │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  安全依赖:                                               │
│  - 硬件 MPU/MMU 提供内存隔离 (可选)                     │
│  - 中断优先级控制硬件                                   │
│  - 软件实现临界区保护                                   │
└─────────────────────────────────────────────────────────┘
```

### 1.2 攻击面分类

| 攻击面类别 | 暴露接口 | 风险等级 |
|-----------|---------|---------|
| **内存管理** | PRT_MemAlloc/PRT_MemFree | 🔴 高 |
| **任务管理** | PRT_TaskCreate/PRT_TaskDelete | 🟠 中 |
| **IPC 机制** | 信号量/队列/事件/读写锁 | 🟠 中 |
| **中断处理** | PRT_HwiCreate/PRT_HwiEnable | 🟡 低-中 |
| **异常处理** | PRT_ExcRegHook | 🟡 低 |

---

## 2. 外部输入清单

### 2.1 内存分配 API

**输入点**: `PRT_MemAlloc(U32 mid, U8 ptNo, U32 size)`

**位置**: `src/mem/prt_mem.c:17`

| 参数 | 类型 | 攻击向量 | 当前校验 | 风险 |
|------|------|---------|---------|------|
| `mid` | U32 | 伪造模块ID | ❌ 无校验 | 🟢 低 |
| `ptNo` | U8 | 越界分区号 | ❌ 无校验 (被忽略) | 🟢 低 |
| `size` | U32 | 整数溢出/超大分配 | ✅ 零检查+溢出检查 | 🟡 中 |

**证据**:
```c
// src/mem/fsc/prt_fscmem.c:80-91
if (size == 0) {
    OS_REPORT_ERROR(OS_ERRNO_MEM_ALLOC_SIZE_ZERO);
    return NULL;
}
allocSize = ALIGN(size, OS_FSC_MEM_SIZE_ALIGN) + ...;
if ((allocSize < size) || allocSize >= OS_FSC_MEM_MAXVAL...) {
    OS_REPORT_ERROR(OS_ERRNO_MEM_ALLOC_SIZETOOLARGE);
    return NULL;
}
```

**风险分析**:
- `mid` 和 `ptNo` 参数虽传入但被下层忽略，可能导致调用者误判模块隔离存在
- 整数溢出检查存在，可防止分配大小绕过

### 2.2 内存释放 API

**输入点**: `PRT_MemFree(U32 mid, void *addr)`

**位置**: `src/mem/prt_mem.c:41`

| 参数 | 类型 | 攻击向量 | 当前校验 | 风险 |
|------|------|---------|---------|------|
| `mid` | U32 | 伪造模块ID | ❌ 被忽略 | 🟢 低 |
| `addr` | void* | 任意地址/野指针/已释放 | ✅ NULL检查+魔术字验证 | 🟠 中 |

**证据** (src/mem/fsc/prt_fscmem.c:141-155):
```c
if (addr == NULL) {
    return OS_ERRNO_MEM_FREE_ADDR_INVALID;
}

currBlk = (struct TagFscMemCtrl *)OsMemGetHeadAddr((uintptr_t)addr);

// Double Free 检测
if ((currBlk->next != OS_FSC_MEM_MAGIC_USED) || (currBlk->size == 0)) {
    return OS_ERRNO_MEM_FREE_SH_DAMAGED;
}

// 内存越界检测
blkTailMagic = (U32 *)((uintptr_t)currBlk + blkSize - OS_FSC_MEM_TAIL_SIZE);
if (*blkTailMagic != OS_FSC_MEM_TAIL_MAGIC) {
    return OS_ERRNO_MEM_OVERWRITE;
}
```

**风险分析**:
- ✅ NULL 指针防护
- ✅ Double Free 检测 (魔术字 `0x5a5aa5a5`)
- ✅ 缓冲区溢出检测 (尾魔术字 `0xABCDDCBA`)
- ⚠️ 无法检测所有非法地址 (只能通过魔术字间接检测)

### 2.3 信号量 API

**输入点**: `PRT_SemCreate/PRT_SemPend/PRT_SemPost`

**位置**: `src/core/ipc/sem/prt_sem.c`, `prt_sem_init.c`

| API | 参数 | 攻击向量 | 当前校验 | 风险 |
|-----|------|---------|---------|------|
| `Create` | count | 超大计数导致溢出 | ✅ `count > OS_SEM_COUNT_MAX` | 🟢 低 |
| `Pend` | semHandle | 伪造句柄 | ✅ 边界检查 | 🟢 低 |
| `Pend` | timeout | 非法超时值 | ⚠️ 部分检查 | 🟢 低 |
| `Post` | semHandle | 伪造句柄 | ✅ 边界检查 | 🟢 低 |

**证据** (src/core/ipc/sem/prt_sem.c:181-196):
```c
// 句柄边界检查
if (semHandle >= (SemHandle)g_maxSem) {
    return OS_ERRNO_SEM_INVALID;
}

// 状态检查
if (semPended->semStat == OS_SEM_UNUSED) {
    OsIntRestore(intSave);
    return OS_ERRNO_SEM_INVALID;
}

// 中断上下文检查
if (OS_INT_ACTIVE) {
    OsIntRestore(intSave);
    return OS_ERRNO_SEM_PEND_INTERR;
}
```

**证据** (src/core/ipc/sem/prt_sem.c:35-42):
```c
// 互斥信号量持有者校验
if (OS_INT_ACTIVE) {
    return OS_ERRNO_SEM_MUTEX_POST_INTERR;
}
if (semPosted->semOwner != RUNNING_TASK->taskPid) {
    return OS_ERRNO_SEM_MUTEX_NOT_OWNER_POST;
}
```

**风险分析**:
- ✅ 信号量句柄边界检查完善
- ✅ 中断上下文安全检查
- ✅ 互斥信号量持有者验证
- ⚠️ 信号量数量无全局限制 (可能导致资源耗尽)

### 2.4 消息队列 API

**输入点**: `PRT_QueueCreate/PRT_QueueRead/PRT_QueueWrite`

**位置**: `src/core/ipc/queue/prt_queue.c`, `prt_queue_init.c`

| API | 参数 | 攻击向量 | 当前校验 | 风险 |
|-----|------|---------|---------|------|
| `Create` | nodeNum/nodeSize | 超大值导致溢出 | ✅ 零检查+溢出检查 | 🟢 低 |
| `Read` | queueId | 伪造ID | ✅ 边界检查 | 🟢 低 |
| `Read` | bufferAddr | 无效缓冲区 | ✅ NULL检查 | 🟢 低 |
| `Read` | len | 缓冲区大小 | ✅ 长度限制+截断 | 🟡 中 |
| `Write` | queueId | 伪造ID | ✅ 边界检查 | 🟢 低 |
| `Write` | bufferAddr | 无效地址 | ✅ NULL检查 | 🟢 低 |
| `Write` | bufferSize | 超大消息 | ✅ 节点大小限制 | 🟢 低 |

**证据** (src/core/ipc/queue/prt_queue.c:283-286):
```c
// 写入大小限制
if (bufferSize > (queueCb->nodeSize - OS_QUEUE_NODE_HEAD_LEN)) {
    ret = OS_ERRNO_QUEUE_SIZE_TOO_BIG;
    goto QUEUE_END;
}

// 安全拷贝
if (memcpy_s((void *)queueNode->buf, (queueCb->nodeSize - OS_QUEUE_NODE_HEAD_LEN),
             (void *)bufferAddr, bufferSize) != EOK) {
    OS_GOTO_SYS_ERROR1();
}
```

**风险分析**:
- ✅ 队列消息大小严格限制
- ✅ 使用 `memcpy_s` 安全拷贝
- ✅ 读取时自动截断到实际大小
- ⚠️ 队列总数无全局限制

### 2.5 任务管理 API

**输入点**: `PRT_TaskCreate`

**位置**: `src/core/kernel/task/prt_task_init.c:359`

| 参数 | 类型 | 攻击向量 | 当前校验 | 风险 |
|------|------|---------|---------|------|
| `initParam` | struct* | 恶意配置 | ✅ 全面校验 | 🟡 中 |
| `stackAddr` | uintptr_t | 无效栈地址 | ✅ 对齐+溢出检查 | 🟡 中 |
| `stackSize` | U32 | 过小/过大 | ✅ 最小值+对齐检查 | 🟡 中 |
| `priority` | TskPrior | 非法优先级 | ✅ 范围检查 (0-63) | 🟢 低 |
| `taskEntry` | func* | 无效入口 | ✅ NULL检查 | 🟢 低 |

**证据** (src/core/kernel/task/prt_task_init.c:175-221):
```c
// 入口函数非空检查
if (initParam->taskEntry == NULL) {
    return OS_ERRNO_TSK_ENTRY_NULL;
}

// 栈大小16字节对齐检查
if (initParam->stackSize != (initParam->stackSize & OS_TSK_STACK_SIZE_ALIGN))

// 最小栈大小检查
if (initParam->stackSize < OS_TSK_MIN_STACK_SIZE)

// 栈地址溢出检查
if ((initParam->stackAddr + initParam->stackSize) < initParam->stackAddr)

// 优先级范围检查
if (initParam->priority >= OS_TSK_PRIORITY_LOWEST)
```

**风险分析**:
- ✅ 全面的参数校验
- ⚠️ 栈地址仅检查对齐，不验证内存区域有效性
- ⚠️ 任务数量无全局配额限制

### 2.6 中断管理 API

**输入点**: `PRT_HwiCreate/PRT_HwiSetAttr`

**位置**: `src/core/kernel/irq/prt_irq.c:344`

| API | 参数 | 攻击向量 | 当前校验 | 风险 |
|-----|------|---------|---------|------|
| `HwiCreate` | hwiNum | 非法中断号 | ✅ 边界检查 | 🟢 低 |
| `HwiCreate` | handler | 无效处理函数 | ✅ NULL检查 | 🟢 低 |
| `HwiSetAttr` | hwiNum | 非法中断号 | ✅ 边界检查 | 🟢 低 |
| `HwiSetAttr` | hwiPrio | 非法优先级 | ✅ 范围检查 | 🟢 低 |

**证据** (src/core/kernel/irq/prt_irq.c:350-354):
```c
if (OS_HWI_NUM_CHECK(hwiNum)) {
    return OS_ERRNO_HWI_NUM_INVALID;
}

if (handler == NULL) {
    return OS_ERRNO_HWI_PROC_FUNC_NULL;
}
```

**风险分析**:
- ✅ 中断号有效性检查
- ✅ 处理函数非空检查
- ✅ 临界区保护 (关中断)
- ⚠️ 处理函数地址不验证是否在有效代码区

---

## 3. 敏感操作清单

### 3.1 特权操作

| 操作 | 接口 | 权限要求 | 当前保护 |
|------|------|---------|---------|
| 关中断 | `PRT_HwiLock()` | 特权模式 | 无条件执行 |
| 开中断 | `PRT_HwiUnLock()` | 特权模式 | 无条件执行 |
| 锁调度 | `PRT_TaskLock()` | 任务上下文 | 无条件执行 |
| 解锁调度 | `PRT_TaskUnlock()` | 任务上下文 | 检查嵌套计数 |
| 复位系统 | `PRT_SysReboot()` | 特权模式 | 无条件执行 |

### 3.2 资源管理操作

| 操作 | 接口 | 安全风险 | 当前保护 |
|------|------|---------|---------|
| 创建任务 | `PRT_TaskCreate` | 资源耗尽 | 数量限制（配置） |
| 删除任务 | `PRT_TaskDelete` | 删除IDLE | 禁止删除IDLE |
| 挂起任务 | `PRT_TaskSuspend` | 挂起IDLE | 禁止挂起IDLE |
| 创建中断 | `PRT_HwiCreate` | 中断风暴 | 数量限制 |
| 删除中断 | `PRT_HwiDelete` | 系统中断 | 禁止删除系统中断 |

### 3.3 内存敏感操作

| 操作 | 位置 | 风险 | 保护机制 |
|------|------|------|---------|
| 内存分配 | `OsFscMemAllocInner` | 堆溢出 | 边界检查 |
| 内存释放 | `OsFscMemFree` | Double Free | 魔术字检测 |
| 内存拷贝 | `memcpy_s` | 缓冲区溢出 | 长度检查 |
| 内存清零 | `memset_s` | 信息泄露 | 长度检查 |

---

## 4. 信任边界分析

### 4.1 边界图

```
┌────────────────────────────────────────────────────────────┐
│                     UniProton 信任边界                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   用户任务   │    │   用户任务   │    │   用户任务   │     │
│  │   (不信任)   │◄──►│   (不信任)   │    │   (不信任)   │     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘     │
│         │                  │                  │            │
│         ▼                  ▼                  ▼            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              IPC 机制 (信号量/队列/事件)              │  │
│  │  ⚠️ 边界: 无内存隔离，共享内核地址空间                 │  │
│  └─────────────────────────────────────────────────────┘  │
│                          │                                │
│                          ▼                                │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              UniProton 内核 (特权模式)                │  │
│  │  ✅ 边界: 特权指令保护 (通过硬件实现)                  │  │
│  └─────────────────────────────────────────────────────┘  │
│                          │                                │
│                          ▼                                │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              硬件抽象层 (Arch/驱动)                   │  │
│  │  ⚠️ 边界: 直接访问硬件寄存器                          │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### 4.2 边界跨越点

| 边界跨越 | 机制 | 验证 | 风险 |
|---------|------|------|------|
| 用户任务 → IPC | 函数调用 | 参数校验 | 缓冲区溢出 |
| 用户任务 → 内核 | 系统调用 | 中断上下文检查 | 权限提升 |
| 中断 → 内核 | 异常入口 | 中断号检查 | 中断风暴 |
| 内核 → 硬件 | 寄存器访问 | 无 | 硬件损坏 |

---

## 5. 攻击向量汇总

### 5.1 内存损坏攻击

**向量**: 利用内存操作漏洞破坏内核数据结构

| 攻击方式 | 目标 | 可行性 | 缓解措施 |
|---------|------|--------|---------|
| 堆溢出 | 堆管理器 | 🟡 中 | 魔术字检测、大小检查 |
| Use-After-Free | 已释放内存 | 🟡 中 | 清零控制头 (memset_s) |
| Double Free | 空闲链表 | 🟢 低 | 魔术字验证 |
| 缓冲区溢出 | 栈/堆缓冲区 | 🟡 中 | 边界检查、编译器保护 |

### 5.2 拒绝服务攻击

**向量**: 耗尽系统资源导致服务不可用

| 攻击方式 | 目标 | 可行性 | 缓解措施 |
|---------|------|--------|---------|
| 任务炸弹 | 任务控制块 | 🟠 中 | 任务数量限制 |
| 内存耗尽 | 堆内存 | 🟠 中 | 内存配额 |
| 信号量耗尽 | 信号量对象 | 🟠 中 | 信号量数量限制 |
| 中断风暴 | 中断处理 | 🟡 中 | 中断优先级、防抖 |
| 调度锁滥用 | 调度器 | 🟢 低 | 锁嵌套检查 |

### 5.3 权限提升攻击

**向量**: 获取更高特权级别

| 攻击方式 | 目标 | 可行性 | 缓解措施 |
|---------|------|--------|---------|
| 篡改 TCB | 任务控制块 | 🟡 中 | 魔术字、边界检查 |
| 中断劫持 | 中断向量表 | 🟢 低 | 只读映射 |
| Hook 注入 | 钩子函数 | 🟡 中 | 函数指针验证 |

### 5.4 信息泄露攻击

**向量**: 提取敏感信息

| 攻击方式 | 目标 | 可行性 | 缓解措施 |
|---------|------|--------|---------|
| 内存读取 | 其他任务内存 | 🟠 中 | MPU隔离 (可选) |
| 栈数据残留 | 任务栈 | 🟡 中 | 栈初始化清零 |
| 错误信息泄露 | 错误码 | 🟢 低 | 错误码抽象 |

---

## 6. 风险矩阵

| 攻击面 | 利用难度 | 影响程度 | 风险等级 | 优先级 |
|--------|---------|---------|---------|--------|
| 内存 Double Free | 中 | 高 | 🟠 **中** | P1 |
| 内存 UAF | 中 | 高 | 🟠 **中** | P1 |
| 资源耗尽 (DoS) | 低 | 中 | 🟠 **中** | P2 |
| 任务管理绕过 | 高 | 中 | 🟡 **低-中** | P3 |
| 中断注入 | 高 | 高 | 🟡 **低-中** | P3 |
| 信息泄露 | 中 | 低 | 🟢 **低** | P4 |

---

## 7. 建议措施

### 7.1 高优先级 (立即实施)

1. **添加资源配额限制**
   - 任务数量上限
   - 信号量/队列全局限制
   - 内存分区配额

2. **强化内存释放检查**
   - 验证地址范围有效性
   - 检查地址对齐
   - 增加额外的安全检查

3. **默认启用栈保护**
   ```gn
   # BUILD.gn 修改建议
   cflags += [ "-fstack-protector-strong" ]  # 默认启用
   ```

### 7.2 中优先级 (3-6个月)

1. **MPU 内存隔离** (如硬件支持)
   - 任务栈隔离
   - 内核数据保护
   - 只读代码段

2. **增强调试信息控制**
   - 发布版本剥离符号
   - 错误码抽象化
   - 日志分级

3. **中断安全检查强化**
   - 处理函数地址验证
   - 中断频率限制

### 7.3 低优先级 (6-12个月)

1. **模糊测试框架**
   - API 模糊测试
   - 边界条件测试

2. **形式化验证**
   - 关键模块验证
   - 状态机验证

---

## 8. 相关文档

- [安全风险评估](./06_Security_Review.md) - 详细安全分析
- [架构设计](./04_Architecture.md) - 系统架构说明
- [API 参考](./03_API_Reference.md) - 接口详细文档

---

*本文档基于 UniProton 源码静态分析生成*
