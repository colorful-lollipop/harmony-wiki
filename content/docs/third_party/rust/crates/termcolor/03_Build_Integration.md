# OH 构建适配

## 构建系统概述

termcolor 在 OpenHarmony 中使用 **OHOS 构建系统（GN + Ninja）** 进行构建，通过 `ohos_cargo_crate` 模板集成 Rust cargo 项目。

## BUILD.gn 完整配置

```
third_party/rust/crates/termcolor/BUILD.gn
```

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
    crate_name = "termcolor"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "1.2.0"
    cargo_pkg_authors = "Andrew Gallant <jamslam@gmail.com>"
    cargo_pkg_name = "termcolor"
    module_output_extension = ".rlib"
    part_name = "rust_termcolor"
    subsystem_name = "thirdparty"
}
```

## 配置字段详解

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | termcolor | Rust crate 名称，对应 Cargo.toml 中的 name |
| `crate_type` | rlib | Rust 静态库格式，用于静态链接 |
| `crate_root` | src/lib.rs | crate 主入口文件 |
| `sources` | ["src/lib.rs"] | 源文件列表（仅主入口） |
| `edition` | 2018 | Rust edition 版本 |
| `cargo_pkg_version` | 1.2.0 | 上游版本号 |
| `cargo_pkg_authors` | Andrew Gallant... | 包作者信息 |
| `cargo_pkg_name` | termcolor | cargo 包名称 |
| `module_output_extension` | .rlib | 输出文件扩展名 |
| `part_name` | rust_termcolor | OH 组件名称 |
| `subsystem_name` | thirdparty | 所属子系统 |

## Cargo.toml 配置

```
third_party/rust/crates/termcolor/Cargo.toml
```

```toml
[package]
name = "termcolor"
version = "1.2.0"
authors = ["Andrew Gallant <jamslam@gmail.com>"]
description = "A simple cross platform library for writing colored text to a terminal."
documentation = "https://docs.rs/termcolor"
homepage = "https://github.com/BurntSushi/termcolor"
repository = "https://github.com/BurntSushi/termcolor"
readme = "README.md"
keywords = ["windows", "win", "color", "ansi", "console"]
license = "Unlicense OR MIT"
edition = "2018"

[lib]
name = "termcolor"
bench = false

[target.'cfg(windows)'.dependencies]
winapi-util = "0.1.3"

[dev-dependencies]
# doc-comment = "0.3"  # 注释掉以满足 MSRV 1.34.0
```

### OH 特定配置说明

| 配置项 | 状态 | 说明 |
|--------|------|------|
| `[lib]` | 无修改 | 与上游一致 |
| `[target.'cfg(windows)'.dependencies]` | 无修改 | Windows 特定依赖，OH 不激活此路径 |
| `[dev-dependencies]` | 有修改 | doc-comment 被注释掉 |

**注意**：`doc-comment` 依赖被注释掉是因为它要求 Rust 1.43+，而 termcolor 声称支持 Rust 1.34.0。这可能是上游的一个历史遗留问题，OH 直接继承了上游的配置。

## 与上游构建系统的差异

| 方面 | 上游 (Cargo) | OH (GN) |
|------|-------------|---------|
| 构建工具 | cargo | gn + ninja |
| 构建模板 | Cargo.toml | ohos_cargo_crate |
| 输出格式 | .rlib | .rlib |
| 版本管理 | Cargo.toml | bundle.json |
| 依赖声明 | Cargo.toml | BUILD.gn deps |
| 条件编译 | cfg 属性 | cfg 属性（兼容） |

### 关键差异分析

**1. 构建系统差异**

上游使用标准的 Cargo 构建系统，而 OH 使用 GN + Ninja 构建系统。`ohos_cargo_crate` 模板充当了两者之间的桥梁。

```gn
# GN 侧
ohos_cargo_crate("lib") {
    # 配置映射到 Cargo.toml
}

# Cargo.toml 侧
[package]
# 同样的配置
```

**2. 条件编译一致性**

termcolor 使用 Rust 的 `#[cfg(...)]` 属性进行条件编译，OH 构建系统完全支持此机制：

```rust
#[cfg(windows)]
use wincolor;  // Windows 路径

#[cfg(unix)]
use ansi;      // Unix/OH 路径
```

在 OpenHarmony 上，`#[cfg(unix)]` 被激活，库使用 ANSI 转义序列进行颜色输出。

**3. 依赖处理**

| 依赖类型 | 上游处理 | OH 处理 |
|---------|---------|---------|
| Windows 依赖 | Cargo 自动解析 | 构建时不激活 |
| dev-dependencies | cargo test 时解析 | hb test 时解析 |

## 构建产物

### 标准构建

```bash
hb build //third_party/rust/crates/termcolor:lib
```

**输出产物**：

```
out/.../thirdparty/rust/crates/termcolor/libtermcolor.rlib
```

### 构建产物验证

```bash
# 查看库信息
rustc --print crate-id out/.../libtermcolor.rlib

# 检查符号表
nm out/.../libtermcolor.rlib | grep -i color
```

**预期符号**：
- `termcolor::` 命名空间下的公开 API
- `WriteColor` trait 相关符号
- `Color`、`ColorSpec`、`StandardStream` 等类型

## 特殊处理

### 无特殊编译选项

termcolor 的 BUILD.gn 中**未配置**以下选项：

| 配置项 | 状态 | 说明 |
|--------|------|------|
| `defines` | 未配置 | 无预处理器宏定义 |
| `configs` | 未配置 | 无特殊编译配置 |
| `cflags` / `cxxflags` | 不适用 | Rust 项目 |
| `ldflags` | 未配置 | 无链接器标志 |
| `extra_configs` | 未配置 | 无额外配置 |

### 版本映射

| 维度 | 值 |
|------|-----|
| 上游版本 | 1.2.0 |
| OH bundle 版本 | 6.1 |
| 版本策略 | bundle 版本独立于上游 |

bundle.json 中的版本 "6.1" 是 OH 内部版本号，与上游语义版本无关。

## 构建测试

### 本地构建测试

```bash
# 完整构建
hb build //third_party/rust/crates/termcolor

# 仅构建库
hb build //third_party/rust/crates/termcolor:lib

# 查看构建配置
hb build --build-variant debug --target dsoftbus -v //third_party/rust/crates/termcolor:lib
```

### 依赖构建测试

由于 termcolor 被多个库依赖，构建后应验证依赖者的构建：

```bash
# 测试 clap 依赖
hb build //third_party/rust/crates/clap:lib

# 测试 env_logger 依赖  
hb build //third_party/rust/crates/env_logger:lib

# 测试 codespan-reporting 依赖
hb build //third_party/rust/crates/codespan/codespan-reporting:lib
```

## 故障排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 构建失败 | Rust 版本过低 | 确保使用 Rust 1.34.0+ |
| 符号未找到 | 错误的链接顺序 | 检查依赖图 |
| Windows 路径激活 | cfg 属性错误 | 确认构建目标是 OH |

### 调试技巧

```bash
# 查看详细的构建命令
hb build -v //third_party/rust/crates/termcolor:lib

# 检查 cargo 配置
cat ~/.cargo/config.toml

# 查看条件编译结果
cargo expand --lib termcolor | grep -i cfg
```
