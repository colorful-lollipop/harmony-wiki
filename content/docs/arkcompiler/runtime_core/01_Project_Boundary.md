# 项目定位与边界

## 项目定位

### 在 OpenHarmony 中的位置

ArkCompiler Runtime Core 是 OpenHarmony **arkcompiler 子系统**的核心组件之一：

```
OpenHarmony 系统架构:
┌─────────────────────────────────────────┐
│           应用层 (Applications)          │
├─────────────────────────────────────────┤
│           应用框架层 (Framework)          │
│         ┌───────────────┐               │
│         │   ArkUI 框架   │               │
│         └───────────────┘               │
├─────────────────────────────────────────┤
│           系统服务层 (System Services)    │
├─────────────────────────────────────────┤
│           子系统层 (Subsystems)           │
│   ┌─────────────────────────────────┐   │
│   │     arkcompiler 子系统           │   │
│   │  ┌───────────────────────────┐  │   │
│   │  │  ets_frontend (编译器前端) │  │   │
│   │  └───────────────────────────┘  │   │
│   │  ┌───────────────────────────┐  │   │
│   │  │  ets_runtime (语言运行时)  │  │   │
│   │  └───────────────────────────┘  │   │
│   │  ┌───────────────────────────┐  │   │
│   │  │ runtime_core (运行时核心) ←─┼──┼───┤   │
│   │  │  ┌─────┐ ┌─────┐ ┌─────┐ │  │   │
│   │  │  │File │ │Base │ │ ISA │ │  │   │
│   │  │  └─────┘ └─────┘ └─────┘ │  │   │
│   │  └───────────────────────────┘  │   │
│   └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│           内核层 (Kernel)                │
│         (Linux / LiteOS)                │
└─────────────────────────────────────────┘
```

### 核心职责边界

Runtime Core 的职责**包括**：

| 领域 | 职责 | 证据位置 |
|------|------|----------|
| 字节码执行 | 加载、解析、执行 .abc 字节码文件 | `libpandafile/`, `static_core/libarkfile/` |
| 内存管理 | 托管堆管理、GC 框架、内存分配器 | `static_core/runtime/mem/` |
| 线程管理 | 托管线程生命周期、线程同步 | `static_core/runtime/include/thread.h` |
| 类型系统 | 类加载、方法解析、字段访问 | `static_core/runtime/include/class.h` |
| 解释执行 | 字节码解释器框架 | `static_core/runtime/interpreter/` |
| 编译优化 | JIT/AOT 编译器框架 | `static_core/compiler/` |
| 工具支持 | 调试器、分析器接口 | `static_core/runtime/tooling/` |
| 原生接口 | ANI、N-API 互操作 | `static_core/plugins/ets/runtime/ani/` |

Runtime Core 的职责**不包括**：

| 领域 | 说明 | 所属组件 |
|------|------|----------|
| 源代码编译 | TS/ETS 到 ABC 的编译 | `arkcompiler_ets_frontend` |
| 语言语法实现 | 特定语言的语法特性 | `arkcompiler_ets_runtime` |
| UI 框架 | 界面渲染、布局 | `arkui` |
| 系统服务 |  Ability 生命周期管理 | `ability_runtime` |

## 核心能力

### 1. 多语言支持能力

Runtime Core 设计为**语言无关**的运行时核心，通过插件机制支持多语言：

```
多语言支持架构:
                    ┌─────────────────┐
                    │   ArkTS (ETS)   │ ← 当前主要支持
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  ETS Plugin     │
                    │  (语言插件)      │
                    │  ┌───────────┐  │
                    │  │ Language  │  │
                    │  │ Context   │  │
                    │  ├───────────┤  │
                    │  │Interpreter│  │
                    │  ├───────────┤  │
                    │  │ Intrinsics│  │
                    │  └───────────┘  │
                    └────────┬────────┘
                             │
┌─────────────┐     ┌────────▼────────┐     ┌─────────────┐
│  EcmaScript │────▶│  Runtime Core   │◀────│    Core     │
│   Plugin    │     │  (Language-     │     │   Plugin    │
└─────────────┘     │   Independent)  │     └─────────────┘
                    └─────────────────┘
```

**当前状态**: 主要支持 ETS 插件，EcmaScript 插件在开发中。

### 2. 执行模式能力

Runtime Core 支持多种代码执行模式：

| 执行模式 | 说明 | 适用场景 |
|----------|------|----------|
| **解释执行** | 逐条解释字节码 | 冷启动、低内存 |
| **JIT 编译** | 运行时热点代码编译 | 平衡性能与启动速度 |
| **AOT 编译** | 提前编译为机器码 | 高性能要求 |
| **混合模式** | 解释 + JIT + AOT | 默认模式 |

### 3. 内存管理能力

```
内存管理架构:
┌─────────────────────────────────────────┐
│           应用代码层                     │
├─────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    │
│  │  托管对象    │    │  原生对象    │    │
│  │ (GC 管理)   │    │ (手动管理)   │    │
│  └──────┬──────┘    └─────────────┘    │
├─────────┼───────────────────────────────┤
│         ▼                               │
│  ┌─────────────────────────────────┐    │
│  │         GC Framework            │    │
│  │  ┌─────┐ ┌─────┐ ┌─────┐      │    │
│  │  │G1GC │ │STW  │ │CMC  │ ...  │    │
│  │  └─────┘ └─────┘ └─────┘      │    │
│  └─────────────────────────────────┘    │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐    │
│  │      Memory Allocator           │    │
│  │  (TLAB / Region / Bump Pointer) │    │
│  └─────────────────────────────────┘    │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐    │
│  │       Pool Manager              │    │
│  │  (MMap / Malloc Memory Pools)   │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

支持的 GC 算法：
- **G1GC**: 分代垃圾收集器（默认）
- **STW GC**: 停止世界 GC
- **Epsilon GC**: 无操作 GC（测试用）
- **CMC GC**: 外部 CMC GC 适配器

### 4. 工具支持能力

| 工具类型 | 能力 | 实现位置 |
|----------|------|----------|
| **调试器** | 断点、单步、变量查看 | `static_core/runtime/tooling/inspector/` |
| **分析器** | CPU/内存/堆分析 | `static_core/runtime/tooling/sampler/` |
| **反汇编器** | ABC 到汇编文本 | `disassembler/`, `static_core/disassembler/` |
| **验证器** | 字节码安全性检查 | `verifier/`, `static_core/verification/` |

## 运行环境

### 支持的平台

| 平台 | 架构 | 状态 |
|------|------|------|
| OpenHarmony (标准系统) | ARM64 / ARM32 | 主要支持 |
| Linux | x86_64 / ARM64 | 开发/测试支持 |
| Windows (MinGW) | x86_64 | 工具链支持 |
| macOS | x86_64 / ARM64 | 开发支持 |
| iOS | ARM64 | 实验性支持 |
| Android | ARM64 | ArkUI-X 支持 |

### 系统要求

- **最小内存**: 取决于应用，基础运行时约 4MB
- **存储空间**: 基础库约 10MB
- **依赖组件**: 
  - `bounds_checking_function` - 边界检查
  - `hilog` - 日志输出
  - `zlib` - 压缩支持
  - `icu` - Unicode 支持

## 关键概念与术语

### 字节码相关

| 术语 | 说明 | 对应文件 |
|------|------|----------|
| **ABC** | Ark Bytecode，Ark 字节码格式 | `.abc` 文件 |
| **PA** | Panda Assembly，汇编文本格式 | `.pa` 文件 |
| **AN** | Ark Native，AOT 编译产物 | `.an` 文件 |
| **ISA** | Instruction Set Architecture，指令集架构 | `isa/` |

### 运行时相关

| 术语 | 说明 | 代码位置 |
|------|------|----------|
| **Managed Thread** | 托管线程，受运行时管理的线程 | `static_core/runtime/include/managed_thread.h` |
| **Object Header** | 对象头，包含 GC 标记和类型信息 | `static_core/runtime/include/object_header.h` |
| **Class Linker** | 类链接器，负责类加载和链接 | `static_core/runtime/include/class_linker.h` |
| **Entrypoint** | 入口点，方法调用的入口 | `static_core/runtime/entrypoints/` |

### 编译相关

| 术语 | 说明 | 代码位置 |
|------|------|----------|
| **IR** | Intermediate Representation，中间表示 | `static_core/compiler/optimizer/ir/` |
| **Inst** | Instruction，IR 指令 | `static_core/compiler/optimizer/ir/inst.h` |
| **BB** | Basic Block，基本块 | `static_core/compiler/optimizer/ir/basicblock.h` |
| **Graph** | 控制流图 | `static_core/compiler/optimizer/ir/graph.h` |

### 接口相关

| 术语 | 说明 | 代码位置 |
|------|------|----------|
| **ANI** | Ark Native Interface，原生接口 | `static_core/plugins/ets/runtime/ani/ani.h` |
| **N-API** | Node-API，JS 互操作接口 | `static_core/plugins/ets/runtime/interop_js/` |
| **Interop** | 互操作，ETS 与 JS 之间的调用 | `static_core/plugins/ets/runtime/interop_js/` |

## 版本与兼容性

### 文件格式版本

ABC 文件格式有版本控制：
- 文件头包含版本信息
- 运行时检查兼容性
- 见 `libpandafile/file.h`

### API 兼容性

- **ANI API**: 向后兼容
- **内部 API**: 可能随版本变化
- **N-API**: 遵循 Node-API 规范

## 下一步

- 查看 [目录结构](02_Directory_Structure.md) 了解代码组织
- 阅读 [架构说明](03_Architecture.md) 理解组件关系
- 了解 [对外 API](04_Public_API.md) 进行 Native 开发
