# 03 - OH 构建适配

## 3.1 BUILD.gn 结构说明

### 完整 BUILD.gn 内容

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

import("//build/templates/rust/ohos_cargo_crate.gni")

ohos_cargo_crate("lib") {
    crate_name = "static_assertions"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "1.1.0"
    cargo_pkg_authors = "Nikolai Vazquez"
    cargo_pkg_name = "static_assertions"
    cargo_pkg_description = "Compile-time assertions to ensure that invariants are met."
    module_output_extension = ".rlib"
    part_name = "rust_static_assertions_rs"
    subsystem_name = "thirdparty"
}
```

---

## 3.2 关键配置项解析

### 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | static_assertions | crate 名称，用于引用 |
| `crate_type` | rlib | 输出 Rust 静态库 |
| `crate_root` | src/lib.rs | crate 根文件 |
| `sources` | ["src/lib.rs"] | 源文件列表 |

### 版本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `edition` | 2015 | Rust Edition 2015 |
| `cargo_pkg_version` | 1.1.0 | 包版本号 |
| `cargo_pkg_authors` | Nikolai Vazquez | 作者信息 |
| `cargo_pkg_name` | static_assertions | Cargo 包名 |

### OH 构建配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `module_output_extension` | .rlib | 输出文件扩展名 |
| `part_name` | rust_static_assertions_rs | OH 组件名 |
| `subsystem_name` | thirdparty | 所属子系统 |

---

## 3.3 与上游 Cargo.toml 的映射

### 字段对比

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `[package] name` | `crate_name` | crate 名称 |
| `[package] version` | `cargo_pkg_version` | 版本号 |
| `[package] authors` | `cargo_pkg_authors` | 作者 |
| `[package] edition` | `edition` | Rust Edition |
| `[lib] crate-type` | `crate_type` | 输出类型 |

### Cargo 功能特性映射

上游 Cargo.toml 中的功能特性：

```toml
[features]
nightly = []
```

在 OH BUILD.gn 中：
- **未启用 nightly**: 不使用 nightly Rust 特性
- **标准功能**: 使用稳定版 Rust 功能

---

## 3.4 特殊编译选项

### defines

该库 BUILD.gn 中**无自定义 defines**。

### configs

该库 BUILD.gn 中**无自定义 configs**。

### deps

该库 BUILD.gn 中**无额外依赖**（完全无依赖）。

### flags

该库 BUILD.gn 中**无特殊编译 flags**。

---

## 3.5 与上游构建系统的差异

### 构建工具对比

| 方面 | 上游 (Cargo) | OpenHarmony (GN) |
|------|-------------|------------------|
| 构建文件 | Cargo.toml | BUILD.gn |
| 构建命令 | cargo build | gn + ninja |
| 依赖管理 | Cargo.toml deps | GN deps |
| 输出目录 | target/ | out/ |

### 构建行为差异

| 行为 | Cargo | GN | 影响 |
|------|-------|-----|------|
| 增量编译 | 支持 | 支持 | 无差异 |
| 交叉编译 | 支持 | 支持 | GN 配置更灵活 |
| 测试运行 | cargo test | 需单独配置 | OH 中可能需单独配置测试 |

### 特殊处理

该库在 OH 构建中**无特殊处理**：

- ✅ 无需禁用特性
- ✅ 无需添加 OH 特定源文件
- ✅ 无需修改编译选项
- ✅ 标准模板直接可用

---

## 3.6 源码结构

```
src/
├── lib.rs                 # crate 根，导出所有宏
├── assert_cfg.rs          # assert_cfg! 宏实现
├── assert_eq_align.rs     # assert_eq_align! 宏实现
├── assert_eq_size.rs      # assert_eq_size! 宏实现
├── assert_fields.rs       # assert_fields! 宏实现
├── assert_impl.rs         # trait 断言宏实现
├── assert_obj_safe.rs     # assert_obj_safe! 宏实现
├── assert_trait.rs        # trait 关系断言宏实现
├── assert_type.rs         # 类型相等/不等断言宏实现
└── const_assert.rs        # 常量断言宏实现
```

### BUILD.gn sources 配置

```gn
sources = ["src/lib.rs"]
```

**注意**: 虽然只有 `src/lib.rs` 在 sources 中，但该文件通过 `mod` 语句包含其他模块：

```rust
// src/lib.rs
mod assert_cfg;
mod assert_eq_align;
mod assert_eq_size;
// ... 其他模块
```

这是 Rust 的标准做法，GN 构建系统会自动处理这些模块依赖。

---

## 3.7 构建输出

### 输出文件

| 输出 | 文件名 | 说明 |
|------|--------|------|
| 静态库 | libstatic_assertions.rlib | Rust 静态库 |

### 使用方式

在其他 GN 目标中依赖：

```gn
rust_library("my_crate") {
    deps = [
        "//third_party/rust/crates/static-assertions-rs:lib",
    ]
}
```

在 Rust 代码中使用：

```rust
use static_assertions::assert_impl_all;

assert_impl_all!(MyType: Send, Sync);
```

---

## 3.8 维护指南

### 升级步骤

1. **更新源码**: 替换为上游新版本
2. **更新版本号**: 修改 BUILD.gn 中的 `cargo_pkg_version`
3. **检查 edition**: 如有变更，更新 `edition`
4. **测试构建**: 运行 GN 构建验证

### 常见问题

#### Q: 为什么 sources 只有 lib.rs？
A: Rust 通过模块系统管理依赖，lib.rs 中的 `mod` 声明会自动包含其他文件。

#### Q: 如何启用 nightly 特性？
A: 不推荐在 OH 中使用 nightly 特性。如需启用，需在 BUILD.gn 中添加相应配置。

#### Q: 为什么无 deps？
A: 该库是纯宏库，零依赖设计。
