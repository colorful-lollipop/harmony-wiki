# 安全风险评审

本文档对方舟工具链进行安全风险分析，包括攻击面识别、信任边界分析和可被利用点（带有代码证据）。

## 评审范围

本评审覆盖以下代码路径（排除测试）：

| 模块 | 路径 | 评审状态 |
|------|------|----------|
| WebSocket | `websocket/` | 已评审 |
| Inspector | `inspector/` | 已评审 |
| Tooling | `tooling/dynamic/`, `tooling/static/` | 已评审 |
| Platform | `platform/` | 已评审 |
| Common | `common/` | 已评审 |

## 攻击面清单

### 1. 网络输入（高风险）

**入口点**: WebSocket 服务器连接

**涉及文件**:
- `websocket/server/websocket_server.cpp` - 服务器接受连接
- `websocket/websocket_base.cpp` - 消息接收与解析
- `websocket/http.cpp` - HTTP 握手解析

**攻击向量**:
- 恶意 WebSocket 握手请求导致拒绝服务
- 超大消息帧导致缓冲区溢出
- 恶意构造的 HTTP 请求头导致解析错误

### 2. 协议解析（中风险）

**入口点**: JSON 协议消息解析

**涉及文件**:
- `tooling/dynamic/dispatcher.cpp` - 消息分发
- `tooling/static/connection/endpoint_base.cpp` - JSON-RPC 处理

**攻击向量**:
- 深层嵌套 JSON 导致栈溢出
- 超大 JSON 字符串导致内存耗尽
- 恶意 JSON 字段名导致哈希碰撞（Hash DoS）

### 3. 文件输入（低风险）

**入口点**: Panda 文件执行

**涉及文件**:
- `tooling/dynamic/client/ark_multi/main.cpp` - 文件加载

**攻击向量**:
- 恶意构造的 ABC/PANDA 文件导致解析错误

### 4. 运行时交互（低风险）

**入口点**: JSNApi 调用

**涉及文件**:
- `tooling/dynamic/backend/debugger_executor.cpp` - 调试器执行器

**攻击向量**:
- 通过调试接口执行恶意代码（仅开发阶段）

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  DevEco     │    │  调试工具链  │    │   Runtime   │     │
│  │  Studio     │───►│  (可信)     │───►│  (可信)     │     │
│  │  (可信)     │    │             │    │             │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│        │                                      │            │
│        │         WebSocket 连接                │            │
│        └──────────────────────────────────────┘            │
│                                                             │
│  边界外：不可信输入（网络、文件）                             │
│  边界内：可信代码（工具链、Runtime）                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 可被利用点

### 风险 1：WebSocket 数据帧处理（低风险）

**证据位置**: `websocket/server/websocket_server.cpp:61-76`

```cpp
auto& buffer = wsFrame.payload;
buffer.resize(msgLen, 0);
if (!RecvUnderLock(buffer)) {
    return false;
}
// XOR 解码
for (size_t i = 0; i < msgLen; i++) {
    buffer[i] = static_cast<uint8_t>(buffer[i]) ^ wsFrame.maskingKey[j];
    j = (j + 1) % WebSocketFrame::MASKING_KEY_LEN;
}
```

**分析**: 使用 `std::vector::resize` 动态分配缓冲区，`RecvUnderLock` 内部实现使用循环接收确保完整读取。XOR 解码操作在已分配的缓冲区范围内进行，边界安全。

**评估**: **低风险** - 标准库容器自动管理内存，无缓冲区溢出风险。

**建议**: 添加消息大小上限检查，防止内存耗尽攻击。

### 风险 2：JSON 解析深度限制（中风险）

**证据位置**: `tooling/dynamic/base/pt_json.cpp:37-55`

```cpp
std::unique_ptr<PtJson> PtJson::Parse(const std::string &data) {
    cJSON *value = cJSON_ParseWithLength(data.c_str(), data.size());
    if (value == nullptr) {
        return nullptr;
    }
    return std::make_unique<PtJson>(value);
}
```

**分析**: 使用 cJSON 库进行解析，依赖外部库的安全性。cJSON 库本身对解析深度有一定限制，但未明确文档化。

**触发方式**: 发送包含数百层嵌套的 JSON 消息（如 `{"a":{"a":{"a":...}}}`）。

**影响**: 可能导致栈溢出或内存耗尽，造成 DoS。

**评估**: **中风险** - 依赖外部库实现，建议添加应用层防护。

**修复建议**:
```cpp
// 在 Parse 前添加大小检查
constexpr size_t MAX_JSON_SIZE = 10 * 1024 * 1024;  // 10MB
if (data.size() > MAX_JSON_SIZE) {
    LOGE("JSON message too large: %zu", data.size());
    return nullptr;
}
```

### 风险 3：WebSocket 消息大小限制不完整（中风险）

**证据位置**: `websocket/websocket_base.cpp:166` 和 `websocket/server/websocket_server.cpp`

```cpp
// HTTP 握手有长度限制
static constexpr size_t HTTP_HANDSHAKE_MAX_LEN = 1024;

// 但 WebSocket 数据帧使用 payloadLen（64位）
// websocket_base.cpp:91-103
uint8_t recvbuf[WebSocketFrame::TWO_BYTES_LENTH] = {0};
if (!RecvUnderLock(recvbuf, WebSocketFrame::TWO_BYTES_LENTH)) {
    return false;
}
wsFrame.payloadLen = NetToHostLongLong(recvbuf, WebSocketFrame::TWO_BYTES_LENTH);
```

**分析**: HTTP 握手阶段有 1KB 限制，但 WebSocket 数据帧依赖 `payloadLen` 字段（最大 64 位），理论上可表示超大消息，实际受限于 `std::vector` 分配能力。

**触发方式**: 发送声明超大 payload 长度的 WebSocket 帧。

**影响**: 尝试分配超大内存可能导致 OOM 或进程被终止。

**评估**: **中风险** - 虽有实际限制，但缺乏明确的应用层检查。

**修复建议**:
```cpp
// 在 Decode 方法中添加显式大小检查
constexpr size_t MAX_WS_MESSAGE_SIZE = 64 * 1024 * 1024;  // 64MB
if (wsFrame.payloadLen > MAX_WS_MESSAGE_SIZE) {
    LOGE("WebSocket message too large: %llu", wsFrame.payloadLen);
    CloseConnection(CloseStatusCode::MESSAGE_TOO_BIG);
    return false;
}
```

### 风险 4：HTTP 头解析（低风险）

**证据位置**: `websocket/http.cpp:79-142`

```cpp
bool DecodeHttpRequest(const std::string& request, HttpRequest& decodedRequest) {
    // 使用 C++ string 操作，非固定缓冲区
    size_t pos = request.find("\r\n");
    if (pos == std::string::npos) {
        return false;
    }
    // 包含 ValidateHttpRequestOrigin 原点验证
    if (!ValidateHttpRequestOrigin(decodedRequest)) {
        return false;
    }
}
```

**分析**: 使用 `std::string` 动态管理内存，不存在固定缓冲区溢出问题。包含原点验证机制。

**评估**: **低风险** - 现代 C++ 容器自动管理内存，且有原点验证。

**建议**: 添加请求头总大小限制，防止资源耗尽。

### 风险 5：调试器权限边界（低风险 - 开发阶段）

**证据位置**: `tooling/dynamic/agent/debugger_impl.cpp:138-167` 和 `tooling/dynamic/utils/utils.cpp:122-138`

```cpp
// 应用文件检查
bool DebuggerImpl::IsApplicationFile(const std::string &fileName) {
    // 检查文件路径是否在应用目录 /data 下
    if (fileName.substr(0, DATA_APP_PATH.length()) != DATA_APP_PATH) {
        return false;
    }
    // 检查 module.json 中的 debugable 标志
    // ...
}

// 路径规范化
std::string RealPath(const std::string& path) {
    char buffer[PATH_MAX] = { '\0' };
    if (!realpath(path.c_str(), buffer)) {
        return "";
    }
    return std::string(buffer);
}
```

**分析**: 
1. `IsApplicationFile` 限制调试文件必须在 `/data` 应用目录下
2. `RealPath` 使用 `realpath` 进行路径规范化，防止路径遍历攻击
3. 调试功能仅在开发阶段启用，生产环境默认关闭

**触发方式**: 尝试通过调试接口访问系统文件或执行越权操作。

**影响**: 理论上可执行任意代码，但受限于：
- 调试器仅在开发模式可用
- 文件访问受限于应用目录
- 需要 DevEco Studio 连接认证

**评估**: **低风险** - 仅在开发阶段暴露，且有多重权限控制。

**建议**: 在生产构建中完全禁用调试器功能（通过宏控制）。

## 安全机制

### 已有的安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| bounds_checking_function | 外部依赖 | 编译时边界检查 |
| OpenSSL 加密 | websocket/BUILD.gn | WebSocket 加密 |
| 文件描述符安全 | platform/file.cpp | fdsan 标签管理 |

### 建议增强的安全机制

1. **输入验证层**: 在消息入口处添加统一的输入验证
2. **资源限制**: 添加内存、CPU、连接数的运行时限制
3. **沙箱隔离**: 考虑使用沙箱执行不可信代码
4. **审计日志**: 记录所有安全相关事件

## 安全评估结论

### 总体评估

方舟工具链的整体安全状况**良好**，主要原因是：

1. **攻击面有限**: 仅在调试阶段暴露，且需要用户主动启用
2. **依赖安全库**: 使用 OpenSSL、bounds_checking_function 等安全库
3. **现代 C++ 实践**: 广泛使用 `std::string`、`std::vector` 等自动管理内存的容器
4. **安全函数**: 使用 `strncpy_s`、`memcpy_s`、`realpath` 等安全函数
5. **权限控制**: 文件访问限制在应用目录，防止路径遍历

### 主要风险等级（更新）

| 风险类型 | 等级 | 位置 | 说明 |
|----------|------|------|------|
| JSON 解析 | 中 | `pt_json.cpp:37` | 需确认 cJSON 深度限制或添加大小检查 |
| WebSocket 消息大小 | 中 | `websocket_base.cpp` | payloadLen 未显式限制 |
| HTTP 头解析 | 低 | `http.cpp:79` | 使用 `std::string`，安全 |
| WebSocket 帧处理 | 低 | `websocket_server.cpp:61` | 使用标准库容器，边界安全 |
| 调试器权限 | 低 | `debugger_impl.cpp:138` | 有路径检查和规范化 |
| 路径遍历 | 低 | `utils.cpp:122` | 使用 `realpath` 规范化 |

### 改进优先级

1. **高优先级**:
   - 添加 WebSocket 数据帧大小上限检查（建议 64MB）
   - 添加 JSON 消息大小检查（建议 10MB）

2. **中优先级**:
   - 确认 cJSON 库解析深度限制
   - 添加 HTTP 请求头总大小限制

3. **低优先级**:
   - 审计日志记录
   - 运行时资源监控
   - 表达式求值沙箱

---

*相关文档：[02_Architecture.md](./02_Architecture.md) | [05_GN_Build.md](./05_GN_Build.md) | [08_Troubleshooting.md](./08_Troubleshooting.md)*
