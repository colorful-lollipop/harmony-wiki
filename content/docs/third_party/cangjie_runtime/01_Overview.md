# 01 - 原始库简介

## 1.1 基本信息

| 属性 | 详情 |
|-----|------|
| **库名称** | cangjie_runtime |
| **中文名** | 仓颉运行时 |
| **版本** | 1.1.0-alpha.69 |
| **许可证** | Apache-2.0 with Runtime Library Exceptions |
| **上游地址** | https://cangjie-lang.cn/ |
| **维护者** | 华为仓颉语言团队 (zhangbin1@huawei.com) |
| **项目定位** | 仓颉编程语言 Native 后端核心运行时 |

## 1.2 功能概述

仓颉运行时是**仓颉编程语言 Native 后端（CJNative）的核心组件**，以高性能和轻量化为设计目标，为仓颉语言在全场景下提供高性能支撑。

### 核心功能模块

```
┌─────────────────────────────────────────────────────────────┐
│                    Cangjie Runtime                          │
├──────────────┬──────────────┬──────────────┬────────────────┤
│   Garbage    │   CJThread   │   Exception  │    Loader      │
│  Collection  │  Management  │   Handling   │   Manager      │
├──────────────┼──────────────┼──────────────┼────────────────┤
│  Stack Trace │   Object     │     FFI      │      DFX       │
│   & Unwind   │   Model      │  (C/ArkTS)   │  Profiling     │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

#### 1. 垃圾回收（Garbage Collection）

**设计目标**：低延迟、全并发、内存紧凑

- **全并发 GC**：消除 Stop-The-World (STW) 停顿，优化执行延迟
- **内存整理（Sweep）**：通过对象重定位整理堆内存碎片，提高内存利用率，支持长时间运行应用
- **指针标记（Pointer Tag）**：在引用处应用指针标记，区分垃圾内存和新复用内存，确保全并发 GC 的正确性

#### 2. 线程管理（CJThread）

提供轻量灵活的线程管理方案，更好应对各种规模的并发场景。

- **调度器（Schedule）**：包含线程、监视器、处理器、schmon 等基础模块，充分利用多核硬件资源
- **栈增长（Stack Grow）**：采用连续栈，容量达到上限时自动翻倍扩容

#### 3. 异常处理（Exception）

两级异常处理机制，按严重程度分为：

- **Exception**：运行时逻辑错误或 IO 故障导致的异常（如数组越界、文件不存在）。必须在程序中显式捕获处理。
- **Error**：仓颉运行时内部系统错误和资源耗尽情况。应用不应抛出此类错误，发生时需通知用户并安全终止。

#### 4. 模块加载（Loader）

支持包粒度加载和管理仓颉代码，具备反射能力。

#### 5. 栈回溯（Stack Trace）

基于帧指针的栈展开机制：
- `rbp` 寄存器存储当前栈帧基地址
- `rsp` 寄存器存储栈顶指针
- 函数调用时将前一栈帧地址压栈保存

#### 6. FFI（Foreign Function Interface）

支持与 C/ArkTS 的函数调用和数据交换。

#### 7. DFX（Debug & Fix）

提供调试调优能力：
- 日志系统（Log）
- CPU Profiling
- 堆快照（Heap Snapshot）
- 运行时状态检查和故障诊断

## 1.3 标准库（Standard Library）

除运行时外，本仓库还包含仓颉标准库（std），提供丰富的内置库功能。

### 标准库模块列表（31个）

| 模块 | 功能 |
|-----|------|
| `std.core` | 核心包 |
| `std.collection` | 常用数据结构 |
| `std.collection.concurrent` | 并发集合 |
| `std.sync` | 并发编程原语 |
| `std.io` | IO 操作 |
| `std.fs` | 文件系统 |
| `std.net` | 网络通信 |
| `std.time` | 时间处理 |
| `std.math` / `std.math.numeric` | 数学计算 |
| `std.regex` | 正则表达式 |
| `std.crypto` / `std.crypto.cipher` / `std.crypto.digest` | 加密/摘要 |
| `std.database` / `std.database.sql` | 数据库访问 |
| `std.reflect` | 反射功能 |
| `std.ast` | 语法解析 |
| `std.process` | 进程管理 |
| `std.posix` | POSIX 接口适配 |
| `std.console` | 控制台交互 |
| `std.convert` | 类型转换 |
| `std.env` | 进程环境 |
| `std.random` | 随机数生成 |
| `std.unicode` | 字符处理 |
| `std.sort` | 排序算法 |
| `std.binary` | 二进制处理 |
| `std.overflow` | 溢出处理 |
| `std.ref` | 弱引用 |
| `std.objectpool` | 对象缓存 |
| `std.argopt` | 命令行解析 |
| `std.deriving` | 宏自动生成 |
| `std.unittest` | 单元测试 |

## 1.4 第三方依赖

运行时和标准库依赖以下第三方组件：

| 库 | 用途 | 集成方式 |
|---|------|---------|
| libboundscheck | 边界检查函数 | 源码依赖，编译集成 |
| OpenSSL | 加密功能 | 动态链接系统库 |
| PCRE2 | 正则表达式 | 源码依赖，编译集成 |
| flatbuffers | 序列化（ast 模块） | 源码依赖，编译集成 |

## 1.5 OpenHarmony 中的定位

### 在 OH 技术栈中的位置

```
┌─────────────────────────────────────────────────┐
│              仓颉应用程序 (Cangjie App)           │
├─────────────────────────────────────────────────┤
│              仓颉标准库 (31+ modules)            │
├─────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────┐   │
│  │         仓颉运行时 (本仓库)              │   │
│  │  • GC / Thread / Exception / Loader     │   │
│  └─────────────────────────────────────────┘   │
├─────────────────────────────────────────────────┤
│  OpenHarmony Native Layer (Musl/Bionic)        │
├─────────────────────────────────────────────────┤
│  OpenHarmony Kernel (Linux)                    │
└─────────────────────────────────────────────────┘
```

### OH 集成模式

与传统第三方库不同，cangjie_runtime 在 OH 中采用**预编译库分发模式**：

```
┌──────────────────────────────────────────────────┐
│                 开发流程                          │
├──────────────────────────────────────────────────┤
│  1. 源码开发（cangjie-lang.cn 上游仓库）          │
│     ↓                                           │
│  2. 交叉编译（Linux → OHOS ARM64/x86_64/ARM32）  │
│     ↓                                           │
│  3. 预编译产物 → OH 预编译 SDK 仓库              │
│     ↓                                           │
│  4. OH BUILD.gn 引用预编译 .so 文件              │
│     ↓                                           │
│  5. 安装到系统目录（platformsdk/cjsdk）          │
└──────────────────────────────────────────────────┘
```

### 支持的 OH 平台

| 平台 | 架构 | 状态 |
|-----|------|------|
| OpenHarmony | aarch64 | ✅ 已支持 |
| OpenHarmony | x86_64 | ✅ 已支持（模拟器） |
| OpenHarmony | arm | ⏳ 计划中（2025 Q4） |

### OH 系统要求

- **最低版本**：OpenHarmony 5.1+
- **系统类型**：Standard（标准系统）
- **ROM**：25MB
- **RAM**：20MB

## 1.6 仓库结构

```
third_party/cangjie_runtime/
├── README.md                    # 英文 README
├── README_zh.md                 # 中文 README
├── LICENSE                      # Apache-2.0 + Runtime Exception
├── README.OpenSource            # OH 开源信息
├── bundle.json                  # OH 组件配置
├── BUILD.gn                     # OH 构建入口（预编译库引用）
├── platform.gni                 # 平台配置
├── OAT.xml                      # OH 开源合规
│
├── runtime/                     # 运行时源码
│   ├── build/                   # 构建脚本
│   │   ├── cmake/toolchain/     # 交叉编译工具链（含 OH）
│   │   └── scripts/             # 辅助脚本
│   └── src/                     # 源代码
│       ├── Base/                # 基础功能
│       ├── CJThread/            # 线程管理
│       ├── Heap/                # 堆内存/GC
│       ├── Exception/           # 异常处理
│       ├── Loader/              # 模块加载
│       ├── ObjectModel/         # 对象模型
│       ├── Signal/              # 信号处理
│       ├── Sync/                # 同步原语
│       ├── UnwindStack/         # 栈展开
│       ├── arch/                # 硬件架构适配
│       └── os/                  # OS 适配层
│           ├── Linux/           # Linux 适配（OH 共用）
│           ├── Macos/           # macOS 适配
│           └── Windows/         # Windows 适配
│
└── stdlib/                      # 标准库源码
    ├── libs/std/                # 标准库各模块
    │   ├── core/
    │   ├── collection/
    │   ├── sync/
    │   ├── io/
    │   ├── fs/
    │   ├── net/
    │   └── ... (31 个模块)
    ├── third_party/             # 标准库依赖的第三方库
    └── cmake/                   # CMake 配置（含 OH 工具链）
```

## 1.7 相关资源

- **官方网站**：https://cangjie-lang.cn/
- **官方文档**：https://cangjie-lang.cn/docs
- **标准库 API**：https://cangjie-lang.cn/docs?url=%2F1.0.0%2Flibs%2Fstd%2Fstd_module_overview.html
- **构建指南**：https://gitcode.com/Cangjie/cangjie_build
- **OH SIG 仓库**：
  - [cangjie_compiler](https://gitcode.com/openharmony-sig/third_party_cangjie_compiler)
  - [cangjie_tools](https://gitcode.com/openharmony-sig/third_party_cangjie_tools)
  - [cangjie_stdx](https://gitcode.com/openharmony-sig/third_party_cangjie_stdx)

---

*本文档基于 cangjie_runtime 1.1.0-alpha.69 版本编写*
