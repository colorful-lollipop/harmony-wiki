# 关键调用链分析

## 概述

本文档分析 OpenHarmony Linux Kernel 5.10 中关键的调用链，从用户空间入口到核心逻辑的实现。

## 1. 系统调用调用链

### 1.1 进程创建 (fork/clone)

```
用户空间
  ↓
sys_clone() [libc wrapper]
  ↓
SYSCALL_DEFINE5(clone, ...) [kernel/fork.c:2627]
  ↓
kernel_clone() [kernel/fork.c]
  ↓
  ├─ copy_process() [kernel/fork.c]
  │    ├─ dup_task_struct()
  │    ├─ copy_files()
  │    ├─ copy_fs()
  │    ├─ copy_sighand()
  │    ├─ copy_mm()
  │    │   └─ dup_mm() [mm/memory.c]
  │    │       └─ copy_page_range()
  │    └─ copy_thread() [arch/arm64/kernel/process.c]
  │
  └─ wake_up_new_task() [kernel/sched/core.c]
       └─ activate_task()
            └─ enqueue_task()
```

### 1.2 execve 执行程序

```
用户空间
  ↓
sys_execve()
  ↓
SYSCALL_DEFINE3(execve, ...) [fs/exec.c]
  ↓
do_execveat_common() [fs/exec.c]
  ↓
  ├─ search_binary_handler() [fs/exec.c]
  │    └─ load_binary() [fs/binfmt_elf.c]
  │         ├─ load_elf_binary()
  │         │   ├─ kernel_read() - 读取 ELF 头
  │         │   ├─ elf_map() - 映射 PT_LOAD 段
  │         │   ├─ create_elf_tables() - 设置参数
  │         │   └─ start_thread() - 开始执行
  │
  └─ begin_new_exec() [fs/exec.c]
       └─ setup_new_exec()
```

### 1.3 内存映射 (mmap)

```
用户空间
  ↓
sys_mmap()
  ↓
SYSCALL_DEFINE6(mmap, ...) [mm/mmap.c]
  ↓
ksys_mmap_pgoff() [mm/mmap.c]
  ↓
vm_mmap_pgoff() [mm/util.c]
  ↓
do_mmap() [mm/mmap.c]
  ↓
  ├─ get_unmapped_area() - 查找虚拟地址
  ├─ mmap_region() [mm/mmap.c]
  │    ├─ vma_merge() - 尝试合并 VMA
  │    ├─ kmem_cache_zalloc() - 分配 VMA
  │    └─ call_mmap() [mm/mmap.c]
  │         └─ file->f_op->mmap() - 调用具体映射
  │              └─ 如: generic_file_mmap() [mm/filemap.c]
  │
  └─ perf_event_mmap()
```

## 2. 文件系统调用链

### 2.1 文件打开 (open)

```
用户空间
  ↓
sys_openat()
  ↓
do_sys_openat2() [fs/open.c]
  ↓
build_open_flags() - 构建打开标志
  ↓
do_filp_open() [fs/namei.c]
  ↓
path_openat() [fs/namei.c]
  ↓
  ├─ path_init() - 初始化路径
  ├─ link_path_walk() - 路径遍历
  │    └─ walk_component() [fs/namei.c]
  │         └─ lookup_fast() / lookup_slow()
  │              └─ inode->i_op->lookup()
  │
  ├─ do_last() [fs/namei.c]
  │    ├─ lookup_open()
  │    ├─ vfs_open() [fs/open.c]
  │    │    └─ do_dentry_open()
  │    │         ├─ file->f_op = fops_get(inode->i_fop)
  │    │         └─ open()
  │    └─ may_open()
  │
  └─ path_to_nameidata()
```

### 2.2 文件读取 (read)

```
用户空间
  ↓
ksys_read() [fs/read_write.c]
  ↓
vfs_read() [fs/read_write.c]
  ↓
  ├─ rw_verify_area() - 验证读写区域
  └─ file->f_op->read_iter() / new_sync_read()
       └─ generic_file_read_iter() [mm/filemap.c]
            ├─ filemap_read() [mm/filemap.c]
            │    ├─ pagecache_get_page() - 获取页缓存
            │    ├─ copy_page_to_iter() - 拷贝数据
            │    └─ mark_page_accessed()
            │
            └─ call_read_iter()
```

### 2.3 页面缓存读取

```
filemap_read() [mm/filemap.c]
  ↓
  ├─ find_get_pages_range_tag() - 查找页缓存
  │    └─ xas_find()
  │
  ├─ 如果未缓存:
  │    └─ page_cache_read_unbounded() [mm/filemap.c]
  │         └─ file->f_mapping->a_ops->readpage()
  │              └─ 如: ext4_readpage() [fs/ext4/inode.c]
  │                   └─ mpage_readpage()
  │                        └─ submit_bio()
  │
  └─ copy_page_to_iter()
       └
```

## 3. 网络调用链

### 3.1 Socket 创建

```
sys_socket() [net/socket.c]
  ↓
__sys_socket() [net/socket.c]
  ↓
  ├─ sock_create() [net/socket.c]
  │    └─ __sock_create()
  │         ├─ sock_alloc() - 分配 inode
  │         ├─ pf = rcu_dereference(net_families[family])
  │         └─ pf->create() [net/ipv4/af_inet.c]
  │              └─ inet_create()
  │                   ├─ sk_alloc() - 分配 sock
  │                   ├─ sock_init_data()
  │                   └─ sk->sk_prot->init()
  │                        └─ tcp_v4_init_sock() / udp_init_sock()
  │
  └─ sock_map_fd() - 关联文件描述符
```

### 3.2 TCP 发送

```
sys_sendto() / sys_sendmsg()
  ↓
sock_sendmsg() [net/socket.c]
  ↓
inet_sendmsg() [net/ipv4/af_inet.c]
  ↓
tcp_sendmsg() [net/ipv4/tcp.c]
  ↓
  ├─ tcp_sendmsg_locked()
  │    ├─ sk_stream_alloc_skb() - 分配 SKB
  │    ├─ skb_add_data_nocache() - 拷贝数据
  │    ├─ tcp_push()
  │    │    └─ __tcp_push_pending_frames()
  │    │         └─ tcp_write_xmit()
  │    │              ├─ tcp_transmit_skb()
  │    │              │    ├─ ip_queue_xmit() [net/ipv4/ip_output.c]
  │    │              │    │    └─ ip_local_out()
  │    │              │    │         └─ ip_output()
  │    │              │    │              └─ ip_finish_output()
  │    │              │    │                   └─ ip_finish_output2()
  │    │              │    │                        └─ neigh_output()
  │    │              │    │                             └─ dev_queue_xmit()
  │    │              │    │                                  └...
```

### 3.3 网络数据包接收

```
网卡中断
  ↓
irq_handler()
  ↓
napi_schedule() / napi_schedule_irqoff()
  ↓
软中断 NET_RX_SOFTIRQ
  ↓
net_rx_action() [net/core/dev.c]
  ↓
napi->poll() - 如: igb_poll()
  ↓
napi_skb_finish() / napi_gro_receive()
  ↓
netif_receive_skb_internal() [net/core/dev.c]
  ↓
__netif_receive_skb()
  ↓
  ├─ packet_type->func() [ptype]
  │    └─ ip_rcv() [net/ipv4/ip_input.c]
  │         ├─ ip_rcv_finish()
  │         │    ├─ ip_route_input_noref()
  │         │    └─ dst_input()
  │         │         └─ ip_local_deliver()
  │         │              └─ ip_protocol_deliver_rcu()
  │         │                   └─ tcp_v4_rcv() / udp_rcv()
  │
  └─ deliver_skb()
```

## 4. 调度器调用链

### 4.1 调度器入口

```
时钟中断 / 系统调用返回
  ↓
schedule() [kernel/sched/core.c]
  ↓
__schedule() [kernel/sched/core.c]
  ↓
  ├─ sched_submit_work()
  ├─ pick_next_task() [kernel/sched/core.c]
  │    ├─ for_each_class(class) {
  │    │    next = class->pick_next_task()
  │    │    // Stop → Deadline → RT → Fair → Idle
  │    │}
  │    └─ pick_next_task_fair() [kernel/sched/fair.c]
  │         ├─ pick_next_entity() - 选择下一个 CFS 任务
  │         └─ set_next_entity()
  │
  ├─ context_switch() [kernel/sched/core.c]
  │    ├─ prepare_task_switch()
  │    ├─ arch_start_context_switch()
  │    ├─ mm/pgd 切换
  │    └─ switch_to() - 汇编切换
  │
  └─ finish_task_switch()
```

### 4.2 CFS 任务选择

```
pick_next_task_fair() [kernel/sched/fair.c]
  ↓
  ├─ pick_next_entity() [kernel/sched/fair.c]
  │    └─ __pick_next_entity() [kernel/sched/fair.c]
  │         └─ cfs_rq->rb_leftmost
  │              // 选择 vruntime 最小的任务
  │
  └─ set_next_entity()
       └─ update_stats_curr_start()
```

## 5. 内存分配调用链

### 5.1 kmalloc

```
kmalloc() [include/linux/slab.h]
  ↓
__kmalloc() [mm/slab.c 或 mm/slub.c]
  ↓
  ├─ 根据大小选择:
  │    ├─ 小对象: kmem_cache_alloc()
  │    │    └─ slab_alloc() / slub_alloc()
  │    │         ├─ get_slab() / get_partial()
  │    │         └─ new_slab() - 如果无可用
  │    │              └─ alloc_slab_page()
  │    │                   └─ alloc_pages()
  │    │                        └─ __alloc_pages_nodemask()
  │    │                             └─ get_page_from_freelist()
  │    │                                  └─ rmqueue()
  │    │
  │    └─ 大对象: __kmalloc_large()
  │         └─ alloc_pages()
```

### 5.2 页面分配

```
__alloc_pages_nodemask() [mm/page_alloc.c]
  ↓
  ├─ get_page_from_freelist() - 快速路径
  │    ├─ prepare_alloc_pages()
  │    ├─ for_each_zone_zonelist_nodemask()
  │    │    └─ rmqueue()
  │    │         └─ __rmqueue_smallest() / __rmqueue()
  │    └─ prep_new_page()
  │
  └─ __alloc_pages_slowpath() - 慢速路径
       ├─ wake_all_kswapds() - 唤醒回收守护进程
       ├- try_to_free_pages() - 直接回收
       └─ __alloc_pages_direct_compact() - 内存压缩
```

## 6. 中断处理调用链

### 6.1 硬件中断处理

```
硬件中断触发
  ↓
CPU 保存上下文
  ↓
IRQ handler [arch/arm64/kernel/entry.S]
  ↓
handle_arch_irq() [drivers/irqchip/irq-gic-v3.c]
  ↓
gic_handle_irq()
  ├─ 读取 IAR 获取中断号
  ├─ handle_domain_irq() [kernel/irq/irqdesc.c]
  │    └─ __handle_domain_irq()
  │         ├─ irq_resolve_mapping()
  │         └─ generic_handle_irq() [kernel/irq/irqdesc.c]
  │              └─ desc->handle_irq() [irq_flow_handler_t]
  │                   └─ handle_level_irq() / handle_edge_irq()
  │                        └─ handle_irq_event() [kernel/irq/handle.c]
  │                             └─ action->handler() [设备中断处理程序]
  │
  └─ 写 EOI 结束中断
```

### 6.2 软中断处理

```
irq_exit() / local_bh_enable()
  ↓
do_softirq() [kernel/softirq.c]
  ↓
__do_softirq() [kernel/softirq.c]
  ↓
  ├─ while (softirq_pending()) {
  │    h = softirq_vec + i
  │    h->action() - 如: net_rx_action(), tasklet_action()
  │}
  │
  └─ 如果处理过多，唤醒 ksoftirqd
       └─ wake_up_process(ksoftirqd)
```

## 7. OpenHarmony 特有调用链

### 7.1 HiLog 写入

```
内核代码调用 printk() / dev_vprintk_emit()
  ↓
console_unlock() [kernel/printk/printk.c]
  ↓
  ├─ 标准 console 输出
  │
  └─ hilog_console_write() [drivers/staging/hilog/hilog.c]
       └─ hilog_ring_buffer_write()
            ├─ 写入环形缓冲区
            └─ 唤醒读取进程

用户空间读取
  ↓
read() on /dev/hilog
  ↓
hilog_read() [drivers/staging/hilog/hilog.c]
  └─ hilog_ring_buffer_read()
```

### 7.2 Blackbox 崩溃收集

```
触发点: panic() / oops
  ↓
bbox_notify_error(EVENT_PANIC) [drivers/staging/blackbox/]
  ↓
  ├─ 收集系统信息
  │    ├─ 寄存器状态
  │    ├─ 调用栈
  │    ├─ 内存使用
  │    └─ 任务信息
  │
  ├─ 写入 blackbox 存储区域
  │    └─ 持久化存储
  │
  └─ 通知用户空间
       └─ netlink / sysfs 通知
```

### 7.3 ZeroHung 冻结检测

```
定时器触发 (定期检测)
  ↓
watchdog_timer_fn() [kernel/watchdog.c] / 
zerohung_check() [drivers/staging/zerohung/]
  ↓
  ├─ 检查各 CPU 响应
  ├─ 检查任务调度延迟
  └─ 检查特定任务状态
       ├─ 如果检测到冻结:
       │    ├─ 收集诊断信息
       │    ├─ 写入日志
       │    └─ 可选择触发 panic
       │
       └─ 否则继续监控
```

---

*生成时间: 2026-02-06*
