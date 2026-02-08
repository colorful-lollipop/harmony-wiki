# 安全风险评审

## 评审范围

本评审覆盖 `netmanager_cangjie_wrapper` 项目的以下方面：

| 模块 | 文件 | 评审范围 |
|------|------|---------|
| HTTP 请求 | `ohos/net/http/*.cj` | 请求参数校验、敏感信息处理 |
| 连接管理 | `ohos/net/connection/*.cj` | 网络参数校验、权限检查 |
| FFI 接口 | `*.cj` | 跨语言数据传递安全 |
| 构建配置 | `BUILD.gn`, `bundle.json` | 依赖安全、SDK 分发 |

**检查局限性**：本评审基于源代码静态分析，底层 C/C++ 实现（位于 `communication_netmanager_base` 和 `communication_netstack` 仓库）未在范围内。

---

## 攻击面清单

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|---------|
| **HTTP 请求参数** | `HttpRequest.request()`, `HttpRequestOptions` | 🔴 高 |
| **URL 参数** | `url: String` 参数 | 🔴 高 |
| **网络连接参数** | `NetSpecifier`, `NetHandle` | 🟡 中 |
| **DNS 解析** | `getAddressesByName()` | 🟡 中 |
| **HTTP 响应数据** | `HttpResponse` 回调 | 🟡 中 |
| **回调注册** | `on()`, `once()` 事件监听器 | 🟡 中 |
| **本地缓存** | `HttpResponseCache` | 🟢 低 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          仓颉应用层 (Kit API)                        │   │
│  │         (受权限保护的 API)                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                         │                                  │
│                         ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            FFI 封装层                               │   │
│  │      (类型转换、内存管理)                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                         │                                  │
│                         ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         底层 C 接口 (netmanager_base/netstack)      │   │
│  │              (IPC/SA 通信)                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 可被利用点分析

### 1. HTTP 请求 URL 未完整校验 ⚠️ 中风险

**证据**：`ohos/net/http/http.cj:233-268`

```cangjie
public func request(url: String, options: HttpRequestOptions, callback: AsyncCallback<HttpResponse>): Unit {
    let (c_url, optPtr) = unsafe { parseParam(url, options) }
    // url 直接传递给底层 C 接口
}
```

**问题**：URL 格式校验在底层 C 接口进行，仓颉层未做预校验

**影响**：可能导致畸形 URL 触发底层解析错误

**建议**：在仓颉层添加 URL 格式预校验

---

### 2. 内存分配失败处理 ⚠️ 中风险

**证据**：`ohos/net/http/http_ffi.cj:120-165`

```cangjie
init(ops: HttpRequestOptions) {
    try {
        this.method = LibC.mallocCString(ops.method.getValue())
        this.header = mallocArrStringOp(ops.header)
        // ...
    } catch (e: Exception) {
        LibC.free(this.method)
        this.header.free()
        // ...
        throw BusinessException(2300027, "Out of memory.")
    }
}
```

**问题**：`mallocCString` 等内存分配可能失败，抛出 `Exception`

**影响**：内存耗尽时可能触发拒绝服务

**当前措施**：已有 try-catch 包裹和资源释放

**建议**：考虑添加请求大小上限控制

---

### 3. Hostname 注入风险 ⚠️ 低风险

**证据**：`ohos/net/connection/net_connection.cj:632-658`

```cangjie
public func getAddressesByName(host: String): Array<NetAddress> {
    if (host.isEmpty() || host.startsWith("\0")) {
        throw BusinessException(2100001, "...")
    }
    let cHost = unsafe { LibC.mallocCString(host) }
    let ret: RetNetAddressArr = unsafe { CJ_GetAddressesByName(netId, cHost) }
}
```

**问题**：已检查空字符串和 null 起始字符，但未检查特殊字符

**当前措施**：已做基础校验

**建议**：考虑添加 DNS 域名格式正则校验

---

### 4. 回调注册缺乏去重 🟢 低风险

**证据**：`ohos/net/http/http.cj:160-172`

```cangjie
private func commonSubscribe0Arg(callbackType: HttpRequestEvent, callback: CallbackObject) {
    if (!registerMap[callbackType]) {
        register(callbackType, argWrapper0(callbackType))
        registerMap[callbackType] = true
    } else {
        if (findCallbackObject(callbackType, callback) >= 0) {
            HTTP_LOG.info("The callback is registered, no need to re-registered")
            return
        }
    }
    callBackMap.addIfAbsent(callbackType, ArrayList<CallbackObject>())
    callBackMap[callbackType].add(callback)
}
```

**问题**：同一回调可能被多次注册（虽会检查但仅返回不抛错）

**当前措施**：已有重复检查机制

**建议**：可考虑统一日志级别（当前混用 info/error）

---

### 5. SSL/TLS 证书路径未校验 🟢 低风险

**证据**：`ohos/net/http/http_ffi.cj:107-108`

```cangjie
var caPath: CString
// ...
this.caPath = mallocStringOp(ops.caPath)
```

**问题**：CA 路径直接传递给底层，可能指向敏感路径

**当前措施**：路径校验在底层 C 接口进行

**建议**：添加路径白名单校验（底层实现）

---

## 权限声明验证

### 权限声明模式

所有需要权限的 API 均通过 `@!APILevel` 注解声明：

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.GET_NETWORK_INFO",
    syscap: "SystemCapability.Communication.NetManager.Core",
    throwexception: true,
    workerthread: true
]
```

### 权限清单

| API | 权限 | 正确性 |
|-----|------|--------|
| `getDefaultNet()` | `GET_NETWORK_INFO` | ✅ |
| `getAllNets()` | `GET_NETWORK_INFO` | ✅ |
| `getConnectionProperties()` | `GET_NETWORK_INFO` | ✅ |
| `getNetCapabilities()` | `GET_NETWORK_INFO` | ✅ |
| `setAppNet()` | `INTERNET` | ✅ |
| `getAddressesByName()` | `INTERNET` | ✅ |
| `HttpRequest.request()` | `INTERNET` | ✅ |

**证据来源**：`ohos/net/connection/net_connection.cj`, `ohos/net/http/http.cj`

---

## 内存安全

### C 内存管理

本项目使用手动内存管理，所有 `@C` 结构体均实现 `free()` 方法：

```cangjie
func free(): Unit {
    unsafe {
        LibC.free(this.method)
        LibC.free(this.caPath)
        this.header.free()
        // ...
    }
}
```

**检查点**：

| 检查项 | 状态 | 说明 |
|--------|------|------|
| malloc 配对 free | ✅ 已实现 | 每个 malloc 都有对应的 free |
| 空指针检查 | ✅ 已实现 | 使用 `isNull()` / `isNotNull()` |
| 异常安全 | ✅ 已实现 | try-catch 包裹内存分配 |
| 数组释放 | ✅ 已实现 | 循环释放数组元素 |

---

## 信息泄露风险

### 敏感信息处理

| 数据类型 | 处理方式 | 评估 |
|---------|---------|------|
| URL | 直接传递 | 🟡 需信任底层 |
| HTTP 响应头 | 通过回调传递 | 🟢 正常业务 |
| 错误信息 | 通过 BusinessException 传递 | 🟢 已脱敏处理 |
| 网络 ID | 整数类型 | 🟢 安全 |

---

## 修复建议优先级

| 优先级 | 问题 | 建议 | 工作量 |
|--------|------|------|--------|
| P1 | Hostname 注入风险 | 添加 DNS 域名格式校验 | 低 |
| P2 | 请求大小无上限 | 添加请求体大小限制 | 中 |
| P3 | URL 未预校验 | 添加 URL 格式校验 | 低 |
| P4 | CA 路径白名单 | 底层添加路径校验 | 高 |

---

## 相关文档

- [API 参考](03_API_Reference.md) - API 权限说明
- [FFI 接口](04_FFI_Interface.md) - 数据传递机制
- [系统架构](02_Architecture.md) - 信任边界说明
