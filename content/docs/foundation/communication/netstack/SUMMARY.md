# OpenHarmony NetStack Wiki 导航

## 新人阅读路线

### 第一阶段：快速入门（约 30 分钟）
1. [README.md](README.md) - Wiki 首页
2. [01_Overview.md](01_Overview.md) - 项目概览与快速开始
3. [02_Architecture.md](02_Architecture.md) - 架构与核心概念

### 第二阶段：API 学习（约 1-2 小时）
4. [03_N-API_Reference.md](03_N-API_Reference.md) - JS API 接口
   - HTTP 模块详解
   - Socket 模块详解
   - WebSocket 模块详解

### 第三阶段：深入理解（约 2-3 小时）
5. [04_Inner_API.md](04_Inner_API.md) - Native 接口
   - http_client 核心类
   - SSL/TLS 安全层
   - 线程模型

### 第四阶段：构建与部署（约 1 小时）
5. [04_Build_Targets.md](04_Build_Targets.md) - GN 构建系统
   - Target 清单
   - 编译产物

### 第五阶段：安全评估（约 2 小时）
6. [05_Security_Review.md](05_Security_Review.md) - 安全风险分析

---

## 文档索引

### 核心文档
| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [README.md](README.md) | Wiki 首页与导航 | 所有开发者 |
| [01_Overview.md](01_Overview.md) | 项目概览与快速开始 | 新人开发者 |
| [02_Architecture.md](02_Architecture.md) | 系统架构与组件关系 | 架构师、中高级开发者 |
| [03_N-API_Reference.md](03_N-API_Reference.md) | JS API 接口文档 | 应用开发者 |
| [04_Inner_API.md](04_Inner_API.md) | Native 接口文档 | 系统开发者 |
| [05_Security_Review.md](05_Security_Review.md) | 安全风险评估 | 安全工程师 |

### 附录
| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 配置开关与宏 |

---

## 模块速查

### HTTP 模块
- **命名空间**: `@ohos.net.http`
- **入口函数**: `createHttp()`
- **核心类**: `HttpRequest`
- **N-API 注册**: `http_module.cpp:552`
- **Native 实现**: `http_client.h` (HttpSession)

### Socket 模块
- **命名空间**: `@ohos.net.socket`
- **入口函数**: `constructTCPSocketInstance()`, `constructUDPSocketInstance()`, `constructTLSSocketInstance()`
- **Socket 类型**: TCP, UDP, TLS, Multicast, LocalSocket
- **Native 实现**: `tls_socket.h`, `socket_module.cpp`

### WebSocket 模块
- **命名空间**: `@ohos.net.webSocket`
- **入口函数**: `createWebSocket()`
- **核心类**: `WebSocket`
- **Native 实现**: `websocket_client_innerapi.h`

---

## 版本兼容性
- **OpenHarmony**: 标准系统 (standard)
- **API Version**: 6+ (基础), 9+ (增强), 10+ (跨平台)
- **ArkTS**: 支持 (部分 API)

---

## 贡献指南
- 所有文档基于代码证据编写
- N-API 文档需包含 `.d.ts` 声明路径
- Native 文档需包含 `.h` 头文件路径
- 安全分析需包含具体代码位置
