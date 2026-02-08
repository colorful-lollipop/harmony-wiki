# DSoftBus N-API 接口文档

## 概述

DSoftBus 通过 N-API 向 JS/TS 应用提供两个核心模块：

| 模块名 | 路径 | 用途 |
|-------|------|------|
| `@ohos.distributedsched.linkEnhance` | `sdk/napi/link_enhance/` | 链路增强连接管理 |
| `@ohos.distributedsched.proxyChannelManager` | `br_proxy/` | BR 代理通道管理 |

## 模块 1: Link Enhance N-API

**模块名**: `distributedsched.linkEnhance`

**代码位置**: `sdk/napi/link_enhance/src/napi_link_enhance_module.cpp:258-266`

### API 清单

| JS 函数 | C++ 处理器 | 用途 |
|--------|-----------|------|
| `createServer()` | `NapiLinkEnhanceServer::Create` | 创建服务器实例 |
| `createConnection()` | `NapiLinkEnhanceConnection::Create` | 创建连接实例 |

### 类: Server

**JS 类名**: `Server`

**定义文件**: `sdk/napi/link_enhance/src/napi_link_enhance_server.cpp`

**构造函数**: `NapiLinkEnhanceServer::Constructor`

#### 方法

| 方法 | 处理器 | 说明 |
|-----|--------|------|
| `start()` | `Start` | 启动服务器 |
| `stop()` | `Stop` | 停止服务器 |
| `close()` | `Close` | 关闭服务器 |
| `on(event, callback)` | `On` | 注册事件监听 |
| `off(event)` | `Off` | 取消事件监听 |

#### 事件类型

| 事件 | 说明 | 回调参数 |
|-----|------|---------|
| `connectionAccepted` | 新连接接受 | `{ deviceId, serverName, handle }` |
| `serverStopped` | 服务器停止 | `{ reason }` |

### 类: Connection

**JS 类名**: `Connection`

**定义文件**: `sdk/napi/link_enhance/src/napi_link_enhance_connection.cpp`

**构造函数**: `NapiLinkEnhanceConnection::Constructor`

#### 方法

| 方法 | 处理器 | 说明 |
|-----|--------|------|
| `connect(deviceId, options)` | `Connect` | 连接到设备 |
| `disconnect()` | `Disconnect` | 断开连接 |
| `close()` | `Close` | 关闭连接 |
| `getPeerDeviceId()` | `GetPeerDeviceId` | 获取对端设备 ID |
| `sendData(data)` | `SendData` | 发送数据 |
| `on(event, callback)` | `On` | 注册事件监听 |
| `off(event)` | `Off` | 取消事件监听 |

#### 事件类型

| 事件 | 说明 | 回调参数 |
|-----|------|---------|
| `connectResult` | 连接结果 | `{ success, reason }` |
| `dataReceived` | 数据接收 | `ArrayBuffer` |
| `disconnected` | 连接断开 | `{ reason }` |

#### 连接状态枚举

```c
// sdk/napi/link_enhance/src/napi_link_enhance_module.cpp
ConnectionState {
    STATE_BASE = 0,
    STATE_DISCONNECTED,
    STATE_CONNECTING,
    STATE_CONNECTED
}
```

### 使用示例

```javascript
import linkEnhance from '@ohos.distributedsched.linkEnhance'

// 创建服务器
const server = linkEnhance.createServer()
server.on('connectionAccepted', (data) => {
    const conn = linkEnhance.createConnection(data.handle)
    // 处理连接
})
server.start()

// 创建连接
const conn = linkEnhance.createConnection()
conn.on('connectResult', (result) => {
    if (result.success) {
        conn.sendData(new ArrayBuffer(1024))
    }
})
conn.connect(deviceId)
```

---

## 模块 2: BR Proxy Channel Manager N-API

**模块名**: `distributedsched.proxyChannelManager`

**代码位置**: `br_proxy/br_proxy_module.c`

### API 清单

| JS 函数 | C 处理器 | 同步/异步 | 说明 |
|--------|---------|----------|------|
| `openProxyChannel(options)` | `NapiOpenProxyChannel` | Promise | 打开 BR 代理通道 |
| `closeProxyChannel(channelId)` | `NapiCloseProxyChannel` | Promise | 关闭代理通道 |
| `sendData(channelId, data)` | `SendDataAsync` | 异步回调 | 发送数据 |
| `on(event, callback)` | `On` | 回调 | 注册事件 |
| `off(event)` | `Off` | 回调 | 取消事件 |

### 通道状态枚举

```c
// br_proxy/br_proxy_module.c
ChannelState {
    CHANNEL_WAIT_RESUME = 0,
    CHANNEL_RESUME,
    CHANNEL_EXCEPTION_SOFTWARE_FAILED,
    CHANNEL_BR_NO_PAIRED
}
```

### 链路类型枚举

```c
LinkType {
    LINK_BR = 0
}
```

### 事件类型

| 事件 | 说明 | 回调参数 |
|-----|------|---------|
| `receiveData` | 接收数据 | `{ channelId, data }` |
| `channelStateChange` | 通道状态变更 | `{ channelId, state, reason }` |

### 使用示例

```javascript
import proxyChannelManager from '@ohos.distributedsched.proxyChannelManager'

proxyChannelManager.on('receiveData', (data) => {
    console.log('Received:', data)
})

proxyChannelManager.on('channelStateChange', (info) => {
    console.log('State changed:', info.state)
})

const channelId = await proxyChannelManager.openProxyChannel({
    deviceId: 'XX:XX:XX:XX:XX:XX',
    proxyUserName: 'test_user'
})

proxyChannelManager.sendData(channelId, new ArrayBuffer(1024))
```

---

## 错误码

### N-API 特有错误

| 错误码 | 说明 |
|-------|------|
| `LINK_ENHANCE_PARAMETER_INVALID` | 参数无效 |
| `LINK_ENHANCE_SERVER_DIED` | 服务已终止 |

### 通用 SoftBus 错误码

所有 N-API 操作可能返回 `softbus_error_code.h` 中定义的错误码：

| 模块 | 基础错误码 |
|------|-----------|
| 公共 | `SOFTBUS_PUBLIC_ERR_BASE` |
| 连接 | `SOFTBUS_CONN_ERR_BASE` |
| 认证 | `SOFTBUS_AUTH_ERR_BASE` |
| 网络 | `SOFTBUS_NETWORK_ERR_BASE` |
| 传输 | `SOFTBUS_TRANS_ERR_BASE` |

> 代码证据: `interfaces/kits/common/softbus_error_code.h`

---

## 调用链

### Link Enhance 调用链

```
JS: createServer()
    ↓
N-API: NapiLinkEnhanceServer::Create()
    ↓
C++: NapiLinkEnhanceServer::Constructor()
    ↓
JS 对象: Server 实例

JS: server.start()
    ↓
N-API: NapiLinkEnhanceServer::Start()
    ↓
IGeneralListener 回调注册
    ↓
softbus_adapter_*.h → core/connection/
```

### BR Proxy 调用链

```
JS: openProxyChannel()
    ↓
N-API: NapiOpenProxyChannel()
    ↓
NAPI Promise 封装
    ↓
br_proxy_session_manager.c
    ↓
core/transmission/
```

---

## 线程安全

- **N-API 回调**: 通过 `napi_create_threadsafe_function` 线程安全调用
- **JS 主线程**: 事件通过 `napi_call_function` 回调到 JS
- **资源管理**: 使用 `napi_reference` 保持 JS 对象引用

> 代码证据: `sdk/napi/link_enhance/src/napi_link_enhance_utils.cpp`

---

## 参数校验

### 必检参数

| 参数 | 检查方式 |
|-----|---------|
| `env` | `CONN_CHECK_AND_RETURN_RET_LOGE` |
| `deviceId` | 空指针检查、长度校验 |
| `data` | `CONN_CHECK_AND_RETURN_LOGE(data != nullptr)` |
| `handle` | 范围校验 |

### 错误处理

- 参数错误返回 `SOFTBUS_INVALID_PARAM`
- 内存错误返回 `SOFTBUS_MEM_ERR`
- 权限错误返回 `SOFTBUS_PERMISSION_DENIED`

---

**相关文档**

- [项目概览](./01_Overview.md)
- [架构说明](./02_Architecture.md)
- [内部 API](./04_InnerAPI.md)
