# 项目概览

## 项目定位

### 所属子系统

`hidumper_lite` 是 OpenHarmony DFX（Debug、Fault-tolerance、eXcellent）子系统的重要组成部分。该子系统专注于为系统提供调试、容错和卓越性保障能力，而 `hidumper_lite` 则专注于提供系统信息转储（dump）功能，帮助开发者诊断和解决系统问题。

**代码证据**：`bundle.json` 中明确声明 `"subsystem": "hiviewdfx"`

### 项目边界

`hidumper_lite` 是一个轻量级的系统信息转储工具，主要服务于 LiteOS_M 和 LiteOS_A 两种轻量级内核。其职责边界包括：

1. **信息采集**：采集 CPU 使用率、内存使用率、任务状态、系统信息、故障日志等
2. **信息输出**：支持通过 AT 命令、命令行工具、文件输出等多种方式展示信息
3. **平台适配**：通过适配器模式支持不同芯片平台的差异化实现
4. **调试支持**：提供崩溃注入等调试功能（仅调试版本）

`hidumper_lite` **不负责**以下功能：

- 实时性能监控（由其他 DFX 子系统组件负责）
- 日志持久化存储（由 hiview_lite 负责）
- 故障上报和告警（由 hievent_lite 负责）
- 内核态信息采集（由内核驱动负责，通过 IOCTL 调用）

### 版本与模块配置

| 属性 | 值 | 证据位置 |
|------|-----|----------|
| 模块名称 | @ohos/hidumper_lite | `bundle.json:2` |
| 版本号 | 4.0.2 | `bundle.json:5` |
| 许可协议 | Apache License 2.0 | `bundle.json:6` |
| 目标系统类型 | mini（LiteOS_M/LiteOS_A） | `bundle.json:19-21` |
| ROM 占用 | 26KB | `bundle.json:22` |
| RAM 占用 | ~10KB | `bundle.json:23` |

## 核心能力

### 1. 系统信息转储

`hidumper_lite` 提供了全面的系统信息转储能力，涵盖以下维度：

| 能力 | 描述 | 证据位置 |
|------|------|----------|
| CPU 使用率转储 | 获取并显示 CPU 使用情况 | `mini/hidumper_core.c:112-114` |
| 内存使用率转储 | 获取并显示内存使用情况 | `mini/hidumper_core.c:117-119` |
| 任务信息转储 | 获取并显示所有任务状态 | `mini/hidumper_core.c:120-122` |
| 系统信息转储 | 获取并显示系统基本信息 | `mini/hidumper_core.c:60` |
| 故障日志转储 | 获取并显示最近保存的故障日志 | `mini/hidumper_core.c:115-116` |
| 内存数据转储 | 以十六进制格式转储内存数据 | `mini/hidumper_core.c:125-130` |

### 2. 多输出模式

支持多种信息输出方式，以适应不同场景需求：

| 输出模式 | 描述 | 适用场景 |
|----------|------|----------|
| AT 命令输出 | 通过 AT 框架输出信息 | 串口调试、设备端调试 |
| 命令行输出 | 通过独立命令行工具输出 | 设备端 shell 调试 |
| 文件输出 | 将内存数据输出到文件 | 离线分析场景 |
| 标准输出 | 输出到 stdout | 实时查看 |

### 3. 平台适配能力

通过适配器模式，`hidumper_lite` 能够在不同芯片平台上运行：

- **核心层**：`mini/hidumper_core.c` 实现平台无关的 AT 命令解析和分发逻辑
- **适配层**：`mini/hidumper_adapter.c` 提供弱函数默认实现
- **平台实现**：各芯片平台（如 hi3861）通过重写弱函数提供具体实现

**证据位置**：`mini/hidumper_adapter.c:33-82` 中的 `WEAK` 函数定义

## 运行环境

### 硬件要求

| 要求 | 说明 |
|------|------|
| 处理器架构 | ARM Cortex-M 系列（LiteOS_M）、ARM Cortex-A 系列（LiteOS_A） |
| 内存 | RAM ≥ 10KB（仅 hidumper_lite 自身占用） |
| 存储 | ROM ≥ 26KB（仅 hidumper_lite 自身占用） |

### 软件依赖

| 依赖项 | 版本/类型 | 用途 |
|--------|-----------|------|
| liteos_m | 内核组件 | 提供任务、内存等内核操作接口 |
| utils_lite | 基础工具库 | 提供 utils_lite 相关工具函数 |
| bounds_checking_function | 第三方安全库 | 提供安全的字符串操作函数 |

**证据位置**：`bundle.json:25-29`

### 支持的内核类型

根据 `BUILD.gn` 配置，项目支持以下内核类型：

| 内核类型 | 选择条件 | 构建目标 |
|----------|----------|----------|
| LiteOS_A | `ohos_kernel_type == "liteos_a"` | `lite:hidumper_lite` |
| LiteOS_M | `ohos_kernel_type == "liteos_m"` | `mini:hidumper_mini` |

**证据位置**：`BUILD.gn:18-22`

## 目录结构

```
hidumper_lite/
├── lite/                                   # [LiteOS_A 版本]
│   ├── hidumper.c                          # 命令行工具入口
│   │   ├── main()                          # 程序入口
│   │   ├── ParameterMatching()             # 参数解析
│   │   ├── Usage()                          # 帮助信息
│   │   ├── ExecAction()                    # IOCTL 执行
│   │   └── 各功能函数                      # DumpCpuUsage/DumpMemUsage 等
│   └── BUILD.gn                            # 构建配置
├── mini/                                   # [LiteOS_M 版本]
│   ├── hidumper_adapter.c                  # 适配层实现
│   │   ├── WEAK 弱函数                     # 默认平台无关实现
│   │   └── HiDumperAdapterInit()           # 初始化函数
│   ├── hidumper_core.c                     # 核心层实现
│   │   ├── HiDumperRegisterAdapter()       # 适配器注册
│   │   ├── at_hidumper()                   # AT 命令入口
│   │   ├── ParameterMatching()             # 参数解析
│   │   └── DumpAllInfo()                   # 全部信息输出
│   ├── BUILD.gn                            # 构建配置
│   └── interfaces/
│       └── native/
│           ├── innerkits/                  # 内部接口
│           │   ├── hidumper.h             # 适配器结构与注册接口
│           │   └── hidumper_adapter.h     # 平台适配接口声明
│           └── kits/                       # 外部接口
│               ├── hidumper.h             # 接口重导出
│               └── hidumper_adapter.h     # 接口重导出
├── BUILD.gn                                # 根构建配置
├── bundle.json                             # 模块配置
└── README.md                               # 项目说明文档
```

## 安全特性

### 调试功能保护

以下调试功能受 `OHOS_DEBUG` 宏保护，仅在调试版本中可用：

| 功能 | 保护位置 | 行为 |
|------|----------|------|
| 内存数据转储 | `lite/hidumper.c:120-130` | 打印 "Unsupported!" |
| 内核崩溃注入 | `lite/hidumper.c:134-140` | 打印 "Unsupported!" |
| 用户态崩溃注入 | `lite/hidumper.c:147-154` | 打印 "Unsupported!" |

### 安全函数使用

项目使用安全函数进行字符串和内存操作，防止缓冲区溢出：

| 操作 | 安全函数 | 证据位置 |
|------|----------|----------|
| 字符串复制 | `strncpy_s()` | `lite/hidumper.c:178-181, 193-196` |
| 内存复制 | `memcpy_s()` | `mini/hidumper_core.c:96-97` |

### 设备节点访问控制

命令行工具通过 `/dev/hidumper` 设备节点与内核通信：

| 属性 | 值 | 证据位置 |
|------|-----|----------|
| 设备路径 | `/dev/hidumper` | `lite/hidumper.c:28` |
| 打开模式 | `O_RDONLY` | `lite/hidumper.c:208` |

---

## 关键概念

### AT 命令框架

AT 命令是一种基于文本的命令交互协议，广泛用于嵌入式系统的串口通信。`hidumper_lite` 的 LiteOS_M 版本通过 AT 框架提供命令行接口。

**证据位置**：`mini/hidumper_core.c:155-165`（`at_hidumper` 函数）

### 适配器模式

适配器模式用于解决不同芯片平台的差异化问题。`hidumper_lite` 定义了 `HiDumperAdapter` 结构体，包含一组函数指针，由各平台提供具体实现。

**证据位置**：
- 结构体定义：`mini/interfaces/native/innerkits/hidumper.h:25-33`
- 注册函数：`mini/hidumper_core.c:79-103`
- 默认弱函数：`mini/hidumper_adapter.c:33-82`

### IOCTL 控制

IOCTL（Input/Output Control）是一种设备 I/O 操作方式，用于向设备发送控制命令。LiteOS_A 版本通过 IOCTL 与内核驱动通信。

**证据位置**：`lite/hidumper.c:55-62`（命令定义）、`lite/hidumper.c:85-96`（执行函数）

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [02_Architecture.md](02_Architecture.md) | 详细架构设计 |
| [03_API.md](03_API.md) | 接口使用说明 |
| [04_Build.md](04_Build.md) | 构建配置说明 |
| [05_Security.md](05_Security.md) | 安全风险评审 |
| [06_Troubleshooting.md](06_Troubleshooting.md) | 问题排查指南 |
| [SUMMARY.md](SUMMARY.md) | 文档导航 |

---

## 快速开始

### LiteOS_M 版本（AT命令）

```
# 转储所有信息
AT+HIDUMPER=

# 仅转储CPU使用率
AT+HIDUMPER=-dc

# 转储内存使用率
AT+HIDUMPER=-dm
```

**代码位置**: `mini/hidumper_core.c:38-55`

### LiteOS_A 版本（命令行）

```bash
# 转储所有信息
hidumper

# 仅转储CPU使用率
hidumper -dc

# 转储指定内存区域（调试版本）
hidumper -m 0x20000000 0x100
```

**代码位置**: `lite/hidumper.c:64-83`

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本，完成项目概览 |
