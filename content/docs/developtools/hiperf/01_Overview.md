# 项目定位与核心能力

## 目的

本文档说明 hiperf 的项目定位、能力边界、运行环境和关键概念。

## 适用范围

- OpenHarmony 开发者
- 性能分析工具开发者
- 系统安全审计人员

## 项目定位

### 在 OpenHarmony 中的位置

```
OpenHarmony 系统架构
├── 应用层 (Applications)
├── 应用框架层 (Application Framework)
├── 系统服务层 (System Services)
│   └── 开发工具子系统 (developtools)
│       ├── hiperf ← 本文档组件
│       ├── bytrace
│       └── profiler
├── 内核层 (Kernel)
│   └── perf_event subsystem
└── 硬件抽象层 (HAL)
```

hiperf 属于**开发工具子系统**，位于系统服务层，直接对接内核 perf_event 子系统。

### 核心定位

**hiperf 是 OpenHarmony 官方性能采样分析工具**，提供：

1. **硬件性能监控**: 通过 PMU（Performance Monitoring Unit）监控 CPU 硬件事件
2. **软件事件追踪**: 监控内核和应用的软件事件
3. **调用链分析**: 采集和分析函数调用链
4. **性能报告生成**: 生成可视化性能报告

## 能力边界

### 支持的功能

| 功能类别 | 具体能力 | 代码位置 |
|----------|----------|----------|
| 事件监控 | 硬件事件（CPU 周期、缓存等） | `include/perf_events.h:75-114` |
| 事件监控 | 软件事件（时钟、页错误等） | `include/perf_events.h:102-114` |
| 事件监控 | Tracepoint 事件 | `src/perf_events.cpp:282-348` |
| 事件监控 | SPE（Statistical Profiling Extension）| `src/spe_decoder.cpp` |
| 采样控制 | 频率/周期控制 | `src/subcommand_record.cpp:78-92` |
| 采样控制 | CPU 限制 | `src/subcommand_record.cpp:51-54` |
| 采样控制 | 时间限制 | `src/subcommand_record.cpp:56-57` |
| 采样控制 | 数据大小限制 | `src/subcommand_record.cpp:336` |
| 调用链 | FP（帧指针）回溯 | `src/callstack.cpp` |
| 调用链 | DWARF 回溯 | `src/callstack.cpp` |
| 调用链 | 内核调用链 | `include/subcommand_record.h:262` |
| 进程追踪 | 指定 PID/TID | `src/subcommand_record.cpp:285-302` |
| 进程追踪 | 全系统采样 | `include/subcommand_record.h:251` |
| 进程追踪 | 应用包名采样 | `src/subcommand_record.cpp:278` |
| 进程追踪 | 应用启动采样 | `include/subcommand_record.h:289` |
| 数据输出 | perf.data 格式 | `src/perf_file_writer.cpp` |
| 数据输出 | JSON 格式 | `src/report_json_file.cpp` |
| 数据输出 | ProtoBuf 格式 | `src/report_protobuf_file.cpp` |
| 数据输出 | HTML 报告 | `script/report.html` |
| 符号解析 | ELF 符号解析 | `src/symbols_file.cpp` |
| 符号解析 | HAP 符号解析 | `src/symbols_file.cpp:1063-1118` |
| 符号解析 | 内核符号解析 | `src/symbols_file.cpp:672-727` |

### 不支持的功能

| 功能 | 说明 | 替代方案 |
|------|------|----------|
| N-API/JS API | hiperf 不提供 JS 接口 | 使用 C++ API 或命令行 |
| eBPF | 不支持 eBPF 探针 | 使用内核 tracepoint |
| 动态探针（kprobes/uprobes）| 不支持动态插桩 | 使用静态 tracepoint |
| 实时分析 | 不支持实时流式分析 | 采样后离线分析 |
| 多设备协同 | 不支持分布式分析 | 分别在各设备采样 |

## 运行环境

### 设备端要求

#### 系统要求

| 项目 | 要求 | 检查方法 |
|------|------|----------|
| OpenHarmony 版本 | 3.0+ | `param get const.ohos.apiversion` |
| 内核版本 | Linux 5.10+ | `uname -r` |
| perf_event 支持 | 必须 | `ls /proc/sys/kernel/perf_event_*` |

#### 权限要求

**入口检查** (`src/main.cpp:64-67`):
```cpp
if (!GetDeveloperMode() && !IsAllowProfilingUid()) {
    printf("error: not in developermode, exit.\n");
    return -1;
}
```

**允许采样的 UID** (`include/utilities.h:104`):
```cpp
inline const std::set<int> ALLOW_UIDS = {1201};
```

**权限检查逻辑**:
1. 必须是 Root 用户，或 UID 为 1201
2. 或开发者模式已开启

#### 应用采样限制

**应用调试检查** (`src/ipc_utilities.cpp:68-99`):
```cpp
bool IsDebugableApp(const std::string& bundleName);
```

- 只能采样标记为 debuggable 的应用
- 加密应用有特殊处理 (`IsApplicationEncryped`)
- 第三方应用有特殊限制 (`IsThirdPartyApp`)

### Host 端要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux (推荐)、Windows、macOS |
| Python | 3.7.0+ |
| HDC | 必须安装 hdc_std 工具 |

## 关键概念

### 1. 性能事件（Perf Event）

#### 硬件事件

定义位置: `include/perf_events.h:75-86`

| 事件名 | 说明 | 典型用途 |
|--------|------|----------|
| hw-cpu-cycles | CPU 周期数 | 计算 CPU 使用率 |
| hw-instructions | 指令数 | 计算 IPC（每周期指令数）|
| hw-cache-references | 缓存访问 | 缓存命中率分析 |
| hw-cache-misses | 缓存未命中 | 缓存优化 |
| hw-branch-instructions | 分支指令 | 分支预测分析 |
| hw-branch-misses | 分支预测失败 | 分支优化 |

#### 软件事件

定义位置: `include/perf_events.h:102-114`

| 事件名 | 说明 |
|--------|------|
| sw-cpu-clock | CPU 时钟 |
| sw-task-clock | 任务时钟 |
| sw-page-faults | 页错误 |
| sw-context-switches | 上下文切换 |

### 2. 采样模式

#### 频率采样（Frequency）

- 参数: `-f <freq>`
- 默认: 4000 samples/second
- 范围: 1-100000 (`include/subcommand_record.h:48-49`)

#### 周期采样（Period）

- 参数: `--period <num>`
- 默认: 1
- 说明: 每发生 `<num>` 个事件采样一次

### 3. 调用链类型

#### Frame Pointer（FP）

- 参数: `-s fp`
- 优点: 速度快，开销小
- 缺点: 需要编译时开启 `-fno-omit-frame-pointer`

#### DWARF

- 参数: `-s dwarf[,size]`
- 默认栈大小: 65528 bytes
- 范围: 8-65528，8 字节对齐 (`include/subcommand_record.h:333`)
- 优点: 精确，无需帧指针
- 缺点: 开销大，采样数据多

### 4. 数据文件格式

#### perf.data

hiperf 生成的数据文件与 Linux perf 兼容：

```
┌─────────────────────────────────────┐
│           File Header               │
├─────────────────────────────────────┤
│        Attribute Section            │
├─────────────────────────────────────┤
│          Data Section               │
│    (PerfEventRecord sequence)       │
├─────────────────────────────────────┤
│        Feature Section              │
│  (symbols, threads, etc.)           │
└─────────────────────────────────────┘
```

格式定义: `include/perf_file_format.h`

## 使用场景

### 场景 1: CPU 热点分析

```bash
# 采样 CPU 周期事件
hiperf record -a -e hw-cpu-cycles -f 4000 -d 10

# 生成火焰图报告
python script/make_report.py -i perf.data
```

### 场景 2: 应用启动优化

```bash
# 采样应用启动过程
hiperf record --app com.example.app --restart -d 10
```

### 场景 3: 缓存命中率分析

```bash
# 同时采样缓存访问和未命中
hiperf record -a -e hw-cache-references,hw-cache-misses -d 10
```

### 场景 4: 集成到 C++ 应用

```cpp
#include "hiperf_client.h"

void PerformanceCriticalSection() {
    HiperfClient::Client client;
    HiperfClient::RecordOption option;
    option.SetSelectPids({getpid()});
    option.SetTimeStopSec(5);
    
    client.Start(option);
    // ... 执行业务逻辑 ...
    client.Stop();
}
```

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 了解代码组织
- [架构说明](03_Architecture.md) - 深入了解系统架构
- [对外 API](04_Public_API.md) - 学习 API 使用
- [安全风险](08_Security.md) - 了解安全限制
