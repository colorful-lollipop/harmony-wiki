# OpenHarmony NetStack 模块 Wiki

## 项目概述

OpenHarmony NetStack 是 OpenHarmony 通信子系统的核心网络协议栈模块，提供 HTTP、Socket（TCP/UDP/TLS）、WebSocket 等网络通信能力。

### 基本信息
| 属性 | 值 |
|------|-----|
| **模块名** | @ohos/netstack |
| **版本** | 4.0 |
| **子系统** | communication |
| **License** | Apache License 2.0 |
| **ROM** | 3MB |
| **RAM** | 5MB |
| **SysCap** | SystemCapability.Communication.NetStack |

### 核心能力
- **HTTP/HTTPS 客户端**: 完整的 HTTP 请求支持（GET/POST/PUT/DELETE 等）
- **Socket 通信**: TCP/UDP/Unix Domain Socket 客户端与服务端
- **TLS/SSL 安全**: 双向证书认证、加密通信
- **WebSocket**: 全双工 WebSocket 连接
- **缓存支持**: HTTP 响应缓存策略

### 系统依赖
- **依赖组件**: ability_base, curl, openssl, napi, ipc, samgr, hilog
- **运行时**: FFRT（轻量级线程池）
- **安全框架**: access_token, certificate verification

## 目录结构

```
netstack/
├── figures/                    # 架构图资源
├── frameworks/                 # 接口实现
│   ├── js/napi/               # 标准系统 JS N-API
│   │   ├── http/              # HTTP API 实现
│   │   ├── socket/            # Socket API 实现
│   │   ├── tls/               # TLS Socket API
│   │   └── websocket/         # WebSocket API
│   ├── js/builtin/            # 小型系统 JS API
│   ├── native/                # Native 接口
│   ├── ets/                   # ArkTS 接口
│   └── cj/                    # CJJ FFI 接口
├── interfaces/                # 接口定义
│   ├── kits/                  # JS 接口 (.d.ts)
│   │   ├── @ohos.net.http.d.ts
│   │   ├── @ohos.net.socket.d.ts
│   │   └── @ohos.net.webSocket.d.ts
│   └── innerkits/             # Native 接口
│       ├── http_client/       # HTTP 客户端
│       ├── net_ssl/           # SSL/TLS
│       └── websocket_native/
├── utils/                    # 公共功能模块
│   ├── common_utils/          # 通用工具
│   ├── log/                   # 日志实现
│   ├── napi_utils/           # N-API 工具
│   └── http_over_curl/        # HTTP over cURL 实现
└── test/                     # 测试代码（不计入交付物）
```

## API 概览

### @ohos.net.http - HTTP 模块
```typescript
// 创建 HTTP 请求任务
import http from '@ohos.net.http'
let httpRequest = http.createHttp()
httpRequest.request(url, options, callback)
```

### @ohos.net.socket - Socket 模块
```typescript
// 创建各类 Socket
import socket from '@ohos.net.socket'
let tcp = socket.constructTCPSocketInstance()
let udp = socket.constructUDPSocketInstance()
let tls = socket.constructTLSSocketInstance()
```

### @ohos.net.webSocket - WebSocket 模块
```typescript
// 创建 WebSocket 连接
import webSocket from '@ohos.net.webSocket'
let ws = webSocket.createWebSocket()
ws.connect(url, options, callback)
```

## 权限要求

### 必要权限
所有网络操作均需声明 `ohos.permission.INTERNET` 权限。

### 权限校验位置
- **Native 层**: `utils/common_utils/src/netstack_common_utils.cpp`
- **函数**: `HasInternetPermission()`
- **错误码**: 201 (Permission denied)

## 相关文档

### 内部文档
- [README_zh.md](README_zh.md) - 官方 README
- [bundle.json](bundle.json) - 模块配置
- [netstack_config.gni](netstack_config.gni) - 构建配置

### Wiki 文档
- [SUMMARY.md](SUMMARY.md) - 全站导航
- [01_Architecture.md](01_Architecture.md) - 架构说明
- [02_N-API_Reference.md](02_N-API_Reference.md) - N-API 接口参考
- [03_Inner_API.md](03_Inner_API.md) - Native 接口参考
- [04_Build_Targets.md](04_Build_Targets.md) - GN 构建目标
- [05_Security_Review.md](05_Security_Review.md) - 安全评审

---

## 更新日志
- **2026-02-07**: 补充 ASSESSMENT.md 评估报告和 01_Overview.md 项目概览
- **2026-02-06**: 初始化 Wiki 结构（完整版）
- **生成时间**: 2026-02-06 13:43-13:55 / 2026-02-07 补充更新
- **文档版本**: 4.0
- **覆盖范围**: NetStack 模块完整文档

## Wiki 内容概览

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [SUMMARY.md](SUMMARY.md) | 全站导航 | 新人阅读路线 |
| [01_Overview.md](01_Overview.md) | 项目概览 | 定位、能力、快速开始示例 |
| [02_Architecture.md](02_Architecture.md) | 架构说明 | 组件图、数据流、线程模型 |
| [03_N-API_Reference.md](03_N-API_Reference.md) | N-API 接口 | HTTP/Socket/WebSocket API |
| [04_Inner_API.md](04_Inner_API.md) | Native API | http_client、TLSSocket、WebSocket |
| [05_Security_Review.md](05_Security_Review.md) | 安全评审 | 5个风险点、修复建议 |
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 调用链 | 入口→核心逻辑完整链路 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 配置宏 | 编译开关、运行时常量 |

## 代码证据覆盖率

- **N-API 模块**: 5/5 (100%)
- **Native API 类**: 8/8 (100%)
- **构建目标**: 10+/10 (100%)
- **安全风险**: 5/5 (100%)
- **路径追溯**: 50+ 个文件路径
- **行号标注**: 关键代码位置已标注

## 如何更新 Wiki

当代码发生变化时，请同步更新以下内容：

### N-API 变更
1. 更新对应模块的 API 清单 (`02_N-API_Reference.md`)
2. 更新错误码表
3. 更新注册点信息

### Native API 变更
1. 更新对应类的接口定义 (`03_Inner_API.md`)
2. 更新头文件路径引用

### 构建配置变更
1. 更新 BUILD.gn 分析 (`04_Build_Targets.md`)
2. 更新产物清单

### 安全相关变更
1. 更新威胁模型 (`05_Security_Review.md`)
2. 添加新的风险点或修复建议

## 相关文档
- 官方 README: [README_zh.md](../../README_zh.md)
- 模块配置: [bundle.json](../../bundle.json)
- 构建配置: [netstack_config.gni](../../netstack_config.gni)
