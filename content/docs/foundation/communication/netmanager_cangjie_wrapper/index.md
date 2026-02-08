# NetManager Cangjie Wrapper

> OpenHarmony 网络管理的仓颉语言封装层

## 项目定位

本项目是 OpenHarmony 网络管理子系统 (`communication/netmanager`) 的仓颉（Cangjie）语言封装层，为仓颉应用提供：

- **网络连接管理**：查询网络状态、管理网络连接、监听网络事件
- **HTTP 数据请求**：发起 HTTP/HTTPS 请求、管理响应缓存

## 核心能力

### 网络连接管理 (`ohos.net.connection`)

| 能力 | 描述 | API 级别 |
|------|------|---------|
| 网络发现 | 创建网络连接、获取默认网络 | 22+ |
| 网络查询 | 获取连接属性、网络能力、代理设置 | 22+ |
| 网络绑定 | 绑定进程/应用到指定网络 | 22+ |
| DNS 解析 | 根据主机名获取 IP 地址 | 22+ |
| 事件监听 | 监听网络可用/丢失/能力变化事件 | 22+ |

### HTTP 请求 (`ohos.net.http`)

| 能力 | 描述 | API 级别 |
|------|------|---------|
| 请求发起 | GET/POST 等 HTTP 方法支持 | 22+ |
| 请求选项 | 超时、缓存、代理、请求头等 | 22+ |
| 响应处理 | 响应头、响应体、状态码 | 22+ |
| 进度监听 | 下载/上传进度回调 | 22+ |
| 流式支持 | `requestInStream` 支持流式响应 | 22+ |
| 响应缓存 | HTTP 响应缓存管理 | 22+ |

## 运行环境要求

| 要求 | 规格 |
|------|------|
| OpenHarmony 版本 | 标准系统 (standard) |
| API 级别 | 22+ |
| 仓颉编译器 | 支持 Cangjie FFI |
| 依赖子系统 | communication_netmanager_base, communication_netstack |

## 快速开始

```cangjie
// 导入 NetworkKit
import kit.NetworkKit

// 发起 HTTP 请求
let http = createHttp()
http.request(url: "https://example.com", options: HttpRequestOptions(), callback: {
    (error: ?BusinessException, response: ?HttpResponse) =>
    match (error) {
        case Some(e) => println("Error: ${e.code}")
        case None => println("Status: ${response!.responseCode}")
    }
})
```

## 项目结构

```
netmanager_cangjie_wrapper/
├── kit/NetworkKit/           # 对外交互层 (Kit)
│   └── index.cj              # 模块导出入口
├── ohos/net/                 # 内部实现层
│   ├── net.cj                # 包定义
│   ├── connection/           # 网络连接模块
│   │   ├── net_connection.cj     # 连接 API 实现
│   │   ├── connection_ffi.cj     # FFI 接口定义
│   │   └── connection_common.cj  # 公共类型
│   └── http/                  # HTTP 请求模块
│       ├── http.cj               # HTTP API 实现
│       ├── http_ffi.cj           # FFI 接口定义
│       └── http_common.cj        # 公共类型
├── mock/                      # Mock 文件 (测试用)
└── test/                      # 测试用例
```

## 相关文档

| 主题 | 链接 |
|------|------|
| API 参考 | [API_Reference](03_API_Reference.md) |
| 架构说明 | [Architecture](02_Architecture.md) |
| 构建配置 | [GN_Build](05_GN_Build.md) |
| 安全评审 | [Security_Review](06_Security_Review.md) |

## 官方资源

- [NetworkKit API 参考 (仓颉)](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/NetworkKit/)
- [Network 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/network/)
