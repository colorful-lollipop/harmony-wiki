# 关键调用链

## 入口 → 核心逻辑

### 1. JS 异步认证调用

```mermaid
sequenceDiagram
    participant JS as JS Application
    participant JSI as JSI Runtime
    participant Kit as kit_device_attest
    participant Client as devattest_client
    participant Server as devattest_server
    participant Core as devattest_core
    participant Net as Network Layer
    participant Cloud as Attestation Server

    JS->>JSI: getAttestStatus(callback)
    JSI->>Kit: GetAttestResultInfoAsync()
    Kit->>Kit: ExecuteAsyncWork()
    Kit->>JsAsyncWork: DispatchAsyncWork()
    
    Note over JsAsyncWork: Async Work Queue
    
    JsAsyncWork->>Kit: ExecuteGetAttestResult()
    Kit->>Core: GetAttestStatus(AttestResultInfo*)
    
    Core->>Core: IsAuthStatusChg()
    Core->>Core: CheckExpireTime()
    
    alt Need Re-authentication
        Core->>Net: GetChallenge()
        Net->>Cloud: TLS Request
        Cloud-->>Net: Challenge + Time
        Net-->>Core: Challenge Response
        
        Core->>Core: GenAuthMsg()
        Core->>Net: SendAuthMsg()
        Net->>Cloud: TLS Request (AuthMsg)
        Cloud-->>Net: Auth Result
        Net-->>Core: ParseAuthResultResp()
        Core->>Core: FlushToken()
    end
    
    Core-->>Kit: AttestResultInfo
    Kit-->>JS: callback(err, result)
```

### 2. JS 同步认证调用

```mermaid
sequenceDiagram
    participant JS as JS Application
    participant JSI as JSI Runtime
    participant Kit as kit_device_attest
    participant Core as devattest_core

    JS->>JSI: getAttestStatusSync()
    JSI->>Kit: GetAttestResultInfoSync()
    
    Note over Kit: Synchronous Call<br/>~10ms
    
    Kit->>Core: GetAttestStatus(AttestResultInfo*)
    Core->>Core: Check Cache
    alt Cache Valid
        Core-->>Kit: Cached Result
    else Cache Invalid
        Core->>Core: Full Attestation
        Core-->>Kit: Fresh Result
    end
    
    Kit-->>JS: AttestResult Object
```

---

## 内部调用链

### 3. 认证服务主流程

```mermaid
flowchart TD
    A[StartDevAttestTask] --> B[ProcAttest]
    B --> C[ProcAttestImpl]
    C --> D[InitSystemData]
    D --> E{Auth Status Changed?}
    E -->|Yes| F[AttestStartup]
    E -->|No| G[CheckExpireTime]
    
    F --> H[ResetAttestDevice]
    H --> I[AuthAttestDevice]
    I --> J[GetChallenge]
    J --> K[GenAuthMsg]
    K --> L[SendAuthMsg]
    L --> M[ParseAuthResultResp]
    M --> N[ActiveToken]
    
    G --> O{Expired?}
    O -->|Yes| F
    O -->|No| P[Return Cached]
    
    N --> O
```

### 4. Token 生命周期

```mermaid
flowchart TD
    A[Token Storage] --> B[ReadToken]
    B --> C[DecryptAesCbc]
    C --> D[GetAesKey]
    D --> E[HKDF manuKey + salt]
    
    F[Token Usage] --> G[GetTokenValueAndId]
    G --> H[EncryptHmac]
    H --> I[HMAC-SHA256 challenge]
    I --> J[Send to Server]
    
    K[Token Update] --> L[FlushToken]
    L --> M[AES-128-CBC Encrypt]
    M --> N[WriteToken]
    N --> A
```

---

## 网络调用链

### 5. CoAP/TLS 通信

```mermaid
flowchart TD
    A[Network Request] --> B[AttestBuildMessage]
    B --> C[CoAP Package]
    C --> D[TLSSession Write]
    D --> E[mbedtls_ssl_write]
    E --> F[TLS Encryption]
    F --> G[TCP Send]
    
    H[Network Response] --> I[TCP Recv]
    I --> J[TLS Decryption]
    J --> K[mbedtls_ssl_read]
    K --> L[TLSSession Read]
    L --> M[CoAP Unpack]
    M --> N[Parse JSON]
    N --> O[Return Result]
```

---

## 关键文件 → 函数映射

### 入口函数

| 功能 | 入口文件 | 主函数 |
|------|---------|--------|
| JS 异步调用 | `native_device_attest.cpp` | `GetAttestResultInfoAsync()` |
| JS 同步调用 | `native_device_attest.cpp` | `GetAttestResultInfoSync()` |
| Inner API | `devattest_interface.h` | `StartDevAttestTask()` |
| Inner API | `devattest_interface.h` | `GetAttestStatus()` |

### 核心服务

| 模块 | 文件 | 关键函数 |
|------|------|---------|
| 认证主流程 | `attest_service.c` | `ProcAttest()` |
| 认证状态 | `attest_service_auth.c` | `IsAuthStatusChg()` |
| 认证消息 | `attest_service_auth.c` | `GenAuthMsg()` |
| Token 管理 | `attest_security_token.c` | `GetTokenValueAndId()` |

### 网络模块

| 模块 | 文件 | 关键函数 |
|------|------|---------|
| 连接 | `attest_network.c` | `D2CConnect()` |
| 消息 | `attest_network.c` | `SendAttestMsg()` |
| TLS | `attest_channel.c` | `LazyVerifyCert()` |

### 安全模块

| 模块 | 文件 | 关键函数 |
|------|------|---------|
| AES 加密 | `attest_security.c` | `EncryptAesCbc()` |
| AES 解密 | `attest_security.c` | `DecryptAesCbc()` |
| 密钥派生 | `attest_security.c` | `GetAesKey()` |
| HMAC | `attest_security_token.c` | `EncryptHmac()` |

---

## 线程/任务关系

```mermaid
flowchart LR
    subgraph JS Thread
        JS[JSI Runtime]
    end
    
    subgraph Async Work Thread
        AW[Async Work Queue]
    end
    
    subgraph Service Thread
        ST[Service Main Loop]
    end
    
    subgraph Timer Thread
        TM[Timer Callbacks]
    end
    
    JS -->|Async Call| AW
    AW -->|Execute| ST
    TM -->|Trigger| ST
```

| 线程 | 职责 | 关键同步 |
|------|------|---------|
| JS 线程 | JSI 调用 | - |
| Async Work | 异步任务 | pthread mutex |
| Service 线程 | SA 消息处理 | g_mtxAttest |
| Timer 线程 | 定时回调 | - |
