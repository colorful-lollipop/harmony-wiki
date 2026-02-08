# HiCollie 目录结构与模块职责

> 目录结构、模块职责划分、关键文件位置

---

## 目的与适用范围

### 文档目的
本文档帮助开发者快速定位代码位置，理解各模块职责，进行代码导航。

### 适用场景
- 🔍 **代码定位** - 查找特定功能的实现位置
- 📁 **模块理解** - 理解各目录的职责边界
- 🔧 **开发扩展** - 在正确位置添加新功能

---

## 完整目录树

```
/Volumes/lexar/code/d/work/oh/base/hiviewdfx/hicollie/
├── bundle.json                    # 组件配置文件
├── hicollie.gni                   # GN构建变量定义
├── hisysevent.yaml                # 系统事件定义
├── LICENSE                        # 许可证
├── README.md / README_zh.md       # 项目文档
│
├── frameworks/                    # 框架实现层
│   ├── app/                       # App层Watchdog实现
│   │   ├── BUILD.gn
│   │   ├── include/
│   │   │   ├── app_watchdog_inner.h
│   │   │   └── app_watchdog_utils.h
│   │   └── src/
│   │       ├── app_watchdog.cpp
│   │       ├── app_watchdog_inner.cpp
│   │       └── app_watchdog_utils.cpp
│   │
│   └── native/                    # Native层核心实现
│       ├── BUILD.gn
│       ├── handler_checker.h          # Handler状态检查器
│       ├── handler_checker.cpp
│       ├── ipc_full.cpp             # IPC满检测实现
│       ├── watchdog.cpp             # Watchdog单例实现
│       ├── watchdog_inner.h          # 核心调度器声明
│       ├── watchdog_inner.cpp        # 核心调度器实现
│       ├── watchdog_inner_data.h     # 内部数据结构
│       ├── watchdog_task.h           # 监控任务定义
│       ├── watchdog_task.cpp         # 监控任务实现
│       ├── xcollie.cpp             # XCollie超时检测实现
│       ├── xcollie_ffrt_task.h     # FFRT异步任务声明
│       ├── xcollie_ffrt_task.cpp   # FFRT异步任务实现
│       ├── xcollie_utils.h         # 工具函数声明
│       ├── xcollie_utils.cpp       # 工具函数实现
│       └── thread_sampler/          # 线程采样器模块
│           ├── BUILD.gn
│           ├── include/
│           │   ├── thread_sampler.h
│           │   ├── thread_sampler_api.h
│           │   └── thread_sampler_utils.h
│           ├── thread_sampler.cpp
│           ├── thread_sampler_api.cpp
│           └── thread_sampler_utils.cpp
│
└── interfaces/                    # 接口层
    ├── app/                       # App层接口
    │   ├── BUILD.gn
    │   └── include/
    │       └── app_watchdog.h
    │
    ├── native/                    # Native内聚接口
    │   └── innerkits/
    │       ├── BUILD.gn
    │       ├── libhicollie.map          # 符号导出控制
    │       └── include/
    │           └── xcollie/
    │               ├── watchdog.h           # Watchdog C++ API
    │               ├── xcollie.h           # XCollie C++ API
    │               ├── xcollie_define.h     # 常量定义
    │               ├── ipc_full.h          # IPC监控 C++ API
    │               └── thread_kill_reason.h  # 线程杀死原因
    │
    ├── ndk/                       # NDK接口（C接口）
    │   ├── BUILD.gn
    │   ├── libohhicollie.map         # NDK符号导出
    │   ├── hicollie.cpp             # NDK C API实现
    │   └── include/
    │       └── hicollie.h            # NDK C API头文件
    │
    └── rust/                      # Rust接口
        ├── BUILD.gn
        └── src/
            └── lib.rs
```

---

## 模块职责详解

### 1. `/interfaces/` - 公共接口层

**职责**: 对外暴露 API，供应用和系统服务调用

#### 1.1 `/interfaces/ndk/` - NDK C API
**证据**: `interfaces/ndk/include/hicollie.h:42-263`

| 文件 | 职责 | 关键内容 |
|-----|------|---------|
| `include/hicollie.h` | NDK C API 头文件 | OH_HiCollie_* 函数声明、错误码定义 |
| `hicollie.cpp` | NDK C API 实现 | 对 C++ 内部实现的封装 |
| `BUILD.gn` | NDK 构建配置 | 生成 `libohhicollie.so` |
| `libohhicollie.map` | 符号导出控制 | 限制导出符号 |

**导出 API**:
- `OH_HiCollie_Init_StuckDetection`
- `OH_HiCollie_Init_JankDetection`
- `OH_HiCollie_Report`
- `OH_HiCollie_SetTimer`
- `OH_HiCollie_CancelTimer`

#### 1.2 `/interfaces/native/innerkits/` - Native C++ API
**证据**: `interfaces/native/innerkits/include/xcollie/`

| 头文件 | 职责 | 关键类/函数 |
|---------|------|------------|
| `watchdog.h` | Watchdog 接口 | `Watchdog::AddThread()`, `RunPeriodicalTask()` |
| `xcollie.h` | XCollie 超时接口 | `XCollie::SetTimer()`, `CancelTimer()` |
| `ipc_full.h` | IPC 监控接口 | `IpcFull::AddIpcFull()`, `AsyncBinderSpaceFull()` |
| `xcollie_define.h` | 常量定义 | `XCOLLIE_FLAG_*`, `DEFAULT_IPC_FULL_INTERVAL` |
| `thread_kill_reason.h` | 线程杀死原因 | 枚举定义 |

#### 1.3 `/interfaces/app/` - App 层接口
**证据**: `interfaces/app/include/app_watchdog.h:23-108`

| 文件 | 职责 | 关键类 |
|-----|------|-------|
| `include/app_watchdog.h` | App Watchdog 单例 | `AppWatchdog::GetReservedTimeForLogging()`, `SetBundleInfo()` |

---

### 2. `/frameworks/native/` - Native 核心实现

**职责**: Watchdog 和 XCollie 的核心实现，包括线程监控、超时检测、IPC 监控等

#### 2.1 核心实现文件

| 文件 | 职责 | 关键类/函数 |
|-----|------|------------|
| `watchdog.cpp` | Watchdog 单例实现 | `Watchdog` 构造/析构，委托给 `WatchdogInner` |
| `watchdog_inner.h` | 核心调度器声明 | `WatchdogInner` 单例，任务队列管理 |
| `watchdog_inner.cpp` | 核心调度器实现 | 任务调度、线程创建、IPC 处理 |
| `watchdog_inner_data.h` | 内部数据结构 | TimeContent, StackContent, TraceContent |
| `watchdog_task.h` | 监控任务定义 | `WatchdogTask` 类，任务封装 |
| `watchdog_task.cpp` | 监控任务实现 | 任务执行、Binder 空间检测 |
| `handler_checker.h` | Handler 检查器声明 | `HandlerChecker` 类，Handler 状态检测 |
| `handler_checker.cpp` | Handler 检查器实现 | `ScheduleCheck()`, `GetCheckState()` |
| `xcollie.cpp` | XCollie 超时检测 | `SetTimer()`, `CancelTimer()` |
| `xcollie_ffrt_task.h` | FFRT 异步任务声明 | `XCollieFfrtTask` 类 |
| `xcollie_ffrt_task.cpp` | FFRT 异步任务实现 | Trace 采集异步任务 |
| `xcollie_utils.h` | 工具函数声明 | 进程操作、字符串处理 |
| `xcollie_utils.cpp` | 工具函数实现 | `KillProcessByPid()`, `GetLimitedSizeName()` |
| `ipc_full.cpp` | IPC 满检测实现 | `IpcFull::AddIpcFull()` |

**证据**: `watchdog_inner.h:38-96` - WatchdogInner 类声明包含所有核心方法

#### 2.2 Thread Sampler 模块
**证据**: `frameworks/native/thread_sampler/include/thread_sampler.h`

| 文件 | 职责 | 关键类/函数 |
|-----|------|------------|
| `thread_sampler.h` | 线程采样器声明 | `ThreadSampler` 单例，采样控制 |
| `thread_sampler.cpp` | 线程采样器实现 | 信号处理、堆栈展开 |
| `thread_sampler_api.h` | 对外 API 声明 | `Init()`, `Start()`, `Stop()` |
| `thread_sampler_api.cpp` | 对外 API 实现 | NDK 接口封装 |
| `thread_sampler_utils.h` | 工具函数声明 | 信号处理工具 |
| `thread_sampler_utils.cpp` | 工具函数实现 | 信号处理实现 |

**关键机制**:
- 使用 SIGURG 信号触发采样
- 双缓冲设计（读索引、写索引）
- 集成 libunwind 进行栈回溯

---

### 3. `/frameworks/app/` - App 层实现

**职责**: 应用层 Watchdog 专用实现，管理应用状态

| 文件 | 职责 | 关键类/函数 |
|-----|------|------------|
| `app_watchdog.cpp` | AppWatchdog 单例实现 | 对外接口封装 |
| `app_watchdog_inner.h` | 内部实现声明 | `AppWatchdogInner` 类 |
| `app_watchdog_inner.cpp` | 内部实现 | 应用状态管理 |
| `app_watchdog_utils.h` | 工具函数声明 | Bundle 信息处理 |
| `app_watchdog_utils.cpp` | 工具函数实现 | Bundle 名称/版本获取 |

**证据**: `app_watchdog_inner.h` - AppWatchdogInternal 类定义

---

## 按功能查找代码

### 需要查找功能？使用下表快速定位

| 功能需求 | 目标文件/目录 | 理由 |
|---------|--------------|------|
| **使用 NDK C API** | `interfaces/ndk/include/hicollie.h` | NDK 接口头文件 |
| **使用 C++ Watchdog** | `interfaces/native/innerkits/include/xcollie/watchdog.h` | C++ 接口头文件 |
| **使用 XCollie 定时器** | `interfaces/native/innerkits/include/xcollie/xcollie.h` | XCollie 接口头文件 |
| **使用 IPC 监控** | `interfaces/native/innerkits/include/xcollie/ipc_full.h` | IPC 监控接口头文件 |
| **修改超时检测逻辑** | `frameworks/native/xcollie.cpp` | XCollie 实现 |
| **修改任务调度逻辑** | `frameworks/native/watchdog_inner.cpp` | 核心调度器实现 |
| **修改 Handler 检查** | `frameworks/native/handler_checker.cpp` | Handler 检查实现 |
| **修改线程采样** | `frameworks/native/thread_sampler/thread_sampler.cpp` | 采样器实现 |
| **查看常量定义** | `interfaces/native/innerkits/include/xcollie/xcollie_define.h` | 常量定义 |
| **查看事件定义** | `hisysevent.yaml` | 系统事件定义 |

---

## 构建配置文件

### BUILD.gn 文件清单

| 路径 | 作用 |
|------|------|
| `/frameworks/native/BUILD.gn` | Native 核心模块构建 |
| `/frameworks/app/BUILD.gn` | App 层模块构建 |
| `/frameworks/native/thread_sampler/BUILD.gn` | Thread Sampler 构建配置 |
| `/interfaces/native/innerkits/BUILD.gn` | Native Innerkits 接口构建 |
| `/interfaces/app/BUILD.gn` | App 接口构建 |
| `/interfaces/ndk/BUILD.gn` | NDK 接口构建 |
| `/interfaces/rust/BUILD.gn` | Rust 接口构建 |

### 配置文件

| 文件 | 内容 | 用途 |
|-----|------|------|
| `bundle.json` | 组件元数据 | 定义子系统、依赖、构建目标 |
| `hicollie.gni` | GN 变量 | 功能开关、路径定义 |
| `hisysevent.yaml` | 系统事件 | 定义 SERVICE_TIMEOUT、IPC_FULL 等事件 |

---

## 关键结论

### 目录组织特点
1. **接口与实现分离** - `interfaces/` 提供公共 API，`frameworks/` 包含实现
2. **多语言支持** - 支持 C (NDK)、C++ (Native)、Rust
3. **模块化设计** - Thread Sampler 作为独立模块可动态加载
4. **App 与 Native 分离** - `frameworks/app/` 专门处理应用层逻辑

### 代码导航建议
- **快速定位 API**: 从 `interfaces/` 找对应头文件
- **理解实现**: 从 `interfaces/` 头文件跳转到 `frameworks/` 实现
- **查看测试**: 测试代码位于各模块的 `test/` 子目录（不在本文档范围）

---

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目定位
- [架构说明](02_Architecture.md) - 深入理解设计
- [GN Targets](05_GN_Targets.md) - 了解构建系统
