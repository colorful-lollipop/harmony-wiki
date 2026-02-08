# 内部 API 文档

本文档描述方舟工具链的内部模块接口，包括 Inner API 定义、模块依赖方向、稳定性标注和使用建议。

## 模块职责边界

### WebSocket 层 (websocket/)

**稳定性**: 稳定

**职责**: 提供可靠的全双工通信通道，实现 WebSocket 协议的握手、帧编解码、消息传输等功能。

**导出接口**:

| 接口 | 类型 | 说明 |
|------|------|------|
| `WebSocketServer` | 类 | WebSocket 服务器端实现 |
| `WebSocketClient` | 类 | WebSocket 客户端实现 |
| `WebSocketBase` | 基类 | 连接状态管理和通用接口 |

**依赖方向**: 被 inspector 层依赖，不依赖其他工具链模块。

**公开头文件**（`bundle.json` inner_kits 配置）：

```json
{
    "header_files": [
        "http.h",
        "server/websocket_server.h",
        "web_socket_frame.h",
        "websocket_base.h"
    ],
    "header_base": "//arkcompiler/toolchain/websocket"
}
```

### Inspector 层 (inspector/)

**稳定性**: 稳定

**职责**: 实现调试协议的接入层，包括 WebSocket 服务器封装、消息分发、会话管理和静态库加载。

**导出接口**:

| 接口 | 类型 | 说明 |
|------|------|------|
| `Inspector` | 类 | 调试器主入口 |
| `WsServer` | 类 | WebSocket 服务器包装 |
| `InitStatic` | 类 | 静态库初始化器 |
| `LibraryLoader` | 类 | 动态库加载器 |
| `ConnectInspector` | 类 | 连接调试器 |
| `ConnectServer` | 类 | 连接服务器 |

**依赖方向**: 依赖 WebSocket 层，输出可执行调试服务器。

### Tooling 层 (tooling/)

**稳定性**: 稳定（核心功能），实验性（部分新功能）

**职责**: 实现调试调优协议的核心逻辑，分为四个功能域。

**子模块结构**:

| 子模块 | 稳定性 | 职责 |
|--------|--------|------|
| `dynamic/` | 稳定 | 动态调试协议实现（Debugger/Profiler/Runtime） |
| `static/` | 稳定 | 静态分析协议实现 |
| `hybrid_step/` | 实验性 | 混合步进功能 |
| `dynamic/client/` | 稳定 | 调试客户端 CLI 工具 |

**导出接口（按功能域）**:

| 功能域 | 主要类 | 说明 |
|--------|--------|------|
| Debugger | `DebuggerImpl` | 断点管理、单步控制、CallFrame 求值 |
| Runtime | `RuntimeImpl` | 运行时信息查询 |
| Profiler | `ProfilerImpl` | CPU 采样分析 |
| HeapProfiler | `HeapProfilerImpl` | 堆内存分析 |
| Tracing | `TracingImpl` | 调用链追踪 |
| DOM/CSS | `*Impl` | DOM/CSS 调试（Chrome DevTools 协议兼容） |

### Platform 层 (platform/)

**稳定性**: 稳定

**职责**: 提供跨平台的文件操作和网络编程抽象，屏蔽 Unix/Windows 平台差异。

**抽象接口**（`platform/file.h`）:

| 接口 | 说明 | 平台支持 |
|------|------|----------|
| `fd_t` | 文件描述符类型 | Unix/Windows 差异屏蔽 |
| `FdsanExchangeOwnerTag()` | fdsan 标签交换 | 仅 OHOS |
| `FdsanClose()` | 关闭文件描述符 | Unix/Windows |

## 依赖关系图

```
inspector/BUILD.gn
├── ark_debugger/ark_inspector.so
│   └── ark_debugger_static
│       └── websocket:libwebsocket_server (静态库)
│           └── platform:file.cpp (平台源文件)
│
└── connectserver_debugger/ark_connect_inspector.so
    └── connectserver_debugger_static
        └── websocket:libwebsocket_server

tooling/BUILD.gn
├── libark_ecma_debugger/libark_tooling.so
│   └── tooling/dynamic/libark_ecma_debugger_static
│       └── tooling/hybrid_step:arkhybridstep
│
└── libarkinspector_plus/arkinspector.so
    └── tooling/static/libarkinspector_plus_static
        └── runtime_core:arktsdisassembler (外部依赖)
        └── runtime_core:libarktsbase (外部依赖)
        └── runtime_core:libarkruntime (外部依赖)
```

## 稳定性标注规则

### 稳定接口（可依赖）

以下接口经过充分测试，可安全依赖：

- WebSocket 协议层（`websocket/`）
- Inspector 核心类（`inspector.cpp`, `ws_server.cpp`）
- 四个主要功能域的实现（Debugger/Runtime/Profiler/HeapProfiler）
- 分发器框架（`dispatcher.h/cpp`）

### 不稳定接口（谨慎使用）

以下接口可能发生变化：

- 混合步进功能（`tooling/hybrid_step/`）
- 客户端 CLI 工具内部实现
- 静态分析的增强功能

### 内部 API（禁止直接使用）

以下接口仅限模块内部使用，不保证稳定性：

- `*_impl.cpp` 中的辅助函数
- 未在头文件中导出的符号
- `common/` 中的内部工具类

## 接口设计模式

### 1. 分发器模式（Dispatcher）

所有协议命令通过分发器路由：

```cpp
// dispatcher.h
class DispatcherBase {
public:
    virtual ~DispatcherBase() = default;
    virtual void Dispatch(const DispatchRequest &request, DispatchResponse *response) = 0;
    virtual void SendResponse(const DispatchResponse &response) = 0;
};

class Dispatcher {
public:
    void Dispatch(const DispatchRequest &request, DispatchResponse *response);
private:
    std::unordered_map<std::string, std::unique_ptr<DispatcherBase>> dispatchers_;
};
```

### 2. 会话管理模式（Session）

```cpp
// session_manager.h (静态调试)
class SessionManager {
public:
    void AddSession(PtThread thread);
    void RemoveSession(const std::string &id);
    std::string GetSessionIdByThread(PtThread thread);
private:
    std::map<std::string, PtThread> sessions_;
};
```

### 3. 端点模式（Endpoint）

```cpp
// endpoint_base.h
class EndpointBase {
public:
    virtual ~EndpointBase() = default;
    virtual void OnCall(const std::string &method, const JsonObject &params, JsonObject *result);
    virtual void HandleMessage(const std::string &message);
    virtual void Reply(int32_t id, const JsonObject &result);
    virtual void ReplyError(int32_t id, int32_t code, const std::string &message);
};
```

## 可替换点

| 组件 | 可替换点 | 替换风险 |
|------|----------|----------|
| WebSocket 层 | 整个 websocket 模块 | 低（接口稳定） |
| 通信协议 | 分发器框架 | 中（需保持 CDP 兼容） |
| 运行时交互 | tooling/dynamic/agent/* | 高（依赖运行时内部 API） |
| 平台抽象 | platform/ | 低（仅文件操作） |

---

*相关文档：[02_Architecture.md](./02_Architecture.md) | [03_NAPI_Reference.md](./03_NAPI_Reference.md) | [05_GN_Build.md](./05_GN_Build.md)*
