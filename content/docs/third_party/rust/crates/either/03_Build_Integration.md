# either - OpenHarmony 构建适配

## 1. BUILD.gn 详解

### 1.1 完整配置

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
    crate_name = "either"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "1.8.1"
    cargo_pkg_authors = "bluss"
    cargo_pkg_name = "either"
    features = ["use_std"]
    module_output_extension = ".rlib"
    part_name = "rust_either"
    subsystem_name = "thirdparty"
}
```

### 1.2 配置项说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `"either"` | Rust crate 名称，用于编译单元命名 |
| `crate_type` | `"rlib"` | 输出类型：Rust 静态库 |
| `crate_root` | `"src/lib.rs"` | crate 入口文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表 (单文件库) |
| `edition` | `"2018"` | Rust 版本 Edition |
| `cargo_pkg_version` | `"1.8.1"` | Cargo.toml 中的版本 |
| `cargo_pkg_authors` | `"bluss"` | 作者信息 |
| `cargo_pkg_name` | `"either"` | Cargo package 名称 |
| `features` | `["use_std"]` | 启用的特性 |
| `module_output_extension` | `".rlib"` | 输出文件扩展名 |
| `part_name` | `"rust_either"` | OH 组件名 (与 bundle.json 对应) |
| `subsystem_name` | `"thirdparty"` | 所属子系统 |

---

## 2. 特性配置

### 2.1 启用的特性

```gn
features = ["use_std"]
```

**`use_std` 特性说明**:

| 状态 | 说明 |
|------|------|
| 默认 | 在上游 Cargo.toml 中默认启用 |
| OH 配置 | 显式启用，确保功能完整 |

**启用 `use_std` 后的额外功能**:

```rust
// src/lib.rs 中条件编译部分
#[cfg(any(test, feature = "use_std"))]
extern crate std;

#[cfg(any(test, feature = "use_std"))]
use std::io::{self, BufRead, Read, Seek, SeekFrom, Write};

#[cfg(any(test, feature = "use_std"))]
use std::error::Error;
```

- ✅ `Read` trait 实现
- ✅ `Write` trait 实现
- ✅ `Seek` trait 实现
- ✅ `BufRead` trait 实现
- ✅ `Error` trait 实现

### 2.2 未启用的特性

```toml
# 上游 Cargo.toml 中的可选特性
[features]
default = ["use_std"]
use_std = []

[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

| 特性 | 状态 | 原因 |
|------|------|------|
| `serde` | 未启用 | 无序列化需求，减少依赖 |

**未启用 serde 的影响**:

- ❌ `serde_untagged` 模块不可用
- ❌ `serde_untagged_optional` 模块不可用
- ❌ 无法对 `Either` 进行 serde 序列化/反序列化

---

## 3. 与上游构建系统对比

### 3.1 上游 (Cargo)

```toml
# Cargo.toml
[package]
name = "either"
version = "1.8.1"
edition = "2018"
rust-version = "1.36"

[features]
default = ["use_std"]
use_std = []

[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

### 3.2 OpenHarmony (GN)

```gn
ohos_cargo_crate("lib") {
    crate_name = "either"
    crate_type = "rlib"
    sources = ["src/lib.rs"]
    edition = "2018"
    features = ["use_std"]
    # ... 元数据字段
}
```

### 3.3 差异对比表

| 方面 | Cargo | GN (OH) | 说明 |
|------|-------|---------|------|
| 构建工具 | Cargo | GN + Ninja | OH 统一构建系统 |
| 依赖声明 | `[dependencies]` | `deps = []` | which-rs 通过 GN 声明依赖 |
| 特性配置 | `features = [...]` | `features = [...]` | 语法相似 |
| 编译选项 | `Cargo.toml` | BUILD.gn | 配置位置不同 |
| 输出格式 | 自动 | `.rlib` | 显式指定 |

---

## 4. 依赖关系

### 4.1 依赖者

```
either:lib
    ▲
    │
    │ deps = ["//third_party/rust/crates/either:lib"]
    │
which-rs:lib
```

**which-rs BUILD.gn 依赖声明**:

```gn
ohos_cargo_crate("lib") {
    crate_name = "which"
    # ...
    deps = [
        "//third_party/rust/crates/either:lib",
        "//third_party/rust/crates/libc:lib",
    ]
}
```

### 4.2 被依赖的 crate

| crate | 路径 | 说明 |
|-------|------|------|
| which-rs | `//third_party/rust/crates/which-rs:lib` | Unix which 命令实现 |

### 4.3 Rust 工具链内部依赖

Rust 编译器和 rust-analyzer 在 `Cargo.toml` 中声明依赖：

```toml
# third_party/rust/rust/compiler/rustc_*/Cargo.toml
either = "1"

# third_party/rust/rust/src/tools/rust-analyzer/crates/*/Cargo.toml
either = "1.7.0"
```

**注意**: 工具链使用自己的 Cargo 构建，不通过 GN。

---

## 5. 编译过程

### 5.1 编译命令

```bash
# GN 生成构建配置
gn gen out/default

# 编译 either 库
ninja -C out/default third_party/rust/crates/either:lib
```

### 5.2 输出文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `libeither.rlib` | `out/default/obj/third_party/rust/crates/either/libeither.rlib` | Rust 静态库 |

### 5.3 编译参数

```bash
# rustc 实际执行的命令示例
rustc \
    --edition 2018 \
    --crate-type rlib \
    --crate-name either \
    --cfg 'feature="use_std"' \
    src/lib.rs \
    -o libeither.rlib
```

---

## 6. 使用方式

### 6.1 添加依赖

在其他 GN 构建的 Rust 项目中使用：

```gn
ohos_cargo_crate("my_crate") {
    crate_name = "my_crate"
    crate_type = "rlib"
    # ...
    deps = [
        "//third_party/rust/crates/either:lib",
    ]
}
```

### 6.2 代码中使用

```rust
// 在依赖 either 的 crate 中使用
use either::{Either, Left, Right};

pub fn example() {
    let value: Either<i32, String> = Left(42);
    
    if let Some(n) = value.left() {
        println!("Left: {}", n);
    }
}
```

### 6.3 Cargo.toml 方式

如果是 Cargo 构建的 crate (如 Rust 工具链)：

```toml
[dependencies]
either = { path = "../../../third_party/rust/crates/either" }
```

---

## 7. 构建注意事项

### 7.1 无特殊配置

| 配置 | 状态 | 说明 |
|------|------|------|
| `defines` | 无 | 无需预定义宏 |
| `configs` | 无 | 无需特殊编译配置 |
| `cflags` | 无 | 无需特殊编译选项 |
| `ldflags` | 无 | 无需特殊链接选项 |

### 7.2 跨平台支持

- ✅ Linux (OH 标准系统)
- ✅ 理论上支持 Windows/macOS (OH 不针对)

### 7.3 编译警告

当前无编译警告，代码质量高。

---

## 8. 升级指南

### 8.1 升级步骤

1. **下载新版本**
   ```bash
   cd third_party/rust/crates/either
   # 替换为新版本源码
   ```

2. **更新 BUILD.gn**
   ```gn
   cargo_pkg_version = "新版本号"
   ```

3. **验证构建**
   ```bash
   ninja -C out/default third_party/rust/crates/either:lib
   ```

4. **验证依赖者**
   ```bash
   ninja -C out/default third_party/rust/crates/which-rs:lib
   ```

### 8.2 兼容性检查

| 检查项 | 方法 |
|--------|------|
| API 兼容性 | 对比上游 CHANGELOG |
| 特性兼容性 | 检查 `features` 配置是否仍适用 |
| 构建兼容性 | 运行完整构建 |
| 运行时兼容性 | 运行依赖者测试 |

---

## 9. 调试与诊断

### 9.1 查看编译输出

```bash
# 查看详细的 rustc 命令
ninja -C out/default -v third_party/rust/crates/either:lib
```

### 9.2 检查生成的 rlib

```bash
# 使用 llvm-ar 查看 rlib 内容
llvm-ar t out/default/obj/third_party/rust/crates/either/libeither.rlib

# 使用 rustc 查看元数据
rustc --print=crate-name out/default/obj/third_party/rust/crates/either/libeither.rlib
```

---

## 10. 总结

| 方面 | 状态 |
|------|------|
| 构建复杂度 | 极低 |
| 配置特殊度 | 无 |
| 跨平台支持 | 良好 |
| 升级难度 | 低 |
| 维护成本 | 极低 |

**结论**: either 的 BUILD.gn 是一个**标准的、极简的**配置，使用 `ohos_cargo_crate` 模板即可满足需求。无特殊编译选项，无平台适配，是典型的零修改集成范例。
