# 内部 API

## 目的

本文档说明 hiperf 内部模块的接口、职责和依赖关系。

## 适用范围

- hiperf 内部开发者
- 需要扩展 hiperf 功能的开发者

## 模块职责

### 1. 命令框架模块

#### Command

**文件**: `include/command.h`, `src/command.cpp`

**职责**: 命令分发和管理

**关键接口**:
```cpp
class Command {
public:
    static void DispatchCommands(std::vector<std::string> &args);
    static void RegisterMainCommand(const std::string &name, 
                                    std::function<void(std::vector<std::string> &)> callback);
};
```

#### SubCommand

**文件**: `include/subcommand.h`, `src/subcommand.cpp`

**职责**: 子命令基类

**关键接口**:
```cpp
class SubCommand {
public:
    virtual HiperfError OnSubCommand(std::vector<std::string> &args) = 0;
    virtual bool ParseOption(std::vector<std::string> &args) = 0;
    static SubCommand *GetSubCommand(const std::string &name);
    static void RegisterSubCommand(const std::string &name, SubCommand *subCommand);
};
```

### 2. 核心功能模块

#### PerfEvents

**文件**: `include/perf_events.h`, `src/perf_events.cpp`

**职责**: perf_event 管理和数据采集

**关键接口**:
```cpp
class PerfEvents {
public:
    bool AddEvents(const std::vector<std::string> &events);
    bool AddEventsToGroup(const std::vector<std::string> &events);
    bool StartTracking();
    bool StopTracking();
    bool PauseTracking();
    bool ResumeTracking();
    bool ReadRecords(RecordCallback callback);
};
```

**依赖**: Linux perf_event 内核接口

#### VirtualRuntime

**文件**: `include/virtual_runtime.h`, `src/virtual_runtime.cpp`

**职责**: 虚拟运行时环境管理

**关键接口**:
```cpp
class VirtualRuntime {
public:
    void UpdateFromRecord(PerfEventRecord &record);
    void UpdateKernelSpaceMaps();
    void UpdateKernelModulesSpaceMaps();
    bool SetSymbolsPaths(const std::vector<std::string> &symbolsPaths);
    const std::vector<std::unique_ptr<SymbolsFile>> &GetSymbolsFiles() const;
};
```

**依赖**: SymbolsFile, VirtualThread, CallStack

#### SymbolsFile

**文件**: `include/symbols_file.h`, `src/symbols_file.cpp`

**职责**: 符号文件解析

**关键接口**:
```cpp
class SymbolsFile {
public:
    bool LoadDebugInfo(const MemMap &map);
    Symbol GetSymbolWithVaddr(uint64_t vaddr, uint64_t offset);
    static std::unique_ptr<SymbolsFile> Create(const std::string &fileName);
};
```

**依赖**: ELF 解析, HAP 解析

#### CallStack

**文件**: `include/callstack.h`, `src/callstack.cpp`

**职责**: 调用链回溯

**关键接口**:
```cpp
class CallStack {
public:
    bool Unwind(const VirtualThread &thread, const u64 *regs, 
                std::vector<uint64_t> &callChain);
    bool UnwindCallChainDwarf(const VirtualThread &thread, 
                              const u64 *regs, 
                              std::vector<uint64_t> &callChain);
};
```

**依赖**: libunwinder

### 3. 数据文件模块

#### PerfFileWriter

**文件**: `include/perf_file_writer.h`, `src/perf_file_writer.cpp`

**职责**: 数据文件写入

**关键接口**:
```cpp
class PerfFileWriter {
public:
    bool Create(const std::string &fileName, bool compress = false);
    bool WriteRecord(const PerfEventRecord &record);
    bool WriteFeatureSection();
    bool Close();
};
```

#### PerfFileReader

**文件**: `include/perf_file_reader.h`, `src/perf_file_reader.cpp`

**职责**: 数据文件读取

**关键接口**:
```cpp
class PerfFileReader {
public:
    static std::unique_ptr<PerfFileReader> Open(const std::string &fileName);
    bool ReadAttrSection();
    bool ReadRecord(RecordCallback callback);
    bool ReadFeatureSection();
};
```

### 4. 报告生成模块

#### Report

**文件**: `include/report.h`, `src/report.cpp`

**职责**: 报告生成基类

**关键接口**:
```cpp
class Report {
public:
    bool LoadPerfData(const std::string &fileName);
    bool OutputReport();
};
```

#### ReportJSONFile

**文件**: `include/report_json_file.h`, `src/report_json_file.cpp`

**职责**: JSON 格式报告

#### ReportProtobufFile

**文件**: `include/report_protobuf_file.h`, `src/report_protobuf_file.cpp`

**职责**: ProtoBuf 格式报告

## 模块依赖图

```
Command
└── SubCommand
    ├── SubCommandList
    ├── SubCommandStat
    ├── SubCommandRecord
    │   ├── PerfEvents
    │   │   └── RingBuffer
    │   ├── VirtualRuntime
    │   │   ├── VirtualThread
    │   │   ├── SymbolsFile
    │   │   │   └── ElfFile
    │   │   └── CallStack
    │   │       └── libunwinder
    │   └── PerfFileWriter
    ├── SubCommandDump
    │   └── PerfFileReader
    └── SubCommandReport
        ├── PerfFileReader
        └── Report
            ├── ReportJSONFile
            └── ReportProtobufFile
```

## 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| Command | 稳定 | 命令框架基类 |
| SubCommand | 稳定 | 子命令基类 |
| PerfEvents | 较稳定 | 核心功能 |
| VirtualRuntime | 可能变化 | 内部实现细节 |
| SymbolsFile | 较稳定 | 符号解析 |
| CallStack | 稳定 | 调用链回溯 |
| PerfFileWriter | 稳定 | 文件格式稳定 |
| PerfFileReader | 稳定 | 文件格式稳定 |

## 相关跳转

- [架构说明](03_Architecture.md) - 系统架构
- [对外 API](04_Public_API.md) - 对外接口
- [目录结构](02_Directory_Structure.md) - 代码组织
