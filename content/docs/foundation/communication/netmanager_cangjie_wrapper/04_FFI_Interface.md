# FFI 接口与数据类型

## 概述

本项目通过仓颉 FFI（Foreign Function Interface）机制调用底层 C 接口。FFI 层定义了所有外部函数声明和 C 结构体映射。

**证据来源**：
- `ohos/net/http/http_ffi.cj`
- `ohos/net/connection/connection_ffi.cj`

---

## HTTP 模块 FFI 接口

### 文件位置

`ohos/net/http/http_ffi.cj:25-63`

### 外部函数声明

```cangjie
foreign {
    // 缓存管理
    func CJ_CreateHttpResponseCache(cacheSize: UInt32): Int32
    func CJ_HttpResponseCacheFlush(): Int32
    func CJ_HttpResponseCacheDelete(): Int32
    
    // 请求生命周期
    func CJ_CreateHttp(): Int64
    func CJ_DestroyRequest(id: Int64): Unit
    func CJ_SendRequest(id: Int64, url: CString, opt: CPointer<CHttpRequestOptions>, isInStream: Bool, callback: Int64): RetDataCString
    
    // 事件监听
    func CJ_OnHeadersReceive(id: Int64, once: Bool, callback: Int64): Unit
    func CJ_OffHeadersReceive(id: Int64): Unit
    func CJ_OnDataReceive(id: Int64, callback: Int64): Unit
    func CJ_OffDataReceive(id: Int64): Unit
    func CJ_OnDataEnd(id: Int64, callback: Int64): Unit
    func CJ_OffDataEnd(id: Int64): Unit
    func CJ_OnDataReceiveProgress(id: Int64, callback: Int64): Unit
    func CJ_OffDataReceiveProgress(id: Int64): Unit
    func CJ_OnDataSendProgress(id: Int64, callback: Int64): Unit
    func CJ_OffDataSendProgress(id: Int64): Unit
    
    // 内存管理
    func FFiOHOSNetHttpFreeCString(p: CString): Unit
    func FFiOHOSNetHttpFreeCArrString(arr: CArrString): Unit
    func FFiOHOSNetHttpFreeCArrUI8(arr: CArrUI8): Unit
}
```

### C 结构体定义

#### CHttpRequestOptions

HTTP 请求选项结构体。

```cangjie
@C
struct CHttpRequestOptions {
    var method: CString
    var extraData: CArrUI8
    var expectDataType: Int32
    var usingCache: Bool
    var priority: UInt32
    var header: CArrString
    var readTimeout: UInt32
    var connectTimeout: UInt32
    var usingProtocol: Int32
    var usingDefaultProxy: Bool
    var usingProxy: CPointer<CHttpProxy>
    var caPath: CString
    var resumeFrom: Int64
    var resumeTo: Int64
    var clientCert: CPointer<CClientCert>
    var dnsOverHttps: CString
    var dnsServers: CArrString
    var maxLimit: UInt32
    var multiFormDataList: CArrMultiFormData
}
```

**证据来源**：`ohos/net/http/http_ffi.cj:85-116`

#### CHttpResponse

HTTP 响应结构体。

```cangjie
@C
struct CHttpResponse {
    let errCode: Int32
    let errMsg: CString
    let result: CArrUI8
    let resultType: Int32
    let responseCode: UInt32
    let header: CArrString
    let cookies: CString
    let setCookie: CArrString
    let performanceTiming: CPerformanceTiming
}
```

**证据来源**：`ohos/net/http/http_ffi.cj:266-288`

#### CPerformanceTiming

性能计时结构体。

```cangjie
@C
struct CPerformanceTiming {
    let dnsTiming: Float64          // DNS 解析耗时 (ms)
    let tcpTiming: Float64           // TCP 连接耗时 (ms)
    let tlsTiming: Float64           // TLS 连接耗时 (ms)
    let firstSendTiming: Float64     // 首次发送耗时 (ms)
    let firstReceiveTiming: Float64  // 首次接收耗时 (ms)
    let totalFinishTiming: Float64  // 总完成耗时 (ms)
    let redirectTiming: Float64      // 重定向耗时 (ms)
    let responseHeaderTiming: Float64 // 响应头耗时 (ms)
    let responseBodyTiming: Float64  // 响应体耗时 (ms)
    let totalTiming: Float64         // 总耗时 (ms)
}
```

**证据来源**：`ohos/net/http/http_ffi.cj:291-323`

#### 辅助结构体

| 结构体 | 描述 |
|-------|------|
| `CDataReceiveProgressInfo` | 数据接收进度信息 (`receiveSize`, `totalSize`) |
| `CDataSendProgressInfo` | 数据发送进度信息 (`sendSize`, `totalSize`) |
| `CClientCert` | 客户端证书配置 |
| `CMultiFormData` | Multipart 表单数据 |
| `CArrMultiFormData` | Multipart 数组 |

**证据来源**：`ohos/net/http/http_ffi.cj:325-429`

---

## 连接模块 FFI 接口

### 文件位置

`ohos/net/connection/connection_ffi.cj:23-75`

### 外部函数声明

```cangjie
foreign {
    // 连接生命周期
    func CJ_CreateNetConnection(netSpecifier: CNetSpecifier, timeout: UInt32): Int64
    func CJ_NetConnectionRegister(id: Int64): Int32
    func CJ_NetConnectionUnRegister(id: Int64): Int32
    
    // 网络查询
    func CJ_GetDefaultNet(netId: CPointer<Int32>): Int32
    func CJ_GetAppNet(netId: CPointer<Int32>): Int32
    func CJ_SetAppNet(netId: Int32): Int32
    func CJ_GetAllNets(): RetDataCArrI32
    func CJ_GetNetCapabilities(netId: Int32, ptr: CPointer<CNetCapabilities>): Int32
    func CJ_GetConnectionProperties(netId: Int32, ret: CPointer<CConnectionProperties>): Int32
    
    // 代理
    func CJ_GetDefaultHttpProxy(chttpProxy: CPointer<CHttpProxy>): Int32
    
    // DNS
    func CJ_GetAddressesByName(netId: Int32, host: CString): RetNetAddressArr
    
    // 状态检查
    func CJ_IsDefaultNetMetered(retPtr: CPointer<Bool>): Int32
    func CJ_HasDefaultNet(retPtr: CPointer<Bool>): Int32
    
    // 事件监听
    func CJ_OnNetAvailable(connId: Int64, callbackId: Int64): Unit
    func CJ_OnNetLost(connId: Int64, callbackId: Int64): Unit
    func CJ_OnNetBlockStatusChange(connId: Int64, callbackId: Int64): Unit
    func CJ_OnNetCapabilitiesChange(connId: Int64, callbackId: Int64): Unit
    func CJ_OnNetConnectionPropertiesChange(connId: Int64, callbackId: Int64): Unit
    func CJ_OnNetUnavailable(connId: Int64, callbackId: Int64): Unit
    
    // 状态报告
    func CJ_ReportNetConnected(netId: Int32): Int32
    func CJ_ReportNetDisconnected(netId: Int32): Int32
    
    // Socket 绑定
    func CJ_NetHandleBindSocket(netId: Int32, socketFd: IntNative): Int32
}
```

### C 结构体定义

#### CNetSpecifier

网络规格器结构体。

```cangjie
@C
struct CNetSpecifier {
    let netCapabilities: CNetCapabilities
    let bearerPrivateIdentifier: CString
    let hasSpecifier!: Bool = false
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:77-110`

#### CNetCapabilities

网络能力结构体。

```cangjie
@C
struct CNetCapabilities {
    let bearedTypeSize: Int64
    let networkCapSize: Int64
    let linkUpBandwidthKbps: UInt32
    let linkDownBandwidthKbps: UInt32
    let bearerTypes: CPointer<Int32>
    let networkCap: CPointer<Int32>
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:214-259`

#### CConnectionProperties

连接属性结构体。

```cangjie
@C
struct CConnectionProperties {
    var interfaceName: CString
    var domains: CString
    var linkAddressSize: Int64
    var dnsSize: Int64
    var routeSize: Int64
    var mtu: UInt16
    var linkAddresses: CPointer<CLinkAddress>
    var dnses: CPointer<CNetAddress>
    var routes: CPointer<CRouteInfo>
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:273-309`

#### CHttpProxy

HTTP 代理结构体。

```cangjie
@C
protected struct CHttpProxy {
    protected let host: CString
    protected let port: UInt16
    protected let exclusionList: CPointer<CString>
    protected let exclusionListSize: Int64
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:112-149`

#### CNetAddress

网络地址结构体。

```cangjie
@C
struct CNetAddress {
    let address: CString
    let family: UInt32
    let port: UInt16
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:151-162`

#### 辅助结构体

| 结构体 | 描述 |
|-------|------|
| `CLinkAddress` | 链路地址 (地址 + 前缀长度) |
| `CRouteInfo` | 路由信息 |
| `RetNetAddressArr` | 返回值包装 (code + data array) |
| `CNetCapabilityInfo` | 网络能力变化信息 |

**证据来源**：`ohos/net/connection/connection_ffi.cj:164-310`

---

## 数据转换函数

### 数组转换

```cangjie
// 仓颉数组 -> C 数组
func cjArr2CArr<CT, T>(arr: Array<T>, writeFunc: (T) -> CT, freeFunc!: (CT) -> Unit): CPointer<CT>

// C 数组 -> 仓颉数组
func cArr2cjArr<CT, T>(size: Int64, head: CPointer<CT>, mapFunc: (CT) -> T): Array<T>
```

### 字符串转换

```cangjie
// 仓颉 String -> CString
unsafe { LibC.mallocCString(str: String): CString }

// CString -> 仓颉 String
let cstr: CString = ...
let str = unsafe { cstr.read().toString() }
```

### 内存释放

```cangjie
// C 内存释放
unsafe { LibC.free(ptr: CPointer<T>) }

// CString 释放
unsafe { LibC.free(cstr: CString) }
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:20`, `:91-95`, `:104-109`

---

## 外部依赖

### HTTP 模块依赖

| 依赖 | 用途 | 类型 |
|------|------|------|
| `netstack:cj_net_http_ffi` | HTTP 栈 C 接口 | `external_deps` |
| `cangjie_ark_interop:ohos.business_exception` | 业务异常 | `cj_external_deps` |
| `cangjie_ark_interop:ohos.callback_invoke` | 回调机制 | `cj_external_deps` |
| `cangjie_ark_interop:ohos.ffi` | FFI 基础类型 | `cj_external_deps` |
| `cangjie_ark_interop:ohos.encoding.json` | JSON 编码 | `cj_external_deps` |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | 日志接口 | `cj_external_deps` |

### 连接模块依赖

| 依赖 | 用途 | 类型 |
|------|------|------|
| `netmanager_base:cj_net_connection_ffi` | 网络管理 C 接口 | `external_deps` |
| `cangjie_ark_interop:*` | 仓颉互操作框架 | `cj_external_deps` |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | 日志接口 | `cj_external_deps` |

**证据来源**：
- `ohos/net/http/BUILD.gn:26-39`
- `ohos/net/connection/BUILD.gn:31-39`

---

## 相关文档

- [API 参考](03_API_Reference.md) - 上层 API 说明
- [系统架构](02_Architecture.md) - 整体架构设计
- [构建配置](05_GN_Build.md) - 构建入口
