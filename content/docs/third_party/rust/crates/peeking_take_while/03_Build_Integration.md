# OH 构建适配

> OpenHarmony 构建系统与 `peeking_take_while` 的集成方式

---

## 概述

`peeking_take_while` 在 OpenHarmony 中通过**BUILD.gn** 和 **bundle.json** 进行构建系统集成。由于是纯上游引入，无需任何源码修改或 Patch。

---

## BUILD.gn 配置

### 配置文件路径

`/third_party/rust/crates/peeking_take_while/BUILD.gn`

### 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "peeking_take_while"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "0.1.2"
    cargo_pkg_authors = "Nick Fitzgerald <fitzgen@gmail.com>"
    cargo_pkg_name = "peeking_take_while"
    cargo_pkg_description = "Like `Iterator::take_while`, but calls the predicate on a peeked value..."
    module_output_extension = ".rlib"
    part_name = "rust_peeking_take_while"
    subsystem_name = "thirdparty"
}
```

---

## 关键配置说明

### 核心参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `crate_name` | `peeking_take_while` | Rust crate 名称 |
| `crate_type` | `rlib` | Rust 静态库（Rust Library） |
| `crate_root` | `src/lib.rs` | 入口文件路径 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |

### 版本相关

| 参数 | BUILD.gn 值 | Cargo.toml 值 | ⚠️ 不一致 |
|------|------------|--------------|---------|
| `edition` | `2015` | `2018` | ✅ |
| `cargo_pkg_version` | `0.1.2` | `1.0.0` | ✅ |

**影响**:
- 可能导致版本混乱
- 用户阅读文档时可能困惑
- 建议统一为 Cargo.toml 的值（1.0.0, 2018）

### OH 系统参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `module_output_extension` | `.rlib` | 输出文件扩展名 |
| `part_name` | `rust_peeking_take_while` | OH 部件名称 |
| `subsystem_name` | `thirdparty` | 所属子系统 |

---

## bundle.json 配置

### 配置文件路径

`/third_party/rust/crates/peeking_take_while/bundle.json`

### 完整配置

```json
{
  "name": "@ohos/rust_peeking_take_while",
  "description": "A Rust library that provides support for taking elements from an iterator while peeking at the next element",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/peeking_take_while"
  },
  "dirs": {},
  "scripts": {},
  "readmePath": {
    "en": "README.md"
  },
  "component": {
    "name": "rust_peeking_take_while",
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
          "name": "//third_party/rust/crates/peeking_take_while:lib"
        }
      ],
      "test": []
    }
  }
}
```

---

## 关键配置说明

### 组件元数据

| 参数 | 值 | 说明 |
|------|-----|------|
| `name` | `@ohos/rust_peeking_take_while` | Bundle 名称 |
| `description` | `A Rust library...` | 组件描述 |
| `version` | `6.1` | Bundle 版本（非 crate 版本） |
| `license` | `Apache License 2.0` | 开源许可证 |
| `publishAs` | `code-segment` | 发布方式 |

### 构建配置

| 参数 | 值 | 说明 |
|------|-----|------|
| `component.name` | `rust_peeking_take_while` | 组件名称 |
| `component.subsystem` | `thirdparty` | 所属子系统 |
| `adapted_system_type` | `["standard"]` | 适配系统类型 |
| `inner_kits` | `[{"name": "...:lib"}]` | 内部 kit |

---

## 版本不一致问题

### 问题描述

BUILD.gn 中的版本信息与 Cargo.toml 不一致：

| 文件 | 版本 | Edition |
|------|------|---------|
| BUILD.gn | 0.1.2 | 2015 |
| Cargo.toml | 1.0.0 | 2018 |

### 可能原因

1. **手动配置历史遗留**: BUILD.gn 可能是手动配置的，未同步更新
2. **版本策略**: OH 可能故意使用旧版本（需确认）
3. **更新不完整**: 更新 Cargo.toml 后未同步 BUILD.gn

### 影响分析

| 影响 | 严重性 | 说明 |
|------|--------|------|
| **版本混乱** | 🟡 中 | 用户可能困惑 |
| **功能差异** | 🟢 低 | API 兼容，功能一致 |
| **构建风险** | 🟢 低 | 两版本均可编译 |
| **文档不一致** | 🟡 中 | 文档中可能引用错误版本 |

### 建议修复

```gn
# 建议修改为：
edition = "2018"           # 改为 2018
cargo_pkg_version = "1.0.0"  # 改为 1.0.0
```

---

## 与上游构建系统差异

### 上游 Cargo.toml

```toml
[package]
name = "peeking_take_while"
version = "1.0.0"
edition = "2018"
license = "MIT OR Apache-2.0"

[dependencies]
# 无依赖
```

### OH BUILD.gn 差异

| 方面 | 上游 (Cargo) | OH (BUILD.gn) | 差异说明 |
|------|------------|--------------|---------|
| **构建工具** | Cargo | GN | OH 使用 GN 构建系统 |
| **库类型** | rlib (隐式) | rlib (显式) | 一致 |
| **Edition** | 2018 | 2015 | ⚠️ 不一致 |
| **版本** | 1.0.0 | 0.1.2 | ⚠️ 不一致 |
| **依赖管理** | Cargo 自动 | 手动配置 | OH 无依赖 |

---

## OH 特殊配置

### 无特殊配置

- ✅ **无 Patch**: 完全使用上游代码
- ✅ **无条件编译**: 无 `#[cfg(ohos)]` 等宏
- ✅ **无特殊依赖**: 零依赖
- ✅ **无特殊源文件**: 仅 `src/lib.rs`

### 标准配置

```gn
# 标准的 OH Rust crate 配置
ohos_cargo_crate("lib") {
    # 标准参数
    crate_name = "peeking_take_while"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    sources = ["src/lib.rs"]

    # 元数据
    part_name = "rust_peeking_take_while"
    subsystem_name = "thirdparty"
}
```

---

## 构建流程

### 构建步骤

1. **GN 解析**: OH 构建系统解析 BUILD.gn
2. **Cargo 调用**: 调用 `ohos_cargo_crate` 模板
3. **Rustc 编译**: 使用 Rust 编译器编译 `src/lib.rs`
4. **输出 rlib**: 生成 `libpeeking_take_while.rlib`

### 输出产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libpeeking_take_while.rlib` | `out/{target}/libpeeking_take_while-*.rlib` | Rust 静态库 |

---

## 依赖关系

### 外部依赖

```toml
# Cargo.toml
[dependencies]
# 无依赖
```

**结论**: 零外部依赖

### OH 内部依赖

```json
{
  "deps": {
    "components": []
  }
}
```

**结论**: 无 OH 组件依赖

---

## 典型使用方式

### 在 BUILD.gn 中依赖

```gn
ohos_rust_executable("my_app") {
  deps = [
    "//third_party/rust/crates/peeking_take_while:lib",
  ]

  sources = ["src/main.rs"]
}
```

### 在 Cargo.toml 中依赖

```toml
[dependencies]
peeking_take_while = "0.1.2"  # 或 1.0.0
```

### 在 Rust 代码中使用

```rust
use peeking_take_while::PeekableExt;

fn main() {
    let mut iter = (0..100).peekable();
    let sum: u32 = iter.by_ref()
        .peeking_take_while(|&x| x < 10)
        .sum();
    println!("Sum of numbers < 10: {}", sum);
}
```

---

## 构建优化

### 当前优化

| 优化项 | 状态 | 说明 |
|--------|------|------|
| `#[inline]` | ✅ | 源码中使用内联 |
| `LTO` | ⏸ | 未启用（可能由全局配置） |
| `opt-level` | ⏸ | 使用默认级别 |

### 建议优化

由于 crate 极小且无外部依赖，默认配置已足够。无需额外优化。

---

## 测试集成

### 测试配置

```gn
"build": {
  "test": []
}
```

**当前状态**: 无 OH 测试集成

**建议**: 如需集成测试，可添加：

```gn
ohos_rust_test("peeking_take_while_test") {
  deps = [":lib"]
  sources = ["src/lib.rs"]
}
```

---

## 版本升级指南

### 升级到 1.0.0

#### 步骤 1: 更新 BUILD.gn

```gn
# 修改前
edition = "2015"
cargo_pkg_version = "0.1.2"

# 修改后
edition = "2018"
cargo_pkg_version = "1.0.0"
```

#### 步骤 2: 验证编译

```bash
./build.sh --product-name <your-product>
```

#### 步骤 3: 运行测试

```bash
./test.py third_party/rust/crates/peeking_take_while
```

#### 步骤 4: 更新文档

在相关文档中更新版本信息。

---

## 故障排查

### 常见问题

#### 问题 1: 版本冲突

**错误信息**:
```
error: package `peeking_take_while v0.1.2` cannot be built because it requires rustc 1.31 or newer
```

**原因**: 使用的 Rust 版本过旧

**解决**: 升级到 1.0.0 版本或更新 Rust 工具链

#### 问题 2: 找不到 crate

**错误信息**:
```
error: cannot find crate `peeking_take_while`
```

**原因**: BUILD.gn 中未正确声明依赖

**解决**: 添加依赖：
```gn
deps = [
    "//third_party/rust/crates/peeking_take_while:lib",
]
```

#### 问题 3: Edition 不匹配

**错误信息**:
```
error: this crate is being compiled with the 2015 edition
```

**原因**: BUILD.gn 中 edition 设置为 2015

**解决**: 升级到 2018 edition

---

## 总结

`peeking_take_while` 在 OpenHarmony 中的构建集成非常简单，仅需要标准的 BUILD.gn 和 bundle.json 配置。由于是纯上游引入且无特殊适配，维护成本极低。唯一需要关注的是 BUILD.gn 和 Cargo.toml 的版本不一致问题，建议统一为 1.0.0 版本。

---

**相关文档**:
- [01_Overview.md](./01_Overview.md) - 原始库简介
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用情况
- [06_Security.md](./06_Security.md) - 安全性分析

**配置文件**:
- [BUILD.gn](../BUILD.gn) - OH 构建配置
- [bundle.json](../bundle.json) - Bundle 元数据
- [Cargo.toml](../Cargo.toml) - Rust 包配置

---

**文档版本**: 1.0.0
**最后更新**: 2026-02-08
