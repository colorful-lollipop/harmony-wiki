# 03 - OpenHarmony 构建适配

## 3.1 BUILD.gn 配置详解

### 完整配置

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
  crate_name = "quote"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2018"
  cargo_pkg_version = "1.0.37"
  cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
  cargo_pkg_name = "quote"
  cargo_pkg_description = "Quasi-quoting macro quote!(...)"
  deps = [ "//third_party/rust/crates/proc-macro2:lib" ]
  features = [ "proc-macro" ]
  module_output_extension = ".rlib"
  part_name = "rust_quote"
  subsystem_name = "thirdparty"
}
```

### 配置项分析

#### 基础元数据

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | quote | Rust crate 名称 |
| `crate_type` | rlib | Rust 库格式（静态库） |
| `crate_root` | src/lib.rs | crate 入口文件 |
| `part_name` | rust_quote | OH 组件名称 |
| `subsystem_name` | thirdparty | 所属子系统 |

#### 版本信息

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cargo_pkg_version` | 1.0.37 | crate 版本 |
| `cargo_pkg_authors` | David Tolnay | 上游作者 |
| `cargo_pkg_name` | quote | Cargo 包名 |
| `cargo_pkg_description` | ... | 包描述 |

**注**: 这些元数据字段用于生成 Cargo.toml 兼容信息，便于工具链处理。

#### 编译配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `edition` | 2018 | Rust Edition |
| `features` | ["proc-macro"] | 启用的功能特性 |
| `sources` | ["src/lib.rs"] | 源文件列表 |
| `module_output_extension` | .rlib | 输出文件扩展名 |

## 3.2 与 Cargo.toml 的映射

### 上游 Cargo.toml

```toml
[package]
name = "quote"
version = "1.0.37"
authors = ["David Tolnay <dtolnay@gmail.com>"]
edition = "2018"
license = "MIT OR Apache-2.0"
description = "Quasi-quoting macro quote!(...)"
rust-version = "1.56"

[dependencies]
proc-macro2 = { version = "1.0.80", default-features = false }

[features]
default = ["proc-macro"]
proc-macro = ["proc-macro2/proc-macro"]
```

### 映射对比表

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `[package] name` | `crate_name` / `cargo_pkg_name` | 名称一致 |
| `[package] version` | `cargo_pkg_version` | 版本一致 |
| `[package] authors` | `cargo_pkg_authors` | 作者信息 |
| `[package] edition` | `edition` | Edition 一致 |
| `[dependencies]` | `deps` | 依赖映射 |
| `[features] default` | `features` | 功能特性 |

### 依赖映射

**Cargo.toml**:
```toml
[dependencies]
proc-macro2 = { version = "1.0.80", default-features = false }
```

**BUILD.gn**:
```gn
deps = [ "//third_party/rust/crates/proc-macro2:lib" ]
```

映射规则：
- Cargo 包名 `proc-macro2` → GN 路径 `//third_party/rust/crates/proc-macro2`
- Cargo target `:lib` → GN target `:lib`

## 3.3 关键配置详解

### 3.3.1 Edition 配置

```gn
edition = "2018"
```

- **说明**: 使用 Rust 2018 Edition
- **影响**: 决定语法解析、模块系统、关键字等行为
- **上游一致**: 与 Cargo.toml 中的 `edition = "2018"` 完全对应

### 3.3.2 Feature 配置

```gn
features = [ "proc-macro" ]
```

#### Feature 详情

| Feature | 默认 | 说明 |
|---------|------|------|
| `proc-macro` | ✅ 是 | 启用 proc-macro2 的 proc-macro 功能 |

#### Feature 传递

```
quote/proc-macro → proc-macro2/proc-macro
```

Cargo.toml 中定义：
```toml
[features]
default = ["proc-macro"]
proc-macro = ["proc-macro2/proc-macro"]
```

**作用**:
- 启用 proc-macro2 对 `proc_macro` API 的封装
- 允许 quote 与 rustc 的过程宏系统交互
- 是 quote 的核心功能，必须启用

### 3.3.3 输出配置

```gn
crate_type = "rlib"
module_output_extension = ".rlib"
```

#### rlib 格式说明

- **rlib**: Rust 静态库格式
- **用途**: 供其他 Rust crate 链接使用
- **特点**: 包含元数据，支持 Rust 特定的链接特性

#### 为什么不使用其他格式

| 格式 | 适用场景 | quote 不使用的原因 |
|------|----------|-------------------|
| rlib | Rust 库依赖 | ✅ quote 的标准格式 |
| dylib | 动态库 | 不需要动态链接 |
| cdylib | C 兼容动态库 | 不需要 C 接口 |
| staticlib | C 兼容静态库 | 不需要 C 接口 |
| bin | 可执行文件 | 是库不是程序 |

## 3.4 与上游构建系统的差异

### 构建系统对比

| 维度 | 上游 (Cargo) | OpenHarmony (GN) |
|------|--------------|------------------|
| 构建工具 | Cargo | GN + Ninja |
| 配置文件 | Cargo.toml | BUILD.gn |
| 包管理 | crates.io | OH 代码仓库 |
| 依赖解析 | Cargo 自动处理 | GN 显式声明 |
| 交叉编译 | 支持 | 支持 |

### OH 特有的构建适配

#### 1. ohos_cargo_crate 模板

```gn
import("//build/ohos.gni")

ohos_cargo_crate("lib") {
  # ...
}
```

- `ohos_cargo_crate` 是 OH 提供的标准模板
- 封装了 Rust 编译的通用逻辑
- 自动处理 Cargo 兼容层

#### 2. 显式源文件声明

**Cargo** (自动发现):
```toml
# 自动发现 src/lib.rs
```

**GN** (显式声明):
```gn
sources = [ "src/lib.rs" ]
```

GN 要求显式声明源文件，这是为了：
- 精确的依赖跟踪
- 更好的增量编译
- 构建系统的可重现性

#### 3. 依赖显式路径

**Cargo** (版本解析):
```toml
[dependencies]
proc-macro2 = "1.0.80"
```

**GN** (显式路径):
```gn
deps = [ "//third_party/rust/crates/proc-macro2:lib" ]
```

OH 使用单一代码仓库，所有依赖都在固定路径。

## 3.5 构建过程分析

### 编译流程

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 构建流程                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. GN 解析 BUILD.gn                                        │
│     └─ 提取 crate_name, edition, features, deps            │
│                                                             │
│  2. 生成 Ninja 文件                                         │
│     └─ 创建 rustc 编译命令                                  │
│                                                             │
│  3. 依赖解析                                                │
│     └─ 确保 proc-macro2 先编译完成                          │
│                                                             │
│  4. rustc 编译                                              │
│     └─ 调用 rustc 编译 quote                                │
│        rustc --edition 2018 \                               │
│              --crate-type rlib \                            │
│              --crate-name quote \                           │
│              --extern proc_macro2=... \                     │
│              --cfg feature=\"proc-macro\" \                 │
│              src/lib.rs                                     │
│                                                             │
│  5. 输出 rlib                                               │
│     └─ libquote.rlib 供其他 crate 链接                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 编译命令示例

基于 BUILD.gn 配置，实际生成的 rustc 命令类似于：

```bash
rustc \
  --edition 2018 \
  --crate-type rlib \
  --crate-name quote \
  --extern proc_macro2=/path/to/libproc_macro2.rlib \
  --cfg feature="proc-macro" \
  -o /output/path/libquote.rlib \
  third_party/rust/crates/quote/src/lib.rs
```

## 3.6 特殊配置说明

### 3.6.1 无特殊配置

与其他一些 third_party 库不同，quote 的 BUILD.gn **无任何特殊配置**：

```gn
# 以下配置均未使用，证明无特殊需求：
# - defines: 无自定义宏定义
# - configs: 无特殊编译配置
# - cflags: 无特殊编译器标志
# - include_dirs: 无额外头文件路径
# - rustflags: 无特殊 Rust 编译器标志
```

这进一步验证了 quote 的原生兼容性。

### 3.6.2 标准模板化配置

```gn
ohos_cargo_crate("lib") {
  # ...
  part_name = "rust_quote"
  subsystem_name = "thirdparty"
}
```

- `part_name` 和 `subsystem_name` 是 OH 组件系统的标准字段
- 用于组件管理和依赖追踪
- 无技术影响，纯管理属性

## 3.7 构建验证

### 编译验证命令

```bash
# 在 OH 代码根目录执行
# 编译 quote crate
hb build //third_party/rust/crates/quote:lib

# 或编译依赖 quote 的组件来间接验证
hb build //third_party/rust/crates/syn:lib
hb build //third_party/rust/crates/serde/serde_derive:lib
```

### 验证要点

| 验证项 | 方法 | 预期结果 |
|--------|------|----------|
| 编译成功 | hb build | 无错误 |
| 输出文件 | ls out/.../*.rlib | 生成 libquote.rlib |
| 依赖正确 | 查看编译日志 | 正确链接 proc-macro2 |
| Feature 生效 | 检查 --cfg | 包含 feature="proc-macro" |

## 3.8 维护指南

### BUILD.gn 修改场景

| 场景 | 操作 | 注意事项 |
|------|------|----------|
| 升级版本 | 修改 cargo_pkg_version | 同步更新 sources 如有新增 |
| 新增 feature | 修改 features 列表 | 确保依赖支持该 feature |
| 新增依赖 | 修改 deps | 确保依赖已适配 OH |
| 新增源文件 | 修改 sources | 保持显式声明 |

### 版本升级示例

假设从 1.0.37 升级到 1.0.38：

```gn
# 修改前
cargo_pkg_version = "1.0.37"

# 修改后
cargo_pkg_version = "1.0.38"
```

**检查项**:
- [ ] 确认上游未新增源文件
- [ ] 确认上游未修改 edition
- [ ] 确认 features 未变更
- [ ] 确认依赖版本要求未变更
- [ ] 全面编译验证

## 3.9 结论

### 构建适配总结

1. **标准配置**: BUILD.gn 是完全标准的 `ohos_cargo_crate` 配置
2. **无特殊适配**: 无需 defines、configs、cflags 等特殊配置
3. **简单依赖**: 仅依赖 proc-macro2，依赖树清晰
4. **易于维护**: 升级只需修改版本号

### 与上游的等价性

| 维度 | 上游 Cargo | OH GN | 等价性 |
|------|------------|-------|--------|
| 版本 | 1.0.37 | 1.0.37 | ✅ 一致 |
| Edition | 2018 | 2018 | ✅ 一致 |
| Features | ["proc-macro"] | ["proc-macro"] | ✅ 一致 |
| 依赖 | proc-macro2 | proc-macro2 | ✅ 一致 |
| 输出 | rlib | rlib | ✅ 一致 |

**结论**: OH 的 BUILD.gn 配置与上游 Cargo 配置完全等价，证明 quote crate 在 OpenHarmony 中保持原生行为。

---

*文档版本*: 1.0  
*最后更新*: 2026-02-08  
*配置文件*: BUILD.gn, Cargo.toml
