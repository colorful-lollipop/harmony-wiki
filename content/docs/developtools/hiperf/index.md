# hiperf - OpenHarmony 性能分析工具

## 简介

hiperf 是 OpenHarmony 为开发人员提供的命令行性能分析工具，用于抓取特定程序或系统的性能数据，功能类似于 Linux 内核的 perf 工具。

## 核心能力

### 1. 性能采样（Sampling）

通过 Linux perf_event 接口采集硬件和软件事件：

- **硬件事件**: CPU 周期、指令数、缓存命中/未命中、分支预测等
- **软件事件**: 任务时钟、页错误、上下文切换、CPU 迁移等
- **Tracepoint 事件**: 内核静态探针事件
- **原始事件**: ARM PMU 原始事件码

### 2. 调用链分析（Call Stack）

支持多种调用链采集方式：

- **FP（Frame Pointer）**: 基于帧指针的调用链回溯
- **DWARF**: 基于 CFI（Call Frame Information）的调用链回溯
- **内核调用链**: 支持采集内核态调用链

### 3. 数据报告（Reporting）

支持多种格式的数据输出：

- **文本报告**: 命令行文本输出
- **JSON 格式**: 结构化数据输出
- **ProtoBuf 格式**: 二进制高效数据传输
- **HTML 报告**: 可视化火焰图和统计信息

### 4. 进程追踪（Tracing）

- 指定进程/线程采样
- 全系统采样（-a）
- 应用启动采样（--restart）
- 应用包名采样（--app）

## 运行环境

### 设备端（Target）

| 项目 | 要求 |
|------|------|
| 操作系统 | OpenHarmony 3.0+ |
| 架构 | ARM64 / ARM32 |
| 权限 | Root 或 UID 1201 |
| 开发者模式 | 必须开启 |

### Host 端（开发机）

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux / Windows / macOS |
| Python | 3.7.0+ |
| 架构 | x86_64 |

## 快速开始

### 命令行使用

```bash
# 列出支持的硬件事件
hiperf list hw

# 对指定进程采样 10 秒
hiperf record -p <pid> -d 10 -o perf.data

# 全系统采样
hiperf record -a -d 10 -o perf.data

# 生成报告
hiperf report -i perf.data
```

### C++ API 使用

```cpp
#include "hiperf_client.h"

using namespace OHOS::Developtools::HiPerf::HiperfClient;

// 创建客户端
Client client("/data/local/tmp/");

// 配置采样选项
RecordOption option;
option.SetSelectPids({pid});
option.SetTimeStopSec(10);
option.SetFrequency(4000);
option.SetCallGraph("dwarf");

// 开始采样
client.Start(option);

// ... 执行业务逻辑 ...

// 停止采样
client.Stop();
```

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                        用户界面层                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │   list   │  │   stat   │  │  record  │  │  report  │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
├─────────────────────────────────────────────────────────────┤
│                        命令处理层                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              SubCommand（子命令基类）                │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │SubCommandRecord│ │SubCommandStat│ │SubCommandReport│ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                        核心功能层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  PerfEvents  │  │VirtualRuntime│  │ SymbolsFile  │      │
│  │  (perf事件)   │  │ (虚拟运行时)  │  │  (符号文件)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                        系统接口层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  perf_event  │  │    /proc     │  │   IPC(SAMGR) │      │
│  │   (内核接口)  │  │  (进程信息)   │  │ (系统服务)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## 关键概念

### 1. Perf Event

Linux 内核提供的性能监控接口，hiperf 通过 `perf_event_open` 系统调用创建事件监控。

### 2. Sample

采样数据，包含以下关键信息：
- 时间戳
- 进程/线程 ID
- 指令指针（IP）
- 调用链
- 事件计数

### 3. Symbol Resolution

将指令地址（IP）解析为符号名称（函数名）的过程，需要：
- ELF 文件解析
- 符号表读取
- 地址映射计算

### 4. Unwinding

调用链回溯过程，支持：
- 基于 FP 的快速回溯
- 基于 DWARF 的精确回溯
- 内核栈回溯

## 与 Linux perf 的关系

hiperf 与 Linux perf 的关系：

| 特性 | hiperf | Linux perf |
|------|--------|------------|
| 运行环境 | OpenHarmony | Linux |
| 数据格式 | 兼容 perf.data | 标准 perf.data |
| 符号解析 | 支持 HAP/ARK | 仅 ELF |
| 调用链 | 支持 FP/DWARF | 支持 FP/DWARF/LBR |
| 脚本支持 | Python | Python/Perl |

## 相关文档

- [项目定位与核心能力](01_Overview.md) - 详细了解项目边界
- [目录结构](02_Directory_Structure.md) - 代码组织说明
- [架构说明](03_Architecture.md) - 系统架构详解
- [对外 API](04_Public_API.md) - API 使用指南

## 许可证

Apache License 2.0
