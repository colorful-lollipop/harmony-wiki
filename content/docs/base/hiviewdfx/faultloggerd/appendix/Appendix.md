# FaultLoggerd 附录

## 目的

本文档提供 faultloggerd 的补充信息，包括关键调用链、配置标志和常见问题。

## 适用范围

- 目标读者：系统开发者、故障诊断工程师
- 内容范围：生产代码（排除测试目录）

## 附录 A：关键调用链

### 1. 进程崩溃抓栈完整流程

```
应用进程
    ↓ 崩溃（异常信号）
[信号处理器] fork [ProcessDump]
    ↓ 保存上下文
    ↓ 申请 pipe
    ↓ [Socket 通信] [faultloggerd 服务]
    ↓ 权限验证
    ↓ 返回 FD
    ↓ 写崩溃日志
    ↓ /data/log/faultlog/temp/
    ↓ 完成
    ↓ [Hiview] 提取简易信息
    ↓ /data/log/faultlog/faultlogger/
    ↓ 生成 HiSysEvent 事件
```

**代码证据**：
- 信号注册：`interfaces/innerkits/signal_handler/dfx_signal_handler.c`
- 进程 fork：`tools/process_dump/process_dumper.cpp`
- Socket 请求：`interfaces/innerkits/faultloggerd_client/faultloggerd_client.cpp:141-159`
- 权限检查：`services/fault_logger_service.cpp:88-103`

### 2. DumpCatcher 主动抓栈流程

```
调用者进程
    ↓ [DumpCatcher API]
    ↓ 判断本地/远程
    ↓
    ├─→ 本地回栈
    │       [backtrace_local]
    │       ↓ 调用 unwinder
    │       ↓ 返回调用栈
    └─→ 远程抓栈
            ↓ 初始化 Socket
            ↓ [faultloggerd 服务]
            ↓ 发送 dump 请求
            ↓ 权限验证
            ↓ 分配 pipe
            ↓ 返回 pipe FD
            ↓ 通过 pipe 传递参数
            ↓ 发送 SIGDUMP(35) 信号
            ↓ [目标进程]
            ↓ 信号处理器触发
            ↓ fork [ProcessDump]
            ↓ 读取父进程内存
            ↓ 调用 unwinder
            ↓ 写入 pipe
            ↓ [调用者]
            ↓ 读取 pipe
            ↓ 返回调用栈
```

**代码证据**：
- DumpCatcher API：`interfaces/innerkits/dump_catcher/include/dfx_dump_catcher.h`
- 本地回栈：`interfaces/innerkits/backtrace/backtrace_local.cpp`
- 客户端 Socket：`interfaces/innerkits/faultloggerd_client/faultloggerd_client.cpp`
- 管道管理：`services/fault_logger_pipe.cpp:153-159`

### 3. Rust Panic 处理流程

```
Rust 进程
    ↓ panic!
    ↓ [panic_handler]
    ↓ 注册的 panic 回调
    ↓ 调用 unwinder (Rust)
    ↓ 解析符号 (rustc_demangle)
    ↓ 返回调用栈
    ↓ 写入文件
    ↓ /data/log/faultlog/faultlogger/
```

**代码证据**：
- Panic 注册：`interfaces/rust/panic_handler/lib.rs`
- Unwinder 接口：`interfaces/rust/stacktrace/`

### 4. 信号处理完整时序

```
进程启动
    ↓ 加载 libdfx_signalhandler.so
    ↓ 调用 signal/sigaction
    ↓ 注册信号处理器
    ↓
    │
    ↓ 进程运行
    ↓ 崩溃（异常信号）
    ↓ 触发信号处理器
    ↓ 保存寄存器上下文
    ↓ fork 子进程
    ↓ 执行 processdump
    ↓ 生成崩溃日志
    ↓
    ↓ 进程退出
```

**支持的信号**（`interfaces/common/dfx_define.h`）：
- SIGILL (4) - 非法指令
- SIGTRAP (5) - 断点或陷阱
- SIGABRT (6) - abort 信号
- SIGBUS (7) - 非法内存访问
- SIGFPE (8) - 浮点异常
- SIGSEGV (11) - 无效内存访问
- SIGSTKFLT (16) - 栈溢出
- SIGSYS (31) - 系统调用异常

**自定义信号**：
- SIGDUMP (35) - Dump 请求信号
- SIGLOCAL_DUMP (38) - 本地 dump 信号
- SIGLEAK_STACK (42) - 内存泄漏检测信号

## 附录 B：配置标志详解

### 编译选项（faultloggerd.gni）

| 选项 | 类型 | 默认值 | 说明 |
|------|------|----------|------|
| `libunwinder_debug` | bool | false | 启用 unwinder 调试日志 |
| `has_libunwindstack` | bool | false | 使用 libunwindstack 库替代 unwinder |
| `processdump_minidebuginfo_enable` | bool | true | 启用 mini 调试信息 |
| `faultloggerd_hisysevent_enable` | bool | false | 启用 HiSysEvent 集成（自动检测） |
| `faultloggerd_liteperf_enable` | bool | true | 启用轻量性能分析 |
| `processdump_parse_lock_owner_enable` | bool | false | 启用锁持有者解析 |
| `faultloggerd_enable_build_targets` | bool | true | 主开关，控制所有目标构建 |
| `faultloggerd_feature_coverage` | bool | false | 启用代码覆盖率 |

### 运行时配置（faultloggerd.conf）

| 配置项 | 默认值 | 说明 |
|----------|----------|------|
| `displayRigister` | true | 是否显示寄存器信息 |
| `displayBacktrace` | true | 是否显示调用栈 |
| `displayMaps` | true | 是否显示虚拟内存映射 |
| `displayFaultStack.switch` | true | 是否显示崩溃线程栈内存 |
| `displayFaultStack.lowAddressStep` | 16 | 崩溃栈内存向高地址读取的块数 |
| `displayFaultStack.highAddressStep` | 4 | 崩溃栈内存向低地址读取的块数 |
| `dumpOtherThreads` | false | 是否转储非崩溃线程信息 |

### 服务配置（faultloggerd.cfg）

| 配置项 | 说明 |
|----------|------|
| `uid` | `faultloggerd` | 服务运行 UID |
| `gid` | `["system", "log", "faultloggerd", "readproc"]` | 服务 GID 列表 |
| `socket.permissions` | `0666` | Socket 权限 |
| `caps` | `["CAP_DAC_READ_SEARCH", "CAP_KILL"]` | Linux Capability 列表 |
| `socket.family` | `AF_UNIX` | Socket 地址族 |
| `socket.type` | `SOCK_STREAM` | Socket 类型 |
| `secon` | `u:r:faultloggerd:s0` | SELinux 上下文 |

### UID 白名单（services/fault_logger_service.cpp:72-78）

| UID | 用途 |
|-----|------|
| 0 | root |
| 1000 | Bundle Manager Service (BMS) |
| 1201 | HiView Service |
| 1212 | HiDumper Service |
| 5523 | Foundation |
| 7400 | Dev Assistant |

**权限说明**：
- 白名单 UID 可以请求 dump 任意进程
- 非白名单 UID 只能 dump 自己的进程
- 如果 PID 不等于连接 PID，则拒绝请求

### 崩溃日志路径

| 日志类型 | 路径 | 文件命名 |
|----------|------|------------|
| 临时日志 | `/data/log/faultlog/temp/` | `cppcrash-{pid}-{timestamp}` |
| 故障日志 | `/data/log/faultlog/faultlogger/` | 由 Hiview 管理 |

**时间戳格式**：Unix 时间戳（毫秒）

## 附录 C：错误码完整列表

### DumpCatcher 错误码

| 错误码 | 常量名 | 说明 |
|----------|----------|------|
| -1 | UNKNOWN_ERROR | 未知错误 |
| -2 | INVALID_PID | 无效的进程 ID |
| -3 | INVALID_TID | 无效的线程 ID |
| -4 | CONNECT_SERVER_FAILED | Socket 连接失败 |
| -5 | SDK_DUMP_REPEAT | SDK dump 重复请求 |
| -6 | SDK_PROCESS_CRASHED | 目标进程已崩溃 |
| -7 | RESOURCE_LIMIT | 资源受限（超出限制） |
| -8 | TIMEOUT | 操作超时 |

### Socket 响应码（dfx_socket_request.h）

| 响应码 | 说明 |
|----------|------|
| 0 | REQUEST_SUCCESS | 请求成功 |
| 1 | UNKNOWN_CLIENT_TYPE | 未知客户端类型 |
| 2 | INVALID_REQUEST_DATA | 无效请求数据 |
| 3 | REQUEST_REJECT | 请求被拒绝（权限或验证失败） |
| 4 | ABNORMAL_SERVICE | 服务异常 |

### HiSysEvent 事件

**CPP_CRASH_EXCEPTION**（`services/fault_logger_service.cpp:128-142`）：
```cpp
HiSysEventWrite(
    HiSysEvent::Domain::RELIABILITY,
    "CPP_CRASH_EXCEPTION",
    HiSysEvent::EventType::FAULT,
    "PID", requestData.pid,
    "UID", requestData.uid,
    "HAPPEN_TIME", requestData.time,
    "ERROR_CODE", requestData.error,
    "ERROR_MSG", requestData.message);
```

**DUMP_CATCHER_STATS**（`services/fault_logger_service.cpp:154-176`）：
```cpp
HiSysEventWrite(
    HiSysEvent::Domain::HIVIEWDFX,
    "DUMP_CATCHER_STATS",
    HiSysEvent::EventType::STATISTIC,
    "CALLER_PROCESS_NAME", stat.callerProcessName,
    "CALLER_FUNC_NAME", stat.callerElfName,
    "TARGET_PROCESS_NAME", stat.targetProcessName,
    "RESULT", stat.result,
    "SUMMARY", stat.summary,
    "PID", stat.pid,
    "REQUEST_TIME", stat.requestTime,
    "OVERALL_TIME", stat.dumpCatcherFinishTime - stat.requestTime,
    "SIGNAL_TIME", stat.signalTime - stat.requestTime,
    "DUMPER_START_TIME", stat.processdumpStartTime - stat.signalTime,
    "WRITE_DUMP_INFO_TIME", stat.writeDumpInfoCost,
    "UNWIND_TIME", stat.processdumpFinishTime - stat.processdumpStartTime,
    "KEY_THREAD_UNWIND_TIMESTAMP", stat.keyThreadUnwindTimestamp,
    "TARGET_PROCESS_THREAD_COUNT", stat.targetProcessThreadCount,
    "PSS_MEMORY", stat.pssMemory);
```

## 附录 D：常见问题与定位

### 崩溃日志为空

**原因**：SELinux 标签失效

**症状**：
- 崩溃发生后生成了 `cppcrash-{pid}-{timestamp}` 文件
- 文件大小为 0

**解决方法**（`docs/usage.md:220-226`）：
```bash
restorecon /data/log/faultlog/temp
```

**原因**：手动删除并重新创建了 `/data/log/faultlog/temp` 目录

### addr2line 无法解析到行号

**原因**：
1. 使用了 LTO（链接时优化）导致调试信息丢失
2. 二进制不匹配（unstripped 版本不匹配运行版本）

**解决方法**（`docs/usage.md:79-82`）：
1. 尝试对地址进行微调（如减 1）
2. 关闭 LTO 编译选项
3. 确保使用带调试信息的二进制：
```bash
# 查找带调试信息的二进制
ls out/rk3568/lib.unstripped/
ls out/rk3568/exe.unstripped/
```

### 进程退出但无崩溃日志

**可能原因**：
1. 信号被屏蔽或拦截（`docs/usage.md:194-218`）
2. 进程使用了 signal/sigaction/sigprocmask
3. 引入的三方库拦截了信号

**排查方法**：
1. 使用 sighook 机制定位信号拦截者
2. 使用 strace 跟踪系统调用：
```bash
strace -p pid
# 查找是否有 signal/sigaction/sigprocmask 调用
```

### 进程崩溃且生成了日志，但内容不完整

**可能原因**：
1. 回栈信息不完整
2. 映射文件读取失败
3. 寄存器信息缺失

**解决方法**：
1. 检查 `/proc/<pid>/maps` 文件权限
2. 检查内存映射文件完整性
3. 检查是否有足够的磁盘空间

### 如何打开 CoreDump

**默认行为**：系统默认不生成 coredump 文件（`docs/usage.md:181-188`）

**临时打开方法**：
```bash
# 设置硬限制
toybox prlimit -c -H unlimited -P [pid of target process]
toybox prlimit -c -S unlimited -P [pid of target process]

# 设置生成文件模板
echo /data/log/coredump.%p.bin > /proc/sys/kernel/core_pattern

# 在 init.cfg 中添加配置（持久化）
```

### 检查 FaultLoggerd 服务状态

**命令**：
```bash
ps -A | grep faultloggerd
```

**预期输出**：
```
system      ?        00:00:00   0      0   S    faultloggerd
```

**检查日志**：
```bash
hilog | grep faultloggerd
```

### 工具使用示例

#### DumpCatcher 命令行工具

```bash
# 抓取指定进程的所有线程
dumpcatcher -p <pid>

# 抓取指定进程的指定线程
dumpcatcher -p <pid> -t <tid>
```

#### processdump 工具

```bash
# 帮用选项
processdump -p <pid> --json  # JSON 格式输出
processdump -p <pid> --mini-debug  # 使用 mini debug info
```

## 相关跳转

- [项目定位与核心能力](00_Overview.md) - 项目概览
- [架构设计](02_Architecture.md) - 完整处理流程
- [对外 API](03_External_API.md) - API 详细文档和错误码
- [安全风险评审](07_Security_Review.md) - 安全模型和最佳实践
