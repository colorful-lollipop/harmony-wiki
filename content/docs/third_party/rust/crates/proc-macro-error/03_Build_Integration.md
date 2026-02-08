# OH 构建适配

## 3.1 构建系统概述

### 3.1.1 OpenHarmony Rust 构建架构

OpenHarmony 使用 **GN (Generate Ninja)** 作为构建系统，并通过 `ohos_cargo_crate` 模板集成 Rust crate。

```
OH 构建流程
    ↓
ohos_crate_crate 模板
    ↓
调用 cargo build --release
    ↓
生成 .rlib 或 .so 文件
    ↓
集成到最终系统
```

### 3.1.2 proc-macro-error 的构建配置

该库包含**两个**需要构建的 crate：

| crate | BUILD.gn 路径 | 类型 | 输出 |
|-------|--------------|------|------|
| 主库 | `proc-macro-error/BUILD.gn` | rlib | `libproc_macro_error.rlib` |
| 属性宏 | `proc-macro-error-attr/BUILD.gn` | proc-macro | `libproc_macro_error_attr.so` |

## 3.2 主 crate 构建配置

### 3.2.1 BUILD.gn 文件

**文件路径**: `third_party/rust/crates/proc-macro-error/BUILD.gn`

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
  crate_name = "proc_macro_error"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2018"
  cargo_pkg_version = "1.0.4"
  cargo_pkg_authors = "CreepySkeleton <creepy-skeleton@yandex.ru>"
  cargo_pkg_name = "proc-macro-error"
  cargo_pkg_description = "Almost drop-in replacement to panics in proc-macros"
  deps = [
    "//third_party/rust/crates/proc-macro-error/proc-macro-error-attr:lib(${host_toolchain})",
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
    "//third_party/rust/crates/syn:lib",
  ]
  features = [
    "syn",
    "syn-error",
  ]
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  build_deps = [ "//third_party/rust/crates/version_check:lib" ]
  module_output_extension = ".rlib"
  part_name = "rust_proc_macro_error"
  subsystem_name = "thirdparty"
}
```

### 3.2.2 配置项详解

#### 基础配置

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `crate_name` | `proc_macro_error` | Rust crate 内部名称 |
| `crate_type` | `rlib` | 编译为静态库 |
| `crate_root` | `src/lib.rs` | 库入口文件 |
| `edition` | `2018` | Rust Edition 版本 |
| `cargo_pkg_version` | `1.0.4` | 与上游版本同步 |

#### 依赖配置

| 依赖 | OH 路径 | 用途 |
|-----|--------|------|
| proc-macro-error-attr | `//third_party/rust/crates/proc-macro-error/proc-macro-error-attr:lib` | `#[proc_macro_error]` 属性宏 |
| proc-macro2 | `//third_party/rust/crates/proc-macro2:lib` | TokenStream 抽象 |
| quote | `//third_party/rust/crates/quote:lib` | 代码生成 |
| syn | `//third_party/rust/crates/syn:lib` | Rust 代码解析 |

#### 特性配置

```gn
features = [
  "syn",      # 启用 syn 依赖
  "syn-error", # 启用 syn 错误处理功能
]
```

**说明**：这两个特性是关联的，`syn-error` 依赖于 `syn`。

#### 构建配置

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `build_root` | `build.rs` | Cargo 构建脚本 |
| `build_sources` | `["build.rs"]` | 构建脚本源文件 |
| `build_deps` | `version_check` | 构建时依赖，用于版本检测 |

#### OH 组件配置

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `part_name` | `rust_proc_macro_error` | 组件名称 |
| `subsystem_name` | `thirdparty` | 子系统名称 |
| `module_output_extension` | `.rlib` | 输出文件扩展名 |

## 3.3 属性宏 crate 构建配置

### 3.3.1 BUILD.gn 文件

**文件路径**: `third_party/rust/crates/proc-macro-error/proc-macro-error-attr/BUILD.gn`

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
  crate_name = "proc_macro_error_attr"
  crate_type = "proc-macro"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2018"
  cargo_pkg_version = "1.0.4"
  cargo_pkg_authors = "CreepySkeleton <creepy-skeleton@yandex.ru>"
  cargo_pkg_name = "proc-macro-error-attr"
  cargo_pkg_description = "Attribute macro for proc-macro-error crate"
  deps = [
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
  ]
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  build_deps = [ "//third_party/rust/crates/version_check:lib" }
}
```

### 3.3.2 配置差异对比

| 配置项 | 主 crate | 属性 crate | 说明 |
|-------|---------|-----------|------|
| `crate_name` | `proc_macro_error` | `proc_macro_error_attr` | 不同 crate 名称 |
| `crate_type` | `rlib` | `proc-macro` | 不同类型 |
| `crate_root` | `src/lib.rs` | `src/lib.rs` | 相同 |
| `deps` | 包含 syn | 不包含 syn | 功能差异 |

**关键差异**：

```gn
# 主 crate: 需要 syn 依赖
deps = [
  "//third_party/rust/crates/proc-macro-error/proc-macro-error-attr:lib",
  "//third_party/rust/crates/proc-macro2:lib",
  "//third_party/rust/crates/quote:lib",
  "//third_party/rust/crates/syn:lib",  # ← 主库需要
]

# 属性 crate: 不需要 syn
deps = [
  "//third_party/rust/crates/proc-macro2:lib",
  "//third_party/rust/crates/quote:lib",
]
```

## 3.4 构建脚本分析

### 3.4.1 主 crate build.rs

**文件路径**: `proc-macro-error/build.rs`

```rust
fn main() {
    // 检查 Rust 版本是否支持特性标志
    if !version_check::is_feature_flaggable().unwrap_or(false) {
        println!("cargo:rustc-cfg=use_fallback");
    }

    // 检查是否需要跳过 UI 测试
    if version_check::is_max_version("1.38.0").unwrap_or(false)
        || !version_check::Channel::read().unwrap().is_stable()
    {
        println!("cargo:rustc-cfg=skip_ui_tests");
    }
}
```

**功能说明**：

| 检测项 | cfg 标志 | 作用 |
|-------|--------|------|
| 特性标志支持 | `use_fallback` | Rust 版本 < 1.32 时使用兼容实现 |
| 版本检测 | `skip_ui_tests` | Rust >= 1.38 或非稳定版时跳过 UI 测试 |

### 3.4.2 属性 crate build.rs

**文件路径**: `proc-macro-error-attr/build.rs`

```rust
fn main() {
    // 检测 Rust 1.36 版本的 unwind 行为变化
    if version_check::is_max_version("1.36.0").unwrap_or(false) {
        println!("cargo:rustc-cfg=always_assert_unwind");
    }
}
```

**功能说明**：

| 检测项 | cfg 标志 | 作用 |
|-------|--------|------|
| unwind 行为 | `always_assert_unwind` | Rust 1.36+ 版本的栈展开行为变化 |

## 3.5 Cargo.toml 配置

### 3.5.1 主 crate Cargo.toml

**文件路径**: `proc-macro-error/Cargo.toml`

```toml
[package]
name = "proc-macro-error"
version = "1.0.4"
authors = ["CreepySkeleton <creepy-skeleton@yandex.ru>"]
description = "Almost drop-in replacement to panics in proc-macros"

repository = "https://gitlab.com/CreepySkeleton/proc-macro-error"
readme = "README.md"
keywords = ["proc-macro", "error", "errors"]
categories = ["development-tools::procedural-macro-helpers"]
license = "MIT OR Apache-2.0"

edition = "2018"
build = "build.rs"

[badges]
maintenance = { status = "passively-maintained" }

[package.metadata.docs.rs]
targets = ["x86_64-unknown-linux-gnu"]

[dependencies]
quote = "1"
proc-macro2 = "1"
proc-macro-error-attr = { path = "./proc-macro-error-attr", version = "=1.0.4"}

[dependencies.syn]
version = "1"
optional = true
default-features = false

[dev-dependencies]
test-crate = { path = "./test-crate" }
proc-macro-hack-test = { path = "./test-crate/proc-macro-hack-test" }
trybuild = { version = "1.0.19", features = ["diff"] }
toml = "=0.5.2" # DO NOT BUMP
serde_derive = "=1.0.107" # DO NOT BUMP

[build-dependencies]
version_check = "0.9"

[features]
default = ["syn-error"]
syn-error = ["syn"]
```

### 3.5.2 与 BUILD.gn 的映射

| Cargo.toml 配置 | BUILD.gn 配置 | 说明 |
|----------------|--------------|------|
| `version = "1.0.4"` | `cargo_pkg_version = "1.0.4"` | 版本同步 |
| `edition = "2018"` | `edition = "2018"` | Edition 同步 |
| `syn = "1"` | `deps += ["//third_party/rust/crates/syn:lib"]` | 依赖映射 |
| `default = ["syn-error"]` | `features = ["syn", "syn-error"]` | 特性启用 |
| `path = "./proc-macro-error-attr"` | `deps += ["...proc-macro-error-attr:lib"]` | 子 crate 依赖 |

## 3.6 与上游构建的差异

### 构建差异对比

| 维度 | 上游 (Cargo) | OH (BUILD.gn) |
|-----|-------------|--------------|
| 构建工具 | cargo | gn + cargo |
| 依赖来源 | crates.io | OH third_party 目录 |
| 输出目录 | `target/` | `out/` |
| 配置方式 | Cargo.toml | BUILD.gn + cargo_pkg_* |
| 组件系统 | 无 | part/subsystem |

### OH 特有的适配

| 适配项 | 实现方式 |
|-------|---------|
| 依赖重定向 | BUILD.gn 中使用 OH third_party 路径 |
| 组件注册 | bundle.json 中声明 part |
| 输出路径 | 由 gn 控制输出到 out/ |
| 构建类型 | rlib/proc-macro 由 crate_type 指定 |

## 3.7 构建测试

### 验证构建

```bash
# 在 OH 构建环境中
hb build -p rust_proc_macro_error
```

### 构建产物

| 产物 | 路径 | 类型 |
|-----|------|------|
| 主库 | `out/.../third_party/rust/crates/proc-macro-error/libproc_macro_error.rlib` | rlib |
| 属性宏 | `out/.../third_party/rust/crates/proc-macro-error/libproc_macro_error_attr.so` | proc-macro |

## 3.8 常见问题

### Q1: 为什么 proc-macro crate 需要单独编译为 .so？

**答案**：过程宏在 rustc 编译过程中作为编译器插件加载。rustc 通过动态加载 `.so` (Unix) 或 `.dll` (Windows) 文件来执行过程宏代码。

### Q2: syn 依赖为什么不使用 default-features？

**答案**：上游使用 `default-features = false` 是为了避免编译整个 `syn` crate（移除 `full` 特性可减少编译时间约 30 秒）。OH BUILD.gn 启用 `syn` 特性是为了使用 `syn::Error` 类型。

### Q3: version_check 的作用是什么？

**答案**：该库支持 Rust 1.31+ 多个版本。`version_check` crate 用于在编译时检测当前 Rust 版本，根据版本差异启用不同的代码路径（如旧版本的 fallback 实现）。
