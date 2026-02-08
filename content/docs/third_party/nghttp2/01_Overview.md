# nghttp2 库概览

## 原始库简介

### 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | nghttp2 |
| **版本** | v1.66.0 |
| **许可证** | MIT License |
| **上游地址** | https://nghttp2.org |
| **源码仓库** | https://github.com/nghttp2/nghttp2 |

### 功能描述

nghttp2 是 **HTTP/2 协议** 及其头部压缩算法 **HPACK** 的 C 语言实现，提供以下功能：

1. **libnghttp2** - 可复用的 C 库，实现 HTTP/2 帧层
2. **nghttp** - HTTP/2 客户端
3. **nghttpd** - HTTP/2 服务器
4. **nghttpx** - HTTP/2 代理服务器（支持 HTTP/2, HTTP/1.1, SPDY）
5. **h2load** - HTTP/2 负载测试工具
6. **HPACK 工具** - 头部压缩编解码器

### 技术特性

- 完整的 HTTP/2 帧层实现（RFC 9113）
- HPACK 头部压缩（RFC 7541）
- 支持 TLS + ALPN (Application-Layer Protocol Negotiation)
- 支持 HTTP Upgrade (h2c)
- 服务器推送
- 流优先级
- 流控制

### 原始库文件结构

```
nghttp2/
├── lib/                    # 核心库
│   ├── includes/nghttp2/   # 公共头文件
│   └── *.c                 # 源文件 (26个)
├── src/                    # 应用程序源码
│   ├── nghttp.cc           # 客户端
│   ├── nghttpd.cc          # 服务器
│   └── nghttpx.cc          # 代理
├── tests/                  # 测试用例
├── third-party/            # 第三方组件
│   └── llhttp/             # HTTP/1.1 解析器
└── examples/               # 示例代码
```

---

## OpenHarmony 中的定位

### OH 组件信息

| 属性 | 值 |
|------|-----|
| **OH 组件名** | @ohos/nghttp2 |
| **OH 版本** | 3.1 |
| **所属子系统** | thirdparty |
| **发布方式** | code-segment |
| **目标路径** | third_party/nghttp2 |
| **适配系统** | small (LiteOS), standard (Linux/Mac) |

### 在 OH 中的作用

在 OpenHarmony 中，nghttp2 的核心作用是 **为 curl 提供 HTTP/2 协议支持**：

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用层                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  网络请求   │  │  下载管理   │  │   ArkUI-X 跨平台    │ │
│  │   API      │  │             │  │        应用         │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
└─────────┼────────────────┼────────────────────┼────────────┘
          │                │                    │
          ▼                ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                         curl 库                             │
│           (使用 nghttp2 实现 HTTP/2 功能)                   │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       libnghttp2                            │
│         (HTTP/2 帧层、HPACK 编解码、流管理)                 │
└─────────────────────────────────────────────────────────────┘
```

### 典型使用场景

1. **HTTP/2 客户端请求**
   - 应用通过 curl API 发起 HTTPS 请求
   - curl 检测到服务器支持 HTTP/2 (通过 ALPN)
   - curl 调用 nghttp2 进行 HTTP/2 帧编码/解码
   - 实现多路复用、头部压缩等功能

2. **ArkUI-X 跨平台应用**
   - 跨平台框架使用 curl 作为网络层
   - 自动支持 HTTP/2 协议

3. **系统服务**
   - 系统更新服务、OTA 下载
   - 应用商店下载
   - 云服务同步

---

## 版本信息

### 当前版本

```c
// lib/includes/nghttp2/nghttp2ver.h
#define NGHTTP2_VERSION "1.66.0"
#define NGHTTP2_VERSION_NUM 0x014200
```

### 版本历史

| 版本 | 日期 | 说明 |
|------|------|-----|
| v1.66.0 | 2024年 | 当前版本 |

### 与上游版本同步状态

- 当前版本较新 (v1.66.0)
- 无 OH 特定的代码修改
- 仅构建系统适配 (BUILD.gn)

---

## 许可证合规

### 许可证信息

- **许可证**: MIT License
- **许可证文件**: `COPYING`
- **SPDX 标识**: MIT

### OH 合规性

- ✅ 许可证在 bundle.json 中正确声明
- ✅ COPYING 文件存在
- ✅ 源代码包含版权声明
- ✅ 通过 OAT (OSS Audit Tool) 审核

---

## 相关资源

### 官方文档

- **主页**: https://nghttp2.org
- **GitHub**: https://github.com/nghttp2/nghttp2
- **文档**: https://nghttp2.org/documentation/

### 规范参考

- **HTTP/2**: RFC 9113 (原 RFC 7540)
- **HPACK**: RFC 7541

### 测试服务器

- https://nghttp2.org/ - 支持 HTTP/2 + HTTP/3

---

## 总结

nghttp2 在 OpenHarmony 中是一个**基础设施库**，不直接暴露给应用开发者，而是通过 curl 间接使用。其特点是：

1. **无侵入式修改** - 没有 OH 特定的代码 patch
2. **完整的 HTTP/2 实现** - 支持所有核心特性
3. **良好的维护状态** - 版本较新，社区活跃
4. **单一主要依赖者** - 目前只有 curl 依赖此库

下一节：[02_Patches.md](./02_Patches.md) - 详细分析唯一的 patch 文件
