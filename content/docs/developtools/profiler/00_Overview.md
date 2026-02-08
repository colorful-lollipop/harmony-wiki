# 00_Overview - 项目概览

## 1. 项目定位

**OpenHarmony Profiler** (`@ohos/hiprofiler`) 是 OpenHarmony 系统的性能调优组件，属于 `developtools` 子系统。

| 属性 | 值 |
|------|-----|
| **项目名称** | @ohos/hiprofiler |
| **版本** | 3.0.9 |
| **子系统** | developtools |
| **系统能力** | SystemCapability.HiviewDFX.HiProfiler.HiDebug |
| **License** | Apache License 2.0 |
| **ROM** | 188KB |
| **RAM** | 2000KB |

### 核心定位

提供一套完整的性能调优平台，用于分析内存、性能等问题。整体分为：

- **PC 端**: 作为 DevEco Studio 插件发布，包含 UI 绘制、设备管理、插件管理、数据分析等模块
- **设备端**: 提供命令行工具、服务进程、插件集合，用于设备上的数据采集

> 证据: `bundle.json:3-4`

## 2. 核心能力

### 2.1 CPU 性能分析

| API | 功能 | 同步/异步 |
|-----|------|----------|
| `startProfiling()` | 开始 CPU 采样分析 | 同步 |
| `stopProfiling()` | 停止分析并生成 trace | 同步 |
| `startJsCpuProfiling()` | 开始 JS CPU 采样 | 同步 |
| `stopJsCpuProfiling()` | 停止 JS CPU 采样 | 同步 |

### 2.2 内存分析

| API | 功能 | 同步/异步 |
|-----|------|----------|
| `getPss()` | 获取进程 PSS 内存 | 同步 |
| `getNativeHeapSize()` | 获取原生堆大小 | 同步 |
| `getNativeHeapAllocatedSize()` | 获取已分配堆内存 | 同步 |
| `dumpHeapData()` | 导出堆快照 | 同步 |
| `getAppNativeMemInfo()` | 获取应用原生内存信息 | 同步 |

### 2.3 运行时信息

| API | 功能 | 同步/异步 |
|-----|------|----------|
| `getCpuUsage()` | 获取 CPU 使用率 | 同步 |
| `getSystemCpuUsage()` | 获取系统 CPU 使用率 | 同步 |
| `getAppThreadCpuUsage()` | 获取线程 CPU 使用率 | 同步 |
| `getSystemMemInfo()` | 获取系统内存信息 | 同步 |
| `getAppMemoryLimit()` | 获取应用内存限制 | 同步 |

### 2.4 Trace 追踪

| API | 功能 | 同步/异步 |
|-----|------|----------|
| `startAppTraceCapture()` | 开始应用级 trace 采集 | 同步 |
| `stopAppTraceCapture()` | 停止 trace 采集 | 同步 |
| `getVMRuntimeStats()` | 获取 VM 运行时统计 | 同步 |

### 2.5 高级功能

| API | 功能 | 同步/异步 |
|-----|------|----------|
| `getGraphicsMemory()` | 获取 GPU 内存 | 异步 (Promise) |
| `enableGwpAsanGrayscale()` | 启用 GWP-ASan 灰度 | 同步 |
| `setJsRawHeapTrimLevel()` | 设置 JS 堆整理级别 | 同步 |

> 证据: `napi_hidebug.cpp:1087-1126`

## 3. 运行环境

### 3.1 系统要求

- **系统类型**: standard (标准系统)
- **OpenHarmony 版本**: 3.0+

### 3.2 依赖组件

#### 核心依赖

| 组件 | 用途 |
|------|------|
| `ability_runtime` | 能力运行时 |
| `ability_base` | 基础能力 |
| `ipc` | IPC 通信 |
| `samgr` | 系统能力管理 |
| `safwk` | 系统能力框架 |
| `hilog` | 日志系统 |
| `hisysevent` | HiSysEvent 事件 |

#### 数据处理依赖

| 组件 | 用途 |
|------|------|
| `protobuf` | 序列化 |
| `grpc` | 远程调用 |
| `abseil-cpp` | C++ 基础库 |

#### 图形与媒体

| 组件 | 用途 |
|------|------|
| `graphic_2d` | 2D 图形 |
| `window_manager` | 窗口管理 |
| `image_framework` | 图片处理 |
| `ffrt` | 快任务调度 |

#### 其他依赖

| 组件 | 用途 |
|------|------|
| `access_token` | 访问令牌管理 |
| `bundle_framework` | 包框架 |
| `libbpf` | BPF 支持 |
| `openssl` | 加密 |

> 证据: `bundle.json:29-68`

## 4. 目录结构

```
profiler/
├── device/                       # 设备侧代码
│   ├── base/                     # 基础功能 (配置、日志)
│   ├── cmds/                     # 命令行工具 (hiprofiler_cmd)
│   ├── plugins/                  # 插件集合 (19+ 插件)
│   │   ├── api/                 # 插件接口定义
│   │   ├── cpu_plugin/          # CPU 分析
│   │   ├── memory_plugin/        # 内存分析
│   │   ├── ftrace_plugin/       # 内核追踪
│   │   ├── hilog_plugin/        # 日志采集
│   │   ├── hiperf_plugin/       # CPU 性能分析
│   │   ├── native_hook/          # Native 堆追踪
│   │   ├── gpu_plugin/          # GPU 分析
│   │   ├── diskio_plugin/       # 磁盘 I/O
│   │   ├── network_plugin/      # 网络分析
│   │   ├── process_plugin/      # 进程信息
│   │   ├── bytrace_plugin/      # 轻量追踪
│   │   ├── hiebpf_plugin/       # eBPF 分析
│   │   ├── hidump_plugin/       # HiDump
│   │   ├── hisysevent_plugin/   # 事件采集
│   │   ├── stream_plugin/       # 数据流
│   │   ├── sample_plugin/       # 采样
│   │   ├── native_daemon/       # Native 守护进程
│   │   ├── ffrt_profiler/       # FFRT 分析
│   │   └── network_profiler/    # 网络性能
│   └── services/                # 系统服务
│       ├── profiler_service/    # 性能分析服务 (SA)
│       ├── plugin_service/       # 插件管理服务
│       ├── ipc/                  # IPC 服务
│       └── shared_memory/        # 共享内存管理
├── host/                        # 主机侧 (SmartPerf UI)
├── hidebug/                     # 调试工具 (N-API)
│   ├── interfaces/
│   │   ├── js/kits/napi/       # N-API 实现
│   │   ├── ets/ani/            # ETS 接口
│   │   ├── cj/                 # CJ 接口
│   │   └── native/kits/         # Native 接口
│   └── frameworks/
├── protos/                      # Protobuf 定义
├── proto_encoder/               # 编码器
├── interfaces/                 # 接口定义
├── timestamps/                 # 时间戳工具
├── tools/                      # 工具集
└── build/                      # 构建配置
```

> 证据: `README_zh.md:36-71`

## 5. 关键概念

### 5.1 插件架构

Profiler 使用**插件化架构**，每个插件负责特定类型的数据采集：

- **轮询插件 (Polling)**: 插件管理框架定期调用 `onPluginReportResult` 获取数据
- **流式插件 (Streaming)**: 插件主动通过 `WriterStruct` 写入数据

### 5.2 Session 管理

性能采集以 **Session** 为单位管理：

1. `CreateSession` - 创建会话
2. `StartSession` - 启动采集
3. `KeepSession` - 维持心跳
4. `StopSession` - 停止采集
5. `DestroySession` - 销毁会话

### 5.3 数据流

```
用户配置 → Profiler Service → Plugin Manager → 插件 → 共享内存 → 文件/PC端
```

---

## 6. 相关跳转

| 主题 | 链接 |
|------|------|
| 架构详解 | [01_Architecture.md](./01_Architecture.md) |
| 插件系统 | [02_Plugin_System.md](./02_Plugin_System.md) |
| API 参考 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*最后更新: 2026-02-06*
