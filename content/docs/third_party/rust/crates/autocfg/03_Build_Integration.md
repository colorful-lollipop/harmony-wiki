# 03 - OpenHarmony 构建适配

## 概述

autocfg 在 OpenHarmony 中使用标准的 Rust crate 构建流程，通过 `ohos_cargo_crate` 模板进行适配。本章详细说明 BUILD.gn 的结构、关键配置以及与上游构建系统的差异。

---

## BUILD.gn 完整分析

### 文件内容

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
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
  crate_name = "autocfg"

  # epoch = "1"
  crate_type = "rlib"

  visibility = [ "//third_party/rust/*" ]
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2015"
  module_output_extension = ".rlib"
  part_name = "rust_autocfg"
  subsystem_name = "thirdparty"
}
```

### 配置项详解

#### 基本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `"autocfg"` | crate 名称，与 Cargo.toml 的 `name` 字段一致 |
| `crate_type` | `"rlib"` | 输出类型为 Rust 静态库（.rlib） |
| `crate_root` | `"src/lib.rs"` | crate 根文件路径 |

#### 源代码配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | `["src/lib.rs"]` | 显式列出源文件。autocfg 是单文件库，只有 lib.rs |
| `edition` | `"2015"` | Rust edition 2015。这是 autocfg 兼容的最低 edition |

**关于 edition**：
- Rust 2015 是 autocfg 支持的最低版本
- 与 Rust 1.0+ 的兼容性承诺一致
- OH 使用的 Rust 版本支持 2015 edition

#### 可见性配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `visibility` | `["//third_party/rust/*"]` | 限制只有 `third_party/rust/` 下的目标可以依赖此库 |

**设计原因**：
- autocfg 是底层构建工具，不应被上层应用直接依赖
- 只有其他 Rust crate 的 build.rs 需要它
- 遵循最小权限原则

#### 组件元数据

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `part_name` | `"rust_autocfg"` | OH 部件名称，对应 bundle.json 中的 `component.name` |
| `subsystem_name` | `"thirdparty"` | 所属子系统 |
| `module_output_extension` | `".rlib"` | 输出文件扩展名 |

---

## 与上游构建系统的对比

### 上游构建方式

autocfg 上游使用 Cargo 构建：

```toml
# Cargo.toml
[package]
name = "autocfg"
version = "1.4.0"
edition = "2015"
rust-version = "1.0"

[dependencies]
# 无依赖
```

构建命令：
```bash
cargo build --release
# 输出: target/release/libautocfg.rlib
```

### OH 构建方式

OpenHarmony 使用 GN + Ninja 构建系统：

```bash
# 在 OH 源码树中编译
gn gen out/default
ninja -C out/default third_party/rust/crates/autocfg:lib
# 输出: out/default/third_party/rust/crates/autocfg/lib.rlib
```

### 差异对比表

| 方面 | 上游 (Cargo) | OpenHarmony (GN) |
|------|-------------|------------------|
| **构建系统** | Cargo | GN + Ninja |
| **配置文件** | `Cargo.toml` | `BUILD.gn` |
| **依赖管理** | Cargo 自动处理 | GN deps 显式声明 |
| **输出路径** | `target/` | `out/<target>/` |
| **交叉编译** | `--target` 参数 | GN 工具链配置 |
| **构建缓存** | Cargo 内置 | Ninja 增量构建 |

### 关键差异说明

#### 1. 依赖处理

**上游（Cargo）**:
```toml
# 下游 crate 的 Cargo.toml
[build-dependencies]
autocfg = "1"
```
Cargo 自动下载并构建。

**OH（GN）**:
```gn
# 下游 crate 的 BUILD.gn
rust_crate("xxx") {
  build_deps = ["//third_party/rust/crates/autocfg:lib"]
}
```
GN 显式声明依赖路径。

#### 2. 版本锁定

**上游**: 使用 `Cargo.lock` 锁定版本

**OH**: 通过 Git 子模块锁定特定 commit
```bash
# autocfg 在 OH 中的版本
# .git/FETCH_HEAD 显示同步自上游 1.4.0 标签
```

---

## 关键编译选项分析

### autocfg 的编译选项

autocfg 的 BUILD.gn 中**没有**显式配置的选项：

| 选项类型 | 状态 | 说明 |
|---------|------|------|
| `defines` | 未配置 | 无需预处理器定义 |
| `configs` | 未配置 | 使用默认 Rust 编译配置 |
| `cflags` / `rustflags` | 未配置 | 使用默认编译标志 |
| `deps` | 未配置 | 无依赖 crate |

### 为什么这么简单？

1. **纯 Rust 实现**: 无 C 依赖，无需链接系统库
2. **标准库依赖**: 仅依赖 Rust std/core，无需额外配置
3. **无特性开关**: Cargo.toml 中无 `[features]` 部分
4. **无平台代码**: 无需条件编译或平台特定逻辑

### 与 memoffset 的对比

作为对比，看看依赖 autocfg 的 memoffset 的 BUILD.gn：

```gn
ohos_cargo_crate("lib") {
  crate_name = "memoffset"
  crate_type = "rlib"
  
  # 关键差异：声明了 build_deps
  build_deps = ["//third_party/rust/crates/autocfg:lib"]
  
  # 其他配置类似...
}
```

memoffset 需要 `build_deps` 来声明对 autocfg 的依赖，而 autocfg 本身无依赖。

---

## 构建流程详解

### 完整的构建流程

```
┌─────────────────────────────────────────────────────────────┐
│                     构建启动                                  │
│  ninja -C out/default third_party/rust/crates/autocfg:lib   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    GN 解析 BUILD.gn                         │
│  1. 加载 ohos_cargo_crate 模板                               │
│  2. 解析 crate_name, sources, edition 等配置                │
│  3. 生成 Ninja 构建规则                                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Rust 编译器调用                            │
│  rustc --edition 2015 --crate-type rlib                       │
│       --crate-name autocfg src/lib.rs                         │
│       -o out/.../lib.rlib                                    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    输出 rlib 文件                            │
│  out/default/third_party/rust/crates/autocfg/lib.rlib       │
└─────────────────────────────────────────────────────────────┘
```

### 实际的 rustc 调用

Ninja 生成的实际编译命令类似于：

```bash
rustc \
  --edition 2015 \
  --crate-type rlib \
  --crate-name autocfg \
  --emit link=lib.rlib \
  --emit metadata=lib.rmeta \
  -C opt-level=2 \
  -C debuginfo=0 \
  --cap-lints allow \
  src/lib.rs
```

**说明**：
- `--edition 2015`: 使用 Rust 2015 edition
- `-C opt-level=2`: 优化级别 2（Release 模式）
- `--cap-lints allow`: 将所有 lint 警告降级为允许（第三方库常见做法）

---

## 特殊处理说明

### 无特殊处理

autocfg 的构建适配**没有任何特殊处理**：

- ❌ 无自定义编译脚本
- ❌ 无预处理器定义
- ❌ 无平台特定代码
- ❌ 无额外链接库
- ❌ 无 post-processing

这是**最简单类型**的第三方 Rust crate 适配。

### 与其他 crate 的对比

| crate | 特殊处理 | 说明 |
|-------|---------|------|
| **autocfg** | **无** | 纯 Rust，标准库依赖 |
| libc | 可能需配置目标平台 | 系统调用号定义 |
| openssl-sys | 需链接系统 OpenSSL | 绑定系统库 |
| bindgen | 需 clang 路径 | 调用 libclang |

---

## 维护指南

### 日常维护

**无需特别维护**，因为：
- 无 Patch 需要更新
- 构建配置简单稳定
- 上游 API 向后兼容

### 升级检查清单

当升级 autocfg 到新版本时，检查：

- [ ] 新版本 `Cargo.toml` 的 `edition` 是否变化
- [ ] 新版本是否新增 `features`
- [ ] 新版本是否新增依赖
- [ ] 下游 crate（memoffset 等）是否仍兼容

### 故障排查

#### 问题 1: 编译失败

**现象**: autocfg 编译报错

**排查步骤**:
1. 检查 Rust 工具链版本
2. 检查 BUILD.gn 的 `edition` 是否匹配
3. 检查 `sources` 列表是否完整

#### 问题 2: 下游 crate 找不到 autocfg

**现象**: memoffset 等 crate 编译时提示找不到 autocfg

**排查步骤**:
1. 检查下游 BUILD.gn 的 `build_deps` 路径是否正确
2. 检查 autocfg 的 `visibility` 是否允许下游访问
3. 检查 GN 依赖图是否包含 autocfg

---

## 总结

autocfg 的 OpenHarmony 构建适配是**极简模式**的典范：

| 方面 | 状态 | 说明 |
|------|------|------|
| 配置复杂度 | **低** | 仅基本配置，无特殊选项 |
| 维护成本 | **零** | 标准模板，无需调整 |
| 升级难度 | **低** | 直接同步上游，无迁移负担 |
| 构建稳定性 | **高** | 无平台依赖，编译可靠 |

这得益于 autocfg 本身的设计：**功能单一、平台无关、纯 Rust 实现**。

---

*文档生成时间: 2025-02-08*
