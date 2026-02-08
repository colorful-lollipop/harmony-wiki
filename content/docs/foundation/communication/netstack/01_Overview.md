# OpenHarmony NetStack 项目概览

## 1. 一句话定义

> **NetStack 是 OpenHarmony 通信子系统的应用框架层网络库，通过 N-API/ANI/FFI 向应用提供 HTTP、Socket(TCP/UDP/TLS)、WebSocket 等网络通信能力。**

---

## 2. 能力边界

### 2.1 能做什么

| 能力 | 说明 | 典型场景 |
|------|------|----------|
| **HTTP/HTTPS 请求** | 完整的 HTTP 客户端，支持 GET/POST/PUT/DELETE 等方法 | REST API 调用、文件下载 |
| **WebSocket 连接** | 全双工 WebSocket 客户端 | 实时通信、在线游戏 |
| **TCP Socket** | 客户端和服务端 Socket | 自定义协议通信 |
| **UDP Socket** | 支持广播和组播 | 局域网发现、音视频流 |
| **TLS/SSL 加密** | 双向证书认证、证书锁定 | 安全通信、金融应用 |
| **HTTP 缓存** | LRU + Disk 缓存策略 | 离线浏览、省流量 |
| **代理支持** | HTTP/HTTPS/SOCKS5 代理 | 企业网络、翻墙 |

### 2.2 不能做什么

| 限制 | 说明 | 替代方案 |
|------|------|----------|
| ❌ 不是 HTTP 服务器 | 无法创建 Web 服务器 | 使用 Node.js 或其他服务框架 |
| ❌ 不是完整协议栈 | 不提供 TCP/IP 实现 | 依赖内核网络栈 |
| ❌ 不管理网络连接 | 不处理 WiFi/移动数据切换 | 由 netmanager_base SA 管理 |
| ❌ 不处理 DNS | 不实现 DNS 协议 | 依赖系统 DNS 或 cURL |
| ❌ 不支持 HTTP/3 (默认) | 需要编译开关开启 | 开启 `netstack_feature_http3` |

---

## 3. 运行环境

### 3.1 系统要求

| 属性 | 要求 |
|------|------|
| **OpenHarmony 版本** | 标准系统 (standard) |
| **API 版本** | 6+ (基础功能), 9+ (增强功能), 10+ (跨平台) |
| **最低 ROM** | 3MB |
| **最低 RAM** | 5MB |

### 3.2 必要权限

所有网络操作必须声明以下权限：

```json
// config.json 或 module.json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET"
      }
    ]
  }
}
```

**权限校验位置**: `utils/common_utils/src/netstack_common_utils.cpp:145-186`

### 3.3 依赖的系统服务

| 服务 | 用途 | 接口 |
|------|------|------|
| **netmanager_base** | 网络连接管理、代理获取 | `NetConnClient::GetInstance()` |
| **access_token** | 权限校验 | `AccessTokenKit::VerifyAccessToken()` |
| **samgr** | 服务发现 | `SystemAbilityManagerClient` |

### 3.4 第三方依赖

| 库 | 用途 | 版本 |
|----|------|------|
| **cURL** | HTTP 协议实现 | 系统内置 |
| **OpenSSL/BoringSSL** | TLS/SSL 加密 | 系统内置 |
| **libwebsockets** | WebSocket 协议 | 系统内置 |
| **FFRT** | 轻量级线程池 | 系统内置 |

---

## 4. 快速开始

### 4.1 HTTP 请求示例

```typescript
// 1. 导入模块
import http from '@ohos.net.http'

// 2. 创建请求对象
let httpRequest = http.createHttp()

// 3. 发起请求
httpRequest.request(
  'https://api.example.com/data',
  {
    method: http.RequestMethod.GET,
    header: {
      'Content-Type': 'application/json'
    },
    connectTimeout: 10000,
    readTimeout: 30000
  },
  (err, data) => {
    if (err) {
      console.error('请求失败:', JSON.stringify(err))
    } else {
      console.info('响应结果:', JSON.stringify(data.result))
    }
  }
)
```

**对应 Native 代码位置**: `frameworks/js/napi/http/http_module/src/http_module.cpp:540-552`

### 4.2 WebSocket 连接示例

```typescript
import webSocket from '@ohos.net.webSocket'

// 1. 创建 WebSocket 对象
let ws = webSocket.createWebSocket()

// 2. 监听消息
ws.on('message', (err, message) => {
  console.info('收到消息:', message)
})

// 3. 建立连接
ws.connect('wss://echo.websocket.org', {}, (err) => {
  if (err) {
    console.error('连接失败:', JSON.stringify(err))
  } else {
    console.info('连接成功')
    // 4. 发送消息
    ws.send('Hello WebSocket', (err) => {
      if (!err) {
        console.info('发送成功')
      }
    })
  }
})
```

**对应 Native 代码位置**: `frameworks/js/napi/websocket/websocket_module/src/websocket_module.cpp:215-227`

### 4.3 Socket (TCP) 示例

```typescript
import socket from '@ohos.net.socket'

// 1. 创建 TCP Socket
let tcp = socket.constructTCPSocketInstance()

// 2. 监听消息
tcp.on('message', (data) => {
  console.info('收到数据:', JSON.stringify(data))
})

// 3. 绑定本地地址
tcp.bind({
  address: '0.0.0.0',
  port: 0  // 0 表示随机端口
})

// 4. 连接到服务端
tcp.connect({
  address: {
    address: '192.168.1.100',
    port: 8080
  },
  timeout: 5000
})

// 5. 发送数据
tcp.send({
  data: 'Hello TCP'
})
```

**对应 Native 代码位置**: `frameworks/js/napi/socket/socket_module/src/socket_module.cpp:1190-1204`

---

## 5. 项目结构速览

```
netstack/
├── figures/                    # 架构图资源
├── frameworks/                 # 【接口实现 - 核心代码】
│   ├── js/napi/               # 标准系统 JS N-API (应用入口)
│   │   ├── http/              # HTTP API 实现
│   │   ├── socket/            # Socket API 实现
│   │   ├── websocket/         # WebSocket API 实现
│   │   ├── net_ssl/           # 网络安全 API
│   │   └── fetch/             # Fetch API (小型系统)
│   ├── ets/ani/               # ArkTS ANI 接口
│   └── cj/                    # Cangjie FFI 接口
├── interfaces/                # 【接口定义】
│   ├── kits/                  # JS 接口声明 (.d.ts)
│   └── innerkits/             # Native 接口头文件
│       ├── http_client/       # HTTP 客户端库
│       ├── net_ssl/           # SSL/TLS 接口
│       └── websocket_native/  # WebSocket Native 接口
└── utils/                     # 【公共工具】
    ├── common_utils/          # 通用工具 (权限检查等)
    ├── napi_utils/            # N-API 工具函数
    └── http_over_curl/        # cURL HTTP 实现
```

**重要说明**: `test/` 目录包含测试代码，不参与运行时交付。

---

## 6. 核心概念

### 6.1 N-API 架构

NetStack 使用 N-API 将 C++ 实现暴露给 JavaScript：

```
JS 应用
    ↓ N-API 调用
┌─────────────────────┐
│ N-API 层 (Module)   │  frameworks/js/napi/*/...module.cpp
│ - 参数解析          │
│ - 权限检查          │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Native 实现层       │  frameworks/native/
│ - HTTP Client       │
│ - TLS Socket        │
│ - WebSocket         │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ 第三方库            │
│ - cURL (HTTP)       │
│ - OpenSSL (TLS)     │
│ - libwebsockets     │
└─────────────────────┘
```

### 6.2 线程模型

| 操作 | 执行线程 | 说明 |
|------|----------|------|
| N-API 调用 | 主线程 (JS) | 同步入口 |
| HTTP 请求 | FFRT 线程池 | 异步执行 |
| Socket I/O | 事件循环线程 | libwebsockets 管理 |
| 回调到 JS | 主线程 | napi_schedule_task |

### 6.3 错误处理

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| **201** | Permission denied | 未声明 INTERNET 权限 |
| **230** | URL error | URL 格式错误 |
| **231** | Network unreachable | 网络不可达 |
| **232** | Connection timeout | 连接超时 |
| **233** | SSL/TLS error | 证书验证失败 |

**错误码定义位置**: `utils/common_utils/include/constant.h`

---

## 7. 与其他模块的关系

```
┌─────────────────────────────────────────────────────────────────┐
│                      应用 (Application)                          │
│         import http from '@ohos.net.http'                       │
└──────────────────────┬──────────────────────────────────────────┘
                       │ 使用 N-API
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    NetStack (本模块)                             │
│  ┌────────────┐ ┌────────────┐ ┌─────────────────────────────┐  │
│  │ HTTP Client│ │  Socket    │ │       WebSocket             │  │
│  └─────┬──────┘ └─────┬──────┘ └──────────────┬──────────────┘  │
│        └──────────────┴───────────────────────┘                 │
│                       │ IPC 调用                                │
└───────────────────────┼─────────────────────────────────────────┘
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              NetManager_Base (communication_netmanager_base)    │
│  - 网络连接管理                                                  │
│  - 代理配置获取                                                  │
│  - 权限校验 (委托)                                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. 学习路径建议

### 8.1 新人开发者 (应用开发)

1. **快速入门** (15 分钟)
   - 阅读本文档 (01_Overview.md)
   - 理解能力边界和运行环境
   - 复制 HTTP/WebSocket 示例代码

2. **API 学习** (30 分钟)
   - 查阅 [03_N-API_Reference.md](03_N-API_Reference.md)
   - 了解完整的 API 清单和参数
   - 尝试修改示例代码

3. **问题排查** (按需)
   - 查看 [05_Security_Review.md](05_Security_Review.md) 了解常见安全风险
   - 查阅 [appendix/Callgraphs.md](appendix/Callgraphs.md) 理解调用链

### 8.2 系统开发者 (框架开发)

1. **架构理解** (30 分钟)
   - 阅读 [02_Architecture.md](02_Architecture.md)
   - 理解组件关系和数据流
   - 查看线程模型

2. **Native API** (1 小时)
   - 查阅 [04_Inner_API.md](04_Inner_API.md)
   - 理解 http_client、TLS Socket 接口
   - 查看头文件定义

3. **构建与集成** (30 分钟)
   - 查阅 [07_Build.md](07_Build.md)
   - 了解 GN targets 和编译产物

### 8.3 安全研究员

1. **攻击面识别** (20 分钟)
   - 直接阅读 [05_Security_Review.md](05_Security_Review.md)
   - 理解 5 个主要风险点

2. **深入分析** (1 小时)
   - 查阅 [appendix/Callgraphs.md](appendix/Callgraphs.md)
   - 跟踪输入处理流程
   - 定位关键代码位置

---

## 9. 相关文档

| 文档 | 说明 | 推荐阅读顺序 |
|------|------|-------------|
| [README.md](README.md) | Wiki 首页和导航 | 第 1 步 |
| [02_Architecture.md](02_Architecture.md) | 详细架构说明 | 第 2 步 |
| [03_N-API_Reference.md](03_N-API_Reference.md) | 完整 API 文档 | 第 3 步 |
| [04_Inner_API.md](04_Inner_API.md) | Native 接口 | 第 4 步 (系统开发) |
| [05_Security_Review.md](05_Security_Review.md) | 安全分析 | 按需 |
| [07_Build.md](07_Build.md) | 构建配置 | 按需 |
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 调用链 | 按需 |

---

## 10. 更新记录

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-06 | 4.0 | 初始 Wiki 创建 |
| 2026-02-07 | 4.0 | 补充 ASSESSMENT.md 和 Overview.md |

---

*文档生成时间: 2026-02-07*  
*基于代码版本: @ohos/netstack v4.0*
