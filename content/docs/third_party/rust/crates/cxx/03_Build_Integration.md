# OH 构建适配

## 构建架构概述

cxx 库在 OpenHarmony 中的构建适配通过三个 `BUILD.gn` 文件完成，分别对应不同的构建目标：

```
third_party/rust/crates/cxx/
├── BUILD.gn              # 主库 + C++ 运行时
├── macro/BUILD.gn        # 过程宏
└── gen/cmd/BUILD.gn      # 命令行工具
```

## 构建目标清单

| 构建目标 | 类型 | 组件名 | 用途 |
|---------|------|--------|------|
| `lib` | ohos_cargo_crate | cxx | Rust 核心库 |
| `cxx_cppdeps` | ohos_static_library | cxx_cppdeps | C++ 运行时支持 |
| `macro_lib` | ohos_cargo_crate | cxxbridge-macro | 代码生成过程宏 |
| `cxxbridge` | ohos_cargo_crate | cxxbridge-cmd | 命令行代码生成器 |

## 主 BUILD.gn 详解

### 文件路径

`//third_party/rust/crates/cxx/BUILD.gn`

### Rust 核心库（lib）

```gn
ohos_cargo_crate("lib") {
  crate_name = "cxx"
  crate_type = "rlib"        # Rust 静态库格式
  crate_root = "src/lib.rs"
  edition = "2018"
  cargo_pkg_version = "1.0.130"
  cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
  cargo_pkg_name = "cxx"
  cargo_pkg_description = "Safe interop between Rust and C++"

  deps = [ "//third_party/rust/crates/cxx/macro:macro_lib(${host_toolchain})" ]

  features = [
    "alloc",   # 启用堆分配支持
    "std",     # 启用标准库支持
  ]

  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  part_name = "rust_cxx"
  subsystem_name = "thirdparty"
}
```

**配置说明**：
- `crate_type = "rlib"`：生成静态库，供其他 Rust crate 链接
- `features`：启用 `alloc` 和 `std`，完整支持标准库类型
- `deps`：依赖过程宏 crate，用于代码生成

### C++ 运行时库（cxx_cppdeps）

```gn
config("cxx_cppdeps_header_config") {
  include_dirs = [ "include" ]
}

ohos_static_library("cxx_cppdeps") {
  part_name = "build_framework"
  subsystem_name = "build"
  defines = [ "RUST_CXX_NO_EXCEPTIONS" ]  # 禁用 C++ 异常
  public_configs = [ ":cxx_cppdeps_header_config" ]
  sources = [
    "//third_party/rust/crates/cxx/include/cxx.h",
    "//third_party/rust/crates/cxx/src/cxx.cc",
  ]
  deps = [ "//third_party/rust/crates/cxx:lib" ]

  if (is_win) {
    defines += [ "CXX_RS_EXPORT=__declspec(dllexport)" ]
  } else {
    defines += [ "CXX_RS_EXPORT=__attribute__((visibility(\"default\")))" ]
  }

  part_name = "rust_cxx"
  subsystem_name = "thirdparty"
}
```

**关键配置解析**：

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `defines` | `RUST_CXX_NO_EXCEPTIONS` | 禁用 C++ 异常处理，符合 OH 安全策略 |
| `sources` | `cxx.h`, `cxx.cc` | C++ 运行时实现 |
| `public_configs` | 头文件路径 | 导出头文件搜索路径 |
| `CXX_RS_EXPORT` | 平台相关 | 控制符号可见性 |

**平台差异处理**：
- **Windows**：使用 `__declspec(dllexport)` 导出符号
- **其他平台**：使用 `__attribute__((visibility("default")))` 导出符号

### 头文件配置

```gn
config("cxx_cppdeps_header_config") {
  include_dirs = [ "include" ]
}
```

此配置被 `cxx_cppdeps` 公开使用，确保依赖模块能够找到 cxx 的 C++ 头文件。

## macro/BUILD.gn 详解

### 文件路径

`//third_party/rust/crates/cxx/macro/BUILD.gn`

### 过程宏构建

```gn
ohos_cargo_crate("macro_lib") {
  crate_name = "cxxbridge_macro"
  crate_type = "proc-macro"         # 声明为过程宏类型
  visibility = [ "//third_party/rust/crates/*" ]  # 限制可见性

  crate_root = "src/lib.rs"
  edition = "2021"
  cargo_pkg_version = "1.0.130"
  cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
  cargo_pkg_name = "cxxbridge-macro"
  cargo_pkg_description = "Implementation detail of the `cxx` crate."

  deps = [
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
    "//third_party/rust/crates/syn:lib",
  ]

  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  part_name = "rust_cxx"
  subsystem_name = "thirdparty"
}
```

**关键配置**：

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `crate_type` | `proc-macro` | Rust 过程宏 crate |
| `visibility` | 仅 Rust crates | 限制仅 Rust 构建使用 |
| `deps` | OH 适配版本 | 使用 OH 生态中的依赖 |

## gen/cmd/BUILD.gn 详解

### 文件路径

`//third_party/rust/crates/cxx/gen/cmd/BUILD.gn`

### 命令行工具构建

```gn
if (host_os != "linux" || host_cpu != "arm64") {
  ohos_cargo_crate("cxxbridge") {
    crate_type = "bin"
    crate_root = "src/main.rs"

    sources = [ "src/main.rs" ]
    edition = "2021"
    cargo_pkg_version = "1.0.130"
    cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
    cargo_pkg_name = "cxxbridge-cmd"
    cargo_pkg_description = "C++ code generator for integrating `cxx` crate into a non-Cargo build."

    deps = [
      "//third_party/rust/crates/clap:lib",
      "//third_party/rust/crates/codespan/codespan-reporting:lib",
      "//third_party/rust/crates/proc-macro2:lib",
      "//third_party/rust/crates/quote:lib",
      "//third_party/rust/crates/syn:lib",
    ]

    part_name = "rust_cxx"
    subsystem_name = "thirdparty"
  }
}
```

**条件编译说明**：

```gn
if (host_os != "linux" || host_cpu != "arm64")
```

此条件排除了 **Linux ARM64** 平台。可能原因：
- 该平台使用预编译的 cxxbridge 工具
- 工具链已内置相关功能
- 构建资源有限，避免重复构建

## 功能特性配置

### Cargo Features

在 `Cargo.toml` 中定义的功能：

```toml
[features]
default = ["std", "cxxbridge-flags/default"]  # C++11
"c++14" = ["cxxbridge-flags/c++14"]
"c++17" = ["cxxbridge-flags/c++17"]
"c++20" = ["cxxbridge-flags/c++20"]
alloc = []
std = ["alloc"]
```

OH 构建中使用 `features = ["alloc", "std"]`，启用完整标准库支持。

### 编译标志

通过 `cxxbridge-flags` 控制 C++ 标准版本：

```toml
cxxbridge-flags = { version = "=1.0.130", path = "flags", default-features = false }
```

## 与上游构建差异

| 方面 | 上游（Cargo） | OH（GN） |
|-----|--------------|----------|
| 构建系统 | Cargo | GN |
| Rust 库 | cargo build | ohos_cargo_crate |
| C++ 库 | cc crate | ohos_static_library |
| 过程宏 | proc-macro crate | ohos_cargo_crate (proc-macro) |
| 工具 | cargo install | ohos_cargo_crate (bin) |
| C++ 标准 | 通过 cxxbridge-flags | 继承 OH 全局配置 |

## 常见构建问题

### 问题 1：符号未导出

**现象**：C++ 代码无法找到 cxx 符号

**解决方案**：检查 `CXX_RS_EXPORT` 配置是否正确设置

### 问题 2：ARM64 Linux 平台缺失 cxxbridge

**现象**：Linux ARM64 平台无法使用 cxxbridge 工具

**解决方案**：确认是否需要移除条件编译或提供预编译版本

### 问题 3：C++ 异常传播失败

**现象**：`Result<T, E>` 异常传播不工作

**解决方案**：检查 `RUST_CXX_NO_EXCEPTIONS` 是否意外启用

## 相关文档

- [02_Patches.md](./02_Patches.md) - Patch 分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - OH 使用方式
