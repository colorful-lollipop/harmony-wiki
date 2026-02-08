# 目录结构与模块职责

本文档描述方舟工具链仓库的目录结构和各模块的职责边界，帮助开发者快速定位代码位置和理解模块划分。

## 顶层目录结构

```
/arkcompiler/toolchain
├── adapter              # 平台适配层，提供跨平台抽象接口
├── common               # 公共工具代码，被多个模块共享使用
├── docs                 # 项目文档
├── figures              # 图片资源（架构图等）
├── inspector            # 调试协议对接层，实现 WebSocket 服务器和消息转发
├── platform             # 平台相关代码（Unix/Windows 实现差异）
├── tooling              # 调试调优协议核心实现
│   ├── dynamic          # 动态分析工具实现（调试器、分析器）
│   ├── static           # 静态分析工具实现
│   └── hybrid_step      # 混合步进相关实现
├── websocket            # WebSocket 协议实现
│   ├── server           # 服务器端实现
│   └── client           # 客户端实现
├── test                 # 测试代码（文档中不引用）
├── BUILD.gn             # 根构建入口
├── toolchain.gni        # 构建配置模板
├── toolchain_config.gni # 构建配置定义
├── bundle.json          # 组件描述文件
└── README_zh.md         # 项目说明文档
```

## 核心模块职责

### tooling 目录

`tooling/` 目录是调试调优协议的核心实现所在，包含四个功能子模块。

`tooling/dynamic/` 实现动态分析相关的协议命令，包括调试器的断点管理、单步控制，以及 CPU 分析器和堆内存分析器的数据采集逻辑。该模块的静态库 `libark_ecma_debugger_static` 被编译为 `libark_tooling.so`（输出名），是运行时加载的核心调试组件。

`tooling/static/` 实现静态分析相关的功能，可能包括字节码反汇编等能力，具体实现需进一步分析源码确认。

`tooling/hybrid_step/` 实现混合步进功能，用于在某些特定场景下的执行控制，具体功能边界需结合代码分析。

### inspector 目录

`inspector/` 目录实现调试协议的接入层，包括以下核心功能。

**WebSocket 服务器**通过 `ws_server.cpp` 实现，负责监听客户端连接、管理会话生命周期。服务器以动态库形式输出为 `ark_inspector.so`（ark_debugger 的 output_name）。

**消息转发**机制将接收到的协议消息路由到对应的协议域处理器，并将处理结果返回给客户端。

**静态库加载**通过 `init_static.cpp` 和 `library_loader.cpp` 实现，支持动态加载静态编译的调试组件。

### websocket 目录

`websocket/` 目录提供完整的 WebSocket 协议实现，被 inspector 和其他模块复用。

**协议基础实现**包括 `websocket_base.cpp`（WebSocket 帧处理）、`frame_builder.cpp`（帧构建）、`handshake_helper.cpp`（握手辅助）、`http.cpp`（HTTP 协议支持）和 `network.cpp`（网络通信）。

**服务器实现**位于 `server/websocket_server.cpp`，提供 WebSocket 服务器端功能。

**客户端实现**位于 `client/websocket_client.cpp`，提供客户端连接能力。

该模块输出静态库 `libwebsocket_server`，被 `inspector/BUILD.gn` 直接依赖。

### platform 目录

`platform/` 目录处理平台相关代码差异，根据目标平台选择不同的实现。

`platform/unix/file.cpp` 提供 Unix/Linux/macOS 平台的系统调用封装。

`platform/windows/file.cpp` 提供 Windows 平台的系统调用封装。

具体差异包括文件操作、网络编程、线程同步等 API 的平台实现细节。

### common 目录

`common/` 目录包含被多个模块共享的公共代码，如日志包装器（`log_wrapper.cpp`）等。

### adapter 目录

`adapter/` 目录提供平台适配层，抽象跨平台的统一接口，具体功能边界需结合代码分析确认。

## 模块依赖关系

根据 `BUILD.gn` 文件分析，主要的模块依赖关系如下：

inspector 模块依赖 websocket 模块的 `libwebsocket_server`，提供 WebSocket 服务器能力。tooling 模块依赖 runtime_core 的多个组件（arktsdisissembler、libarktsbase、libarkruntime），提供运行时交互能力。所有模块依赖 hiviewdfx 相关的系统能力（hilog、hitrace、faultloggerd），提供日志和追踪能力。

---

*相关文档：[00_Overview.md](./00_Overview.md) | [02_Architecture.md](./02_Architecture.md) | [05_GN_Build.md](./05_GN_Build.md)*
