# 项目概览

## 什么是 ArkCompiler Runtime Core

ArkCompiler Runtime Core（以下简称 Runtime Core）是 OpenHarmony 方舟编译器的运行时核心组件，提供语言无关的基础运行时能力，支撑 ArkTS（Extended TypeScript）等语言在 OpenHarmony 上的高效执行。

## 核心定位

Runtime Core 是 OpenHarmony 语言运行时的**公共基础模块**，位于编译器架构的底层支撑位置：

```
┌─────────────────────────────────────────┐
│         ArkTS 应用/框架层                │
├─────────────────────────────────────────┤
│      ArkCompiler ETS Frontend           │
│         (编译器前端：es2panda)            │
├─────────────────────────────────────────┤
│      ArkCompiler ETS Runtime            │
│    ┌─────────────────────────────┐      │
│    │  Language-specific Runtime  │      │
│    │  (语言相关运行时)            │      │
│    └─────────────────────────────┘      │
├─────────────────────────────────────────┤
│      ArkCompiler Runtime Core  ←────    │
│    ┌─────────────────────────────┐      │
│    │  ┌─────┐ ┌─────┐ ┌─────┐  │      │
│    │  │File │ │Base │ │ ISA │  │      │
│    │  │(文件)│ │(基础)│ │(指令)│  │      │
│    │  └─────┘ └─────┘ └─────┘  │      │
│    │  ┌─────┐ ┌─────────────┐  │      │
│    │  │Tooling│ │ Compiler    │  │      │
│    │  │(工具) │ │ (编译器)    │  │      │
│    │  └─────┘ └─────────────┘  │      │
│    └─────────────────────────────┘      │
├─────────────────────────────────────────┤
│         操作系统抽象层 (OSAL)            │
├─────────────────────────────────────────┤
│      Linux / OpenHarmony / ...          │
└─────────────────────────────────────────┘
```

## 四大核心组件

### 1. ArkCompiler File（字节码文件）

**职责**: 定义和执行 Ark 字节码（.abc 文件）

**关键能力**:
- 字节码文件格式定义与解析
- 类、方法、字段的元数据管理
- 常量池与字符串表管理
- 调试信息存储

**主要代码位置**:
- `libpandafile/` - 字节码文件处理（旧版）
- `static_core/libarkfile/` - 字节码文件处理（新版静态核心）

**关键类/符号**:
- `panda_file::File` - 字节码文件表示
- `panda_file::ClassDataAccessor` - 类数据访问器
- `panda_file::MethodDataAccessor` - 方法数据访问器

### 2. Base（基础运行时库）

**职责**: 提供平台无关的基础工具能力

**关键能力**:
- 内存管理（内存池、分配器）
- 同步原语（锁、信号量、条件变量）
- 日志系统
- 命令行参数解析
- JSON 处理
- UTF 字符串处理

**主要代码位置**:
- `libpandabase/` - 基础库（旧版）
- `static_core/libarkbase/` - 基础库（新版）

**关键类/符号**:
- `PoolManager` - 内存池管理器
- `Logger` - 日志系统
- `PandArgParser` - 命令行解析
- `Mutex` / `ConditionVariable` - 同步原语

### 3. ISA（指令集架构）

**职责**: 定义语言无关的字节码指令集

**关键能力**:
- 字节码指令定义（YAML 描述）
- 指令编码/解码
- 指令模板生成

**主要代码位置**:
- `isa/` - ISA 描述文件与生成脚本
- `static_core/isa/` - 静态核心 ISA

**关键文件**:
- `isa/isa.yaml` - 指令集定义
- `templates/` - 代码生成模板

### 4. Tooling（工具支持）

**职责**: 支撑开发与调试工具

**关键能力**:
- 运行时调试器（Debugger）
- 性能分析（Profiler）
- 堆转储（Heap Dump）
- 采样器（Sampler）

**主要代码位置**:
- `static_core/runtime/tooling/` - 工具实现

## 扩展组件

### Compiler（编译器）

**职责**: JIT/AOT 编译优化

**主要代码位置**:
- `compiler/` - 编译器（旧版）
- `static_core/compiler/` - 编译器（新版）

**关键能力**:
- 中间表示（IR）优化
- 代码生成
- AOT 提前编译

### Bytecode Optimizer（字节码优化器）

**职责**: 字节码层面的优化

**主要代码位置**:
- `bytecode_optimizer/`
- `static_core/bytecode_optimizer/`

### Verifier（字节码验证器）

**职责**: 字节码安全性验证

**主要代码位置**:
- `verifier/`
- `static_core/verification/`

## 关键概念

### ABC 文件格式

Ark Bytecode（.abc）是 Runtime Core 的执行载体：

```
ABC 文件结构:
┌─────────────────┐
│   Header        │  文件头（魔数、版本、校验和）
├─────────────────┤
│   Index Section │  索引区（类、方法、字符串索引）
├─────────────────┤
│   Code Section  │  代码区（字节码指令）
├─────────────────┤
│   Data Section  │  数据区（常量池、字符串表）
├─────────────────┤
│   Debug Section │  调试信息（行号、变量名）
└─────────────────┘
```

### 托管内存与非托管内存

Runtime Core 区分两种内存：

- **托管内存（Managed Memory）**: 由 GC 管理的对象内存
  - 位于堆空间
  - 自动回收
  - 需要 GC 屏障

- **非托管内存（Native Memory）**: 由开发者管理的内存
  - 使用 `malloc`/`free`
  - 不受 GC 影响
  - 用于运行时内部结构

### 语言插件机制

Runtime Core 通过插件机制支持多语言：

```
插件架构:
┌─────────────────────────────────┐
│        Language Plugin          │
│    ┌─────────────────────┐      │
│    │  Language Context   │      │
│    │  (语言上下文)        │      │
│    └─────────────────────┘      │
│    ┌─────────────────────┐      │
│    │  Interpreter        │      │
│    │  (解释器)            │      │
│    └─────────────────────┘      │
│    ┌─────────────────────┐      │
│    │  Intrinsics         │      │
│    │  (内建函数)          │      │
│    └─────────────────────┘      │
└─────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│     Runtime Core (Language-     │
│        Independent Core)        │
└─────────────────────────────────┘
```

当前主要插件：
- **ETS 插件** (`static_core/plugins/ets/`) - ArkTS 语言支持

## 系统能力

Runtime Core 对外暴露的系统能力（syscap）：

- `SystemCapability.ArkCompiler.ANI` - Ark Native Interface 支持

## 特性开关

通过 GN args 可启用的主要特性：

| 特性 | GN Arg | 说明 |
|------|--------|------|
| 代码生成 | `runtime_core_enable_codegen` | 启用 JIT/AOT 编译器 |
| FFRT | `runtime_core_enable_ffrt` | 启用 FFRT 运行时 |
| IRTOC | `enable_irtoc` | 启用 IR 到代码转换 |
| LLVM 后端 | `is_llvmbackend` | 使用 LLVM 作为编译后端 |

## 与相关仓库的关系

```
arkcompiler/
├── runtime_core/          ← 本文档覆盖范围
│   ├── 基础运行时
│   ├── 字节码文件
│   └── 工具链
│
├── ets_runtime/           ← 依赖 runtime_core
│   └── ArkTS 运行时实现
│
└── ets_frontend/          ← 生成 .abc 文件
    └── es2panda 编译器前端
```

## 下一步

- 了解 [项目定位与边界](01_Project_Boundary.md)
- 查看 [目录结构](02_Directory_Structure.md)
- 深入 [架构说明](03_Architecture.md)
