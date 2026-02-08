# BUILD.gn 构建集成

## BUILD.gn 完整内容

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
    crate_name = "unicode_ident"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "1.0.14"
    cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
    cargo_pkg_name = "unicode-ident"
    cargo_pkg_description = "Determine whether characters have the XID_Start or XID_Continue properties according to Unicode Standard Annex #31"
    module_output_extension = ".rlib"
    part_name = "rust_unicode-ident"
    subsystem_name = "thirdparty"
}
```

---

## 配置详解

### 基础 crate 配置

| 属性 | 值 | 说明 |
|------|-----|------|
| `crate_name` | `"unicode_ident"` | Rust crate 名称（下划线格式） |
| `crate_type` | `"rlib"` | Rust 库类型（rlib = Rust 静态库） |
| `crate_root` | `"src/lib.rs"` | crate 入口文件 |

**说明**: 
- `crate_name` 使用下划线（Rust 规范）
- `cargo_pkg_name` 保持原始连字符格式

### 源文件配置

```gn
sources = ["src/lib.rs"]
```

**注意**: 虽然 `src/tables.rs` 也是源文件，但 GN 构建系统通过 Rust 的模块系统（`mod tables;`）自动发现，无需显式列出。

**对比其他 crate**:

| Crate | sources 配置 | 说明 |
|-------|-------------|------|
| unicode-ident | `["src/lib.rs"]` | 单文件入口 |
| syn | `["src/lib.rs"]` | 单文件入口 |
| serde | `["src/lib.rs"]` | 单文件入口 |

### Rust 版本配置

```gn
edition = "2018"
```

- **当前使用**: Rust 2018 Edition
- **上游要求**: `rust-version = "1.31"`（Cargo.toml 中）
- **兼容性**: OpenHarmony 构建系统完全支持

### 包元数据

```gn
cargo_pkg_version = "1.0.14"
cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
cargo_pkg_name = "unicode-ident"
cargo_pkg_description = "..."
```

这些元数据：
- 用于生成 Cargo 兼容的构建环境
- 不影响实际编译，用于追踪和调试
- 应与上游 Cargo.toml 保持一致

### OpenHarmony 组件标识

```gn
part_name = "rust_unicode-ident"
subsystem_name = "thirdparty"
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `part_name` | `"rust_unicode-ident"` | bundle.json 中定义的组件名 |
| `subsystem_name` | `"thirdparty"` | 所属子系统 |

---

## 缺失的配置项（及原因）

### 1. 无 `deps`

```gn
# 未配置 - 因为无依赖
deps = []
```

**原因**: unicode-ident 是**叶子 crate**，不依赖任何其他 crate。

**对比**:
- `proc-macro2` 依赖 `unicode-ident`
- `syn` 依赖 `unicode-ident` 和 `proc-macro2`

### 2. 无 `features`

```gn
# 未配置 - 因为无特性标志
features = []
```

**原因**: 该库无可选特性。上游 Cargo.toml:
```toml
[package]
# 无 [features] 段
```

### 3. 无 `build_root` / `build_sources`

```gn
# 未配置 - 因为无 build.rs
# build_root = "build.rs"
# build_sources = ["build.rs"]
```

**原因**: 无构建脚本，数据表已预生成在 `src/tables.rs` 中。

**对比需要 build.rs 的 crate**:
- `syn` - 检测编译器版本
- `proc-macro2` - 检测平台特性
- `bindgen` - 调用 libclang

### 4. 无 `build_script_outputs`

**原因**: 无 build.rs，因此无输出文件。

### 5. 无 `rustenv`

**原因**: 不需要环境变量传递给构建脚本（无 build.rs）。

### 6. 无平台检查

```gn
# 某些 crate 有：
# if (host_os != "linux" || host_cpu != "arm64") {
#     ohos_cargo_crate("lib") { ... }
# }
```

**原因**: unicode-ident 是纯 Rust 代码，在所有平台行为一致。

---

## 与上游构建对比

### 上游构建（Cargo）

```toml
# Cargo.toml
[package]
name = "unicode-ident"
version = "1.0.14"
edition = "2018"
rust-version = "1.31"

[dependencies]
# 无依赖

[dev-dependencies]
criterion = "0.5"
# ... 仅测试依赖
```

**构建命令**:
```bash
cargo build          # 发布模式
cargo test           # 运行测试
cargo bench          # 运行基准测试
```

### OpenHarmony 构建（GN）

**构建命令**:
```bash
# 通过 OpenHarmony 构建系统
hb build -T rust_unicode-ident

# 或全系统构建
hb build
```

**关键差异**:

| 方面 | Cargo | OpenHarmony GN |
|------|-------|----------------|
| 构建入口 | Cargo.toml | BUILD.gn |
| 依赖管理 | crates.io | 本地 //third_party/rust/crates |
| 测试 | `cargo test` | 未配置（bundle.json 中 test: []） |
| 特性标志 | Cargo features | GN features |
| 构建脚本 | build.rs 自动检测 | 需显式配置 build_root |

---

## BUILD.gn 标准模板对比

### unicode-ident（极简模板）

```gn
ohos_cargo_crate("lib") {
    # 必需：基础配置
    crate_name = "unicode_ident"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    
    # 必需：源文件
    sources = ["src/lib.rs"]
    
    # 必需：版本信息
    edition = "2018"
    cargo_pkg_version = "1.0.14"
    
    # 必需：OH 标识
    part_name = "rust_unicode-ident"
    subsystem_name = "thirdparty"
    
    module_output_extension = ".rlib"
    
    # 可选：无
}
```

### 中等复杂度模板（如 quote）

```gn
ohos_cargo_crate("lib") {
    crate_name = "quote"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    
    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "1.0.35"
    
    # 依赖
    deps = [
        "//third_party/rust/crates/proc-macro2:lib",
    ]
    
    part_name = "rust_quote"
    subsystem_name = "thirdparty"
    module_output_extension = ".rlib"
}
```

### 复杂模板（如 syn）

```gn
ohos_cargo_crate("lib") {
    crate_name = "syn"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    
    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "2.0.48"
    
    # 构建脚本
    build_root = "build.rs"
    build_sources = ["build.rs"]
    
    # 依赖
    deps = [
        "//third_party/rust/crates/proc-macro2:lib",
        "//third_party/rust/crates/unicode-ident:lib",  # ← 引用本库
        "//third_party/rust/crates/quote:lib",
    ]
    
    # 特性
    features = [
        "clone-impls",
        "derive",
        "parsing",
        "printing",
        "proc-macro",
    ]
    
    part_name = "rust_syn"
    subsystem_name = "thirdparty"
    module_output_extension = ".rlib"
}
```

---

## 依赖关系声明

### 如何被其他 crate 依赖

```gn
# third_party/rust/crates/proc-macro2/BUILD.gn
deps = [
    "//third_party/rust/crates/unicode-ident:lib",
]

# third_party/rust/crates/syn/BUILD.gn
deps = [
    "//third_party/rust/crates/unicode-ident:lib",
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
]
```

### 依赖路径格式

```gn
"//third_party/rust/crates/<crate-name>:lib"
```

| 组件 | 路径 |
|------|------|
| crate-name | Cargo.toml 中的 package name（连字符格式） |
| `:lib` | GN target 名称，对应 `ohos_cargo_crate("lib")` |

---

## 构建输出

### 生成的文件

```
out/<board>/gen/third_party/rust/crates/unicode-ident/
├── libunicode_ident.rlib          # Rust 静态库
└── （其他元数据文件）
```

### 链接方式

- **输出格式**: `.rlib`（Rust 原生静态库格式）
- **链接方式**: 被 Rust 编译器自动链接到依赖 crate
- **最终产物**: 嵌入到使用它的二进制文件中

---

## 维护检查清单

当升级 unicode-ident 版本时，检查：

- [ ] `cargo_pkg_version` 已更新
- [ ] `edition` 未变更（或已同步更新）
- [ ] `sources` 路径未变更
- [ ] 新增依赖（概率极低）
- [ ] 新增特性（概率极低）
- [ ] 新增 build.rs（概率极低）

---

## 总结

**unicode-ident 的 BUILD.gn 是 OpenHarmony Rust crate 的最简模板**:

- ✅ 无依赖（deps）
- ✅ 无特性（features）
- ✅ 无构建脚本（build.rs）
- ✅ 无平台条件
- ✅ 标准配置项

这种极简配置是**设计使然**而非简化：unicode-ident 作为纯数据表库，本身就不需要复杂的构建配置。

---

*下一章: [OH 中的使用](04_Usage_in_OH.md)*
