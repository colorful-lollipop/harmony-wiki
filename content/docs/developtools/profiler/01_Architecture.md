# 01_Architecture - 系统架构

## 1. 整体架构

### 1.1 架构概览

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         OpenHarmony Profiler                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     PC 端 (DevEco Studio 插件)                    │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │   │
│  │  │ UI 绘制   │ │ 设备管理  │ │ 数据分析  │ │ Session  │         │   │
│  │  │          │ │          │ │          │ │ 管理      │         │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│                        gRPC/IPC 通信                                     │
│                              ↓                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     设备端 (Profiler Service)                       │   │
│  │  ┌─────────────────┐  ┌─────────────────────────────────────┐   │   │
│  │  │ Profiler Service │──│ Plugin Manager (19+ 插件)            │   │   │
│  │  │     (SA)         │  │ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐│   │   │
│  │  └─────────────────┘  │ │CPU插件│ │内存插件│ │Ftrace │ │Hilog ││   │   │
│  │                      │ └──────┘ └──────┘ └──────┘ └──────┘│   │   │
│  │  ┌─────────────────┐  └─────────────────────────────────────┘   │   │
│  │  │ hiprofiler_cmd  │                                         │   │
│  │  └─────────────────┘                                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│                        共享内存 / 文件                                     │
│                              ↓                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      目标进程 (被分析应用)                          │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                        │   │
│  │  │ Native   │ │ JS/ArkVM │ │ System    │                        │   │
│  │  │ Hook     │ │ Profiler │ │ Services  │                        │   │
│  │  └──────────┘ └──────────┘ └──────────┘                        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 核心组件

| 组件 | 类型 | 职责 | 源码位置 |
|------|------|------|----------|
| **Profiler Service** | System Ability | 性能分析主服务，Session 管理 | `device/services/profiler_service/` |
| **Plugin Manager** | 插件框架 | 插件加载、生命周期管理、数据路由 | `device/plugins/api/` |
| **hiprofiler_cmd** | 命令行工具 | 离线数据抓取 | `device/cmds/` |
| **Hidebug** | N-API 模块 | JS/ArkVM 调试接口 | `hidebug/interfaces/js/kits/napi/` |
| **Native Daemon** | Native 守护进程 | Native 内存追踪 | `device/plugins/native_daemon/` |
| **Shared Memory** | IPC 服务 | 插件间数据共享 | `device/services/shared_memory/` |

> 证据: `bundle.json:79-87`, `README_zh.md:28-32`

## 2. 插件系统架构

### 2.1 插件接口定义

所有插件必须实现 `PluginModuleCallbacks` 回调函数表：

```c
// 文件: interfaces/kits/plugin_module_api.h:207-245
struct PluginModuleCallbacks {
    // 会话开始回调（必需）
    PluginSessionStartCallback onPluginSessionStart;
    
    // 数据上报回调（轮询插件必需）
    PluginReportResultCallback onPluginReportResult;
    
    // 会话停止回调（必需）
    PluginSessionStopCallback onPluginSessionStop;
    
    // 写接口注册回调（流式插件必需）
    RegisterWriterStructCallback onRegisterWriterStruct;
    
    // 基础数据上报
    PluginReportBasicDataCallback onReportBasicDataCallback;
    
    // 优化版数据上报
    PluginReportResultOptimizeCallback onPluginReportResultOptimize;
    
    // 状态查询
    PluginReportStateCallback onReportStateCallback;
};
```

### 2.2 插件注册结构

```c
// 文件: interfaces/kits/plugin_module_api.h:262-287
struct PluginModuleStruct {
    PluginModuleCallbacks* callbacks;    // 回调函数表
    char name[PLUGIN_MODULE_NAME_MAX + 1];      // 插件名 (最大127字符)
    char version[PLUGIN_MODULE_VERSION_MAX + 1]; // 版本 (最大7字符)
    uint32_t resultBufferSizeHint;      // 缓冲区大小提示
    bool isStandaloneFileData;          // 是否输出到独立文件
    char outFileName[PATH_MAX + 1];     // 输出文件路径
};

// 插件必须导出此全局变量
extern PluginModuleStruct g_pluginModule;
```

### 2.3 插件类型

| 类型 | 标识 | 数据流 | 示例 |
|------|------|--------|------|
| **轮询插件 (Polling)** | `onPluginReportResult != nullptr` | 框架定期调用回调获取数据 | cpu_plugin, memory_plugin |
| **流式插件 (Streaming)** | `onRegisterWriterStruct != nullptr` | 插件主动写入共享内存 | ftrace_plugin, hilog_plugin |
| **独立文件插件** | `isStandaloneFileData = true` | 直接输出到文件 | hiperf_plugin, hiebpf_plugin |

> 证据: `interfaces/kits/plugin_module_api.h:295-310`

### 2.4 插件列表

| 插件名 | 类型 | 功能 | 配置文件 |
|--------|------|------|----------|
| **cpu_plugin** | 轮询 | CPU 使用率采样 | `protos/types/plugins/cpu_data/cpu_plugin_config.proto` |
| **memory_plugin** | 轮询 | 内存信息采集 | `protos/types/plugins/memory_data/memory_plugin_config.proto` |
| **ftrace_plugin** | 流式 | 内核追踪 | `protos/types/plugins/ftrace_data/default/trace_plugin_config.proto` |
| **hilog_plugin** | 流式 | 日志采集 | `protos/types/plugins/hilog_data/hilog_plugin_config.proto` |
| **hiperf_plugin** | 独立文件 | CPU 性能分析 | `protos/types/plugins/hiperf_data/hiperf_plugin_config.proto` |
| **native_hook** | 轮询 | Native 内存追踪 | `protos/types/plugins/native_hook/native_hook_config.proto` |
| **gpu_plugin** | 轮询 | GPU 分析 | `protos/types/plugins/gpu_data/gpu_plugin_config.proto` |
| **diskio_plugin** | 轮询 | 磁盘 I/O | `protos/types/plugins/diskio_data/diskio_plugin_config.proto` |
| **network_plugin** | 轮询 | 网络统计 | `protos/types/plugins/network_data/network_plugin_config.proto` |
| **process_plugin** | 轮询 | 进程信息 | `protos/types/plugins/process_data/process_plugin_config.proto` |
| **bytrace_plugin** | 流式 | 轻量追踪 | `protos/types/plugins/bytrace_plugin/bytrace_plugin_config.proto` |
| **hiebpf_plugin** | 独立文件 | eBPF 分析 | `protos/types/plugins/hiebpf_data/hiebpf_plugin_config.proto` |
| **hidump_plugin** | 流式 | HiDump | - |
| **hisysevent_plugin** | 轮询 | 事件采集 | - |
| **stream_plugin** | 流式 | 数据流 | `protos/types/plugins/stream_data/stream_plugin_config.proto` |
| **sample_plugin** | 轮询 | 采样 | - |
| **ffrt_profiler** | 轮询 | FFRT 分析 | `protos/types/plugins/ffrt_profiler/ffrt_profiler_config.proto` |
| **network_profiler** | 轮询 | 网络性能 | `protos/types/plugins/network_profiler/network_profiler_config.proto` |
| **xpower_plugin** | 轮询 | 功耗分析 | `protos/types/plugins/xpower_data/xpower_plugin_config.proto` |

> 证据: `README_zh.md:56-77`, `plugin_module_api.h`

## 3. 数据流

### 3.1 轮询插件数据流

```
┌─────────────────────────────────────────────────────────────────────┐
│                      轮询插件数据流                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Profiler Service                                                   │
│       │                                                             │
│       ▼                                                             │
│  Plugin Manager ──PullResult()─────────────────────────────────→   │
│       │         │                                                   │
│       │         ▼                                                   │
│       │    PluginModule::ReportResult()                              │
│       │         │                                                   │
│       │         ▼                                                   │
│       │    插件填充 protobuf 数据到缓冲区                              │
│       │         │                                                   │
│       ▼         ▼                                                   │
│  BufferWriter ──Write()──→ 共享内存 (ShareMemoryBlock)                │
│       │                                                             │
│       ▼                                                             │
│  Flush() ──通知服务读取数据                                          │
│       │                                                             │
│       ▼                                                             │
│  Profiler Service ──FetchData()──→ gRPC ──→ PC 端                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 流式插件数据流

```
┌─────────────────────────────────────────────────────────────────────┐
│                      流式插件数据流                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Profiler Service                                                   │
│       │                                                             │
│       ▼                                                             │
│  Plugin Manager ──RegisterWriterStruct()──→ 插件                     │
│       │                                                             │
│       ▼                                                             │
│  插件创建独立线程 ──Loop()──→                                        │
│       │                                                             │
│       ├───writer->write(data, size) ──→ 共享内存                     │
│       │                                                             │
│       └───writer->flush() ───通知服务读取                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.3 WriterStruct 接口

```c
// 文件: interfaces/kits/plugin_module_api.h:165-190
struct WriterStruct {
    // 写数据接口
    WriteFuncPtr write = nullptr;      // long (*write)(WriterStruct*, const void*, size_t)
    
    // 刷新接口
    FlushFuncPtr flush = nullptr;      // bool (*flush)(WriterStruct*)
    
    // 优化报告接口
    StartReportFuncPtr startReport = nullptr;  // RandomWriteCtx* (*startReport)()
    FinishReportFuncPtr finishReport = nullptr; // void (*finishReport)(WriterStruct*, int32_t)
    
    // 序列化方式: true=protobuf, false=protoencoder
    bool isProtobufSerialize = true;
};
```

> 证据: `interfaces/kits/plugin_module_api.h:165-190`

## 4. IPC 通信机制

### 4.1 Binder IPC (Native Memory Profiler SA)

Profiler 使用 OpenHarmony Binder IPC 框架进行跨进程通信：

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Binder IPC 架构                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  客户端 (Native Daemon)                    服务端 (SA)                │
│       │                                         │                    │
│       │  SendRequest()                          │                    │
│       ├────────────────────────────────────────→│                    │
│       │                                         │                    │
│       │         IRemoteProxy ──IPCSkeleton────→│ Stub               │
│       │                                         │                    │
│       │  1. WriteInterfaceToken()              │                    │
│       │  2. WriteParcelable/Primitive          │                    │
│       │  3. SendRequest(code, data, reply)     │                    │
│       │                                         │                    │
│       │                                         ▼                    │
│       │                                 OnRemoteRequest()            │
│       │                                         │                    │
│       │  DeathRecipient                         │                    │
│       │    OnRemoteDied() ◄─────────────────────┘                    │
│       │                                         │                    │
└─────────────────────────────────────────────────────────────────────┘
```

#### SA 信息

| 属性 | 值 |
|------|-----|
| **SA ID** | `DFX_SYS_NATIVE_MEMORY_PROFILER_SERVICE_ABILITY_ID` |
| **接口描述符** | `OHOS.NativeMemory.INativeMemoryProfilerSa` |
| **接口文件** | `i_native_memory_profiler_sa.h` |

#### 接口方法

| 方法名 | Code | 功能 |
|--------|------|------|
| `START` | 0 | 启动追踪 |
| `STOP_HOOK_PID` | 1 | 按 PID 停止 |
| `STOP_HOOK_NAME` | 2 | 按名称停止 |
| `DUMP_DATA` | 3 | 导出数据 |
| `DUMP_SIMP_DATA` | 4 | 导出简化数据 |

> 证据: `device/plugins/native_daemon/native_memory_profiler_sa/`

### 4.2 Unix Socket IPC

插件通信使用 Unix Domain Socket：

| 组件 | 路径 | 用途 |
|------|------|------|
| **ServiceEntry** | `service_entry.h` | Unix Socket 服务端 |
| **ServiceBase** | `service_base.h` | 服务基类 |
| **UnixSocketServer** | `unix_socket_server.cpp` | 基于 epoll 的服务器 |
| **UnixSocketClient** | `unix_socket_client.cpp` | 客户端实现 |

#### 协议类型

| 类型 | 值 | 说明 |
|------|------|------|
| `PROTOCOL_TYPE_RAW` | 0x0 | 原始数据 |
| `PROTOCOL_TYPE_PROTOBUF` | 0x10000000 | Protobuf 序列化 |

> 证据: `device/services/ipc/`

## 5. Session 生命周期

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Session 状态机                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  INITIAL ──CreateSession()───────────────────────────────┐          │
│     │                                                     │          │
│     │ DestroySession()                                   │          │
│     ▼                                                     ▼          │
│  CREATED ◄──StartSession()──────┐                        │          │
│     │                          │                        │          │
│     │ StopSession()            │                        │          │
│     ▼                          │                        │          │
│  STARTED ◄─KeepSession()───────┘                        │          │
│     │                                                     │          │
│     │ DestroySession()                                    │          │
│     ▼                                                     │          │
│  DONE ◄──────────────────────────────────────────────────┘          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.1 Session 操作

| 操作 | 触发方 | 功能 |
|------|--------|------|
| `CreateSession` | Profiler Service | 创建会话，分配资源 |
| `StartSession` | Profiler Service | 启动所有插件 |
| `KeepSession` | Profiler Service | 维持心跳 |
| `StopSession` | Profiler Service | 停止所有插件 |
| `DestroySession` | Profiler Service | 销毁会话，释放资源 |

> 证据: `device/services/plugin_service/include/plugin_session.h`

## 6. 共享内存管理

### 6.1 ShareMemoryBlock

```c
// 文件: device/services/shared_memory/src/share_memory_block.cpp
class ShareMemoryBlock {
    // 创建内存块
    static ShareMemoryBlock* Create(const std::string& name, size_t size);
    
    // 写入数据
    int Write(const uint8_t* data, size_t size);
    
    // 读取数据
    int Read(uint8_t* data, size_t size);
    
    // 刷新（通知消费者）
    void Flush();
};
```

### 6.2 BufferWriter

```c
// 文件: device/plugins/api/src/buffer_writer.h
class BufferWriter {
    // 带时间戳写入
    int64_t Write(const uint8_t* data, size_t size);
    
    // 序列化支持
    int Serialize(MessageLite* proto);
};
```

## 7. 线程模型

### 7.1 线程角色

| 线程 | 数量 | 职责 | 插件 |
|------|------|------|------|
| **Profiler 主线程** | 1 | Session 管理、IPC 处理 | - |
| **插件轮询线程** | N | 周期性采集数据 | 轮询插件 |
| **流式采集线程** | N | 实时数据采集 | ftrace, hilog, native_daemon |
| **数据写入线程** | 1 | 共享内存写入 | - |
| **gRPC 传输线程** | N | 数据发送到 PC 端 | - |

### 7.2 FFRT 集成

部分插件使用 FFRT (Fast Future Runtime) 进行异步任务调度：

```cpp
// 外部依赖: ffrt:libffrt
ffrt::task_handle handle = ffrt::submit([]() {
    // 执行异步任务
});
```

> 证据: `device/plugins/api/BUILD.gn`

---

## 8. 相关跳转

| 主题 | 链接 |
|------|------|
| 插件系统详解 | [02_Plugin_System.md](./02_Plugin_System.md) |
| N-API 接口 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) |
| 内部 API | [04_Inner_API.md](./04_Inner_API.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*最后更新: 2026-02-06*
