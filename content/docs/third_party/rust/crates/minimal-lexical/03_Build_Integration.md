# 03 - OpenHarmony 构建系统集成

## 3.1 BUILD.gn 完整配置

### 文件位置

```
//third_party/rust/crates/minimal-lexical/BUILD.gn
```

### 完整内容

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
    crate_name = "minimal_lexical"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.2.1"
    cargo_pkg_authors = "Alex Huszagh <ahuszagh@gmail.com>"
    cargo_pkg_name = "minimal-lexical"
    cargo_pkg_description = "Fast float parsing conversion routines."
    features = ["std"]
    module_output_extension = ".rlib"
    part_name = "rust_minimal_lexical"
    subsystem_name = "thirdparty"
}
```

## 3.2 配置项详解

### 3.2.1 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `minimal_lexical` | Rust crate 名称（用于导入） |
| `crate_type` | `rlib` | 输出类型：Rust 静态库 |
| `crate_root` | `src/lib.rs` | crate 入口文件 |
| `edition` | `"2018"` | Rust Edition 2018 |

### 3.2.2 源码配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | `["src/lib.rs"]` | 主源文件（Rust 的 lib.rs 会导入其他模块） |

**注意**：虽然只列出了 `src/lib.rs`，但实际上 Rust 的模块系统会通过 `mod` 声明自动包含其他文件。完整源文件包括：

```
src/
├── lib.rs              # 入口（在 sources 中列出）
├── bellerophon.rs      # 通过 mod bellerophon 包含
├── bigint.rs           # 通过 mod bigint 包含
├── extended_float.rs   # 通过 mod extended_float 包含
├── fpu.rs              # 通过 mod fpu 包含
├── heapvec.rs          # 通过 mod heapvec 包含
├── lemire.rs           # 通过 mod lemire 包含
├── libm.rs             # 通过 mod libm 包含
├── mask.rs             # 通过 mod mask 包含
├── num.rs              # 通过 mod num 包含
├── number.rs           # 通过 mod number 包含
├── parse.rs            # 通过 mod parse 包含
├── rounding.rs         # 通过 mod rounding 包含
├── slow.rs             # 通过 mod slow 包含
├── stackvec.rs         # 通过 mod stackvec 包含
├── table.rs            # 通过 mod table 包含
├── table_bellerophon.rs
├── table_lemire.rs
└── table_small.rs
```

### 3.2.3 元数据配置

| 配置项 | 值 | 来源 |
|--------|-----|------|
| `cargo_pkg_version` | `"0.2.1"` | Cargo.toml: `version` |
| `cargo_pkg_authors` | `"Alex Huszagh <ahuszagh@gmail.com>"` | Cargo.toml: `authors` |
| `cargo_pkg_name` | `"minimal-lexical"` | Cargo.toml: `name` |
| `cargo_pkg_description` | `"Fast float parsing conversion routines."` | Cargo.toml: `description` |

### 3.2.4 Feature 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `features` | `["std"]` | 启用的 Cargo features |

**与上游 Cargo.toml 对比**：

```toml
# 上游 Cargo.toml features
[features]
default = ["std"]
std = []              # ✅ OH 启用
compact = []          # ❌ OH 未启用
alloc = []            # ❌ OH 未启用
nightly = []          # ❌ OH 未启用
lint = []             # ❌ OH 未启用（内部使用）
```

**OH 仅启用 `std` feature 的原因**：
- `std` 是上游默认 feature，提供完整功能
- 不需要 `compact`（体积优化）带来的体积缩减
- 不需要 `alloc`（无标准库但有分配器）模式
- 不需要 `nightly`（需要 nightly Rust 编译器）

### 3.2.5 OH 系统配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `module_output_extension` | `.rlib` | 输出文件扩展名 |
| `part_name` | `rust_minimal_lexical` | OH 组件名 |
| `subsystem_name` | `thirdparty` | 所属子系统 |

## 3.3 与上游 Cargo.toml 的映射关系

### 3.3.1 完整映射表

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `[package].name` | `cargo_pkg_name` | 包名 |
| `[package].version` | `cargo_pkg_version` | 版本 |
| `[package].authors` | `cargo_pkg_authors` | 作者 |
| `[package].description` | `cargo_pkg_description` | 描述 |
| `[package].edition` | `edition` | Rust Edition |
| `[features].default` | `features` | 默认启用的 features |
| `[lib].name` | `crate_name` | crate 名称 |
| `[lib].crate-type` | `crate_type` | 输出类型 |
| `[lib].path` | `crate_root` | 入口文件 |

### 3.3.2 Cargo.toml 完整内容

```toml
[package]
authors = ["Alex Huszagh <ahuszagh@gmail.com>"]
autoexamples = false
categories = ["parsing", "no-std"]
description = "Fast float parsing conversion routines."
documentation = "https://docs.rs/minimal-lexical"
edition = "2018"
keywords = ["parsing", "no_std"]
license = "MIT/Apache-2.0"
name = "minimal-lexical"
readme = "README.md"
repository = "https://github.com/Alexhuszagh/minimal-lexical"
version = "0.2.1"
exclude = [
    "assets/*",
    "ci/*",
    "docs/*",
    "etc/*",
    "fuzz/*",
    "examples/*",
    "scripts/*"
]

[features]
default = ["std"]
std = []
compact = []
alloc = []
nightly = []
lint = []
```

## 3.4 构建输出

### 3.4.1 输出文件

构建完成后，输出文件位于：

```
out/{device-type}/third_party/rust/crates/minimal-lexical/
├── libminimal_lexical.rlib    # Rust 静态库
└── ...
```

### 3.4.2 库类型说明

| 扩展名 | 类型 | 说明 |
|--------|------|------|
| `.rlib` | Rust Library | Rust 静态库，供其他 Rust 代码链接 |

**注意**：minimal-lexical 只输出 `.rlib` 格式，不输出：
- `.so`（动态库）
- `.a`（C 静态库）
- `.dylib`（macOS 动态库）

这是因为 minimal-lexical 是 Rust 内部使用的库，不提供 C ABI。

## 3.5 在其他模块中使用

### 3.5.1 添加依赖

在其他模块的 `BUILD.gn` 中添加依赖：

```gn
ohos_cargo_crate("your_crate") {
    # ... 其他配置 ...
    
    deps = [
        "//third_party/rust/crates/minimal-lexical:lib",
        # 其他依赖...
    ]
}
```

### 3.5.2 在 Rust 代码中使用

```rust
// 导入库
extern crate minimal_lexical;

// 使用 parse_float 函数
use minimal_lexical::parse_float;

fn main() {
    let integer = b"3";
    let fraction = b"14159";
    let pi: f64 = parse_float(integer.iter(), fraction.iter(), 0);
    println!("π ≈ {}", pi);  // π ≈ 3.14159
}
```

### 3.5.3 实际使用示例（nom 库）

查看 nom 如何使用 minimal-lexical：

```gn
# //third_party/rust/crates/nom/BUILD.gn
ohos_cargo_crate("lib") {
    crate_name = "nom"
    # ...
    deps = [
        "//third_party/rust/crates/memchr:lib",
        "//third_party/rust/crates/minimal-lexical:lib",  # <-- 依赖
    ]
    features = [
        "alloc",
        "std",
    ]
    # ...
}
```

## 3.6 特殊配置说明

### 3.6.1 无特殊配置

minimal-lexical 的 BUILD.gn **没有以下配置**：

- ❌ `defines` - 无自定义宏定义
- ❌ `configs` - 无特殊编译配置
- ❌ `cflags` / `ldflags` - 无自定义编译/链接选项
- ❌ `include_dirs` - 无额外头文件路径
- ❌ `deps` - 无依赖其他库（零依赖）

### 3.6.2 为什么如此简单？

minimal-lexical 的 BUILD.gn 配置非常简单，原因如下：

1. **零依赖**：不依赖其他 Rust crate 或 C 库
2. **纯 Rust**：无需链接系统库
3. **标准模板**：使用 `ohos_cargo_crate` 标准模板即可
4. **无平台代码**：无需条件编译或平台特定配置

### 3.6.3 与其他 Rust crate 的对比

| 库 | BUILD.gn 行数 | 复杂度 | 原因 |
|------|--------------|--------|------|
| minimal-lexical | ~32 | 低 | 零依赖，纯 Rust |
| nom | ~39 | 中 | 有 2 个依赖 |
| openssl | 100+ | 高 | 依赖系统 OpenSSL |
| libc | 50+ | 中 | 需要条件编译 |

## 3.7 构建系统差异

### 3.7.1 上游构建 vs OH 构建

| 方面 | 上游（Cargo） | OH（GN + Cargo） |
|------|--------------|-----------------|
| 构建工具 | `cargo build` | `gn gen` + `ninja` |
| 配置文件 | `Cargo.toml` | `BUILD.gn` + `Cargo.toml` |
| 依赖解析 | Cargo 自动处理 | GN 处理 + Cargo 处理 |
| 输出目录 | `target/` | `out/{device-type}/` |
| 交叉编译 | 需配置 target | GN 自动处理 |

### 3.7.2 构建命令示例

**上游构建**：
```bash
cd minimal-lexical
cargo build --release
```

**OH 构建**：
```bash
cd oh/# 完整 OH 构建
./build.sh --product {product}

# 或仅构建此库
ninja -C out/{device-type} third_party/rust/crates/minimal-lexical:lib
```

## 3.8 调试与诊断

### 3.8.1 查看构建输出

```bash
# 查看详细的构建日志
ninja -C out/{device-type} third_party/rust/crates/minimal-lexical:lib -v
```

### 3.8.2 检查生成的 rlib

```bash
# 查看 rlib 内容
llvm-ar tv out/{device-type}/third_party/rust/crates/minimal-lexical/libminimal_lexical.rlib

# 查看符号
llvm-nm out/{device-type}/third_party/rust/crates/minimal-lexical/libminimal_lexical.rlib
```

### 3.8.3 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| 编译错误 | Rust 版本不匹配 | 检查 MSRV (1.36+) |
| feature 错误 | feature 未启用 | 检查 BUILD.gn 的 `features` |
| 找不到 crate | 依赖未添加 | 在 deps 中添加 `//third_party/rust/crates/minimal-lexical:lib` |

---

*本文档最后更新：2026-02-07*
