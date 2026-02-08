# 目录结构与模块职责

> **文档版本**: 1.0
> **生成时间**: 2026-02-06
> **适用范围**: OpenHarmony TEE OS Kernel 代码组织

---

## 文档目的

本文档提供 tee_os_kernel 仓库的完整目录结构，按模块职责归类，帮助新人快速定位代码。

---

## 目录

- [目录树概览](#目录树概览)
- [内核模块](#内核模块)
- [用户态服务](#用户态服务)
- [用户态库](#用户态库)
- [构建与配置](#构建与配置)
- [文档与工具](#文档与工具)
- [文件统计](#文件统计)

---

## 目录树概览

```
tee_os_kernel/
├── README.md              # 项目文档（中文）
├── README.en.md           # 项目文档（英文）
├── LICENSE                # Mulan PSL v2 许可证
├── OAT.xml                # OpenHarmony 合规文件
├── bundle.json            # Bundle 配置
├── config.mk              # 构建配置（CHCORE_* 宏）
├── Makefile               # 根 Makefile
│
├── build/                 # 构建系统
│   ├── BUILD.gn         # GN 构建配置（仅入口点）
│   ├── build_tee.sh      # TEE 完整构建脚本
│   ├── oh_build_tee.sh   # OpenHarmony 构建包装器
│   └── clean.sh          # 清理脚本
│
├── figures/               # 文档图片资源
│   └── overview-of-opentrustee.png
│
├── kernel/               # 内核核心（168 文件）
│   ├── arch/aarch64/    # ARM64 架构相关（62 文件）
│   │   ├── head.S         # 内核入口汇编
│   │   ├── main.c         # 内核初始化
│   │   ├── tools.S        # 汇编工具
│   │   ├── boot/          # 平台启动代码
│   │   │   ├── rk3568/   # RK3568 启动
│   │   │   └── rk3399/   # RK3399 启动
│   │   ├── irq/           # 中断处理
│   │   │   ├── irq.S
│   │   │   └── ipi.c      # 核间中断
│   │   ├── mm/            # 内存管理（arch）
│   │   ├── sched/         # 调度（arch）
│   │   ├── plat/          # 平台相关
│   │   │   ├── rk3568/
│   │   │   └── rk3399/
│   │   ├── trustzone/      # TrustZone 支持
│   │   │   ├── tlogger.c # 安全日志
│   │   │   └── spd/        # Secure Payload Dispatcher
│   │   │       ├── opteed/  # OP-TEE dispatcher
│   │   │       └── teed/    # TEE dispatcher
│   │   └── backtrace/     # 栈回溯
│   ├── ipc/              # IPC（3 文件）
│   │   ├── connection.c
│   │   ├── notification.c
│   │   └── channel.c
│   ├── irq/              # 中断（2 文件）
│   │   ├── ipi.c
│   │   └── timer.c
│   ├── lib/              # 内核库（4 文件）
│   │   ├── printk.c
│   │   ├── rbtree.c
│   │   ├── radix.c
│   │   └── ring_buffer.c
│   ├── mm/               # 内存管理（7 文件）
│   │   ├── mm.c
│   │   ├── buddy.c
│   │   ├── slab.c
│   │   ├── kmalloc.c
│   │   ├── vmspace.c
│   │   ├── pgfault_handler.c
│   │   └── extable.c
│   ├── object/            # 内核对象（10 文件）
│   │   ├── thread.c
│   │   ├── cap_group.c
│   │   ├── capability.c
│   │   ├── memory.c
│   │   ├── irq.c
│   │   ├── recycle.c
│   │   ├── user_fault.c
│   │   └── set_thread_env.c
│   ├── sched/             # 线程调度（4 文件）
│   │   ├── sched.c
│   │   ├── context.c
│   │   ├── policy_rr.c
│   │   └── policy_pbrr.c
│   ├── syscall/           # 系统调用（3 文件）
│   │   ├── syscall.c
│   │   ├── syscall_hooks.c
│   │   └── syscall_num.h
│   └── include/           # 内核头文件（72 文件）
│       ├── arch/aarch64/
│       ├── common/
│       ├── io/
│       ├── irq/
│       ├── mm/
│       ├── sched/
│       ├── object/
│       ├── ipc/
│       └── syscall/
│
├── patches/              # 补丁
│   └── patches.json
│
├── tools/                # 开发工具
│   └── read_procmgr_elf_tool/
│       ├── main.c
│       ├── elf.c
│       └── elf.h
│
├── user/                 # 用户空间（139 文件）
│   ├── chcore-libs/
│   │   └── sys-libs/libohtee/    # TEE 库（23 文件）
│   │       ├── include/
│   │       ├── ipclib.c
│   │       ├── teecall.c
│   │       ├── mem_ops.c
│   │       └── drv_*.c
│   └── system-services/
│       └── system-servers/
│           ├── chcore-libc/         # ChCore libc（70 文件）
│           │   └── libchcore/
│           ├── procmgr/             # 进程管理器（16 文件）
│           ├── fs_base/             # VFS 基础（9 文件）
│           ├── fsm/                 # 文件系统管理（9 文件）
│           ├── tmpfs/               # 内存文件系统（8 文件）
│           └── chanmgr/             # Channel 管理器（3 文件）
│
└── wiki/                # 本文档
    ├── README.md
    └── _work/
        ├── NOTES.md
        └── PLAN.md
```

---

## 内核模块

### kernel/arch/aarch64/ - 架构相关代码

**职责**：ARM64 架构特定实现

| 子模块 | 文件 | 职责 |
|--------|------|------|
| **启动** | head.S, main.c, tools.S | 内核入口、初始化序列、汇编工具 |
| **boot/** | rk3568/, rk3399/ | 平台启动代码、页表初始化 |
| **irq/** | irq.S, ipi.c, pgfault.c | 中断入口、IPI、页错误处理 |
| **mm/** | page_table.c, tlb.c, cache.c, uaccess.c | 页表管理、TLB、缓存、用户空间访问 |
| **sched/** | context.c, fpu.c, sched.c, idle.S | 上下文切换、FPU、调度、空闲线程 |
| **plat/** | rk3568/, rk3399/ | 平台配置、UART、定时器、电源管理 |
| **trustzone/** | tlogger.c, spd/ | TrustZone 日志、Secure Payload Dispatcher |

**关键文件**：
- `kernel/arch/aarch64/main.c:1-50` - 内核主初始化函数
- `kernel/arch/aarch64/head.S` - 早期启动汇编入口
- `kernel/arch/aarch64/irq/irq_entry.c` - 异常和中断入口

### kernel/ipc/ - 进程间通信

**职责**：实现微内核 IPC 机制

| 文件 | 职责 |
|------|------|
| `connection.c` | ChCore Connection-based IPC（register_server/client, ipc_call/return）|
| `channel.c` | TEE 专用消息通道（sys_tee_msg_*）|
| `notification.c` | 通知机制（sys_create_notifc, sys_wait, sys_notify）|

**关键结构**：
- `ipc_connection` - 客户端-服务端连接对象
- `channel` - TEE IPC 通道对象
- `notification` - 通知对象

### kernel/irq/ - 中断处理

**职责**：中断和定时器管理

| 文件 | 职责 |
|------|------|
| `ip i.c` | 核间中断（IPI）处理器 |
| `timer.c` | 定时器中断管理 |

### kernel/lib/ - 内核库

**职责**：内核通用工具和数据结构

| 文件 | 职责 |
|------|------|
| `printk.c` | 内核打印（类似 kernel printk）|
| `rbtree.c` | 红黑树实现 |
| `radix.c` | Radix 树实现 |
| `ring_buffer.c` | 环形缓冲区 |

### kernel/mm/ - 内存管理

**职责**：物理内存和虚拟内存管理

| 文件 | 职责 |
|------|------|
| `mm.c` | 内存管理初始化、核心接口 |
| `buddy.c` | Buddy 物理页分配器 |
| `slab.c` | Slab 对象分配器 |
| `kmalloc.c` | 内核 malloc 实现 |
| `vmspace.c` | 虚拟地址空间管理 |
| `pgfault_handler.c` | 页错误处理 |
| `extable.c` | 异常表 |

**关键结构**：
- `vmregion` - 虚拟内存区域
- `vmspace` - 虚拟地址空间
- `pmobject` - 物理内存对象

### kernel/object/ - 内核对象系统

**职责**：管理所有内核对象类型

| 文件 | 职责 |
|------|------|
| `thread.c` | 线程对象管理 |
| `cap_group.c` | Capability Group（进程容器）管理 |
| `capability.c` | Capability 分配/复制/撤销 |
| `memory.c` | PMO（物理内存对象）管理 |
| `irq.c` | IRQ 对象管理 |
| `recycle.c` | 对象回收机制 |
| `user_fault.c` | 用户态错误处理 |
| `set_thread_env.c` | 线程环境设置 |

**关键结构**：
- `struct object` - 所有对象的基类
- `struct cap_group` - Capability Group（进程标识）
- `struct thread` - 线程对象

### kernel/sched/ - 线程调度

**职责**：实现线程调度算法

| 文件 | 职责 |
|------|------|
| `sched.c` | 核心调度器 |
| `context.c` | 上下文切换 |
| `policy_rr.c` | 简单轮转策略 |
| `policy_pbrr.c` | 优先级轮转策略（PBRR）|

**关键结构**：
- `enum thread_state` - 线程状态（TS_INIT, TS_READY, TS_RUNNING, TS_WAITING）
- `enum thread_type` - 线程类型（TYPE_IDLE, TYPE_KERNEL, TYPE_USER, TYPE_SHADOW, TYPE_REGISTER）

### kernel/syscall/ - 系统调用

**职责**：系统调用分发和实现

| 文件 | 职责 |
|------|------|
| `syscall.c` | 系统调用表、分发逻辑、参数校验 |
| `syscall_hooks.c` | 系统调用权限钩子 |
| `syscall_num.h` | 系统调用号定义（256 个）|

### kernel/include/ - 内核头文件

**职责**：提供内核接口定义

| 目录 | 头文件数量 | 典型文件 |
|------|------------|----------|
| `arch/aarch64/` | ~30 | machine.h, mm.h, sched.h, trustzone/smc.h |
| `common/` | ~15 | types.h, macro.h, errno.h, lock.h |
| `io/` | ~5 | uart.h |
| `irq/` | ~3 | irq.h, irq_num.h |
| `mm/` | ~8 | vmspace.h, page_fault.h |
| `sched/` | ~2 | sched.h, context.h |
| `object/` | ~5 | object.h, thread.h, cap_group.h, memory.h |
| `ipc/` | ~2 | connection.h, channel.h |
| `syscall/` | ~1 | syscall_hooks.h |

---

## 用户态服务

### user/system-services/system-servers/procmgr/ - 进程管理器

**职责**：中央进程管理服务

| 文件 | 职责 |
|------|------|
| `procmgr.c` | 主循环、服务管理、系统调用处理 |
| `loader.c` | ELF 加载器、动态链接 |
| `srvmgr.c` | 服务管理器 |
| `proc_node.c` | 进程节点管理 |
| `recycle.c` | 进程回收机制 |
| `oh_mem_ops.c` | TEE 内存操作 |
| `include/srvmgr.h` | 服务管理接口 |
| `include/loader.h` | 加载器接口 |
| `include/proc_node.h` | 进程节点接口 |

**关键功能**：
- 进程创建/销毁
- 服务注册/查找
- ELF 加载
- 进程资源回收
- 堆大小限制

### user/system-services/system-servers/fs_base/ - VFS 基础

**职责**：虚拟文件系统基础抽象

| 文件 | 职责 |
|------|------|
| `fs_vnode.c` | VNode 管理 |
| `fs_page_cache.c` | 页缓存实现 |
| `fs_page_fault.c` | 页错误处理 |
| `fs_wrapper.c` | VFS 包装器 |
| `fs_wrapper_ops.c` | VFS 操作实现 |

**关键结构**：
- `struct vnode` - 虚拟文件节点
- `struct page_cache` - 页缓存

### user/system-services/system-servers/fsm/ - 文件系统管理器

**职责**：文件系统挂载和分发

| 文件 | 职责 |
|------|------|
| `fsm.c` | 文件系统管理主逻辑 |
| `device.c` | 设备管理 |
| `mount_info.c` | 挂载信息管理 |
| `fsm_client_cap.c` | FSM 客户端能力 |

### user/system-services/system-servers/tmpfs/ - 内存文件系统

**职责**：RAM 文件系统（tmpfs）实现

| 文件 | 职责 |
|------|------|
| `tmpfs.c` | tmpfs 核心实现 |
| `namei.c` | 名称索引 |
| `internal_ops.c` | 内部操作 |
| `main.c` | 主入口 |
| `mem_usage_tool.c` | 内存使用追踪工具（DEBUG）|

**特点**：
- 纯内存存储
- 无持久化
- 适合 TEE 内部文件操作

### user/system-services/system-servers/chanmgr/ - Channel 管理器

**职责**：TEE IPC Channel 命名和索引

| 文件 | 职责 |
|------|------|
| `chanmgr.c` | Channel 注册、查找、分发 |
| `main.c` | 主入口 |
| `include/chanmgr.h` | 接口定义 |

**关键功能**：
- Channel 按名称注册
- Channel 按 task_id 查找
- TA 管理（TAMgr）
- 缓存管理

---

## 用户态库

### user/chcore-libs/sys-libs/libohtee/ - TEE 库

**职责**：OpenHarmony TEE 用户态库

| 文件 | 职责 |
|------|------|
| `ipclib.c` | IPC 客户端库 |
| `teecall.c` | TEE 系统调用封装 |
| `mem_ops.c` | 共享内存操作 |
| `drv_hwi_share.c` | 硬件中断共享 |
| `drv_io_share.c` | I/O 共享 |
| `fileio.c` | 文件 I/O |
| `hm_cache_flush.c` | 缓存刷新 |
| `usrsyscall_irq.c` | 用户态中断系统调用 |
| `usrsyscall_smc.c` | 用户态 SMC 系统调用 |

**公共头**（`include/`）：
- `ipclib.h` - IPC 接口
- `teecall.h` - TEE 调用接口
- `mem_ops.h` - 内存操作接口
- `tee_uuid.h` - UUID 定义

### user/system-services/chcore-libc/ - ChCore libc

**职责**：musl libc 的 ChCore 移植

**关键目录**：
- `libchcore/porting/overrides/` - ChCore 特有覆盖
- `libchcore/porting/patches/` - musl 补丁
- `libchcore/arch/aarch64/` - ARM64 特有代码

**主要接口**：
- `chcore/ipc.h` - 用户态 IPC
- `chcore/syscall.h` - 系统调用封装
- `chcore-internal/*` - 内部接口

---

## 构建与配置

### build/ - 构建系统

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | GN 构建入口（仅定义 tee_os target）|
| `build_tee.sh` | TEE 完整构建（调用框架构建）|
| `oh_build_tee.sh` | OpenHarmony 构建包装（当前为 stub）|
| `clean.sh` | 清理脚本 |

### 配置文件

| 文件 | 职责 |
|------|------|
| `config.mk` | 构建配置（CHCORE_* 宏）|
| `bundle.json` | OpenHarmony Bundle 定义 |
| `OAT.xml` | OpenHarmony 测试适配（OAT）|
| `Makefile` | 根 Makefile（协调 kernel 和 user 构建）|

---

## 文档与工具

### figures/ - 文档资源

| 文件 | 用途 |
|------|------|
| `overview-of-opentrustee.png` | 系统架构图 |

### tools/ - 开发工具

| 目录 | 用途 |
|------|------|
| `read_procmgr_elf_tool/` | procmgr ELF 读取工具（用于调试）|

**工具功能**：
- 读取 procmgr ELF 格式
- 分析进程布局
- 调试进程加载

---

## 文件统计

### 按模块统计

| 模块 | 文件数 | 说明 |
|--------|--------|------|
| kernel/ | ~168 | 内核核心代码 |
| user/ | ~139 | 用户态服务与库 |
| build/ | ~4 | 构建脚本 |
| tools/ | ~3 | 开发工具 |
| **总计** | **~314** | 不包括文档和配置 |

### 按类型统计

| 类型 | 大约数量 |
|------|----------|
| .c 源文件 | ~230 | C 源代码 |
| .h 头文件 | ~70 | 头文件 |
| .S 汇编文件 | ~14 | ARM 汇编 |

### 按平台统计

| 平台 | 支持文件 |
|------|----------|
| RK3568 | `kernel/arch/aarch64/plat/rk3568/` |
| RK3399 | `kernel/arch/aarch64/plat/rk3399/` |

---

## 模块依赖关系

### 核心依赖链

```
kernel/arch/aarch64/
├── 依赖 kernel/include/common/*
├── 依赖 kernel/include/mm/*
├── 依赖 kernel/include/sched/*
└── 平台相关代码

kernel/ipc/
├── 依赖 kernel/object/*
├── 依赖 kernel/mm/*
├── 依赖 kernel/irq/*
└── 依赖 kernel/sched/*

kernel/mm/
├── 依赖 kernel/object/*
├── 依赖 kernel/lib/*
└── 依赖 kernel/include/mm/*

kernel/sched/
├── 依赖 kernel/object/*
├── 依赖 kernel/mm/*
└── 依赖 kernel/lib/*

kernel/syscall/
├── 依赖 kernel/object/*
├── 依赖 kernel/mm/*
├── 依赖 kernel/sched/*
├── 依赖 kernel/ipc/*
└── 依赖 kernel/irq/*
```

### 用户态依赖链

```
user/system-services/system-servers/procmgr/
├── 依赖 chcore-libc/libchcore/
└── 依赖 libohtee/

user/system-services/system-servers/fs_base/
├── 依赖 chcore-libc/libchcore/
└── 被 fsm/ 依赖

user/system-services/system-servers/fsm/
├── 依赖 chcore-libc/libchcore/
├── 依赖 fs_base/
└── 依赖 libohtee/

user/system-services/system-servers/tmpfs/
├── 依赖 chcore-libc/libchcore/
├── 依赖 fs_base/
└── 依赖 libohtee/

user/system-services/system-servers/chanmgr/
├── 依赖 chcore-libc/libchcore/
└── 依赖 libohtee/

user/chcore-libs/sys-libs/libohtee/
├── 依赖 chcore-libc/libchcore/
└── 提供接口给用户服务
```

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位与核心能力
- [架构设计](02_Architecture.md) - 组件图、数据流、线程模型
- [系统调用接口](03_Syscall_Interfaces.md) - 完整的 API 清单
- [内部 API](04_Internal_APIs.md) - 模块间接口与依赖
- [GN Targets](05_GN_Targets.md) - 构建系统详解
- [编译产物](06_Build_Artifacts.md) - 产物清单与路径
- [安全评审](07_Security_Review.md) - 攻击面与信任边界

---

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2026-02-06
