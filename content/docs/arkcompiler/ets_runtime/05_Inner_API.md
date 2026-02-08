# 内部 API

本文档描述 ArkCompiler ETS Runtime 的内部 API（Inner API），包括模块接口、依赖方向、稳定性标注和可替换点。

## Inner API 概述

Inner API 是运行时内部各模块之间使用的编程接口，与 N-API 不同，这些接口不保证稳定性，可能随版本变化。

### 稳定性分级

| 级别 | 说明 | 使用范围 |
|------|------|----------|
| **稳定（Stable） | 跨模块使用的公共接口，变更需评审 | 多个模块使用 |
| **实验性（Experimental） | 新功能接口，可能变化 | 单模块或少数模块使用 |
| **内部（Internal） | 模块私有接口 | 仅模块内部使用 |
| **废弃（Deprecated） | 已废弃接口，不建议使用 | 兼容保留 |

## 核心 Inner API

### 1. VM 核心接口

**头文件**：`ecmascript/ecma_vm.h`

| 接口 | 说明 | 稳定性 |
|------|------|--------|
| `EcmaVM::GetGlobal() | 获取全局对象 | 稳定 |
| `EcmaVM::GetNativePointerMap()` | 获取原生指针映射 | 稳定 |
| `EcmaVM::GetJSThread()` | 获取 JS 线程 | 稳定 |
| `EcmaVM::GetHeap() | 获取堆对象 | 稳定 |
| `EcmaVM::GetRuntime() | 获取 Runtime | 稳定 |

### 2. 对象模型接口

**头文件**：`ecmascript/js_object.h`

| 接口 | 说明 | 稳定性 |
|------|------|--------|
| `JSObject::GetPrototype() | 获取原型链 | 稳定 |
| `JSObject::SetPrototype() | 设置原型链 | 稳定 |
| `JSObject::GetProperty() | 获取属性 | 稳定 |
| `JSObject::SetProperty() | 设置属性 | 稳定 |

### 3. 数组操作接口

**头文件**：`ecmascript/js_array.h`

| 接口 | 说明 | 稳定性 |
|------|------|--------|
| `JSArray::GetLength() | 获取数组长度 | 稳定 |
| `JSArray::SetLength() | 设置数组长度 | 稳定 |
| `JSArray::IsFast() | 检查是否为快数组 | 稳定 |

### 4. 字符串接口

**头文件**：`ecmascript/ecma_string.h`

| 接口 | 说明 | 稳定性 |
|------|------|--------|
| `EcmaString::GetLength() | 获取字符串长度 | 稳定 |
| `EcmaString::GetData() | 获取字符串数据 | 稳定 |
| `EcmaString::Concat() | 字符串连接 | 稳定 |

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                       模块依赖关系图                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────┐                                            │
│   │   napi/       │                                            │
│   └──────┬───────┘                                            │
│          │                                                     │
│          ▼                                                     │
│   ┌──────────────┐                                            │
│   │   js_api/     │                                            │
│   │  containers/  │                                            │
│   └──────┬───────┘                                            │
│          │                                                     │
│          ▼                                                     │
│   ┌──────────────┐                                            │
│   │  builtins/    │                                            │
│   └──────┬───────┘                                            │
│          │                                                     │
│          ▼                                                     │
│   ┌──────────────┐                                            │
│   │ interpreter/  │◄────────────────────────────────┐            │
│   └──────┬───────┘                                 │            │
│          │                                          │            │
│          ▼                                          │            │
│   ┌──────────────┐                                  │            │
│   │   compiler/   │                                  │            │
│   └──────┬───────┘                                  │            │
│          │                                          │            │
│          ▼                                          │            │
│   ┌──────────────┐                                  │            │
│   │   mem/        │─────────────────────────────────┘            │
│   └──────────────┘                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 主要模块接口

### interpreter 模块

**头文件**：`ecmascript/interpreter/interpreter.h`

| 接口 | 说明 | 依赖模块 |
|------|------|----------|
| `Interpreter::Execute()` | 执行字节码 | runtime |
| `Interpreter::Call()` | 函数调用 | runtime, gc |
| `FrameHandler::Push()` | 推入栈帧 | runtime |
| `FrameHandler::Pop()` | 弹出栈帧 | runtime |

### compiler 模块

**头文件**：`ecmascript/compiler/compiler.h`

| 接口 | 说明 | 依赖模块 |
|------|------|----------|
| `Compiler::Compile()` | 编译方法 | interpreter |
| `AOTCompiler::CompileFile()` | 编译文件 | jspandafile |
| `JITCompiler::Compile()` | JIT 编译 | interpreter |

### mem 模块

**头文件**：`ecmascript/mem/mem.h`

| 接口 | 说明 | 依赖模块 |
|------|------|----------|
| `Heap::Allocate()` | 分配对象 | thread |
| `GC::CollectGarbage()` | 垃圾回收 | heap |
| `GC::WaitForFinish()` | 等待 GC 完成 | thread |

### jspandafile 模块

**头文件**：`ecmascript/jspandafile/js_pandafile.h`

| 接口 | 说明 | 依赖模块 |
|------|------|----------|
| `JsPandafile::Load()` | 加载文件 | - |
| `JsPandafileManager::GetInstance()` | 获取管理器 | - |
| `ClassInfoExtractor::Extract()` | 提取类信息 | builtin_classes |

## 可替换点

以下位置设计为可替换或可扩展点：

### 1. GC 策略可替换

**位置**：`ecmascript/mem/gc*.cpp`

- 分代 GC 可替换为 CMS GC
- 支持自定义回收策略

### 2. 编译器后端可替换

**位置**：`ecmascript/compiler/codegen/`

- LLVM 后端
- Maple 后端

### 3. 平台适配层可替换

**位置**：`ecmascript/platform/`

- Unix 平台适配
- Windows 平台适配
- OpenHarmony 适配

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
- [构建系统](06_Build_System.md)
