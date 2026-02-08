# 文档导航

本文档为 `netmanager_cangjie_wrapper` 项目的工程 Wiki，提供完整的技术参考。

## 新人阅读路线

```
1️⃣ 入门阶段
   └── index.md (项目首页)
       └── 01_Overview.md (项目定位与核心能力)
           └── 02_Architecture.md (系统架构说明)

2️⃣ API 学习阶段
   └── 03_API_Reference.md (Cangjie API 参考)
       └── 04_FFI_Interface.md (FFI 接口与数据类型)

3️⃣ 开发扩展阶段
   └── 05_GN_Build.md (构建配置与产物)
       └── 06_Security_Review.md (安全风险评审)

4️⃣ 深入了解 (可选)
   └── appendix/Callgraphs.md (关键调用链)
```

## 完整目录

### 核心文档

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README](README.md) | Wiki 说明与更新方式 | 📖 必读 |
| [index](index.md) | 项目首页 | 📖 必读 |
| [01_Overview](01_Overview.md) | 项目定位与核心能力 | 🔥 高 |
| [02_Architecture](02_Architecture.md) | 系统架构说明 | 🔥 高 |
| [03_API_Reference](03_API_Reference.md) | Cangjie API 参考 | 🔥 高 |
| [04_FFI_Interface](04_FFI_Interface.md) | FFI 接口与数据类型 | 📚 中 |
| [05_GN_Build](05_GN_Build.md) | 构建配置与产物 | 📚 中 |
| [06_Security_Review](06_Security_Review.md) | 安全风险评审 | ⚠️ 重要 |

### 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |

## 模块索引

### 网络连接管理 (ohos.net.connection)

| API | 描述 | 权限要求 |
|-----|------|---------|
| `createNetConnection` | 创建网络连接 | - |
| `getDefaultNet` | 获取默认网络 | `ohos.permission.GET_NETWORK_INFO` |
| `getAllNets` | 获取所有已激活网络 | `ohos.permission.GET_NETWORK_INFO` |
| `getConnectionProperties` | 获取连接属性 | `ohos.permission.GET_NETWORK_INFO` |
| `getNetCapabilities` | 获取网络能力 | `ohos.permission.GET_NETWORK_INFO` |
| `setAppNet` | 绑定进程到网络 | `ohos.permission.INTERNET` |

### HTTP 请求 (ohos.net.http)

| API | 描述 | 权限要求 |
|-----|------|---------|
| `createHttp` | 创建 HTTP 请求任务 | - |
| `createHttpResponseCache` | 创建响应缓存 | - |
| `HttpRequest.request` | 发起 HTTP 请求 | `ohos.permission.INTERNET` |
| `HttpRequest.requestInStream` | 流式 HTTP 请求 | `ohos.permission.INTERNET` |

## 快速跳转

- **架构图** → [02_Architecture.md](02_Architecture.md)
- **API 列表** → [03_API_Reference.md](03_API_Reference.md)
- **错误码** → [03_API_Reference.md#错误码](03_API_Reference.md#错误码)
- **构建配置** → [05_GN_Build.md](05_GN_Build.md)
- **安全说明** → [06_Security_Review.md](06_Security_Review.md)

## 版本信息

- **当前版本**：6.1
- **API 级别**：22+
- **最后更新**：2024-02-06
