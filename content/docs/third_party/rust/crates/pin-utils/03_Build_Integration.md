# 03 - OH 构建适配

## 3.1 BUILD.gn 分析

### 3.1.1 完整配置

**文件**: `third_party/rust/crates/pin-utils/BUILD.gn`

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
    crate_name = "pin_utils"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.1.0"
    cargo_pkg_authors = "Josef Brandl <mail@josefbrandl.de>"
    cargo_pkg_name = "pin-utils"
    cargo_pkg_description = "Utilities for pinning"
    module_output_extension = ".rlib"
    part_name = "rust_pin_utils"
    subsystem_name = "thirdparty"
}
```

### 3.1.2 配置项解析

| 配置项 | 值 | 说明 |
|-------|-----|-----|
| `crate_name` | `pin_utils` | Rust crate 名称 (GN 规则要求下划线) |
| `crate_type` | `rlib` | 生成静态库 (.rlib) |
| `crate_root` | `src/lib.rs` | crate 入口文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |
| `edition` | `2018` | Rust 版本 (2018 Edition) |
| `cargo_pkg_version` | `0.1.0` | Cargo 包版本 |
| `cargo_pkg_authors` | `Josef Brandl ...` | 包作者信息 |
| `cargo_pkg_name` | `pin-utils` | Cargo 包名称 |
| `cargo_pkg_description` | `Utilities for pinning` | 包描述 |
| `module_output_extension` | `.rlib` | 输出文件扩展名 |
| `part_name` | `rust_pin_utils` | OH 组件名称 |
| `subsystem_name` | `thirdparty` | 所属子系统 |

### 3.1.3 关键观察

#### 1. 无特殊编译选项

对比典型的复杂 crate BUILD.gn，pin-utils **无任何特殊配置**：

```gn
# 典型的复杂 crate 可能有：
cargo_pkg_features = ["feature1", "feature2"]  # pin-utils: 无
cargo_pkg_deps = ["dep1", "dep2"]             # pin-utils: 无
cargo_pkg_build = "build.rs"                   # pin-utils: 无
cargo_pkg_proc_macro = true                    # pin-utils: 无

# 无 defines
# 无 configs
# 无 cflags
# 无 ldflags
```

#### 2. 单文件源

```gn
sources = ["src/lib.rs"]  # 仅需编译入口文件
```

虽然实际代码分布在：
- `src/lib.rs`
- `src/stack_pin.rs`
- `src/projection.rs`

但由于 Rust 模块系统，`lib.rs` 会自动包含其他模块。

---

## 3.2 与上游构建对比

### 3.2.1 Cargo.toml 分析

**上游 Cargo.toml**:

```toml
[package]
name = "pin-utils"
edition = "2018"
version = "0.1.0"
authors = ["Josef Brandl <mail@josefbrandl.de>"]
license = "MIT OR Apache-2.0"
readme = "README.md"
repository = "https://github.com/rust-lang-nursery/pin-utils"
documentation = "https://docs.rs/pin-utils"
description = """
Utilities for pinning
"""

# 注意: 无 [dependencies] 段
# 注意: 无 [features] 段
# 注意: 无 build.rs
```

### 3.2.2 差异分析

| 方面 | Cargo 构建 | GN 构建 | 差异 |
|-----|-----------|---------|-----|
| 依赖 | 自动解析 | 显式声明 | 两者都无依赖 |
| 特性 | Cargo features | GN features | 两者都无特性 |
| 构建脚本 | build.rs | 无需 | 两者都无 |
| 输出 | .rlib | .rlib | 一致 |

**结论**: BUILD.gn 完全映射了 Cargo.toml 的配置，无额外适配。

---

## 3.3 构建系统模板

### 3.3.1 `ohos_cargo_crate` 模板

BUILD.gn 使用 OH 提供的标准模板：

```gn
import("//build/templates/rust/ohos_cargo_crate.gni")

ohos_cargo_crate("lib") {
    # ... 配置
}
```

该模板处理：
- Rust 编译器调用
- 依赖解析
- 输出文件生成
- 与 GN 构建系统集成

### 3.3.2 模板参数说明

pin-utils 使用的都是**标准参数**，无特殊设置：

| 参数类别 | 使用情况 |
|---------|---------|
| 必需参数 | 全部使用标准值 |
| 可选参数 | 全部使用默认值 |
| 特殊参数 | 未使用 |

---

## 3.4 构建输出

### 3.4.1 输出文件

构建完成后生成：

```
out/标准系统类型/对象/third_party/rust/crates/pin-utils/
├── lib.pin_utils.rlib      # 静态库
├── lib.pin_utils.rlib.d    # 依赖文件
└── ...
```

### 3.4.2 依赖关系

由于 pin-utils 是**叶节点库**（无依赖），构建顺序不受其他库影响。

```
构建顺序:
1. pin-utils (叶节点，可最先构建)
2. nix (依赖 pin-utils)
3. 其他依赖 nix 的库
```

---

## 3.5 特殊适配说明

### 3.5.1 无特殊适配

pin-utils 的 BUILD.gn 是**最简单的标准配置**，无任何特殊处理：

- ✅ 无平台条件编译
- ✅ 无特性开关
- ✅ 无额外源文件
- ✅ 无自定义编译选项
- ✅ 无外部依赖

### 3.5.2 为什么不需特殊适配

1. **纯 Rust 库**: 无 C/C++ 依赖，无需交叉编译适配
2. **标准库 only**: 仅使用 `core::pin`，无平台差异
3. **无特性**: 无条件编译特性，无需 feature 映射
4. **代码简单**: 145 行代码，功能边界清晰

---

## 3.6 构建配置参考

### 3.6.1 最小化 Rust Crate BUILD.gn 模板

基于 pin-utils 的 BUILD.gn，可提炼出**最小化模板**：

```gn
import("//build/templates/rust/ohos_cargo_crate.gni")

ohos_cargo_crate("lib") {
    crate_name = "crate_name"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "x.y.z"
    cargo_pkg_name = "crate-name"
    part_name = "rust_crate_name"
    subsystem_name = "thirdparty"
}
```

### 3.6.2 与其他 Crate 对比

| Crate | 特性数 | 依赖数 | BUILD.gn 复杂度 |
|-------|-------|-------|----------------|
| pin-utils | 0 | 0 | 极简 |
| nix | 多 | 多 | 中等 |
| libc | 0 | 0 | 极简 |
| tokio | 多 | 多 | 复杂 |

pin-utils 属于**极简**类别，是理解 OH Rust 构建系统的理想示例。

---

## 3.7 维护注意事项

### 3.7.1 升级时无需关注

由于 BUILD.gn 是标准配置，升级时：

- ✅ 无需修改 BUILD.gn
- ✅ 无需更新编译选项
- ✅ 只需验证能否正常编译

### 3.7.2 潜在变更场景

以下情况可能需要更新 BUILD.gn：

1. **Rust Edition 升级**: 如从 2018 升级到 2021
2. **添加特性**: 如上游添加新 feature
3. **添加依赖**: 如上游新增依赖项

但鉴于上游已归档，这些变更极不可能发生。

---

*本文档分析基于 BUILD.gn 当前版本*
