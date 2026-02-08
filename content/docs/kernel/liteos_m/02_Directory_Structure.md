# 目录结构 - 02_Directory_Structure

## 顶层结构

```
/kernel/liteos_m/
├── arch/                 # 架构层（CPU 架构相关）
├── components/           # 可选组件
├── drivers/              # 驱动配置
├── kal/                  # 内核抽象层
├── kernel/               # 最小内核功能集
├── testsuites/           # 测试套件 (忽略)
├── tools/                # 工具
├── utils/                # 公共工具
├── BUILD.gn              # 根构建文件
├── liteos.gni            # GN 模板定义
├── Kconfig               # menuconfig 配置
├── config.gni            # 架构配置
└── README.md
```

## 各目录职责详解

### arch/ - 架构层

```
arch/
├── arm/                  # ARM32 架构
│   ├── arm9/
│   ├── cortex-m3/
│   ├── cortex-m4/
│   ├── cortex-m33/
│   ├── cortex-m7/
│   └── include/          # ARM 公共头文件
├── risc-v/               # RISC-V 架构
│   ├── nuclei/
│   └── riscv32/
├── csky/                 # C-Sky 架构
│   └── v2/
├── xtensa/               # Xtensa 架构
│   └── lx6/
└── include/              # 架构层对外 API
```

### kernel/ - 最小内核功能集

```
kernel/
├── include/              # 对外 API 头文件
│   ├── los_config.h     # 内核配置
│   ├── los_task.h       # 任务管理 API
│   ├── los_queue.h      # 消息队列 API
│   ├── los_mux.h        # 互斥锁 API
│   ├── los_sem.h        # 信号量 API
│   ├── los_event.h      # 事件 API
│   ├── los_swtmr.h      # 软件定时器 API
│   ├── los_sched.h      # 调度器 API
│   └── ...
├── src/                 # 核心实现
│   ├── los_task.c       # 任务管理
│   ├── los_queue.c      # 消息队列
│   ├── los_mux.c        # 互斥锁
│   ├── los_sem.c        # 信号量
│   ├── los_event.c      # 事件
│   ├── los_swtmr.c      # 软件定时器
│   ├── los_sched.c      # 调度器
│   ├── los_tick.c       # 系统节拍
│   ├── los_init.c       # 内核初始化
│   └── mm/              # 内存管理
└── BUILD.gn
```

### kal/ - 内核抽象层

```
kal/
├── cmsis/               # CMSIS-RTOS API 支持
├── posix/               # POSIX API 支持
├── libc/                # C 标准库
├── libsec/              # 安全库
└── BUILD.gn
```

### components/ - 可选组件

| 组件 | 路径 | 说明 |
|------|------|------|
| backtrace | `components/backtrace/` | 栈回溯支持 |
| cppsupport | `components/cppsupport/` | C++ 支持 |
| cpup | `components/cpup/` | CPU 占用率统计 |
| dynlink | `components/dynlink/` | 动态加载 **(安全关键)** |
| exchook | `components/exchook/` | 异常钩子 |
| fs | `components/fs/` | 文件系统 |
| lmk | `components/lmk/` | 低内存杀手 |
| lms | `components/lms/` | 内存 sanitizer |
| net | `components/net/` | 网络功能 |
| power | `components/power/` | 电源管理 |
| shell | `components/shell/` | Shell 命令 |
| signal | `components/signal/` | 信号处理 |
| trace | `components/trace/` | 跟踪工具 |

### utils/ - 公共工具

```
utils/
├── los_list.h           # 双向链表实现
├── los_error.h/c        # 错误处理
├── los_debug.h/c        # 调试输出
├── los_hook.h/c         # 钩子机制
├── los_compiler.h      # 编译器适配
├── los_reg.h            # 寄存器操作
└── internal/           # 内部头文件
```

## 模块职责矩阵

| 模块 | 职责 | 对外 API 文件 |
|------|------|--------------|
| kernel | 核心内核功能 | `kernel/include/los_*.h` |
| kal | API 适配层 | `kal/*/include/` |
| arch | 架构移植层 | `arch/*/include/` |
| components | 可选功能 | 各组件 `include/` |
| utils | 公共基础 | `utils/los_*.h` |

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](03_Architecture.md)
- [内核 API](04_Kernel_API.md)
