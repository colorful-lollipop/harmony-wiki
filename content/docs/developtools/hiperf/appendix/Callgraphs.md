# 附录：关键调用链

## 目的

本文档记录 hiperf 关键功能的调用链，用于代码理解和调试。

## 采样启动调用链

```
main()
└── Command::DispatchCommands()
    └── SubCommandRecord::OnSubCommand()
        ├── SubCommandRecord::GetOptions()
        ├── SubCommandRecord::CheckOptions()
        ├── SubCommandRecord::PreparePerfEvent()
        │   └── PerfEvents::AddEvents()
        │       └── PerfEvents::CreateFdEvents()
        │           ├── syscall(perf_event_open)
        │           └── mmap()
        ├── SubCommandRecord::PrepareVirtualRuntime()
        │   └── VirtualRuntime::UpdateKernelSpaceMaps()
        └── SubCommandRecord::StartSamplingAndFile()
            └── PerfEvents::StartTracking()
                └── ioctl(PERF_EVENT_IOC_ENABLE)
```

## 采样数据处理调用链

```
PerfEvents::ReadRecords()
└── PerfEvents::ReadRecordFromBuf()
    ├── RingBuffer::GetData()
    └── SubCommandRecord::ProcessRecord() [callback]
        ├── PerfEventRecord::GetType()
        ├── VirtualRuntime::UpdateFromRecord()
        │   ├── VirtualRuntime::UpdateThread()
        │   ├── VirtualRuntime::UpdateMap()
        │   └── VirtualRuntime::UpdateCallChain()
        │       └── CallStack::Unwind()
        └── PerfFileWriter::WriteRecord()
```

## 符号解析调用链

```
VirtualRuntime::UpdateFromRecord()
└── VirtualRuntime::GetSymbol()
    └── SymbolsFile::GetSymbolWithVaddr()
        ├── SymbolsFile::LoadDebugInfo() [if needed]
        │   ├── ElfFile::Parse()
        │   └── DwarfEncoding::Parse()
        └── SymbolsFile::SearchSymbol()
```

## 报告生成调用链

```
SubCommandReport::OnSubCommand()
└── SubCommandReport::OutputReport()
    ├── PerfFileReader::Open()
    │   ├── PerfFileReader::ReadHeader()
    │   ├── PerfFileReader::ReadAttrSection()
    │   └── PerfFileReader::ReadFeatureSection()
    ├── VirtualRuntime::UpdateFromRecord() [for each record]
    └── Report::Output()
        ├── ReportJSONFile::Output() [if JSON]
        └── ReportProtobufFile::Output() [if ProtoBuf]
```

## API 调用链

### hiperf_client 启动

```
Client::Start(RecordOption)
├── Client::Setup()
├── pipe() [create pipes]
├── fork()
│   └── ChildProcessHandle()
│       └── Client::ChildRunExecv()
│           └── execv("/system/bin/hiperf")
└── ParentHandleProcess()
    └── Client::WaitCommandReply()
        └── poll() [wait for "OK\n"]
```

### hiperf_local 采样

```
Lperf::StartProcessStackSampling()
└── Lperf::Impl::StartProcessStackSampling()
    └── LitePerf::StartProcessStackSampling()
        └── libdfx_dumpcatcher [external library]
```

## 相关跳转

- [架构说明](../03_Architecture.md) - 系统架构
- [内部 API](../05_Internal_API.md) - 模块接口
