# OH 构建适配

> **说明**: 本文档说明 lazycell 在 OpenHarmony 中的构建系统适配。
> **核心内容**: BUILD.gn 配置、与上游构建系统的差异。

---

## BUILD.gn 结构

### 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0
# ...

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "lazycell"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "1.3.0"
    cargo_pkg_authors = "Alex Crichton <alex@alexcrichton.com>, Nikita Pekin <contact@nikitapek.in>"
    cargo_pkg_name = "lazycell"
    cargo_pkg_description = "A library providing a lazily filled Cell struct"
    module_output_extension = ".rlib"
    part_name = "rust_lazycell"
    subsystem_name = "thirdparty"
}
```

---

## 配置项详解

### ohos_cargo_crate 模板

`ohos_cargo_crate` 是 OH 专门用于构建 Rust crate 的 GN 模板，负责：

- 解析 `Cargo.toml` 元数据
- 调用 Rust 编译器（rustc）
- 生成静态库（`.rlib`）
- 处理依赖关系
- 集成到 OH 构建系统

### 核心配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `lazycell` | Rust crate 名称 |
| `crate_type` | `rlib` | 输出 Rust 静态库 |
| `crate_root` | `src/lib.rs` | crate 根文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |
| `edition` | `2015` | Rust Edition（与上游一致） |
| `cargo_pkg_version` | `1.3.0` | 版本号（与上游有差异） |
| `cargo_pkg_name` | `lazycell` | crate 名称 |
| `cargo_pkg_description` | "A library providing a lazily filled Cell struct" | 包描述 |
| `module_output_extension` | `.rlib` | 输出文件扩展名 |
| `part_name` | `rust_lazycell` | OH 构件名 |
| `subsystem_name` | `thirdparty` | OH 子系统 |

### 关键特性

#### 1. crate_type = "rlib"

- **含义**: 生成 Rust 静态库（Rlib）
- **用途**: 仅供 Rust 代码使用，无法被 C/C++ 直接链接
- **原因**: lazycell 是 Rust 专用库，无需提供 C ABI

#### 2. edition = "2015"

- **含义**: 使用 Rust 2015 Edition
- **原因**: lazycell 上游使用 2015 Edition，OH 保持一致
- **影响**: 代码风格和某些 API 与 2018/2021 Edition 不同

#### 3. 无额外配置

- **无 `deps`**: lazycell 无外部依赖（标准库除外）
- **无 `features`**: 未启用任何特性
- **无 `defines`**: 无编译宏定义
- **无 `configs`**: 无特殊编译配置

---

## 与上游构建系统的差异

### Cargo.toml vs BUILD.gn

| 项目 | Cargo.toml (上游) | BUILD.gn (OH) | 备注 |
|------|------------------|---------------|------|
| **包名** | `[package].name = "lazycell"` | `cargo_pkg_name = "lazycell"` | 一致 |
| **版本** | `version = "1.2.1"` | `cargo_pkg_version = "1.3.0"` | ⚠️ 不一致 |
| **类型** | `[lib]` (隐含) | `crate_type = "rlib"` | 一致 |
| **依赖** | `[dependencies]` (无) | `deps = []` | 一致 |
| **特性** | `[features]` | 无使用 | OH 未启用特性 |
| **作者** | `[package].authors` | `cargo_pkg_authors` | 一致 |
| **描述** | `[package].description` | `cargo_pkg_description` | 一致 |

### 构建流程对比

#### 上游构建流程（Cargo）

```bash
# 1. Cargo 解析 Cargo.toml
cargo build

# 2. 调用 rustc 编译
rustc --crate-type lib src/lib.rs \
      --edition 2015 \
      --cfg 'feature="default"'

# 3. 输出 target/debug/liblazycell.rlib
```

#### OH 构建流程（GN）

```bash
# 1. GN 解析 BUILD.gn
ohos_build //third_party/rust/crates/lazycell:lib

# 2. ohos_cargo_crate 调用 rustc
rustc --crate-type rlib \
      src/lib.rs \
      --edition 2015 \
      --crate-name lazycell

# 3. 输出 out/rust/x86_64/liblazycell-*.rlib
```

**关键差异**:

| 方面 | Cargo | OH (GN) |
|------|-------|----------|
| **配置文件** | `Cargo.toml` | `BUILD.gn` |
| **构建工具** | `cargo build` | `ohos_build` |
| **依赖管理** | Cargo 自动解析 | GN 手动声明 |
| **输出目录** | `target/` | `out/` |
| **测试执行** | `cargo test` | GN test 目标 |

---

## OH 特定适配

### 无 OH 特定源文件

lazycell 未添加任何 OH 特定的源文件：

```
lazycell/
├── src/
│   └── lib.rs          # 唯一源文件（与上游一致）
└── tests/
    └── lib.rs          # 测试代码（与上游一致）
```

### 无 OH 特定宏

```bash
# 搜索 OH 特定宏
$ grep -r "ohos\|OHOS\|#[cfg(ohos)" src/
# 结果：无

# 搜索条件编译
$ grep -r "cfg!(" src/
# 结果：仅有 cfg_attr(not(test), no_std)
```

### 无特殊编译选项

- **无 `defines`**: 无编译宏定义
- **无 `configs`**: 无特殊编译配置
- **无 `features`**: 未启用任何特性

---

## 编译产物

### 输出文件

```
out/rust/<arch>/
└── liblazycell-<hash>.rlib
```

- **类型**: Rust 静态库（.rlib）
- **内容**: 字节码、元数据、依赖信息
- **用途**: 仅供 Rust 代码链接使用

### 使用方式

#### 在 BUILD.gn 中依赖

```gn
ohos_cargo_crate("my_crate") {
    deps = [
        "//third_party/rust/crates/lazycell:lib"
    ]
}
```

#### 在 Rust 代码中使用

```rust
// Cargo.toml (如果使用 Cargo)
[dependencies]
lazycell = "1.2.1"

// src/lib.rs
use lazycell::LazyCell;

let cell = LazyCell::new();
```

---

## 测试构建

### BUILD.gn 测试目标

lazycell 的 `BUILD.gn` 中**未定义**测试目标。测试通过 GN 的通用测试框架执行：

```bash
# 运行 lazycell 测试
ohos_build //third_party/rust/crates/lazycell:lib --test
```

### 测试覆盖

- **单元测试**: 包含在 `src/lib.rs` 和 `tests/lib.rs`
- **测试数量**: 约 20+ 个测试函数
- **覆盖范围**: 核心功能、边界情况、线程安全

---

## 最佳实践

### 集成新 Rust crate 时

1. **使用 `ohos_cargo_crate` 模板**
   ```gn
   ohos_cargo_crate("lib") {
       crate_name = "<crate_name>"
       crate_type = "rlib"
       crate_root = "src/lib.rs"
       # ...
   }
   ```

2. **保持版本号一致**
   - `Cargo.toml`
   - `README.OpenSource`
   - `BUILD.gn` 中的 `cargo_pkg_version`

3. **最小化 OH 特定适配**
   - 避免添加 OH 源文件
   - 避免使用 `#[cfg(ohos)]`
   - 优先通过编译选项解决问题

4. **正确声明依赖**
   ```gn
   deps = [
       "//third_party/rust/crates/dep1:lib",
       "//third_party/rust/crates/dep2:lib",
   ]
   ```

---

## 常见问题

### Q1: 为什么 `cargo_pkg_version` 是 1.3.0，而 `Cargo.toml` 是 1.2.1？

**A**: 这是版本号不一致问题。上游的最新版本为 1.2.1，建议统一版本号。

### Q2: 为什么 `crate_type` 是 `rlib` 而不是 `dylib`？

**A**:
- `rlib` 是 Rust 静态库，仅 Rust 代码可使用
- `dylib` 是动态库，可被 C/C++ 链接
- lazycell 是 Rust 专用库，无需提供 C ABI

### Q3: 如何在 OH 中使用 lazycell？

**A**: 在你的 `BUILD.gn` 中添加依赖：

```gn
ohos_cargo_crate("my_crate") {
    deps = [
        "//third_party/rust/crates/lazycell:lib"
    ]
}
```

然后在 Rust 代码中：

```rust
use lazycell::LazyCell;
```

---

## 参考资源

- **完整评估报告**: [_work/ASSESSMENT.md](_work/ASSESSMENT.md)
- **上游 Cargo.toml**: [../Cargo.toml](../Cargo.toml)
- **上游 API 文档**: https://indiv0.github.io/lazycell/lazycell
