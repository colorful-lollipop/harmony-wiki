# 关键调用链

本文档描述 `netmanager_cangjie_wrapper` 项目中的关键调用链，包括从仓颉 API 到 FFI 再到底层 C 接口的完整调用路径。

## 调用链概览

```
kit/NetworkKit (Kit 层)
    │
    ├──► ohos.net.connection (连接模块)
    │       ├──► CJ_CreateNetConnection() ──────────────► netmanager_base
    │       ├──► CJ_GetDefaultNet() ────────────────────► netmanager_base
    │       ├──► CJ_GetNetCapabilities() ─────────────────► netmanager_base
    │       ├──► CJ_GetConnectionProperties() ───────────► netmanager_base
    │       └──► CJ_OnNetAvailable/Lost() ──────────────► netmanager_base
    │
    └──► ohos.net.http (HTTP 模块)
            ├──► CJ_CreateHttp() ────────────────────────► netstack
            ├──► CJ_SendRequest() ──────────────────────► netstack
            ├──► CJ_OnHeadersReceive() ─────────────────► netstack
            ├──► CJ_OnDataReceive() ────────────────────► netstack
            └──► CJ_DestroyRequest() ───────────────────► netstack
```

---

## HTTP 请求调用链

### 1. 创建 HTTP 请求任务

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant Http as HttpRequest
    participant FFI as http_ffi.cj
    participant C as netstack (C接口)

    App->>Http: createHttp()
    Http->>FFI: CJ_CreateHttp()
    FFI->>C: CJ_CreateHttp()
    C-->>FFI: requestId (Int64)
    FFI-->>Http: requestId
    Http-->>App: HttpRequest 实例
```

**证据来源**：`ohos/net/http/http.cj:34-41`

```cangjie
public func createHttp(): HttpRequest {
    let id = unsafe { CJ_CreateHttp() }
    HttpRequest(id)
}
```

---

### 2. 发起 HTTP 请求

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant Http as HttpRequest
    participant Parse as 参数解析
    participant FFI as http_ffi.cj
    participant C as netstack (C接口)
    participant CB as 回调处理

    App->>Http: request(url, options, callback)
    Http->>Parse: parseParam(url, options)
    Parse-->>Http: (c_url, optPtr)
    
    Http->>FFI: CJ_SendRequest(id, url, optPtr, false, callbackId)
    FFI->>C: CJ_SendRequest(...)
    C-->>FFI: RetDataCString
    FFI-->>Http: retCode, retData
    
    alt 成功
        C->>CB: callback(CHttpResponse)
        CB-->>App: callback(None, HttpResponse)
    else 失败
        CB-->>App: callback(BusinessException, None)
    end
    
    Http->>Parse: freeOptions(optPtr)
```

**证据来源**：`ohos/net/http/http.cj:233-268`

```cangjie
public func request(url: String, options: HttpRequestOptions, 
                    callback: AsyncCallback<HttpResponse>): Unit {
    let (c_url, optPtr) = unsafe { parseParam(url, options) }
    
    let wrapper = { resp: CHttpResponse =>
        if (resp.errCode != SUCCESS_CODE) {
            callback(error, None)
        } else {
            callback(None, HttpResponse(resp))
        }
        resp.free()
    }
    
    let ret = unsafe { CJ_SendRequest(id, c_url, optPtr, false, registerCall.getID()) }
    // ...
}
```

---

### 3. 流式 HTTP 请求

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant Http as HttpRequest
    participant FFI as http_ffi.cj
    participant C as netstack
    participant Headers as on_headersReceive
    participant Data as on_dataReceive

    App->>Http: requestInStream(url, options, callback)
    Http->>FFI: CJ_SendRequest(..., isInStream: true, ...)
    
    FFI->>C: CJ_SendRequest(..., true, ...)
    C-->>FFI: responseCode
    
    Note over C,App: 数据流传输阶段
    C->>Headers: CJ_OnHeadersReceive(callback)
    Headers-->>App: callback(headers)
    
    loop DataReceive
        C->>Data: CJ_OnDataReceive(callback)
        Data-->>App: callback(data)
    end
    
    C->>App: on_dataEnd(callback)
```

**证据来源**：`ohos/net/http/http.cj:394-432`

---

### 4. HTTP 事件监听注册

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant Http as HttpRequest
    participant Wrap as 回调包装
    participant FFI as http_ffi.cj
    participant C as netstack

    App->>Http: on(HeadersReceive, callback)
    Http->>Wrap: 创建 Callback1Param
    
    alt 首次注册
        Http->>FFI: CJ_OnHeadersReceive(id, false, callbackId)
        FFI->>C: 注册回调
        C-->>FFI: 注册成功
    else 重复注册
        Http->>Http: 检查已注册
        Http-->>App: 日志: 已注册
    end
    
    Http->>Http: 添加到 callBackMap
```

**证据来源**：`ohos/net/http/http.cj:545-558`

```cangjie
public func on(event: HttpRequestEvent, 
               callback: Callback1Argument<HashMap<String, String>>): Unit {
    if (event != HttpRequestEvent.HeadersReceive) {
        throw BusinessException(2100001, "Invalid parameter value.")
    }
    commonSubscribe1Arg(event, callback) {
        cArrString: CArrString => cArrString2Map(cArrString)
    }
}
```

---

## 网络连接调用链

### 1. 创建网络连接

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant NC as NetConnection
    participant Spec as CNetSpecifier
    participant FFI as connection_ffi.cj
    participant C as netmanager_base (C接口)

    App->>NC: createNetConnection(netSpecifier, timeout)
    NC->>Spec: CNetSpecifier(netSpecifier)
    
    NC->>FFI: CJ_CreateNetConnection(specifier, timeout)
    FFI->>C: CJ_CreateNetConnection(...)
    C-->>FFI: connectionId (Int64)
    FFI-->>NC: connectionId
    NC-->>App: NetConnection 实例
```

**证据来源**：`ohos/net/connection/net_connection.cj:40-52`

```cangjie
public func createNetConnection(netSpecifier!: ?NetSpecifier = None, 
                                 timeout!: UInt32 = 0): NetConnection {
    let specifier = match (netSpecifier) {
        case Some(v) => CNetSpecifier(v)
        case None => CNetSpecifier()
    }
    let id = unsafe { CJ_CreateNetConnection(specifier, timeout) }
    if (id < 0) {
        throw BusinessException(2100003, "System internal error.")
    }
    NetConnection(id)
}
```

---

### 2. 获取默认网络

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant Func as getDefaultNet()
    participant FFI as connection_ffi.cj
    participant C as netmanager_base

    App->>Func: getDefaultNet()
    Func->>FFI: CJ_GetDefaultNet(&netId)
    FFI->>C: CJ_GetDefaultNet(&netId)
    
    C-->>FFI: retCode, netId
    FFI-->>Func: retCode
    
    alt SUCCESS_CODE
        Func-->>App: NetHandle(netId)
    else 失败
        Func-->>App: BusinessException
    end
```

**证据来源**：`ohos/net/connection/net_connection.cj:68-78`

---

### 3. 解析

``` DNSmermaid
sequenceDiagram
    participant App as 仓颉应用
    participant NH as NetHandle
    participant Conv as Host转换
    participant FFI as connection_ffi.cj
    participant C as netmanager_base

    App->>NH: getAddressesByName(host)
    NH->>Conv: LibC.mallocCString(host)
    Conv-->>NH: cHost
    
    NH->>FFI: CJ_GetAddressesByName(netId, cHost)
    FFI->>C: CJ_GetAddressesByName(...)
    C-->>FFI: RetNetAddressArr
    FFI-->>NH: (code, size, data)
    
    NH->>Conv: 转换 C数组->Array<NetAddress>
    NH->>Conv: LibC.free(cHost)
    NH->>Conv: ret.free()
    
    NH-->>App: Array<NetAddress>
```

**证据来源**：`ohos/net/connection/net_connection.cj:632-658`

```cangjie
public func getAddressesByName(host: String): Array<NetAddress> {
    if (host.isEmpty() || host.startsWith("\0")) {
        throw BusinessException(2100001, "...")
    }
    let cHost = unsafe { LibC.mallocCString(host) }
    let ret: RetNetAddressArr = unsafe { CJ_GetAddressesByName(netId, cHost) }
    unsafe { LibC.free(cHost) }
    
    let arr = Array<NetAddress>(
        size,
        { i => NetAddress(unsafe { ptr.read(i) }) }
    )
    ret.free()
    arr
}
```

---

### 4. 网络事件监听

```mermaid
sequenceDiagram
    participant App as 仓颉应用
    participant NC as NetConnection
    participant Wrap as 回调包装
    participant FFI as connection_ffi.cj
    participant C as netmanager_base

    App->>NC: on(NetAvailable, callback)
    NC->>Wrap: Callback1Param<Int32, Unit>
    
    NC->>FFI: CJ_OnNetAvailable(connId, callbackId)
    FFI->>C: CJ_OnNetAvailable(...)
    C-->>FFI: 注册成功
    
    Note over C,App: 等待网络事件
    
    C->>Wrap: invoke(netId)
    Wrap->>App: callback(NetHandle(netId))
```

**证据来源**：`ohos/net/connection/net_connection.cj:416-430`

```cangjie
public func on(event: NetConnectionEvent, 
               callback: Callback1Argument<NetHandle>): Unit {
    let wrapper = { netId: Int32 =>
        let handle = NetHandle(netId)
        callback.invoke(None, handle)
    }
    let registerCall = Callback1Param<Int32, Unit>(wrapper)
    unsafe {
        match (event) {
            case NetConnectionEvent.NetAvailable => 
                CJ_OnNetAvailable(getID(), registerCall.getID())
            case NetConnectionEvent.NetLost => 
                CJ_OnNetLost(getID(), registerCall.getID())
            case _ => throw BusinessException(2100001, "...")
        }
    }
}
```

---

## 数据类型转换调用链

### 1. 字符串转换

```mermaid
sequenceDiagram
    participant CJ as 仓颉 String
    participant Mem as LibC.mallocCString
    participant C as CString
    participant Free as LibC.free

    CJ->>Mem: mallocCString(str)
    Mem->>C: malloc + strcpy
    C-->>Mem: CString 指针
    Mem-->>CJ: CString
    
    Note right of C: 传递给 FFI/C 接口
    
    CJ->>Free: free(cstr)
    Free-->>C: 释放内存
```

---

### 2. 数组转换

```mermaid
sequenceDiagram
    participant CJ as Array<T>
    participant Conv as cjArr2CArr
    participant C as CPointer<CT>
    participant Use as FFI 使用
    participant Free as LibC.free

    CJ->>Conv: cjArr2CArr(arr, toC, freeC)
    Conv->>Conv: malloc CArray
    Conv->>Conv: 复制元素
    Conv-->>CJ: CPointer<CT>
    
    Use->>C: 传递给 C 接口
    C-->>Use: 返回数据
    
    Use->>Free: free CArray
    Free-->>C: 释放内存
```

---

## 内存管理调用链

### 资源释放模式

```mermaid
flowchart TD
    A[创建资源] --> B{成功?}
    B -->|是| C[使用资源]
    B -->|否| D[抛出异常]
    C --> E{操作完成?}
    E -->|成功| F[返回结果]
    E -->|失败| G[释放资源]
    G --> H[抛出异常]
    
    F --> I{需要清理?}
    I -->|是| J[free 方法]
    J --> K[LibC.free]
    K --> L[释放内存]
```

**证据来源**：`ohos/net/http/http_ffi.cj:167-186`

```cangjie
func free(): Unit {
    unsafe {
        LibC.free(this.method)
        LibC.free(this.caPath)
        LibC.free(this.dnsOverHttps)
        LibC.free<UInt8>(this.extraData.head)
        this.header.free()
        this.dnsServers.free()
        // ... 完整释放链
    }
}
```

---

## 异常传播调用链

```mermaid
sequenceDiagram
    participant CJ as 仓颉层
    participant FFI as FFI 层
    participant C as 底层 C
    participant Err as 错误处理

    C->>FFI: 返回错误码
    FFI->>Err: getErrorCode(retCode)
    Err-->>FFI: 错误码映射
    
    alt 权限错误 201
        Err-->>CJ: BusinessException(201, msg)
    else 参数错误 2100001
        Err-->>CJ: BusinessException(2100001, msg)
    else 服务错误 2100002
        Err-->>CJ: BusinessException(2100002, msg)
    else 系统错误 2100003
        Err-->>CJ: BusinessException(2100003, msg)
    end
    
    CJ->>App: throw BusinessException
```

**证据来源**：`ohos/net/connection/net_connection.cj:73-75`

```cangjie
if (retCode != SUCCESS_CODE) {
    let errCode = getErrorCode(retCode)
    throw BusinessException(errCode, "getDefaultNet failed: ${getErrorMsg(errCode)}")
}
```

---

## 相关文档

- [API 参考](03_API_Reference.md) - API 详细说明
- [FFI 接口](04_FFI_Interface.md) - FFI 函数定义
- [系统架构](02_Architecture.md) - 整体架构设计
- [安全评审](06_Security_Review.md) - 安全风险分析
