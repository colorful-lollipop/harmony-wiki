# 02_Plugin_System - 插件系统详解

## 1. 插件接口

### 1.1 核心头文件

| 文件 | 用途 |
|------|------|
| `interfaces/kits/plugin_module_api.h` | 插件接口定义 |

### 1.2 回调函数表

```c
// 文件: interfaces/kits/plugin_module_api.h:207-245
struct PluginModuleCallbacks {
    // 1. 会话开始回调（必需）
    PluginSessionStartCallback onPluginSessionStart;
    
    // 2. 数据上报回调（轮询插件必需）
    PluginReportResultCallback onPluginReportResult;
    
    // 3. 会话停止回调（必需）
    PluginSessionStopCallback onPluginSessionStop;
    
    // 4. 写接口注册回调（流式插件必需）
    RegisterWriterStructCallback onRegisterWriterStruct;
    
    // 5. 基础数据上报（可选）
    PluginReportBasicDataCallback onReportBasicDataCallback;
    
    // 6. 优化版数据上报（可选）
    PluginReportResultOptimizeCallback onPluginReportResultOptimize;
    
    // 7. 状态查询（可选）
    PluginReportStateCallback onReportStateCallback;
};
```

### 1.3 插件结构体

```c
// 文件: interfaces/kits/plugin_module_api.h:262-287
struct PluginModuleStruct {
    // 回调函数表
    PluginModuleCallbacks* callbacks;
    
    // 插件名称（最大 127 字符）
    char name[PLUGIN_MODULE_NAME_MAX + 1];
    
    // 插件版本（最大 7 字符）
    char version[PLUGIN_MODULE_VERSION_MAX + 1];
    
    // 缓冲区大小提示（轮询插件使用）
    uint32_t resultBufferSizeHint;
    
    // 是否输出到独立文件
    bool isStandaloneFileData;
    
    // 输出文件名
    char outFileName[PATH_MAX + 1];
};

// 插件必须导出的全局变量
extern PluginModuleStruct g_pluginModule;
```

### 1.4 写接口结构体

```c
// 文件: interfaces/kits/plugin_module_api.h:165-190
struct WriterStruct {
    // 写数据函数指针
    WriteFuncPtr write = nullptr;
    
    // 刷新函数指针
    FlushFuncPtr flush = nullptr;
    
    // 开始报告（优化模式）
    StartReportFuncPtr startReport = nullptr;
    
    // 结束报告（优化模式）
    FinishReportFuncPtr finishReport = nullptr;
    
    // 序列化方式: true=protobuf, false=protoencoder
    bool isProtobufSerialize = true;
};
```

---

## 2. 插件实现模板

### 2.1 最小插件实现

```c
// my_plugin.cpp
#include "plugin_module_api.h"

// 1. 实现回调函数
static int PluginSessionStart(const uint8_t* configData, uint32_t configSize)
{
    // 解析配置数据 (protobuf)
    MyPluginConfig config;
    config.ParseFromArray(configData, configSize);
    
    // 初始化插件资源
    // ...
    return 0;
}

static int PluginReportResult(uint8_t* bufferData, uint32_t bufferSize)
{
    // 采集数据
    MyPluginData data = CollectData();
    
    // 序列化到缓冲区
    if (!data.SerializeToArray(bufferData, bufferSize)) {
        return -1;
    }
    return data.ByteSizeLong();
}

static int PluginSessionStop()
{
    // 清理资源
    // ...
    return 0;
}

// 2. 定义回调表
static PluginModuleCallbacks g_callbacks = {
    .onPluginSessionStart = PluginSessionStart,
    .onPluginReportResult = PluginReportResult,
    .onPluginSessionStop = PluginSessionStop,
};

// 3. 导出插件结构体
PluginModuleStruct g_pluginModule = {
    .callbacks = &g_callbacks,
    .name = "my-plugin",
    .version = "1.0.0",
    .resultBufferSizeHint = 4096,
};
```

---

## 3. 插件类型详解

### 3.1 轮询插件 (Polling Plugin)

**特点**: 框架定期调用 `onPluginReportResult` 获取数据

**适用场景**: 数据采集频率较低、周期性统计

**示例**: cpu_plugin, memory_plugin, process_plugin

```c
// 轮询插件实现
class PollingPlugin {
    uint32_t sampleIntervalMs_;
    
    static int OnReportResult(uint8_t* buffer, uint32_t size) {
        // 采集当前数据
        auto data = CollectData();
        
        // 序列化
        return SerializeToBuffer(data, buffer, size);
    }
};

PluginModuleCallbacks callbacks = {
    .onPluginSessionStart = [](auto cfg, auto size) { /* 初始化 */ return 0; },
    .onPluginReportResult = PollingPlugin::OnReportResult,
    .onPluginSessionStop = [] { /* 清理 */ return 0; },
};
```

### 3.2 流式插件 (Streaming Plugin)

**特点**: 插件主动写入数据，实时性高

**适用场景**: 高频数据采集、实时追踪

**示例**: ftrace_plugin, hilog_plugin, native_daemon

```c
// 流式插件实现
class StreamingPlugin {
    WriterStruct* writer_ = nullptr;
    std::thread采集线程;
    
    static int OnRegisterWriter(const WriterStruct* writer) {
        auto self = static_cast<StreamingPlugin*>(PluginGetInstance());
        self->writer_ = const_cast<WriterStruct*>(writer);
        
        // 创建采集线程
        self->采集线程 = std::thread([self] {
            while (采集Running) {
                auto data = CollectData();
                
                // 写入数据
                self->writer_->write(self->writer_, data.data(), data.size());
                
                // 通知框架读取
                self->writer_->flush(self->writer_);
                
                std::this_thread::sleep_for(采集间隔);
            }
        });
        return 0;
    }
};

PluginModuleCallbacks callbacks = {
    .onPluginSessionStart = [](auto cfg, auto size) { /* 初始化 */ return 0; },
    .onRegisterWriterStruct = StreamingPlugin::OnRegisterWriter,
    .onPluginSessionStop = [](void) { /* 停止线程、清理 */ return 0; },
};
```

### 3.3 独立文件插件 (Standalone File Plugin)

**特点**: 直接输出到文件，不经过共享内存

**适用场景**: 大数据量、需要特定文件格式

**示例**: hiperf_plugin, hiebpf_plugin

```c
PluginModuleStruct g_pluginModule = {
    .callbacks = &callbacks,
    .name = "hiperf-plugin",
    .version = "1.0.0",
    .isStandaloneFileData = true,
    .outFileName = "/data/local/tmp/perf.data",
};
```

---

## 4. Protobuf 配置定义

### 4.1 配置.proto 示例

```protobuf
// my_plugin_config.proto
syntax = "proto3";

message MyPluginConfig {
    // 采样间隔 (毫秒)
    uint32 sample_interval = 1;
    
    // 是否启用详细模式
    bool verbose = 2;
    
    // 过滤条件
    string filter_pattern = 3;
    
    // 缓冲区大小
    uint32 buffer_size = 4;
}
```

### 4.2 数据.proto 示例

```protobuf
// my_plugin_data.proto
syntax = "proto3";

message MyPluginData {
    // 时间戳
    int64 timestamp = 1;
    
    // 数据点列表
    repeated DataPoint points = 2;
    
    message DataPoint {
        string name = 1;
        int64 value = 2;
    }
}
```

---

## 5. 插件生命周期

### 5.1 状态机

```
┌─────────────────────────────────────────────────────────────────────┐
│                     插件生命周期状态机                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  LOADED ──LoadPlugin()────────────────────────────────────────┐    │
│     │                                                           │    │
│     │ GetPluginInfo() ──验证插件信息                              │    │
│     │                                                           │    │
│     ▼                                                           │    │
│  CREATED ◄─CreatePluginSession()──────┐                        │    │
│     │                                 │                        │    │
│     │ DestroyPluginSession()          │                        │    │
│     ▼                                 │                        │    │
│  READY ◄─PreparePluginSession(cfg)───┘                        │    │
│     │                                                         │    │
│     ▼                                                         │    │
│  STARTED ◄─StartPluginSession()───┐                           │    │
│     │                             │                           │    │
│     │ StopPluginSession()        │                           │    │
│     ▼                             │                           │    │
│  STOPPED ◄─KeepPluginSession()───┘                           │    │
│     │                                                         │    │
│     │ DestroyPluginSession()                                  │    │
│     ▼                                                         │    │
│  DESTROYED ◄─────────────────────────────────────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 关键方法

| 方法 | 触发 | 插件回调 | 职责 |
|------|------|----------|------|
| `LoadPlugin` | 框架加载 | - | 加载 .so、解析符号 |
| `CreatePluginSession` | 用户请求 | - | 创建会话对象 |
| `PreparePluginSession` | 框架配置 | - | 分配缓冲区 |
| `StartPluginSession` | 用户启动 | `onPluginSessionStart` | 启动采集 |
| `KeepPluginSession` | 周期性 | - | 心跳维持 |
| `StopPluginSession` | 用户停止 | `onPluginSessionStop` | 停止采集 |
| `DestroyPluginSession` | 用户销毁 | - | 释放资源 |
| `UnloadPlugin` | 框架卸载 | - | 关闭 .so |

---

## 6. 数据上报机制

### 6.1 轮询模式数据流

```
┌─────────────────────────────────────────────────────────────────────┐
│                    轮询模式数据上报流程                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PluginManager                                                      │
│      │                                                              │
│      ▼                                                              │
│  PluginSession::PullResult() ────────────────────────────────────→ │
│      │                                                              │
│      ├───→ onPluginReportResult(buffer, size)                      │
│      │     │                                                       │
│      │     └── 插件填充 protobuf 数据                                │
│      │                                                            │
│      ▼                                                            │
│  BufferWriter::Write(data, size)                                    │
│      │                                                            │
│      ├───→ 时间戳添加                                               │
│      ├───→ 序列化编码                                               │
│      └───→ 写入共享内存                                             │
│                                                                     │
│      ▼                                                            │
│  ShareMemoryBlock::Flush() ── 通知服务读取                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.2 流式模式数据流

```
┌─────────────────────────────────────────────────────────────────────┐
│                    流式模式数据上报流程                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PluginManager ──RegisterWriterStruct(writer)────────────────────→ │
│      │                                                              │
│      └── 传递 WriterStruct 给插件                                     │
│                                                                     │
│  插件内部                                                            │
│      │                                                              │
│      ▼                                                              │
│  std::thread 采集线程 ──Loop()──→                                  │
│      │                                                              │
│      ├─── writer->write(data, size)                                │
│      │     │                                                       │
│      │     └── 写入共享内存                                         │
│      │                                                             │
│      ├─── writer->flush()                                          │
│      │     │                                                       │
│      │     └── 通知服务读取                                         │
│      │                                                             │
│      └─── sleep(interval)                                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. 相关跳转

| 主题 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*最后更新: 2026-02-06*
