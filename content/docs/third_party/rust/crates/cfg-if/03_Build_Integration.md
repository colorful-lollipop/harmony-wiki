# OH 构建适配

## 构建系统集成

### 构建模板

该库使用 OpenHarmony 的 `ohos_cargo_crate` 模板进行构建集成，这是专门为 Rust cargo crate 设计的构建模板。

### BUILD.gn 配置

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
    crate_name = "cfg_if"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "1.0.0"
    cargo_pkg_authors = "Alex Crichton <alex@alexcrichton.com>"
    cargo_pkg_name = "cfg-if"
    cargo_pkg_description = "A macro to ergonomically define an item depending on a large number of #[cfg]parameters. Structured like an if-else chain, the first matching branch is theitem that gets emitted."
    module_output_extension = ".rlib"
    part_name = "rust_cfg_if"
    subsystem_name = "thirdparty"
}
```

## 配置项详解

### 核心配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `cfg_if` | Crate 内部名称 |
| `crate_type` | `rlib` | Rust 静态库类型 |
| `crate_root` | `src/lib.rs` | 入口源文件 |
| `edition` | `2018` | Rust Edition 版本 |
| `part_name` | `rust_cfg_if` | OH 组件名称 |
| `subsystem_name` | `thirdparty` | 所属子系统 |

### 包元数据

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cargo_pkg_version` | `1.0.0` | 与上游版本同步 |
| `cargo_pkg_authors` | `Alex Crichton` | 原始作者信息 |
| `cargo_pkg_name` | `cfg-if` | 上游包名称 |
| `cargo_pkg_description` | - | 原始描述 |

### 无特殊配置项

与其他 Rust 库相比，该库没有以下配置：

| 配置项 | 状态 | 说明 |
|--------|------|------|
| `defines` | 无 | 无需定义额外的编译宏 |
| `configs` | 无 | 无需自定义配置 |
| `flags` | 无 | 无需额外的编译标志 |
| `external_deps` | 无 | 无需外部依赖 |
| `features` | 无 | 无需启用/禁用特性 |

## Cargo.toml 配置

### 原始配置

```toml
[package]
name = "cfg-if"
version = "1.0.0"
authors = ["Alex Crichton <alex@alexcrichton.com>"]
license = "MIT/Apache-2.0"
readme = "README.md"
repository = "https://github.com/alexcrichton/cfg-if"
homepage = "https://github.com/alexcrichton/cfg-if"
documentation = "https://docs.rs/cfg-if"
description = """
A macro to ergonomically define an item depending on a large number of #[cfg]
parameters. Structured like an if-else chain, the first matching branch is the
item that gets emitted.
"""
edition = "2018"

[badges]
travis-ci = { repository = "alexcrichton/cfg-if" }

[dependencies]
core = { version = "1.0.0", optional = true, package = 'rustc-std-workspace-core' }
compiler_builtins = { version = '0.1.2', optional = true }

[features]
rustc-dep-of-std = ['core', 'compiler_builtins']
```

### 配置说明

| 配置块 | 说明 |
|--------|------|
| `[dependencies]` | 可选依赖，仅在 `rustc-dep-of-std` 特性启用时使用 |
| `[features]` | 定义 `rustc-dep-of-std` 特性，用于 Rust 标准库开发 |

## 与上游构建的差异

### 差异对比表

| 维度 | 上游构建 | OH 构建 |
|------|----------|---------|
| **构建工具** | Cargo | GN + Ninja（通过 ohos_cargo_crate） |
| **输出格式** | rlib | rlib（相同） |
| **入口文件** | src/lib.rs | src/lib.rs（相同） |
| **Edition** | 2018 | 2018（相同） |
| **依赖管理** | Cargo.toml | BUILD.gn + Cargo.toml |
| **元数据** | Cargo.toml | BUILD.gn 中的镜像配置 |

### 关键差异说明

1. **构建系统切换**：上游使用标准的 Cargo 构建，OH 使用 GN + Ninja 构建系统
2. **元数据映射**：BUILD.gn 中的 `cargo_pkg_*` 字段从 Cargo.toml 镜像而来
3. **输出目录**：上游输出到 `target/`，OH 输出到构建系统指定的目录

## 组件声明

### bundle.json 配置

```json
{
  "name": "@ohos/rust_cfg_if",
  "description": "A macro that allows conditional compilation based on a set of configuration options",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/cfg-if"
  },
  "component": {
    "name": "rust_cfg_if",
    "subsystem": "thirdparty",
    "adapted_system_type": [
      "standard"
    ],
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/cfg-if:lib"
        }
      ],
      "test": []
    }
  }
}
```

### 配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | `@ohos/rust_cfg_if` | OH NPM 风格名称 |
| `version` | `6.1` | OH 包版本（独立于上游版本号） |
| `subsystem` | `thirdparty` | 所属子系统 |
| `adapted_system_type` | `standard` | 适配的标准系统类型 |
| `inner_kits` | `:lib` 目标 | 导出库目标 |

## 编译注意事项

### 1. Edition 兼容性

当前使用 Rust 2018 Edition，如需升级到 2021 或更新 Edition：

```gn
edition = "2021"  # 修改此行
```

### 2. 特性启用

该库支持 `rustc-dep-of-std` 特性，如需启用：

```gn
ohos_cargo_crate("lib") {
    # ... 其他配置
    features = ["rustc-dep-of-std"]
}
```

### 3. 依赖管理

该库在 OH 中没有额外的依赖需要管理，仅通过标准的 cargo crate 机制处理。
