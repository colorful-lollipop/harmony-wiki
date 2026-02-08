# UniProton 安全风险评审

## 评审范围

本评审基于 UniProton 内核源码进行分析，范围包括：

| 模块 | 评审状态 | 说明 |
|------|----------|------|
| 任务管理 | ✅ 已覆盖 | `src/core/kernel/task/` |
| IPC 机制 | ✅ 已覆盖 | `src/core/ipc/` |
| 内存管理 | ✅ 已覆盖 | `src/mem/` |
| 中断处理 | ✅ 已覆盖 | `src/core/kernel/irq/` |
| 安全模块 | ✅ 已覆盖 | `src/security/` |
| 构建系统 | ✅ 已覆盖 | `BUILD.gn`, `uniproton.gni` |

**排除范围**:
- 测试代码 (`**/test/`, `**/tests/`)
- 上层框架 (N-API, JavaScript 绑定 - 本内核无此模块)
- 硬件驱动 (由 HDF 提供)

---

## 安全模型概述

UniProton 是**裸机 RTOS 内核**，安全模型特点：

| 特性 | 现状 | 说明 |
|------|------|------|
| 内存隔离 | ⚠️ 有限 | 单地址空间，无 MMU/MPU 隔离 |
| 权限模型 | ❌ 无 | 无用户/进程权限框架 |
| 身份认证 | ❌ 无 | 无身份概念 |
| 签名验证 | ❌ 无 | 无固件签名机制 |
| 栈保护 | ✅ 有 | Canary 随机化 |
| 内存安全 | ✅ 有 | SecureC 库 |

---

## 攻击面分析

### 1. API 输入攻击

#### 风险点 1: 内存分配参数校验

**风险等级**: 中

**位置**: `src/mem/prt_mem.c:17`, `src/mem/fsc/prt_fscmem.c:69`

**证据**: 

```c
// PRT_MemAlloc 仅做中断保护，未校验参数
OS_SEC_TEXT void *PRT_MemAlloc(U32 mid, U8 ptNo, U32 size)
{
    void *addr;
    uintptr_t intSave;

    intSave = PRT_HwiLock();
    addr = g_memArithAPI.alloc(mid, ptNo, size);  // mid 和 ptNo 被忽略
    PRT_HwiRestore(intSave);
    return addr;
}

// OsFscMemAllocInner 中的校验 (prt_fscmem.c:80-91)
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

**分析**:
- ✅ `size` 参数有零检查和整数溢出检查
- ❌ `mid` (模块号) 被完全忽略，无模块统计功能
- ❌ `ptNo` (分区号) 被完全忽略，无分区隔离功能

**影响**: 调用者可能误判模块隔离存在，导致安全问题

**建议**:
- 实现模块号统计功能，或明确文档说明不支持
- 添加分区号有效性检查

---

#### 风险点 2: 信号量 API 参数校验

**风险等级**: 低

**位置**: `src/core/ipc/sem/prt_sem.c:174`

**证据**:

```c
OS_SEC_L0_TEXT U32 PRT_SemPend(SemHandle semHandle, U32 timeout)
{
    // 行 181-183: 句柄边界检查
    if (semHandle >= (SemHandle)g_maxSem) {
        return OS_ERRNO_SEM_INVALID;
    }
    
    intSave = OsIntLock();
    
    // 行 188-191: 信号量状态检查
    if (semPended->semStat == OS_SEM_UNUSED) {
        OsIntRestore(intSave);
        return OS_ERRNO_SEM_INVALID;
    }
    
    // 行 193-196: 中断上下文检查
    if (OS_INT_ACTIVE) {
        OsIntRestore(intSave);
        return OS_ERRNO_SEM_PEND_INTERR;
    }
    // ...
}
```

**分析**:
- ✅ 信号量句柄边界检查
- ✅ 信号量状态检查 (UNUSED)
- ✅ 中断上下文安全检查
- ✅ 任务锁检查 (OsSemPendParaCheck)

**结论**: 信号量 API 参数校验完善

---

#### 风险点 3: 消息队列缓冲区校验

**风险等级**: 低

**位置**: `src/core/ipc/queue/prt_queue.c:283`

**证据**:

```c
// 行 283-286: 写入大小限制
if (bufferSize > (queueCb->nodeSize - OS_QUEUE_NODE_HEAD_LEN)) {
    ret = OS_ERRNO_QUEUE_SIZE_TOO_BIG;
    goto QUEUE_END;
}

// 行 244-247: 安全拷贝
if (memcpy_s((void *)queueNode->buf, (queueCb->nodeSize - OS_QUEUE_NODE_HEAD_LEN),
             (void *)bufferAddr, bufferSize) != EOK) {
    OS_GOTO_SYS_ERROR1();
}
```

**分析**:
- ✅ 消息大小严格限制
- ✅ 使用 `memcpy_s` 安全拷贝 (SecureC 库)
- ✅ 读取时自动截断到实际大小

**结论**: 消息队列缓冲区保护完善

---

### 2. 栈溢出攻击

#### 风险点 2: 栈保护依赖配置

**风险等级**: 中

**证据**: `BUILD.gn` 第 118-134 行

```gn
config("ssp_config") {
  cflags = []
  if (defined(CC_STACKPROTECTOR_ALL)) {
    cflags += [ "-fstack-protector-all" ]
  } else if (defined(CC_STACKPROTECTOR_STRONG)) {
    cflags += [ "-fstack-protector-strong" ]
  } else if (defined(CC_STACKPROTECTOR)) {
    cflags += [ "-fstack-protector", "--param", "ssp-buffer-size=4" ]
  } else {
    cflags += [ "-fno-stack-protector" ]  // 可能被禁用!
  }
}
```

**影响**: 若栈保护被禁用 (默认配置可能)，缓冲区溢出可能导致任意代码执行

**建议**:
- 默认启用强栈保护 (`CC_STACKPROTECTOR_STRONG`)
- 添加 `-fstack-protector-strong` 到基础编译选项

---

### 3. 内存破坏风险

#### 风险点 3: 部分代码未使用安全内存函数

**风险等级**: 中

**证据**: 部分 IPC 实现可能仍使用标准 C 库函数

```c
// 需要确认: 是否有代码使用 strcpy/strncpy/memcpy 而非 *_s 版本
// 搜索: src/core/ipc/ 目录下的 memcpy, strcpy
```

**影响**: 缓冲区溢出风险

**建议**:
- 统一使用 `securec.h` 中的安全函数
- 添加编译期检查 (如 `-Werror=format-security`)

---

### 4. 资源耗尽攻击

#### 风险点 4: 信号量/队列资源无上限控制

**风险等级**: 低

**证据**: 信号量创建支持 0-0xFFFFFFFE 计数，但无全局资源限制

```c
// src/include/uapi/prt_sem.h
#define OS_SEM_COUNT_MAX 0xFFFFFFFEU
```

**影响**: 恶意任务可能创建大量信号量耗尽内存

**建议**:
- 添加系统级资源配额
- 任务创建时限制可持有信号量数量

---

### 5. 优先级反转风险

#### 风险点 5: 信号量优先级继承可选

**风险等级**: 低

**证据**: 优先级继承是可选配置，非默认启用

**影响**: 高优先级任务可能被低优先级任务长期阻塞

**建议**:
- 关键场景默认启用优先级继承
- 添加优先级反转检测机制

---

### 6. 竞争条件

#### 风险点 6: 中断上下文资源访问

**风险等级**: 低

**证据**: 中断处理程序中可能访问非中断安全的数据结构

```c
// 风险: prt_hwi.c 中的中断处理可能访问共享资源
// 位置: src/core/kernel/irq/prt_irq.c
```

**建议**:
- 中断处理程序应最小化
- 延迟处理使用信号量/事件通知任务

---

### 7. 信息泄露

#### 风险点 7: 调试符号可能泄露敏感信息

**风险等级**: 低

**证据**: 构建产物包含完整符号表 (`OHOS_Image.sym`)

```gn
// BUILD.gn 第 276 行
command += " && sh -c '$objdump -t $uniproton_name | sort >$uniproton_name.sym.sorted'"
```

**影响**: 符号表泄露内部实现细节

**建议**:
- 生产版本使用 strip 移除符号
- 分离调试符号到独立文件

---

## 已实现的安全机制

### 1. 栈 Canary 随机化

**证据**: `src/security/rnd/prt_rnd_set.c`

```c
void PRT_SysSetRndNum(void) {
    g_memCanaryRdm = OsGetRngNum();
}
```

**作用**: 检测栈溢出攻击

---

### 2. SecureC 安全内存库

**依赖**: `//third_party/bounds_checking_function:libsec_static`

**证据**: `BUILD.gn` 第 227 行

```gn
deps = [ "//third_party/bounds_checking_function:libsec_static" ]
```

**提供的安全函数**:
- `memcpy_s()`
- `memset_s()`
- `strncpy_s()`
- `snprintf_s()`

---

### 3. 代码段保护属性

**证据**: IPC 源代码中的段属性

```c
// 信号量核心操作使用 L0 时间关键段
OS_SEC_L0_TEXT
// 队列/事件 API 使用 L4 标准段
OS_SEC_L4_TEXT
```

**作用**: 隔离关键代码路径，防止被意外覆盖

---

## 威胁模型总结

```
┌─────────────────────────────────────────────────────────┐
│                   UniProton 威胁模型                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  外部攻击面:                                             │
│  ┌─────────────────────────────────────────────────┐  │
│  │  串口/UART 输入        →  注入恶意命令           │  │
│  │  调试接口              →  内存读写               │  │
│  │  外设 DMA             →  内存覆盖               │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
│  内部攻击面:                                             │
│  ┌─────────────────────────────────────────────────┐  │
│  │  任务间通信            →  缓冲区溢出            │  │
│  │  系统调用               →  参数校验不足          │  │
│  │  动态内存分配          →  内存泄漏/耗尽        │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
│  信任边界:                                              │
│  ┌─────────────────────────────────────────────────┐  │
│  │  硬件 (唯一信任根)                             │  │
│  │  ├── UniProton 内核 (特权模式) ← 已分析       │  │
│  │  └── 任务 (非特权) ← 无法信任                  │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 安全改进建议

### 立即可做 (高优先级)

| # | 建议 | 难度 | 影响 |
|---|------|------|------|
| 1 | 启用强栈保护 (默认) | 低 | 防止栈溢出 |
| 2 | API 参数 NULL 检查 | 中 | 防止空指针崩溃 |
| 3 | 统一使用 SecureC 函数 | 中 | 防止缓冲区溢出 |

### 中期改进 (中优先级)

| # | 建议 | 难度 | 影响 |
|---|------|------|------|
| 4 | 添加资源配额限制 | 中 | 防止资源耗尽 |
| 5 | 默认启用优先级继承 | 低 | 防止优先级反转 |
| 6 | 中断处理最小化 | 中 | 减少竞态 |

### 长期考虑 (低优先级)

| # | 建议 | 难度 | 影响 |
|---|------|------|------|
| 7 | 添加 MPU 支持 | 高 | 内存隔离 |
| 8 | 固件签名验证 | 高 | 防篡改 |
| 9 | 安全启动链 | 高 | 防恶意固件 |

---

## 结论

UniProton 是一个**面向嵌入式场景的轻量级 RTOS**，安全机制相对简单：

| 评估项 | 评级 | 说明 |
|--------|------|------|
| 代码质量 | ⚠️ 中 | 建议加强输入校验 |
| 内存安全 | ⚠️ 中 | 部分代码需迁移到 SecureC |
| 威胁缓解 | ⚠️ 中 | 栈保护依赖配置 |
| 安全文档 | ❌ 缺失 | 无安全编码指南 |

**总体评级**: 对于资源受限的嵌入式场景，UniProton 的安全措施基本够用，但生产环境建议启用全部安全编译选项。

---

## 相关文档

- [概览](./01_Overview.md) - 项目定位
- [架构设计](./04_Architecture.md) - 内部机制
- [构建系统](./05_Build_System.md) - 编译配置
- [故障排查](./07_Troubleshooting.md) - 调试方法
