# SmartPerf 项目概览

## 项目定位

SmartPerf 是专为 OpenHarmony 打造的性能功耗调优工具，通过 GUI 泳道图细粒度分析 CPU 调度、频点、线程时间片、内存及帧率等数据。

**代码仓库**: `developtools/smartperf_host`

**子系统**: `developtools`

**组件**: `smartperf_host`

## 核心能力

### Device 端能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 命令行采集 | SP_daemon 命令行工具，支持 CSV 导出 | `smartperf_device/device_command/` |
| UI 悬浮窗 | device_ui 可视化采集，实时监控 | `smartperf_device/device_ui/` |
| 多指标采集 | CPU/GPU/FPS/内存/温度/功耗/网络 | `smartperf_device/device_command/collector/` |
| Trace 录制 | 支持 ftrace、hiperf、hisysevent 录制 | `smartperf_device/device_command/collector/ByTrace.cpp` |

### Host 端能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| Trace 解析 | trace_streamer 将 trace 解析为 SQLite | `smartperf_host/trace_streamer/src/` |
| Web IDE | 可视化 trace 分析工具 | `smartperf_host/ide/src/` |
| WASM 支持 | trace_streamer 可编译为 WASM 运行 | `smartperf_host/trace_streamer/src/rpc/wasm_func.cpp` |
| HDC 通信 | 支持在线录制和设备数据拉取 | `smartperf_host/ide/src/hdc/` |

## 技术架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     SmartPerf 整体架构                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────┐     ┌──────────────────────────────┐ │
│  │    SmartPerf Host     │     │     SmartPerf Device        │ │
│  │  ┌─────────────────┐  │     │  ┌────────────────────────┐│ │
│  │  │      IDE        │  │     │  │   device_ui (HAP)      ││ │
│  │  │  TypeScript/Go  │  │     │  │   ArkTS/ETS             ││ │
│  │  └────────┬────────┘  │     │  └───────────┬────────────┘│ │
│  │           │ WASM       │     │              │ Socket      │ │
│  │           ▼            │     │              ▼             │ │
│  │  ┌─────────────────┐  │     │  ┌────────────────────────┐│ │
│  │  │ trace_streamer  │  │     │  │  SP_daemon (C++)       ││ │
│  │  │  C++ / WASM     │  │     │  │  - collector/          ││ │
│  │  │  - parser/      │  │     │  │  - cmds/              ││ │
│  │  │  - filter/      │  │     │  │  - services/ipc      ││ │
│  │  │  - table/       │  │     │  │  - utils/             ││ │
│  │  └─────────────────┘  │     │  └────────────────────────┘│ │
│  └──────────────────────┘     └──────────────────────────────┘ │
│                                                                  │
│         ┌─────────────────────────┐                            │
│         │   设备端系统服务依赖     │                            │
│         │ - ipc / samgr           │                            │
│         │ - graphic_2d / window   │                            │
│         │ - ability_base / hiview  │                            │
│         └─────────────────────────┘                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 目录结构

```
smartperf_host/
├── bundle.json                          # 组件配置
├── README.md / README_zh.md             # 项目说明
│
├── smartperf_device/                    # 设备端
│   ├── device_command/                  # SP_daemon 命令行工具
│   │   ├── include/                     # 公共头文件
│   │   ├── interface/                   # Inner Kit 对外接口
│   │   │   ├── GameServicePlugin.h    # 游戏服务插件接口
│   │   │   ├── GameEventCallback.h    # 游戏事件回调
│   │   │   └── GpuCounterCallback.h   # GPU 计数器回调
│   │   ├── cmds/                       # 命令处理
│   │   ├── collector/                  # 数据采集器（25+ 个）
│   │   ├── services/                   # 服务模块
│   │   │   ├── ipc/                   # Socket IPC
│   │   │   └── task_mgr/              # 任务管理
│   │   ├── scenarios/                  # 场景化采集
│   │   └── utils/                     # 工具模块
│   │
│   ├── device_ui/                       # UI 悬浮窗 HAP
│   │   ├── entry/src/main/ets/         # ArkTS 源码
│   │   │   ├── Application/           # 应用入口
│   │   │   ├── MainAbility/           # UIAbility
│   │   │   ├── common/                # 公共组件
│   │   │   │   ├── profiler/         # 采集逻辑
│   │   │   │   ├── ui/                # UI 组件
│   │   │   │   └── utils/            # 工具类
│   │   │   └── pages/                 # 页面
│   │   └── signature/                 # 签名文件
│   │
│   └── build/                           # 构建配置
│       └── config.gni                   # 设备端 GN 变量
│
└── smartperf_host/                      # Host 端
    ├── ide/                             # Web IDE 工具
    │   ├── src/
    │   │   ├── base-ui/               # 基础 UI 组件
    │   │   ├── trace/                 # Trace 分析核心
    │   │   ├── hdc/                   # HDC 通信
    │   │   └── ...
    │   └── server/                    # 后端服务
    │
    └── trace_streamer/                 # Trace 解析引擎
        ├── src/
        │   ├── trace_streamer/         # 核心解析器
        │   ├── parser/                 # 解析器模块
        │   │   ├── pbreader_parser/    # Protobuf 解析
        │   │   └── ptreader_parser/    # 文本解析
        │   ├── filter/                 # 数据过滤器
        │   ├── table/                  # 数据表定义
        │   ├── rpc/                    # RPC/WASM
        │   └── proto_reader/           # Protobuf 读取
        ├── sdk/                        # SDK 接口
        │   └── demo_sdk/
        └── build/                      # 构建配置
```

## 运行环境

### Device 端要求

| 要求 | 说明 |
|------|------|
| 系统版本 | OpenHarmony Standard |
| 依赖组件 | ipc, samgr, graphic_2d, window_manager, ability_base |
| 权限配置 | INTERNET, GET_INSTALLED_BUNDLE_LIST, SYSTEM_FLOAT_WINDOW 等 |

### Host 端要求

| 要求 | 说明 |
|------|------|
| 运行环境 | Node.js 18+, Go 1.20+ |
| 依赖工具 | HDC (Harmony Device Connector) |
| 浏览器 | Chrome 90+ (IDE Web) |

## 关键概念

### 采集器 (Collector)

SP_daemon 中的数据采集模块，每个采集器负责一类性能指标。

**内置采集器列表**：
- `CPU` — CPU 使用率、频率、调度信息
- `GPU` — GPU 负载、渲染帧率
- `FPS` — 帧率、掉帧统计
- `RAM` — 内存使用、PSS/VSS/USS
- `Power` — 功耗数据
- `Temperature` — 温度传感器
- `DDR` — 内存带宽
- `Network` — 网络流量
- `ByTrace` — ftrace 录制
- `Hiperf` — 性能采样
- `GameEvent` — 游戏性能事件
- `GpuCounter` — GPU 计数器

### Trace Streamer

Host 端的 trace 数据解析引擎，将多种格式的 trace 文件解析为 SQLite 数据库。

**支持的输入格式**：
- ftrace (文本格式)
- Hiperf (protobuf 格式)
- HiSysEvent (事件格式)
- XPower (功耗数据)
- Raw Trace (原始数据)

**核心处理流程**：
```
输入文件 → Parser → Filter → Table → SQLite DB → SQL Query
```

### WASM 接口

trace_streamer 可通过 Emscripten 编译为 WebAssembly，在浏览器中运行。

**主要导出函数**：
- `Initialize` — 初始化
- `TraceStreamerParseDataEx` — 解析数据
- `TraceStreamerSqlQueryEx` — SQL 查询
- `TraceStreamerSqlMetricsQuery` — Metrics 计算

## 相关文档

- [系统架构](01_Architecture.md)
- [WASM 接口](02_WASM_API.md)
- [Inner Kit API](03_InnerAPI.md)
- [GN 构建配置](04_GNBuild.md)
- [编译产物](05_BuildArtifacts.md)
