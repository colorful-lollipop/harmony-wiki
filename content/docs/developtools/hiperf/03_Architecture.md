# 架构说明

## 目的

本文档说明 hiperf 的系统架构、组件关系、数据流、线程模型和关键时序。

## 适用范围

- 系统架构师
- 性能优化开发者
- 需要理解内部实现的人员

## 系统架构图

```mermaid
graph TB
    subgraph "用户层"
        CLI["命令行工具<br/>hiperf"]
        API["C++ API<br/>hiperf_client"]
        Script["Python 脚本<br/>command_script.py"]
    end
    
    subgraph "命令层"
        CMD["Command 分发器"]
        SUB["SubCommand 子命令"]
    end
    
    subgraph "核心功能层"
        PE["PerfEvents<br/>事件管理"]
        VR["VirtualRuntime<br/>虚拟运行时"]
        SF["SymbolsFile<br/>符号文件"]
        CS["CallStack<br/>调用链回溯"]
        RB["RingBuffer<br/>环形缓冲区"]
    end
    
    subgraph "数据层"
        PFR["PerfFileReader<br/>数据读取"]
        PFW["PerfFileWriter<br/>数据写入"]
        RPT["Report<br/>报告生成"]
    end
    
    subgraph "系统接口层"
        KE["Kernel perf_event"]
        PROC["/proc 文件系统"]
        IPC["IPC (SAMGR)"]
    end
    
    CLI --> CMD
    API --> CLI
    Script --> CLI
    CMD --> SUB
    SUB --> PE
    SUB --> VR
    SUB --> PFW
    PE --> RB
    RB --> VR
    VR --> SF
    VR --> CS
    PFR --> VR
    VR --> RPT
    PE --> KE
    VR --> PROC
    SF --> PROC
    SUB --> IPC
```

## 组件关系

### 1. 命令分发组件

```
┌─────────────────────────────────────────┐
│              main.cpp                   │
│         (程序入口，权限检查)              │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│           Command 类                    │
│      (命令解析和分发)                    │
│  ┌─────────────────────────────────┐   │
│  │  RegisterCommandComponent()     │   │
│  │  - RegisterMainCommandDebug()   │   │
│  │  - RegisterSubCommandHelp()     │   │
│  │  - RegisterSubCommandStat()     │   │
│  │  - RegisterSubCommandRecord()   │   │
│  │  - RegisterSubCommandDump()     │   │
│  │  - RegisterSubCommandReport()   │   │
│  └─────────────────────────────────┘   │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         SubCommand 子命令               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  list   │ │  stat   │ │ record  │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│  ┌─────────┐ ┌─────────┐               │
│  │  dump   │ │ report  │               │
│  └─────────┘ └─────────┘               │
└─────────────────────────────────────────┘
```

### 2. 核心功能组件

#### PerfEvents - 事件管理

**职责**: 管理 perf_event 的创建、配置和数据采集

**关键类**: `PerfEvents` (`include/perf_events.h`)

```
┌─────────────────────────────────────────┐
│            PerfEvents                   │
├─────────────────────────────────────────┤
│  事件配置                                │
│  - PERF_HW_CONFIGS (硬件事件)            │
│  - PERF_SW_CONFIGS (软件事件)            │
│  - PERF_RAW_CONFIGS (原始事件)           │
├─────────────────────────────────────────┤
│  事件组管理                              │
│  - AddEvents()                          │
│  - AddEventsToGroup()                   │
├─────────────────────────────────────────┤
│  数据采集                                │
│  - StartTracking()                      │
│  - StopTracking()                       │
│  - PauseTracking()                      │
│  - ResumeTracking()                     │
├─────────────────────────────────────────┤
│  环形缓冲区                              │
│  - RingBuffer 数组                       │
└─────────────────────────────────────────┘
```

#### VirtualRuntime - 虚拟运行时

**职责**: 维护进程、线程、内存映射和符号信息

**关键类**: `VirtualRuntime` (`include/virtual_runtime.h`)

```
┌─────────────────────────────────────────┐
│          VirtualRuntime                 │
├─────────────────────────────────────────┤
│  线程管理                                │
│  - virtualThreads_ (pid -> VirtualThread)│
├─────────────────────────────────────────┤
│  符号文件管理                             │
│  - symbolsFiles_ (SymbolsFile 列表)      │
├─────────────────────────────────────────┤
│  内核符号                                │
│  - kernelSymbols_                       │
│  - kernelThreadSymbols_                 │
├─────────────────────────────────────────┤
│  调用链回溯                              │
│  - CallStack 实例                        │
└─────────────────────────────────────────┘
```

#### SymbolsFile - 符号文件

**职责**: 解析 ELF/HAP 文件的符号信息

**关键类**: `SymbolsFile` (`include/symbols_file.h`)

```
┌─────────────────────────────────────────┐
│           SymbolsFile                   │
├─────────────────────────────────────────┤
│  文件类型                                │
│  - SYMBOL_ELF_FILE                      │
│  - SYMBOL_KERNEL_FILE                   │
│  - SYMBOL_HAP_FILE                      │
├─────────────────────────────────────────┤
│  符号解析                                │
│  - LoadDebugInfo()                      │
│  - GetSymbolWithVaddr()                 │
├─────────────────────────────────────────┤
│  特殊处理                                │
│  - HAP 文件解析                          │
│  - MiniDebugInfo 解析                    │
│  - VDSO 处理                             │
└─────────────────────────────────────────┘
```

## 数据流

### 采样数据流（record 命令）

```mermaid
sequenceDiagram
    participant User as 用户
    participant SC as SubCommandRecord
    participant PE as PerfEvents
    participant KE as Kernel
    participant RB as RingBuffer
    participant VR as VirtualRuntime
    participant PFW as PerfFileWriter
    
    User->>SC: hiperf record
    SC->>PE: PreparePerfEvent()
    PE->>KE: perf_event_open()
    KE-->>PE: fd
    PE->>RB: mmap()
    
    loop 采样循环
        KE->>RB: 写入采样数据
        SC->>PE: ReadRecord()
        PE->>RB: 读取原始数据
        RB-->>PE: PerfEventRecord
        PE->>VR: UpdateFromRecord()
        VR->>VR: 解析符号/调用链
        SC->>PFW: WriteRecord()
    end
    
    SC->>PFW: Close()
    PFW-->>User: perf.data
```

### 报告数据流（report 命令）

```mermaid
sequenceDiagram
    participant User as 用户
    participant SC as SubCommandReport
    participant PFR as PerfFileReader
    participant VR as VirtualRuntime
    participant RPT as Report
    
    User->>SC: hiperf report
    SC->>PFR: Open()
    PFR->>PFR: ReadHeader()
    PFR->>PFR: ReadAttrSection()
    
    loop 读取记录
        PFR->>PFR: ReadRecord()
        PFR->>VR: UpdateFromRecord()
        VR->>VR: 解析符号/调用链
    end
    
    SC->>RPT: OutputReport()
    RPT->>RPT: 生成报告
    RPT-->>User: 输出结果
```

## 线程模型

### 1. 主线程（record 命令）

```
┌─────────────────────────────────────────┐
│              主线程                      │
├─────────────────────────────────────────┤
│  1. 解析命令参数                         │
│  2. 准备 PerfEvent                       │
│  3. 创建采样线程                         │
│  4. 等待采样完成                         │
│  5. 后处理和数据写入                      │
└─────────────────────────────────────────┘
```

### 2. 采样线程

```
┌─────────────────────────────────────────┐
│            采样线程                      │
├─────────────────────────────────────────┤
│  while (running) {                      │
│    1. poll() 等待数据                    │
│    2. 从 RingBuffer 读取数据              │
│    3. 解析 PerfEventRecord               │
│    4. 调用回调函数处理                    │
│  }                                      │
└─────────────────────────────────────────┘
```

代码位置: `src/perf_events.cpp` (具体实现在 `ReadRecords()` 相关函数)

### 3. 控制线程（Client API）

当使用 `hiperf_client` API 时，会创建额外的控制线程：

```
┌─────────────────────────────────────────┐
│           Client 控制线程                │
├─────────────────────────────────────────┤
│  ClientCommandHandle()                  │
│  - 读取管道命令                          │
│  - 执行控制操作（start/stop/pause）       │
├─────────────────────────────────────────┤
│  ReplyCommandHandle()                   │
│  - 发送响应到客户端                      │
└─────────────────────────────────────────┘
```

代码位置: `src/subcommand_record.cpp:346-348`

## 关键时序

### 1. 采样启动时序

```mermaid
sequenceDiagram
    participant Main as 主线程
    participant PE as PerfEvents
    participant Kernel as 内核
    
    Main->>PE: PrepareFdEvents()
    PE->>PE: 配置 perf_event_attr
    loop 每个 CPU/事件
        PE->>Kernel: perf_event_open()
        Kernel-->>PE: 返回 fd
        PE->>PE: mmap() 创建 RingBuffer
    end
    PE->>PE: PrepareFdEvents() 完成
    
    Main->>PE: CreateFdEvents()
    PE->>PE: 设置采样回调
    
    Main->>PE: StartTracking()
    PE->>Kernel: ioctl(PERF_EVENT_IOC_ENABLE)
    Kernel-->>PE: 开始采样
```

### 2. 单条采样记录处理时序

```mermaid
sequenceDiagram
    participant Kernel as 内核
    participant RB as RingBuffer
    participant PE as PerfEvents
    participant VR as VirtualRuntime
    participant SC as SubCommandRecord
    
    Kernel->>RB: 写入采样数据
    Note over Kernel,RB: struct perf_event_header + data
    
    PE->>RB: 读取 header
    RB-->>PE: header (type, misc, size)
    
    PE->>RB: 读取数据体
    RB-->>PE: 采样数据
    
    PE->>PE: 创建 PerfEventRecord
    
    PE->>SC: 回调 ProcessRecord()
    SC->>VR: UpdateFromRecord()
    
    alt Sample 类型
        VR->>VR: 更新线程信息
        VR->>VR: 更新内存映射
        VR->>CS: 调用链回溯
        CS-->>VR: 调用链
        VR->>SF: 符号解析
        SF-->>VR: 符号名
    end
    
    SC->>PFW: WriteRecord()
```

### 3. API 调用时序

```mermaid
sequenceDiagram
    participant App as 应用
    participant Client as HiperfClient::Client
    participant Pipe as 匿名管道
    participant Hiperf as hiperf 进程
    
    App->>Client: Start(option)
    Client->>Client: fork()
    
    alt 子进程
        Client->>Hiperf: execv(hiperf)
        Hiperf->>Hiperf: 初始化
        Hiperf->>Pipe: 写入 "OK\n"
    else 父进程
        Client->>Pipe: 读取响应
        Pipe-->>Client: "OK\n"
        Client-->>App: 返回 true
    end
    
    App->>Client: Pause()
    Client->>Pipe: 写入 "PAUSE\n"
    Pipe->>Hiperf: 读取命令
    Hiperf->>Hiperf: PauseTracking()
    Hiperf->>Pipe: 写入 "OK\n"
    
    App->>Client: Stop()
    Client->>Pipe: 写入 "STOP\n"
    Pipe->>Hiperf: 读取命令
    Hiperf->>Hiperf: StopTracking()
    Hiperf->>Hiperf: 保存数据文件
    Hiperf->>Pipe: 写入 "OK\n"
    Hiperf->>Hiperf: exit()
```

## 内存模型

### RingBuffer 布局

```
┌─────────────────────────────────────────────────────────────┐
│                     RingBuffer 内存布局                      │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐   │
│  │              perf_event_mmap_page                   │   │
│  │  (元数据页，包含读写位置、事件计数等)                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                         ↓                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              数据页 (mmap_pages 个)                  │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐               │   │
│  │  │ Record 1│ │ Record 2│ │ Record 3│ ...            │   │
│  │  └─────────┘ └─────────┘ └─────────┘               │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

代码位置: `include/ring_buffer.h`

### VirtualRuntime 数据关系

```
┌─────────────────────────────────────────┐
│         VirtualRuntime                  │
│  ┌─────────────────────────────────┐   │
│  │   virtualThreads_               │   │
│  │   (map<pid_t, VirtualThread>)   │   │
│  │                                 │   │
│  │   pid=1000 ────────┐            │   │
│  │   pid=1001 ────────┼──────┐     │   │
│  │   ...              │      │     │   │
│  └────────────────────┼──────┼─────┘   │
│                       │      │          │
│                       ▼      ▼          │
│  ┌─────────────────────────────────┐   │
│  │        VirtualThread            │   │
│  │  ┌─────────────────────────┐   │   │
│  │  │      threadMaps_        │   │   │
│  │  │  (vector<MemMap>)       │   │   │
│  │  │                         │   │   │
│  │  │  map[0] ──────┐         │   │   │
│  │  │  map[1] ──────┼────┐    │   │   │
│  │  └───────────────┼────┼────┘   │   │
│  │                  │    │        │   │
│  │                  ▼    ▼        │   │
│  │  ┌─────────────────────────┐   │   │
│  │  │  MemMap (内存映射)       │   │   │
│  │  │  - beginVaddr           │   │   │
│  │  │  - len                  │   │   │
│  │  │  - offset               │   │   │
│  │  │  - symbolFileIndex ────┼───┼───┼────┐
│  │  └─────────────────────────┘   │   │    │
│  └─────────────────────────────────┘   │    │
│                                        │    │
│  ┌─────────────────────────────────┐   │    │
│  │      symbolsFiles_              │   │    │
│  │  (vector<unique_ptr<SymbolsFile>>)│   │    │
│  │                                 │   │    │
│  │  [0] ──────────────────────────┼───┘    │
│  │  [1] ──────────────────────────┼────────┘
│  └─────────────────────────────────┘
└─────────────────────────────────────────┘
```

## 关键结论

1. **分层架构**: 用户层 → 命令层 → 核心层 → 系统层，职责清晰
2. **事件驱动**: 基于 Linux perf_event 的事件驱动架构
3. **零拷贝**: 通过 mmap 实现内核到用户空间的零拷贝数据传输
4. **延迟解析**: 符号解析延迟到数据读取阶段，减少采样开销
5. **插件化**: 子命令模式支持功能扩展

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 代码组织
- [对外 API](04_Public_API.md) - API 使用
- [内部 API](05_Internal_API.md) - 内部接口
- [调用链附录](appendix/Callgraphs.md) - 详细调用链
