# OH 构建适配

## 3.1 BUILD.gn 结构说明

libloading 在 OpenHarmony 中的构建配置非常简洁，使用标准的 `ohos_cargo_crate` 模板。

### 完整 BUILD.gn 配置

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
    crate_name = "libloading"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    output_name = "liblibloading"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "0.7.4"
    cargo_pkg_authors = "Simonas Kazlauskas <libloading@kazlauskas.me>"
    cargo_pkg_name = "libloading"
    cargo_pkg_description = "Bindings around the platform's dynamic library loading primitives with greatly improved memory safety."
    deps = ["//third_party/rust/crates/cfg-if:lib"]
    module_output_extension = ".rlib"
    part_name = "rust_autocfg"
    subsystem_name = "thirdparty"
}
```

## 3.2 配置项详解

### 核心配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| crate_name | "libloading" | Rust crate 名称 |
| crate_type | "rlib" | 输出类型为 Rust 静态库 |
| crate_root | "src/lib.rs" | 入口文件路径 |
| output_name | "liblibloading" | 输出文件名 |

### 构建配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| edition | "2015" | Rust Edition 版本 |
| cargo_pkg_version | "0.7.4" | 上游版本号 |
| deps | cfg-if | 唯一依赖 |
| module_output_extension | ".rlib" | Rust 库文件扩展名 |

### OH 特有配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| part_name | "rust_autocfg" | 所属 Part 名称 |
| subsystem_name | "thirdparty" | 所属子系统名称 |

## 3.3 与上游构建系统的差异

### 上游构建配置（Cargo.toml）

```toml
[package]
name = "libloading"
version = "0.7.4"
edition = "2015"

[target.'cfg(unix)'.dependencies]
cfg-if = "1"

[target.'cfg(windows)'.dependencies]
winapi = { version = "0.3", features = [...] }
```

### OH 构建配置对比

| 配置项 | 上游 (Cargo.toml) | OH (BUILD.gn) | 差异 |
|--------|-------------------|---------------|------|
| 入口文件 | Cargo.toml 自动推断 | crate_root 显式指定 | 无差异 |
| 依赖管理 | Cargo.toml | deps 数组 | 机制不同 |
| 条件编译 | target.'cfg(...)' | 通过 cfg-if 在源码中 | 无差异 |
| 构建产物 | target/debug/liblibloading.rlib | ohos_crate 输出 | 机制不同 |

### 主要差异说明

1. **依赖声明机制不同**
   - 上游：使用 Cargo.toml 的 dependencies 表
   - OH：使用 BUILD.gn 的 deps 数组

2. **条件编译位置不同**
   - 上游：在 Cargo.toml 中声明 target-specific dependencies
   - OH：通过 Rust 源码中的 cfg-if 实现

3. **构建产物位置不同**
   - 上游：target/ 目录
   - OH：ohos_crate 模板输出到构建产物目录

## 3.4 关键编译选项

### Defines

libloading 的 BUILD.gn 中**未定义**任何 OH 特定的 `defines`。

这意味着：
- 源码使用标准的 Rust 条件编译
- 无需 OH 特定的编译宏
- 平台检测通过 cfg-if 自动完成

### Configs

未使用 `configs` 数组进行特殊配置。

### Flags

未使用 `cflags`, `ldflags`, `rustflags` 等特殊标志。

## 3.5 特殊处理说明

### 1. Rust Edition 2015

libloading 使用 Rust 2015 Edition，这是一个相对较老的版本：

```rust
// Rust 2015 风格的函数定义
pub fn new(path: &Path) -> Result<Library, Error> {
    // ...
}
```

**说明**：
- 2015 Edition 的代码与现代 Rust 兼容
- OH 的 Rust 工具链支持 2015 Edition
- 升级到新 Edition 需要评估兼容性

### 2. cfg-if 依赖

libloading 依赖 `cfg-if` 实现跨平台条件编译：

```rust
// src/lib.rs
#[cfg(unix)]
mod unix;

#[cfg(windows)]
mod windows;

#[cfg(unix)]
pub use self::unix::Library;

#[cfg(windows)]
pub use self::windows::Library;
```

在 OH 中：
- cfg-if 的 Unix 分支包含 OpenHarmony 的支持
- 无需额外配置即可工作

### 3. 依赖项配置

```gn
deps = ["//third_party/rust/crates/cfg-if:lib"]
```

| 依赖项 | 用途 | OH 路径 |
|--------|------|---------|
| cfg-if | 跨平台条件编译 | //third_party/rust/crates/cfg-if |

## 3.6 构建产物

### 输出产物

| 产物类型 | 文件名 | 路径 |
|---------|--------|------|
| rlib 静态库 | liblibloading.rlib | 构建产物目录 |

### 符号可见性

libloading 编译为 `rlib` 类型：
- **rlib**: Rust 静态库，主要用于静态链接到其他 Rust crate
- **不生成**: .so/.dylib 动态库（libloading 本身不提供动态库）

## 3.7 构建验证

### 构建命令

```bash
# 在 libloading 目录下执行
cd third_party/rust/crates/libloading

# 使用 OH 构建系统
hb build -p rust_autocfg -T lib
```

### 成功标志

| 检查项 | 预期结果 |
|--------|----------|
| 编译错误 | 无 |
| 警告数量 | 极少或无 |
| 产物生成 | liblibloading.rlib |
| 产物大小 | ~50KB-100KB（典型值） |

## 3.8 常见问题

### Q1: 为什么 libloading 不需要 .so 动态库？

**答**: libloading 是一个**加载动态库的工具库**，它本身编译为静态库（rlib）供 Rust 程序链接使用。libloading 编译出的代码会被链接到使用它的 Rust 程序中，运行时通过系统调用（dlopen 等）来加载实际的动态库文件。

### Q2: 如何在 OH 中使用 libloading？

**答**: 在使用 libloading 的 Rust crate 的 BUILD.gn 中添加依赖：

```gn
ohos_rust_library("my_rust_crate") {
    # ...
    deps = [
        "//third_party/rust/crates/libloading:lib",
    ]
}
```

然后在 Rust 代码中：

```rust
extern crate libloading;

use libloading::{Library, Symbol};
```

### Q3: libloading 是否支持 OpenHarmony 的特殊功能？

**答**: libloading 目前只提供标准的动态库加载功能。如果需要 OH 特有的动态加载功能，可能需要扩展或定制，但这超出了当前 libloading 的设计范围。

## 3.9 总结

libloading 在 OpenHarmony 中的构建适配具有以下特点：

| 特点 | 说明 |
|------|------|
| 适配复杂度 | 极低（仅使用标准模板） |
| Patch 数量 | 0（无需 OH 特定修改） |
| 维护成本 | 低（跟随上游版本即可） |
| 构建产物 | rlib 静态库 |

这种简洁的适配方式表明：
1. OpenHarmony 的 Rust 构建系统成熟且标准
2. libloading 的跨平台设计有效
3. POSIX 兼容性良好
