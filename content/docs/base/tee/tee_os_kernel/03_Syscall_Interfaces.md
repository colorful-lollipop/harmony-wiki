# 系统调用接口

> **文档版本**: 1.0
> **生成时间**: 2026-02-06
> **适用范围**: OpenHarmony TEE OS Kernel 系统调用 API

---

## 文档目的

本文档提供 tee_os_kernel 系统调用的完整清单和说明。

**⚠️ 重要说明**：
- 本项目（tee_os_kernel）是 TEE 微内核，**不存在 N-API（Node-API）接口**
- 对外 API 通过 **系统调用（Syscall）** 和 **IPC 消息传递** 提供
- N-API 相关功能由 `tee_os_framework` 仓库提供（不在本仓库范围内）

---

## 目录

- [系统调用概览](#系统调用概览)
- [系统调用分类](#系统调用分类)
- [完整系统调用清单](#完整系统调用清单)
- [参数校验机制](#参数校验机制)
- [错误码定义](#错误码定义)
- [权限检查机制](#权限检查机制)
- [TEE 专用系统调用](#tee-专用系统调用)

---

## 系统调用概览

### 1.1 系统调用表

**定义位置**：`kernel/syscall/syscall.c`

**系统调用总数**：256 个（`NR_SYSCALL=256`）

**系统调用表结构**：
```c
// kernel/syscall/syscall.c
static const syscall_func_t syscall_table[NR_SYSCALL] = {
    [SYS_putstr] = sys_putstr,
    [SYS_getc] = sys_getc,
    [SYS_create_pmo] = sys_create_pmo,
    [SYS_map_pmo] = sys_map_pmo,
    // ... 256 个系统调用
};
```

**系统调用分发流程**：
```
用户态程序
    ↓ svc 指令
EL0 异常入口
    ↓ kernel/arch/aarch64/irq/irq_entry.c
EL1 内核处理
    ↓ kernel/syscall/syscall.c
syscall_table[syscall_num]()
    ↓ 执行系统调用逻辑
    ↓ 返回值（寄存器 x0）
EL1 → EL0 返回
    ↓ eret
用户态程序继续执行
```

---

## 系统调用分类

### 2.1 字符 I/O (0-1)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_putstr` | 0 | `char *str, size_t len` | void | 输出字符串到控制台 | `kernel/syscall/syscall.c:66-80` |
| `sys_getc` | 1 | 无 | int | 从控制台读取字符 | `kernel/syscall/syscall.c`（待查）|

### 2.2 内存管理 (10-31)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_create_pmo` | 10 | `size_t size, pmo_type_t type` | cap_t | 创建物理内存对象 | `kernel/object/memory.c` |
| `sys_create_device_pmo` | 11 | `paddr_t paddr, size_t size` | cap_t | 创建设备内存映射 | `kernel/object/memory.c` |
| `sys_map_pmo` | 12 | `cap_t target_group, cap_t pmo_cap, vaddr_t addr, vmr_prop_t perm, size_t len` | int | 映射 PMO 到地址空间 | `kernel/object/memory.c` |
| `sys_unmap_pmo` | 13 | `cap_t target_group, cap_t pmo_cap, vaddr_t addr, size_t len` | int | 解除 PMO 映射 | `kernel/object/memory.c` |
| `sys_write_pmo` | 14 | `cap_t pmo_cap, size_t offset, vaddr_t user_ptr, size_t len` | int | 写入 PMO 内容 | `kernel/object/memory.c` |
| `sys_read_pmo` | 15 | `cap_t pmo_cap, size_t offset, vaddr_t user_ptr, size_t len` | int | 读取 PMO 内容 | `kernel/object/memory.c` |
| `sys_get_phys_addr` | 30 | `vaddr_t va, paddr_t *pa_buf` | int | 获取虚拟地址的物理地址 | `kernel/object/memory.c` |
| `sys_create_ns_pmo` | 16 | `cap_t cap_group, unsigned long paddr, unsigned long size` | cap_t | 创建非安全 PMO | `kernel/object/memory.c:91-94` |
| `sys_destroy_ns_pmo` | 17 | `cap_t cap_group, cap_t pmo` | int | 销毁非安全 PMO | `kernel/object/memory.c:95-97` |
| `sys_create_tee_shared_pmo` | 19 | `cap_t cap_group, struct tee_uuid *uuid, size_t size, cap_t *self_cap` | cap_t | 创建 UUID 保护共享内存 | `kernel/object/memory.c:99-101` |
| `sys_transfer_pmo_owner` | 20 | `cap_t pmo, cap_t cap_group` | int | 转移 PMO 所有权 | `kernel/object/memory.c:101-103` |

### 2.3 能力管理 (18, 62)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_revoke_cap` | 18 | `cap_t obj_cap, bool revoke_copy` | int | 撤销能力（及副本）| `kernel/object/capability.c` |
| `sys_transfer_caps` | 62 | `cap_t dest_group, unsigned long src_caps_buf, int nr_caps, unsigned long dst_caps_buf` | int | 转移多个能力 | `kernel/object/capability.c` |

### 2.4 中断管理 (40-41)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_irq_op` | 40 | `cap_t irq_cap, unsigned long args` | int | 中断操作 | `kernel/object/irq.c` |
| `sys_irq_stop` | 41 | `cap_t irq_cap` | int | 停止中断 | `kernel/object/irq.c` |

### 2.5 多任务管理 (80-104)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_create_cap_group` | 80 | `unsigned long cap_group_args_p` | cap_t | 创建进程组（新进程）| `kernel/object/cap_group.c` |
| `sys_exit_group` | 81 | `int exitcode` | void | 退出进程组 | `kernel/object/cap_group.c` |
| `sys_create_thread` | 82 | `struct create_thread_args *args` | cap_t | 创建线程 | `kernel/object/thread.c` |
| `sys_thread_exit` | 83 | `int exitcode` | void | 退出线程 | `kernel/object/thread.c` |
| `sys_get_thread_id` | 84 | 无 | unsigned long | 获取线程 ID | `kernel/object/thread.c` |
| `sys_terminate_thread` | 85 | `cap_t thread_cap` | int | 终止线程 | `kernel/object/thread.c` |
| `sys_kill_group` | 86 | `int exitcode` | void | 杀死进程组 | `kernel/object/cap_group.c` |
| `sys_register_recycle` | 90 | `cap_t notifc_cap, vaddr_t msg_buffer` | int | 注册回收通知 | `kernel/object/recycle.c` |
| `sys_cap_group_recycle` | 91 | `int exitcode` | void | 回收进程组资源 | `kernel/object/recycle.c` |
| `sys_ipc_close_connection` | 92 | `cap_t conn_cap` | int | 关闭 IPC 连接 | `kernel/object/recycle.c` |
| `sys_yield` | 100 | 无 | void | 让出 CPU | `kernel/sched/sched.c` |
| `sys_set_affinity` | 101 | `int cpu` | int | 设置 CPU 亲和性 | `kernel/sched/sched.c` |
| `sys_get_affinity` | 102 | 无 | int | 获取 CPU 亲和性 | `kernel/sched/sched.c` |
| `sys_set_prio` | 103 | `unsigned int prio` | int | 设置线程优先级 | `kernel/sched/sched.c` |
| `sys_get_prio` | 104 | 无 | unsigned int | 获取线程优先级 | `kernel/sched/sched.c` |

### 2.6 IPC (120-146)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_register_server` | 120 | `unsigned long ipc_routine, cap_t register_thread_cap` | int | 注册 IPC 服务器 | `kernel/ipc/connection.c` |
| `sys_register_client` | 121 | `cap_t server_cap` | int | 注册 IPC 客户端 | `kernel/ipc/connection.c` |
| `sys_ipc_register_cb_return` | 122 | `unsigned long register_thread_cap` | int | IPC 注册回调返回 | `kernel/ipc/connection.c` |
| `sys_ipc_call` | 123 | `cap_t conn_cap, struct ipc_msg *msg_in_client` | int | IPC 同步调用 | `kernel/ipc/connection.c` |
| `sys_ipc_return` | 124 | `int ret_val` | void | IPC 同步返回 | `kernel/ipc/connection.c` |
| `sys_ipc_exit_routine_return` | 125 | `int ret_val` | void | IPC 例程退出返回 | `kernel/ipc/connection.c` |
| `sys_create_notifc` | 130 | 无 | cap_t | 创建通知对象 | `kernel/ipc/notification.c` |
| `sys_wait` | 131 | `cap_t notifc_cap` | int | 等待通知 | `kernel/ipc/notification.c` |
| `sys_notify` | 132 | `cap_t notifc_cap` | int | 发送通知 | `kernel/ipc/notification.c` |

### 2.7 中断与异常 (150-166)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_irq_register` | 150 | `int irq` | cap_t | 注册中断处理 | `kernel/object/irq.c` |
| `sys_irq_wait` | 151 | `cap_t notifc_cap` | int | 等待中断 | `kernel/object/irq.c` |
| `sys_irq_ack` | 152 | `cap_t notifc_cap` | int | 应答中断 | `kernel/object/irq.c` |
| `sys_disable_irq` | 153 | `cap_t notifc_cap` | int | 禁用中断 | `kernel/object/irq.c` |
| `sys_enable_irq` | 154 | `cap_t notifc_cap` | int | 启用中断 | `kernel/object/irq.c` |
| `sys_disable_irqno` | 155 | `int irqno` | int | 禁用指定中断号 | `kernel/object/irq.c` |
| `sys_enable_irqno` | 156 | `int irqno` | int | 启用指定中断号 | `kernel/object/irq.c` |
| `sys_clear_irqno` | 157 | `int irqno` | int | 清除指定中断号 | `kernel/object/irq.c` |
| `sys_cache_config` | 158 | `unsigned long args` | int | 缓存配置 | `kernel/arch/aarch64/plat/*/irq/irq.c` |
| `sys_disable_local_irq` | 159 | 无 | void | 禁用本地中断 | `kernel/arch/aarch64/plat/*/irq/irq.c` |
| `sys_enable_local_irq` | 160 | 无 | void | 启用本地中断 | `kernel/arch/aarch64/plat/*/irq/irq.c` |
| `sys_user_fault_register` | 165 | `vaddr_t handler_func` | cap_t | 注册用户态错误处理 | `kernel/object/user_fault.c` |
| `sys_user_fault_map` | 166 | `badge_t client_badge, vaddr_t fault_va, vaddr_t remap_va` | int | 映射用户态错误地址 | `kernel/object/user_fault.c` |

### 2.8 硬件访问 (180-185)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_cache_flush` | 180 | `vaddr_t start, size_t size` | int | 刷新缓存 | `user/chcore-libs/sys-libs/libohtee/hm_cache_flush.c` |
| `sys_get_current_tick` | 185 | 无 | unsigned long | 获取当前时钟滴答 | `kernel/irq/timer.c` |

### 2.9 POSIX 兼容 (200-213)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_clock_gettime` | 200 | `clockid_t clk_id, struct timespec *tp` | int | 获取时间 | `kernel/irq/timer.c` |
| `sys_clock_nanosleep` | 201 | `clockid_t clk_id, struct timespec *req` | int | 纳秒睡眠 | `kernel/irq/timer.c` |
| `sys_handle_brk` | 210 | `unsigned long addr, unsigned long heap_start` | unsigned long | 调整堆 | `kernel/object/memory.c` |
| `sys_handle_mprotect` | 213 | `vaddr_t addr, size_t length, int prot` | int | 修改内存保护 | `kernel/object/memory.c` |

### 2.10 调试与性能 (220-234)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_debug_log` | 220 | `const char *str, size_t len` | void | 调试日志输出 | `kernel/syscall/syscall.c` |
| `sys_top` | 221 | 无 | void | 显示进程/线程列表 | `kernel/sched/sched.c` |
| `sys_get_free_mem_size` | 222 | 无 | unsigned long | 获取空闲内存 | `kernel/mm/mm.c` |
| `sys_get_mem_usage_msg` | 223 | `void *info, int print_history` | int | 获取内存使用信息 | `kernel/mm/mm.c` |
| `sys_perf_start` | 230 | 无 | void | 性能测试开始 | `kernel/syscall/syscall.c` |
| `sys_perf_end` | 231 | 无 | void | 性能测试结束 | `kernel/syscall/syscall.c` |
| `sys_perf_null` | 232 | 无 | void | 性能测试空操作 | `kernel/syscall/syscall.c` |
| `sys_get_pci_device` | 233 | `unsigned long pci_addr, void *buf, size_t len` | int | 获取 PCI 设备信息 | `kernel/syscall/syscall.c` |
| `sys_poweroff` | 234 | 无 | void | 关机 | `kernel/include/common/poweroff.h` |

### 2.11 虚拟化 (240)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_virt_dispatch` | 240 | `unsigned long args` | int | 虚拟化分发 | `kernel/syscall/syscall.c` |

---

## TEE 专用系统调用

### 3.1 TEE 消息通道 (140-146)

这些系统调用仅当 `CHCORE_OH_TEE=ON` 时可用。

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_tee_msg_create_msg_hdl` | 140 | 无 | cap_t | 创建消息句柄 | `kernel/ipc/channel.c:94-95` |
| `sys_tee_msg_create_channel` | 141 | `unsigned long args` | cap_t | 创建消息通道 | `kernel/ipc/channel.c:96-97` |
| `sys_tee_msg_stop_channel` | 142 | `int channel_cap` | int | 停止消息通道 | `kernel/ipc/channel.c:98-99` |
| `sys_tee_msg_receive` | 143 | `int channel_cap, void *recv_buf, size_t recv_len, int msg_hdl_cap, void *info, int timeout` | int | 接收消息 | `kernel/ipc/channel.c:100-102` |
| `sys_tee_msg_call` | 144 | `int channel_cap, void *send_buf, size_t send_len, void *recv_buf, size_t recv_len, struct timespec *timeout` | int | 调用消息 | `kernel/ipc/channel.c:103-105` |
| `sys_tee_msg_reply` | 145 | `int msg_hdl_cap, void *reply_buf, size_t reply_len` | int | 回复消息 | `kernel/ipc/channel.c:106-107` |
| `sys_tee_msg_notify` | 146 | `int channel_cap, void *send_buf, size_t send_len` | int | 通知消息 | `kernel/ipc/channel.c:108-110` |

**关键结构**（`kernel/include/ipc/channel.h`）：
```c
struct channel {
    struct list_head thread_queue;    // 等待的服务器线程队列
    struct list_head msg_queue;     // 消息队列
    struct lock lock;
    struct cap_group *creater;       // 创建者
    int state;                     // 通道状态
};

struct msg_hdl {
    struct list_head thread_queue_node;
    struct client_msg_record client_msg_record;
    struct server_msg_record server_msg_record;
    struct lock lock;
};
```

### 3.2 TrustZone SMC 接口 (250-253)

| 系统调用 | 编号 | 参数 | 返回值 | 用途 | 实现文件 |
|----------|------|------|--------|------|----------|
| `sys_tee_wait_switch_req` | 250 | 无 | int | 等待切换请求 | `kernel/arch/aarch64/trustzone/spd/teed/smc.c` |
| `sys_tee_switch_req` | 251 | `unsigned long args` | int | 切换到安全世界 | `kernel/arch/aarch64/trustzone/spd/teed/smc.c` |
| `sys_tee_create_ns_pmo` | 252 | `unsigned long paddr, unsigned long size` | cap_t | 创建非安全 PMO | `kernel/object/memory.c:94-98` |
| `sys_tee_pull_kernel_var` | 253 | `kernel_shared_varibles_t *pVar` | int | 拉取内核变量 | `kernel/arch/aarch64/trustzone/spd/teed/smc.c` |

---

## 参数校验机制

### 4.1 用户态地址验证

**实现位置**：`kernel/include/mm/uaccess.h`

**关键函数**：
```c
// 检查地址是否在用户空间范围内
int check_user_addr_range(vaddr_t addr, size_t len);

// 安全地从用户空间复制数据
int copy_from_user(void *kernel_dst, void *user_src, size_t len);

// 安全地向用户空间复制数据
int copy_to_user(void *user_dst, void *kernel_src, size_t len);
```

**地址空间布局**：
```
用户空间: 0x0000_0000_0000 - 0x7FFF_FFFF_FFFF (0 ~ 2^47-1)
内核空间: 0xFFFF_FF00_0000_0000+ (KBASE)
```

### 4.2 Capability 验证

**实现位置**：`kernel/object/capability.c`

**验证机制**：
- Capability ID 槽位有效性检查
- Capability 类型匹配检查
- 拥有者 Cap_group 检查

**关键函数**：
```c
void *obj_get(struct cap_group *cap_group, cap_t slot_id, int type);
void obj_put(void *obj);
int cap_free(struct cap_group *cap_group, cap_t slot_id);
```

### 4.3 权限钩子

**实现位置**：`kernel/syscall/syscall_hooks.c`

**Badge 授权检查**：
```c
// hook_sys_create_device_pmo - 仅驱动允许
// hook_sys_get_phys_addr - badge 范围检查
// hook_sys_create_cap_group - 仅 ROOT/FSM/PROCMGR 允许
```

**固定 Badge**：
```c
// kernel/include/object/cap_group.h:131-138
#define ROOT_CAP_GROUP_BADGE (1)  // PROCMGR (init 进程）
#define FSM_BADGE            (2)  // 文件系统管理器
#define LWIP_BADGE           (3)  // 网络栈
#define TMPFS_BADGE          (4)  // 临时文件系统
#define SERVER_BADGE_START   (5)  // 动态服务起点
#define DRIVER_BADGE_START   (100) // 设备驱动起点
#define APP_BADGE_START      (200) // 应用程序起点
```

---

## 错误码定义

**定义位置**：`kernel/include/common/errno.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `EBADSYSCALL` | 未实现的系统调用 |
| `ECAPBILITY` | 能力无效或不足 |
| `EINVAL` | 无效参数 |
| `ENOMEM` | 内存不足 |
| `EAGAIN` | 资源暂时不可用 |
| `EIDLE` | 资源空闲 |

---

## IPC 使用模式

### 5.1 标准 IPC (ChCore Connection）

**注册流程**：
```
Server Thread: sys_register_server(ipc_routine, register_thread_cap)
    ↓
Client Thread: sys_register_client(server_cap)
    ↓
系统调用调度线程: sys_ipc_register_cb_return(handler_thread)
    ↓
返回连接 cap 给客户端
```

**数据流程**：
```
Client Thread:
    sys_ipc_call(conn_cap, ipc_msg)
    ↓ 直接上下文切换到 Server Handler
Server Handler:
    处理请求
    ↓
    sys_ipc_return(ret_val)
    ↓
    上下文切换回 Client Thread
```

### 5.2 TEE IPC Channel

**接收流程**：
```
Server:
    sys_tee_msg_create_channel()  // 创建通道
    sys_tee_msg_receive(channel, buf, msg_hdl)  // 阻塞接收
    ↓
    sys_tee_msg_reply(msg_hdl, reply_buf)  // 回复
```

**调用流程**：
```
Client:
    sys_tee_msg_call(channel, send_buf, recv_buf, timeout)
    ↓ 阻塞等待回复
```

---

## 相关跳转

- [架构设计](02_Architecture.md) - 组件图、数据流
- [内部 API](04_Internal_APIs.md) - 模块接口与依赖
- [安全评审](07_Security_Review.md) - 权限检查机制

---

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2026-02-06
