# OpenHarmony NetStack Native API 参考

## 1. HTTP Client Native API

### 1.1 HttpSession

**文件**: `interfaces/innerkits/http_client/include/http_client.h:41`

```cpp
namespace OHOS {
namespace NetStack {
namespace HttpClient {
class HttpSession {
public:
    // 获取单例
    static HttpSession &GetInstance();
    
    // 创建 HTTP 任务
    [[nodiscard]] std::shared_ptr<HttpClientTask> CreateTask(const HttpClientRequest &request);
    
    // 创建带文件路径的任务
    [[nodiscard]] std::shared_ptr<HttpClientTask> CreateTask(const HttpClientRequest &request, 
                                                              TaskType type,
                                                              const std::string &filePath);
};
} // namespace HttpClient
} // namespace NetStack
} // namespace OHOS
```

### 1.2 HttpClientTask

**文件**: `interfaces/innerkits/http_client/include/http_client_task.h:53`

```cpp
class HttpClientTask : public std::enable_shared_from_this<HttpClientTask> {
public:
    // 构造函数
    HttpClientTask(const HttpClientRequest &request);
    HttpClientTask(const HttpClientRequest &request, TaskType type, const std::string &filePath);
    
    // 生命周期
    bool Start();           // 启动请求
    void Cancel();         // 取消请求
    TaskStatus GetStatus(); // 获取状态
    TaskType GetType();    // 获取类型
    
    // 访问器
    unsigned int GetTaskId();
    const std::string &GetFilePath();
    HttpClientRequest &GetRequest();
    HttpClientResponse &GetResponse();
    HttpClientError &GetError();
    CURL *GetCurlHandle();
    
    // 回调设置
    void OnSuccess(std::function<void(HttpClientRequest&, HttpClientResponse&)> onSucceeded);
    void OnFail(std::function<void(HttpClientRequest&, HttpClientError&)> onFailed);
    void OnDataReceive(std::function<void(const char*, size_t)> onResponse);
};

enum TaskStatus {
    IDLE,       // 空闲
    RUNNING,    // 运行中
};

enum TaskType {
    DEFAULT,    // 默认请求
    UPLOAD,     // 上传任务
};
```

### 1.3 HttpClientRequest

**文件**: `interfaces/innerkits/http_client/include/http_client_request.h:112`

```cpp
class HttpClientRequest {
public:
    // URL 设置
    void SetUrl(const std::string &url);
    std::string GetUrl() const;
    
    // HTTP 方法
    void SetMethod(const std::string &method);
    std::string GetMethod() const;
    
    // 请求头
    void SetHeader(const std::string &key, const std::string &value);
    std::string GetHeader(const std::string &key) const;
    
    // 请求体
    void SetBody(const std::string &body);
    void SetBody(const char *data, size_t len);
    
    // 代理设置
    void SetProxy(const HttpProxy &proxy);
    HttpProxy GetProxy() const;
    
    // TLS 配置
    void SetTlsOption(const TlsOption &tlsOption);
    TlsOption GetTlsOption() const;
};

enum HttpProxyType {
    NONE,
    HTTP,
    SOCKS5,
};

struct HttpProxy {
    HttpProxyType type;
    std::string host;
    int port;
    std::string username;
    std::string password;
};
```

### 1.4 HttpClientResponse

**文件**: `interfaces/innerkits/http_client/include/http_client_response.h:103`

```cpp
class HttpClientResponse {
public:
    // 响应状态
    int32_t GetResponseCode() const;
    std::string GetReason() const;
    
    // 响应头
    std::string GetHeader(const std::string &key) const;
    std::map<std::string, std::string> GetAllHeaders() const;
    
    // 响应体
    std::string GetResult() const;
    const char *GetData() const;
    size_t GetDataSize() const;
    
    // 性能信息
    int64_t GetTotalBytes() const;
    int64_t GetReadBytes() const;
    int64_t GetConnectTime() const;
    int64_t GetPreTransferTime() const;
};

struct PerformanceInfo {
    int64_t totalBytes;      // 总字节数
    int64_t readBytes;       // 已读字节数
    int64_t connectTime;     // 连接时间
    int64_t preTransferTime; // 预传输时间
};
```

### 1.5 HttpClientError

**文件**: `interfaces/innerkits/http_client/include/http_client_error.h:68`

```cpp
class HttpClientError {
public:
    int32_t GetErrorCode() const;
    std::string GetErrorMessage() const;
};

enum HttpErrorCode {
    ERROR_NONE = 0,
    ERROR_INVALID_PARAM = 1,
    ERROR_PERMISSION = 201,
    // ... 其他错误码
};
```

---

## 2. TLS Socket Native API

### 2.1 TLSSocket

**文件**: `interfaces/innerkits/tls_socket/include/tls_socket.h:307`

```cpp
class TLSSocket : public std::enable_shared_from_this<TLSSocket> {
public:
    // 连接管理
    int32_t Bind(const Socket::NetAddress &address);
    int32_t Connect(const TLSConnectOptions &options, 
                    std::shared_ptr<EventManager> manager);
    int32_t Send(const std::string &data);
    int32_t Send(const std::vector<uint8_t> &data);
    int32_t Close();
    
    // 状态查询
    int32_t GetRemoteAddress(Socket::NetAddress &address);
    int32_t GetLocalAddress(Socket::NetAddress &address);
    int32_t GetSocketState(Socket::SocketStateBase &state);
    
    // TLS 信息
    int32_t GetCertificate(X509CertRawData &cert);
    int32_t GetRemoteCertificate(X509CertRawData &cert);
    int32_t GetProtocol(std::string &protocol);
    int32_t GetCipherSuite(std::vector<std::string> &suite);
    int32_t GetSignatureAlgorithms(std::vector<std::string> &algorithms);
    
    // 事件回调
    void SetOnMessageCallback(OnMessageCallback callback);
    void SetOnConnectCallback(OnConnectCallback callback);
    void SetOnCloseCallback(OnCloseCallback callback);
    void SetOnErrorCallback(OnErrorCallback callback);
};

// 回调类型定义
using OnMessageCallback = std::function<void(const std::string &data, 
                                             const Socket::SocketRemoteInfo &remoteInfo)>;
using OnConnectCallback = std::function<void(void)>;
using OnCloseCallback = std::function<void(void)>;
using OnErrorCallback = std::function<void(int32_t errorNumber, 
                                           const std::string &errorString)>;
```

### 2.2 TLSConnectOptions

**文件**: `interfaces/innerkits/tls_socket/include/tls_socket.h:219`

```cpp
class TLSConnectOptions {
public:
    void SetAddress(const Socket::NetAddress &address);
    void SetSecureOptions(const TLSSecureOptions &options);
    void SetALPNProtocols(const std::vector<std::string> &protocols);
};

class TLSSecureOptions {
public:
    void SetCaChain(const std::vector<std::string> &caChain);
    void SetCert(const std::string &cert);
    void SetKey(const std::string &key);
    void SetKeyPassword(const std::string &password);
    void SetProtocols(const std::vector<Protocol> &protocols);
    void SetSignatureAlgorithms(const std::string &algorithms);
    void SetCipherSuite(const std::string &cipherSuite);
    void SetVerifyMode(VerifyMode mode);
};

enum Protocol {
    TLSv12,
    TLSv13,
};

enum VerifyMode {
    ONE_WAY_MODE,      // 单向认证
    TWO_WAY_MODE,      // 双向认证
};
```

---

## 3. WebSocket Native API

### 3.1 WebSocketClient

**文件**: `interfaces/innerkits/websocket_native/include/websocket_client_innerapi.h:61`

```cpp
class WebSocketClient : public std::enable_shared_from_this<WebSocketClient> {
public:
    // 连接管理
    int32_t Connect(const std::string &url, const OpenOptions &options);
    int32_t Send(const std::string &data);
    int32_t SendBinary(const uint8_t *data, size_t len);
    int32_t Close(int32_t code, const std::string &reason);
    
    // 状态查询
    int32_t GetState(SocketStateBase &state);
    
    // 事件回调
    void SetOnOpenCallback(OnOpenCallback callback);
    void SetOnMessageCallback(OnMessageCallback callback);
    void SetOnCloseCallback(OnCloseCallback callback);
    void SetOnErrorCallback(OnErrorCallback callback);
    void SetOnHeaderReceiveCallback(OnHeaderReceiveCallback callback);
};

// 回调类型
using OnOpenCallback = std::function<void(WebSocketClient*, OpenResult openResult)>;
using OnMessageCallback = std::function<void(WebSocketClient*, const std::string &data, size_t length)>;
using OnCloseCallback = std::function<void(WebSocketClient*, CloseResult closeResult)>;
using OnErrorCallback = std::function<void(WebSocketClient*, ErrorResult error)>;
using OnHeaderReceiveCallback = std::function<void(WebSocketClient*, 
                                                   const std::map<std::string, std::string> &headers)>;

struct OpenResult {
    int32_t errorCode;
    std::string errorMessage;
};

struct CloseResult {
    int32_t code;
    std::string reason;
};

struct ErrorResult {
    int32_t errorCode;
    std::string errorMessage;
};
```

### 3.2 WebSocketServer

**文件**: `interfaces/innerkits/websocket_native/include/websocket_server_innerapi.h:26`

```cpp
class WebSocketServer {
public:
    // 服务端管理
    int32_t Bind(const Socket::NetAddress &address);
    int32_t Listen();
    int32_t Stop();
    
    // 连接管理
    int32_t Send(SocketConnection connection, const std::string &data);
    int32_t Close(SocketConnection connection, int32_t code, const std::string &reason);
    
    // 状态查询
    int32_t GetState(SocketStateBase &state);
    
    // 事件回调
    void SetOnAcceptCallback(OnAcceptCallback callback);
    void SetOnMessageReceiveCallback(OnMessageReceiveCallback callback);
    void SetOnCloseCallback(OnCloseCallback callback);
    void SetOnErrorCallback(OnErrorCallback callback);
};
```

---

## 4. SSL/TLS Certificate API

### 4.1 TLSCertificate

**文件**: `frameworks/native/tls_socket/include/tls_certificate.h:30`

```cpp
class TLSCertificate {
public:
    enum EncodingFormat {
        PEM,
        DER,
        P12,
    };
    
    TLSCertificate(const std::string &data, EncodingFormat format, CertType certType);
    
    // 证书信息
    std::string GetSignatureAlgorithm() const;
    int64_t GetNotBefore() const;
    int64_t GetNotAfter() const;
    std::string GetSubject() const;
    std::string GetIssuer() const;
    std::string GetSerialNumber() const;
    
    // 导出
    std::string ToPEM() const;
    std::vector<uint8_t> ToDER() const;
    
    // 验证
    bool IsValid() const;
};
```

### 4.2 Certificate Verification

```cpp
// 系统 CA 路径
constexpr char SYSPRECAPATH[] = "/etc/security/certificates/user/";
// 用户安装 CA 路径
constexpr char USERINSTALLEDCAPATH[] = "/data/certificates/user_cacerts/";

// 证书验证
uint32_t VerifyCert(const CertBlob *cert);

// 获取用户 CA 路径
std::string GetUserInstalledCaPath();
```

---

## 5. Cache API

### 5.1 HttpCacheStrategy

**文件**: `interfaces/innerkits/http_client/cache/cache_strategy/include/http_cache_strategy.h:35`

```cpp
class HttpCacheStrategy {
public:
    enum CacheStatus {
        NONE,
        ONLY_CACHE,
        FIRST_CACHE_THEN_REQUEST,
        ONLY_REQUEST,
    };
    
    void SetCacheStatus(CacheStatus status);
    CacheStatus GetCacheStatus() const;
    
    void SetMaxCacheAge(int64_t maxAge);
    int64_t GetMaxCacheAge() const;
    
    void SetMaxCacheSize(int64_t maxSize);
    int64_t GetMaxCacheSize() const;
    
    // 缓存查询
    bool HasCachedResponse(const std::string &url) const;
    std::shared_ptr<HttpCacheResponse> GetCachedResponse(const std::string &url);
    
    // 缓存存储
    void SaveResponse(const std::string &url, const HttpCacheResponse &response);
    void Invalidate(const std::string &url);
    void Clear();
};
```

---

## 6. Common Utilities

### 6.1 Security Utils

```cpp
// 权限检查
bool HasInternetPermission();

// 原子服务检查
bool IsAtomicService(std::string &bundleName);
bool IsAllowedHostname(const std::string &bundleName, 
                      const std::string &domainType, 
                      const std::string &url);

// 证书相关
bool Sha256sum(unsigned char *buf, size_t buflen, std::string &digestStr);
bool IsCertPubKeyInPinned(const std::string &certPubKeyDigest, 
                         const std::string &pinnedPubkey);

// SHA256 格式
// "sha256//base64hash1;base64hash2;base64hash3"
```

---

## 相关文档
- [Architecture](01_Architecture.md)
- [N-API Reference](02_N-API_Reference.md)
- [Build Targets](04_Build_Targets.md)
- [Security Review](05_Security_Review.md)
