# FaultLoggerd 项目定位与核心能力

## 目的

本文档介绍 faultloggerd 组件的项目定位、功能边界、核心能力及运行环境。

## 适用范围

- 目标读者：OpenHarmony 系统开发者、应用开发者、故障诊断工程师、安全审计员
- 项目版本：基于 main 分支（2026-02-06）
- 内容范围：生产代码（排除测试目录）

## 项目定位

faultloggerd 是 OpenHarmony DFX（诊断与故障处理）子系统中的**C/C++ 运行时崩溃日志生成及管理模块**。

### 核心价值

1. **自动崩溃捕获**：捕获未处理的异常信号，自动生成崩溃日志
2. **故障现场保存**：保存信号、寄存器、调用栈、内存映射等完整现场
3. **主动抓栈能力**：提供 API 供主动抓取进程/线程堆栈
4. **轻量级设计**：资源占用小，适合嵌入式设备

### 与 FaultLogger 的区别

| 维度 | faultloggerd | FaultLogger |
|------|--------------|-------------|
| **位置** | hiviewdfx/faultloggerd | hiviewdfx/hiview |
| **职责** | 崩溃日志生成和临时存储 | 故障日志管理、查询、导出 |
| **启动方式** | init 守护进程 | hiview 服务 |
| **接口** | Socket API + Native API | JS API + hidumper |
| **拆分原因** | 权限分离、轻量部署、简单可靠 |

## 边界

### 功能边界

**包含**：
- ✅ C/C++ 进程崩溃日志自动生成
- ✅ 信号处理和故障现场捕获
- ✅ 主动抓栈 API（DumpCatcher、Backtrace）
- ✅ 崩溃日志临时存储
- ✅ Rust Panic 处理支持

**不包含**：
- ❌ Java/Kotlin 层崩溃（由其他模块处理）
- ❌ JavaScript/N-API 接口（纯 C/C++ 服务）
- ❌ 故障日志长期存储和查询（由 FaultLogger 处理）
- ❌ 性能分析和采样（非主要目标）

### 技术边界

- **语言**：C/C++（核心）、Rust（panic_handler 模块）
- **通信方式**：Unix Domain Socket（非 IPC/SA）
- **平台支持**：标准系统 + 轻量级系统
- **目标架构**：ARM64、ARM32、x86_64

## 核心能力

### 1. 崩溃日志自动生成

当进程因未处理信号崩溃时，自动生成包含以下信息的崩溃日志：

```
/data/log/faultlog/temp/cppcrash-{pid}-{timestamp}
```

**日志内容**：
- 进程信息：PID、UID、进程名
- 故障信息：信号类型、错误码、崩溃地址
- 故障线程：TID、线程名
- 调用栈：完整的函数调用链
- 寄存器现场：关键寄存器值
- 崩溃栈内存：栈帧原始数据
- 虚拟内存映射：进程地址空间分布

**支持的信号**（基于 `README_zh.md:32-41`）：

| 信号 | 说明 | 典型触发场景 |
|------|------|----------------|
| SIGILL (4) | 非法指令 | 执行数据段、栈溢出 |
| SIGTRAP (5) | 断点/陷阱 | 断点指令触发 |
| SIGABRT (6) | abort 调用 | 主动终止、资源检查失败 |
| SIGBUS (7) | 非法内存访问 | 未对齐访问、硬件错误 |
| SIGFPE (8) | 浮点异常 | 除零、浮点溢出 |
| SIGSEGV (11) | 无效内存访问 | 空指针、野指针 |
| SIGSTKFLT (16) | 栈溢出 | 栈空间耗尽 |
| SIGSYS (31) | 非法系统调用 | 无效 syscall |

### 2. 主动抓栈（DumpCatcher）

提供 `DfxDumpCatcher` 类，支持主动抓取指定进程/线程的调用栈：

**关键 API**（`interfaces/innerkits/dump_catcher/include/dfx_dump_catcher.h:30-86`）：
- `DumpCatch(pid, tid, msg)` - 抓取单个线程堆栈
- `DumpCatchFd(pid, tid, msg, fd)` - 抓取并写入文件
- `DumpCatchMultiPid(pids, msg)` - 批量抓取多个进程
- `DumpCatchWithTimeout(pid, msg, timeout)` - 带超时的抓栈

**特性**：
- ✅ 支持 C++ 和 C++-JS 混合栈
- ✅ FP 快速回栈和 DWARF 精确回栈
- ✅ JSON 格式和文本格式输出
- ✅ 最大帧数可配置（默认 256 帧）

### 3. 本地回栈（Backtrace）

提供进程内本地回栈能力，无需额外进程：

**关键 API**（`interfaces/innerkits/backtrace/include/backtrace_local.h:26-131`）：
- `GetBacktrace(out, fast)` - 获取当前线程调用栈
- `GetBacktraceStringByTid(out, tid, ...)` - 获取指定线程调用栈
- `PrintBacktrace(fd, ...)` - 打印到文件描述符
- `GetProcessStacktrace(...)` - 获取整个进程的堆栈（含线程信息）

**特性**：
- ✅ 快速 FP 回栈（fast=true）
- ✅ 精确 DWARF 回栈（fast=false）
- ✅ 内核栈回退（enableKernelStack）
- ✅ 跳帧和最大帧数配置

### 4. Rust Panic 处理

为 Rust 模块提供 Panic 故障处理器：

**API**（`interfaces/rust/panic_handler`）：
```rust
panic_handler::init()  // 注册 panic 处理器
```

**功能**：
- ✅ 捕获 Rust panic
- ✅ 生成故障日志到 `/data/log/faultlog/faultlogger`
- ✅ 支持 Rust 符号解析（rustc_demangle）

### 5. Signal Handler

提供信号处理器库，应用可集成以增强崩溃捕获：

**关键 API**（`interfaces/innerkits/signal_handler/include/dfx_signal_handler.h:25-136`）：
- `SetThreadInfoCallback(func)` - 自定义线程信息回调
- `DFX_SetAsyncStackCallback(func)` - 设置异步栈跟踪
- `DFX_SetCrashObj(type, addr)` - 附加诊断信息
- `DFX_SetCrashLogConfig(type, value)` - 配置日志生成

**崩溃对象类型**：
- `OBJ_STRING` - 字符串（最大 64KB）
- `OBJ_MEMORY_64B/256B/1024B/2048B/4096B` - 不同大小内存块

## 运行环境

### 系统要求

**操作系统**：
- OpenHarmony 3.0+（标准系统）
- OpenHarmony Lite（轻量级系统）

**依赖组件**（基于 `bundle.json:24-42`）：
- `hilog` - 日志记录
- `hisysevent` - 系统事件上报
- `hitrace` - 性能追踪
- `ipc` - 进程间通信
- `samgr` - 系统服务管理（可选）
- `selinux` - 安全策略
- `cJSON` - JSON 处理
- `libuv` - 异步 I/O

**资源占用**（基于 `bundle.json:22-23`）：
- ROM：~1024KB
- RAM：~1024KB

### 安装路径

**服务端**：
- 二进制：`/system/bin/faultloggerd`
- 配置：`/system/etc/faultloggerd.cfg`
- 配置：`/system/etc/faultlogger.conf`
- 日志目录：`/data/log/faultlog/temp`

**客户端库**：
- `libdfx_dumpcatcher.so` - DumpCatcher 库
- `libbacktrace_local.so` - Backtrace 库
- `libfaultloggerd.so` - 客户端库
- `dfx_signalhandler.so` - 信号处理库

**工具**：
- `processdump` - `/system/bin/processdump`
- `dumpcatcher` - `/system/bin/dumpcatcher`（仅 Debug 版本）

## 关键概念

### 1. 信号处理

faultloggerd 依赖系统信号机制捕获崩溃：
- **进程启动时**：加载 `dfx_signalhandler` 库注册信号处理器
- **崩溃发生时**：触发信号处理器，保存现场并 fork 子进程
- **子进程**：执行 `processdump` 工具生成崩溃日志

### 2. 主动抓栈 vs 自动崩溃

| 特性 | 自动崩溃捕获 | 主动抓栈 |
|------|----------------|----------|
| 触发方式 | 信号触发 | API 调用 |
| 目标进程 | 自己 | 任意进程（需权限） |
| 日志位置 | `/data/log/faultlog/temp` | 返回给调用者 |
| 通信 | 通过 pipe | Socket + Pipe |
| 权限要求 | 无 | system/root 或自己进程 |

### 3. 回栈技术

- **FP 回栈**：基于帧指针，快速但不准确（优化时可能失败）
- **DWARF 回栈**：基于调试信息，精确但较慢
- **内核栈**：当用户栈失败时，尝试从内核获取栈帧

### 4. 混合栈

支持 C++ 和 JavaScript 混合调用栈，用于 NAPI/Ace 应用：
- 自动识别 JS 引擎帧
- 格式化显示 C++ → JS 调用链

## 常见问题

### Q1: faultloggerd 和 FaultLogger 是同一个模块吗？

**否**。faultloggerd 是底层崩溃日志生成器，FaultLogger 是上层故障日志管理服务。

### Q2: 支持 Java/Kotlin 崩溃吗？

**不支持**。Java 崩溃由其他模块（如 AppSpawn、HiView）处理。

### Q3: 有 JavaScript/N-API 接口吗？

**没有**。faultloggerd 是纯 C/C++ 原生服务，不提供 JS 绑定。

### Q4: 崩溃日志保存多久？

临时存储在 `/data/log/faultlog/temp`，由 Hiview 后续处理迁移到 `/data/log/faultlog/faultlogger`。

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md) - 了解代码组织
- [架构设计](02_Architecture.md) - 深入理解组件交互
- [对外 API](03_External_API.md) - API 详细文档
- [安全风险评审](07_Security_Review.md) - 安全模型和边界
