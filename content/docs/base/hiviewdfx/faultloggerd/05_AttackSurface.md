# 攻击面分析

**文档用途**：帮助安全研究员快速识别所有外部输入入口、敏感操作和信任边界

**适用范围**：安全审计、渗透测试、威胁建模

---

## 1. 外部输入入口清单

### 1.1 IPC 接口处理器 (Socket-based)

faultloggerd 使用 Unix Domain Socket 而非标准 IPC/SA 框架，主要 Socket 如下：

| Socket 名称 | 路径 | 用途 | 风险等级 |
|-------------|------|------|----------|
| `faultloggerd.server` | `/dev/unix/socket/` | 通用服务请求 | **高** |
| `faultloggerd.crash.server` | `/dev/unix/socket/` | 崩溃日志请求 | **高** |
| `faultloggerd.sdkdump.server` | `/dev/unix/socket/` | SDK dump 请求 | **高** |

#### 请求类型定义

**位置**: `interfaces/common/dfx_socket_request.h:55-69`

```cpp
typedef enum {
    LOG_FILE_DES_CLIENT = 0,          // 请求文件描述符
    SDK_DUMP_CLIENT,                  // 请求堆栈 dump
    PIPE_FD_CLIENT,                   // 请求管道 FD
    REPORT_EXCEPTION_CLIENT,          // 报告异常
    DUMP_STATS_CLIENT,                // Dump 统计
    COREDUMP_CLIENT,                  // Coredump 请求
    COREDUMP_PROCESS_DUMP_CLIENT,     // Coredump 进程 dump
    PIPE_FD_LITEPERF_CLIENT,          // LitePerf 管道
    LIMITED_PROCESS_DUMP_CLIENT,      // 受限进程 dump
    PIPE_FD_LIMITED_CLIENT,           // 受限管道
} FaultLoggerClientType;
```

#### IPC 处理入口

| 文件 | 处理函数 | 请求类型 | 风险等级 |
|------|----------|----------|----------|
| `services/fault_logger_server.cpp` | `OnEventPoll()` | 所有 Socket 请求 | **高** |
| `services/fault_logger_service.cpp` | `FileDesService::OnRequest()` | LOG_FILE_DES_CLIENT | **高** |
| `services/fault_logger_service.cpp` | `SdkDumpService::OnRequest()` | SDK_DUMP_CLIENT | **高** |
| `services/fault_logger_service.cpp` | `PipeService::OnRequest()` | PIPE_FD_CLIENT | **中** |
| `services/fault_logger_service.cpp` | `LitePerfPipeService::OnRequest()` | PIPE_FD_LITEPERF_CLIENT | **中** |
| `services/fault_logger_service.cpp` | `ExceptionReportService::OnRequest()` | REPORT_EXCEPTION_CLIENT | **中** |

### 1.2 命令行工具

| 工具 | 路径 | 描述 | 风险等级 |
|------|------|------|----------|
| `processdump` | `/system/bin/processdump` | 进程信息抓取工具 | **高** |
| `dumpcatcher` | `/system/bin/dumpcatcher` | 堆栈抓取命令行工具 | **中** |
| `crash_validator` | `/system/bin/crashvalidator` | 崩溃验证工具 | **低** |

**processdump 参数解析** (`tools/process_dump/main.cpp`):
```cpp
// -p [pid]: 指定目标进程
// -t [tid]: 指定目标线程
// 示例: processdump -p 114 -t 114
```

### 1.3 文件系统操作

#### 敏感文件读取

| 文件路径 | 操作 | 位置 | 风险等级 |
|----------|------|------|----------|
| `/proc/[pid]/maps` | 读取内存映射 | `interfaces/innerkits/unwinder/src/maps/dfx_maps.cpp` | **高** |
| `/proc/[pid]/status` | 读取进程状态 | `interfaces/innerkits/procinfo/procinfo.cpp` | **中** |
| `/proc/[pid]/cmdline` | 读取命令行 | `common/dfxutil/proc_util.cpp` | **中** |
| `/proc/[pid]/task/` | 遍历线程 | `interfaces/innerkits/procinfo/procinfo.cpp` | **中** |
| `/proc/[pid]/stat` | 读取状态 | `common/dfxutil/proc_util.cpp` | **中** |
| `/proc/self/smaps_rollup` | 读取内存统计 | `common/dfxutil/proc_util.cpp` | **低** |

#### 敏感文件写入

| 文件路径 | 操作 | 位置 | 风险等级 |
|----------|------|------|----------|
| `/data/log/faultlog/temp/` | 创建崩溃日志 | `services/temp_file_manager.cpp` | **高** |
| 可配置输出路径 | Coredump 文件 | `tools/process_dump/coredump/coredump_file_manager.cpp` | **高** |

### 1.4 信号处理输入

| 信号 | 值 | 处理器位置 | 风险等级 |
|------|-----|-----------|----------|
| `SIGILL` | 4 | `interfaces/innerkits/signal_handler/dfx_signal_handler.c` | **高** |
| `SIGTRAP` | 5 | 同上 | **高** |
| `SIGABRT` | 6 | 同上 | **高** |
| `SIGBUS` | 7 | 同上 | **高** |
| `SIGFPE` | 8 | 同上 | **高** |
| `SIGSEGV` | 11 | 同上 | **高** |
| `SIGSTKFLT` | 16 | 同上 | **高** |
| `SIGSYS` | 31 | 同上 | **高** |
| `SIGDUMP` | 35 | `interfaces/innerkits/sigdump_handler/dfx_sigdump_handler.cpp` | **高** |

### 1.5 环境变量

| 变量名 | 用途 | 位置 | 风险等级 |
|--------|------|------|----------|
| `HAP_DEBUGGABLE` | 控制调试行为 | `interfaces/innerkits/signal_handler/dfx_signal_handler.c` | **中** |
| `HAP_DEBUGGABLE` | 控制异步栈 | `interfaces/innerkits/async_stack/async_stack.cpp` | **中** |

---

## 2. 敏感操作清单

### 2.1 系统调用 (关键)

#### ptrace 操作

**位置**: `interfaces/innerkits/unwinder/src/utils/dfx_ptrace.cpp`

```cpp
// PTRACE_SEIZE - 附加到目标进程
long DfxPtraceAttach(pid_t tid);

// PTRACE_DETACH - 分离
long DfxPtraceDetach(pid_t tid);

// PTRACE_INTERRUPT - 中断线程
long DfxPtraceInterrupt(pid_t tid);

// PTRACE_PEEKDATA - 读取内存
long DfxPtracePeekText(pid_t tid, void* addr);
```

#### 跨进程内存读取

**位置**: `common/dfxutil/dfx_util.cpp:70-95`

```cpp
bool ReadProcMemByPid(pid_t pid, uint64_t addr, void* data, size_t size)
{
    struct iovec localIoVec = { data, size };
    struct iovec remoteIoVec = { reinterpret_cast<void*>(addr), size };
    ssize_t bytes = process_vm_readv(pid, &localIoVec, 1, &remoteIoVec, 1, 0);
    return bytes == static_cast<ssize_t>(size);
}
```

#### 进程/线程创建

| 系统调用 | 用途 | 位置 | 风险等级 |
|----------|------|------|----------|
| `fork()` | 创建 processdump 子进程 | `services/fault_logger_service.cpp` | **高** |
| `syscall(SYS_clone)` | 创建线程 | `interfaces/innerkits/signal_handler/dfx_dumprequest.c` | **高** |
| `syscall(SYS_tgkill)` | 发送信号到线程 | `interfaces/innerkits/signal_handler/dfx_dumprequest.c` | **高** |
| `syscall(SYS_rt_tgsigqueueinfo)` | 发送信号带信息 | `services/fault_common_util.cpp` | **高** |

### 2.2 信号操作

| 操作 | 描述 | 位置 | 风险等级 |
|------|------|------|----------|
| `sigaction()` | 注册信号处理器 | `interfaces/innerkits/signal_handler/dfx_signal_handler.c` | **高** |
| `kill()` / `tgkill()` | 发送信号 | 多处 | **高** |
| `sigaltstack()` | 设置信号栈 | `interfaces/innerkits/signal_handler/dfx_signal_handler.c` | **中** |

### 2.3 特权操作

| 操作 | 描述 | 位置 | 风险等级 |
|------|------|------|----------|
| `setuid()` | 设置用户 ID | 服务启动 | **高** |
| `setgid()` | 设置组 ID | 服务启动 | **高** |
| Capability 设置 | `CAP_KILL`, `CAP_DAC_READ_SEARCH` | `services/config/faultloggerd.cfg` | **高** |

---

## 3. 信任边界图

### 3.1 架构分层

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           USER SPACE (UNTRUSTED)                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ 3rd Party    │  │ System Apps  │  │  Root/       │  │  Crash       │     │
│  │  Apps        │  │  (HIDumper)  │  │  System UID  │  │  Validator   │     │
│  │  (UID>10000) │  │  (UID=5523)  │  │  (UID=0)     │  │              │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                 │                 │             │
│         └─────────────────┴────────┬────────┴─────────────────┘             │
│                                    │                                        │
└────────────────────────────────────┼────────────────────────────────────────┘
                                     │
                              Unix Socket IPC
                                     │
┌────────────────────────────────────┼────────────────────────────────────────┐
│                                    ▼                                        │
│                      FAULTLOGGERD SERVICE (system)                          │
│                    ┌──────────────────────────────┐                        │
│                    │   SocketServer (epoll)       │                        │
│                    │   - Connection limit: 30     │                        │
│                    │   - SO_PEERCRED validation   │                        │
│                    └──────────────┬───────────────┘                        │
│                                   │                                         │
│         ┌─────────────────────────┼─────────────────────────┐              │
│         │                         │                         │              │
│         ▼                         ▼                         ▼              │
│  ┌──────────────┐        ┌──────────────┐        ┌──────────────┐         │
│  │ FileDes      │        │ SdkDump      │        │ Pipe/LitePerf│         │
│  │ Service      │        │ Service      │        │ Services     │         │
│  └──────┬───────┘        └──────┬───────┘        └──────┬───────┘         │
│         │                       │                       │                  │
│         ▼                       ▼                       ▼                  │
│  ┌──────────────┐        ┌──────────────┐        ┌──────────────┐         │
│  │ TempFileMgr  │        │ Signal Send  │        │ Pipe Pairs   │         │
│  │ Create FDs   │        │ SIGDUMP to   │        │ Management   │         │
│  │              │        │ target       │        │              │         │
│  └──────────────┘        └──────────────┘        └──────────────┘         │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
                                     │
                              Process Fork
                                     │
┌────────────────────────────────────┼────────────────────────────────────────┐
│                                    ▼                                        │
│                      PROCESSDUMP (privileged child)                         │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │  - Reads stdin for ProcessDumpRequest                             │      │
│  │  - ptrace attach to target process                                │      │
│  │  - Reads /proc/[pid]/maps, /proc/[pid]/status                     │      │
│  │  - process_vm_readv() for memory reading                          │      │
│  │  - Writes crash logs to temp files                                │      │
│  │  - Optional: coredump generation                                  │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 权限检查点

#### UID 白名单验证

**位置**: `services/fault_logger_service.cpp:150-170`

```cpp
bool CheckCallerUID(uid_t uid)
{
    static std::vector<int> allowedUids = {0, 1000, 1201, 1212, 5523, 7400};
    // 0: root
    // 1000: bmsUid
    // 1201: hiviewUid
    // 1212: hidumperServiceUid
    // 5523: foundationUid
    // 7400: dev_assistant
    return std::find(allowedUids.begin(), allowedUids.end(), uid) != allowedUids.end();
}
```

#### Peer Credential 验证

**位置**: `services/fault_logger_service.cpp:172-195`

```cpp
bool CheckRequestCredential(int32_t fd, int32_t requestPid)
{
    struct ucred creds{};
    if (!GetUcredByPeerCred(creds, fd)) {
        return false;
    }
    // 白名单 UID 直接通过
    if (CheckCallerUID(creds.uid)) {
        return true;
    }
    // 非白名单 UID 必须 PID 匹配
    if (creds.pid != requestPid) {
        return false;
    }
    return true;
}
```

#### Coredump 权限验证

**位置**: `tools/process_dump/coredump/coredump_controller.cpp`

```cpp
// VerifyTrustList() - 检查 bundle 名称白名单
// IsCoredumpAllowed() - 检查信号是否允许 coredump
// HasCoredumpPermission() - 综合权限检查
```

### 3.3 特权转换点

| 转换点 | 源 | 目标 | 机制 | 验证方式 |
|--------|-----|------|------|----------|
| IPC 请求 | Client (any UID) | faultloggerd (system) | Unix Socket + SO_PEERCRED | UID 白名单或 PID 匹配 |
| SDK Dump | App (own UID) | faultloggerd | SdkDumpService::Filter() | CheckCallerUID() 或 creds.pid == requestPid |
| Exception Report | signal_handler | faultloggerd | Socket | creds.uid == requestData.uid |
| Processdump 启动 | faultloggerd | processdump | fork() + execl() | 进程名检查 /proc/[pid]/cmdline |
| Ptrace 附加 | processdump | Target Process | ptrace(PTRACE_SEIZE) | 内核 ptrace 权限检查 |
| Coredump 生成 | processdump | Coredump File | File write | VerifyTrustList() + HasCoredumpPermission() |

---

## 4. 攻击向量汇总

### 4.1 网络/IPC 攻击

| 攻击向量 | 描述 | 可行性 | 影响 |
|----------|------|--------|------|
| Socket 连接伪造 | 伪造 Socket 请求 | 低 (需要 SELinux 权限) | 未授权 dump |
| UID 欺骗 | 伪造 UID 绕过检查 | 低 (SO_PEERCRED 防止) | 权限提升 |
| PID 欺骗 | 伪造 PID 绕过检查 | 中 | 越权 dump |
| 拒绝服务 | 大量连接耗尽资源 | 中 | 服务不可用 |

### 4.2 权限提升攻击

| 攻击向量 | 描述 | 可行性 | 影响 |
|----------|------|--------|------|
| ptrace 滥用 | 利用 UID 白名单读取任意进程 | 低 (白名单控制) | 信息泄露 |
| Coredump 绕过 | 绕过白名单生成 coredump | 低 (多重检查) | 敏感数据泄露 |
| 日志注入 | 伪造崩溃日志 | 低 (文件名唯一) | 误导诊断 |

### 4.3 资源耗尽攻击

| 攻击向量 | 描述 | 可行性 | 影响 |
|----------|------|--------|------|
| Pipe 耗尽 | 创建大量 pipe | 中 (UID 限制 20) | 资源耗尽 |
| Dump 配额耗尽 | 耗尽每日 dump 配额 | 低 (60/天限制) | DoS |
| 连接耗尽 | 耗尽连接数 | 中 (30 全局限制) | DoS |

### 4.4 信息泄露攻击

| 攻击向量 | 描述 | 可行性 | 影响 |
|----------|------|--------|------|
| /proc 信息读取 | 读取其他进程信息 | 低 (UID 隔离) | 进程信息泄露 |
| 内存读取 | 读取其他进程内存 | 低 (ptrace 限制) | 敏感数据泄露 |
| 日志读取 | 读取其他应用崩溃日志 | 低 (权限控制) | 应用数据泄露 |

---

## 5. 安全检查清单

### 5.1 已实现的安全控制

| 控制项 | 实现位置 | 有效性 |
|--------|----------|--------|
| UID 白名单 | `services/fault_logger_service.cpp:150-170` | ✅ 有效 |
| PID 匹配验证 | `services/fault_logger_service.cpp:172-195` | ✅ 有效 |
| SO_PEERCRED 验证 | `services/fault_logger_service.cpp` | ✅ 有效 |
| 连接数限制 (30) | `services/main.cpp:18` | ✅ 有效 |
| UID 级 Pipe 限制 (20) | `services/fault_logger_pipe.cpp:467-471` | ✅ 有效 |
| 每日 Dump 限制 (60) | `services/fault_logger_service.cpp:51-62` | ✅ 有效 |
| SELinux 策略 | `services/config/faultloggerd.cfg` | ✅ 有效 |
| Coredump 白名单 | `tools/process_dump/coredump/coredump_controller.cpp` | ✅ 有效 |
| 进程名验证 | `services/fault_logger_service.cpp` | ✅ 有效 |
| 超时控制 | `services/fault_logger_pipe.cpp:167-188` | ✅ 有效 |

### 5.2 建议增强

| 建议 | 优先级 | 说明 |
|------|--------|------|
| 全局资源限制 | 中 | 当前仅 UID 级别限制 |
| 滑动窗口限制 | 低 | 替代每日限制 |
| 审计日志 | 中 | 记录所有特权操作 |
| seccomp-bpf | 低 | 限制 processdump 系统调用 |

---

## 6. 相关链接

- [安全风险评审](07_Security_Review.md) - 详细风险分析
- [架构设计](02_Architecture.md) - 组件交互
- [对外 API](03_External_API.md) - API 权限要求
- [项目定位与核心能力](00_Overview.md) - 项目概述

---

*本文档基于代码分析生成，所有代码引用均可追溯到具体文件和行号。*
