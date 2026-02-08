# 项目定位与核心能力

## 项目概述

`netmanager_cangjie_wrapper` 是 OpenHarmony 网络管理子系统的仓颉（Cangjie）语言封装层，提供统一的网络管理能力和 HTTP 数据请求能力。

**证据来源**：`README.md:1-5`

## 项目定位

本项目属于 OpenHarmony `communication` 子系统的网络管理组件：

| 维度 | 描述 |
|------|------|
| **子系统** | communication |
| **组件** | netmanager_cangjie_wrapper |
| **版本** | 6.1 |
| **ROM 占用** | ~550KB |
| **RAM 占用** | ~524KB |
| **系统类型** | standard (标准设备) |

**证据来源**：`bundle.json:2-19`

## 核心能力

### 网络连接管理

提供网络连接的统一管理能力，包括：

| 能力 | 功能描述 | API 级别 |
|------|---------|---------|
| 网络发现 | 创建网络连接、获取默认网络 | 22+ |
| 网络查询 | 获取网络状态、属性、能力信息 | 22+ |
| 网络绑定 | 将应用绑定到指定网络 | 22+ |
| DNS 解析 | 主机名到 IP 地址的解析 | 22+ |
| 事件监听 | 监听网络状态变化事件 | 22+ |

### HTTP 数据请求

提供 HTTP/HTTPS 请求能力，包括：

| 能力 | 功能描述 | API 级别 |
|------|---------|---------|
| 请求发起 | 支持多种 HTTP 方法和请求选项 | 22+ |
| 响应处理 | 获取响应头、状态码、响应体 | 22+ |
| 流式处理 | 支持响应流式读取 | 22+ |
| 进度监听 | 下载/上传进度回调 | 22+ |
| 响应缓存 | HTTP 响应缓存管理 | 22+ |

**证据来源**：`README.md:52-55`

## 系统能力要求

### 必备系统能力

```typescript
// 网络管理核心能力
SystemCapability.Communication.NetManager.Core

// HTTP 栈能力
SystemCapability.Communication.NetStack
```

### 权限要求

| 权限 | 用途 | 敏感级别 |
|------|------|---------|
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | normal |
| `ohos.permission.INTERNET` | 互联网访问 | normal |

**证据来源**：`net_connection.cj:63-64`, `:150`, `:174-175`, `:205`, `:274`, `:300`, `:327-328`, `:352`, `:378`

## 当前限制

以下 ArkTS 支持的功能在仓颉版本中尚未实现：

| 功能 | 状态 |
|------|------|
| 以太网连接管理 | ❌ 不支持 |
| mDNS 管理 | ❌ 不支持 |
| 网络策略管理 | ❌ 不支持 |
| Socket 连接 | ❌ 不支持 |
| 流量管理 | ❌ 不支持 |
| 网络共享 | ❌ 不支持 |
| VPN 管理 | ❌ 不支持 |
| WebSocket 连接 | ❌ 不支持 |
| 网络防火墙 | ❌ 不支持 |
| 网络安全 | ❌ 不支持 |
| 可扩展身份验证 | ❌ 不支持 |

**证据来源**：`README.md:65-79`

## 依赖关系

### 内部模块依赖

```
kit/NetworkKit/
    └── 导出:
        ├── ohos.net.*
        ├── ohos.net.connection.*
        └── ohos.net.http.*

ohos/net/
    ├── net.cj (包定义)
    ├── connection/ (网络连接模块)
    │   ├── net_connection.cj
    │   ├── connection_ffi.cj
    │   └── connection_common.cj
    └── http/ (HTTP 请求模块)
        ├── http.cj
        ├── http_ffi.cj
        └── http_common.cj
```

### 外部组件依赖

| 依赖组件 | 用途 | 类型 |
|---------|------|------|
| `cangjie_ark_interop` | 仓颉互操作框架 | 系统组件 |
| `hiviewdfx_cangjie_wrapper` | 日志接口 | 系统组件 |
| `netmanager_base` | 网络管理 C 接口 | 子系统组件 |
| `netstack` | HTTP 栈 C 接口 | 子系统组件 |

**证据来源**：`bundle.json:23-29`

## 相关文档

- [API 参考](03_API_Reference.md) - 详细的接口说明
- [系统架构](02_Architecture.md) - 架构设计与数据流
- [FFI 接口](04_FFI_Interface.md) - 底层 FFI 定义
