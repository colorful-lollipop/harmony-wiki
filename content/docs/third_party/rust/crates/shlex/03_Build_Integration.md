# shlex OH 构建适配

## 概述

shlex 在 OpenHarmony 中的构建适配主要通过以下文件完成：

1. **BUILD.gn** - GN 构建系统配置
2. **bundle.json** - OH 部件化配置
3. **README.OpenSource** - 开源协议追踪

本章详细说明 OH 构建系统的适配方式。

---

## BUILD.gn 结构说明

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
    crate_name = "shlex"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "1.1.0"
    cargo_pkg_authors = "comex <comexk@gmail.com>, Fenhl <fenhl.net>"
    cargo_pkg_name = "shlex"
    cargo_pkg_description = "Split a string into shell words, like Python's shlex."
    features = ["std"]
    module_output_extension = ".rlib"
    part_name = "rust_shlex"
    subsystem_name = "thirdparty"
}
```

### 关键配置项解析

| 配置项 | 值 | 说明 |
|-------|---|------|
| `crate_name` | `"shlex"` | Rust crate 名称 |
| `crate_type` | `"rlib"` | Rust 静态库（rlib） |
| `crate_root` | `"src/lib.rs"` | crate 根文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |
| `edition` | `"2015"` | Rust 版本（2015 edition） |
| `cargo_pkg_version` | `"1.1.0"` | Cargo.toml 中的版本号 |
| `cargo_pkg_authors` | `"comex <...>, Fenhl <...>"` | Cargo.toml 中的作者信息 |
| `cargo_pkg_name` | `"shlex"` | Cargo.toml 中的包名 |
| `cargo_pkg_description` | `"Split a string..."` | Cargo.toml 中的描述 |
| `features` | `["std"]` | 启用的 features（std feature） |
| `module_output_extension` | `".rlib"` | 输出文件扩展名 |
| `part_name` | `"rust_shlex"` | OH 部件名称 |
| `subsystem_name` | `"thirdparty"` | OH 子系统名称 |

### GN 模板说明

**`ohos_cargo_crate`**:

这是 OH 提供的 GN 模板，用于编译 Rust crate。该模板会：

1. 调用 Rust 编译器（rustc）
2. 处理 Cargo.toml 中的依赖关系
3. 生成 `.rlib` 文件（Rust 静态库）
4. 将输出集成到 OH 构建系统

---

## 关键编译选项

### 1. features 配置

```gn
features = ["std"]
```

**说明**:
- 启用 Rust 标准库支持
- 这是 shlex 的默认 feature
- OH 编译环境支持 Rust std

**与上游 Cargo.toml 对应**:

```toml
[features]
std = []
default = ["std"]
```

**为何选择 `std`**:
- shlex 默认启用 std
- OH 编译环境支持 std
- no_std 模式不适用于 OH 场景

### 2. crate_type 配置

```gn
crate_type = "rlib"
```

**说明**:
- `rlib`: Rust 静态库（Rust Library）
- 与 `dylib`（动态库）不同，rlib 仅用于 Rust-to-Rust 链接
- 不能被非 Rust 代码直接使用

**输出文件**:
- 编译后生成 `libshlex-*.rlib` 文件
- 文件位于 OH 编译输出目录中

### 3. module_output_extension 配置

```gn
module_output_extension = ".rlib"
```

**说明**:
- 明确指定输出文件扩展名为 `.rlib`
- 确保 GN 构建系统正确识别 Rust 库

### 4. part_name 和 subsystem_name 配置

```gn
part_name = "rust_shlex"
subsystem_name = "thirdparty"
```

**说明**:
- `part_name`: OH 部件名称，用于部件管理
- `subsystem_name`: 归属的子系统（thirdparty）
- 这些信息用于 OH 构建系统的组织和依赖管理

---

## 与上游构建系统的差异

### 上游构建系统（Cargo）

**Cargo.toml**:

```toml
[package]
name = "shlex"
version = "1.1.0"
authors = ["comex <comexk@gmail.com>", "Fenhl <fenhl.net>"]
license = "MIT OR Apache-2.0"
repository = "https://github.com/comex/rust-shlex"
description = "Split a string into shell words, like Python's shlex."
categories = ["command-line-interface", "parser-implementations"]

[features]
std = []
default = ["std"]
```

**上游构建命令**:
```bash
cargo build --release
```

**上游输出**:
- `target/release/libshlex.rlib`
- 自动处理依赖
- 运行测试套件

### OH 构建系统（GN）

**BUILD.gn**:
- 复制 Cargo.toml 中的关键信息
- 使用 `ohos_cargo_crate` 模板
- 添加 OH 特定配置（part_name, subsystem_name）

**OH 构建命令**:
```bash
hb build -f
```

**OH 输出**:
- `out/.../rust_shlex/libshlex-*.rlib`
- 集成到 OH 构建系统
- 可被其他 OH 组件依赖

### 主要差异

| 方面 | Cargo（上游） | GN（OH） |
|-----|--------------|---------|
| **配置文件** | Cargo.toml | BUILD.gn + bundle.json |
| **构建工具** | cargo | ohos_cargo_crate 模板 |
| **依赖管理** | Cargo 自动解析 | GN 依赖系统 |
| **输出位置** | target/release/ | out/.../rust_shlex/ |
| **测试支持** | 自动运行 | 需单独配置 |
| **部件化** | 无 | 有（bundle.json） |

---

## 特殊处理

### 1. 无禁用功能

shlex 在 OH 中**没有禁用任何上游功能**：

- std feature：已启用（默认）
- 所有公开 API：全部保留
- 测试用例：代码中保留（但 OH 构建不运行）

### 2. 无添加 OH 特定源文件

OH **没有添加任何特定源文件**：

- 所有源文件来自上游
- 仅新增配置文件（BUILD.gn, bundle.json 等）
- 代码保持原样

### 3. 无条件编译宏

代码中**无 OH 特定条件编译**：

```rust
// 代码中无类似以下内容：
// #[cfg(feature = "ohos")]
// #[cfg(target_os = "openharmony")]
```

**原因**:
- shlex 是平台无关的库
- 无需针对 OH 做特殊处理
- 标准库接口在 OH 中完全兼容

### 4. 无特殊编译标志

BUILD.gn 中**没有添加特殊编译标志**：

```gn
// 无类似以下内容：
// rustflags = ["-C", "target-feature=..."]
// cflags = ["-DOHOS"]
```

---

## 构建流程

### 1. OH 构建流程图

```mermaid
graph LR
    A[OH 构建系统] --> B[BUILD.gn]
    B --> C[ohos_cargo_crate 模板]
    C --> D[Rust 编译器 rustc]
    D --> E[libshlex-*.rlib]
    E --> F[其他 OH 组件]
    F --> G[最终产物]
```

### 2. 依赖解析

shlex 的依赖关系：

```
ohos_cargo_crate("lib") {
    // 无外部依赖
    // shlex 是零依赖库
}
```

**说明**:
- shlex 无外部依赖
- OH 构建系统无需处理依赖
- 编译速度快

### 3. 编译输出

**输出目录**:
```
out/ohos-arm64/rust_shlex/
├── libshlex-1.1.0.rlib
└── ...
```

**输出文件**:
- `libshlex-1.1.0.rlib`: Rust 静态库
- 可被其他 Rust crate 链接

---

## 集成到 OH 构建系统

### 1. bundle.json 部件化

`bundle.json` 将 shlex 定义为 OH 部件：

```json
{
  "name": "@ohos/rust_shlex",
  "description": "A Rust library that provides support for parsing shell-like syntax",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/shlex"
  },
  "component": {
    "name": "rust_shlex",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/shlex:lib"
        }
      ],
      "test": []
    }
  }
}
```

### 2. 暴露的内部库

```json
"inner_kits": [
  {
    "name": "//third_party/rust/crates/shlex:lib"
  }
]
```

**说明**:
- 暴露 `//third_party/rust/crates/shlex:lib` 作为内部库
- 其他 OH 组件可通过此路径依赖 shlex

### 3. 被其他组件依赖

shlex 被以下 OH 组件依赖：

| 组件 | 依赖方式 | 用途 |
|-----|---------|------|
| bindgen | Cargo.toml 间接依赖 | 解析编译器参数 |
| clap | Cargo.toml 间接依赖 | 解析命令行参数 |

---

## 总结

### 构建适配特点

1. **无源代码修改**: 完全通过配置文件完成适配
2. **零依赖**: 无需处理复杂的依赖关系
3. **标准接口**: 完全遵循 Rust 标准库接口
4. **简单直接**: BUILD.gn 配置清晰明了

### 适配质量

- **完整性**: 100%（上游功能全部保留）
- **兼容性**: 100%（无破坏性修改）
- **维护成本**: 低（无需维护 Patch）

### 最佳实践

shlex 的构建适配可以作为**纯 Rust 库集成 OH 的最佳实践**：

1. 使用 `ohos_cargo_crate` 模板
2. 保持源代码不变
3. 仅通过配置文件完成适配
4. 充分利用 OH 的部件化机制
