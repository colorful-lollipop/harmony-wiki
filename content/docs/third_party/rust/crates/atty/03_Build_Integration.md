# 03 - OH 构建适配

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
    crate_name = "atty"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "0.2.14"
    cargo_pkg_authors = "softprops <d.tangren@gmail.com>"
    cargo_pkg_name = "atty"
    cargo_pkg_description = "A simple interface for querying atty"
    external_deps = [ "rust_libc:lib" ]
    module_output_extension = ".rlib"
    part_name = "rust_atty"
    subsystem_name = "thirdparty"
}
```

## 配置项详解

### 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | "atty" | Rust crate 名称 |
| `crate_type` | "rlib" | 生成 Rust 静态库 |
| `crate_root` | "src/lib.rs" | 库入口文件 |
| `sources` | ["src/lib.rs"] | 源文件列表 |

### Rust 版本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `edition` | "2015" | Rust 2015 edition |

**说明**: atty 使用 Rust 2015 edition，这是较老的版本。OH 的 Rust 工具链向后兼容 2015 edition，因此无需修改。

### Cargo 元数据

| 配置项 | 值 | 来源 |
|--------|-----|------|
| `cargo_pkg_version` | "0.2.14" | Cargo.toml version |
| `cargo_pkg_authors` | "softprops <d.tangren@gmail.com>" | Cargo.toml authors |
| `cargo_pkg_name` | "atty" | Cargo.toml name |
| `cargo_pkg_description` | "A simple interface for querying atty" | Cargo.toml description |

**作用**: 这些元数据用于生成 Cargo 兼容的编译环境。

### 依赖配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `external_deps` | [ "rust_libc:lib" ] | 外部依赖 |

**依赖映射关系**:

```
上游 Cargo.toml:
  [target.'cfg(unix)'.dependencies]
  libc = { version = "0.2", default-features = false }

OH BUILD.gn:
  external_deps = [ "rust_libc:lib" ]
```

### OH 组件标识

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `part_name` | "rust_atty" | OH 组件名 |
| `subsystem_name` | "thirdparty" | 所属子系统 |
| `module_output_extension` | ".rlib" | 输出文件扩展名 |

## 与上游构建系统的差异

### 上游构建方式

**Cargo 构建**:
```bash
cargo build --release
```

**输出**:
- `target/release/libatty.rlib` (Linux)
- `target/release/libatty.dll` (Windows)

### OH 构建方式

**GN/Ninja 构建**:
```bash
gn gen out
ninja -C out third_party/rust/crates/atty:lib
```

**输出**:
- `out/.../libatty.rlib`

### 差异对比

| 方面 | 上游 (Cargo) | OH (GN) |
|------|--------------|---------|
| 构建工具 | Cargo | GN + Ninja |
| 配置格式 | Cargo.toml | BUILD.gn |
| 依赖管理 | crates.io | OH 组件系统 |
| libc 来源 | crates.io | rust_libc (OH) |
| 输出路径 | target/ | out/ |

## 关键编译选项分析

### 编译器标志

该库 BUILD.gn **未设置**以下常用配置：

| 配置类型 | 配置项 | 使用情况 | 说明 |
|----------|--------|----------|------|
| defines | - | 未使用 | 无自定义宏 |
| configs | - | 未使用 | 无特殊编译选项 |
| cflags | - | 未使用 | 使用默认值 |
| rustflags | - | 未使用 | 使用默认值 |

**原因分析**:
- atty 代码简洁，无需特殊编译选项
- 不依赖特定 CPU 特性
- 无 unsafe 代码需要特殊处理（除了必要的 FFI）

### 与 Cargo.toml 的映射

**上游 Cargo.toml**:
```toml
[package]
name = "atty"
version = "0.2.14"
edition = "2015"

[target.'cfg(unix)'.dependencies]
libc = { version = "0.2", default-features = false }
```

**OH 对应处理**:

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `name` | `crate_name` | 直接映射 |
| `version` | `cargo_pkg_version` | 直接映射 |
| `edition` | `edition` | 直接映射 |
| `libc` dep | `external_deps = ["rust_libc:lib"]` | 映射到 OH 组件 |

## 特殊处理说明

### 1. 无特殊处理

atty 的 BUILD.gn 是标准的 `ohos_cargo_crate` 模板应用，**无任何特殊处理**：

- ✅ 无自定义 defines
- ✅ 无特殊编译器选项
- ✅ 无额外的源文件
- ✅ 无平台特定配置

### 2. 依赖处理

**外部依赖映射**:
```
上游: libc crate (crates.io)
   ↓
OH: rust_libc 组件 (//third_party/rust/crates/libc)
```

**版本兼容性**:
- atty 要求: `libc = "0.2"`
- OH rust_libc: 版本匹配

### 3. 平台支持

atty 通过条件编译支持多平台：

```rust
#[cfg(all(unix, not(target_arch = "wasm32")))]
// Unix/Linux 实现 - OpenHarmony 使用此路径

#[cfg(target_os = "hermit")]
// Hermit OS 实现

#[cfg(windows)]
// Windows 实现

#[cfg(target_arch = "wasm32")]
// WebAssembly 实现
```

**OH 构建**:
- 目标平台: Linux/ARM64 或 Linux/x86_64
- 自动选择 Unix 代码路径
- 无需额外配置

## 构建示例

### 完整构建命令

```bash
# 1. 进入 OH 源码目录
cd /path/to/ohos

# 2. 生成构建配置
./build.sh --product {product_name} --ccache

# 3. 单独构建 atty
ninja -C out/{product_name} third_party/rust/crates/atty:lib
```

### 验证构建结果

```bash
# 检查输出文件
ls out/{product_name}/.../libatty.rlib

# 检查符号表
rust-nm out/{product_name}/.../libatty.rlib | grep atty
```

## 升级构建配置指南

### 版本升级时的 BUILD.gn 修改

当升级上游版本时，可能需要更新以下字段：

```gn
ohos_cargo_crate("lib") {
    # 需要更新的字段
    cargo_pkg_version = "0.2.15"  # 新版本号
    
    # 通常不变的字段
    crate_name = "atty"
    crate_type = "rlib"
    sources = ["src/lib.rs"]
    edition = "2015"
    external_deps = [ "rust_libc:lib" ]
    part_name = "rust_atty"
    subsystem_name = "thirdparty"
}
```

### 新增依赖的处理

如果新版本增加了依赖：

```gn
# 在 external_deps 中添加
external_deps = [
    "rust_libc:lib",
    "rust_new_dep:lib",  # 新增依赖
]
```

## 总结

atty 的 BUILD.gn 展示了 OH 中 Rust crate 的标准集成方式：

- ✅ 使用 `ohos_cargo_crate` 模板
- ✅ 映射 Cargo 依赖到 OH 组件
- ✅ 无需特殊编译选项
- ✅ 维护简单，升级方便
