# OpenHarmony NetStack 架构说明

## 1. 系统定位

### 1.1 模块定位

**NetStack 是 OpenHarmony 通信子系统的核心网络协议栈库**，而非系统服务。

| 特性 | 说明 |
|------|------|
| **类型** | 客户端网络库 (Client Library) |
| **角色** | 为应用提供 HTTP/Socket/WebSocket API |
| **依赖** | 调用 netmanager_base 系统服务 (SA) |
| **非角色** | 不是 SystemAbility、无 IPC 服务端实现 |

### 1.2 与系统服务的关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                       │
│  JS/ETS 应用通过 N-API 调用                                      │
│  C/C++ 应用通过 FFI 调用                                         │
└─────────────────────────────────────────────────────────────────┘
                              │ 调用网络 API
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      NetStack (当前代码库)                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ HTTP Client │  │ WebSocket   │  │ TLS Socket              │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
│                                                                    │
│  使用 IPC 客户端模式调用:                                           │
│  • NetConnClient::GetInstance().GetDefaultHttpProxy()             │
│  • NetConnClient::GetInstance().GetAllNets()                      │
│  • NetConnClient::ObtainBundleNameForSelf()                       │
└─────────────────────────────────────────────────────────────────┘
                              │ IPC 调用 (samgr_proxy)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│               NetManager_Base (System Ability)                   │
│                    SA ID: 约 115                                 │
│  • 网络连接管理                                                  │
│  • 权限校验                                                      │
│  • 默认代理配置                                                   │
└─────────────────────────────────────────────────────────────────┘
```

**关键证据**:
- `frameworks/js/napi/http/BUILD.gn:145-148`: `external_deps = ["samgr:samgr_proxy", "ipc:ipc_single"]`
- `frameworks/js/napi/http/http_exec.cpp:2246-2360`: 使用 `sptr<NetManagerStandard::NetHandle>`

---

## 2. 组件架构

### 2.1 代码组织结构

```
netstack/
│
├── frameworks/                    # 【对外接口实现】
│   ├── js/napi/                 # 标准系统 JS N-API
│   │   ├── http/               # HTTP 请求 API
│   │   ├── socket/            # Socket API (TCP/UDP/TLS/Unix)
│   │   ├── websocket/         # WebSocket API
│   │   ├── net_ssl/           # 网络安全 API
│   │   └── fetch/             # Fetch API
│   ├── js/builtin/            # 小型系统 JS API
│   ├── native/                # Native 接口
│   ├── ets/                  # ArkTS 接口 (ANI)
│   └── cj/                    # CJJ FFI 接口
│
├── interfaces/                  # 【接口定义】
│   ├── kits/                  # JS 接口声明 (.d.ts)
│   │   ├── @ohos.net.http.d.ts
│   │   ├── @ohos.net.socket.d.ts
│   │   └── @ohos.net.webSocket.d.ts
│   └── innerkits/             # Native 接口头文件 (.h)
│       ├── http_client/
│       ├── net_ssl/
│       └── websocket_native/
│
└── utils/                      # 【内部公共模块】
    ├── common_utils/           # 通用工具
    ├── log/                   # 日志
    ├── napi_utils/            # N-API 工具函数 ⭐ 核心
    └── http_over_curl/        # HTTP over cURL 实现
```

### 2.2 N-API 模块注册架构

**两种注册模式**:

| 模式 | 使用模块 | 示例 |
|------|----------|------|
| **NAPI_MODULE 宏** | fetch | `NAPI_MODULE(fetch, FetchModule::InitFetchModule)` |
| **napi_module_register** | http, websocket, net_ssl, socket | `napi_module_register(&g_httpModule)` |

**N-API 工具层** (`utils/napi_utils/src/napi_utils.cpp`):
```cpp
// 核心工具函数
void DefineProperties(...)    // → napi_define_properties
napi_value CreateFunction(...) // → napi_create_function  
void SetNamedProperty(...)   // → napi_set_named_property
```

---

## 3. 核心模块详解

### 3.1 HTTP 模块

**入口**: `createHttp()` → 返回 `HttpRequest` 对象

**N-API 注册**: `http_module.cpp:540-552`
```cpp
static napi_module g_httpModule = {
    .nm_version = 1,
    .nm_modname = "net.http",
    .nm_register_func = HttpModuleExports::InitHttpModule,
};
napi_module_register(&g_httpModule);
```

**核心类** (Native):
| 类 | 文件 | 说明 |
|---|------|------|
| `HttpSession` | `interfaces/innerkits/http_client/include/http_client.h` | HTTP 会话管理器 |
| `HttpClientTask` | `interfaces/innerkits/http_client/include/http_client_task.h` | HTTP 请求任务 |
| `HttpRequest` | `interfaces/innerkits/http_client/include/http_client_request.h` | HTTP 请求封装 |
| `HttpResponse` | `interfaces/innerkits/http_client/include/http_client_response.h` | HTTP 响应封装 |

**功能特性**:
- ✅ HTTP/HTTPS 请求 (GET/POST/PUT/DELETE 等)
- ✅ 请求/响应头管理
- ✅ HTTP 缓存策略
- ✅ 客户端证书认证
- ✅ HTTP 代理配置
- ✅ 证书锁定 (Certificate Pinning)

### 3.2 Socket 模块

**入口函数**:
- `constructTCPSocketInstance()` → TCPSocket
- `constructUDPSocketInstance()` → UDPSocket
- `constructTLSSocketInstance()` → TLSSocket

**N-API 注册**: `socket_module.cpp:1190-1204`

**Socket 类型**:
| 类型 | 支持能力 |
|------|----------|
| **TCPSocket** | connect, bind, send, close, listen, accept |
| **UDPSocket** | bind, send, close, multicast, broadcast |
| **TLSSocket** | TLS 握手, 证书验证, 双向认证 |
| **MulticastSocket** | 组播管理 (addMembership/dropMembership) |
| **LocalSocket** | Unix Domain Socket 客户端 |
| **LocalSocketServer** | Unix Domain Socket 服务端 |

**TLS 子模块**:
TLS 模块**不是独立 N-API**，而是通过 socket 模块初始化:
```cpp
// socket_module.cpp:510-530
TlsSocket::TLSSocketModuleExports::InitTLSSocketModule(env, exports);
TlsSocketServer::TLSSocketServerModuleExports::InitTLSSocketServerModule(env, exports);
```

### 3.3 WebSocket 模块

**入口**: `createWebSocket()` → 返回 `WebSocket` 对象

**N-API 注册**: `websocket_module.cpp:215-227`

**核心能力**:
- ✅ WebSocket 连接建立/关闭
- ✅ 消息发送/接收 (text/binary)
- ✅ 心跳保活 (ping/pong)
- ✅ HTTP 代理支持
- ✅ 客户端证书认证

### 3.4 网络安全模块 (NetSSL)

**入口**: 主要作为内部模块使用

**N-API 注册**: `net_ssl_module.cpp:119-131`

**核心功能**:
| 功能 | 说明 |
|------|------|
| `certVerification()` | X509 证书异步验证 |
| `certVerificationSync()` | X509 证书同步验证 |
| `isCleartextPermitted()` | 明文传输许可检查 |
| `isCleartextPermittedByHostName()` | 按主机名检查明文许可 |

---

## 4. 数据流

### 4.1 HTTP 请求数据流

```
JS/ETS 应用
    │
    ▼
N-API 层 (http_module)
    │
    ├─► 参数解析与校验
    │   ├── URL 格式验证
    │   ├── 请求头解析
    │   └── 权限检查 (HasInternetPermission)
    │
    ├─► HttpRequest 构建
    │   └── 调用 native http_client
    │
    ▼
Native 层 (http_client)
    │
    ├─► HttpSession::CreateTask()
    │   ├── 配置 cURL 选项
    │   ├── 设置 TLS/SSL 参数
    │   └── 配置代理
    │
    ▼
HTTP over cURL (utils/http_over_curl)
    │
    ├─► DNS 解析
    ├─► TCP 连接建立
    ├─► TLS 握手 (HTTPS)
    ├─► HTTP 请求发送
    ├─► 响应接收
    │
    ▼
IPC 调用 netmanager_base
    │
    ├─► 获取默认代理
    ├─► 权限校验
    │
    ▼
返回响应数据 (回调/Promise)
```

### 4.2 Socket 连接数据流

```
JS Socket API
    │
    ▼
N-API 层 (socket_module)
    │
    ├─► Socket 类型分发
    │   ├── TCPSocket → tcp_*
    │   ├── UDPSocket → udp_*
    │   └── TLSSocket → tls_*
    │
    ▼
Native TLS Socket 实现
    │
    ├─► OpenSSL 初始化
    ├─► socket() → 创建 fd
    ├─► connect() / bind()
    ├─► TLS 握手 (如需要)
    │
    ▼
事件循环 (libwebsockets / epoll)
    │
    ├─► 消息接收
    ├─► 回调通知 JS 层
    │
    ▼
JS 事件监听器 (on('message'))
```

---

## 5. 线程模型

### 5.1 HTTP 请求线程

**依赖**: FFRT (轻量级线程池)

**关键类**: `HttpSession`
- 位置: `interfaces/innerkits/http_client/include/http_client.h:41`
- 模式: 单例模式 `HttpSession::GetInstance()`

**线程使用**:
1. 主线程: N-API 调用入口
2. FFRT 线程池: 实际 HTTP 请求执行
3. 回调线程: 响应返回到 JS

### 5.2 Socket 线程模型

**模式**: 异步事件驱动

| 组件 | 线程 |
|------|------|
| N-API 调用 | 主线程 |
| socket I/O | libwebsockets 事件线程 |
| TLS 握手 | 事件线程 |
| JS 回调 | 主线程 (通过 napi_schedule_task) |

---

## 6. 关键设计决策

### 6.1 为什么 NetStack 不是 SystemAbility？

| 考虑因素 | 决定 |
|----------|------|
| **性能** | 库加载更快，无需 IPC 即可执行 |
| **权限** | 权限校验委托给 netmanager_base SA |
| **复杂度** | 简化网络编程模型 |
| **职责** | 专注于协议实现，不做连接管理 |

### 6.2 子模块设计 (TLS within Socket)

**设计原因**:
- TLS Socket 与 TCP Socket 共享大部分代码
- 复用 `SocketModuleExports` 的类和函数
- 避免命名空间污染

**实现方式**:
```cpp
// socket_module.cpp 中
class SocketModuleExports {
public:
    static napi_value InitSocketModule(napi_env env, napi_value exports) {
        // 初始化 TLS 子模块
        TlsSocket::TLSSocketModuleExports::InitTLSSocketModule(env, exports);
        // ... 其他初始化
    }
};
```

---

## 7. 安全边界

### 7.1 信任边界

| 边界 | 说明 |
|------|------|
| **应用 ↔ NetStack** | JS 运行时沙箱 |
| **NetStack ↔ netmanager_base** | IPC 通道，权限校验 |
| **NetStack ↔ 网络** | TLS 加密通道 |

### 7.2 安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| **权限检查** | `CommonUtils::HasInternetPermission()` | 调用 netmanager_base |
| **证书验证** | OpenSSL + net_ssl | X509 验证 |
| **证书锁定** | http_request_options.cpp | Certificate Pinning |
| **TLS 版本控制** | tls_configuration.cpp | 禁用旧版本 |
| **输入验证** | 各模块 napi_utils | URL、header 等 |

---

## 相关文档
- [N-API Reference](02_N-API_Reference.md)
- [Native API](03_Inner_API.md)
- [Build Targets](04_Build_Targets.md)
- [Security Review](05_Security_Review.md)
