# 项目概览

本文档描述 ArkCompiler ETS Runtime 的项目定位、边界、核心能力、运行环境和关键概念。

## 项目定位

ArkCompiler ETS Runtime 是 OpenHarmony 系统的 **ETS（ECMAScript TypeScript）运行时**，负责执行由 ArkCompiler 前端工具链（ts2abc）生成的字节码（ABC 文件）。它位于应用层与系统内核之间，为 OpenHarmony 应用提供 JavaScript/TypeScript 执行环境。

### 在系统架构中的位置

```
┌─────────────────────────────────────────────────────────────────┐
│                    Application Layer (应用层)                      │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │              ArkTS/JS Applications (ArkTS/JS 应用)        │   │
│  └───────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│              ArkCompiler Toolchain (编译工具链)                   │
│  ┌──────────────────┐  ┌──────────────────────────────────┐   │
│  │  ArkTS Compiler   │→│     ts2abc → ABC Bytecode        │   │
│  └──────────────────┘  └──────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│              ArkCompiler ETS Runtime（本项目）                   │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │  解释器 │ 编译器 │ 内存管理 │ N-API │ 工具链              │   │
│  └───────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                    System Kernel (系统内核)                      │
│     进程管理 │ 内存管理 │ 文件系统 │ 网络 │ 安全机制            │
└─────────────────────────────────────────────────────────────────┘
```

## 项目边界

### 职责范围

- **字节码执行**：解释执行 ABC 字节码，支持解释器模式
- **编译优化**：AOT 静态编译和 JIT 运行时编译
- **内存管理**：对象分配、垃圾回收、内存泄漏检测
- **API 接口**：为 C++ 原生模块提供 N-API 编程接口
- **标准支持**：实现 ECMAScript 2021 标准库
- **工具支持**：调试器、性能分析器、热修复工具

### 非职责范围

- **前端编译**：由 arkcompiler_ets_frontend 负责
- **系统调用**：依赖 OpenHarmony 系统内核
- **UI 渲染**：由 ArkUI 或其他 UI 框架负责
- **网络通信**：由系统网络库负责

## 核心能力

### 1. 字节码解释与执行

ArkCompiler ETS Runtime 实现了高效的字节码解释器，能够直接执行 ABC 格式的字节码文件。解释器采用栈式架构，支持 ECMAScript 标准的所有指令集。

**关键文件**：
- `ecmascript/interpreter/interpreter.cpp` - 解释器核心实现
- `ecmascript/interpreter/interpreter-inl.cpp` - 内联优化实现

### 2. AOT 与 JIT 编译

运行时支持两种编译优化模式：
- **AOT（Ahead-of-Time）**：在应用启动前将字节码编译为机器码，显著提升启动性能
- **JIT（Just-in-Time）**：运行时动态编译热点代码，优化执行效率

**关键文件**：
- `ecmascript/compiler/` - 编译器实现
- `ecmascript/jit/` - JIT 编译器实现

### 3. 内存管理

实现了多种垃圾回收算法：
- **分代 GC（Generational GC）**：基于对象年龄的分代回收
- **并发标记清除（CMS GC）**：并发标记减少暂停时间
- **CMC GC**：并发混合回收器

**关键文件**：
- `ecmascript/mem/` - 内存管理模块

### 4. N-API 接口

提供稳定的 C++ 原生 API 接口，允许原生模块与运行时交互。这是 OpenHarmony 原生模块开发的基础接口。

**关键文件**：
- `ecmascript/napi/jsnapi.cpp` - N-API 实现
- `ecmascript/napi/include/jsnapi.h` - N-API 头文件

### 5. ECMAScript 标准库

完整实现了 ECMAScript 2021 标准规定的内置对象和函数，包括：
- 数据类型（String、Number、Boolean、Symbol）
- 集合对象（Array、Map、Set、WeakMap、WeakSet）
- 异步支持（Promise、AsyncFunction）
- 工具对象（Object、Function、JSON）

**关键文件**：
- `ecmascript/builtins/` - 内置对象实现
- `ecmascript/js_api/` - 非标准扩展 API

## 运行环境

### 支持平台

| 平台 | 状态 | 说明 |
|------|------|------|
| OpenHarmony（标准系统） | 支持 | 主要目标平台 |
| Linux | 支持 | 开发调试平台 |
| Windows | 支持 | 开发调试平台（预览） |
| macOS | 支持 | 开发调试平台 |
| Android | 有限支持 | 运行时兼容 |
| iOS | 有限支持 | 运行时兼容 |

### 硬件要求

- **CPU**：ARM64（推荐）、x86_64、x86
- **内存**：最小 64MB，推荐 128MB 以上
- **存储**：运行时库约 10-20MB

### 依赖库

- **libuv**：异步 I/O 事件循环
- **ICU**：国际化支持（可选）
- **zlib**：数据压缩
- **libc**：标准 C 库

## 关键概念

### ABC 字节码格式

ABC（ArkCompiler Bytecode）是 ArkCompiler 定义的字节码格式，由 ts2abc 工具从 TypeScript/ArkTS 源码生成。每个 ABC 文件包含常量池、方法定义、类型信息等。

### TaggedValue

运行时内部使用 TaggedValue 表示所有 JavaScript 值，采用指针标记技术（Pointer Tagging）区分数据类型，提高内存效率。

### 句柄（Handle）

运行时使用句柄机制管理 JavaScript 对象的引用，确保在 GC 期间正确追踪和更新引用。

### 安全上下文（Context）

每个执行实例拥有独立的 Context，包含全局对象、符号表、执行状态等。

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
