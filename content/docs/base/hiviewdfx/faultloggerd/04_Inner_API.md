# FaultLoggerd 内部 API

## 目的

本文档介绍 faultloggerd 模块间的内部接口和依赖关系。

## 适用范围

- 目标读者：模块开发者、架构设计师
- 内容范围：生产代码（排除测试目录）

## 模块依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                   应用/工具层                          │
└─────────────────────┬─────────────────────────────────────┘
                      │
        ┌───────────┴──────────┐
        │                       │
  [dump_catcher]        [backtrace]        [signal_handler]
        │                       │
        │                       │
        └───────────┬───────────┘
                    │
         ┌──────────┴───────────┐
         │                      │
    [unwinder]            [procinfo]
         │                      │
         └───────────┬───────────┘
                      │
              ┌───────────┴───────────┐
              │                      │
    [faultloggerd_client]    [formatter]
              │                      │
              └───────────┬───────────┘
                          │
                   ┌──────────┴───────────┐
                   │                      │
          [socket_server]        [coredump]
                   │                      │
                   └───────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    │                      │
          [fault_logger_service]    [temp_file_manager]
                    │                      │
                    └───────────┬───────────┘
                                 │
                           [epoll_manager]
```

## 核心模块接口

### 1. Unwinder（符号解析引擎）

**头文件**：`interfaces/innerkits/unwinder/include/unwinder.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class Unwinder {
public:
    Unwinder() = default;
    virtual ~Unwinder() = default;

    /**
     * @brief Unwind stack for specific thread
     */
    virtual int32_t UnwindStack(uint64_t* framePCs, size_t maxFrames,
                               uint64_t* frameSPs, size_t* frameSizes) = 0;
};
} // namespace HiviewDFX
} // namespace OHOS
```

**依赖**：
- `dfx_elf_parser` - ELF 文件解析
- `dfx_symbols` - 符号表管理
- `dwarf_entry_parser` - DWARF 调试信息解析
- `src/registers/` - 架构相关寄存器处理

**稳定性**：✅ 稳定（公共接口）
**替代性**：❌ 不可替换（核心引擎）

### 2. ProcInfo（进程信息）

**头文件**：`interfaces/innerkits/procinfo/include/procinfo.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
/**
 * @brief Read process name from /proc/[pid]/cmdline
 */
std::string GetProcessName(int pid);

/**
 * @brief Read thread info from /proc/[pid]/task
 */
bool GetThreadInfo(int pid, int tid, ThreadInfo& threadInfo);

/**
 * @brief Read memory maps from /proc/[pid]/maps
 */
bool GetMaps(int pid, std::vector<MapItem>& maps);

} // namespace HiviewDFX
} // namespace OHOS
```

**依赖**：
- Linux `/proc` 文件系统

**稳定性**：✅ 稳定（基于标准 procfs）

### 3. StackPrinter（栈打印）

**头文件**：`interfaces/innerkits/stack_printer/include/stack_printer.h`

**稳定性**：✅ 稳定（工具函数）

### 4. Formatter（JSON 格式化）

**头文件**：`interfaces/innerkits/formatter/include/dfx_json_formatter.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class JsonStackFormatter {
public:
    /**
     * @brief Format stack frames to JSON
     */
    static std::string FormatToJson(const std::vector<Frame>& frames,
                                   const std::string& processName);
};
} // namespace HiviewDFX
} // namespace OHOS
```

**稳定性**：✅ 稳定

### 5. AsyncStack（异步栈）

**头文件**：`interfaces/innerkits/async_stack/include/async_stack.h`

**功能**：
- 异步调用栈跟踪
- CPP-JS 混合栈支持
- 唯一栈表管理

**稳定性**：✅ 稳定

### 6. CrashException（崩溃异常）

**头文件**：`interfaces/innerkits/crash_exception/crash_exception.h`

**功能**：
- 崩溃异常类型定义
- 异常信息封装

**稳定性**：✅ 稳定

## 服务端接口

### 1. SocketServer（Socket 服务器框架）

**头文件**：`services/fault_logger_server.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class SocketServer {
public:
    bool Init();
    bool StartEpoll(int32_t maxConnection, int32_t timeout);
    void StopEpoll();

private:
    void AddService(const std::string& name, std::unique_ptr<IFaultLoggerService> service);
    bool AddServerListener(const std::string& socketName);
};
} // namespace HiviewDFX
} // namespace OHOS
```

**依赖**：
- `epoll_manager` - 事件循环
- `IFaultLoggerService` - 服务接口

**稳定性**：✅ 稳定

### 2. IFaultLoggerService（服务接口）

**头文件**：`services/fault_logger_service.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class IFaultLoggerService {
public:
    virtual ~IFaultLoggerService() = default;
    virtual int32_t OnRequest(const std::string& socketName, int32_t connectionFd,
                                const void* requestData) = 0;
};
} // namespace HiviewDFX
} // namespace OHOS
```

**实现类**：
- `FileDesService` - 文件描述符服务
- `ExceptionReportService` - 异常上报服务
- `StatsService` - 统计服务
- `SdkDumpService` - SDK dump 服务
- `PipeService` - 管道服务
- `CoredumpManagerService` - Coredump 管理服务

**稳定性**：✅ 稳定（接口抽象）

### 3. TempFileManager（临时文件管理）

**头文件**：`services/temp_file_manager.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class TempFileManager {
public:
    bool Init();
    std::string CreateTempFile(const std::string& type, int pid, const std::string& fileName);
    bool DeleteTempFile(const std::string& filePath);
    void ClearTempFiles();
};
} // namespace HiviewDFX
} // namespace OHOS
```

**依赖**：
- Linux 文件系统 API

**稳定性**：✅ 稳定

### 4. EpollManager（事件循环）

**头文件**：`services/epoll_manager.h`

**核心接口**：
```cpp
namespace OHOS {
namespace HiviewDFX {
class EpollManager {
public:
    bool Init(int32_t maxEpollEvent);
    bool StartEpoll(int32_t maxConnection, int32_t timeout);
    void StopEpoll();
    bool AddListener(std::unique_ptr<SocketServerListener> listener);
    bool DelListener(int32_t fd);
};
} // namespace HiviewDFX
} // namespace OHOS
```

**稳定性**：✅ 稳定

## 依赖方向

### 正向依赖

```
fault_logger_service → unwinder (符号解析)
fault_logger_service → procinfo (进程信息)
fault_logger_service → stack_printer (栈打印)
fault_logger_service → formatter (格式化)
temp_file_manager → Linux VFS
epoll_manager → Linux epoll API
```

### 反向依赖（避免循环）

```
dump_catcher → faultloggerd_client (客户端)
backtrace → unwinder
signal_handler → unwinder
processdump → unwinder
processdump → procinfo
```

## 接口稳定性标注

### 稳定接口（可安全使用）

| 模块 | 接口 | 稳定性说明 |
|------|------|-----------|
| **Unwinder** | `interfaces/innerkits/unwinder/include/unwinder.h` | 公共接口，核心功能 |
| **ProcInfo** | `interfaces/innerkits/procinfo/include/procinfo.h` | 基于标准 procfs |
| **SocketServer** | `services/fault_logger_server.h` | 服务端框架 |
| **IFaultLoggerService** | `services/fault_logger_service.h` | 抽象接口 |
| **TempFileManager** | `services/temp_file_manager.h` | 文件管理 |
| **EpollManager** | `services/epoll_manager.h` | 事件循环 |
| **JsonStackFormatter** | `interfaces/innerkits/formatter/include/dfx_json_formatter.h` | JSON 格式化 |

### 不稳定接口（内部实现，可能变更）

| 模块 | 接口 | 不稳定性说明 |
|------|------|------------|
| **FaultLoggerService 具体实现** | `services/fault_logger_service.cpp` | 业务逻辑实现，可能重构 |
| **Unwinder 内部解析器** | `interfaces/innerkits/unwinder/src/` | DWARF/ELF 解析细节 |
| **AsyncStack 内部实现** | `interfaces/innerkits/async_stack/async_stack.cpp` | 异步栈逻辑 |

### 可替换点

| 模块 | 可替换性 | 替换方案 |
|------|-----------|----------|
| **Unwinder** | ❌ 不可替换 | 核心栈回退引擎 |
| **ProcInfo** | ✅ 可替换 | procfs 是标准接口 |
| **Formatter** | ✅ 可替换 | 独立格式化模块 |
| **TempFileManager** | ✅ 可替换 | 文件管理逻辑 |
| **EpollManager** | ❌ 不可替换 | 事件循环框架 |

## 模块间通信

### 调用链示例：DumpCatcher → Unwinder

```cpp
// DumpCatcher 调用 Unwinder 解析符号
DfxDumpCatcher::DumpCatch()
  → Unwinder::UnwindStack()
  → DfxElfParser::GetSymbol()
  → DfxSymbols::Demangle()
```

### 调用链示例：ProcessDump → ProcInfo

```cpp
// ProcessDump 读取进程信息
ProcessDump::Execute()
  → ProcInfo::GetProcessName()
  → ProcInfo::GetThreadInfo()
  → ProcInfo::GetMaps()
```

## TODO

- [ ] 📋 补充每个模块的详细接口列表
- [ ] 📋 补充模块间数据流说明
- [ ] 📋 分析循环依赖风险

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md) - 代码组织
- [架构设计](02_Architecture.md) - 组件交互
- [对外 API](03_External_API.md) - 公共 API
