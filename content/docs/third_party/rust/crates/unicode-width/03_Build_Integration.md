# 03 - OH 构建适配

## 概述

unicode-width 使用标准的 OpenHarmony Rust 构建模板 `ohos_cargo_crate`，配置简洁，无需特殊适配。

## BUILD.gn 完整配置

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
    crate_name = "unicode_width"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "0.1.14"
    cargo_pkg_authors = "kwantam <kwantam@gmail.com>, Manish Goregaokar <manishsmail@gmail.com>"
    cargo_pkg_name = "unicode-width"
    module_output_extension = ".rlib"
    part_name = "rust_unicode_width"
    subsystem_name = "thirdparty"
}
```

## 配置解析

### 基本元数据

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | `unicode_width` | Rust crate 名称（下划线格式） |
| `crate_type` | `rlib` | 静态库格式，用于链接 |
| `crate_root` | `src/lib.rs` | crate 入口文件 |

### 源文件配置

```gn
sources = ["src/lib.rs"]
```

**特点**：
- 单一源文件（src/lib.rs）
- 实际实现分散在 `src/lib.rs` 和 `src/tables.rs`
- `tables.rs` 通过 `mod tables;` 引入

**注意**：虽然 sources 只列出 `lib.rs`，但 Rust 编译器会自动处理模块引用。

### Rust 版本

```gn
edition = "2021"
```

- 使用 Rust 2021 Edition
- 与上游 Cargo.toml 保持一致
- 现代 Rust 特性支持

### Cargo 元数据

```gn
cargo_pkg_version = "0.1.14"
cargo_pkg_authors = "kwantam <kwantam@gmail.com>, Manish Goregaokar <manishsmail@gmail.com>"
cargo_pkg_name = "unicode-width"
```

这些字段用于：
- 版本追踪
- 生成 crate 元数据
- 与 Cargo 生态兼容

### OH 组件标识

```gn
part_name = "rust_unicode_width"
subsystem_name = "thirdparty"
```

| 字段 | 值 | 来源 |
|------|-----|------|
| `part_name` | `rust_unicode_width` | bundle.json 中的 `component.name` |
| `subsystem_name` | `thirdparty` | bundle.json 中的 `component.subsystem` |

## 配置特点

### 无特殊配置

与其他复杂库相比，unicode-width 的 BUILD.gn **非常简洁**：

| 配置项 | 状态 | 说明 |
|--------|------|------|
| `defines` | ❌ 无 | 无需宏定义覆盖 |
| `configs` | ❌ 无 | 无需特殊编译配置 |
| `deps` | ❌ 无 | 无外部依赖 |
| `cflags` | ❌ 无 | 无需特殊编译选项 |
| `ldflags` | ❌ 无 | 无需特殊链接选项 |

### 标准模板使用

BUILD.gn 完全使用 `ohos_cargo_crate` 标准模板，无自定义逻辑。

## 与 Cargo.toml 的映射

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `[package] name` | `cargo_pkg_name` | 包名称 |
| `[package] version` | `cargo_pkg_version` | 版本号 |
| `[package] authors` | `cargo_pkg_authors` | 作者信息 |
| `[package] edition` | `edition` | Rust 版本 |
| `[lib] crate-type` | `crate_type` | 输出类型 |

## Features 处理

### Cargo.toml 中的 features

```toml
[features]
cjk = []
default = ["cjk"]
rustc-dep-of-std = ['std', 'core', 'compiler_builtins']
```

### OH 构建中的处理

当前 BUILD.gn **未显式配置 features**，这意味着：
- 使用 Cargo.toml 中的默认 features
- `cjk` feature 被启用（默认）

### 如需禁用 CJK

如果需要构建无 CJK 支持的版本（减小体积），需要修改 BUILD.gn：

```gn
ohos_cargo_crate("lib") {
    # ... 其他配置
    features = []  # 禁用默认 features
}
```

## 构建输出

### 输出文件

```
out/{target}/obj/third_party/rust/crates/unicode-width/lib.rlib
```

### 使用方式

其他 GN 目标通过以下方式依赖：

```gn
deps = [
    "//third_party/rust/crates/unicode-width:lib",
]
```

## 与上游构建系统的差异

| 方面 | 上游 (Cargo) | OH (GN) |
|------|-------------|---------|
| 构建工具 | cargo | gn + ninja |
| 配置文件 | Cargo.toml | BUILD.gn |
| 依赖管理 | Cargo.toml [dependencies] | GN deps |
| 输出格式 | .rlib | .rlib |
| 测试运行 | cargo test | 需单独配置 |

### 差异说明

1. **构建工具不同**：OH 使用 GN/Ninja 而非 Cargo
2. **依赖声明**：OH 在 BUILD.gn 中声明 GN 风格的依赖
3. **功能等价**：最终输出相同的 .rlib 格式

## bundle.json 配置

```json
{
  "name": "@ohos/rust_unicode_width",
  "description": "A Rust library that provides support for working with Unicode character widths",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/unicode-width"
  },
  "component": {
    "name": "rust_unicode_width",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "build": {
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/unicode-width:lib"
        }
      ]
    }
  }
}
```

### 关键字段

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | `@ohos/rust_unicode_width` | NPM 风格包名 |
| `publishAs` | `code-segment` | 源码发布 |
| `adapted_system_type` | `["standard"]` | 适配标准系统 |
| `inner_kits` | `["//third_party/..."]` | 暴露的构建目标 |

## 维护建议

### 升级步骤

1. **更新源码**
   - 替换为新的上游版本

2. **更新 BUILD.gn**
   ```gn
   cargo_pkg_version = "新版本号"
   ```

3. **检查 features**
   - 确认 Cargo.toml features 是否有变更
   - 必要时更新 BUILD.gn

4. **验证构建**
   ```bash
   gn gen out && ninja -C out third_party/rust/crates/unicode-width:lib
   ```

### 注意事项

- 保持 BUILD.gn 简洁，避免不必要的配置
- 遵循 `ohos_cargo_crate` 模板
- 版本号必须与 Cargo.toml 一致

## 总结

| 项目 | 状态 |
|------|------|
| 构建复杂度 | **低** |
| 特殊配置 | **无** |
| 依赖数量 | **0** |
| 维护难度 | **极低** |

unicode-width 的构建配置展示了如何用最简洁的方式集成 Rust crate 到 OH 构建系统。由于其无外部依赖、无平台相关代码的特点，BUILD.gn 配置非常标准，易于维护。
