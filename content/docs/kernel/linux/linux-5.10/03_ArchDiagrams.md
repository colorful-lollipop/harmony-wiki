# 架构图与数据流

## 概述

本文档使用 Mermaid 图表展示 OpenHarmony Linux Kernel 5.10 的架构、数据流和线程模型。

## 1. 整体架构图

### 1.1 内核架构概览

```mermaid
graph TB
    subgraph "用户空间"
        APP[应用程序]
        SVC[系统服务]
        LIBC[C 库]
    end

    subgraph "系统调用层"
        SC[系统调用接口
        SYSCALL_DEFINE*]
    end

    subgraph "内核核心子系统"
        direction TB
        SCHED[调度器
        kernel/sched/]
        MM[内存管理
        mm/]
        FS[文件系统
        fs/]
        NET[网络协议栈
        net/]
    end

    subgraph "内核基础设施"
        IRQ[中断管理
        kernel/irq/]
        TIME[时间管理
        kernel/time/]
        SYNC[同步机制
        kernel/locking/]
        TRACE[跟踪调试
        kernel/trace/]
    end

    subgraph "驱动框架"
        DD[设备驱动
        drivers/]
        BUS[总线框架
        drivers/base/]
        DT[设备树
        arch/*/boot/dts/]
    end

    subgraph "硬件抽象"
        BLOCK[块设备层
        block/]
        CHAR[字符设备]
        NETDEV[网络设备]
    end

    subgraph "硬件层"
        HW[硬件设备]
    end

    APP --> LIBC
    SVC --> LIBC
    LIBC --> SC
    SC --> SCHED & MM & FS & NET
    
    SCHED --> IRQ & TIME
    MM --> SYNC
    FS --> BLOCK
    NET --> NETDEV
    
    DD --> BUS
    BUS --> DT
    DT --> HW
    
    BLOCK --> HW
    NETDEV --> HW
    
    TRACE -.-> SCHED & MM & FS & NET
```

### 1.2 OpenHarmony 特有组件

```mermaid
graph TB
    subgraph "OpenHarmony 扩展"
        DFX[DFX 框架
        include/dfx/]
        
        subgraph "Staging 驱动"
            BLACKBOX[Blackbox
            崩溃收集]
            HILOG[HiLog
            日志系统]
            HIEVENT[HiEvent
            事件上报]
            ZEROHUNG[ZeroHung
            冻结检测]
            HUNGTASK[Hungtask
            任务挂起检测]
        end
        
        subgraph "特有功能"
            HMDFS[HMDFS
            分布式 FS]
            SHAREFS[ShareFS
            共享 FS]
            HYPERHOLD[HyperHold
            内存扩展]
            HCK[厂商钩子
            drivers/hck/]
        end
    end

    DFX --> BLACKBOX & HILOG & HIEVENT & ZEROHUNG & HUNGTASK
    
    HMDFS --> FS[标准文件系统层]
    SHAREFS --> FS
    HYPERHOLD --> MM[内存管理层]
    HCK --> KERNEL[内核核心]
```

## 2. 数据流图

### 2.1 系统调用处理流程

```mermaid
sequenceDiagram
    participant User as 用户空间
    participant Entry as 入口代码
    participant Table as 系统调用表
    participant Handler as 系统调用处理
    participant Kernel as 内核子系统

    User->>Entry: 执行 svc #0 (ARM64)
    Note over Entry: arch/arm64/kernel/syscall.c
    
    Entry->>Entry: el0_sync_handler
    Entry->>Entry: invoke_syscall
    
    Entry->>Table: 查询 sys_call_table[scno]
    Note over Table: 系统调用号映射
    
    Table->>Handler: 调用 SYSCALL_DEFINE*
    Note over Handler: 如 kernel/sys.c:prctl
    
    Handler->>Kernel: 执行内核操作
    Kernel-->>Handler: 返回结果
    
    Handler-->>Entry: 返回值
    Entry-->>User: 返回用户空间
```

### 2.2 内存管理数据流

```mermaid
flowchart LR
    subgraph "用户空间"
        MMAP[mmap 系统调用]
        MALLOC[malloc 库函数]
        ACCESS[内存访问]
    end

    subgraph "系统调用层"
        SC_MMAP[sys_mmap
        mm/mmap.c]
        SC_BRK[sys_brk
        mm/mmap.c]
    end

    subgraph "虚拟内存管理"
        VMA[VMA 管理
        mm/mmap.c]
        PG_FAULT[页故障处理
        mm/memory.c]
        RMAP[反向映射
        mm/rmap.c]
    end

    subgraph "物理内存管理"
        PAGE_ALLOC[页面分配
        mm/page_alloc.c]
        SLAB[SLAB 分配器
        mm/slab.c]
        VM_SCAN[页回收
        mm/vmscan.c]
    end

    subgraph "硬件层"
        MMU[MMU 硬件]
        TLB[TLB 缓存]
    end

    MMAP --> SC_MMAP
    MALLOC --> SC_BRK
    
    SC_MMAP --> VMA
    SC_BRK --> VMA
    
    ACCESS --> PG_FAULT
    PG_FAULT --> RMAP
    PG_FAULT --> PAGE_ALLOC
    
    VMA --> RMAP
    PAGE_ALLOC --> SLAB
    PAGE_ALLOC --> VM_SCAN
    
    PG_FAULT --> MMU
    PAGE_ALLOC --> MMU
    MMU --> TLB
```

### 2.3 文件系统数据流

```mermaid
flowchart TB
    subgraph "用户空间"
        OPEN[open 系统调用]
        READ[read 系统调用]
        WRITE[write 系统调用]
    end

    subgraph "VFS 层"
        VFS_OPEN[vfs_open
        fs/open.c]
        VFS_READ[vfs_read
        fs/read_write.c]
        VFS_WRITE[vfs_write
        fs/read_write.c]
        DCACHE[dentry 缓存
        fs/dcache.c]
        ICACHE[inode 缓存
        fs/inode.c]
    end

    subgraph "具体文件系统"
        EXT4[Ext4
        fs/ext4/]
        F2FS[F2FS
        fs/f2fs/]
        HMDFS[HMDFS
        fs/hmdfs/]
        PROC[procfs
        fs/proc/]
    end

    subgraph "页缓存"
        PAGECACHE[页缓存
        mm/pagecache]
        BIO[BIO 层
        fs/buffer.c]
    end

    subgraph "块层"
        BLOCK[块层
        block/]
        IO_SCHED[IO 调度器
        block/]
    end

    OPEN --> VFS_OPEN
    READ --> VFS_READ
    WRITE --> VFS_WRITE
    
    VFS_OPEN --> DCACHE --> ICACHE
    VFS_READ --> PAGECACHE
    VFS_WRITE --> PAGECACHE
    
    ICACHE --> EXT4 & F2FS & HMDFS & PROC
    PAGECACHE --> BIO
    BIO --> BLOCK
    BLOCK --> IO_SCHED
```

### 2.4 网络数据流

```mermaid
flowchart LR
    subgraph "用户空间"
        SOCK[socket 系统调用]
        SEND[send 系统调用]
        RECV[recv 系统调用]
    end

    subgraph "Socket 层"
        SYS_SOCKET[sys_socket
        net/socket.c]
        SYS_SEND[sys_send
        net/socket.c]
        SYS_RECV[sys_recv
        net/socket.c]
        SOCK_STRUCT[sock 结构
        net/core/sock.c]
    end

    subgraph "协议层"
        TCP[TCP
        net/ipv4/tcp.c]
        UDP[UDP
        net/ipv4/udp.c]
        IP[IP 层
        net/ipv4/ip_input.c]
        NF[Netfilter
        net/netfilter/]
    end

    subgraph "驱动层"
        DEV[网络设备
        net/core/dev.c]
        QDISC[队列规则
        net/sched/]
        NAPI[NAPI
        net/core/dev.c]
    end

    subgraph "硬件"
        NIC[网卡硬件]
    end

    SOCK --> SYS_SOCKET --> SOCK_STRUCT
    SEND --> SYS_SEND --> SOCK_STRUCT
    RECV --> SYS_RECV --> SOCK_STRUCT
    
    SOCK_STRUCT --> TCP & UDP
    TCP --> IP
    UDP --> IP
    IP --> NF
    NF --> DEV
    DEV --> QDISC
    QDISC --> NAPI
    NAPI --> NIC
```

## 3. 线程模型

### 3.1 调度器架构

```mermaid
graph TB
    subgraph "调度器核心"
        CORE[调度器核心
        kernel/sched/core.c]
        
        subgraph "调度类"
            STOP[Stop
            sched_class]
            DL[Deadline
            sched_class]
            RT[Real-Time
            sched_class]
            FAIR[Fair
            sched_class]
            IDLE[Idle
            sched_class]
        end
        
        CFS[完全公平调度器
        kernel/sched/fair.c]
        RTG[RTG 调度
        kernel/sched/rtg/]
        WALT[WALT
        kernel/sched/walt.c]
    end

    subgraph "任务管理"
        TASK[任务结构
        task_struct]
        RUNQ[运行队列
        rq]
        CPUMASK[CPU 亲和性
        cpumask]
    end

    subgraph "时间管理"
        TICK[时钟 tick
        kernel/time/]
        HRTIMER[高精度定时器
        kernel/time/hrtimer.c]
        CPUTIME[CPU 时间统计
        kernel/sched/cputime.c]
    end

    CORE --> STOP & DL & RT & FAIR & IDLE
    FAIR --> CFS
    CFS --> WALT
    RT --> RTG
    
    CORE --> TASK
    TASK --> RUNQ
    RUNQ --> CPUMASK
    
    CORE --> TICK
    TICK --> HRTIMER
    CORE --> CPUTIME
```

### 3.2 中断处理流程

```mermaid
sequenceDiagram
    participant HW as 硬件
    participant CPU as CPU
    participant ENTRY as 入口代码
    participant IRQ as IRQ 层
    participant HANDLER as 设备处理程序
    participant BH as 底半部
    participant USER as 用户空间

    HW->>CPU: 触发中断
    
    CPU->>ENTRY: 保存上下文
    Note over ENTRY: arch/arm64/kernel/entry.S
    
    ENTRY->>IRQ: handle_arch_irq
    Note over IRQ: kernel/irq/irqdesc.c
    
    IRQ->>IRQ: generic_handle_irq
    IRQ->>HANDLER: 调用设备处理程序
    
    alt 需要底半部处理
        HANDLER->>BH: 调度底半部
        Note over BH: softirq / tasklet / workqueue
        
        BH->>BH: 稍后执行
        BH->>USER: 唤醒等待进程
    else 直接完成
        HANDLER->>USER: 唤醒等待进程
    end
    
    HANDLER-->>IRQ: 返回
    IRQ-->>ENTRY: 返回
    ENTRY-->>CPU: 恢复上下文
```

### 3.3 工作队列模型

```mermaid
graph TB
    subgraph "工作队列系统"
        WORKQ[工作队列核心
        kernel/workqueue.c]
        
        subgraph "工作队列类型"
            BOUND[绑定队列
            per-CPU]
            UNBOUND[非绑定队列
            unbound]
            ORDERED[有序队列
            ordered]
        end
        
        POOL[worker pool
        线程池]
        WORKER[worker 线程]
    end

    subgraph "使用者"
        DRIVER[设备驱动]
        FS[文件系统]
        NET[网络子系统]
    end

    DRIVER --> WORKQ
    FS --> WORKQ
    NET --> WORKQ
    
    WORKQ --> BOUND & UNBOUND & ORDERED
    BOUND --> POOL
    UNBOUND --> POOL
    ORDERED --> POOL
    POOL --> WORKER
```

## 4. 内存映射关系

### 4.1 内核地址空间布局 (ARM64)

```mermaid
graph TB
    subgraph "ARM64 地址空间 (48-bit)"
        direction TB
        
        USER_END["0x0000_0000_0000_0000
用户空间结束"]
        VMALLOC["VMALLOC 区域
模块、vmalloc"]
        KMAP["KMAP 区域
永久映射"]
        FIXMAP["FIXMAP 区域
固定映射"]
        PCI_IO["PCI I/O 区域"]
        PCI_MEM["PCI 内存区域"]
        LINEAR["线性映射区域
物理内存直接映射"]
        KERNEL_TEXT["内核代码/数据
_text ~ _end"]
        
        USER_END --> VMALLOC
        VMALLOC --> KMAP
        KMAP --> FIXMAP
        FIXMAP --> PCI_IO
        PCI_IO --> PCI_MEM
        PCI_MEM --> LINEAR
        LINEAR --> KERNEL_TEXT
    end
```

### 4.2 物理内存管理

```mermaid
graph TB
    subgraph "物理内存管理"
        MEMBLOCK[Memblock
早期内存分配]
        
        subgraph "内存区域"
            DMA[DMA 区域
<16MB]
            NORMAL[NORMAL 区域
<4GB]
            HIGHMEM[HIGHMEM 区域
>4GB]
        end
        
        PAGE_ALLOC[页面分配器
buddy system]
        
        subgraph "SLAB 分配器"
            SLAB[SLAB
通用]
            SLUB[SLUB
默认]
            SLOB[SLOB
嵌入式]
        end
        
        VMALLOC[vmalloc
虚拟连续]
    end

    MEMBLOCK --> DMA & NORMAL & HIGHMEM
    DMA --> PAGE_ALLOC
    NORMAL --> PAGE_ALLOC
    HIGHMEM --> PAGE_ALLOC
    
    PAGE_ALLOC --> SLAB & SLUB & SLOB
    PAGE_ALLOC --> VMALLOC
```

## 5. 安全架构

### 5.1 LSM 框架

```mermaid
graph TB
    subgraph "LSM 框架"
        HOOK[安全 Hook 点
security/security.c]
        
        subgraph "安全模块"
            SELINUX[SELinux
security/selinux/]
            APPARMOR[AppArmor
security/apparmor/]
            SMACK[Smack
security/smack/]
            TOMOYO[TOMOYO
security/tomoyo/]
            YAMA[Yama
security/yama/]
            LOCKDOWN[Lockdown
security/lockdown/]
        end
        
        subgraph "完整性"
            IMA[IMA
security/integrity/ima/]
            EVM[EVM
security/integrity/evm/]
        end
    end

    subgraph "受保护资源"
        FILE[文件]
        TASK[进程]
        NET[网络]
        IPC[IPC]
    end

    FILE --> HOOK
    TASK --> HOOK
    NET --> HOOK
    IPC --> HOOK
    
    HOOK --> SELINUX & APPARMOR & SMACK & TOMOYO & YAMA & LOCKDOWN
    HOOK --> IMA & EVM
```

## 6. 设备驱动模型

### 6.1 驱动框架架构

```mermaid
graph TB
    subgraph "驱动核心"
        BUS[总线类型
bus_type]
        DEVICE[设备
struct device]
        DRIVER[驱动
struct device_driver]
        CLASS[设备类
struct class]
    end

    subgraph "总线类型"
        PLATFORM[Platform 总线]
        PCI[PCI 总线]
        USB[USB 总线]
        I2C[I2C 总线]
        SPI[SPI 总线]
    end

    subgraph "设备类型"
        CHAR[字符设备]
        BLOCK[块设备]
        NET[网络设备]
    end

    subgraph "设备树"
        DTB[DTB 二进制]
        OF[Open Firmware]
    end

    BUS --> PLATFORM & PCI & USB & I2C & SPI
    PLATFORM --> DEVICE
    PCI --> DEVICE
    USB --> DEVICE
    I2C --> DEVICE
    SPI --> DEVICE
    
    DEVICE --> DRIVER
    DEVICE --> CLASS
    CLASS --> CHAR & BLOCK & NET
    
    DTB --> OF
    OF --> DEVICE
```

---

*生成时间: 2026-02-06*
