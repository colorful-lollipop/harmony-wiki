# OH 构建适配

## 构建系统概述

memoffset 使用 OpenHarmony 的 GN 构建系统进行编译，通过 `ohos_cargo_crate` 模板集成到 OH 构建流程中。

### 构建模板

```gn
import("//build/templates/rust/ohos_cargo_crate.gni")
```

`ohos_cargo_crate` 是 OpenHarmony 为 Rust crates 提供的标准构建模板，封装了 cargo 编译、依赖管理、输出处理等逻辑。

## BUILD.gn 配置详解

### 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/templates/rust/ohos_cargo_crate.gni")

ohos_cargo_crate("lib") {
    # 基础信息
    crate_name = "memoffset"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    # 源文件配置
    sources = ["src/lib.rs"]

    # Rust 版本配置
    edition = "2015"

    # 包元数据
    cargo_pkg_version = "0.9.1"
    cargo_pkg_authors = "Gilad Naaman <gilad.naaman@gmail.com>"
    cargo_pkg_name = "memoffset"
    cargo_pkg_description = "offset_of functionality for Rust structs."

    # 构建依赖
    build_deps = ["//third_party/rust/crates/autocfg:lib"]

    # 特性配置
    features = ["default"]

    # 构建脚本配置
    build_root = "build.rs"
    build_sources = ["build.rs"]
    build_script_outputs = ["probe0.ll"]

    # 输出配置
    module_output_extension = ".rlib"

    # 部件配置
    part_name = "rust_autocfg"
    subsystem_name = "thirdparty"
}
```

### 配置项详解

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `crate_name` | memoffset | Rust crate 内部名称 |
| `crate_type` | rlib | Rust 静态库类型 |
| `crate_root` | src/lib.rs | crate 入口文件 |
| `edition` | 2015 | Rust Edition 版本 |
| `cargo_pkg_version` | 0.9.1 | 与 Cargo.toml 版本一致 |
| `cargo_pkg_name` | memoffset | cargo 包名称 |
| `build_deps` | autocfg | 版本检测构建依赖 |
| `features` | default | 启用的 cargo features |
| `part_name` | rust_autocfg | 所属 OH 部件 |

## 与上游构建系统的差异

### 上游构建配置

上游使用标准的 Cargo 构建系统：

```toml
# Cargo.toml
[package]
name = "memoffset"
version = "0.9.1"
edition = "2015"

[build-dependencies]
autocfg = "1"

[features]
default = []
unstable_offset_of = []
unstable_const = []
```

### 差异对比

| 维度 | 上游 | OH |
|-----|------|-----|
| 构建系统 | Cargo | GN + ohos_cargo_crate |
| 输出格式 | .rlib | .rlib |
| 依赖管理 | Cargo.toml | BUILD.gn deps |
| 部件归属 | 无 | rust_autocfg 部件 |
| 特殊配置 | 无 | 与上游一致 |

### 适配策略

OH 的 BUILD.gn 配置**与上游 Cargo.toml 保持高度一致**：

| 适配项 | 处理方式 |
|-------|---------|
| 源文件 | 直接使用 `sources = ["src/lib.rs"]` |
| 版本 | `cargo_pkg_version` 同步上游 |
| Edition | 使用 Rust 2015，兼容老工具链 |
| 构建依赖 | `build_deps` 指向 OH 的 autocfg |
| 特性 | `features = ["default"]` 使用默认特性 |

## 编译选项分析

### 内联配置（无）

memoffset 没有定义 OH 特定的编译宏：

```gn
# 无以下配置
defines = []          // 无 OH 特有宏
configs = []          // 无特殊配置
cflags = []           // 无 C 编译器标志
rustflags = []        // 无 Rust 编译器标志
```

### 构建脚本输出

```gn
build_script_outputs = ["probe0.ll"]
```

`probe0.ll` 是 build.rs 通过 autocfg 检测 Rust 版本后生成的 LLVM IR 探针文件，用于验证版本检测逻辑。

## 依赖关系

### 构建依赖

```gn
build_deps = ["//third_party/rust/crates/autocfg:lib"]
```

| 依赖项 | 用途 | 来源 |
|-------|------|------|
| autocfg | Rust 版本检测 | //third_party/rust/crates/autocfg |

### autocfg 依赖详解

autocfg 是一个简单的 Rust 版本检测工具：

```rust
// build.rs
extern crate autocfg;

fn main() {
    let ac = autocfg::new();

    // 检测 Rust 版本并输出 cargo:rustc-cfg
    if ac.probe_rustc_version(1, 20) {
        println!("cargo:rustc-cfg=tuple_ty");
    }
    // ... 其他版本检测
}
```

**检测的版本特性**：

| Rust 版本 | 启用的 cfg | 用途 |
|-----------|-----------|------|
| 1.20+ | tuple_ty | 元组类型支持 |
| 1.31+ | allow_clippy | Clippy 允许 |
| 1.36+ | maybe_uninit | MaybeUninit 支持 |
| 1.40+ | doctests | 文档测试 |
| 1.51+ | raw_ref_macros | 原始引用宏 |
| 1.65+ | stable_const | 稳定常量求值 |
| 1.77+ | stable_offset_of | 稳定 offset_of |

## 输出配置

### 模块输出

```gn
module_output_extension = ".rlib"
```

生成的输出文件为 `librlibmemoffset.rlib`，作为静态库链接到依赖方。

### 打包配置

```gn
part_name = "rust_autocfg"
subsystem_name = "thirdparty"
```

该库被封装为 `rust_autocfg` 部件的一部分，属于 `thirdparty` 子系统。

## 构建流程

### OH 构建步骤

```
1. GN 解析
   └── 读取 BUILD.gn 配置
       └── 确定 crate_name、edition、sources 等

2. Cargo 配置生成
   └── 生成临时 Cargo.toml
       └── 注入版本信息和依赖

3. 构建脚本执行
   └── 运行 build.rs
       └── 使用 autocfg 检测 Rust 版本
           └── 输出 probe0.ll 和 cargo:rustc-cfg

4. Rust 编译
   └── cargo build --release
       └── 编译为 .rlib 静态库

5. 输出处理
   └── 复制输出到 build 目录
       └── 供依赖方链接使用
```

### 与上游构建对比

| 步骤 | 上游构建 | OH 构建 |
|-----|---------|---------|
| 1 | cargo init | GN 解析 |
| 2 | cargo metadata | 生成 Cargo.toml |
| 3 | cargo build | cargo build |
| 4 | 生成 target/.../libmemoffset.rlib | 输出到 build/ |

## 特性配置

### 默认特性

```gn
features = ["default"]
```

上游 Cargo.toml 中 `default = []`，表示无默认启用的特性。

### 可用特性

| 特性 | 状态 | 说明 |
|------|------|------|
| default | 空 | 无特殊功能 |
| unstable_offset_of | 不可用 | 已废弃 |
| unstable_const | 不可用 | 已废弃 |

**说明**：由于该库版本较旧，部分 unstable 特性已被移除或不再需要。

## 构建验证

### 验证命令

```bash
# 在 OH 构建环境中
hb build //third_party/rust/crates/memoffset:lib

# 或直接使用 GN
gn gen out/rk3568
ninja -C out/rk3568 third_party/rust/crates/memoffset:lib
```

### 输出产物

| 文件 | 说明 |
|------|------|
| `librlibmemoffset.rlib` | Rust 静态库 |
| `probe0.ll` | 版本检测探针文件 |

## 常见问题

### Q1: 为何使用 Rust 2015 edition？

memoffset v0.9.1 发布时，Rust 2018 尚未成为主流。该版本使用 2015 edition 是为了保持与上游一致。

**影响**：
- 语法兼容性问题：使用较老的语法风格
- 工具链支持：需要支持 Rust 1.19+ 的工具链

### Q2: build_script_outputs 的作用是什么？

`probe0.ll` 是 LLVM IR 探针文件，用于验证 Rust 版本检测逻辑是否正确工作。

**生成时机**：build.rs 执行期间
**用途**：构建系统验证
**可删除**：是，但下次构建会重新生成

### Q3: part_name 为什么是 rust_autocfg？

memoffset 的构建依赖 autocfg，而 autocfg 是 Rust 工具链配置库。OH 将 memoffset 归入 rust_autocfg 部件是出于依赖管理的考虑。

**建议**：如需独立管理，可考虑将 part_name 改为 rust_memoffset

## 维护建议

### 版本升级

升级 memoffset 版本时：

1. **同步版本号**：更新 `cargo_pkg_version` 和 `version` in bundle.json
2. **验证兼容性**：确保新版本与当前 Rust 工具链兼容
3. **检查依赖变更**：如依赖有变化，更新 `build_deps`
4. **测试构建**：在 OH 构建环境中验证

### 配置优化建议

| 建议 | 优先级 | 说明 |
|------|-------|------|
| 更新 Edition | 低 | Rust 2021 可能带来更好的优化 |
| 独立部件 | 中 | 独立 part_name 便于管理 |
| 特性暴露 | 低 | 如需要，暴露更多 cargo features |
