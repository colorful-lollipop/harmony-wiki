# OpenSSL 在 OpenHarmony 中的使用

## 依赖关系概述

OpenSSL 是 OpenHarmony 第三方库生态中最基础的安全组件之一，被十个第三方模块直接依赖。这些依赖关系体现了 OpenSSL 在系统安全基础设施中的核心地位，覆盖了网络通信、数据存储、系统服务等多个关键领域。理解这些依赖关系对于评估 OpenSSL 变更的影响范围、排查安全问题以及规划版本升级都至关重要。

从依赖类型来看，大多数模块选择动态链接方式使用 OpenSSL（通过 libcrypto_shared 和 libssl_shared），这种方式有利于减少最终二进制的体积，并便于安全更新的统一推送。部分模块（如 sqlite、fsverity-utils）同时提供静态链接变体，以满足特定部署场景的需求。从功能使用角度看，网络通信模块主要依赖 libssl 的 TLS 功能，数据处理模块主要依赖 libcrypto 的加密原语，语言绑定模块则同时使用两者。

---

## 直接依赖者详情

### curl — HTTP/HTTPS 客户端

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/curl/BUILD.gn |
| OpenSSL 依赖 | libssl_shared、libcrypto_shared |
| 链接类型 | 动态链接 |
| 主要用途 | HTTPS 连接、HTTP 客户端功能 |

curl 是 OpenHarmony 系统中最重要的 HTTP/HTTPS 客户端库，被众多上层应用和网络服务使用。curl 对 OpenSSL 的依赖主要集中在 TLS 加密通信方面，包括 HTTPS 请求的握手过程、证书验证、加密数据传输等。curl 选择动态链接 OpenSSL，使得系统可以通过更新 OpenSSL 库来修复安全漏洞，而无需重新编译 curl 本身。

### grpc — 高性能 RPC 框架

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/grpc/BUILD.gn |
| OpenSSL 依赖 | libssl_shared、libcrypto_shared |
| 链接类型 | 动态链接 |
| 主要用途 | TLS 加密 RPC、证书认证 |

grpc 框架使用 OpenSSL 提供 TLS 加密的 RPC 通信能力，支持服务端和客户端的双向证书认证。grpc 的安全模型深度集成 OpenSSL，使用 libssl 处理 TLS 握手，使用 libcrypto 生成和验证证书。grpc 构建配置中定义了 OPENSSL_SUPPRESS_DEPRECATED 宏，用于抑制 OpenSSL 内部废弃 API 的警告。

### wpa_supplicant — Wi-Fi 认证

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/wpa_supplicant/wpa_supplicant-2.9_standard/BUILD.gn |
| OpenSSL 依赖 | libssl_shared、libcrypto_shared |
| 链接类型 | 动态链接 |
| 主要用途 | EAP-TLS 认证、Wi-Fi 安全 |

wpa_supplicant 是 OpenHarmony Wi-Fi 功能的核心组件，负责处理 802.11 认证协议。它使用 OpenSSL 实现 EAP-TLS（Extensible Authentication Protocol over TLS）认证，这是企业级 Wi-Fi 网络最常用的认证方式之一。wpa_supplicant 还依赖 OpenSSL 进行证书管理、密钥派生和加密操作，是设备无线网络安全的关键依赖。

### libwebsockets — WebSocket 库

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/libwebsockets/BUILD.gn |
| OpenSSL 依赖 | libssl_static、libcrypto_static（iOS）、libssl_shared、libcrypto_shared（其他） |
| 链接类型 | 混合（静态+动态） |
| 主要用途 | WSS 安全连接 |

libwebsockets 提供 WebSocket 协议实现，使用 OpenSSL 支持 WSS（WebSocket Secure）加密连接。在 iOS 平台上，libwebsockets 静态链接 OpenSSL 以满足该平台的部署要求；在其他平台上则使用动态链接方式。这种灵活的链接配置展示了 OpenSSL 多形态库支持的价值。

### sqlite — 嵌入式数据库

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/sqlite/BUILD.gn |
| OpenSSL 依赖 | libcrypto_shared、libcrypto_static、libcrypto_restool |
| 链接类型 | 混合（静态+动态） |
| 主要用途 | 数据库透明加密（TE） |

sqlite 使用 libcrypto 提供透明加密功能，保护数据库内容的机密性。sqlite 构建配置中定义了 SQLITE_HAS_CODEC 宏启用加密支持，以及 OPENSSL_SUPPRESS_DEPRECATED 宏处理废弃 API。sqlite 同时提供静态链接（sqlite_static、sqlite_sdk）和动态链接（sqlite）两种形态，以满足不同场景需求。

### cups — 打印系统

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/cups/BUILD.gn |
| OpenSSL 依赖 | libssl_shared、libcrypto_shared |
| 链接类型 | 动态链接 |
| 主要用途 | IPP 安全打印、TLS 加密 |

CUPS（Common UNIX Printing System）在 OpenHarmony 中提供打印服务支持。它使用 OpenSSL 保护 IPP（Internet Printing Protocol）通信的安全，支持加密的打印任务传输。cups 的多个子组件（cups、rastertopwg、backend、lpd 等）都依赖 OpenSSL。

### toybox — 命令行工具集

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/toybox/BUILD.gn |
| OpenSSL 依赖 | libssl_shared、libcrypto_shared |
| 链接类型 | 动态链接 |
| 主要用途 | openssl 命令行工具（SELinux 构建） |

toybox 是 Linux 常用的命令工具集合，包含数百个简化的 Unix 工具。在启用 SELinux 支持的构建中，toybox 的 openssl 命令实现依赖 OpenSSL 库，提供对称加密、摘要计算、证书操作等功能。

### fsverity-utils — 文件完整性工具

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/fsverity-utils/BUILD.gn |
| OpenSSL 依赖 | libcrypto_shared、libcrypto_static |
| 链接类型 | 混合（静态+动态） |
| 主要用途 | 文件哈希计算、fsverity 完整性验证 |

fsverity-utils 工具使用 OpenSSL 计算文件哈希值，用于 Android/HarmonyOS 系统的只读分区完整性保护。该工具使用 SHA-256 等哈希算法验证系统文件未被篡改，是系统安全启动链的重要组成部分。

### rust-openssl — Rust 语言绑定

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/rust/crates/rust-openssl/openssl-sys/BUILD.gn |
| OpenSSL 依赖 | libssl_shared、libcrypto_shared |
| 链接类型 | 动态链接 |
| 主要用途 | Rust FFI 绑定 |

rust-openssl 项目提供了 Rust 语言对 OpenSSL 的 FFI（外部函数接口）绑定，使得 Rust 代码可以调用 OpenSSL 的加密功能。这一依赖对于 OpenHarmony 上使用 Rust 开发的应用和库具有重要意义，降低了 Rust 生态接入 OpenSSL 的门槛。

### cangjie_runtime — 输入法运行时

| 属性 | 值 |
|------|-----|
| BUILD.gn 路径 | third_party/cangjie_runtime/runtime/src/CJThread/BUILD.gn |
| OpenSSL 依赖 | 仅头文件引用 |
| 链接类型 | 头文件依赖 |
| 主要用途 | 可能的加密功能扩展 |

cangjie_runtime（仓颉输入法运行时）在构建配置中引用了 OpenSSL 的 include 路径，表明可能在某些功能中使用 OpenSSL API。这一依赖目前仅限于头文件引用，不涉及库链接。

---

## 依赖关系图

```
                        OpenSSL (third_party/openssl)
                    ┌────────────────────────────────┐
                    │ libcrypto + libssl (3.0.9)     │
                    │ • 动态库: .z.so                 │
                    │ • 静态库: .a                    │
                    │ • 配置: /system/etc/openssl.cnf │
                    └────────────────────────────────┘
                                      │
           ┌────────────┬─────────────┼─────────────┬────────────┐
           ▼            ▼             ▼             ▼            ▼
      ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌─────────┐
      │  curl   │  │  grpc   │  │ wpa_sup- │  │ libweb- │  │ sqlite  │
      │ HTTP库  │  │ RPC框架 │  │ plicant  │  │sockets  │  │ 数据库  │
      └─────────┘  └─────────┘  └──────────┘  └─────────┘  └─────────┘
           │            │             │             │            │
      HTTPS/TLS    TLS/RPC      Wi-Fi认证    WSS连接    加密存储
```

---

## 链接方式说明

### 动态链接

动态链接是 OpenSSL 在 OpenHarmony 中的主要使用方式。所有依赖模块（除 libwebsockets iOS 版本外）都使用 libcrypto_shared 和 libssl_shared 动态库。动态链接的优势包括：

1. **安全更新统一**：修复 OpenSSL 安全漏洞时，只需更新动态库文件，所有依赖模块自动获得修复
2. **减少二进制体积**：多个模块共享同一份库代码，节省存储空间
3. **便于版本管理**：组件系统可以独立更新 OpenSSL 版本

### 静态链接

部分模块在特定场景下使用静态链接：

- **sqlite**：sqlite_static、sqlite_sdk 使用静态链接的 libcrypto
- **fsverity-utils**：libfsverity_utils_static 变体使用静态链接
- **libwebsockets（iOS）**：iOS 平台要求静态链接

静态链接适用于以下场景：需要独立部署的单体应用、目标平台不支持动态库、部署环境需要确定性依赖。

---

## 使用场景分类

### 网络安全通信

| 模块 | 协议 | OpenSSL 功能 |
|------|------|-------------|
| curl | HTTPS/HTTP | TLS 客户端、证书验证 |
| grpc | gRPC/TLS | TLS 双向认证 |
| libwebsockets | WSS | TLS 服务器端 |
| wpa_supplicant | 802.11 | EAP-TLS |
| cups | IPP/TLS | TLS 打印协议 |

### 数据加密存储

| 模块 | 功能 | OpenSSL API |
|------|------|-------------|
| sqlite | 透明加密 | EVP_CIPHER_* |
| fsverity-utils | 文件哈希 | EVP_Digest_* |

### 系统安全功能

| 模块 | 功能 | OpenSSL 用途 |
|------|------|-------------|
| toybox | 命令行工具 | openssl 子命令 |
| rust-openssl | 语言绑定 | FFI 接口 |

---

## API 使用注意事项

### 废弃 API 抑制

部分依赖模块（grpc、sqlite）在构建时定义了 OPENSSL_SUPPRESS_DEPRECATED 宏，用于抑制 OpenSSL 内部废弃 API 的警告。这些模块可能使用了一些在新版本中标记为 deprecated 的功能，在升级 OpenSSL 版本时需要关注兼容性问题。

### 内存安全

使用 OpenSSL API 时，依赖模块应当注意内存管理：
- 释放所有通过 OPENSSL_malloc 分配的内存
- 正确清除敏感数据（使用 OPENSSL_cleanse）
- 避免在栈上存储密钥材料

### 线程安全

OpenSSL 3.0 在线程安全方面有改进，但使用前仍需：
- 调用 OPENSSL_init_crypto 初始化库
- 确保多线程访问时已正确设置锁回调（3.0 版本自动处理）

---

## 依赖影响评估

### 高影响模块（需优先关注）

| 模块 | 影响原因 |
|------|----------|
| curl | 系统级 HTTP 客户端，影响面广 |
| wpa_supplicant | Wi-Fi 安全关键组件 |
| grpc | RPC 基础设施 |

### 中影响模块

| 模块 | 影响原因 |
|------|----------|
| libwebsockets | WebSocket 服务 |
| sqlite | 广泛应用的数据存储 |

### 低影响模块

| 模块 | 影响原因 |
|------|----------|
| toybox | 仅 SELinux 构建 |
| cangjie_runtime | 仅头文件引用 |

---

*文档版本：1.0*  
*创建日期：2026-02-08*  
*数据来源：BUILD.gn 依赖分析*