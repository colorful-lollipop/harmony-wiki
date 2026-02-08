# 附录 A: 调用链图谱

## 1. 关键调用链入口

### 1.1 N-API 调用链

#### startProfiling 调用链

```
hidebug.startProfiling(fileName)
    │
    ▼ napi_hidebug.cpp:160 (StartProfiling)
    │
    ├─► 参数解析: GetNapiStringValue()
    ├─► 权限检查: VerifyAccessToken()
    │
    ▼ hidebug_native_interface_impl.cpp
    │
    ├─► ProfilerService.StartSession()
    │       │
    │       ▼ plugin_manager.cpp
    │       │
    │       ├─► PluginManager::CreatePluginSession()
    │       ├─► PluginManager::PreparePluginSession()
    │       │
    │       ▼ PluginManager::StartPluginSession()
    │       │
    │       └─► Plugin::Start() [CPU Plugin]
    │               │
    │               ▼ PluginModule::InvokeSessionStart()
    │                       │
    │                       ▼ onPluginSessionStart(config)
    │                               │
    │                               ▼ cpu_plugin_module.cpp
    │
    ▼ 返回 undefined
```

#### getCpuUsage 调用链

```
hidebug.getCpuUsage()
    │
    ▼ napi_hidebug.cpp:265 (GetCpuUsage)
    │
    ├─► /proc/stat 文件读取
    ├─► CPU 时间计算
    │       │
    │       ├─► idle = user + nice + system + iowait + irq + softirq + steal
    │       └─► total = idle + (所有 CPU 时间)
    │
    ├─► usage = (1 - idle/total) * 100
    │
    ▼ napi_create_double(env, usage, &result)
```

### 1.2 插件数据流调用链

#### 轮询插件数据流

```
Profiler Service
    │
    ▼ PluginManager::PullResult(sessionId)
            │
            ├─► PluginSession::PullResult()
            │       │
            │       ├─► PluginModule::GetData(buffer)
            │       │       │
            │       │       └─► onPluginReportResult(buffer, size)
            │       │               │
            │       │               └─► cpu_data_plugin.cpp
            │       │                       │
            │       │                       └─► Serialize(CpuData)
            │       │
            │       ▼ BufferWriter::Write()
            │               │
            │               ├─► timestamp 添加
            │               ├─► Serialize(ProfilerDataHeader)
            │               └─► ShareMemoryBlock::Write()
            │
            └─► DataRepeater::Repeat()

Profiler Data Repeater ──► gRPC Stream ──► PC 端
```

#### 流式插件数据流

```
Profiler Service
    │
    ▼ PluginManager::RegisterWriter(sessionId, pluginId)
            │
            └─► Plugin::SetWriter(writer)
                    │
                    └─► onRegisterWriterStruct(writer)
                            │
                            └─► ftrace_plugin.cpp
                                    │
                                    ▼ std::thread (采集线程)
                                            │
                                            ├─► Loop()
                                            │       │
                                            │       ├─► ReadKernelBuffer()
                                            │       ├─► writer->write(data, size)
                                            │       │       │
                                            │       │       └─► ShareMemoryBlock::Write()
                                            │       │
                                            │       └─► writer->flush()
                                            │               │
                                            │               └─► EventNotifier::Notify()
                                            │
                                            └─► sleep(interval)
```

### 1.3 IPC 调用链

#### Native Memory Profiler SA 调用链

```
Native Daemon Client
    │
    ▼ NativeMemoryProfilerSaClientManager::GetRemoteService()
            │
            ├─► SystemAbilityManager::GetSystemAbility()
            │       │
            │       └─► 返回 IRemoteObject
            │
            ▼ NativeMemoryProfilerSaProxy
                    │
                    ├─► WriteInterfaceToken()
                    │
                    ├─► WriteParcelable(config)
                    │
                    ▼ SendRequest(START, data, reply)
                            │
                            ▼ IPCSkeleton::Transact()
                                    │
                                    ▼ Binder Driver
                                            │
                                            ▼ NativeMemoryProfilerSaStub::OnRemoteRequest()
                                                    │
                                                    ▼ StubStart()
                                                            │
                                                            ▼ StartService(config)
```

---

## 2. 核心入口点

### 2.1 可执行文件入口

| 文件 | 入口函数 | 说明 |
|------|----------|------|
| `device/cmds/src/main.cpp` | `main()` | hiprofiler_cmd 入口 |
| `device/plugins/native_daemon/main.cpp` | `main()` | native_daemon 入口 |
| `device/services/profiler_service/src/main.cpp` | `main()` | SA 服务入口 |
| `hiebpf/src/hiebpf.cpp` | `main()` | hiebpf 工具入口 |

### 2.2 共享库入口

| 文件 | 入口函数 | 说明 |
|------|----------|------|
| `hidebug/interfaces/js/kits/napi/napi_hidebug.cpp` | `HiDebugRegisterModule()` | N-API 模块注册 |
| `device/plugins/api/src/plugin_module.cpp` | `PluginModule::Load()` | 插件加载 |
| `device/services/plugin_service/src/plugin_service_impl.cpp` | `PluginServiceImpl::Init()` | 服务初始化 |

### 2.3 N-API 入口

| 函数 | 行号 | 功能 |
|------|------|------|
| `StartProfiling` | 160 | 开始 CPU 分析 |
| `StopProfiling` | 223 | 停止 CPU 分析 |
| `DumpHeapData` | 241 | 导出堆快照 |
| `GetCpuUsage` | 265 | 获取 CPU 使用率 |
| `GetPss` | 239 | 获取 PSS 内存 |
| `GetServiceDump` | 303 | 获取服务转储 |

---

## 3. 数据流向图

### 3.1 采集-存储-传输

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   目标进程   │───►│   插件采集   │───►│   共享内存   │───►│  gRPC 传输   │
│             │    │             │    │             │    │             │
│ Native Hook │    │ CPU/Memory/  │    │ BufferWriter│    │ DataRepeater│
│ JS/ArkVM    │    │ Ftrace/HiLog│    │ ShareMemory │    │ Protobuf    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                          │                                    │
                          ▼                                    ▼
                   ┌─────────────┐                     ┌─────────────┐
                   │  配置文件   │                     │   PC 端     │
                   │  (.proto)   │                     │ DevEco      │
                   └─────────────┘                     └─────────────┘
```

### 3.2 控制流向

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   PC 端     │───►│ hiprofiler_ │───►│ Profiler    │───►│   插件       │
│ DevEco     │    │   cmd       │    │ Service     │    │             │
│ UI/CLI     │    │             │    │ (SA)        │    │ 状态机控制   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                          │                                    │
                          ▼                                    ▼
                   ┌─────────────┐                     ┌─────────────┐
                   │  配置解析   │                     │   共享内存   │
                   │ ConfigMgr   │                     │ Session 数据 │
                   └─────────────┘                     └─────────────┘
```

---

## 4. 关键文件索引

| 主题 | 关键文件 |
|------|----------|
| N-API 注册 | `napi_hidebug.cpp:1086-1146` |
| 插件加载 | `plugin_module.cpp:PluginModule::Load()` |
| Session 管理 | `plugin_session.cpp` |
| 数据写入 | `buffer_writer.cpp` |
| 共享内存 | `share_memory_block.cpp` |
| IPC 通信 | `native_memory_profiler_sa_service.cpp` |
| 配置解析 | `profiler_config_manager.cpp` |

---

*最后更新: 2026-02-06*
