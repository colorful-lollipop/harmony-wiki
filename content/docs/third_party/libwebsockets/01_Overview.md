# 01 - 原始库简介

## 库基本信息

| 属性 | 内容 |
|-----|------|
| **名称** | libwebsockets (LWS) |
| **版本** | 4.3.3 |
| **许可证** | MIT License |
| **上游地址** | https://libwebsockets.org |
| **Git 仓库** | https://libwebsockets.org/git/libwebsockets |

## 原始功能

libwebsockets 是一个灵活、轻量级的纯 C 库，用于实现现代网络协议：

- **WebSocket**: RFC 6455 客户端和服务器
- **HTTP/1**: 完整的 HTTP/1.1 支持
- **HTTP/2**: 基于 nghttp2 的实现
- **MQTT**: MQTT 客户端支持
- **TLS/SSL**: 支持 OpenSSL 和 MbedTLS v2/v3
- **事件循环**: 支持 libuv、libevent、libev、sdevent、glib、uloop

## 架构特点

### 角色（Roles）设计
libwebsockets 使用模块化角色设计，每个协议实现为一个 role：
- `http`: HTTP 协议处理
- `h1`: HTTP/1.x 具体实现
- `h2`: HTTP/2 实现
- `ws`: WebSocket 协议
- `raw`: 原始 socket
- `mqtt`: MQTT 协议
- `netlink`: Linux Netlink（系统级网络接口）

### 平台抽象层
- `plat/unix`: Unix/Linux 平台
- `plat/windows`: Windows 平台
- `plat/freertos`: FreeRTOS 嵌入式平台

### 核心模块
- `core`: 基础功能和上下文管理
- `core-net`: 网络相关核心功能
- `tls`: TLS/SSL 抽象层
- `misc`: 辅助工具（JSON、base64、SHA-1 等）

## 在 OpenHarmony 中的作用和定位

### 功能定位

libwebsockets 是 OpenHarmony **netstack 子系统** 的核心依赖，提供：

1. **WebSocket 协议栈**: 为应用层提供 WebSocket 客户端功能
2. **HTTP 客户端**: 支持基于 HTTP 的网络请求
3. **TLS 加密**: 通过 OpenSSL 提供安全的网络通信

### 架构位置

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Application)                    │
│  JS/ArkTS API    Cangjie API    C API                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   NetStack 子系统                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ WebSocket   │  │ HTTP        │  │ Socket      │          │
│  │ Manager     │  │ Client      │  │ API         │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              libwebsockets (本库)                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ WebSocket   │  │ HTTP/1      │  │ HTTP/2      │          │
│  │ Protocol    │  │ Client      │  │ Client      │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   传输层 (Transport)                         │
│           OpenSSL (TLS)    /    系统 Socket                 │
└─────────────────────────────────────────────────────────────┘
```

### 使用场景

1. **应用开发**: 开发者使用 ArkTS/JS/Cangjie WebSocket API 进行实时通信
2. **系统服务**: 内部服务通过 WebSocket 进行消息推送
3. **开发工具**: IDE Previewer 使用 WebSocket 进行设备通信

### OH 特有的简化

在 OpenHarmony 中，libwebsockets 的配置相对精简：

- **启用功能**: WebSocket 客户端、HTTP/1、HTTP/2、TLS
- **禁用功能**: MQTT、服务器模式（使用系统其他组件）、Netlink（iOS）
- **平台支持**: Linux/Android（标准系统）、iOS（ArkUI-X）

### 关键价值

1. **成熟稳定**: 11 年开发历史，4.3K+ patches，广泛生产验证
2. **轻量高效**: 适合资源受限的嵌入式/移动设备
3. **协议完整**: 完整的 WebSocket RFC 6455 实现
4. **安全合规**: 支持现代 TLS 版本，安全审计充分

---

## 参考链接

- 上游官网: https://libwebsockets.org
- API 文档: https://libwebsockets.org/lws-api-doc-main/html/index.html
- 示例代码: https://libwebsockets.org/git/libwebsockets/tree/minimal-examples
- README 集合: https://libwebsockets.org/git/libwebsockets/tree/READMEs

---

*本文档重点在于说明 libwebsockets 在 OpenHarmony 中的定位和作用。详细的技术文档请参考上游官方文档。*
