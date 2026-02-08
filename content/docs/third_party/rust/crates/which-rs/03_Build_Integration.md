# OH 构建适配

## 概述

which-rs 使用 OpenHarmony 的标准 Rust 构建模板 `ohos_cargo_crate` 进行构建。本章节详细解析 BUILD.gn 的配置及其与上游 Cargo.toml 的差异。

## BUILD.gn 完整配置

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
    crate_name = "which"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "4.4.0"
    cargo_pkg_authors = "Harry Fei <tiziyuanfang@gmail.com>"
    cargo_pkg_name = "which"
    cargo_pkg_description = "A Rust equivalent of Unix command \"which\". Locate installed executable in cross platforms."
    deps = [
        "//third_party/rust/crates/either:lib",
        "//third_party/rust/crates/libc:lib",
    ]
    module_output_extension = ".rlib"
    part_name = "rust_which_rs"
    subsystem_name = "thirdparty"
}
```

## 关键配置项解析

### 1. 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | "which" | Rust crate 名称，与 Cargo.toml 一致 |
| `crate_type` | "rlib" | 生成 Rust 静态库（.rlib） |
| `crate_root` | "src/lib.rs" | crate 根文件 |
| `edition` | "2018" | Rust 语言版本 |

### 2. 元数据配置

这些元数据直接来源于 Cargo.toml：

```toml
# Cargo.toml
[package]
name = "which"
version = "4.4.0"
edition = "2018"
authors = ["Harry Fei <tiziyuanfang@gmail.com>"]
description = "A Rust equivalent of Unix command \"which\"..."
```

对应的 BUILD.gn：
- `cargo_pkg_version = "4.4.0"`
- `cargo_pkg_authors = "Harry Fei <tiziyuanfang@gmail.com>"`
- `cargo_pkg_name = "which"`
- `cargo_pkg_description = "..."`

### 3. 依赖配置

```gn
deps = [
    "//third_party/rust/crates/either:lib",
    "//third_party/rust/crates/libc:lib",
]
```

对应 Cargo.toml：

```toml
[dependencies]
either = "1.6.1"
libc = "0.2.121"
```

### 4. OH 组件标识

```gn
part_name = "rust_which_rs"
subsystem_name = "thirdparty"
```

与 bundle.json 对应：

```json
{
  "name": "@ohos/rust_which_rs",
  "component": {
    "name": "rust_which_rs",
    "subsystem": "thirdparty"
  }
}
```

## 与上游 Cargo.toml 的差异

### 完整差异对比表

| 配置项 | Cargo.toml | BUILD.gn | 差异分析 |
|--------|------------|----------|----------|
| **edition** | 2018 | 2018 | ✅ 一致 |
| **核心依赖** | either, libc | either, libc | ✅ 一致 |
| **regex feature** | 可选 | ❌ 未启用 | OH 未启用正则功能 |
| **Windows 依赖** | once_cell | ❌ 未包含 | OH 不支持 Windows |
| **开发依赖** | tempfile | ❌ 未包含 | 仅构建库，不运行测试 |
| **测试文件** | tests/ | ❌ 未包含 | 仅构建库 |

### 差异 1: regex feature 未启用

**上游配置**:
```toml
[dependencies]
regex = { version = "1.5.5", optional = true }

[features]
default = []
regex = ["dep:regex"]
```

**OH 配置**:
- 未在 deps 中包含 regex
- 未启用 regex feature

**影响**:
- OH 中无法使用 `which_re()` 和 `which_re_in()` 函数
- 这些函数需要启用 `regex` feature 才能编译

**代码体现**:
```rust
// lib.rs 中的条件编译
#[cfg(feature = "regex")]
pub fn which_re(regex: impl Borrow<Regex>) -> Result<impl Iterator<Item = path::PathBuf>> {
    // ...
}
```

### 差异 2: Windows 支持未纳入

**上游配置**:
```toml
[target.'cfg(windows)'.dependencies]
once_cell = "1"
```

**OH 配置**:
- 未包含 `helper.rs` 中的 Windows 特有代码
- 未包含 `once_cell` 依赖

**代码体现**:
```rust
// helper.rs 仅在 Windows 编译
#[cfg(windows)]
mod helper;

// finder.rs 中的 Windows 扩展名处理
#[cfg(windows)]
fn append_extension<P>(paths: P) -> impl IntoIterator<Item = PathBuf> { ... }

#[cfg(unix)]
fn append_extension<P>(paths: P) -> impl IntoIterator<Item = PathBuf> {
    paths  // Unix 直接返回，不做处理
}
```

这是合理的，因为 OpenHarmony 标准系统基于 Linux。

### 差异 3: 源文件精简

**完整源码文件**:
```
src/
├── lib.rs
├── finder.rs
├── checker.rs
├── error.rs
└── helper.rs  # Windows only
```

**OH BUILD.gn 中**:
```gn
sources = ["src/lib.rs"]
```

这里只显式声明了 `lib.rs`，但实际上 `ohos_cargo_crate` 模板会：
1. 从 `crate_root` 开始分析模块依赖
2. 自动包含所有被引用的模块文件

因此 `finder.rs`, `checker.rs`, `error.rs` 仍会被编译，只是无需在 `sources` 中显式列出。

## 构建流程

### GN 构建过程

```
┌─────────────┐
│  BUILD.gn   │
│ 配置定义    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ ohos_cargo_ │
│ _crate      │
│ 模板处理    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ rustc       │
│ 编译        │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ libwhich.rlib│
│ 输出        │
└─────────────┘
```

### 依赖传递

```
which-rs (libwhich.rlib)
    ├── either (libeither.rlib)
    └── libc (liblibc.rlib)
```

## 使用方式

### 在 Rust 项目中引用

**Cargo.toml**:
```toml
[dependencies]
which = { path = "../../third_party/rust/crates/which-rs" }
```

**GN 依赖**:
```gn
deps += [ "//third_party/rust/crates/which-rs:lib" ]
```

**代码使用**:
```rust
use which::which;

fn main() {
    let path = which("clang").expect("clang not found");
    println!("Found clang at: {:?}", path);
}
```

## 构建配置建议

### 如需启用 regex 功能

如需在 OH 中使用正则匹配功能，需要修改 BUILD.gn：

```gn
ohos_cargo_crate("lib") {
    # ... 现有配置 ...
    
    # 添加 regex 依赖
    deps += [ "//third_party/rust/crates/regex:lib" ]
    
    # 启用 feature（如模板支持）
    # 注意：ohos_cargo_crate 模板可能需要定制以支持 features
}
```

**注意**: 这需要确保 regex crate 及其依赖（如 regex-syntax、aho-corasick 等）也在 OH 中可用。

### 升级上游版本

升级步骤：

1. 替换源码文件
2. 更新 BUILD.gn 中的版本号：
   ```gn
   cargo_pkg_version = "x.y.z"
   ```
3. 检查依赖变更：
   - 对比新旧 Cargo.toml 的 dependencies
   - 在 BUILD.gn 中同步更新 deps
4. 验证编译：
   ```bash
   gn gen out && ninja -C out which-rs:lib
   ```

## 总结

which-rs 的 OH 构建配置遵循标准模板，主要特点：

1. **配置简洁**: 使用 `ohos_cargo_crate` 模板，无需复杂配置
2. **功能裁剪**: 未启用 regex feature，减少依赖
3. **平台聚焦**: 仅保留 Linux 支持代码
4. **易于维护**: 无 Patch，升级简单

这种配置方式适用于大多数标准 Rust crate 的 OH 集成。
