# OpenHarmony NetStack N-API 接口参考

## 1. HTTP 模块 (@ohos.net.http)

### 1.1 模块概述

| 属性 | 值 |
|------|-----|
| **命名空间** | `ohos.net.http` |
| **N-API 注册位置** | `frameworks/js/napi/http/http_module/src/http_module.cpp:540-552` |
| **模块名** | `"net.http"` |
| **初始化函数** | `HttpModuleExports::InitHttpModule` |

### 1.2 API 清单

#### 函数

| JS API | 类型 | 参数 | 返回值 | 实现位置 |
|--------|------|------|--------|----------|
| `createHttp()` | 函数 | 无 | `HttpRequest` | `http_module.cpp:InitHttpModule` |

#### 接口定义

```typescript
// 核心接口
interface HttpRequest {
  request(url: string, options?: HttpRequestOptions): Promise<HttpResponse>;
  request(url: string, callback: AsyncCallback<HttpResponse>): void;
  request(url: string, options: HttpRequestOptions, callback: AsyncCallback<HttpResponse>): void;
  destroy(): void;
  on(event: string, callback: Callback<unknown>): void;
  off(event: string, callback?: Callback<unknown>): void;
}

interface HttpRequestOptions {
  method?: RequestMethod;
  extraData?: string | Object | ArrayBuffer;
  expectDataType?: HttpDataType;
  usingCache?: boolean;
  priority?: number;
  header?: Object;
  readTimeout?: number;
  connectTimeout?: number;
  usingProtocol?: HttpProtocol;
  usingProxy?: boolean | HttpProxy;
  caPath?: string;
  clientCert?: ClientCert;
  dnsOverHttps?: string;
  dnsServers?: Array<string>;
  // ... 更多选项
}

interface HttpResponse {
  result: string | Object | ArrayBuffer;
  responseCode: ResponseCode | number;
  header: Object;
  cookies?: string;
}
```

### 1.3 错误码

| 错误码 | 说明 | 位置 |
|--------|------|------|
| 201 | Permission denied | `base_context.h:34` |
| 401 | Parameter error | - |
| 2300001 | Unsupported protocol | `constant.h` |
| 2300003 | Invalid URL format | `http_utils.cpp` |
| ... | 其他 cURL 错误 | - |

### 1.4 权限要求

- **必要权限**: `ohos.permission.INTERNET`
- **校验位置**: `CommonUtils::HasInternetPermission()`
- **错误码**: 201

---

## 2. Socket 模块 (@ohos.net.socket)

### 2.1 模块概述

| 属性 | 值 |
|------|-----|
| **命名空间** | `ohos.net.socket` |
| **N-API 注册位置** | `frameworks/js/napi/socket/socket_module/src/socket_module.cpp:1190-1204` |
| **模块名** | `"net.socket"` |
| **初始化函数** | `SocketModuleExports::InitSocketModule` |

### 2.2 API 清单

#### 工厂函数

| JS API | 类型 | 参数 | 返回值 | 说明 |
|--------|------|------|--------|------|
| `constructTCPSocketInstance()` | 函数 | 无 | `TCPSocket` | 创建 TCP Socket |
| `constructUDPSocketInstance()` | 函数 | 无 | `UDPSocket` | 创建 UDP Socket |
| `constructTLSSocketInstance()` | 函数 | 无 | `TLSSocket` | 创建 TLS Socket |
| `constructMulticastSocketInstance()` | 函数 | 无 | `MulticastSocket` | 创建组播 Socket |
| `constructLocalSocketInstance()` | 函数 | 无 | `LocalSocket` | 创建 Unix Domain Socket |
| `constructTCPSocketServerInstance()` | 函数 | 无 | `TCPSocketServer` | 创建 TCP 服务端 |
| `constructTLSSocketServerInstance()` | 函数 | 无 | `TLSSocketServer` | 创建 TLS 服务端 |

#### TCPSocket 接口

```typescript
interface TCPSocket {
  bind(address: NetAddress): Promise<void>;
  connect(options: TCPConnectOptions): Promise<void>;
  send(options: TCPSendOptions): Promise<void>;
  close(): Promise<void>;
  getRemoteAddress(): Promise<NetAddress>;
  getState(): Promise<SocketStateBase>;
  setExtraOptions(options: TCPExtraOptions): Promise<void>;
  on(event: string, callback: Callback<unknown>): void;
  off(event: string, callback?: Callback<unknown>): void;
}
```

#### UDPSocket 接口

```typescript
interface UDPSocket {
  bind(address: NetAddress): Promise<void>;
  send(options: UDPSendOptions): Promise<void>;
  close(): Promise<void>;
  getState(): Promise<SocketStateBase>;
  setExtraOptions(options: UDPExtraOptions): Promise<void>;
  on(event: 'message' | 'listening' | 'close' | 'error', callback: Callback<unknown>): void;
  off(event: string, callback?: Callback<unknown>): void;
}
```

#### TLSSocket 接口

```typescript
interface TLSSocket {
  connect(options: TLSConnectOptions): Promise<void>;
  send(data: string): Promise<void>;
  close(): Promise<void>;
  getCertificate(): Promise<X509CertRawData>;
  getRemoteCertificate(): Promise<X509CertRawData>;
  getProtocol(): Promise<string>;
  getCipherSuite(): Promise<Array<string>>;
  on(event: string, callback: Callback<unknown>): void;
  off(event: string, callback?: Callback<unknown>): void;
}
```

### 2.3 TLS 子模块初始化

TLS 模块**不是独立 N-API**，而是通过 Socket 模块初始化：

```cpp
// socket_module.cpp:510-530
static napi_value InitSocketModule(napi_env env, napi_value exports) {
    // 初始化 TLS Socket
    TlsSocket::TLSSocketModuleExports::InitTLSSocketModule(env, exports);
    // 初始化 TLS Server
    TlsSocketServer::TLSSocketServerModuleExports::InitTLSSocketServerModule(env, exports);
    // ...
}
```

---

## 3. WebSocket 模块 (@ohos.net.webSocket)

### 3.1 模块概述

| 属性 | 值 |
|------|-----|
| **命名空间** | `ohos.net.webSocket` |
| **N-API 注册位置** | `frameworks/js/napi/websocket/websocket_module/src/websocket_module.cpp:215-227` |
| **模块名** | `"net.webSocket"` |
| **初始化函数** | `WebSocketModule::InitWebSocketModule` |

### 3.2 API 清单

| JS API | 类型 | 参数 | 返回值 | 说明 |
|--------|------|------|--------|------|
| `createWebSocket()` | 函数 | 无 | `WebSocket` | 创建 WebSocket |
| `WebSocket.connect()` | 方法 | url, options | Promise | 建立连接 |
| `WebSocket.send()` | 方法 | data | Promise | 发送消息 |
| `WebSocket.close()` | 方法 | code?, reason? | Promise | 关闭连接 |
| `WebSocket.on()` | 方法 | event, callback | void | 事件监听 |
| `WebSocket.off()` | 方法 | event, callback? | void | 取消监听 |

#### WebSocket 接口

```typescript
interface WebSocket {
  connect(url: string, options?: WebSocketRequestOptions): Promise<void>;
  send(data: string | ArrayBuffer): Promise<void>;
  close(code?: number, reason?: string): Promise<void>;
  on(event: 'open' | 'message' | 'close' | 'error' | 'pong', callback: Callback<unknown>): void;
  off(event: string, callback?: Callback<unknown>): void;
}

interface WebSocketRequestOptions {
  header?: Object;
  caPath?: string;
  clientCert?: ClientCert;
  proxy?: ProxyConfiguration;
  skipServerCertVerification?: boolean;
  pingInterval?: number;
  pongTimeout?: number;
}
```

### 3.3 错误码

| 错误码 | 说明 | 位置 |
|--------|------|------|
| 201 | Permission denied | `constant.h:42` |
| WEBSOCKET_ERROR_PERMISSION_DENIED | 无权限 | `websocket_client.cpp` |
| WEBSOCKET_ERROR_DISALLOW_HOST | 原子服务主机名不允许 | `websocket_client.cpp` |

---

## 4. 网络安全模块 (@ohos.net.networkSecurity)

### 4.1 模块概述

| 属性 | 值 |
|------|-----|
| **命名空间** | `ohos.net.networkSecurity` |
| **N-API 注册位置** | `frameworks/js/napi/net_ssl/net_ssl_module/src/net_ssl_module.cpp:119-131` |
| **模块名** | `"net.networkSecurity"` |
| **初始化函数** | `NetSslModuleExports::InitNetSslModule` |

### 4.2 API 清单

| JS API | 类型 | 参数 | 返回值 | 说明 |
|--------|------|------|--------|------|
| `certVerification()` | 函数 | cert, CA | Promise<number> | 证书验证 |
| `certVerificationSync()` | 函数 | cert, CA | number | 同步证书验证 |
| `isCleartextPermitted()` | 函数 | 无 | boolean | 是否允许明文 |
| `isCleartextPermittedByHostName()` | 函数 | hostname | boolean | 按主机名检查 |

---

## 5. Fetch 模块 (@system.fetch)

### 5.1 模块概述

| 属性 | 值 |
|------|-----|
| **命名空间** | `@system.fetch` |
| **N-API 注册位置** | `frameworks/js/napi/fetch/fetch_module/src/fetch_module.cpp:42` |
| **注册宏** | `NAPI_MODULE(fetch, FetchModule::InitFetchModule)` |

### 5.2 API 清单

```typescript
interface FetchResponse {
  headers: Object;
  url: string;
  status: number;
  statusText: string;
  data: string | Object | ArrayBuffer;
}

interface FetchRequest {
  url: string;
  method?: string;
  data?: string | Object;
  headers?: Object;
}

declare class Fetch {
  fetch(request: FetchRequest): Promise<FetchResponse>;
}
```

---

## 6. N-API 工具函数参考

### 6.1 核心工具位置

文件: `utils/napi_utils/src/napi_utils.cpp`

### 6.2 工具函数清单

| 函数 | 行号 | 对应 N-API | 用途 |
|------|------|------------|------|
| `DefineProperties()` | 528 | `napi_define_properties` | 定义对象属性 |
| `CreateFunction()` | 459 | `napi_create_function` | 创建 JS 函数 |
| `SetNamedProperty()` | 117 | `napi_set_named_property` | 设置命名属性 |
| `GetValueType()` | 76 | `napi_typeof` | 获取值类型 |
| `SetErrorMessage()` | 143 | - | 设置错误消息 |
| `ConvertJsValueToString()` | 175 | - | JS 值转字符串 |

### 6.3 模块模板函数

文件: `utils/napi_utils/src/module_template.cpp`

| 函数 | 行号 | 用途 |
|------|------|------|
| `DefineClass()` | 325 | 定义 JS 类 |
| `NewInstanceWithSharedManager()` | 422 | 创建共享管理器实例 |
| `OnSharedManager()` | 494 | 注册事件监听 |
| `OffSharedManager()` | 518 | 注销事件监听 |

---

## 7. 上下文与异步工作

### 7.1 BaseContext

```cpp
// utils/napi_utils/include/base_context.h
class BaseContext {
public:
    void SetPermissionDenied(bool needThrowException);
    [[nodiscard]] bool IsPermissionDenied() const;
    void SetParseOK(bool parseResult);
    [[nodiscard]] bool IsParseOK() const;
    void SetNoAllowedHost(bool noAllowedHost);
    [[nodiscard]] bool IsNoAllowedHost() const;
    
private:
    bool permissionDenied_;     // 权限检查结果
    bool parseOK_;              // 解析结果
    bool noAllowedHost_;         // 原子服务主机名检查
};
```

### 7.2 异步工作模式

```cpp
// utils/napi_utils/include/base_async_work.h
class BaseAsyncWork {
public:
    static void ExecuteCallback(napi_env env, void *data);
    static void CallbackOfPromise(napi_env env, napi_status status, void *data);
};
```

---

## 8. 权限校验集成

### 8.1 权限检查入口

```cpp
// utils/common_utils/include/netstack_common_utils.h
bool HasInternetPermission();
```

### 8.2 上下文权限状态

```cpp
// utils/napi_utils/src/base_context.cpp:247-254
void BaseContext::SetPermissionDenied(bool permissionDenied) {
    permissionDenied_ = permissionDenied;
}

bool BaseContext::IsPermissionDenied() const {
    return permissionDenied_;
}
```

---

## 相关文档
- [Architecture](01_Architecture.md)
- [Native API](03_Inner_API.md)
- [Build Targets](04_Build_Targets.md)
- [Security Review](05_Security_Review.md)
