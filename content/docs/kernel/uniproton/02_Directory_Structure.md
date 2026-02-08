# UniProton 目录结构

## 顶层目录

```
uniproton/
├── src/                  # ★ 核心源代码目录
├── demos/               # 示例应用
├── doc/                 # 设计文档
├── platform/            # 平台适配
├── wiki/                # ★ 本 Wiki 文档
├── BUILD.gn             # ★ GN 构建入口
├── uniproton.gni        # ★ GN 配置模板
├── bundle.json          # OpenHarmony 组件清单
├── README.md            # 项目说明
└── Kconfig             # 内核配置
```

## 源代码目录结构 (src/)

```
src/
├── core/                    # ★ 内核核心模块
│   ├── kernel/              # ★ 任务/调度/定时器/中断
│   │   ├── include/         # 内核内部头文件
│   │   ├── task/            # ★ 任务管理实现
│   │   ├── timer/           # 软件定时器
│   │   ├── tick/            # Tick 管理
│   │   ├── irq/             # 中断处理
│   │   ├── kexc/            # 内核异常
│   │   └── sys/             # 系统初始化/控制
│   └── ipc/                 # ★ 进程间通信
│       ├── include/         # IPC 头文件
│       ├── sem/             # ★ 信号量
│       ├── queue/           # ★ 消息队列
│       ├── event/           # ★ 事件标志
│       └── rwlock/          # 读写锁
│
├── arch/                    # ★ 架构支持代码
│   ├── include/             # 架构公共头文件
│   ├── cpu/                 # ★ CPU 架构实现
│   │   ├── armv7-m/         # ARMv7-M 架构
│   │   │   ├── common/      # 公共代码 (boot/exception/hwi/tick)
│   │   │   └── cortex-m4/   # Cortex-M4 特定代码
│   │   └── armv8/           # ARMv8 架构
│   │       └── common/      # 公共代码
│   └── drv/                 # 驱动代码
│       └── gic/             # GIC 中断控制器驱动
│
├── mem/                     # ★ 内存管理
│   ├── include/             # 内存头文件
│   └── fsc/                 # 固定大小块分配器
│
├── fs/                      # ★ 文件系统
│   ├── vfs/                 # VFS 虚拟文件系统层
│   └── littlefs/            # LittleFS 适配
│
├── net/                     # 网络栈
│   └── lwip-2.1/            # lwIP TCP/IP 栈
│       ├── include/         # lwIP 头文件
│       ├── src/             # lwIP 源码
│       └── enhancement/     # 增强功能
│
├── om/                      # 运维监控
│   ├── include/             # OM 头文件
│   ├── err/                 # 错误处理
│   ├── cpup/                # ★ CPU 占用率统计
│   └── hook/                # Hook 回调机制
│
├── security/                # 安全功能
│   └── rnd/                 # 随机数生成 (栈保护)
│
├── osal/                    # OS 抽象层
│   └── posix/               # POSIX 兼容实现
│       ├── include/         # POSIX 头文件
│       └── src/             # POSIX 实现
│
├── utility/                 # 工具库
│   └── lib/                 # 数学库等
│
├── include/                 # ★ 用户 API 头文件
│   └── uapi/                # ★ 公共 API (prt_*.h)
│       └── hw/             # 架构特定 UAPI
│
└── config/                  # 内核配置
    └── prt_config.h        # 运行时配置
```

## 模块职责说明

### core/kernel (内核核心)

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `task/` | 任务生命周期管理、调度 | `prt_task.c`, `prt_task_init.c` |
| `timer/` | 软件定时器管理 | `prt_timer.c`, `swtmr/` |
| `tick/` | 系统时钟节拍 | `prt_tick.c`, `prt_tick_init.c` |
| `irq/` | 中断请求处理 | `prt_irq.c` |
| `kexc/` | 内核异常处理 | `prt_kexc.c` |
| `sys/` | 系统初始化与控制 | `prt_sys.c`, `prt_sys_init.c` |

### core/ipc (进程间通信)

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `sem/` | 信号量 (计数/二元/递归) | `prt_sem.c`, `prt_sem_init.c` |
| `queue/` | 消息队列 | `prt_queue.c`, `prt_queue_init.c` |
| `event/` | 事件标志 | `prt_event.c` |
| `rwlock/` | 读写锁 | `prt_rwlock.c` |

### arch/cpu (架构支持)

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `armv7-m/common/` | ARMv7-M 公共代码 | `prt_hw_boot.c`, `prt_exc.c`, `prt_hwi.c` |
| `armv7-m/cortex-m4/` | Cortex-M4 特定代码 | `prt_dispatch.S`, `prt_hw.S`, `prt_vector.S` |
| `armv8/common/` | ARMv8 公共代码 | `prt_hwi_internal.h` |
| `drv/gic/` | GIC 驱动 | `prt_gic_common_internal.h` |

## 头文件组织

### 用户 API (UAPI)

路径: `src/include/uapi/`

| 头文件 | 功能 |
|--------|------|
| `prt_task.h` | 任务管理 |
| `prt_sem.h` | 信号量 |
| `prt_queue.h` | 消息队列 |
| `prt_event.h` | 事件标志 |
| `prt_timer.h` | 定时器 |
| `prt_tick.h` | Tick |
| `prt_mem.h` | 内存管理 |
| `prt_hwi.h` | 硬件中断 |
| `prt_exc.h` | 异常处理 |
| `prt_sys.h` | 系统控制 |
| `prt_fs.h` | 文件系统 |
| `prt_cpup.h` | CPU 占用率 |
| `prt_hook.h` | Hook 回调 |
| `prt_errno.h` | 错误码 |
| `prt_typedef.h` | 类型定义 |

### 内部 API (Internal)

| 路径模式 | 职责 |
|----------|------|
| `core/kernel/include/prt_*_external.h` | 内核内部接口 |
| `core/ipc/*/prt_*_external.h` | IPC 内部接口 |
| `arch/include/prt_*_external.h` | 架构公共接口 |

## 构建配置文件

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | GN 构建入口，定义 targets |
| `uniproton.gni` | GN 模板和变量定义 |
| `bundle.json` | OpenHarmony 组件清单 |
| `src/Kconfig` | 内核功能配置 |
| `src/*/Kconfig` | 各模块配置 |

## 示例应用

| 目录 | 描述 |
|------|------|
| `demos/helloworld/` | 基础示例 |
| `demos/hi3093/` | HiSilicon Hi3093 开发板 |
| `demos/raspi4/` | Raspberry Pi 4 (ARMv8) |
| `demos/alientek/` | 正点原子开发板 |

## 测试目录 (不纳入文档)

以下测试相关目录**不**在 Wiki 文档范围内：

```
**/test/
**/tests/
**/unittest/
**/unit_test/
**/fuzz/
**/*_test.*
**/*_fuzzer.*
```

---

## 相关文档

- [API 参考](./03_API_Reference.md) - 完整 API 清单
- [架构设计](./04_Architecture.md) - 组件交互
- [构建系统](./05_Build_System.md) - 编译配置
