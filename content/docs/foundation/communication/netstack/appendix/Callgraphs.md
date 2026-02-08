# OpenHarmony NetStack 关键调用链

## 1. HTTP 请求调用链

### 1.1 JS → Native → cURL

```
┌─────────────────────────────────────────────────────────────────────┐
│ JS 层                                                                   │
├─────────────────────────────────────────────────────────────────────┤
│ @ohos.net.http.createHttp()                                          │
│   ↓                                                                   │
│ HttpRequest.request(url, options)                                      │
└─────────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────────┐
│ N-API 层 (http_module.cpp)                                            │
├─────────────────────────────────────────────────────────────────────┤
│ HttpModuleExports::InitHttpModule(env, exports)                        │
│   ↓                                                                   │
│ HttpRequest::NewInstance(env, callback)                               │
│   ↓                                                                   │
│ HttpExec::ExecuteRequest(env, context)                                │
│   ↓                                                                   │
│ napi_create_async_work() → ExecuteCallback + CompleteCallback        │
└─────────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────────┐
│ Native 层 (http_client)                                              │
├─────────────────────────────────────────────────────────────────────┤
│ HttpSession::GetInstance()                                            │
│   ↓                                                                   │
│ HttpSession::CreateTask(request) → HttpClientTask                     │
│   ↓                                                                   │
│ HttpClientTask::Start()                                               │
│   ↓                                                                   │
│ HttpOverCurl::Execute()                                               │
└─────────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────────┐
│ cURL 层 (utils/http_over_curl)                                        │
├─────────────────────────────────────────────────────────────────────┤
│ EpollMultiDriver::AddRequest(request)                                 │
│   ↓                                                                   │
│ CurlSocketContext::MultiPerform()                                     │
│   ↓                                                                   │
│ curl_easy_perform(handle)                                            │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 权限检查调用链

```
JS Request
    ↓
CommonUtils::HasInternetPermission()
    ↓
┌─────────────────────┐
│ 条件编译            │
├─────────────────────┤
│ #ifdef OH_CORE_...  │ → socket(AF_INET, SOCK_STREAM, 0)
│     (测试模式)       │   ↓
│     return true;    │   errno == EPERM?
└─────────────────────┘         ↓
        ↓               ┌─────────────────────┐
   getgroups()         │ 无权限              │
   ↓                   │ return false;       │
   检查 inetGroup      └─────────────────────┘
   (40002003)
        ↓
   ┌─────────────────────┐
   │ 找到权限组          │
   │ return true;        │
   └─────────────────────┘
```

### 1.3 证书验证调用链

```
TLS Handshake
    ↓
TLSContext::SetVerify()
    ↓
┌──────────────────────────────┐
│ 验证模式选择                 │
├──────────────────────────────┤
│ 无客户端证书 → ONE_WAY_MODE  │
│ 有客户端证书 → TWO_WAY_MODE │
│ skipFlag → VERIFY_NONE ⚠️  │
└──────────────────────────────┘
    ↓
SSL_CTX_set_verify()
    ↓
X509_verify_cert()
    ↓
┌──────────────────────────────────────────────┐
│ CA 证书加载                                     │
├──────────────────────────────────────────────┤
│ 1. 系统 CA: "/etc/security/certificates/user/"  │
│ 2. 用户 CA: "/data/certificates/user_cacerts/ │
│    /{uid}/"                                  │
│ 3. 自定义 CA (options.ca)                     │
└──────────────────────────────────────────────┘
    ↓
验证结果返回
```

---

## 2. Socket 连接调用链

### 2.1 TCP Socket 连接

```
JS: tcpSocket.connect({address, timeout})
    ↓
N-API: ConnectContext::RunConnectFunc()
    ↓
Native: socket(AF_INET, SOCK_STREAM, 0)
    ↓
connect(fd, addr, addrlen)
    ↓
┌──────────────────────────────────────────────┐
│ 非阻塞模式                                    │
├──────────────────────────────────────────────┤
│ select() / epoll_wait()                       │
│   ↓                                          │
│ 可写事件 → 连接建立                           │
│ 超时 → ETIMEDOUT                             │
└──────────────────────────────────────────────┘
```

### 2.2 TLS Socket 连接

```
JS: tlsSocket.connect(options)
    ↓
TLSSocket::Connect(options, manager)
    ↓
┌──────────────────────────────────────────────┐
│ TCP 连接建立                                  │
├──────────────────────────────────────────────┤
│ socket() → bind() → connect()                │
│ (复用 TCPSocket 逻辑)                         │
└──────────────────────────────────────────────┘
    ↓
SSL_new(ctx) → SSL_set_fd()
    ↓
SSL_connect()
    ↓
┌──────────────────────────────────────────────┐
│ TLS 握手过程                                  │
├──────────────────────────────────────────────┤
│ ClientHello → ServerHello                     │
│ Certificate ← ServerCertificate               │
│ CertificateRequest ← (双向认证时)            │
│ ClientCertificate → Certificate               │
│ ClientKeyExchange → Finished                │
│ Finished → [Encrypted Data]                  │
└──────────────────────────────────────────────┘
    ↓
证书验证 (见证书验证调用链)
    ↓
连接就绪回调 → JS on('connect')
```

---

## 3. WebSocket 连接调用链

```
JS: ws.connect(url, options)
    ↓
WebSocketModule::Connect(env, info)
    ↓
WebSocketClient::Connect(url, options)
    ↓
┌──────────────────────────────────────────────┐
│ HTTP Upgrade 请求                             │
├──────────────────────────────────────────────┤
│ GET / HTTP/1.1                               │
│ Host: {host}                                 │
│ Upgrade: websocket                          │
│ Connection: Upgrade                         │
│ Sec-WebSocket-Key: {随机Base64}            │
│ Sec-WebSocket-Version: 13                   │
└──────────────────────────────────────────────┘
    ↓
curl_http_exec() 发送请求
    ↓
┌──────────────────────────────────────────────┐
│ 101 Switching Protocols 响应                  │
├──────────────────────────────────────────────┤
│ HTTP/1.1 101 Switching Protocols            │
│ Upgrade: websocket                          │
│ Connection: Upgrade                        │
│ Sec-WebSocket-Accept: {计算值}             │
└──────────────────────────────────────────────┘
    ↓
libwebsocket 创建会话
    ↓
事件循环: lws_service()
    ↓
JS on('open') 回调
```

---

## 4. 事件监听注册链

### 4.1 Socket 消息事件

```
JS: socket.on('message', callback)
    ↓
N-API: OnMessage(env, info)
    ↓
EventManager::AddEventListener()
    ↓
┌──────────────────────────────────────────────┐
│ EventListener 结构                            │
├──────────────────────────────────────────────┤
│ type: "message"                               │
│ callback: NAPI 回调函数                       │
│ threadId: 当前线程                           │
│ pending: false                               │
└──────────────────────────────────────────────┘
    ↓
注册到 Native EventManager
    ↓
Socket I/O 事件触发
    ↓
EventManager::NotifyEvent()
    ↓
napi_call_function() → JS callback
```

---

## 5. 异步工作流程

### 5.1 Promise 模式

```
JS: promise = httpRequest.request(url)
    ↓
napi_create_promise()
    ↓
napi_create_async_work()
    ↓
┌──────────────────────────────────────────────┐
│ ExecuteCallback (工作线程)                   │
├──────────────────────────────────────────────┤
│ • 解析 URL                                    │
│ • DNS 解析                                    │
│ • TCP 连接                                    │
│ • TLS 握手 (HTTPS)                          │
│ • 发送请求                                    │
│ • 接收响应                                    │
└──────────────────────────────────────────────┘
    ↓
CompleteCallback (主线程)
    ↓
napi_resolve_promise()
    ↓
JS .then() 回调
```

### 5.2 Callback 模式

```
JS: httpRequest.request(url, callback)
    ↓
napi_create_async_work()
    ↓
工作线程执行 (同 Promise)
    ↓
CompleteCallback (主线程)
    ↓
napi_call_function(env, global, callback, argc, argv)
    ↓
JS callback(err, data)
```

---

## 6. 关键文件索引

| 功能 | 文件路径 |
|------|----------|
| HTTP N-API 入口 | `frameworks/js/napi/http/http_module/src/http_module.cpp` |
| HTTP 执行 | `frameworks/js/napi/http/http_exec/src/http_exec.cpp` |
| HTTP Native | `frameworks/native/http/http_client/http_client.cpp` |
| 权限检查 | `utils/common_utils/src/netstack_common_utils.cpp:145-186` |
| 证书验证 | `frameworks/native/net_ssl/net_ssl_verify_cert.cpp` |
| TLS 上下文 | `frameworks/native/tls_socket/src/tls_context.cpp` |
| Socket N-API | `frameworks/js/napi/socket/socket_module/src/socket_module.cpp` |
| WebSocket N-API | `frameworks/js/napi/websocket/websocket_module/src/websocket_module.cpp` |
| EventManager | `utils/napi_utils/src/event_manager.cpp` |

---

## 相关文档
- [Architecture](01_Architecture.md)
- [N-API Reference](02_N-API_Reference.md)
- [Native API](03_Inner_API.md)
- [Build Targets](04_Build_Targets.md)
- [Security Review](05_Security_Review.md)
