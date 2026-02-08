# io-lifetimes 构建适配

## BUILD.gn 完整分析

### 配置文件位置

```
third_party/rust/crates/io-lifetimes/BUILD.gn
```

### 完整配置内容

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
  crate_name = "io_lifetimes"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2018"
  cargo_pkg_version = "1.0.5"
  cargo_pkg_authors = "Dan Gohman <dev@sunfishcode.online>"
  cargo_pkg_name = "io-lifetimes"
  cargo_pkg_description = "A low-level I/O ownership and borrowing library"
  external_deps = [ "rust_libc:lib" ]
  features = [
    "close",
    "libc",
    "windows-sys",
  ]
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  build_script_outputs = [ "librust_out.rmeta" ]
  module_output_extension = ".rlib"
  part_name = "rust_io_lifetimes"
  subsystem_name = "thirdparty"
}
```

---

## 配置项详细分析

### 1. 基础构建配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `io_lifetimes` | Rust crate 名称（下划线格式） |
| `crate_type` | `rlib` | 输出类型：Rust 静态库 |
| `crate_root` | `src/lib.rs` | crate 入口文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表（lib.rs 会拉取其他模块） |
| `edition` | `2018` | Rust 语言版本 |

### 2. 包元数据

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cargo_pkg_version` | `1.0.5` | 上游版本号 |
| `cargo_pkg_authors` | `Dan Gohman...` | 上游作者 |
| `cargo_pkg_name` | `io-lifetimes` | 上游包名（连字符格式） |
| `cargo_pkg_description` | `A low-level...` | 包描述 |

这些元数据用于生成 Cargo 兼容的构建环境。

### 3. 依赖配置

```gn
external_deps = [ "rust_libc:lib" ]
```

**依赖映射分析**：

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `libc = { optional = true }` | `external_deps = ["rust_libc:lib"]` | 映射到 OH 的 libc crate |

**依赖来源**：
- `rust_libc` 是 OpenHarmony 的 `third_party/rust/crates/libc` 组件
- 通过 `ohos_cargo_crate` 模板自动处理依赖关系

### 4. Features 配置

```gn
features = [
  "close",       # 启用 close() 系统调用支持
  "libc",        # 启用 libc 集成
  "windows-sys", # 启用 Windows 支持
]
```

**Features 详细说明**：

#### `close` Feature

```rust
// src/types.rs (简化)
#[cfg(feature = "close")]
impl Drop for OwnedFd {
    fn drop(&mut self) {
        unsafe {
            libc::close(self.fd);
        }
    }
}

#[cfg(not(feature = "close"))]
impl Drop for OwnedFd {
    fn drop(&mut self) {
        unreachable!("drop called without the \"close\" feature in io-lifetimes");
    }
}
```

**作用**：控制 `OwnedFd` 是否在 drop 时调用 `close(2)`。

**OH 配置**：✅ 启用（默认且必需）

#### `libc` Feature

启用对 `libc` crate 的依赖，用于：
- 调用 `close()`、`dup()` 等系统调用
- 使用 `RawFd` 类型定义

**OH 配置**：✅ 启用

#### `windows-sys` Feature

启用 Windows 平台支持，依赖 `windows-sys` crate。

**OH 配置**：✅ 启用（即使 OH 不支持 Windows，也保持上游兼容）

### 5. Build Script 配置

```gn
build_root = "build.rs"
build_sources = [ "build.rs" ]
build_script_outputs = [ "librust_out.rmeta" ]
```

**build.rs 作用**：

```rust
// build.rs (io-lifetimes)
use std::env;

fn main() {
    // 检测 Rust 版本是否支持 io_safety_is_in_std
    let rustc = env::var("RUSTC").unwrap();
    let output = std::process::Command::new(&rustc)
        .arg("--version")
        .output()
        .unwrap();
    
    // 根据版本设置 cfg 标志
    // Rust >= 1.63: 使用 std 的 I/O Safety 类型
    // Rust < 1.63: 使用 io-lifetimes 自带类型
}
```

**输出文件**：`librust_out.rmeta` 是构建脚本生成的元数据。

### 6. OH 组件标识

```gn
part_name = "rust_io_lifetimes"
subsystem_name = "thirdparty"
```

用于 OH 的构建系统和组件管理。

---

## 与上游 Cargo.toml 的对比

### Cargo.toml（上游）

```toml
[package]
name = "io-lifetimes"
version = "1.0.5"
edition = "2018"

[features]
default = ["close"]
close = ["libc", "windows-sys"]

[target.'cfg(not(windows))'.dependencies]
libc = { version = "0.2.96", optional = true }

[target.'cfg(windows)'.dependencies.windows-sys]
version = "0.45.0"
optional = true
features = [...]
```

### 配置映射表

| Cargo.toml | BUILD.gn | 映射说明 |
|------------|----------|----------|
| `name = "io-lifetimes"` | `cargo_pkg_name = "io-lifetimes"` | 直接映射 |
| `version = "1.0.5"` | `cargo_pkg_version = "1.0.5"` | 直接映射 |
| `edition = "2018"` | `edition = "2018"` | 直接映射 |
| `libc` (optional) | `external_deps = ["rust_libc:lib"]` | 映射到 OH 组件 |
| `default = ["close"]` | `features = ["close", ...]` | 显式启用 |
| `close = ["libc", "windows-sys"]` | `features = ["close", "libc", "windows-sys"]` | 展开 features |

---

## OH 特有适配

### 1. 无额外适配

与许多其他 third_party 库不同，io-lifetimes **无需任何 OH 特有适配**：

- ❌ 无 OH 特有的 `sources` 添加
- ❌ 无特殊的 `cflags` / `defines`
- ❌ 无 `deps` 覆盖
- ❌ 无条件编译分支

### 2. 标准模板使用

完全遵循标准 `ohos_cargo_crate` 模板：

```gn
ohos_cargo_crate("lib") {
  # 标准配置...
}
```

### 3. 依赖关系

```
io-lifetimes BUILD.gn
    └── rust_libc:lib (external_deps)
            └── //third_party/rust/crates/libc:lib
```

---

## 构建流程

### 编译步骤

```
1. 解析 BUILD.gn
   └── 识别为 ohos_cargo_crate 模板

2. 处理依赖
   └── 确保 rust_libc 已编译

3. 执行 build.rs
   └── 检测 Rust 版本特性
   └── 生成 librust_out.rmeta

4. 编译 Rust 源码
   └── rustc --edition 2018 --crate-type rlib
   └── 启用 features: close, libc, windows-sys

5. 生成输出
   └── libio_lifetimes.rlib
```

### 输出产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libio_lifetimes.rlib` | `out/.../third_party/rust/crates/io-lifetimes/` | 静态库 |
| `librust_out.rmeta` | 构建目录 | 构建脚本输出 |

---

## 使用示例

### 在另一个 crate 中依赖 io-lifetimes

```gn
# other_crate/BUILD.gn
ohos_cargo_crate("lib") {
  crate_name = "my_crate"
  # ...
  deps = [
    "//third_party/rust/crates/io-lifetimes:lib",
  ]
}
```

### 在 Rust 代码中使用

```rust
// src/lib.rs
use io_lifetimes::{AsFd, FromFd, IntoFd, OwnedFd};

pub fn example() {
    // 使用 io-lifetimes 类型进行安全的 I/O 操作
}
```

---

## 常见问题

### Q1: 为什么启用 `windows-sys` feature？

OH 不支持 Windows，但启用该 feature 可以：
- 保持与上游代码的完全兼容
- 便于代码同步（无需修改 BUILD.gn）
- 该 feature 在 Linux 上无实际代码生成

### Q2: `sources` 只有 `lib.rs`，其他文件呢？

```gn
sources = [ "src/lib.rs" ]
```

Rust 编译器通过 `mod` 声明自动拉取其他文件：
```rust
// src/lib.rs
mod portability;  // 自动包含 src/portability.rs
mod traits;       // 自动包含 src/traits.rs
mod types;        // 自动包含 src/types.rs
// ...
```

### Q3: 如何升级到新版本？

1. 替换源码文件
2. 更新 `cargo_pkg_version`
3. 检查是否有新增 features 或依赖
4. 运行依赖该库的其他组件测试

---

## 文档导航

- [概览](01_Overview.md)
- [Patch 分析](02_Patches.md)
- **构建适配**（本页）
- [依赖与使用](04_Usage_in_OH.md)
