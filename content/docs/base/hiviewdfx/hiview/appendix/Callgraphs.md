# 关键调用链

## 概述

本文档描述 Hiview 模块的关键调用链，从入口到核心逻辑的完整路径。

---

## 启动调用链

### main() → 服务启动

```
main.cpp:29
    │
    ├── MemoryUtil::DisableThreadCache()
    │   └── libc: mallopt(M_THREAD_DISABLE_CACHE, ...)
    │
    ├── HiviewPlatform::GetInstance()
    │   └── HiviewPlatform::InitEnvironment()
    │       ├── PluginBundle::LoadConfig()
    │       │   └── Load: /system/etc/hiview/plugin_config
    │       │
    │       ├── PluginFactory::CreatePlugins()
    │       │   ├── Create: SyseventSourcePlugin
    │       │   ├── Create: EventStorePlugin
    │       │   ├── Create: FaultLoggerPlugin
    │       │   └── ...
    │       │
    │       └── Plugin::Init()
    │           └── EventLoop::Init()
    │
    ├── DumpManagerCpuService::StartService()
    │   └── Register SA: DFX_SYS_HIVIEW_ABILITY_ID
    │
    ├── HiviewService::StartService()
    │   ├── HiviewServiceAbility::StartServiceAbility()
    │   │   └── Register SA: DFX_SYS_EVENT_SERVICE_ABILITY_ID
    │   │
    │   └── FaultloggerServiceOhos::StartService()
    │       └── Register SA: DFX_FAULT_LOGGER_ABILITY_ID
    │
    └── HiviewPlatform::StartLoop()
        └── EventLoop::Run()
            │
            └── PipelineWorker::Run()
                └── Plugin::OnEvent()
```

---

## N-API 调用链

### faultLogger.querySelfFaultLog()

```
JS: faultLogger.querySelfFaultLog(type, callback)
    │
    └── napi_faultlogger.cpp: QuerySelfFaultLog()
        │
        ├── GetHiViewRemoteService()
        │   └── hiview_remote_service.cpp: GetHiViewRemoteService()
        │       └── SAMgr::CheckSystemAbility(DFX_FAULT_LOGGER_ABILITY_ID)
        │           └── IPC: GetRemoteObject()
        │
        ├── napi_faultlogger.cpp: CreateCallback()
        │   └── Work::StartThread()
        │
        └── faultlogger_service_proxy.cpp: SendRequest()
            │
            └── IPC: SendRequest(QUERY_SELF_FAULTLOG)
                │
                └── FaultloggerServiceStub::OnRemoteRequest()
                    │
                    └── FaultloggerServiceStub::QuerySelfFaultLog()
                        │
                        └── FaultLoggerImpl::QuerySelfFaultLog()
                            │
                            ├── QueryDB()
                            │   └── EventStore::Query()
                            │
                            └── GetFaultLogInfo()
                                └── HiSysEventParser::Parse()
```

### logLibrary.list()

```
JS: logLibrary.list(logType)
    │
    └── napi_hiview_js.cpp: List()
        │
        ├── hiview_napi_util.cpp: IsSystemAppCall()
        │   └── AccessTokenKit::VerifyAccessToken()
        │
        ├── hiview_service_agent.cpp: List()
        │   └── IPC Proxy: HiviewServiceAbilityProxy::ListFiles()
        │       │
        │       └── IPC: SendRequest(LIST_FILES)
        │           │
        │           └── HiviewServiceAbilityStub::OnRemoteRequest()
        │               │
        │               └── HiviewServiceAbility::ListFiles()
        │                   │
        │                   ├── FileUtil::ListFiles()
        │                   └── CreateFileInfoArray()
        │
        └── hiview_napi_util.cpp: GenerateFileInfoResult()
            └── CreateJSArray()
```

---

## 事件流调用链

### HiSysEvent 上报 → 插件处理

```
HiSysEvent::Event()
    │
    └── HiSysEventSource::OnEvent()
        │
        └── HiSysEventSource::PushEvent()
            │
            └── EventLoop::PostEvent()
                │
                └── Pipeline::Dispatch()
                    │
                    └── EventDispatcher::Dispatch()
                        │
                        └── Plugin::OnEvent()
                            │
                            ├── EventValidatorPlugin::OnEvent()
                            │   └── EventValidator::Validate()
                            │
                            ├── EventStorePlugin::OnEvent()
                            │   └── EventStore::Save()
                            │       └── SQLite::Insert()
                            │
                            └── FaultLoggerPlugin::OnEvent()
                                └── FaultLogger::Report()
                                    └── FileWriter::Write()
```

---

## 性能采集调用链

### Trace 采集

```
TraceCollectorClient::StartTrace()
    │
    └── TraceCollectorImpl::Start()
        │
        ├── RegisterTraceListener()
        │   └── TraceListener::Register()
        │
        └── StartWorkerThread()
            │
            └── Worker::Run()
                │
                └── CollectLoop()
                    │
                    ├── ReadTraceBuffer()
                    │   └── kernel: /sys/kernel/debug/trace
                    │
                    └── ParseTraceData()
                        │
                        └── TraceParser::Parse()
                            │
                            └── TraceStorage::Store()
```

### CPU 采集

```
CpuCollectorClient::Start()
    │
    └── CpuCollectorImpl::Start()
        │
        ├── ReadProcStat()
        │   └── kernel: /proc/stat
        │
        ├── ReadProcPidStat()
        │   └── kernel: /proc/<pid>/stat
        │
        └── CalculateCpuUsage()
            │
            └── CpuUsageCalculator::Calculate()
```

---

## 隐私控制调用链

### 事件隐私检查

```
Plugin::OnEvent()
    │
    └── PrivacyManager::IsAllowed()
        │
        ├── EventPrivacyLevel::Get()
        │   └── PrivacyConfig::GetLevel()
        │
        └── BundlePrivacyChecker::IsAllowed()
            │
            ├── BundleNameChecker::Check()
            │   └── IsBundleNameInList()
            │
            └── PreInstallChecker::Check()
                └── IsPreInstallApp()
```

---

## 文件操作调用链

### 日志文件复制

```
logLibrary.copy(logType, logName, destDir)
    │
    └── napi_hiview_js.cpp: Copy()
        │
        ├── hiview_napi_util.cpp: IsSystemAppCall()
        │   └── AccessTokenKit::VerifyAccessToken()
        │
        ├── hiview_service_agent.cpp: Copy()
        │   └── IPC Proxy: HiviewServiceAbilityProxy::CopyFile()
        │       │
        │       └── IPC: SendRequest(COPY_FILE)
        │           │
        │           └── HiviewServiceAbilityStub::OnRemoteRequest()
        │               │
        │               └── HiviewServiceAbility::CopyFile()
        │                   │
        │                   ├── HiviewServiceAbility::IsSafePath()
        │                   │   ├── FileUtil::PathToRealPath()
        │                   │   └── IsPathInBaseDir()
        │                   │
        │                   ├── FileUtil::CopyFile()
        │                   │   ├── open(src)
        │                   │   ├── open(dst)
        │                   │   ├── read(fd_src, buf)
        │                   │   └── write(fd_dst, buf)
        │                   │
        │                   └── UpdateFileInfo()
```

---

## IPC 调用链详解

### System Ability 注册

```
SystemAbility::OnStart()
    │
    └── HiviewServiceAbility::OnStart()
        │
        └── HiviewServiceAbility::StartServiceAbility()
            │
            └── SAMgr::AddSystemAbility()
                │
                └── SystemAbilityManager::AddSA()
                    │
                    └── PublishServiceAbility()
                        │
                        └── Binder::Register()
```

### System Ability 获取

```
Client::GetService()
    │
    └── SAMgr::GetSystemAbility()
        │
        └── SystemAbilityManager::GetSA()
            │
            └── Binder::GetRemoteObject()
                │
                └── Return Proxy
```

---

## 插件加载调用链

```
PluginBundle::LoadConfig()
    │
    └── Read: /system/etc/hiview/plugin_config
        │
        └── ParseConfig()
            │
            ├── ParsePluginList()
            └── ParsePluginConfig()
                │
                └── ValidateConfig()
                    │
                    └── VerifySignature()
                        │
                        └── LogSignTools::VerifyFileSign()
                            │
                            ├── ReadCertificate()
                            │   └── Read: /system/etc/hiview/cert.enc
                            │
                            ├── CalculateHash()
                            │   └── SHA256::Calc()
                            │
                            └── VerifySignature()
                                └── RSA::Verify()
```

---

## 关键路径总结

| 场景 | 调用链深度 | 关键节点 |
|------|-----------|----------|
| 服务启动 | 10+ | PluginBundle::LoadConfig → PluginFactory::Create → EventLoop::Run |
| API 调用 | 5-7 | N-API → Proxy → Stub → Impl → DB/FS |
| 事件处理 | 5-8 | HiSysEvent → EventSource → Pipeline → Plugin |
| 性能采集 | 6-10 | CollectorClient → CollectorImpl → Kernel Interface |
| 文件操作 | 6-8 | N-API → Service → SecurityCheck → FileUtil → FS |
