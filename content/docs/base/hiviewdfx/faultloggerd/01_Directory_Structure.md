# 目录结构与模块职责

## 目的

本文档介绍 faultloggerd 项目的目录结构及各模块职责。

## 适用范围

- 目标读者：系统开发者、模块开发者
- 排除内容：测试目录（test/、*_test.*）

## 顶层目录结构

```
faultloggerd/
├── common/                    # 工具库和公共定义
├── docs/                      # 文档
├── example/                   # 示例代码
├── frameworks/                 # 主动抓栈实现
├── interfaces/                 # 接口层（对外 API）
│   ├── innerkits/             # Native 内部接口
│   └── rust/                 # Rust 接口
├── services/                   # faultloggerd 守护服务
├── tools/                     # 工具集
├── figures/                   # 架构图
├── BUILD.gn                   # 根构建文件
├── faultloggerd.gni            # 构建配置
├── bundle.json                # 组件元数据
├── OAT.xml                   # 测试配置
└── LICENSE                    # Apache 2.0 许可证
```

## 模块职责详解

### 1. common/ - 公共库

**职责**：提供跨模块的通用工具和定义

**子模块**：

#### common/dfxlog/
- **职责**：统一日志接口
- **关键文件**：
  - `dfx_log.h` - 日志宏定义（DFXLOGI、DFXLOGE 等）

#### common/dfxutil/
- **职责**：通用工具函数
- **关键文件**：
  - `dfx_util.h` - 工具函数（字符串处理、时间等）

#### common/trace/
- **职责**：追踪工具支持
- **关键文件**：
  - `include/dfx_trace.h` - 追踪接口

#### common/cutil/
- **职责**：C 工具库
- **关键文件**：
  - 链接至 `//utils/native/base`

#### common/build/
- **职责**：构建配置
- **关键文件**：
  - 构建相关的 .gni 配置

### 2. interfaces/ - 接口层

**职责**：定义对外 API 和内部模块接口

#### interfaces/innerkits/ - Native 接口

##### interfaces/innerkits/dump_catcher/
**职责**：主动抓取调用栈的 API

**关键文件**：
- `include/dfx_dump_catcher.h` - 主 API 接口定义
- `include/lite_perf.h` - 性能统计接口
- `dfx_dump_catcher_errno.h` - 错误码定义

**关键类**：
```cpp
class DfxDumpCatcher {
    bool DumpCatch(int pid, int tid, std::string& msg, ...);
    bool DumpCatchFd(int pid, int tid, std::string& msg, int fd, ...);
    bool DumpCatchMultiPid(const std::vector<int> &pids, std::string& msg);
};
```

##### interfaces/innerkits/backtrace/
**职责**：进程内本地回栈

**关键文件**：
- `include/backtrace_local.h` - 回栈 API
- `include/fp_backtrace.h` - FP 回栈实现
- `include/dfx_kernel_stack.h` - 内核栈接口

**关键 API**：
```cpp
bool GetBacktrace(std::string& out, bool fast, size_t maxFrameNums);
bool GetBacktraceStringByTid(std::string& out, int32_t tid, ...);
std::string GetProcessStacktrace(size_t maxFrameNums, ...);
```

##### interfaces/innerkits/unwinder/
**职责**：符号解析和栈回退

**关键文件**：
- `include/unwinder.h` - 符号解析器接口
- `include/dfx_elf_parser.h` - ELF 解析器
- `include/dfx_symbols.h` - 符号管理
- `include/dwarf_entry_parser.h` - DWARF 解析

**支持格式**：
- ELF 动态库
- DWARF 调试信息
- Ark TS 运行时符号

##### interfaces/innerkits/signal_handler/
**职责**：信号处理和崩溃上下文管理

**关键文件**：
- `include/dfx_signal_handler.h` - 信号处理 API
- `include/dfx_unique_crash_obj.h` - RAII 崩溃对象管理

**关键 API**：
```c
void SetThreadInfoCallback(ThreadInfoCallBack func);
void DFX_SetCrashObj(uint8_t type, uintptr_t addr);
void DFX_ResetCrashObj(uintptr_t crashObj);
```

##### interfaces/innerkits/faultloggerd_client/
**职责**：faultloggerd 服务客户端

**关键文件**：
- `include/faultloggerd_client.h` - Socket 客户端 API
- `faultloggerd_socket.h` - Socket 协议定义

**关键 API**（`faultloggerd_client.h:32-126`）：
```c
int32_t RequestFileDescriptor(int32_t type);
int32_t RequestPipeFd(int32_t pid, int32_t pipeType, int (&pipeFd)[2]);
int32_t RequestDelPipeFd(int32_t pid);
int32_t RequestSdkDump(int32_t pid, int32_t tid, ...);
int32_t ReportDumpStats(FaultLoggerdStatsRequest *request);
```

##### interfaces/innerkits/async_stack/
**职责**：异步调用栈跟踪

**关键文件**：
- `include/async_stack.h` - 异步栈 API
- `include/unique_stack_table.h` - 唯一栈表

##### interfaces/innerkits/procinfo/
**职责**：进程信息读取

**关键文件**：
- `include/procinfo.h` - 进程信息 API

**功能**：
- 读取 `/proc/[pid]/` 下的进程状态
- 解析线程、内存映射、文件描述符

##### interfaces/innerkits/stack_printer/
**职责**：栈信息格式化输出

**关键文件**：
- `include/stack_printer.h` - 栈打印接口

##### interfaces/innerkits/formatter/
**职责**：JSON 格式化输出

**关键文件**：
- `include/dfx_json_formatter.h` - JSON 格式化器

##### interfaces/innerkits/crash_exception/
**职责**：崩溃异常处理

**关键文件**：
- `crash_exception.h` - 异常类型定义

##### interfaces/innerkits/sigdump_handler/
**职责**：SIGDUMP 信号处理（主动抓栈）

**关键文件**：
- `include/dfx_sigdump_handler.h` - SIGDUMP 处理器

#### interfaces/rust/ - Rust 接口

##### interfaces/rust/panic_handler/
**职责**：Rust panic 处理器

**关键功能**：
```rust
pub fn init()  // 注册 panic 处理器
```

##### interfaces/rust/rustc_demangle/
**职责**：Rust 符号解析

##### interfaces/rust/stacktrace/
**职责**：Rust 栈跟踪

### 3. services/ - 守护服务

**职责**：运行 faultloggerd 守护进程，处理客户端请求

**关键模块**：

#### services/config/
- **职责**：配置管理
- **关键文件**：
  - `faultlogger.conf` - 日志配置（路径：`system/etc/faultlogger.conf`）

**配置项**（基于 `docs/usage.md:48-59`）：
```
displayRigister=true          # 显示寄存器
displayBacktrace=true          # 显示调用栈
displayMaps=true               # 显示内存映射
displayFaultStack.switch=true  # 显示崩溃栈内存
displayFaultStack.lowAddressStep=16
displayFaultStack.highAddressStep=4
dumpOtherThreads=false        # 转储非崩溃线程
```

#### services/temp_file_manager.cpp/h
- **职责**：临时文件管理
- **功能**：
  - 管理崩溃日志文件生命周期
  - 清理过期日志文件
  - 控制磁盘占用

#### services/fault_logger_daemon.cpp/h
- **职责**：服务主入口
- **代码位置**：`services/fault_logger_daemon.cpp:38-71`
- **功能**：
  - 初始化主服务器
  - 初始化辅助服务器
  - 注册 SIGCHLD/SIGPIPE 信号忽略

#### services/fault_logger_server.cpp/h
- **职责**：Socket 服务器处理
- **代码位置**：`services/fault_logger_server.cpp:88-160`
- **功能**：
  - 监听客户端连接
  - 权限验证
  - 请求分发

#### services/fault_logger_service.cpp/h
- **职责**：业务逻辑处理
- **关键功能**：
  - UID 权限检查（`services/fault_logger_service.cpp:62-86`）
  - Dump 请求处理
  - LitePerf 管道服务
  - 统计信息上报

**权限白名单**（`services/fault_logger_service.cpp:72-78`）：
```cpp
const uint32_t whitelist[] = {
    0,      // rootUid
    1000,    // bmsUid
    1201,    // hiviewUid
    1212,    // hidumperServiceUid
    5523,    // foundationUid
    7400,    // dev_assistant
};
```

#### services/fault_logger_pipe.cpp/h
- **职责**：管道管理
- **功能**：
  - 创建和管理通信管道
  - 管道超时控制
  - 并发限制

**关键类**（`services/fault_logger_pipe.h:73-81`）：
```cpp
class LitePerfPipePair {
    int32_t uid_;
    uint64_t timeOutTime_;
    static LitePerfPipePair& CreatePipePair(int uid, uint64_t timeOutTime);
    static void DelPipePair(int uid);
};
```

#### services/snapshot/
- **职责**：内核快照处理
- **关键文件**：
  - `kernel_snapshot_task.cpp/h` - 快照任务
  - `kernel_snapshot_parser.cpp/h` - 快照解析器

**功能**：
- 解析内核崩溃快照
- 生成结构化崩溃报告
- 集成到 Hiview

#### services/coredump/
- **职责**：Core dump 管理
- **关键文件**：
  - `coredump_manager_service.cpp/h` - Core dump 管理器
  - `coredump_facade.cpp/h` - Core dump 接口
  - `coredump_signal_service.cpp/h` - 信号服务

**功能**：
- 管理 core dump 文件
- UID 白名单控制（`services/coredump/coredump_manager_service.cpp:141-194`）
- 生命周期管理

#### services/epoll_manager.cpp/h
- **职责**：事件循环管理
- **功能**：
  - 使用 epoll 监听文件描述符
  - 处理异步 I/O 事件

### 4. frameworks/ - 主动抓栈实现

**职责**：主动抓栈的具体实现

#### frameworks/localhandler/
- **职责**：本地处理器
- **功能**：
  - 进程内本地抓栈实现
  - 异常现场捕获

#### frameworks/allocator/
- **职责**：内存分配器
- **功能**：
  - 提供受限环境下的内存分配

#### frameworks/limited/
- **职责**：限制性功能
- **功能**：
  - 资源受限环境下的功能裁剪

### 5. tools/ - 工具集

**职责**：独立可执行工具

#### tools/process_dump/
**职责**：进程 dump 工具
- **可执行文件**：`processdump`
- **安装路径**：`/system/bin/processdump`
- **功能**：
  - fork 子进程后执行
  - 读取目标进程内存
  - 生成崩溃日志文件
  - 支持管道返回结果

#### tools/dump_catcher/
**职责**：DumpCatcher 命令行工具
- **可执行文件**：`dumpcatcher`
- **安装路径**：`/system/bin/dumpcatcher`
- **功能**：
  - 命令行抓栈工具
  - 仅 Debug 版本提供

**使用方式**：
```bash
dumpcatcher -p <pid> -t <tid>
```

#### tools/crasher_c/
**职责**：C 崩溃构造器
- **可执行文件**：`crasher_c`
- **功能**：
  - 构造各种类型的崩溃
  - 测试故障日志生成

#### tools/crasher_cpp/
**职责**：C++ 崩溃构造器
- **可执行文件**：`crasher_cpp`
- **功能**：
  - 构造各种类型的崩溃
  - 测试故障日志生成

#### tools/panic_maker/
**职责**：Rust panic 构造器
- **可执行文件**：`panic_maker`
- **功能**：
  - 构造 Rust panic
  - 测试 panic 处理器

#### tools/crash_validator/
**职责**：崩溃验证工具
- **可执行文件**：`crash_validator`
- **功能**：
  - 验证崩溃日志完整性
  - 需要编译选项 `faultloggerd_hisysevent_enable`

### 6. example/ - 示例代码

**职责**：提供 API 使用示例

**关键示例**：
- `dumpcatcherdemo` - DumpCatcher 使用示例
- BUILD.gn - 示例编译配置

### 7. docs/ - 文档

**关键文件**：
- `design.md` - 设计文档
- `usage.md` - 使用说明

## 模块依赖关系

```
interfaces/innerkits/
├── dump_catcher ──→ unwinder
├── backtrace ──────→ unwinder
│                 └── procinfo
├── faultloggerd_client ─→ fault_logger_service
└── signal_handler

services/
├── fault_logger_daemon
│   ├── fault_logger_server
│   │   ├── fault_logger_service
│   │   │   ├── fault_logger_pipe
│   │   │   ├── temp_file_manager
│   │   │   └── epoll_manager
│   └── snapshot
└── coredump

tools/
├── process_dump ──→ unwinder, procinfo
└── dump_catcher ──→ dump_catcher (innerkits)
```

## 代码组织特点

### 1. 分层架构

```
[工具层]     common/
     ↓
[接口层]     interfaces/innerkits/
     ↓
[实现层]     frameworks/ + services/
     ↓
[工具层]     tools/
```

### 2. 接口封装

- **公共接口**：在 `interfaces/innerkits/include/` 定义头文件
- **内部实现**：在 `interfaces/innerkits/*/src/` 实现逻辑
- **依赖隔离**：每个 innerkit 独立编译，可按需链接

### 3. 配置驱动

- **编译选项**：`faultloggerd.gni` 定义关键开关
- **运行时配置**：`faultlogger.conf` 定义日志行为
- **服务配置**：`faultloggerd.cfg` 定义启动参数

## TODO

- [ ] 📋 确认 `frameworks/` 下各子模块的详细职责
- [ ] 📋 补充 `common/` 下工具库的详细 API 文档

## 相关跳转

- [项目定位与核心能力](00_Overview.md) - 项目概览
- [架构设计](02_Architecture.md) - 组件交互和流程
- [对外 API](03_External_API.md) - 接口详细文档
- [内部 API](04_Inner_API.md) - 模块间接口
