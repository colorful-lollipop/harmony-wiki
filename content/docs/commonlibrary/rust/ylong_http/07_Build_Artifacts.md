# 编译产物

> 本文档说明 ylong_http 项目生成的编译产物、类型和安装位置

---

## 目的

本文档的目的是让读者在 10 分钟内了解：
- GN 构建系统生成的产物类型
- 输出文件的位置和命名规则
- 运行时加载关系
- 动态库和静态库的区别

## 适用范围

- ylong_http 静态库
- ylong_http_client 共享库
- 单元测试可执行文件

---

## 关键结论

### 1. 产物类型

| Target | GN 类型 | Rust Crate Type | 输出文件 | 证据 |
|--------|---------|----------------|----------|------|
| `ylong_http` | `ohos_rust_static_library` | staticlib | `libylong_http.a` | ylong_http/BUILD.gn:17 |
| `ylong_http_client_inner` | `ohos_rust_shared_library` | dylib | `libylong_http_client_inner.so` | ylong_http_client/BUILD.gn:17 |
| `rust_ylong_http_test_ut` | `ohos_rust_unittest` | bin | 测试可执行文件 | ylong_http/BUILD.gn:35 |
| `rust_ylong_http_client_test_ut` | `ohos_rust_unittest` | bin | 测试可执行文件 | ylong_http_client/BUILD.gn:44 |

### 2. 输出路径（OpenHarmony）

基于 OpenHarmony 构建系统，产物将被安装到以下路径：

| 产物类型 | 预期输出路径 | 安装前缀 |
|-----------|-------------|---------|
| 静态库（.a） | `/usr/lib/` 或 `/system/lib/` | `libylong_http.a` |
| 共享库（.so） | `/system/lib64/` 或 `/vendor/lib64/` | `libylong_http_client_inner.so` |
| 测试可执行文件 | `/data/tests/ylong_http/` 或输出目录 | 测试程序 |

**注意**：具体路径取决于设备架构（arm64-v8a, x86_64 等）

**证据**: OpenHarmony 构建系统标准产物位置

### 3. 运行时加载关系

```mermaid
graph LR
    App[上层应用] -->|netstack|
    |netstack| -->|ylong_http_client_inner.so|
    |ylong_http_client_inner.so| -->|ylong_runtime.so|
    |ylong_http_client_inner.so| -->|libylong_http.a|
    |ylong_http_client_inner.so| -->|openssl:libssl.so|
    |ylong_http_client_inner.so| -->|openssl:libcrypto.so|
    |ylong_http_client_inner.so| -->|libc.so|
```

**加载顺序**：
1. **ylong_http_client_inner.so** 被上层应用动态加载
2. **libylong_http.a** 被共享库静态链接
3. **ylong_runtime.so** 提供异步运行时
4. **openssl:libssl.so** 和 **libcrypto.so** 提供 TLS/SSL 功能
5. **libc.so** 提供 C 标准库绑定

**证据**:
- `ylong_http_client/BUILD.gn:35-40` - `external_deps` 配置
- `ylong_http/BUILD.gn:32` - `external_deps` 配置

### 4. OpenSSL 依赖

**OpenSSL 共享库**：
- `libssl.so` - OpenSSL SSL/TLS 库
- `libcrypto.so` - OpenSSL 加密库

**OpenSSL 版本**: 3.0（通过 `c_openssl_3_0` feature 确认）

**证据**:
- `ylong_http_client/BUILD.gn:37-39` - `"openssl:libssl_shared"`, `"openssl:libcrypto_shared"`
- `ylong_http_client/Cargo.toml:15` - `c_openssl_3_0` feature

### 5. HTTP/3 依赖

**Quiche 库**（可选，未在 GN 中启用）：
- 通过 `quiche` crate 提供 QUIC 协议实现
- 版本: 0.22.0
- 用于 HTTP/3 实现

**证据**:
- `ylong_http_client/Cargo.toml:15` - `quiche = { version = "0.22.0", optional = true }`
- `ylong_http/src/h3/` - HTTP/3 模块存在

**注意**：HTTP/3 功能未在 GN 配置中启用（无 `http3` feature），需要在 Cargo.toml 中手动启用

### 6. 构建配置选项

#### 6.1 Rust Edition

**版本**: 2021 Edition

**证据**:
- `ylong_http/BUILD.gn:22` - `edition = "2021"`
- `ylong_http_client/BUILD.gn:22` - `edition = "2021"`

#### 6.2 Feature Flags

**编译时 Feature 传递**：
- GN 的 `rustflags` 配置使用 `--cfg=feature="..."` 格式传递给 Rust 编译器

**主要 Features**：
- `http1_1` - HTTP/1.1 支持（默认启用）
- `http2` - HTTP/2 支持（默认启用）
- `huffman` - Huffman 编码（默认启用）
- `async` - 异步客户端（客户端默认启用）
- `ylong_base` - 使用 ylong_runtime（客户端默认启用）
- `c_openssl_3_0` - OpenSSL 3.0 TLS（客户端默认启用）
- `__tls` - TLS 支持（内部标记）
- `__c_openssl` - OpenSSL 集成（内部标记）

**可选 Features**：
- `http3` - HTTP/3 支持（代码已实现）
- `tokio_base` - 使用 tokio 运行时（替代 ylong_runtime）
- `sync` - 同步客户端（可通过 Cargo.toml 启用）

**证据**:
- `ylong_http/BUILD.gn:24-26` - features 配置
- `ylong_http_client/BUILD.gn:24-25` - features 配置

#### 6.3 符号导出

**ylong_http（静态库）**：
- 所有公共 API 通过 `pub` 关键字导出
- 不使用 `#[no_mangle]`（纯 Rust 库）

**ylong_http_client（共享库）**：
- 所有公共 API 通过 `pub` 关键字导出
- 不使用 `#[no_mangle]`（纯 Rust 库）

### 7. 构建命令

**使用 GN 构建**：
```bash
# 构建静态库
gn build //commonlibrary/rust/ylong_http/ylong_http:ylong_http

# 构建共享库
gn build //commonlibrary/rust/ylong_http/ylong_http_client:ylong_http_client_inner

# 运行单元测试
gn test //commonlibrary/rust/ylong_http/ylong_http_client:unittest
```

**使用 Cargo 构建**：
```bash
# 进入 ylong_http 目录
cd ylong_http
cargo build --features "http1_1,http2,ylong_base"

# 进入 ylong_http_client 目录
cd ylong_http_client
cargo build --features "async,http1_1,http2,ylong_base,c_openssl_3_0"
```

**证据**: README.md:67-80`

### 8. 测试产物

**测试可执行文件**：
- `rust_ylong_http_test_ut` - ylong_http 的单元测试
- `rust_ylong_http_client_test_ut` - ylong_http_client 的单元测试

**测试运行**：
```bash
# 运行所有测试
gn test //commonlibrary/rust/ylong_http/ylong_http_client:unittest
```

**证据**: ylong_http_client/BUILD.gn:69-78`

---

## 相关跳转

- **[GN 目标梳理](06_GN_Targets.md)** - GN targets 和配置详情
- **[对外 API](04_External_API.md)** - 公共 API 使用指南

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
