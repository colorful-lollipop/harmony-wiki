# 文档导航

> 本文档提供 ylong_http Wiki 的完整导航和新人阅读路线

---

## 新人阅读顺序

按以下顺序阅读可以快速理解项目：

1. **[项目概览](00_Overview.md)** - 了解项目定位、目标和核心功能（15 分钟）
2. **[项目定位与边界](01_Project_Position.md)** - 理解在 OpenHarmony 中的位置和依赖关系（10 分钟）
3. **[目录结构与模块职责](02_Directory_Structure.md)** - 熟悉代码组织和模块划分（20 分钟）
4. **[架构说明](03_Architecture.md)** - 理解组件交互、数据流和线程模型（30 分钟）
5. **[对外 API](04_External_API.md)** - 学习如何使用库的公共 API（40 分钟）
6. **[内部 API](05_Internal_API.md)** - 了解内部接口和可扩展点（开发者选项）（20 分钟）
7. **[GN 目标梳理](06_GN_Targets.md)** - 理解构建系统和编译产物（15 分钟）
8. **[编译产物](07_Build_Artifacts.md)** - 了解生成的库文件和安装位置（10 分钟）
9. **[安全风险评审](08_Security_Review.md)** - 了解攻击面和注意事项（20 分钟）
10. **[常见问题](09_Troubleshooting.md)** - 快速定位构建和运行时问题（按需）

**总计阅读时间**: 约 3 小时

---

## 按功能分类

### 协议实现
- [HTTP/1.1](02_Directory_Structure.md#h1-模块) - 请求/响应编解码
- [HTTP/2](02_Directory_Structure.md#h2-模块) - 帧处理、HPACK、流管理
- [HTTP/3](02_Directory_Structure.md#h3-模块) - 帧处理、QPACK、QUIC 集成

### 客户端功能
- [异步客户端](03_Architecture.md#异步客户端) - 基于 ylong_runtime/tokio 的非阻塞 API
- [同步客户端](03_Architecture.md#同步客户端) - 阻塞式 API，无运行时依赖
- [连接池](02_Directory_Structure.md#pool模块) - 连接复用和生命周期管理
- [代理支持](02_Directory_Structure.md#proxy模块) - HTTP/HTTPS 代理
- [自动重定向](02_Directory_Structure.md#redirect模块) - 3xx 状态码自动处理

### TLS 和安全
- [证书验证](08_Security_Review.md#tls证书验证机制) - 自定义验证器和公钥固定
- [OpenSSL 集成](07_Build_Artifacts.md#openssl依赖) - FFI 绑定和配置选项

### 构建和部署
- [GN 构建系统](06_GN_Targets.md) - OpenHarmony 构建配置
- [Feature Flags](appendix/Config_Flags.md) - 编译选项和功能开关
- [编译产物](07_Build_Artifacts.md) - 静态库、共享库、测试

---

## 快速参考

### 常用类型
- `Request<T>` - HTTP 请求结构
- `Response<T>` - HTTP 响应结构
- `Headers` - HTTP 头部集合
- `Method` - HTTP 方法（GET/POST/PUT/DELETE 等）
- `StatusCode` - HTTP 状态码
- `Client<Connector>` - 客户端类型
- `Body` trait - 消息体接口

### 常用配置
- `ClientConfig` - 客户端全局配置
- `TlsConfig` - TLS/SSL 配置
- `Proxy` - 代理配置
- `Redirect` - 重定向策略
- `Timeout` - 超时配置
- `Retry` - 重试策略

### 错误类型
- `HttpError` - HTTP 协议错误（ylong_http）
- `HttpClientError` - HTTP 客户端错误（ylong_http_client）
- `ErrorKind` - 错误分类

---

## 附录

- **[调用链图](appendix/Callgraphs.md)** - 关键函数调用流程图
- **[配置标志](appendix/Config_Flags.md)** - 所有 feature flags 说明

---

**文档版本**: 1.0
**最后更新**: 2026-02-06
