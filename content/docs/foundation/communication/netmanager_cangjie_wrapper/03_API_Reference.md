# Cangjie API 参考

## 命名空间概览

| 命名空间 | 描述 | 导出路径 |
|---------|------|---------|
| `ohos.net` | 网络模块包定义 | `kit/NetworkKit/index.cj` |
| `ohos.net.connection` | 网络连接管理 | `kit/NetworkKit/index.cj` |
| `ohos.net.http` | HTTP 请求 | `kit/NetworkKit/index.cj` |

**证据来源**：`kit/NetworkKit/index.cj:18-22`

---

## 网络连接管理 (ohos.net.connection)

### 模块导入

```cangjie
import ohos.net.connection.*
```

### 函数 API

#### createNetConnection

创建网络连接。

```cangjie
public func createNetConnection(netSpecifier!: ?NetSpecifier = None, timeout!: UInt32 = 0): NetConnection
```

| 参数 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `netSpecifier` | `?NetSpecifier` | 否 | `None` | 网络规格器，指定要连接的网络类型 |
| `timeout` | `UInt32` | 否 | `0` | 超时时间（毫秒） |

**返回**：`NetConnection` - 网络连接对象

**证据来源**：`ohos/net/connection/net_connection.cj:36-52`

---

#### getDefaultNet

获取默认激活的数据网络。

```cangjie
public func getDefaultNet(): NetHandle
```

| 返回 | 描述 |
|------|------|
| `NetHandle` | 默认激活数据网络的句柄 |

**权限**：`ohos.permission.GET_NETWORK_INFO`

**错误码**：
- `201` - 权限拒绝
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:61-78`

---

#### getDefaultHttpProxy

获取默认 HTTP 代理设置。

```cangjie
public func getDefaultHttpProxy(): HttpProxy
```

| 返回 | 描述 |
|------|------|
| `HttpProxy` | HTTP 代理设置 |

**错误码**：
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:90-112`

---

#### getAppNet

获取通过 `setAppNet` 绑定到进程的 NetHandle。

```cangjie
public func getAppNet(): NetHandle
```

| 返回 | 描述 |
|------|------|
| `NetHandle` | 绑定到进程的 NetHandle |

**错误码**：
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:120-136`

---

#### setAppNet

将进程绑定到指定网络。

```cangjie
public func setAppNet(netHandle: NetHandle): Unit
```

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `netHandle` | `NetHandle` | 是 | 要绑定的网络句柄 |

**权限**：`ohos.permission.INTERNET`

**错误码**：
- `201` - 权限拒绝
- `2100001` - 无效参数
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:148-163`

---

#### getAllNets

获取所有已激活的网络列表。

```cangjie
public func getAllNets(): Array<NetHandle>
```

| 返回 | 描述 |
|------|------|
| `Array<NetHandle>` | 已激活网络的句柄数组 |

**权限**：`ohos.permission.GET_NETWORK_INFO`

**错误码**：
- `201` - 权限拒绝
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:172-192`

---

#### getConnectionProperties

查询网络的连接属性。

```cangjie
public func getConnectionProperties(netHandle: NetHandle): ConnectionProperties
```

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `netHandle` | `NetHandle` | 是 | 要查询的网络句柄 |

| 返回 | 描述 |
|------|------|
| `ConnectionProperties` | 网络连接属性 |

**权限**：`ohos.permission.GET_NETWORK_INFO`

**错误码**：
- `201` - 权限拒绝
- `2100001` - 无效参数
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:203-228`

---

#### getNetCapabilities

获取网络的能力信息。

```cangjie
public func getNetCapabilities(netHandle: NetHandle): NetCapabilities
```

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `netHandle` | `NetHandle` | 是 | 要查询的网络句柄 |

| 返回 | 描述 |
|------|------|
| `NetCapabilities` | 网络能力信息 |

**权限**：`ohos.permission.GET_NETWORK_INFO`

**错误码**：
- `201` - 权限拒绝
- `2100001` - 无效参数
- `2100002` - 连接服务失败
- `2100003` - 系统内部错误

**证据来源**：`ohos/net/connection/net_connection.cj:239-263`

---

#### isDefaultNetMetered

检查当前网络是否按流量计费。

```cangjie
public func isDefaultNetMetered(): Bool
```

| 返回 | 描述 |
|------|------|
| `Bool` | `true` 表示按流量计费，`false` 表示否 |

**权限**：`ohos.permission.GET_NETWORK_INFO`

**证据来源**：`ohos/net/connection/net_connection.cj:272-289`

---

#### hasDefaultNet

检查默认数据网络是否已激活。

```cangjie
public func hasDefaultNet(): Bool
```

| 返回 | 描述 |
|------|------|
| `Bool` | `true` 表示已激活，`false` 表示否 |

**权限**：`ohos.permission.GET_NETWORK_INFO`

**证据来源**：`ohos/net/connection/net_connection.cj:298-315`

---

#### getAddressesByName

解析主机名获取所有 IP 地址（基于默认网络）。

```cangjie
public func getAddressesByName(host: String): Array<NetAddress>
```

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `host` | `String` | 是 | 主机名或域名 |

| 返回 | 描述 |
|------|------|
| `Array<NetAddress>` | IP 地址数组 |

**权限**：`ohos.permission.INTERNET`

**证据来源**：`ohos/net/connection/net_connection.cj:376-385`

---

### 类 API

#### NetConnection

网络连接处理器，用于监听网络状态变化。

**继承**：`RemoteDataLite`

```cangjie
public class NetConnection <: RemoteDataLite
```

**构造函数**：

```cangjie
init(id: Int64)
```

**方法**：

| 方法 | 描述 |
|------|------|
| `on(event: NetConnectionEvent, callback)` | 注册网络事件监听器 |
| `register()` | 开始接收网络状态变化通知 |
| `unregister()` | 取消监听网络状态变化 |

**事件类型** (`NetConnectionEvent`)：

| 事件 | 描述 |
|------|------|
| `NetAvailable` | 网络可用 |
| `NetLost` | 网络丢失 |
| `NetCapabilitiesChange` | 网络能力变化 |
| `NetConnectionPropertiesChange` | 连接属性变化 |
| `NetBlockStatusChange` | 阻塞状态变化 |
| `NetUnavailable` | 网络不可用 |

**证据来源**：`ohos/net/connection/net_connection.cj:389-589`

---

#### NetHandle

网络句柄类。

```cangjie
public class NetHandle
```

**属性**：

| 属性 | 类型 | 描述 |
|------|------|------|
| `netId` | `Int32` | 网络 ID，0 表示无默认网络，100 及以上为有效网络 |

**方法**：

| 方法 | 描述 |
|------|------|
| `getAddressesByName(host: String)` | 解析主机名获取 IP 地址 |
| `getAddressByName(host: String)` | 获取第一个 IP 地址 |

**证据来源**：`ohos/net/connection/net_connection.cj:591-685`

---

## HTTP 请求 (ohos.net.http)

### 模块导入

```cangjie
import ohos.net.http.*
```

### 函数 API

#### createHttp

创建 HTTP 请求任务。

```cangjie
public func createHttp(): HttpRequest
```

| 返回 | 描述 |
|------|------|
| `HttpRequest` | HTTP 请求任务对象 |

**证据来源**：`ohos/net/http/http.cj:34-41`

---

#### createHttpResponseCache

创建 HTTP 响应缓存。

```cangjie
public func createHttpResponseCache(cacheSize!: UInt32 = MAX_CACHE_SIZE): HttpResponseCache
```

| 参数 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `cacheSize` | `UInt32` | 否 | `10 * 1024 * 1024` | 缓存大小，最大 10MB |

| 返回 | 描述 |
|------|------|
| `HttpResponseCache` | 响应缓存对象 |

**证据来源**：`ohos/net/http/http.cj:44-61`

---

### 类 API

#### HttpRequest

HTTP 请求任务类。

```cangjie
public class HttpRequest
```

**构造函数**：

```cangjie
init(id: Int64)
```

**方法**：

| 方法 | 描述 |
|------|------|
| `request(url, options, callback)` | 发起 HTTP 请求 |
| `requestInStream(url, options, callback)` | 发起流式 HTTP 请求 |
| `destroy()` | 销毁 HTTP 请求 |
| `on(event, callback)` | 注册事件监听器 |
| `once(event, callback)` | 注册一次性事件监听器 |
| `off(event, callback?)` | 取消事件监听 |

**事件类型** (`HttpRequestEvent`)：

| 事件 | 描述 |
|------|------|
| `HeadersReceive` | 收到响应头 |
| `DataReceive` | 收到响应数据 |
| `DataEnd` | 响应数据结束 |
| `DataReceiveProgress` | 数据接收进度 |
| `DataSendProgress` | 数据发送进度 |

**证据来源**：`ohos/net/http/http.cj:67-704`

---

#### HttpResponseCache

HTTP 响应缓存类。

```cangjie
public class HttpResponseCache
```

**方法**：

| 方法 | 描述 |
|------|------|
| `flush()` | 将缓存数据写入文件系统 |
| `delete()` | 禁用缓存并删除数据 |

**证据来源**：`ohos/net/http/http.cj:706-750`

---

## 错误码参考

### 通用错误码

| 错误码 | 描述 |
|--------|------|
| `201` | 权限拒绝 |
| `2100001` | 无效参数值 |
| `2100002` | 连接服务失败 |
| `2100003` | 系统内部错误 |
| `2101007` | 回调不存在 |
| `2101008` | 回调已存在 |
| `2101019` | 地址未找到 |
| `2101022` | 请求次数超过最大允许 |

### HTTP 错误码

| 错误码 | 描述 |
|--------|------|
| `2300001` | 不支持的协议 |
| `2300003` | 无效的 URL 格式或缺少 URL |
| `2300005` | 代理名称解析失败 |
| `2300006` | 主机名解析失败 |
| `2300007` | 连接服务器失败 |
| `2300008` | 无效的服务器响应 |
| `2300009` | 远程资源访问被拒绝 |
| `2300016` | HTTP2 帧层错误 |
| `2300018` | 传输了部分文件 |
| `2300023` | 写入磁盘或应用失败 |
| `2300025` | 上传失败 |
| `2300026` | 打开或读取本地数据失败 |
| `2300027` | 内存不足 |
| `2300028` | 操作超时 |
| `2300047` | 重定向次数达到最大 |
| `2300052` | 服务器未返回任何内容 |
| `2300055` | 发送数据失败 |
| `2300056` | 接收数据失败 |
| `2300058` | 本地 SSL 证书错误 |
| `2300059` | 指定的 SSL 密码套件无法使用 |
| `2300060` | 无效的 SSL 对等证书或 SSH 远程密钥 |
| `2300061` | 无效的 HTTP 编码格式 |
| `2300063` | 超出最大文件大小 |
| `2300070` | 远程磁盘已满 |
| `2300073` | 远程文件已存在 |
| `2300077` | SSL CA 证书不存在或不可访问 |
| `2300078` | 远程文件未找到 |
| `2300094` | 身份验证错误 |
| `2300997` | 不允许明文流量 |
| `2300998` | 不允许访问此域名 |
| `2300999` | 内部错误 |

**证据来源**：`ohos/net/http/http.cj:189-225`

---

## 相关文档

- [FFI 接口](04_FFI_Interface.md) - 底层 FFI 函数定义
- [系统架构](02_Architecture.md) - 架构设计与数据流
