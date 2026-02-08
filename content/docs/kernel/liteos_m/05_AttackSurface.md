# 攻击面分析

> 识别所有外部输入点、敏感操作和信任边界

**最后更新**：2026-02-07
**适用人群**：安全研究员
**证据来源**：代码侦查结果 + 人工分析

---

## 外部输入清单

### 1. 系统调用入口

**入口位置**：`components/security/syscall/los_syscall.c:68`

**处理函数**：`OsSyscallHandle(UINT32 *args)`

**输入源**：
- 用户态通过 SVC 指令触发
- 参数通过 R0-R6 寄存器传递
- 调用号存储在 R7 寄存器

**关键验证**：
- 调用号边界检查：`svcNum >= SYS_CALL_NUM_LIMIT`
- 参数数量检查：`nArgs > ARG_NUM_7`

**系统调用表**：`components/security/syscall/syscall_lookup.h`

#### 主要系统调用

| 调用号 | 名称 | 处理函数 | 参数 | 权限检查 |
|--------|------|----------|------|---------|
| __NR_sched_setscheduler | 设置调度器 | `SysSchedSetScheduler` | (tid, policy, priority) | tid 范围、priority 范围 |
| __NR_set_thread_area | 设置线程区域 | `SysSetThreadArea` | (area) | TODO(待分析) |
| __NR_get_thread_area | 获取线程区域 | `SysGetThreadArea` | () | TODO(待分析) |
| __NR_create_user_thread | 创建用户线程 | `SysUserTaskCreate` | (entry, userArea, userSp, joinable) | userSp 验证 |

**证据**：
- `components/security/syscall/los_syscall.c:68` - 系统调用处理入口
- `components/security/syscall/syscall_lookup.h` - 系统调用表定义
- `components/security/syscall/pthread_syscall.c:47,68` - 用户线程创建和调度器设置

---

### 2. 文件系统操作

#### 文件打开

**入口位置**：`components/fs/vfs/vfs_fs.c:494`

**函数**：`open(const char *path, int flags, ...)`

**输入源**：
- 用户态路径字符串
- 打开标志（O_RDONLY, O_WRONLY, O_RDWR, O_CREAT 等）
- 模式参数（创建文件时的权限）

**关键验证**：
- 路径规范化：`GetCanonicalPath()` - 处理 `/.`, `//`, `/../`
- 挂载点权限检查：只读挂载点禁止创建文件
- 路径长度限制：`PATH_MAX`

**潜在攻击**：
- 路径遍历攻击（通过 `../` 逃逸到任意目录）
- 符号链接攻击（如支持）
- 权限提升（通过敏感路径）

**证据**：
- `components/fs/vfs/vfs_fs.c:494` - `open()` 函数实现

---

#### 文件读写

**读取入口**：`components/fs/vfs/vfs_fs.c:590`

**函数**：`read(int fd, void *buff, size_t bytes)`

**写入入口**：`components/fs/vfs/vfs_fs.c:635`

**函数**：`write(int fd, const void *buff, size_t bytes)`

**输入源**：
- 文件描述符 `fd`
- 用户态缓冲区指针 `buff`
- 读写长度 `bytes`

**关键验证**：
- 文件描述符有效性检查
- 用户态缓冲区指针可读/可写性
- 读写长度边界检查

**潜在攻击**：
- 缓冲区溢出（如果长度验证不足）
- 信息泄露（通过越界读取）
- 任意写入（通过验证绕过）

**证据**：
- `components/fs/vfs/vfs_fs.c:590,635` - `read()` 和 `write()` 函数实现

---

### 3. 网络接口

**入口位置**：`components/fs/vfs/vfs_fs.c:55`（通过 LwIP）

**API**：socket, bind, listen, accept, recv, send, recvfrom, sendto

**输入源**：
- Socket 地址结构（IP + 端口）
- 网络数据包内容
- Socket 选项

**关键验证**：
- 地址结构长度检查
- 端口范围检查
- 数据包长度边界检查

**潜在攻击**：
- 缓冲区溢出（通过超长数据包）
- 拒绝服务（通过大量连接）
- 协议栈漏洞（LwIP 已知漏洞）

**证据**：
- `components/fs/vfs/vfs_fs.c:55` - LwIP socket 集成
- `components/net/lwip-2.1/` - LwIP 协议栈实现

---

### 4. 消息队列 IPC

**入口位置**：`kal/posix/src/mqueue.c:262`

**函数**：`mq_open(const char *mqName, int openFlag, ...)`

**输入源**：
- 队列名称
- 打开标志
- 队列属性（优先级、最大消息数、消息大小）

**关键验证**：
- 队列名称格式验证
- 队列属性边界检查
- 权限验证

**潜在攻击**：
- 资源耗尽（创建过多队列）
- 拒绝服务（发送超大消息）
- 信息泄露（通过未初始化缓冲区）

**证据**：
- `kal/posix/src/mqueue.c:262` - `mq_open()` 函数实现

---

### 5. 动态加载入口

**入口位置**：`components/dynlink/los_dynlink.c:748`

**函数**：`LOS_SoLoad(const CHAR *fileName, VOID *pool)`

**输入源**：
- ELF 文件路径
- 外部内存池指针

**关键验证**：
- ELF 头验证（魔数、类型、架构匹配）
- 文件大小限制：`FILE_LENGTH_MAX`
- 程序头数量限制：`PHDR_NUM_MAX`
- 偏移量验证：`e_phoff > fileLen`
- 禁止依赖其他共享库：`DT_NEEDED` 检查返回 `-ENOTSUP`
- 禁止 TEXT 重定位：`DT_TEXTREL` 检查

**潜在攻击**：
- ELF 格式畸形（导致解析漏洞）
- 路径遍历（通过文件名）
- 代码注入（通过符号重定位）
- 内存破坏（通过畸形段）

**证据**：
- `components/dynlink/los_dynlink.c:748` - `LOS_SoLoad()` 函数
- `components/dynlink/los_dynlink.c:103` - `OsVerifyEhdr()` ELF 头验证
- `components/dynlink/los_dynlink.c:581` - `OsDoReloc()` 重定位处理

---

## 敏感操作清单

### 1. ELF 文件验证

**位置**：`components/dynlink/los_dynlink.c:103`

**操作**：`OsVerifyEhdr()`

**敏感点**：
- ELF 魔数检查
- 类型检查（必须是 ET_DYN）
- 架构匹配检查
- 程序头数量限制（PHDR_NUM_MAX）
- 偏移量有效性（e_phoff > fileLen）
- 文件名长度检查（PATH_MAX）

**风险**：如果验证绕过，可能导致任意代码执行或内存破坏

**证据**：
- `components/dynlink/los_dynlink.c:103` - ELF 头验证函数

---

### 2. 符号重定位

**位置**：`components/dynlink/los_dynlink.c:581`

**操作**：`OsDoReloc()`

**敏感点**：
- 重定位类型：R_ARCH_GLOB_DAT, R_ARCH_JUMP_SLOT, R_ARCH_ABS32, R_ARCH_RELATIVE
- **关键写入操作**：`*(UINTPTR *)relocAddr = symAddr + addend`

**风险**：
- 如果 `relocAddr` 未验证，可能导致任意地址写入
- 如果 `symAddr` 未验证，可能导致代码注入

**证据**：
- `components/dynlink/los_dynlink.c:581` - 重定位处理函数

---

### 3. 任务创建

**位置**：`components/security/syscall/pthread_syscall.c:47`

**操作**：`SysUserTaskCreate(entry, userArea, userSp, joinable)`

**敏感点**：
- 用户态栈地址 `userSp` 需指向有效用户空间
- 入口函数 `entry` 需指向有效代码段
- 栈边界检查

**风险**：
- 如果 `userSp` 未验证，可能导致用户态读取内核栈
- 如果 `entry` 未验证，可能导致任意代码执行

**证据**：
- `components/security/syscall/pthread_syscall.c:47` - 用户任务创建函数

---

### 4. 优先级设置

**位置**：`components/security/syscall/pthread_syscall.c:68`

**操作**：`SysSchedSetScheduler(tid, policy, priority)`

**敏感点**：
- 任务 ID 范围检查：`tid > LOSCFG_BASE_CORE_TSK_LIMIT`
- 策略检查：policy 必须为 0
- 优先级范围检查：`priority <= 0 || priority > OS_TASK_PRIORITY_LOWEST`

**风险**：
- 如果验证绕过，可能导致优先级反转或拒绝服务
- 如果允许修改其他任务的优先级，可能导致特权提升

**证据**：
- `components/security/syscall/pthread_syscall.c:68` - 调度器设置函数

---

### 5. 内存分配操作

**操作**：
- `LOS_MemAlloc(OS_SYS_MEM_ADDR, size)` - 系统堆分配
- `LOS_MemAllocAlign(dso->pool, loadSize, boundary)` - 对齐分配（动态加载）
- `LOS_MemFree(OS_SYS_MEM_ADDR, ptr)` - 内存释放

**敏感点**：
- 大小边界检查
- 指针有效性检查
- 对齐边界检查

**风险**：
- 如果大小验证不足，可能导致堆溢出
- 如果指针验证不足，可能导致 Use-After-Free

**证据**：
- `kernel/src/los_memory.c` - 堆内存管理实现
- `components/dynlink/los_dynlink.c` - 动态加载内存分配

---

## 信任边界

### 用户态 / 内核态边界

```
┌─────────────────────────────────────────┐
│         用户态（User Mode）             │
│  - 应用代码                            │
│  - 用户库（userlib）                  │
├─────────────────────────────────────────┤
│ 信任边界（系统调用 - SVC）             │
│ - 参数验证                             │
│ - 权限检查                             │
├─────────────────────────────────────────┤
│         内核态（Kernel Mode）           │
│  - 内核核心（kernel/src/）            │
│  - 安全组件（security/）               │
│  - 文件系统（fs/）                    │
│  - 网络协议栈（net/）                 │
├─────────────────────────────────────────┤
│  硬件层（HAL）                        │
└─────────────────────────────────────────┘
```

**边界机制**：
- **系统调用**：`components/security/syscall/` - 用户态到内核态的受控接口
- **沙箱**：`components/security/box/` - 用户态任务隔离
- **MPU/TrustZone**：架构层内存保护

**证据**：
- `components/security/syscall/los_syscall.c` - 系统调用处理
- `components/security/box/los_box.c` - 沙箱实现
- `arch/arm/cortex-m*/gcc/los_mpu.c` - MPU 配置（ARM）
- `arch/arm/cortex-m*/gcc/los_trustzone.c` - TrustZone 支持（ARM Cortex-M33/M55）

---

### 外部输入信任边界

```
外部输入源
    ↓
┌─────────────────────────────────────┐
│ 输入验证层（第一道防线）           │
│ - 类型验证                           │
│ - 长度验证                           │
│ - 范围验证                           │
├─────────────────────────────────────┤
│ 业务逻辑层                           │
│ - 功能实现                           │
│ - 权限检查                           │
├─────────────────────────────────────┤
│ 内核资源层                           │
│ - 内存管理                           │
│ - 资源分配                           │
└─────────────────────────────────────┘
```

**外部输入流向**：
1. **系统调用参数** → `los_syscall.c` → 各子系统
2. **文件路径** → `vfs_fs.c` → 文件系统驱动
3. **网络数据包** → `lwip` → 协议栈处理
4. **ELF 文件** → `los_dynlink.c` → 动态加载

---

## 动态加载入口

### 加载流程

```
1. LOS_SoLoad(fileName, pool)
   ↓
2. OsVerifyEhdr() - ELF 头验证
   ↓
3. OsLoadPhdrs() - 加载程序头
   ↓
4. OsLoadSegments() - 加载段到内存
   ↓
5. OsDoReloc() - 符号重定位
   ↓
6. OsInitDso() - 初始化 DSO
```

### 关键验证点

| 阶段 | 验证内容 | 文件位置 |
|------|---------|---------|
| ELF 头验证 | 魔数、类型、架构、程序头数量、偏移量 | `los_dynlink.c:103` |
| 段加载 | 段类型、虚拟地址、文件大小、内存大小 | `los_dynlink.c:200+` |
| 重定位 | 重定位类型、符号有效性、目标地址 | `los_dynlink.c:581` |
| 依赖检查 | 禁止 DT_NEEDED | `los_dynlink.c:455` |
| TEXT 保护 | 禁止 DT_TEXTREL | `los_dynlink.c:464` |

### 潜在利用路径

```
畸形 ELF 文件
    ↓
绕过 ELF 头验证（如果验证不足）
    ↓
导致段加载错误（如 e_phoff 指向无效地址）
    ↓
触发越界读取或任意地址写入
    ↓
导致信息泄露或代码执行
```

**证据**：
- `components/dynlink/los_dynlink.c` - 完整的动态加载实现

---

## 安全组件

### 1. Syscall 组件

**功能**：系统调用分发处理，提供用户态到内核态的受控接口

**位置**：`components/security/syscall/`

**关键函数**：
- `OsSyscallHandleInit()` - 初始化系统调用表
- `OsSyscallHandle()` - 系统调用处理入口

**安全特性**：
- 调用号边界检查：`svcNum >= SYS_CALL_NUM_LIMIT`
- 参数数量验证：`nArgs <= ARG_NUM_7`

**证据**：
- `components/security/syscall/los_syscall.c` - 系统调用核心逻辑
- `components/security/syscall/syscall_lookup.h` - 系统调用表

---

### 2. Box 组件（沙箱）

**功能**：用户态任务隔离管理，实现用户/内核态分离

**位置**：`components/security/box/`

**关键函数**：
- `OsUserTaskInit()` - 初始化用户任务上下文
- `HalUserTaskStackInit()` - 设置用户态栈
- `LOS_BoxStart()` - 启动沙箱

**数据结构**：
- `LosBoxCB` - 沙箱配置（代码段、堆、栈地址和大小）

**安全特性**：
- 用户态任务独立栈空间
- 系统调用强制经过沙箱检查

**证据**：
- `components/security/box/los_box.c` - 沙箱实现
- `components/security/box/los_box.h` - 沙箱数据结构

---

### 3. Userlib 组件

**功能**：用户态库支持

**位置**：`components/security/userlib/`

**说明**：提供用户态可调用的库函数接口（具体实现需进一步分析）

---

### 4. 动态链接安全

**功能**：ELF 共享库加载与符号解析

**位置**：`components/dynlink/`

**关键函数**：
- `OsVerifyEhdr()` - ELF 头验证
- `OsDoReloc()` - 重定位处理
- `OsFindSymInDso()` / `OsFindSymInTable()` - 符号查找

**安全特性**：
- ELF 头验证（魔数、类型、架构）
- 程序头数量限制（PHDR_NUM_MAX）
- 禁止依赖其他共享库（DT_NEEDED）
- 禁止 TEXT 重定位（DT_TEXTREL）
- 支持 Hash 表查找

**证据**：
- `components/dynlink/los_dynlink.c` - 动态加载器实现

---

## 关键安全检查点

### 参数验证

| 检查点 | 位置 | 说明 |
|--------|------|------|
| 系统调用号边界 | `los_syscall.c` | `svcNum >= SYS_CALL_NUM_LIMIT` |
| 参数数量 | `los_syscall.c` | `nArgs > ARG_NUM_7` |
| 任务 ID 范围 | `pthread_syscall.c:68` | `tid > LOSCFG_BASE_CORE_TSK_LIMIT` |
| 优先级范围 | `pthread_syscall.c:68` | `priority <= 0 \|\| priority > OS_TASK_PRIORITY_LOWEST` |

---

### 内存操作

| 检查点 | 说明 |
|--------|------|
| 安全字符串函数 | memcpy_s, strcpy_s, strncpy_s（securec.h） |
| 堆内存分配检查 | NULL 返回值处理 |
| 边界检查 | 数组访问边界验证 |

---

### 文件系统安全

| 检查点 | 位置 | 说明 |
|--------|------|------|
| 路径规范化 | `vfs_fs.c` | `GetCanonicalPath()` - 处理 `/.`, `//`, `/../` |
| 挂载点权限 | `vfs_fs.c` | 只读挂载点禁止创建文件 |

---

### ELF 加载安全

| 检查点 | 位置 | 说明 |
|--------|------|------|
| 文件大小限制 | `los_dynlink.c` | `FILE_LENGTH_MAX` |
| 程序头数量限制 | `los_dynlink.c` | `PHDR_NUM_MAX` |
| 偏移量验证 | `los_dynlink.c` | `e_phoff > fileLen` |
| 禁止依赖库 | `los_dynlink.c:455` | DT_NEEDED 检查返回 `-ENOTSUP` |
| 禁止 TEXTREL | `los_dynlink.c:464` | DT_TEXTREL 检查 |

---

### 中断保护

| 检查点 | 说明 |
|--------|------|
| 关键操作保护 | 使用 `LOS_IntLock()` / `LOS_IntRestore()` |
| 中断嵌套计数 | `g_intCount` |

---

## 优先级建议（安全审计）

### 高优先级审计点

1. **系统调用参数验证** - `components/security/syscall/los_syscall.c`
   - 检查所有系统调用的参数验证是否完整
   - 重点关注用户态指针验证（可读/可写性）

2. **动态链接重定位** - `components/dynlink/los_dynlink.c:581`
   - 验证 `relocAddr` 的可信性
   - 检查是否存在任意地址写入风险

3. **文件路径处理** - `components/fs/vfs/vfs_fs.c`
   - 验证 `GetCanonicalPath()` 是否正确处理所有边界情况
   - 检查路径遍历防护是否完整

4. **用户栈初始化** - `components/security/syscall/pthread_syscall.c:47`
   - 验证 `userSp` 是否指向有效用户区域
   - 检查栈边界是否正确设置

5. **系统调用返回值** - `components/security/syscall/los_syscall.c:112`
   - 确认 `args` 指针是否指向内核安全区域

---

**下一节**：[安全风险评估](06_SecurityReview.md) - 深度分析潜在漏洞和利用路径
