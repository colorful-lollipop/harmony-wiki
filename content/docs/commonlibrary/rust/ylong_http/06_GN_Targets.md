# GN 目标梳理

> 本文档详细说明 ylong_http 项目的 GN 构建系统、targets、依赖关系和编译产物

---

## 目的

本文档的目的是让读者在 15 分钟内了解：
- GN 构建系统的配置
- 所有 targets 的详细信息和依赖关系
- 编译产物的类型和位置
- Feature flags 的作用和配置

## 适用范围

- ylong_http/BUILD.gn
- ylong_http_client/BUILD.gn

---

## 关键结论

### 1. 构建系统概览

**ylong_http 使用 OpenHarmony GN 构建系统**，包含以下模板类型：

| GN 模板类型 | 用途 | 证据 |
|------------|------|------|
| `ohos_rust_static_library` | 生成 Rust 静态库（.a） | ylong_http/BUILD.gn:17 |
| `ohos_rust_shared_library` | 生成 Rust 共享库（.so） | ylong_http_client/BUILD.gn:17 |
| `ohos_rust_unittest` | 生成单元测试可执行文件 | ylong_http/BUILD.gn:35, ylong_http_client/BUILD.gn:44 |

**导入模板**：
```gn
import("//build/ohos.gni")
import("//build/test.gni")
```

**证据**: `ylong_http/BUILD.gn:14-15`, `ylong_http_client/BUILD.gn:14-15`

### 2. ylong_http Targets

#### 2.1 ylong_http 静态库

**Target 名称**: `ylong_http`

**GN 类型**: `ohos_rust_static_library`

**完整配置**（ylong_http/BUILD.gn:17-33）:
```gn
ohos_rust_static_library("ylong_http") {
  subsystem_name = "commonlibrary"
  part_name = "ylong_http"

  crate_name = "ylong_http"
  edition = "2021"

  features = [
    "http1_1",
    "huffman",
    "http2",
    "ylong_base",
  ]

  sources = [ "src/lib.rs" ]
  external_deps = [ "ylong_runtime:ylong_runtime" ]
  deps = []
  public_deps = []
  configs = []
}
```

| 配置项 | 值 | 说明 |
|---------|------|------|
| `subsystem_name` | `commonlibrary` | 所属子系统 |
| `part_name` | `ylong_http` | 部件名称 |
| `crate_name` | `ylong_http` | Rust crate 名称 |
| `edition` | `2021` | Rust Edition |
| `sources` | `src/lib.rs` | 源文件 |
| `features` | 见下表 | 编译 feature flags |
| `external_deps` | 见下表 | 外部依赖 |
| `output_name` | `ylong_http` | 输出库名（默认） |

**Features 配置**：
| Feature | 说明 | 默认启用 |
|---------|------|----------|
| `http1_1` | HTTP/1.1 支持 | ✅ 是 |
| `huffman` | Huffman 编码 | ✅ 是 |
| `http2` | HTTP/2 支持 | ✅ 是 |
| `http3` | HTTP/3 支持 | ❌ 否 |
| `tokio_base` | tokio 运行时 | ❌ 否 |
| `ylong_base` | ylong_runtime 运行时 | ✅ 是 |

**外部依赖**：
| 依赖名称 | 类型 | GN target |
|---------|------|------------|
| `ylong_runtime:ylong_runtime` | rust_library | ylong_runtime |

#### 2.2 rust_ylong_http_test_ut 单元测试

**Target 名称**: `rust_ylong_http_test_ut`

**GN 类型**: `ohos_rust_unittest`

**完整配置**（ylong_http/BUILD.gn:35-48）:
```gn
ohos_rust_unittest("rust_ylong_http_test_ut") {
  module_out_path = "ylong_http/ylong_http"

  rustflags = [
    "--cfg=feature=\"http1_1\"",
    "--cfg=feature=\"huffman\"",
    "--cfg=feature=\"http2\"",
    "--cfg=feature=\"ylong_base\""
  ]

  sources = [ "src/lib.rs" ]
  deps = []
  external_deps = [ "ylong_runtime:ylong_runtime" ]
}
```

| 配置项 | 值 | 说明 |
|---------|------|------|
| `module_out_path` | `ylong_http/ylong_http` | 测试输出路径 |
| `rustflags` | 见下表 | 传递给 Rust 编译器的 feature flags |

**Rustflags 配置**：
| Flag | 说明 |
|------|------|
| `--cfg=feature="http1_1"` | 启用 HTTP/1.1 feature |
| `--cfg=feature="huffman"` | 启用 Huffman 编码 |
| `--cfg=feature="http2"` | 启用 HTTP/2 feature |
| `--cfg=feature="ylong_base"` | 启用 ylong_runtime |

### 3. ylong_http_client Targets

#### 3.1 ylong_http_client_inner 共享库

**Target 名称**: `ylong_http_client_inner`

**GN 类型**: `ohos_rust_shared_library`

**完整配置**（ylong_http_client/BUILD.gn:17-42）:
```gn
ohos_rust_shared_library("ylong_http_client_inner") {
  part_name = "ylong_http"
  subsystem_name = "commonlibrary"

  crate_name = "ylong_http_client_inner"
  edition = "2021"

  features = [
    "async",
    "c_openssl_3_0",
    "http1_1",
    "http2",
    "ylong_base",
    "__c_openssl",
    "__tls",
  ]

  sources = [ "src/lib.rs" ]
  deps = [ "../ylong_http:ylong_http" ]
  external_deps = [
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
    "rust_libc:lib",
    "ylong_runtime:ylong_runtime"
  ]
}
```

| 配置项 | 值 | 说明 |
|---------|------|------|
| `crate_name` | `ylong_http_client_inner` | Rust crate 名称 |
| `sources` | `src/lib.rs` | 源文件 |
| `deps` | 见下表 | 内部依赖 |
| `external_deps` | 见下表 | 外部依赖 |
| `output_name` | `libylong_http_client_inner` | 输出库名（默认） |

**Features 配置**：
| Feature | 说明 | 默认启用 |
|---------|------|----------|
| `async` | 异步接口支持 | ✅ 是 |
| `c_openssl_3_0` | OpenSSL 3.0 TLS | ✅ 是 |
| `http1_1` | HTTP/1.1 支持 | ✅ 是 |
| `http2` | HTTP/2 支持 | ✅ 是 |
| `http3` | HTTP/3 支持 | ❌ 否 |
| `sync` | 同步接口支持 | ❌ 否 |
| `tokio_base` | tokio 运行时 | ❌ 否 |
| `ylong_base` | ylong_runtime 运行时 | ✅ 是 |
| `__tls` | TLS 支持（内部标记） | ✅ 是 |
| `__c_openssl` | OpenSSL 集成（内部标记） | ✅ 是 |

**内部依赖**：
| 依赖名称 | 依赖类型 | GN target |
|---------|----------|------------|
| `../ylong_http:ylong_http` | static_library | ylong_http |

**外部依赖**：
| 依赖名称 | 类型 | 库类型 |
|---------|------|-------|
| `openssl:libssl_shared` | shared_library | SSL 库 |
| `openssl:libcrypto_shared` | shared_library | 加密库 |
| `rust_libc:lib` | shared_library | Rust libc 绑定 |
| `ylong_runtime:ylong_runtime` | rust_library | 异步运行时 |

#### 3.2 rust_ylong_http_client_test_ut 单元测试

**Target 名称**: `rust_ylong_http_client_test_ut`

**GN 类型**: `ohos_rust_unittest`

**完整配置**（ylong_http_client/BUILD.gn:44-67）:
```gn
ohos_rust_unittest("rust_ylong_http_client_test_ut") {
  module_out_path = "ylong_http/ylong_http"

  rustflags = [
    "--cfg=feature=\"async\"",
    "--cfg=feature=\"http1_1\"",
    "--cfg=feature=\"http2\"",
    "--cfg=feature=\"c_openssl_3_0\"",
    "--cfg=feature=\"__tls\"",
    "--cfg=feature=\"__c_openssl\"",
    "--cfg=feature=\"ylong_base\""
  ]

  sources = [ "src/lib.rs" ]
  deps = [ "../ylong_http:ylong_http" ]
  external_deps = [
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
    "rust_libc:lib",
    "ylong_runtime:ylong_runtime"
  ]
}
```

| 配置项 | 值 | 说明 |
|---------|------|------|
| `module_out_path` | `ylong_http/ylong_http` | 测试输出路径 |

#### 3.3 unittest 分组

**Target 名称**: `unittest`

**GN 类型**: `group`

**完整配置**（ylong_http_client/BUILD.gn:69-78）:
```gn
group("unittest") {
  testonly = true
  deps = [
    ":rust_ylong_http_client_test_ut",
    "../ylong_http:rust_ylong_http_test_ut"
  ]
}
```

| 配置项 | 值 | 说明 |
|---------|------|------|
| `testonly` | true | 标记为测试目标 |
| `condition` | `!use_clang_coverage` | 排除覆盖率测试场景 |

### 4. 依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│                   构建依赖关系                      │
├─────────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────────────────────────────────────┐         │
│  │       unittest (group)               │         │
│  │                                      │         │
│  │  ┌──────────────────┐ ┌──────────────────┐  │
│  │  │ rust_ylong_    │ │ rust_ylong_   │  │
│  │  │ http_client_   │ │ http_test_ut  │  │
│  │  │ test_ut        │ │                │  │
│  │  └───────┬───────┘ └────────────────┘  │
│  │          │                             │         │
│  │          ▼                             │         │
│  │  ┌─────────────────────────────┐         │  │
│  │  │ ylong_http_client_   │         │  │
│  │  │     inner           │         │  │
│  │  └────────┬─────────────────┘         │  │
│  │           │                        │         │
│  │     ┌─────▼─────┐              │         │
│  │     │ openssl   │              │         │
│  │     │ rust_libc  │              │         │
│  │     └────┬──────┘              │         │
│  │           │                       │         │
│  │     ┌─────▼─────┐              │         │
│  │     │ ylong_http   │              │         │
│  │     └──────────────┘              │         │
│  │           │                       │         │
│  │     ┌─────▼─────┐              │         │
│  │     │ ylong_runtime│              │         │
│  │     └──────────────┘              │         │
│  └─────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────┘
```

**依赖说明**：
- `rust_ylong_http_client_test_ut` 依赖 `rust_ylong_http_test_ut`
- `rust_ylong_http_client_test_ut` 依赖 `ylong_http`
- `rust_ylong_http_client_test_ut` 依赖 `openssl`, `rust_libc`, `ylong_runtime`
- `ylong_http_client_inner` 依赖 `ylong_http`

**证据**: 各 target 的 `deps` 和 `external_deps` 配置

### 5. Feature Flags 详解

#### 5.1 ylong_http Features

**来源**: `ylong_http/Cargo.toml:18-28`

| Feature | GN 配置 | Cargo 配置 | 依赖 features | 说明 |
|---------|----------|-------------|------|
| `http1_1` | `"http1_1"` | `http1_1 = []` | HTTP/1.1 协议 |
| `http2` | `"http2"` | `http2 = []` | HTTP/2 协议 |
| `http3` | 未配置 | `http3 = []` | HTTP/3 协议（未启用） |
| `huffman` | `"huffman"` | `huffman = []` | Huffman 编码（用于 HTTP/2 HPACK 和 HTTP/3 QPACK） |
| `tokio_base` | 未配置 | `tokio = { version = "1.20.1", optional = true }` | tokio 异步运行时 |
| `ylong_base` | `"ylong_base"` | `ylong_runtime = { git = "...", optional = true }` | ylong_runtime 异步运行时 |

**说明**：
- `ylong_http` 默认使用 `ylong_base` 运行时
- 可通过 `tokio_base` feature 切换到 `tokio`
- `http3` 代码已实现，但 GN 配置中未启用

#### 5.2 ylong_http_client Features

**来源**: `ylong_http_client/Cargo.toml:11-26`

| Feature | GN 配置 | Cargo 配置 | 依赖 features | 说明 |
|---------|----------|-------------|------|
| `async` | `"async"` | `async = []` | 异步客户端接口 |
| `sync` | 未配置 | `sync = []` | 同步客户端接口 |
| `http1_1` | `"http1_1"` | `http1_1 = ["ylong_http/http1_1"]` | HTTP/1.1 支持 |
| `http2` | `"http2"` | `http2 = ["ylong_http/http2", "ylong_http/huffman"]` | HTTP/2 支持 |
| `http3` | 未配置 | `http3 = ["ylong_http/http3", "quiche", "ylong_http/huffman"]` | HTTP/3 支持（未启用） |
| `tokio_base` | 未配置 | `tokio = { version = "1.20.1", ... }` | tokio 异步运行时 |
| `ylong_base` | `"ylong_base"` | `ylong_base = ["ylong_runtime", "ylong_http/ylong_base"]` | ylong_runtime 异步运行时 |
| `tls_default` | 未配置 | `tls_default = ["__tls"]` | 默认 TLS 配置 |
| `c_openssl_3_0` | `"c_openssl_3_0"` | `c_openssl_3_0 = ["__tls", "libc"]` | OpenSSL 3.0 TLS |
| `__tls` | `"__tls"` | 内部 TLS 标记 |
| `__c_openssl` | `"__c_openssl"` | 内部 OpenSSL 标记 |

**说明**：
- `ylong_http_client` 默认使用 `async` + `ylong_base` 运行时
- 可通过 feature flags 灵活配置各种组合

---

## 相关跳转

- **[编译产物](07_Build_Artifacts.md)** - 编译产物和安装位置
- **[目录结构与模块职责](02_Directory_Structure.md)** - 代码组织

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
