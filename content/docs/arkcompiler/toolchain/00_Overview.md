# 项目概览

本文档介绍 OpenHarmony 方舟工具链组件的整体定位、核心能力、运行环境依赖和关键概念定义。

## 项目定位

方舟工具链（ArkCompiler Toolchain）是 OpenHarmony 系统中面向 ArkTS 应用程序的调试与性能调优工具集。它作为 DevEco Studio 与 ArkCompiler Runtime 之间的桥梁，为开发者提供完整的调试、性能分析和内存分析能力。该组件属于 `arkcompiler` 子系统，组件名称为 `toolchain`，版本为 3.1，采用 Apache License 2.0 开源协议。

从功能定位来看，方舟工具链主要解决 ArkTS 应用程序开发过程中的调试难题、性能瓶颈定位和内存优化问题。它通过实现标准化的调试调优协议，使 DevEco Studio 能够与运行时的虚拟机实例进行交互，从而实现断点调试、CPU 采样分析、堆内存快照等功能。

## 核心能力

方舟工具链提供四大核心功能域，每一域对应不同的调试调优场景。

**调试器域（Debugger Domain）** 提供完整的调试控制能力，包括断点设置与管理、单步执行控制（Step In/Step Over/Step Out）、执行暂停与恢复、调用栈帧（CallFrame）求值以及断点命中事件通知。这些能力使开发者能够在代码层面精确控制程序执行流程，观察变量状态，定位逻辑错误。

**CPU 分析器域（Profiler Domain）** 提供 CPU 性能采样能力，支持启动和停止采样、配置采样间隔、收集采样数据并传输给前端展示。通过 CPU 采样，开发者可以识别热点函数和性能瓶颈，优化代码执行效率。

**堆内存分析器域（HeapProfiler Domain）** 提供堆内存分析能力，支持获取堆内存快照、触发垃圾回收、追踪内存分配。这些能力帮助开发者发现内存泄漏、定位内存占用过高的对象，从而优化应用的内存使用。

**运行时域（Runtime Domain）** 提供运行时信息的查询能力，包括堆内存使用情况统计、对象属性获取、运行时状态暴露等。这些信息为调试和分析提供基础数据支撑。

## 运行环境

方舟工具链的运行依赖以下系统组件和环境条件。

**运行时依赖**方面，方舟工具链依赖 ArkCompiler Runtime（ets_runtime）提供运行时信息，依赖 runtime_core 提供核心运行时能力。这两个组件是工具链正常运行的前提条件。

**系统能力依赖**方面，根据 `bundle.json` 的配置，方舟工具链适配标准系统（standard），依赖以下系统能力：`bounds_checking_function`（安全函数）、`faultloggerd`（故障日志）、`init`（系统初始化）、`hitrace`（调用链追踪）、`hilog`（日志输出）、`hisysevent`（系统事件）以及 `ffrt`（任务调度）。

**平台支持**方面，方舟工具链支持多种目标平台，包括 OHOS（标准设备）、Linux、Android、iOS、macOS 和 Windows。不同平台通过 `platform/` 目录下的平台相关代码进行适配，具体实现差异见 `platform/unix/file.cpp` 和 `platform/windows/file.cpp`。

**编译器支持**方面，方舟工具链支持 GN 构建系统，配置通过 `toolchain.gni` 和 `toolchain_config.gni` 定义，支持独立编译模式（ark_standalone_build）和集成编译模式。

## 关键概念

### 调试调优协议

调试调优协议是 DevEco Studio 与工具链之间通信的标准规范，采用基于 JSON 的消息格式，通过 WebSocket 连接传输。协议定义了多个域（Domain），每个域包含一系列协议命令（Command），用于实现特定的调试或调优功能。

### 会话与连接

工具链采用客户端-服务器架构，会话（Session）代表一次完整的调试连接。服务器端通过 `ws_server.cpp` 实现 WebSocket 服务器，监听连接请求并管理会话生命周期。连接建立后，消息在客户端（DevEco Studio）和服务器端之间双向传输。

### 消息帧与协议解析

WebSocket 消息采用帧（Frame）结构传输，通过 `frame_builder.cpp` 和 `websocket_base.cpp` 实现消息的封装与解析。协议消息包含消息类型、域标识、命令参数和数据负载等字段。

### 静态编译与动态链接

工具链组件存在两种形态：静态库和动态库。静态库（如 `libwebsocket_server`）作为内部依赖被链接到最终产物，动态库（如 `libark_tooling.so`）则在运行时被加载。这种设计既保证了代码复用的效率，又提供了插件化扩展的能力。

---

*相关文档：[01_Directory_Structure.md](./01_Directory_Structure.md) | [02_Architecture.md](./02_Architecture.md)*
