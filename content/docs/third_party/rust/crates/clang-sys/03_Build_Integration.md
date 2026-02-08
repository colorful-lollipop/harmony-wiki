# OH 构建适配

## 概述

OpenHarmony 使用 GN（Generate Ninja）构建系统，而非 Rust 生态常用的 Cargo。因此需要将 Cargo-based 的 Rust 库适配为 GN 构建。

**适配方式**: 使用 `ohos_cargo_crate` GN 模板，通过配置而非 Patch 实现适配。

## BUILD.gn 完整解析

### 文件位置
```
third_party/rust/crates/clang-sys/BUILD.gn
```

### 完整配置

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ... (License Header)

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
  crate_name = "clang_sys"
  
  crate_type = "rlib"
  
  visibility = [ "//third_party/rust/crates/*" ]
  crate_root = "src/lib.rs"
  
  sources = [ "src/lib.rs" ]
  edition = "2015"
  
  deps = [
    "//third_party/rust/crates/glob:lib",
    "//third_party/rust/crates/libc:lib",
    "//third_party/rust/crates/libloading:lib",
  ]
  build_deps = [ "//third_party/rust/crates/glob:lib" ]
  
  features = [
    "clang_3_5",
    "clang_3_6",
    "clang_3_7",
    "clang_3_8",
    "clang_3_9",
    "clang_4_0",
    "clang_5_0",
    "clang_6_0",
    "libloading",
    "static",
  ]
  
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  # deps += [ "//build/rust/clanglibs" ]
  module_output_extension = ".rlib"
  part_name = "rust_clang_sys"
  subsystem_name = "thirdparty"
}
```

## 关键配置项详解

### 1. 基础配置

| 配置项 | 值 | 说明 |
|-------|-----|-----|
| `crate_name` | `clang_sys` | Rust crate 名称（下划线格式） |
| `crate_type` | `rlib` | Rust 库类型（rlib = Rust 静态库） |
| `crate_root` | `src/lib.rs` | crate 根文件 |
| `edition` | `2015` | Rust 版本（2015/2018/2021） |
| `visibility` | `//third_party/rust/crates/*` | 可见范围，限制为 Rust crates |

### 2. 源码配置

```gn
sources = [ "src/lib.rs" ]
```

**说明**: clang-sys 是单文件库，所有代码集中在 `src/lib.rs` 中。

### 3. 依赖配置

#### 运行时依赖 (deps)

```gn
deps = [
  "//third_party/rust/crates/glob:lib",       # 文件路径匹配
  "//third_party/rust/crates/libc:lib",       # C 类型定义
  "//third_party/rust/crates/libloading:lib", # 动态库加载
]
```

对应 Cargo.toml：
```toml
[dependencies]
glob = "0.3"
libc = { version = "0.2.39", default-features = false }
libloading = { version = "0.7", optional = true }
```

#### 构建依赖 (build_deps)

```gn
build_deps = [ "//third_party/rust/crates/glob:lib" ]
```

对应 Cargo.toml：
```toml
[build-dependencies]
glob = "0.3"
```

### 4. 特性配置 (features)

这是 OH 适配的核心，通过 features 控制编译行为：

```gn
features = [
  # Clang 版本支持（从 3.5 到 6.0）
  "clang_3_5",
  "clang_3_6",
  "clang_3_7",
  "clang_3_8",
  "clang_3_9",
  "clang_4_0",
  "clang_5_0",
  "clang_6_0",
  
  # 功能特性
  "libloading",  # 启用动态库加载支持
  "static",      # 使用静态链接
]
```

#### 与上游的差异

| 特性 | 上游默认 | OH BUILD.gn | 说明 |
|-----|---------|-------------|-----|
| clang_3_5 | ✓ | ✓ | 基础版本支持 |
| clang_3_6-6_0 | ✗ | ✓ | OH 启用更多版本 |
| clang_7_0-16_0 | ✗ | ✗ | OH 未启用（暂不需要） |
| runtime | ✗ | ✗ | 运行时加载 |
| static | ✗ | ✓ | **静态链接** |
| libloading | ✗(optional) | ✓ | 动态库加载能力 |

**关键决策**: 
- 启用 `static`: OH 构建系统偏好静态链接
- 启用 `libloading`: 即使静态链接，也保留动态库检测能力
- 限制 Clang 版本: 3.5-6.0 覆盖 OH 当前使用的 Clang 版本

### 5. 构建脚本配置

```gn
build_root = "build.rs"
build_sources = [ "build.rs" ]
```

build.rs 是 Cargo 构建脚本，在 OH GN 构建中也会被执行。

### 6. 注释掉的配置

```gn
# deps += [ "//build/rust/clanglibs" ]
```

**说明**: 这行被注释的代码暗示未来可能接入 OH 内置的 Clang 库，而非依赖系统安装的 libclang。

### 7. 模块元数据

```gn
module_output_extension = ".rlib"
part_name = "rust_clang_sys"
subsystem_name = "thirdparty"
```

对应 bundle.json 中的配置。

## 与上游构建系统对比

### 构建流程对比

```
上游 (Cargo)                    OH (GN)
    │                              │
    ▼                              ▼
Cargo.toml ─────>          BUILD.gn
    │                              │
    ▼                              ▼
cargo build                gn gen + ninja
    │                              │
    ▼                              ▼
自动处理依赖               依赖预定义在 BUILD.gn
    │                              │
    ▼                              ▼
自动运行 build.rs          配置 build_root
    │                              │
    ▼                              ▼
输出 .rlib/.so             输出到 out 目录
```

### 配置映射表

| Cargo 概念 | GN 配置 | clang-sys 实例 |
|-----------|---------|----------------|
| `[package] name` | `crate_name` | `clang_sys` |
| `[dependencies]` | `deps` | `glob`, `libc`, `libloading` |
| `[build-dependencies]` | `build_deps` | `glob` |
| `[features]` | `features` | `clang_3_5`...`clang_6_0`, `static` |
| `build = "build.rs"` | `build_root` | `build.rs` |
| `edition` | `edition` | `2015` |

## build.rs 行为

### 构建脚本功能

build.rs 在编译时执行，主要功能：

1. **检测 libclang**: 查找系统中的 libclang 库
2. **配置链接**: 根据 features 决定动态/静态链接
3. **输出配置**: 设置 `cargo:include` 等编译配置

### OH 环境下的行为

由于 OH 启用了 `static` feature（但未启用 `runtime`），build.rs 会：

```rust
#[cfg(not(feature = "runtime"))]
fn main() {
    if cfg!(feature = "static") {
        // r#static::link();  // 注释掉了！
    } else {
        dynamic::link();
    }
    // ...
}
```

**注意**: 实际静态链接逻辑被注释掉了，OH 可能通过其他方式处理链接。

## 构建适配的价值

### 对 OH 的价值

1. **统一构建系统**: 所有组件（C/C++/Rust）使用同一套 GN 构建
2. **依赖可追踪**: GN 的依赖图完整，便于分析和优化
3. **增量构建**: Ninja 支持高效的增量编译

### 对 Rust 生态的价值

1. **零侵入**: 不修改 Rust 源码，保持与上游兼容
2. **可更新**: 上游更新后只需同步源码，配置通常不变
3. **透明**: Rust 开发者无感知，使用方式与 Cargo 一致

## 维护指南

### 修改 BUILD.gn 的场景

| 场景 | 操作 |
|-----|-----|
| 上游更新 dependencies | 同步更新 `deps` 和 `build_deps` |
| 需要更多 Clang 版本 | 扩展 `features` 列表 |
| 更改链接方式 | 调整 `features`（添加/移除 `static`） |
| 添加 OH 特有源文件 | 修改 `sources` |

### 验证构建

```bash
# 在 OH 源码根目录执行
hb build //third_party/rust/crates/clang-sys:lib

# 或构建依赖该库的目标
hb build //third_party/rust/crates/bindgen/bindgen:lib
```
