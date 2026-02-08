# UniProton 项目概览

## 项目定位

**UniProton** 是 OpenHarmony 的轻量级实时操作系统 (RTOS) 内核，具备以下特性：

| 特性 | 说明 |
|------|------|
| **极低时延** | 针对工业控制场景优化，中断响应和任务切换时间最短化 |
| **混合关键性** | 支持安全关键和非安全关键任务混合部署 |
| **多核支持** | 支持多核 CPU (AMP/SMP 架构) |
| **裸机运行** | 无需 MMU/MPU 即可运行，适用于 MCU 和应用处理器 |

> **证据**: `README.md` 第 3 行 - "UniProton是一款实时操作系统，具备极致的低时延和灵活的混合关键性部署特性"

## 核心能力

### 1. 任务管理 (Task)

- 支持多任务创建、删除、挂起、恢复
- 优先级调度 (0-31，0 最高，31 为 IDLE)
- 任务间同步与通信
- 支持 AMP (非对称多处理) 和 SMP (对称多处理)

**证据**: `src/core/kernel/task/` 目录下的 `prt_task.c` 及相关文件

### 2. 中断处理 (HWI)

- 硬件中断向量表管理
- 中断嵌套支持
- 中断延迟处理机制
- 支持 ARMv7-M 和 ARMv8 架构

**证据**: `src/core/kernel/irq/` 及 `src/arch/cpu/armv7-m/common/hwi/`

### 3. IPC 机制

UniProton 提供四种内核级 IPC 用于任务间通信：

| 机制 | 用途 | 头文件 |
|------|------|--------|
| **信号量 (Sem)** | 同步、互斥 | `prt_sem.h` |
| **消息队列 (Queue)** | 异步消息传递 | `prt_queue.h` |
| **事件标志 (Event)** | 事件通知 | `prt_event.h` |
| **读写锁 (Rwlock)** | 读写同步 | `prt_rwlock.h` |

**证据**: `src/core/ipc/` 目录下的实现文件

### 4. 内存管理

- 固定大小块分配器 (FSC - Fixed Size Chunk)
- 内存池管理
- 动态内存分配

**证据**: `src/mem/` 目录，`prt_mem.c`, `prt_fscmem.c`

### 5. 定时器

- 软件定时器 (SwTimer)
- 系统 Tick 管理
- 任务延时

**证据**: `src/core/kernel/timer/`, `src/core/kernel/tick/`

### 6. 文件系统

- LittleFS 集成
- VFS (Virtual File System) 层
- 文件操作接口

**证据**: `src/fs/` 目录，`prt_fs.h`

### 7. 网络

- lwIP 2.1 TCP/IP 栈
- Socket API 兼容

**证据**: `src/net/lwip-2.1/` 目录

### 8. 异常与错误处理

- 硬件异常捕获
- 软件错误处理
- Hook 机制 (用户自定义回调)

**证据**: `src/core/kernel/kexc/`, `src/om/hook/`

## 支持的硬件架构

| 架构 | 芯片系列 | 代码位置 |
|------|----------|----------|
| **ARMv7-M** | Cortex-M3/M4 | `src/arch/cpu/armv7-m/` |
| **Cortex-M4** | STM32F4 等 | `src/arch/cpu/armv7-m/cortex-m4/` |
| **ARMv8-M/ARMv8-A** | 64位处理器 | `src/arch/cpu/armv8/` |

**证据**: `README.md` 第 56 行 - "当前开源版本支持cortex_m4和armv8芯片"

## 运行环境

### 编译要求

| 工具 | 版本/要求 |
|------|-----------|
| GCC | GNU Arm Embedded Toolchain 10-2020-q4-major |
| 构建系统 | GN + Ninja |
| Python | 3.x (用于构建脚本) |

### 已验证开发板

| 开发板 | 示例目录 |
|--------|----------|
| STM32F407ZG | `device_soc_st/stm32f407zg/uniproton` |
| Alientek | `demos/alientek/` |
| Raspberry Pi 4 | `demos/raspi4/` (armv8) |
| Hi3093 | `demos/hi3093/` |

## 许可证

**Mulan PSL v2** - 开源许可证，允许自由使用和修改

**证据**: `README.md` 第 58 行 - "遵循MulanPSL2开源许可协议"

## 与 OpenHarmony 的关系

```
┌─────────────────────────────────────────────┐
│           OpenHarmony 系统架构               │
├─────────────────────────────────────────────┤
│  应用层 │  Framework │  系统服务              │
├─────────────────────────────────────────────┤
│              用户态运行时                     │
├─────────────────────────────────────────────┤
│  HDF (Hardware Driver Foundation)           │
├─────────────────────────────────────────────┤
│  UniProton 内核 (kernel_uniproton) ✓        │
├─────────────────────────────────────────────┤
│              硬件                            │
└─────────────────────────────────────────────┘
```

UniProton 是 OpenHarmony 的轻量级设备内核选项之一，与 Linux 内核 (kernel_linux) 并列。

## 关键概念

### 1. PID (Process/Task ID)

UniProton 使用 PID 标识任务，格式定义：

```c
/* 证据: src/include/uapi/prt_task.h */
#define OS_TSK_TCB_INDEX_BITS ((4 - OS_TSK_CORE_BYTES_IN_PID) * 8)
#define GET_HANDLE(pid)       ((pid) & ((1U << OS_TSK_TCB_INDEX_BITS) - 1))
#define GET_COREID(pid)      ((U8)((pid) >> OS_TSK_TCB_INDEX_BITS))
#define COMPOSE_PID(coreid, handle) ((((U32)(coreid)) << OS_TSK_TCB_INDEX_BITS) + ((U8)(handle)))
```

### 2. 任务优先级

- 范围: 0-31 (0 最高优先级)
- IDLE 任务占用优先级 31
- 用户任务不能使用优先级 31

**证据**: `src/include/uapi/prt_task.h` 第 69 行

### 3. 时间片 (Tick)

系统 Tick 是时间管理的基本单位，由硬件定时器驱动。

## 相关文档

- [API 参考](./03_API_Reference.md) - 完整 API 列表
- [目录结构](./02_Directory_Structure.md) - 代码组织
- [架构设计](./04_Architecture.md) - 内部实现
