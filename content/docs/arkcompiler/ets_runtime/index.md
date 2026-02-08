# ArkCompiler ETS Runtime

ArkCompiler ETS Runtime 是 OpenHarmony 的默认 ETS 运行时，支持 ECMAScript 标准库和高效的容器库，提供一组原生 C++ API，以及高性能的垃圾回收器。

## 核心能力

- **解释器**：高效执行 ArkCompiler 字节码（ABC 文件）
- **编译器**：支持 AOT（Ahead-of-Time）和 JIT（Just-in-Time）编译优化
- **内存管理**：多种垃圾回收器（分代 GC、并发 GC 等）
- **N-API 接口**：为 C++ 原生模块提供稳定的编程接口
- **工具链**：调试器、性能分析器、热修复工具

## 快速开始

### 运行 ABC 字节码

```bash
# 设置运行时库路径
export LD_LIBRARY_PATH=out/.../arkcompiler/ets_runtime:...

# 运行字节码文件
./ark_js_vm helloworld.abc
```

### 构建项目

```bash
./build.sh --product-name hispark_taurus_standard --build-target ark_js_host_linux_tools_packages
```

## 项目定位

ArkCompiler ETS Runtime 位于 OpenHarmony 系统架构的**运行时层**，负责执行 ArkCompiler 生成的字节码，为应用提供运行环境。

```
┌─────────────────────────────────────────────────┐
│           Application Layer (应用层)              │
├─────────────────────────────────────────────────┤
│           ArkCompiler Toolchain (编译工具链)      │
│           ts2abc → abc 文件                       │
├─────────────────────────────────────────────────┤
│      ArkCompiler ETS Runtime (本项目)            │
│      解释器 / 编译器 / GC / N-API                 │
├─────────────────────────────────────────────────┤
│           OpenHarmony Kernel (内核层)            │
└─────────────────────────────────────────────────┘
```

## 技术约束

- 仅支持 ArkCompiler 字节码（ts2abc 生成）
- 仅支持 ES2021 标准与严格模式
- 不支持动态函数创建（如 `new Function(...)`）

## 相关仓库

- [arkcompiler_runtime_core](https://gitee.com/openharmony/arkcompiler_runtime_core)
- [arkcompiler_ets_frontend](https://gitee.com/openharmony/arkcompiler_ets_frontend)
