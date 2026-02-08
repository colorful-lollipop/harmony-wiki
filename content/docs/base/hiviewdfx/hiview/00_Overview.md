# 项目概览

## 项目定位

Hiview 是 OpenHarmony DFX 子系统的核心组件，提供**跨平台终端设备维测服务集**。

> 证据: `bundle.json:3` - "Hiview is the module of OpenHarmony that provides toolkits for device maintenance across different platforms."

### 核心能力

| 能力分类 | 具体功能 |
|----------|----------|
| 故障日志 | C++/JS 崩溃日志采集、故障日志查询、异常上报 |
| 性能监控 | CPU/GPU/内存/IO 采集、Trace 管理、性能分析 |
| 事件管理 | 系统事件源、事件存储、事件分发 |
| 隐私控制 | 隐私级别管理、敏感数据保护 |
| 日志库 | 日志文件列表、复制、移动、删除操作 |

### 系统能力声明

| Syscap | 说明 |
|--------|------|
| `SystemCapability.HiviewDFX.Hiview.FaultLogger` | 故障日志能力 |
| `SystemCapability.HiviewDFX.Hiview.LogLibrary` | 日志库能力 |

> 证据: `bundle.json:15-18`

## 项目边界

```
/base/hiviewdfx/hiview/
├── adapter/          # 平台适配层（IPC、SA、OS服务适配）
├── base/             # 基础定义（插件、事件、工具类）
├── core/             # 核心模块（插件管理、配置）
├── service/          # 平台服务
├── plugins/          # 插件模块（17个独立功能）
├── framework/        # 框架实现（统一采集器）
├── utility/          # 工具模块（解析、日志分析）
└── interfaces/       # 对外接口（N-API、Inner API）
```

### 不包含

- **测试代码**: `test/`, `*_test.*`, `fuzztest/`
- **其他 DFX 组件**: hilog、hiappevent、hisysevent（在独立仓库）
- **设备侧 lite 版本**: hiviewdfx_hiview_lite（在独立仓库）

> 证据: `README.md` 目录结构说明

## 运行环境

| 环境 | 要求 |
|------|------|
| 系统类型 | standard（标准设备） |
| 编程语言 | C++14 |
| libc 版本 | C++14 及以上 |
| 依赖 | OpenHarmony 标准系统能力（IPC、SAMgr、Ability 等） |

> 证据: `README_zh.md:62` - "使用C++14的特性，依赖C++14及以上的libc实现。"

### 启动方式

Hiview 服务随设备启动自动启动，按配置文件加载插件。

> 证据: `README_zh.md:70`

## 关键概念

### 1. 事件驱动架构

Hiview 维测服务由事件驱动，核心为分布在系统各处的 **HiSysEvent 桩点**。

```
HiSysEvent API → 格式化事件上报 → HiSysEventSource → 插件处理 → 日志存储
```

### 2. 插件机制

Hiview 采用**插件化架构**，支持：
- 插件配置管理
- 插件动态加载
- 插件生命周期管理
- 插件间事件分发

### 3. 统一采集器 (Unified Collector)

统一采集框架，提供多种性能指标的采集能力：
- CPU/GPU/Memory/IO
- Trace/Hilog/Thermal/WM
- Network/eBPF/Perf

### 4. 隐私分级

事件按隐私级别分类处理：
- `PUBLIC_PRIVACY = 4` - 公开
- `USER_PANIC_WARNING_PRIVACY = 2` - 用户警告
- `LONGPRESS_PRIVACY = 1` - 长按隐私

## 模块职责

| 模块 | 职责 | 证据文件 |
|------|------|----------|
| `adapter` | 操作系统适配层，适配系统服务接口 | `README_zh.md:22` |
| `base` | 插件定义、检测器定义、工具类 | `README_zh.md:24` |
| `core` | 插件配置、插件管理、事件源 | `README_zh.md:26` |
| `service` | 平台服务（运行信息导出） | `README_zh.md:28` |
| `plugins` | 独立业务模块（故障、性能、事件等） | `README_zh.md:30` |

## 依赖关系

### 系统依赖

| 组件 | 用途 |
|------|------|
| `ability_base/ability_runtime` | Ability 框架 |
| `access_token` | 权限管理 |
| `bundle_framework` | 包管理 |
| `ipc` | 进程间通信 |
| `safgr/samgr` | 系统能力管理 |
| `hilog/hisysevent` | 日志服务 |
| `napi` | JavaScript 接口 |
| `init` | 系统启动 |

### 内部依赖

```
hiview (executable)
├── hiviewbase (libhiviewbase.so)
│   ├── logger
│   ├── event_publish
│   ├── event_report
│   ├── event_store
│   └── utility
├── hiview_core (source_set)
└── hiview_service (source_set)
```

> 证据: `BUILD.gn` 和 `hiview.gni` 构建依赖分析
