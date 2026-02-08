# 目录结构

## 顶层目录概览

```
kernel_liteos_a/
├── apps                   # 用户空间应用 (init/shell)
├── arch                   # 架构相关代码 (ARM)
├── bsd                    # FreeBSD 驱动适配 (USB)
├── compat                 # POSIX 兼容性层
├── drivers                # 内核驱动 (字符设备)
├── fs                     # 文件系统 (来自 NuttX)
├── kernel                 # 核心内核模块
├── lib                    # 内核库
├── net                    # 网络栈 (来自 lwIP)
├── platform              # SOC 适配代码
├── security              # 安全机制 (Cap/VID)
├── shell                 # Shell 命令
├── syscall               # 系统调用入口
├── tools                 # 构建工具
├── BUILD.gn              # GN 构建入口
├── liteos.gni            # GN 模板定义
├── Kconfig               # 内核配置
└── Makefile              # 传统 Make 构建
```

## 目录职责详解

### apps/ - 用户空间应用
| 子目录 | 职责 |
|--------|------|
| `init` | 系统初始化进程 |
| `shell` | Shell 命令解释器 |
| `perf` | 性能分析工具 |

**证据**: `apps/` 目录存在于 `BUILD.gn:355` 作为 deps 依赖

### arch/ - 架构支持
```
arch/
└── arm/
    ├── arm/          # ARM32/ARM64 通用代码
    ├── include/      # 架构头文件
    └── aarch64/     # AArch64 特定代码
```

**关键文件**:
- `arch/arm/include/los_hwi.h` - 硬件中断
- `arch/arm/include/los_strncpy_from_user.h` - 用户态拷贝

### bsd/ - FreeBSD 驱动
来自 FreeBSD 的 USB 驱动代码适配层

### compat/ - POSIX 兼容
```
compat/
└── posix/           # POSIX API 兼容性实现
```

### drivers/ - 字符设备驱动
```
drivers/
└── char/
    ├── mem/         # 物理 I/O 访问
    ├── quickstart/  # 系统快速启动
    ├── random/      # 随机数生成
    └── video/       # Framebuffer 驱动
```

### fs/ - 文件系统 (来自 NuttX)
| 子目录 | 职责 |
|--------|------|
| `fat` | FAT12/16/32 文件系统 |
| `jffs2` | JFFS2 闪存文件系统 |
| `nfs` | NFS 客户端 |
| `proc` | Proc 文件系统 |
| `ramfs` | RAM 文件系统 |
| `vfs` | 虚拟文件系统层 |

### kernel/ - 核心内核模块

#### kernel/base/ - 基础模块
```
kernel/base/
├── core/        # 核心调度、进程、时钟
├── ipc/         # Event/Futex/Mutex/Queue/Sem
├── mem/         # 内存分配器 (TLSF/MemBox)
├── misc/        # 杂项 (kill/栈信息/内存统计)
├── mp/          # 多处理器支持
├── om/          # 操作监控
├── sched/       # 调度器相关
└── vm/          # 虚拟内存管理
```

#### kernel/extended/ - 扩展模块
| 子目录 | 职责 |
|--------|------|
| `blackbox` | 黑盒日志 |
| `container` | 容器支持 (命名空间) |
| `cppsupport` | C++ 运行时支持 |
| `cpup` | CPU 性能监控 |
| `dynload` | ELF 动态加载 |
| `hidumper` | 系统转储 |
| `hilog` | 日志框架 |
| `hook` | 钩子机制 |
| `liteipc` | 轻量级 IPC |
| `lms` | 内存安全检测 |
| `perf` | 性能分析 |
| `pipes` | 管道 |
| `plimit` | 资源限制 |
| `power` | 电源管理 |
| `trace` | 追踪框架 |
| `vdso` | vDSO 支持 |

#### kernel/include/ - 对外头文件
约 78 个头文件，主要包括：
- `los_task.h` - 任务管理
- `los_process.h` - 进程管理
- `los_memory.h` - 内存管理
- `los_vm.h` - 虚拟内存
- `los_queue.h` - 消息队列
- `los_mux.h` - 互斥锁
- `los_sem.h` - 信号量
- `los_event.h` - 事件
- `los_smp.h` - SMP 支持

### security/ - 安全机制
```
security/
├── cap/          # Linux 能力 (Capability)
└── vid/          # 虚拟 ID 映射
```

### syscall/ - 系统调用入口
| 文件 | 职责 |
|------|------|
| `fs_syscall.c` | 文件系统 syscall |
| `ipc_syscall.c` | IPC syscall |
| `process_syscall.c` | 进程相关 syscall |
| `net_syscall.c` | 网络 syscall |
| `time_syscall.c` | 时间/定时器 syscall |
| `vm_syscall.c` | 虚拟内存 syscall |
| `misc_syscall.c` | 杂项 syscall |
| `syscall_lookup.h` | syscall 查找表 |
| `syscall_pub.h` | syscall 公共接口 |

**证据**: `syscall/BUILD.gn:35-45` 定义了所有 syscall 源文件

### shell/ - Shell 命令
shell 命令实现，位于 `BUILD.gn:339` 作为 deps

### platform/ - SOC 适配
```
platform/
├── hw/           # 时钟和中断逻辑
├── include/      # 平台头文件
└── uart/         # 串口逻辑
```

### tools/ - 构建工具
- `build/liteos.ld` - 链接脚本 (LLVM)
- `build/liteos_llvm.ld` - LLVM 链接脚本
- `build_lite.py` - 构建脚本

## 第三方依赖

| 项目 | 用途 | 位置 |
|------|------|------|
| FreeBSD | USB 驱动 | `third_party/FreeBSD` |
| NuttX | 文件系统 | `third_party/NuttX` |
| lwIP | 网络栈 | `third_party/lwip` |
| musl | C 库 | `third_party/musl` |
| FatFs | FAT 文件系统 | `third_party/FatFs` |
| zlib | 压缩 | `third_party/zlib` |

## 相关文档

- [项目概览](/00_Overview.md)
- [架构说明](/02_Architecture.md)
- [内核模块详解](/04_Kernel_Modules.md)
- [系统调用接口](/05_Syscall_API.md)
