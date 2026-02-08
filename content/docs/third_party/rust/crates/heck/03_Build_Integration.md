# heck OH 构建适配

## 1. BUILD.gn 完整分析

### 1.1 文件位置
```
third_party/rust/crates/heck/BUILD.gn
```

### 1.2 完整内容

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
    crate_name = "heck"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.4.1"
    cargo_pkg_authors = "Without Boats <woboats@gmail.com>"
    cargo_pkg_name = "heck"
    cargo_pkg_description = "heck is a case conversion library."
    module_output_extension = ".rlib"
    part_name = "rust_heck"
    subsystem_name = "thirdparty"
}
```

## 2. 构建配置详解

### 2.1 基本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `"heck"` | Rust crate 名称 |
| `crate_type` | `"rlib"` | 输出类型：Rust 静态库 |
| `crate_root` | `"src/lib.rs"` | crate 入口文件 |

### 2.2 源码配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | `["src/lib.rs"]` | 编译源文件列表 |

**注意**：heck 使用单文件 lib.rs 作为入口，其他模块通过 `mod xxx;` 在 lib.rs 中声明。

### 2.3 Rust 版本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `edition` | `"2018"` | Rust 2018 Edition |

与上游 `Cargo.toml` 一致：
```toml
[package]
edition = "2018"
```

### 2.4 元数据配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cargo_pkg_version` | `"0.4.1"` | 包版本 |
| `cargo_pkg_authors` | `"Without Boats <...>"` | 作者信息 |
| `cargo_pkg_name` | `"heck"` | 包名称 |
| `cargo_pkg_description` | `"heck is a case conversion library."` | 包描述 |

### 2.5 OH 系统配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `module_output_extension` | `".rlib"` | 输出文件扩展名 |
| `part_name` | `"rust_heck"` | OH 组件名（与 bundle.json 一致） |
| `subsystem_name` | `"thirdparty"` | 所属子系统 |

## 3. 关键编译选项分析

### 3.1 缺失的配置项（与复杂 crate 对比）

heck BUILD.gn 中**未配置**以下项目：

| 未配置项 | 说明 | 为何无需配置 |
|----------|------|--------------|
| `deps` | 依赖其他 crate | heck 零依赖 |
| `build_deps` | 构建依赖 | 无 build.rs |
| `configs` | 编译配置 | 使用默认配置 |
| `defines` | 预定义宏 | 无需特殊宏 |
| `cflags` / `ldflags` | 编译/链接标志 | 纯 Rust，无特殊需求 |

### 3.2 特性配置

heck 的 Cargo.toml 支持可选特性：

```toml
[features]
default = []
unicode = ["unicode-segmentation"]

[dependencies]
unicode-segmentation = { version = "1.2.0", optional = true }
```

**OH 配置**：未启用任何特性（使用 `default = []`）

**BUILD.gn 中特性处理**：

OH 的 `ohos_cargo_crate` 模板处理特性的方式：
- 未在 `features` 参数中列出的特性 = 不启用
- heck 的 BUILD.gn 未指定 `features`，因此使用默认空特性集

### 3.3 与上游构建对比

| 维度 | 上游 Cargo | OH BUILD.gn | 差异 |
|------|-----------|-------------|------|
| 构建工具 | Cargo | GN + rustc | 不同构建系统 |
| 依赖解析 | Cargo.lock | GN deps | OH 显式声明 |
| 特性选择 | 可自定义 | 固定 default | OH 未启用 unicode |
| 输出格式 | .rlib / .a / .so | .rlib | OH 仅构建 rlib |

## 4. 源码结构

### 4.1 文件组织

```
src/
├── lib.rs           # 主模块，声明子模块，实现核心转换逻辑
├── kebab.rs         # KebabCase 实现
├── lower_camel.rs   # LowerCamelCase 实现
├── shouty_kebab.rs  # ShoutyKebabCase 实现
├── shouty_snake.rs  # ShoutySnakeCase 实现
├── snake.rs         # SnakeCase 实现
├── title.rs         # TitleCase 实现
├── train.rs         # TrainCase 实现
└── upper_camel.rs   # UpperCamelCase 实现
```

### 4.2 lib.rs 结构

```rust
// 1. 模块声明
mod kebab;
mod lower_camel;
// ... 其他模块

// 2. Trait 重导出
pub use kebab::{AsKebabCase, ToKebabCase};
pub use snake::{AsSnakeCase, AsSnakeCase as AsSnekCase, ...};
// ...

// 3. 核心转换函数
fn transform<F, G>(...) -> fmt::Result { ... }
fn lowercase(...) -> fmt::Result { ... }
fn uppercase(...) -> fmt::Result { ... }
fn capitalize(...) -> fmt::Result { ... }
```

## 5. 构建流程

### 5.1 正常构建流程

```bash
# 在 OH 构建系统中
# 1. GN 解析 BUILD.gn
gn gen out

# 2. Ninja 编译
ninja -C out third_party/rust/crates/heck:lib

# 3. 输出
# out/third_party/rust/crates/heck/libheck.rlib
```

### 5.2 依赖构建流程

当构建 `clap_derive` 时：

```
构建 clap_derive
    ↓ 发现依赖 "//third_party/rust/crates/heck:lib"
    ↓ 检查已构建？
    ↓ 否 → 先构建 heck
    ↓
构建 heck:lib
    ↓
输出 libheck.rlib
    ↓
链接到 clap_derive
```

## 6. 与上游构建系统的差异

### 6.1 Cargo 构建流程

```bash
# 上游使用 Cargo
cargo build

# 输出
# target/debug/libheck.rlib
```

### 6.2 关键差异

| 方面 | Cargo | OH GN |
|------|-------|-------|
| 依赖管理 | 自动解析 Cargo.toml | 显式声明在 BUILD.gn |
| 增量构建 | Cargo 内置 | Ninja 处理 |
| 交叉编译 | 配置 target | GN 配置处理 |
| 输出目录 | target/debug or release | out/... |

### 6.3 为何需要 BUILD.gn？

1. **统一构建系统**：OH 所有组件使用 GN + Ninja
2. **增量构建**：Ninja 的高效增量构建
3. **依赖可视化**：GN 的依赖图清晰
4. **跨语言支持**：GN 可混合 Rust/C/C++ 构建

## 7. 维护指南

### 7.1 升级版本流程

假设上游发布 v0.4.2：

1. **更新源码**
   ```bash
   # 替换 src/ 下所有文件为上游 v0.4.2
   ```

2. **更新 BUILD.gn**
   ```gn
   cargo_pkg_version = "0.4.2"
   ```

3. **验证构建**
   ```bash
   ninja -C out third_party/rust/crates/heck:lib
   ```

4. **验证依赖者**
   ```bash
   ninja -C out third_party/rust/crates/clap/clap_derive:lib
   ```

### 7.2 常见问题排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 编译错误 | 上游 API 变更 | 检查 clap_derive 是否适配新 API |
| 特性缺失 | 需要 unicode 特性 | 在 BUILD.gn 中添加 features 配置 |

### 7.3 启用 Unicode 特性（如需要）

若未来需要启用 unicode 支持：

```gn
ohos_cargo_crate("lib") {
    # ... 现有配置 ...
    
    features = [
        "unicode",
    ]
    deps = [
        "//third_party/rust/crates/unicode-segmentation:lib",
    ]
}
```

**注意**：需先确保 `unicode-segmentation` crate 已在 OH 中集成。

## 8. 总结

| 项目 | 状态 |
|------|------|
| 构建复杂度 | **简单**（标准 ohos_cargo_crate 模板） |
| 特殊配置 | 无 |
| 依赖数量 | 0 |
| 维护工作量 | **极低** |

heck 的构建配置是 OH Rust crate 的标准示例——简洁、无特殊配置、直接使用 `ohos_cargo_crate` 模板。
