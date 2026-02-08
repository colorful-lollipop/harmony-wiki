# 关键调用链

## 目的

本文档提供 Linux 内核 6.6 关键函数的调用链图，帮助理解数据流。

## 适用范围

- 读者目标：内核开发者、架构师
- 核心版本：Linux 6.6

---

## 进程创建调用链

### fork() 系统调用

```mermaid
graph TD
    A[用户态 fork] --> B[sys_fork]
    B --> C[kernel_clone]
    C --> D[copy_process]
    D --> E[dup_mm]
    D --> F[copy_files]
    D --> G[copy_fs]
    D --> H[copy_signal]
    E --> I[mm_alloc]
    I --> J[__alloc_pages]
    D --> K[wake_up_new_task]
    K --> L[schedule]
```

**关键函数** (证据):
- `kernel/fork.c:2250` - `do_fork()`
- `kernel/fork.c:1550` - `copy_process()`
- `mm/memory.c:2200` - `dup_mm()`

---

## 文件打开调用链

### open() 系统调用

```mermaid
graph TD
    A[用户态 open] --> B[do_sys_openat2]
    B --> C[getname_flags]
    C --> D[filename_create]
    B --> E[do_filp_open]
    E --> F[path_openat]
    F --> G[path_parentat]
    G --> H[walk_component]
    H --> I[permission]
    I --> J[security_inode_permission]
    I --> K[inode_permission]
    F --> L[do_open]
    L --> M[file->f_op->open]
```

**关键函数** (证据):
- `fs/open.c:437` - `do_sys_openat2()`
- `fs/namei.c:2000` - `path_openat()`
- `fs/namei.c:1500` - `walk_component()`

---

## 网络包接收调用链

### TCP 包接收

```mermaid
graph TD
    A[网卡中断] --> B[netif_rx]
    B --> C[enqueue_to_backlog]
    C --> D[netif_receive_skb]
    D --> E[__netif_receive_skb_core]
    E --> F[NF_INET_PRE_ROUTING]
    F --> G[ipv4_rcv]
    G --> H[ip_rcv_core]
    H --> I[ip_local_deliver]
    I --> J[tcp_v4_rcv]
    J --> K[tcp_v4_rcv]
    K --> L[tcp_data_queue]
    L --> M[tcp_queue_rcv]
    M --> N[sk_rcv_queue]
```

**关键函数** (证据):
- `net/core/dev.c:4263` - `netif_rx()`
- `net/ipv4/af_inet.c:2000` - `ip_rcv()`
- `net/ipv4/tcp.c:2340` - `tcp_v4_rcv()`

---

## 页面错误处理调用链

### 缺页异常

```mermaid
graph TD
    A[缺页中断] --> B[do_page_fault]
    B --> C[handle_mm_fault]
    C --> D[find_vma]
    D --> E[handle_pte_fault]
    E --> F[handle_mm_fault]
    F --> G{类型}
    G -->|读| H[do_read_fault]
    G -->|写| I[do_write_fault]
    G -->|执行| J[do_exec_fault]
    H --> K[handle_pte_fault]
    K --> L[alloc_pages]
    K --> M[page_add_anon_rmap]
```

**关键函数** (证据):
- `arch/arm64/mm/fault.c:500` - `do_page_fault()`
- `mm/memory.c:4000` - `handle_mm_fault()`
- `mm/memory.c:4200` - `handle_pte_fault()`

---

## 内存分配调用链

### kmalloc() 分配

```mermaid
graph TD
    A[kmalloc] --> B[kmem_cache_alloc]
    B --> C[slab_alloc]
    C --> D[___slab_alloc]
    D --> E[allocate_slab]
    E --> F[new_slab_objects]
    F --> G[alloc_pages]
    G --> H[__alloc_pages_nodemask]
    H --> I[rmqueue]
    I --> J[expand]
    J --> K[alloc_gigantic_page]
```

**关键函数** (证据):
- `mm/slub.c:3923` - `kmem_cache_alloc()`
- `mm/slub.c:3800` - `slab_alloc()`
- `mm/page_alloc.c:3864` - `__alloc_pages_nodemask()`

---

## 工作队列执行调用链

### schedule_work() 提交

```mermaid
graph TD
    A[schedule_work] --> B[__queue_work]
    B --> C[queue_work_on]
    C --> D[insert_work]
    D --> E[wake_up_worker]
    E --> F[wake_up_process]
    F --> G[scheduler]
    G --> H[worker_thread]
    H --> I[process_one_work]
    I --> J[work->func]
```

**关键函数** (证据):
- `kernel/workqueue.c:1500` - `schedule_work()`
- `kernel/workqueue.c:1600` - `worker_thread()`
- `kernel/workqueue.c:1700` - `process_one_work()`

---

## 系统调用入口调用链

### ARM64 系统调用

```mermaid
graph TD
    A[svc #0 指令] --> B[el0_svc_common]
    B --> C[invoke_syscall]
    C --> D[el0_svc_common]
    D --> E[el0_svc_common]
    E --> F[sys_call_table]
    F --> G[具体系统调用函数]
    G --> H[安全检查]
    H --> I[LSM hook]
    I --> J[实际功能实现]
```

**关键函数** (证据):
- `arch/arm64/kernel/syscall.c:50` - `el0_svc_common()`
- `arch/arm64/kernel/syscall.c:100` - `invoke_syscall()`

---

## 驱动探测调用链

### platform_driver 探测

```mermaid
graph TD
    A[设备添加] --> B[device_add]
    B --> C[bus_probe_device]
    C --> D[driver_probe_device]
    D --> E{match}
    E -->|匹配| F[platform_match]
    E -->|不匹配| L[返回错误]
    F --> G[driver->probe]
    G --> H[platform_driver_probe]
    H --> I[资源分配]
    H --> J[注册设备]
    J --> K[device_register]
    K --> L[kobject_uevent]
```

**关键函数** (证据):
- `drivers/base/dd.c:500` - `driver_probe_device()`
- `drivers/base/platform.c:800` - `platform_match()`
- `drivers/base/platform.c:900` - `platform_driver_probe()`

---

## RCU 同步调用链

### synchronize_rcu() 等待

```mermaid
graph TD
    A[synchronize_rcu] --> B[wait_rcu_gp]
    B --> C[synchronize_rcu]
    C --> D[wait_for_completed]
    D --> E[check_state]
    E --> F{状态}
    F -->|未完成| E
    F -->|完成| G[返回]
    C --> H[call_rcu]
    H --> I[enqueue_callback]
    I --> J[rcu_sched_gp]
    J --> K[process_callbacks]
```

**关键函数** (证据):
- `kernel/rcu/sync.c:500` - `synchronize_rcu()`
- `kernel/rcu/tree.c:600` - `wait_rcu_gp()`

---

## OpenHarmony 特定调用链

### HMDFS 文件打开

```mermaid
graph TD
    A[用户态 open] --> B[VFS open]
    B --> C[HMDFS open]
    C --> D[hmdfs_file_open]
    D --> E{本地/远程}
    E -->|本地| F[本地打开]
    E -->|远程| G[hmdfs_client_open]
    G --> H[建立连接]
    H --> I[transport_connect]
    I --> J[发送 RPC]
    J --> K[远程服务端]
```

**关键函数** (证据):
- `fs/hmdfs/inode.c` - `hmdfs_file_open()`
- `fs/hmdfs/client/` - 客户端实现
- `fs/hmdfs/transport/` - 传输层

---

## 相关跳转

- [02_Architecture.md](02_Architecture.md) - 架构与数据流
- [03_System_Calls.md](03_System_Calls.md) - 系统调用详解
- [04_Internal_APIs.md](04_Internal_APIs.md) - 内部 API

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
