# 内部 API

## 目的

本文档梳理 Linux 内核 6.6 的内部子系统 API，包括模块接口、依赖方向和稳定性标注。

## 适用范围

- 读者目标：内核开发者、模块开发者
- 核心版本：Linux 6.6

---

## API 稳定性分级

| 级别 | 说明 | 示例 |
|--------|------|------|
| **稳定 (Stable)** | 不向后兼容修改的 API | `kmalloc()`, `schedule()` |
| **内部 (Internal)** | 仅内核内部使用 | `__alloc_pages_nodemask()` |
| **导出符号 (EXPORT_SYMBOL)** | 供模块使用的 API | `printk()`, `request_irq()` |

**判断依据**:
- `EXPORT_SYMBOL()` / `EXPORT_SYMBOL_GPL()` - 模块可用
- 包含头文件层级 (`include/linux/` vs `kernel/`)
- 函数命名约定（`__` 前缀 = 内部）

---

## 调度器 API (kernel/sched/)

### 稳定接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `schedule()` | `include/linux/sched.h` | 主动调度 |
| `wake_up_process()` | `include/linux/sched.h` | 唤醒进程 |
| `yield()` | `include/linux/sched.h` | 让出 CPU |
| `set_current_state()` | `include/linux/sched.h` | 设置进程状态 |

### 内部接口

| 函数 | 位置 | 功能 |
|------|------|------|
| `__schedule()` | `kernel/sched/core.c` | 调度器内部实现 |
| `pick_next_task()` | `kernel/sched/core.c` | 选择下一个进程 |
| `context_switch()` | `kernel/sched/core.c` | 进程上下文切换 |

**依赖方向**:
- 被依赖: 所有需要睡眠/让出 CPU 的代码
- 依赖: 内存管理（task_struct 分配）、中断管理（时钟中断）

**证据**:
- `kernel/sched/core.c:6556` - `schedule()`
- `include/linux/sched.h:687` - 稳定接口声明

---

## 内存管理 API (mm/)

### 稳定接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `kmalloc()` | `include/linux/slab.h` | 小内存分配 |
| `kfree()` | `include/linux/slab.h` | 释放内存 |
| `vmalloc()` | `include/linux/vmalloc.h` | 虚拟内存分配 |
| `vfree()` | `include/linux/vmalloc.h` | 释放虚拟内存 |
| `alloc_pages()` | `include/linux/gfp.h` | 分配页面 |
| `free_pages()` | `include/linux/gfp.h` | 释放页面 |
| `get_user_pages()` | `include/linux/mm.h` | 获取用户态页面 |
| `kmap()` / `kunmap()` | `include/linux/highmem.h` | 映射高内存 |

### 内部接口

| 函数 | 位置 | 功能 |
|------|------|------|
| `__alloc_pages_nodemask()` | `mm/page_alloc.c` | Buddy 分配器核心 |
| `__get_free_pages()` | `mm/page_alloc.c` | 内部页面分配 |
| `__kmalloc()` | `mm/slab.c` | SLUB 分配器核心 |

**依赖方向**:
- 被依赖: 所有内核子系统（都需要分配内存）
- 依赖: 架构（页表）、调度器（kswapd 内核线程）

**证据**:
- `mm/page_alloc.c:3864` - `__alloc_pages_nodemask()`
- `mm/slub.c:3923` - `kmem_cache_alloc()`

---

## 文件系统 API (fs/)

### VFS 核心接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `vfs_open()` | `include/linux/fs.h` | VFS 打开文件 |
| `vfs_read()` | `include/linux/fs.h` | VFS 读取 |
| `vfs_write()` | `include/linux/fs.h` | VFS 写入 |
| `vfs_unlink()` | `include/linux/fs.h` | 删除文件 |
| `vfs_mkdir()` | `include/linux/fs.h` | 创建目录 |

### 文件操作结构

**证据**: `include/linux/fs.h:1980`
```c
struct file_operations {
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*release) (struct inode *, struct file *);
    // ...
};
```

**依赖方向**:
- 被依赖: 用户态 I/O 操作
- 依赖: 内存管理（页面缓存）、块设备层（存储）、LSM（权限）

**证据**:
- `fs/read_write.c:411` - `vfs_read()`
- `fs/open.c:437` - `do_sys_openat2()`

---

## 网络协议栈 API (net/)

### Socket 层接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `sock_create_kern()` | `include/linux/net.h` | 内核创建 socket |
| `sock_release()` | `include/linux/net.h` | 释放 socket |
| `sock_recvmsg()` | `include/linux/net.h` | 接收消息 |
| `sock_sendmsg()` | `include/linux/net.h` | 发送消息 |

### 设备接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `register_netdev()` | `include/linux/netdevice.h` | 注册网络设备 |
| `unregister_netdev()` | `include/linux/netdevice.h` | 注销网络设备 |
| `netif_rx()` | `include/linux/netdevice.h` | 接收数据包 |
| `dev_queue_xmit()` | `include/linux/netdevice.h` | 发送数据包 |

**依赖方向**:
- 被依赖: 网络应用程序
- 依赖: 驱动模型（网络设备）、内存管理（SKB）、LSM（网络过滤）

**证据**:
- `net/socket.c:2000` - `sock_sendmsg()`
- `net/core/dev.c:4263` - `netif_rx()`

---

## 驱动模型 API (drivers/base/)

### 设备/驱动注册

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `device_register()` | `include/linux/device.h` | 注册设备 |
| `device_unregister()` | `include/linux/device.h` | 注销设备 |
| `driver_register()` | `include/linux/device_driver.h` | 注册驱动 |
| `driver_unregister()` | `include/linux/device_driver.h` | 注销驱动 |
| `platform_driver_register()` | `include/linux/platform_device.h` | 注册平台驱动 |
| `platform_driver_unregister()` | `include/linux/platform_device.h` | 注销平台驱动 |

### 设备探测

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `driver_probe_device()` | `drivers/base/dd.c` | 探测设备 |
| `device_bind_driver()` | `drivers/base/dd.c` | 绑定驱动 |

**依赖方向**:
- 被依赖: 所有设备驱动
- 依赖: 内核核心（kobject）、sysfs（设备展示）

**证据**:
- `drivers/base/core.c` - `device_register()`
- `drivers/base/dd.c` - `driver_probe_device()`

---

## 中断 API (kernel/irq/)

### 稳定接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `request_irq()` | `include/linux/interrupt.h` | 注册中断处理 |
| `free_irq()` | `include/linux/interrupt.h` | 释放中断 |
| `enable_irq()` | `include/linux/interrupt.h` | 启用中断 |
| `disable_irq()` | `include/linux/interrupt.h` | 禁用中断 |

**依赖方向**:
- 被依赖: 设备驱动
- 依赖: 架构（中断控制器）

**证据**:
- `kernel/irq/manage.c` - `request_irq()`

---

## 工作队列 API (kernel/workqueue.c)

### 稳定接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `create_workqueue()` | `include/linux/workqueue.h` | 创建工作队列 |
| `destroy_workqueue()` | `include/linux/workqueue.h` | 销毁工作队列 |
| `schedule_work()` | `include/linux/workqueue.h` | 提交工作 |
| `flush_work()` | `include/linux/workqueue.h` | 等待工作完成 |
| `INIT_WORK()` | `include/linux/workqueue.h` | 初始化工作项 |

**依赖方向**:
- 被依赖: 需要延迟执行的代码
- 依赖: 调度器（内核线程）

**证据**:
- `kernel/workqueue.c` - 工作队列实现

---

## RCU API (kernel/rcu/)

### 稳定接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `rcu_read_lock()` | `include/linux/rcupdate.h` | 读者锁 |
| `rcu_read_unlock()` | `include/linux/rcupdate.h` | 读者解锁 |
| `synchronize_rcu()` | `include/linux/rcupdate.h` | 等待读者完成 |
| `call_rcu()` | `include/linux/rcupdate.h` | 延迟回调 |
| `rcu_dereference()` | `include/linux/rcupdate.h` | 安全解引用 |

**依赖方向**:
- 被依赖: 需要无锁读取的代码
- 依赖: 调度器（RCU 内核线程）

**证据**:
- `kernel/rcu/` - RCU 实现

---

## 符号导出

### EXPORT_SYMBOL 使用

**目的**: 导出内核符号供模块使用。

**语法**:
```c
EXPORT_SYMBOL(symbol_name);           // 非专有
EXPORT_SYMBOL_GPL(symbol_name);       // GPL 专有
EXPORT_SYMBOL_GPL_FUTURE(symbol_name); // 未来 GPL
```

**证据**:
- `kernel/sched/core.c` - `EXPORT_SYMBOL(schedule())`

---

## 模块接口 (kernel/module/)

### 稳定接口

| 函数 | 头文件 | 功能 |
|------|---------|------|
| `request_module()` | `include/linux/kmod.h` | 请求加载模块 |
| `try_module_get()` | `include/linux/module.h` | 引用计数增加 |
| `module_put()` | `include/linux/module.h` | 引用计数减少 |

**模块入口**:
```c
module_init(init_function);    // 模块初始化
module_exit(exit_function);    // 模块退出
```

**依赖方向**:
- 被依赖: 内核模块
- 依赖: 文件系统（模块加载）、LSM（模块权限）

**证据**:
- `kernel/module/main.c` - 模块加载核心

---

## 相关跳转

- [02_Architecture.md](02_Architecture.md) - 架构与依赖关系
- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录与模块职责
- [03_System_Calls.md](03_System_Calls.md) - 系统调用接口

---

**最后更新**: 2026-02-06
**文档版本**: v1.0
