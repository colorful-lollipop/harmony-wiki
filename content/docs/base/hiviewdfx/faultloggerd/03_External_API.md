# FaultLoggerd 对外 API

## 目的

本文档详细介绍 faultloggerd 提供的对外 Native API，包括参数、返回值、错误码和调用示例。

## 适用范围

- 目标读者：应用开发者、系统开发者
- 接口类型：C/C++ Native API
- **注意**：faultloggerd 不提供 JavaScript/N-API 接口

## API 概览

| API 模块 | 头文件 | 主要用途 |
|-----------|--------|----------|
| **DumpCatcher** | `dfx_dump_catcher.h` | 主动抓取进程/线程调用栈 |
| **Backtrace** | `backtrace_local.h` | 进程内本地回栈 |
| **SignalHandler** | `dfx_signal_handler.h` | 崩溃信号处理和自定义回调 |
| **FaultloggerdClient** | `faultloggerd_client.h` | 客户端 Socket 通信 |
| **Rust PanicHandler** | `panic_handler` | Rust panic 处理器 |

## DumpCatcher API

### 接口定义

**头文件**：`interfaces/innerkits/dump_catcher/include/dfx_dump_catcher.h`

**类定义**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class DfxDumpCatcher {
public:
    static constexpr size_t DEFAULT_MAX_FRAME_NUM = 256;

    DfxDumpCatcher();

    /**
     * @brief Dump native stack by specify pid and tid
     */
    bool DumpCatch(int pid, int tid, std::string& msg,
                size_t maxFrameNums = DEFAULT_MAX_FRAME_NUM, bool isJson = false);

    /**
     * @brief Dump native stack by specify pid and tid to file
     */
    bool DumpCatchFd(int pid, int tid, std::string& msg, int fd,
                   size_t maxFrameNums = DEFAULT_MAX_FRAME_NUM);

    /**
     * @brief Dump native stack by multi-pid
     */
    bool DumpCatchMultiPid(const std::vector<int> &pids, std::string& msg);

    /**
     * @brief Dump stack of process with timeout
     */
    std::pair<int, std::string> DumpCatchWithTimeout(int pid, std::string& msg,
        int timeout = 3000, int tid = 0, bool isJson = false);
private:
    class Impl;
    std::shared_ptr<Impl> impl_;
};
} // namespace HiviewDFX
} // namespace OHOS
```

### API 详解

#### 1. DumpCatch()

**签名**：
```cpp
bool DumpCatch(int pid, int tid, std::string& msg,
            size_t maxFrameNums = 256, bool isJson = false);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `pid` | `int` | 目标进程 ID。如果抓取自己进程，则 tid 应为当前线程 |
| `tid` | `int` | 目标线程 ID。如果为 0，则抓取进程所有线程 |
| `msg` | `std::string&` | 输出参数，调用栈结果会存储在此字符串中 |
| `maxFrameNums` | `size_t` | 最大回栈帧数（默认 256） |
| `isJson` | `bool` | 是否返回 JSON 格式（默认 false，返回文本格式） |

**返回值**：

| 值 | 说明 |
|------|------|
| `true` | 抓栈成功，结果存储在 `msg` 中 |
| `false` | 抓栈失败，`msg` 包含错误信息 |

**错误码**（`interfaces/innerkits/dump_catcher/dfx_dump_catcher_errno.h`）：
```cpp
typedef enum DumpCatcherError : int32_t {
    UNKNOWN_ERROR = -1,
    INVALID_PID = -2,
    INVALID_TID = -3,
    CONNECT_SERVER_FAILED = -4,
    SDK_DUMP_REPEAT = -5,
    SDK_PROCESS_CRASHED = -6,
    RESOURCE_LIMIT = -7,
    TIMEOUT = -8,
};
```

**权限要求**：
- 调用者必须是 **root、system** 用户
- 或者只能抓取**自己用户**拥有的进程
- 需要读取 `/proc/<pid>/maps` 和执行 `ptrace`

**调用示例**：
```cpp
#include "dfx_dump_catcher.h"
#include <iostream>

using namespace OHOS::HiviewDFX;

void DumpMyProcess() {
    DfxDumpCatcher dumplog;
    std::string msg = "";
    int pid = getpid();
    int tid = gettid();

    // 抓取当前线程
    if (dumplog.DumpCatch(pid, tid, msg)) {
        std::cout << "Dump success:\n" << msg << std::endl;
    } else {
        std::cout << "Dump failed: " << msg << std::endl;
    }
}
```

**输出格式**（文本格式）：
```
Tid:1234, Name:my_process
#00 pc 0001234 /system/lib/libmyapp.so
#01 pc 0002345 /system/lib/libmyapp.so
#02 pc 0003456 /system/lib/libark_jsruntime.so
```

#### 2. DumpCatchFd()

**签名**：
```cpp
bool DumpCatchFd(int pid, int tid, std::string& msg, int fd,
                   size_t maxFrameNums = 256);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `pid` | `int` | 目标进程 ID |
| `tid` | `int` | 目标线程 ID |
| `msg` | `std::string&` | 输出参数（用于错误信息） |
| `fd` | `int` | 文件描述符，调用栈结果写入此 fd |
| `maxFrameNums` | `size_t` | 最大回栈帧数（默认 256） |

**返回值**：
- `true`：成功，调用栈写入 fd
- `false`：失败

**权限要求**：同 `DumpCatch()`

#### 3. DumpCatchMultiPid()

**签名**：
```cpp
bool DumpCatchMultiPid(const std::vector<int> &pids, std::string& msg);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `pids` | `std::vector<int>&` | 要抓取的进程 ID 列表 |
| `msg` | `std::string&` | 输出参数，结果存储在此 |

**返回值**：
- `true`：至少一个进程抓取成功
- `false`：所有进程抓取失败

#### 4. DumpCatchWithTimeout()

**签名**：
```cpp
std::pair<int, std::string> DumpCatchWithTimeout(
    int pid, std::string& msg, int timeout = 3000,
    int tid = 0, bool isJson = false);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `pid` | `int` | 目标进程 ID |
| `msg` | `std::string&` | 输出参数 |
| `timeout` | `int` | 超时时间（毫秒，至少 1000，默认 3000） |
| `tid` | `int` | 目标线程 ID（0 表示所有线程） |
| `isJson` | `bool` | 是否返回 JSON 格式 |

**返回值**（`std::pair<int, std::string>`）：
- `first`（`int`）：
  - `-1`：dump 失败
  - `0`：msg 是正常调用栈
  - `1`：msg 是内核栈（非 JSON 格式）
- `second`（`std::string`）：
  - 调用栈字符串
  - 如果 `first` 为 `-1` 或 `1`，则包含失败原因

## Backtrace API

### 接口定义

**头文件**：`interfaces/innerkits/backtrace/include/backtrace_local.h`

**C API**：
```c
extern "C" {
    /**
     * @brief Get a thread of backtrace string by specify tid
     */
    bool GetBacktraceStringByTid(std::string& out, int32_t tid,
        size_t skipFrameNum, bool fast, size_t maxFrameNums = 256, bool enableKernelStack = true);

    /**
     * @brief Get backtrace string of current thread
     */
    bool GetBacktrace(std::string& out, bool fast = false,
        size_t maxFrameNums = 256);

    /**
     * @brief Print backtrace information to fd
     */
    bool PrintBacktrace(int32_t fd = -1, bool fast = false, size_t maxFrameNums = 256);
} // extern "C"
```

**C++ API**：
```cpp
namespace OHOS {
namespace HiviewDFX {
    /**
     * @brief Get a thread of backtrace string by specify tid enable mix
     */
    bool GetBacktraceStringByTidWithMix(std::string& out, int32_t tid,
        size_t skipFrameNum, bool fast, size_t maxFrameNums = 256, bool enableKernelStack = true);

    /**
     * @brief Get formatted stacktrace string of current process
     */
    std::string GetProcessStacktrace(size_t maxFrameNums = 256,
        bool enableKernelStack = true, bool includeThreadInfo = true);

} // namespace HiviewDFX
} // namespace OHOS
```

### API 详解

#### 1. GetBacktrace()

**签名**：
```cpp
bool GetBacktrace(std::string& out, bool fast = false,
               size_t skipFrameNum = 0, size_t maxFrameNums = 256);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `out` | `std::string&` | 输出参数，调用栈结果 |
| `fast` | `bool` | `true`：使用 FP 回栈（快速但不精确）<br>`false`：使用 DWARF 回栈（精确但较慢） |
| `skipFrameNum` | `size_t` | 跳过前 N 帧 |
| `maxFrameNums` | `size_t` | 最大帧数（默认 256） |

**返回值**：
- `true`：成功
- `false`：失败

**权限要求**：无（仅访问自己的进程）

**调用示例**：
```cpp
#include "backtrace_local.h"
#include <iostream>

using namespace OHOS::HiviewDFX;

void PrintCurrentStack() {
    std::string stack;
    if (GetBacktrace(stack, false)) {
        std::cout << stack << std::endl;
    }
}
```

#### 2. GetBacktraceStringByTid()

**签名**：
```cpp
bool GetBacktraceStringByTid(std::string& out, int32_t tid,
        size_t skipFrameNum, bool fast, size_t maxFrameNums = 256,
        bool enableKernelStack = true);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `out` | `std::string&` | 输出参数 |
| `tid` | `int` | 目标线程 ID（不能等于当前线程） |
| `skipFrameNum` | `size_t` | 跳过帧数 |
| `fast` | `bool` | 回栈模式 |
| `maxFrameNums` | `size_t` | 最大帧数 |
| `enableKernelStack` | `bool` | 用户栈失败时尝试内核栈<br>**需要 ioctl 权限** |

**返回值**：
- `true`：成功
- `false`：失败

**权限要求**：
- 需要读取目标进程的 `/proc/<tid>/stack`
- `enableKernelStack=true` 需要 ioctl 系统调用权限

#### 3. PrintBacktrace()

**签名**：
```cpp
bool PrintBacktrace(int32_t fd = -1, bool fast = false, size_t maxFrameNums = 256);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `fd` | `int` | 文件描述符（小于 0 则无效，-1 表示输出到 stdout） |
| `fast` | `bool` | 回栈模式 |
| `maxFrameNums` | `size_t` | 最大帧数 |

**返回值**：
- `true`：成功
- `false`：失败

#### 4. GetProcessStacktrace()

**签名**：
```cpp
std::string GetProcessStacktrace(size_t maxFrameNums = 256,
        bool enableKernelStack = true, bool includeThreadInfo = true);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `maxFrameNums` | `size_t` | 每线程最大帧数 |
| `enableKernelStack` | `bool` | 是否尝试内核栈 |
| `includeThreadInfo` | `bool` | 是否包含线程状态信息 |

**返回值**：
- 格式化的进程栈字符串（包含所有线程）

**权限要求**：
- 需要读取所有线程的栈信息
- `enableKernelStack=true` 需要 ioctl 权限

## SignalHandler API

### 接口定义

**头文件**：`interfaces/innerkits/signal_handler/include/dfx_signal_handler.h`

**C API**：
```c
extern "C" {
    /**
     * @brief Callback function for collecting thread information during a crash
     */
    typedef void(*ThreadInfoCallBack)(char* buf, size_t len, void* ucontext);

    /**
     * @brief Registers a callback for collecting thread information
     */
    void SetThreadInfoCallback(ThreadInfoCallBack func);

    /**
     * @brief Registers a callback for retrieving stack identifier
     */
    typedef uint64_t(*GetStackIdFunc)(void);

    void DFX_SetAsyncStackCallback(GetStackIdFunc func);

    /**
     * @brief Retrieves application's running unique identifier
     */
    const char* DFX_GetAppRunningUniqueId(void);

    /**
     * @brief Sets application's running unique identifier
     */
    int DFX_SetAppRunningUniqueId(const char* appRunningUniqueId, size_t len);

    /**
     * @brief Types of crash objects for diagnostic information
     */
    enum CrashObjType : uint8_t {
        OBJ_STRING = 0,       // Null-terminated string (max 64KB)
        OBJ_MEMORY_64B,       // 64-byte memory block
        OBJ_MEMORY_256B,      // 256-byte memory block
        OBJ_MEMORY_1024B,     // 1KB memory block
        OBJ_MEMORY_2048B,     // 2KB memory block
        OBJ_MEMORY_4096B,     // 4KB memory block
    };

    /**
     * @brief Attaches diagnostic information to current crash context
     */
    uintptr_t DFX_SetCrashObj(uint8_t type, uintptr_t addr);

    /**
     * @brief Detaches diagnostic information from current crash context
     */
    void DFX_ResetCrashObj(uintptr_t crashObj);

    /**
     * @brief Configuration options for crash log generation
     */
    enum CrashLogConfigType : uint8_t {
        EXTEND_PRINT_PC_LR = 0,    // Export PC/LR registers
        CUT_OFF_LOG_FILE,          // Limit log file size
        SIMPLIFY_PRINT_MAPS,       // Simplified maps
    };

    /**
     * @brief Configures crash log generation behavior
     */
    int DFX_SetCrashLogConfig(uint8_t type, uint32_t value);

    /**
     * @brief notify watchdog thread start
     */
    int DfxNotifyWatchdogThreadStart();
} // extern "C"
```

### API 详解

#### 1. SetThreadInfoCallback()

**签名**：
```c
void SetThreadInfoCallback(ThreadInfoCallBack func);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `func` | `ThreadInfoCallBack` | 回调函数指针，传递 NULL 恢复默认行为 |

**功能**：
- 在崩溃时调用回调，允许自定义线程信息写入到缓冲区
- 缓冲区大小：`len` 参数

**使用示例**：
```c
#include "dfx_signal_handler.h"

void MyThreadInfoCallback(char* buf, size_t len, void* ucontext) {
    // 自定义线程信息处理
    // buf 最大 64KB
}

void InitSignalHandler() {
    SetThreadInfoCallback(MyThreadInfoCallback);
}
```

#### 2. DFX_SetAsyncStackCallback()

**签名**：
```c
void DFX_SetAsyncStackCallback(GetStackIdFunc func);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `func` | `GetStackIdFunc` | 返回唯一栈 ID 的回调函数 |

**功能**：
- 设置异步栈跟踪回调
- 用于 CPP-JS 混合栈场景

#### 3. DFX_SetCrashObj() / DFX_ResetCrashObj()

**签名**：
```c
uintptr_t DFX_SetCrashObj(uint8_t type, uintptr_t addr);
void DFX_ResetCrashObj(uintptr_t crashObj);
```

**CrashObjType 枚举**：

| 类型 | 大小 | 用途 |
|------|------|------|
| `OBJ_STRING` | 64KB | 字符串信息 |
| `OBJ_MEMORY_64B` | 64B | 内存块 |
| `OBJ_MEMORY_256B` | 256B | 内存块 |
| `OBJ_MEMORY_1024B` | 1KB | 内存块 |
| `OBJ_MEMORY_2048B` | 2KB | 内存块 |
| `OBJ_MEMORY_4096B` | 4KB | 内存块 |

**功能**：
- 在崩溃日志中附加诊断信息
- `addr` 指向的内存必须在崩溃前保持有效

**使用示例**：
```cpp
#include "dfx_signal_handler.h"

void AttachDiagnosticInfo() {
    std::string diagMsg = "Diagnostic information";
    uintptr_t handle = DFX_SetCrashObj(OBJ_STRING, (uintptr_t)diagMsg.c_str());
    // ... 崩溃后自动包含此信息
}
```

#### 4. DFX_SetCrashLogConfig()

**签名**：
```c
int DFX_SetCrashLogConfig(uint8_t type, uint32_t value);
```

**CrashLogConfigType 枚举**：

| 类型 | 说明 |
|------|------|
| `EXTEND_PRINT_PC_LR` | 导出 PC/LR 寄存器 |
| `CUT_OFF_LOG_FILE` | 限制日志文件大小 |
| `SIMPLIFY_PRINT_MAPS` | 简化 maps 输出 |

**返回值**：
- `0`：成功
- `-1`：失败（检查 errno）

**使用示例**：
```cpp
#include "dfx_signal_handler.h"

void ConfigureCrashLog() {
    // 导出寄存器信息
    DFX_SetCrashLogConfig(EXTEND_PRINT_PC_LR, 1);
    // 简化 maps
    DFX_SetCrashLogConfig(SIMPLIFY_PRINT_MAPS, 1);
}
```

## FaultloggerdClient API

### 接口定义

**头文件**：`interfaces/innerkits/faultloggerd_client/include/faultloggerd_client.h`

**C API**：
```c
extern "C" {
    /**
     * @brief request file descriptor
     */
    int32_t RequestFileDescriptor(int32_t type);

    /**
     * @brief request pipe file descriptor
     */
    int32_t RequestPipeFd(int32_t pid, int32_t pipeType, int (&pipeFd)[2]);

    /**
     * @brief request delete file descriptor
     */
    int32_t RequestDelPipeFd(int32_t pid);

    /**
     * @brief request file descriptor
     */
    int32_t RequestFileDescriptorEx(struct FaultLoggerdRequest* request);

    /**
     * @brief request lipeperf pipe file descriptor
     */
    int32_t RequestLitePerfPipeFd(int32_t pipeType, int (&pipeFd)[2], int timeout, bool checkLimit);

    /**
     * @brief request delete lite perf file descriptor
     */
    int32_t RequestLitePerfDelPipeFd();

    /**
     * @brief request dump stack about process
     */
    int32_t RequestSdkDump(int32_t pid, int32_t tid, int (&pipeReadFd)[2],
        bool isjson = false, int timeout = 10000);

    /**
     * @brief report sdk dump result to faultloggerd for stats collection
     */
    int32_t ReportDumpStats(struct FaultLoggerdStatsRequest *request);

    /**
     * @brief cancel coredump request
     */
    int32_t CancelCoredump(int32_t targetPid);

    /**
     * @brief start coredump request
     */
    int32_t StartCoredumpCb(int32_t targetPid, int32_t processDumpPid);

    /**
     * @brief finish coredump request
     */
    int32_t FinishCoredumpCb(int32_t targetPid, std::string& fileName, int32_t ret);

    /**
     * @brief do coredump request
     */
    std::string SaveCoredumpToFileTimeout(int32_t targetPid, int32_t timeout = 10000);
} // extern "C"
```

### API 详解

#### 1. RequestFileDescriptor()

**签名**：
```c
int32_t RequestFileDescriptor(int32_t type);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `int32_t` | 请求类型（定义在 `dfx_socket_request.h`） |

**返回值**：
- 成功：文件描述符（>= 0）
- 失败：错误码（-1, -2, -3, -4）

**错误码**（`interfaces/common/dfx_socket_request.h`）：
```cpp
typedef enum ResponseCode : int32_t {
    RECEIVE_DATA_FAILED = -4,
    SEND_DATA_FAILED = -3,
    CONNECT_FAILED = -2,
    DEFAULT_ERROR_CODE = -1,
    REQUEST_SUCCESS = 0,
    UNKNOWN_CLIENT_TYPE = 1,
    INVALID_REQUEST_DATA = 2,
    REQUEST_REJECT = 3,
    ABNORMAL_SERVICE = 4,
    SDK_DUMP_REPEAT = 5,
    SDK_DUMP_NOPROC = 6,
    SDK_PROCESS_CRASHED = 7,
    CORE_DUMP_REPEAT = 8,
    CORE_PROCESS_CRASHED = 9,
    CORE_DUMP_NOPROC = 10,
    CORE_DUMP_CANCEL = 11,
    CORE_DUMP_GENERATE_FAIL = 12,
    RESOURCE_LIMIT = 13,
};
```

#### 2. RequestPipeFd()

**签名**：
```c
int32_t RequestPipeFd(int32_t pid, int32_t pipeType, int (&pipeFd)[2]);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `pid` | `int32_t` | 目标进程 ID |
| `pipeType` | `int32_t` | 管道类型 |
| `pipeFd` | `int (&)[2]` | 输出数组，`pipeFd[0]` 读端，`pipeFd[1]` 写端 |

**返回值**：
- 成功：0，pipeFd 填充
- 失败：错误码

#### 3. RequestSdkDump()

**签名**：
```c
int32_t RequestSdkDump(int32_t pid, int32_t tid, int (&pipeReadFd)[2],
        bool isjson = false, int timeout = 10000);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `pid` | `int32_t` | 目标进程 ID |
| `tid` | `int32_t` | 目标线程 ID |
| `pipeReadFd` | `int (&)[2]` | 管道描述符数组 |
| `isjson` | `bool` | 是否返回 JSON 格式 |
| `timeout` | `int` | 超时时间（毫秒，默认 10000） |

**返回值**：
- 成功：0，pipeReadFd 填充
- 失败：错误码

#### 4. SaveCoredumpToFileTimeout()

**签名**：
```cpp
std::string SaveCoredumpToFileTimeout(int32_t targetPid, int32_t timeout = 10000);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `targetPid` | `int32_t` | 目标进程 ID |
| `timeout` | `int32_t` | 超时时间（毫秒） |

**返回值**：
- 成功：coredump 文件名（字符串）
- 失败：空字符串

**Coredump 流程**：
1. `StartCoredumpCb()` - 开始 coredump
2. `SaveCoredumpToFileTimeout()` - 保存文件（可超时）
3. `FinishCoredumpCb()` - 完成并上报状态

## Rust PanicHandler API

### 接口定义

**头文件**：`interfaces/rust/panic_handler`

**Rust API**：
```rust
pub fn init();
```

**功能**：
- 注册 panic 处理器
- 捕获 Rust panic 并生成故障日志

**使用示例**：
```rust
extern crate panic_handler;

fn main() {
    panic_handler::init();
    // 触发 panic 会自动生成日志
    panic!("This is a panic!");
}
```

**故障日志位置**：
- `/data/log/faultlog/faultlogger/`

## 权限要求总结

| API | 权限要求 | 说明 |
|------|------------|------|
| **DumpCatcher::DumpCatch()** | root/system 或自己用户 | 需要 ptrace 和 /proc 权限 |
| **DumpCatcher::DumpCatchFd()** | root/system 或自己用户 | 需要自己进程权限 |
| **Backtrace::GetBacktrace()** | 无 | 仅访问自己进程 |
| **Backtrace::GetBacktraceStringByTid()** | 无 | 仅访问自己进程 |
| **SignalHandler APIs** | 无 | 自动注入到进程 |
| **FaultloggerdClient APIs** | 无 | Socket 通信，服务端权限验证 |
| **Rust PanicHandler** | 无 | 自动注入 |

## 常见问题

### Q1: DumpCatcher 调用失败返回 false

**可能原因**：
1. 权限不足（非 root/system，且目标进程不属于自己用户）
2. 目标进程不存在或已退出
3. 目标进程在沙箱中无法访问
4. 资源限制（`RESOURCE_LIMIT`）

**排查方法**：
- 检查调用者 UID（`getuid()`）
- 检查目标进程状态（`/proc/<pid>/status`）
- 查看 faultloggerd 日志（`hilog | grep faultloggerd`）

### Q2: Backtrace 返回空栈

**可能原因**：
1. 二进制缺少调试信息（unwind-tables）
2. 栈帧被优化掉了
3. FP 回栈在优化模式下失败

**排查方法**：
- 确认编译时包含 `unwind-tables`
- 尝试使用 DWARF 回栈（`fast=false`）

### Q3: Socket 连接失败

**错误码**：`CONNECT_FAILED (-2)`

**可能原因**：
1. faultloggerd 服务未启动
2. Socket 路径不存在或权限问题
3. SELinux 策略阻止

**排查方法**：
- 检查服务状态：`ps -A | grep faultloggerd`
- 检查 Socket 权限：`ls -l /dev/unix/socket/faultloggerd.*`
- 检查 SELinux：`getenforce`

## 相关跳转

- [项目定位与核心能力](00_Overview.md) - 项目概览
- [架构设计](02_Architecture.md) - 完整处理流程
- [目录结构与模块职责](01_Directory_Structure.md) - 代码组织
- [安全风险评审](07_Security_Review.md) - 权限控制详解
