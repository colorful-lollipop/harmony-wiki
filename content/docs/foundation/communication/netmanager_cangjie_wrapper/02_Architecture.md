# 系统架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      仓颉应用层 (Cangjie App)                     │
├─────────────────────────────────────────────────────────────────┤
│  kit/NetworkKit                                                  │
│  └── index.cj (模块导出)                                         │
├─────────────────────────────────────────────────────────────────┤
│                    FFI 封装层 (ohos.net)                         │
│  ┌───────────────────────────┐  ┌────────────────────────────┐  │
│  │    ohos.net.connection    │  │      ohos.net.http         │  │
│  │  net_connection.cj        │  │    http.cj                 │  │
│  │  connection_ffi.cj       │  │    http_ffi.cj             │  │
│  │  connection_common.cj     │  │    http_common.cj          │  │
│  └───────────────────────────┘  └────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                    底层 C 接口 (Native)                          │
│  ┌───────────────────────────┐  ┌────────────────────────────┐  │
│  │   communication_          │  │    communication_         │  │
│  │   netmanager_base        │  │    netstack                │  │
│  │   (N-API/Ffi)            │  │    (N-API/Ffi)             │  │
│  └───────────────────────────┘  └────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                    系统服务层                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │        NetManager Service / Network Stack Service         │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**证据来源**：`README.md:7-29`, `figures/netmanager_cangjie_wrapper_architecture_en.png`

## 模块职责

### Kit 层 (`kit/NetworkKit`)

**职责**：对外暴露仓颉 API，提供模块化导入能力

**证据来源**：`kit/NetworkKit/index.cj:18-22`

```cangjie
package kit.NetworkKit

public import ohos.net.*
public import ohos.net.connection.*
public import ohos.net.http.*
```

### 连接管理模块 (`ohos.net.connection`)

**职责**：提供网络连接管理能力

| 子模块 | 职责 |
|--------|------|
| `net_connection.cj` | 网络连接 API 实现，包含 `NetConnection`、`NetHandle` 等类 |
| `connection_ffi.cj` | FFI 接口定义，声明 `CJ_*` 外部函数 |
| `connection_common.cj` | 公共类型定义（待确认） |

**关键类**：
- `NetConnection` - 网络连接处理器
- `NetHandle` - 网络句柄
- `NetCapabilities` - 网络能力
- `ConnectionProperties` - 连接属性

**证据来源**：`ohos/net/connection/net_connection.cj:18-749`

### HTTP 请求模块 (`ohos.net.http`)

**职责**：提供 HTTP 数据请求能力

| 子模块 | 职责 |
|--------|------|
| `http.cj` | HTTP 请求 API 实现，包含 `HttpRequest`、`HttpResponseCache` 等类 |
| `http_ffi.cj` | FFI 接口定义，声明 `CJ_*` 外部函数 |
| `http_common.cj` | 公共类型定义（待确认） |

**关键类**：
- `HttpRequest` - HTTP 请求任务
- `HttpResponseCache` - HTTP 响应缓存

**证据来源**：`ohos/net/http/http.cj:18-751`

## FFI 调用机制

本项目采用仓颉 FFI（Foreign Function Interface）机制调用底层 C 接口。

### FFI 声明模式

```cangjie
foreign {
    func CJ_CreateNetConnection(netSpecifier: CNetSpecifier, timeout: UInt32): Int64
    func CJ_GetDefaultNet(netId: CPointer<Int32>): Int32
    // ...
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:23-75`

### C 结构体映射

```cangjie
@C
struct CNetSpecifier {
    let netCapabilities: CNetCapabilities
    let bearerPrivateIdentifier: CString
    let hasSpecifier!: Bool = false
}
```

**证据来源**：`ohos/net/connection/connection_ffi.cj:77-110`

### 异步回调模式

```cangjie
// 使用 Callback1Param 处理异步回调
let wrapper = { netId: Int32 =>
    let handle = NetHandle(netId)
    callback.invoke(None, handle)
}
let registerCall = Callback1Param<Int32, Unit>(wrapper)
```

**证据来源**：`ohos/net/connection/net_connection.cj:416-422`

## 依赖方向

```
kit/NetworkKit
    │
    ├──► ohos.net
    │       │
    │       ├──► ohos.net.connection
    │       │       │
    │       │       ├──► cangjie_ark_interop (ohos.ffi, ohos.callback_invoke, ohos.business_exception)
    │       │       ├──► hiviewdfx_cangjie_wrapper (ohos.hilog)
    │       │       └──► netmanager_base (cj_net_connection_ffi)
    │       │
    │       └──► ohos.net.http
    │               │
    │               ├──► cangjie_ark_interop
    │               ├──► hiviewdfx_cangjie_wrapper
    │               └──► netstack (cj_net_http_ffi)
```

**证据来源**：
- `ohos/net/connection/BUILD.gn:31-39`
- `ohos/net/http/BUILD.gn:26-39`

## 数据类型转换

### 仓颉 ↔ C 数据转换

| 仓颉类型 | C 类型 | 转换方式 |
|---------|--------|---------|
| `String` | `CString` | `LibC.mallocCString()` |
| `Array<T>` | `CArrXxx` | `cjArr2CArr()` |
| `Bool` | `Bool` | 直接传递 |
| `Int32/UInt32` | `int32_t/uint32_t` | 直接传递 |
| `?T` | 指针 | `safeMalloc()` |

### 内存管理

- **C 内存分配**：`LibC.malloc()`, `safeMalloc()`
- **C 内存释放**：`LibC.free()`
- **C 字符串**：`LibC.mallocCString()`, `LibC.free()`
- **资源释放**：每个 `@C` 结构体都有 `free()` 方法

**证据来源**：
- `ohos/net/connection/connection_ffi.cj:20-21`, `:91-95`, `:104-109`
- `ohos/net/http/http_ffi.cj:20-23`, `:142-148`, `:167-186`

## 相关文档

- [API 参考](03_API_Reference.md) - 详细 API 说明
- [FFI 接口](04_FFI_Interface.md) - FFI 函数清单
- [构建配置](05_GN_Build.md) - 编译产物说明
