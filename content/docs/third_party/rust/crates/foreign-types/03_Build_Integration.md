# 03_Build_Integration.md - OH 构建适配

本文档详细说明 OpenHarmony 如何将 `foreign-types` 的 Cargo 构建系统适配到 GN 构建系统。

---

## 📋 构建系统概览

### 上游构建系统 vs OH 构建系统

| 维度 | 上游（Cargo） | OH（GN） | 说明 |
|------|---------------|----------|------|
| **构建工具** | `cargo build` | `gn gen` + `ninja` | 完全不同的构建系统 |
| **依赖管理** | `Cargo.toml` | `BUILD.gn` | 声明格式不同 |
| **输出类型** | 自动识别 | 显式指定 `crate_type` | GN 需要明确声明 |
| **依赖路径** | 相对路径 | GN 绝对路径 | GN 使用 `//` 前缀 |
| **组件注册** | crates.io | bundle.json | OH 组件化系统 |
| **许可证检查** | 自动（cargo publish） | 手动（README.OpenSource） | OH 需要合规声明 |

### OH 适配策略

OH 采用 **"仅构建适配"** 策略：
1. ✅ **源代码不变**: 所有 Rust 源文件与上游完全一致
2. ✅ **仅新增配置**: 添加 BUILD.gn 和 bundle.json
3. ✅ **最小侵入**: 无 OH 特有宏、条件编译或代码分支

---

## 🔧 BUILD.gn 文件结构

### 文件清单

| 文件路径 | Crate 类型 | 版本 | 说明 |
|---------|-----------|------|------|
| `foreign-types/BUILD.gn` | 主 crate | 0.3.2 | 对应 foreign-types-0.3.2 |
| `foreign-types-shared/BUILD.gn | Shared crate | 0.1.1 | 对应 foreign-types-shared-0.1.1 |

### 文件模板结构

所有 BUILD.gn 文件遵循统一模板：

```gn
# 1. 版权声明（OH 特有）
# Copyright (c) 2023 Huawei Device Co., Ltd.
# ...

# 2. 导入 OH 构建模板
import("//build/ohos.gni")

# 3. 定义 crate 构建目标
ohos_cargo_crate("lib") {
  # 基本配置
  crate_name = "foreign_types"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  # 源文件
  sources = [ "src/lib.rs" ]

  # Cargo 元数据（用于版本追踪）
  edition = "2015"
  cargo_pkg_version = "0.3.2"
  cargo_pkg_authors = "Steven Fackler <sfackler@gmail.com>"
  cargo_pkg_name = "foreign-types"
  cargo_pkg_description = "A framework for Rust wrappers over C APIs"

  # 依赖（GN 格式）
  deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]

  # 输出配置
  module_output_extension = ".rlib"

  # OH 组件元数据
  part_name = "rust_foreign_types"
  subsystem_name = "thirdparty"
}
```

---

## 📦 foreign-types/BUILD.gn 详解

### 完整文件内容

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
  crate_name = "foreign_types"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2015"
  cargo_pkg_version = "0.3.2"
  cargo_pkg_authors = "Steven Fackler <sfackler@gmail.com>"
  cargo_pkg_name = "foreign-types"
  cargo_pkg_description = "A framework for Rust wrappers over C APIs"
  deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
  module_output_extension = ".rlib"
  part_name = "rust_foreign_types"
  subsystem_name = "thirdparty"
}
```

### 配置项详解

#### 1. 版权声明

**OH 特有**:
```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
```

**目的**: OH 合规性要求，声明华为拥有该构建配置文件的版权。

#### 2. 导入 OH 构建模板

```gn
import("//build/ohos.gni")
```

**说明**: 导入 OH 的构建模板，包含 `ohos_cargo_crate` 等规则。

#### 3. ohos_cargo_crate 目标

```gn
ohos_cargo_crate("lib") {
  ...
}
```

**说明**: 定义一个 Rust crate 构建目标，名称为 `lib`。

#### 4. 基本配置

| 配置项 | 值 | 说明 | 对应 Cargo.toml |
|-------|-----|------|----------------|
| `crate_name` | `foreign_types` | crate 名称 | `[package].name` |
| `crate_type` | `"rlib"` | 输出类型 | 隐式（Cargo 自动识别） |
| `crate_root` | `src/lib.rs` | 入口文件 | 隐式（Cargo 自动识别） |

**crate_type 说明**:
- `rlib`: Rust 静态库（`.rlib`），用于链接到其他 Rust crates
- `dylib`: Rust 动态库（`.so` / `.dll`）
- `bin`: 可执行文件

foreign-types 使用 `rlib`，因为它是一个库 crate。

#### 5. 源文件

```gn
sources = [ "src/lib.rs" ]
```

**说明**: 列出所有源文件。foreign-types 只有一个 `src/lib.rs` 文件。

#### 6. Cargo 元数据

| 配置项 | 值 | 说明 | 对应 Cargo.toml |
|-------|-----|------|----------------|
| `edition` | `"2015"` | Rust edition | 隐式（Cargo 2018 默认） |
| `cargo_pkg_version` | `"0.3.2"` | 版本号 | `[package].version` |
| `cargo_pkg_authors` | `"Steven Fackler <sfackler@gmail.com>"` | 作者 | `[package].authors` |
| `cargo_pkg_name` | `"foreign-types"` | 包名 | `[package].name` |
| `cargo_pkg_description` | `"A framework for Rust wrappers over C APIs"` | 描述 | `[package].description` |

**说明**: 这些配置项以 `cargo_pkg_` 前缀开头，用于追踪原始 Cargo 元数据，便于版本管理和审计。

#### 7. 依赖

```gn
deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
```

**GN 依赖格式**:
- `//third_party/rust/crates/foreign-types/foreign-types-shared`: 目标路径
- `:lib`: 目标名称

**对比 Cargo 依赖**:
```toml
[dependencies]
foreign-types-shared = { version = "0.1", path = "../foreign-types-shared" }
```

**差异**:
- Cargo 使用相对路径 `../foreign-types-shared`
- GN 使用绝对路径 `//third_party/rust/crates/...`

#### 8. 输出配置

```gn
module_output_extension = ".rlib"
```

**说明**: 指定输出文件的扩展名为 `.rlib`（Rust 静态库）。

#### 9. OH 组件元数据

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `part_name` | `rust_foreign_types` | OH 部件名称 |
| `subsystem_name` | `thirdparty` | OH 子系统名称 |

**说明**: 这些配置项用于 OH 的构建系统，将 crate 注册到正确的部件和子系统。

---

## 📦 foreign-types-shared/BUILD.gn 详解

### 完整文件内容

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
  crate_name = "foreign_types_shared"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2015"
  cargo_pkg_version = "0.1.1"
  cargo_pkg_authors = "Steven Fackler <sfackler@gmail.com>"
  cargo_pkg_name = "foreign-types-shared"
  cargo_pkg_description = "An internal crate used by foreign-types"
}
```

### 与主 crate 的差异

| 配置项 | foreign-types | foreign-types-shared |
|-------|---------------|----------------------|
| **crate_name** | `foreign_types` | `foreign_types_shared` |
| **version** | `0.3.2` | `0.1.1` |
| **description** | "A framework for Rust wrappers over C APIs" | "An internal crate used by foreign-types" |
| **deps** | 有 | 无 |
| **part_name** | `rust_foreign_types` | 无 |
| **subsystem_name** | `thirdparty` | 无 |

**说明**: foreign-types-shared 是一个内部 crate，无外部依赖，也不需要注册到 OH 部件系统（因为它只是一个依赖，不对外暴露）。

---

## 🔗 构建依赖图

### 依赖关系

```
OH 构建系统
    ↓
ohos_cargo_crate("lib")  // foreign-types
    ↓
deps: //third_party/rust/crates/foreign-types/foreign-types-shared:lib
    ↓
ohos_cargo_crate("lib")  // foreign-types-shared
```

### 输出产物

| 目标 | 输出文件 | 类型 |
|------|---------|------|
| `//third_party/rust/crates/foreign-types:lib` | `libforeign_types-<hash>.rlib` | Rust 静态库 |
| `//third_party/rust/crates/foreign-types/foreign-types-shared:lib` | `libforeign_types_shared-<hash>.rlib` | Rust 静态库 |

---

## 🔍 与上游构建系统的差异

### Cargo.toml vs BUILD.gn 对照表

| 功能 | Cargo.toml | BUILD.gn | 说明 |
|------|-----------|----------|------|
| **包名** | `[package].name` | `crate_name` + `cargo_pkg_name` | GN 双重声明 |
| **版本** | `[package].version` | `cargo_pkg_version` | GN 仅元数据 |
| **类型** | 隐式（lib/bin） | 显式 `crate_type` | GN 需明确 |
| **入口** | 隐式 `src/lib.rs` | 显式 `crate_root` | GN 需明确 |
| **依赖** | `[dependencies]` | `deps = []` | 路径格式不同 |
| **作者** | `[package].authors` | `cargo_pkg_authors` | GN 仅元数据 |
| **描述** | `[package].description` | `cargo_pkg_description` | GN 仅元数据 |
| **Rust Edition** | 隐式（2018） | 显式 `edition` | GN 需明确 |

### 依赖路径格式对比

**Cargo（相对路径）**:
```toml
[dependencies]
foreign-types-shared = { path = "../foreign-types-shared" }
```

**GN（绝对路径）**:
```gn
deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
```

**GN 路径格式说明**:
- `//`: OH 根目录
- `third_party/rust/crates/foreign-types/foreign-types-shared`: 相对于根的路径
- `:lib`: 目标名称

### 构建命令对比

**Cargo（上游）**:
```bash
# 在 workspace 根目录
cargo build --release
```

**GN（OH）**:
```bash
# 在 OH 根目录
./build.sh --product-name <product> --build-target foreign_types

# 或手动
gn gen out/ohos
ninja -C out/ohos //third_party/rust/crates/foreign-types:lib
```

---

## 🛠️ OH 构建系统特有配置

### ohos_cargo_crate 模板

`ohos_cargo_crate` 是 OH 的自定义 GN 模板，用于编译 Rust crates。

**功能**:
1. 调用 Rust 编译器（rustc）
2. 处理依赖关系
3. 生成 `.rlib` 文件
4. 注册到 OH 构建系统

**内部实现**（推测）:
```gn
template("ohos_cargo_crate") {
  # 调用 rustc 编译
  action(target_name) {
    script = "//build/rust/build_rust.py"
    sources = invoker.sources
    outputs = [ "${invoker.module_output_extension}" ]
    deps = invoker.deps
    # ...
  }
}
```

### part_name 和 subsystem_name

**part_name**: OH 的部件名称，表示该 crate 属于哪个部件。
- `rust_foreign_types`: 表示这是 Rust 外部类型部件

**subsystem_name**: OH 的子系统名称，表示部件所属的子系统。
- `thirdparty`: 第三方库子系统

**作用**:
- 用于构建系统的模块化组织
- 支持选择性编译（只编译需要的子系统）
- 用于权限和访问控制

---

## 🔧 构建配置示例

### 构建 foreign-types

**方法 1: 通过 OH 构建脚本**:
```bash
./build.sh --product-name <product> --build-target rust_foreign_types
```

**方法 2: 手动使用 GN + Ninja**:
```bash
# 生成构建文件
gn gen out/ohos

# 编译
ninja -C out/ohos //third_party/rust/crates/foreign-types:lib
```

**方法 3: 编译依赖方（间接编译）**:
```bash
# 编译 rust-openssl 会自动编译 foreign-types
./build.sh --product-name <product> --build-target rust_openssl
```

### 查看构建依赖

```bash
# 使用 gn 查看依赖图
gn desc out/ohos //third_party/rust/crates/foreign-types:lib deps

# 使用 gn 查看依赖该库的目标
gn desc out/ohos //third_party/rust/crates/foreign-types:lib all_dependent_configs
```

---

## 📊 构建输出分析

### 输出文件

**主 crate**:
```
out/ohos/obj/third_party/rust/crates/foreign-types/obj/libforeign_types-{hash}.rlib
```

**Shared crate**:
```
out/ohos/obj/third_party/rust/crates/foreign-types/foreign-types-shared/obj/libforeign_types_shared-{hash}.rlib
```

**Hash 说明**: Hash 是基于依赖关系和源文件内容计算的，确保增量编译的正确性。

### 编译产物使用

**其他 crate 依赖 foreign-types**:
```gn
# 例如 rust-openssl 的 BUILD.gn
deps = [ "//third_party/rust/crates/foreign-types:lib" ]
```

**Rust 代码中使用**:
```rust
// rust-openssl/src/ssl/mod.rs
use foreign_types::{ForeignType, ForeignTypeRef};
```

---

## 🎯 特殊处理

### 无条件编译

foreign-types 的构建配置**无任何条件编译**：
- ❌ 无 `if (is_mingw)` 等平台判断
- ❌ 无 `defines` 配置
- ❌ 无 `configs` 配置

**原因**: foreign-types 是纯 Rust 库，与平台无关，无需特殊处理。

### 无额外编译选项

foreign-types 的构建配置**无额外的编译选项**：
- ❌ 无 `rustflags`
- ❌ 无 `features`（Cargo features）
- ❌ 无 `cfg` 条件

**原因**: foreign-types 使用简单的宏和 trait，无需特殊编译选项。

### 无测试配置

foreign-types 的 BUILD.gn **无测试配置**：
- ❌ 无 `test` 目标
- ❌ 无 `unittest` 配置

**原因**: OH 将测试放在单独的测试套件中，不在 BUILD.gn 中定义。

---

## 🚀 性能优化

### 增量编译

OH 构建系统支持**增量编译**：
- 仅重新编译修改过的文件
- 基于源文件内容和依赖关系计算 hash
- 避免不必要的重新编译

### 并行编译

OH 构建系统支持**并行编译**：
- 使用 Ninja 的并行能力
- 默认使用所有 CPU 核心
- 可通过 `-jN` 参数控制并发数

### 静态链接优化

foreign-types 使用 `rlib` 格式：
- ✅ 编译时链接，无运行时开销
- ✅ 支持链接时优化（LTO）
- ✅ 支持跨 crate 内联

---

## 📝 维护指南

### 更新依赖版本

**场景**: foreign-types-shared 版本更新

**步骤**:
1. 更新 `foreign-types-shared/BUILD.gn`:
   ```gn
   cargo_pkg_version = "0.1.2"  # 新版本
   ```
2. 更新 `foreign-types/BUILD.gn`:
   ```gn
   deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
   # 无需修改（路径不变）
   ```
3. 验证构建:
   ```bash
   ./build.sh --product-name <product> --build-target rust_foreign_types
   ```

### 添加新特性（如果需要）

**场景**: 需要启用某个 Cargo feature

**步骤**:
1. 在 `BUILD.gn` 中添加 `features`:
   ```gn
   features = [ "std", "alloc" ]
   ```
2. （目前 foreign-types 无 features，仅为示例）

### 修改编译选项（如果需要）

**场景**: 需要启用 LTO 或其他优化

**步骤**:
1. 在 `BUILD.gn` 中添加 `rustflags`:
   ```gn
   rustflags = [ "-C", "lto=fat" ]
   ```
2. 验证构建和性能

---

## 🔍 故障排查

### 构建失败: "unknown crate"

**错误信息**:
```
error: unknown crate `foreign_types`
```

**可能原因**:
1. `deps` 路径错误
2. 目标名称错误（不是 `:lib`）
3. BUILD.gn 文件不存在

**解决方案**:
```gn
# 检查 deps 路径
deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
#     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#     路径和目标名称必须正确
```

### 编译错误: "unresolved import"

**错误信息**:
```
error[E0433]: failed to resolve: use of undeclared crate or module `foreign_types`
```

**可能原因**:
1. `deps` 中未声明依赖
2. 版本不匹配

**解决方案**:
```gn
# 确保 deps 中声明了所有依赖
deps = [
  "//third_party/rust/crates/foreign-types:lib",
  "//third_party/rust/crates/foreign-types/foreign-types-shared:lib",
]
```

### 链接错误: "multiple definitions"

**错误信息**:
```
error: multiple definitions of symbol `foreign_types::...`
```

**可能原因**:
1. 重复链接
2. 多个 crate 输出相同符号

**解决方案**:
- 确保依赖树正确，避免重复链接

---

## 📌 总结

### OH 构建适配特点

1. **零源代码修改**: BUILD.gn 仅是构建配置，不影响源代码
2. **简单直接**: 无条件编译、无特殊选项、无复杂配置
3. **标准化**: 遵循 OH Rust crate 构建标准模板
4. **可维护**: 升级上游版本仅需更新版本号

### 关键配置项

| 配置项 | 必需 | 说明 |
|-------|------|------|
| `crate_name` | ✅ | crate 名称 |
| `crate_type` | ✅ | 输出类型（rlib） |
| `crate_root` | ✅ | 入口文件 |
| `cargo_pkg_version` | ✅ | 版本号（元数据） |
| `deps` | ⚠️ | 依赖（如有） |
| `part_name` | ✅ | OH 部件名称（主 crate） |
| `subsystem_name` | ✅ | OH 子系统名称（主 crate） |

### 最佳实践

1. **保持源代码纯净**: 不要修改上游源文件
2. **使用标准模板**: 参考其他 Rust crate 的 BUILD.gn
3. **及时更新版本**: 定期同步上游更新
4. **验证构建**: 升级后务必运行依赖方的测试
