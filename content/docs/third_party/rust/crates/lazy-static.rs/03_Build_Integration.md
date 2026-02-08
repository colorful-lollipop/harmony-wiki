# OpenHarmony 构建适配

## 3.1 构建系统概述

OpenHarmony 使用 GN（Generate Ninja）作为主要构建系统，并通过专门设计的 `ohos_cargo_crate` 模板来构建 Rust crates。这种设计使得 Rust 库可以无缝集成到 OpenHarmony 的整体构建流程中，同时保持与上游 Cargo 项目的兼容性。

lazy_static 的构建配置简洁明了，展示了 OpenHarmony 对成熟 Rust 库的标准化集成方式。

## 3.2 BUILD.gn 配置详解

### 3.2.1 完整配置

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
    crate_name = "lazy_static"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "1.4.0"
    cargo_pkg_authors = "Marvin Löbel <loebel.marvin@gmail.com>"
    cargo_pkg_name = "lazy_static"
    cargo_pkg_description = "A macro for declaring lazily evaluated statics in Rust."
    module_output_extension = ".rlib"
    part_name = "rust_lazy_static"
    subsystem_name = "thirdparty"
}
```

### 3.2.2 配置字段说明

**模板调用**：

`ohos_cargo_crate("lib")`：调用 OpenHarmony 提供的 Rust crates 构建模板，创建一个名为 "lib" 的构建目标。

**基本配置**：

| 字段 | 值 | 说明 |
|------|---|------|
| crate_name | "lazy_static" | 库内部名称，用于构建系统标识 |
| crate_type | "rlib" | Rust 库类型，rlib 是静态库格式 |
| crate_root | "src/lib.rs" | 库的主入口文件路径 |

**源文件配置**：

| 字段 | 值 | 说明 |
|------|---|------|
| sources | ["src/lib.rs"] | 需要编译的源文件列表 |
| edition | "2015" | Rust Edition 版本 |

**包元数据（从 Cargo.toml 映射）**：

| 字段 | 值 | 说明 |
|------|---|------|
| cargo_pkg_version | "1.4.0" | 版本号，与上游保持一致 |
| cargo_pkg_authors | "Marvin Löbel <loebel.marvin@gmail.com>" | 作者信息 |
| cargo_pkg_name | "lazy_static" | Cargo 包名称 |
| cargo_pkg_description | "A macro for declaring lazily evaluated statics in Rust." | 包描述 |

**输出配置**：

| 字段 | 值 | 说明 |
|------|---|------|
| module_output_extension | ".rlib" | 输出文件扩展名 |
| part_name | "rust_lazy_static" | OpenHarmony 部件名称 |
| subsystem_name | "thirdparty" | 所属子系统 |

### 3.2.3 关键配置分析

**edition = "2015"**：选择 Rust 2015 Edition 是因为 lazy_static 1.4.0 版本发布时，Rust 2018 Edition 尚未完全普及。这一选择确保了与原始代码的兼容性。

**crate_type = "rlib"**：rlib 是 Rust crates 的标准静态库格式，适合编译时链接。与 dylib（动态库）或 cdylib（C 动态库）相比，rlib 具有更简单的依赖管理和更快的链接速度。

## 3.3 与上游构建系统的对比

### 3.3.1 上游构建配置（ Cargo.toml ）

```toml
[package]
name = "lazy_static"
version = "1.4.0"
authors = ["Marvin Löbel <loebel.marvin@gmail.com>"]
license = "MIT/Apache-2.0"

[dependencies.spin]
version = "0.5.0"
optional = true

[features]
spin_no_std = ["spin"]
```

### 3.3.2 配置映射关系

| Cargo.toml 字段 | BUILD.gn 字段 | 映射说明 |
|----------------|--------------|---------|
| name | cargo_pkg_name | 直接映射 |
| version | cargo_pkg_version | 直接映射 |
| authors | cargo_pkg_authors | 直接映射 |
| description | cargo_pkg_description | 直接映射 |
| edition | edition | 直接映射 |
| - | crate_root | 根据 Cargo 规范确定 |
| - | crate_type | 根据库类型确定 |

### 3.3.3 未使用的配置项

BUILD.gn 中未显式配置以下内容，但它们由模板自动处理：

**依赖管理**：`ohos_cargo_crate` 模板自动解析 Cargo.toml 中的依赖项（spin），并处理依赖关系链。

**Features**：虽然上游支持 `spin_no_std` feature，但 OpenHarmony 构建中未启用，使用默认的标准库实现。

**目标过滤**：未指定特定的 target 过滤，lazy_static 将为所有 OpenHarmony 支持的目标架构编译。

## 3.4 构建流程详解

### 3.4.1 构建步骤

当执行 `hb build` 或 `bazel build` 命令时，lazy_static 的构建流程如下：

**步骤一：配置解析**

构建系统读取 BUILD.gn 文件，解析 ohos_cargo_crate 模板参数，生成对应的 ninja 构建规则。

**步骤二：依赖分析**

系统分析 Cargo.toml 中的依赖项（spin 0.5.0，可选依赖），确定是否需要拉取和编译。

**步骤三：Cargo 集成**

构建系统调用 cargo 命令执行实际的 Rust 编译，生成 .rlib 静态库文件。

**步骤四：输出归档**

编译产物被复制到指定的输出目录，供依赖该库的模块链接使用。

### 3.4.2 构建产物

构建完成后，会生成以下文件：

| 文件 | 路径 | 说明 |
|-----|------|-----|
| liblazy_static.rlib | out/.../obj/third_party/rust/crates/lazy-static.rs/ | 主库文件 |
| liblazy_static.so | out/.../obj/third_party/rust/crates/lazy-static.rs/ | 动态库版本（可选） |

## 3.5 特殊处理说明

### 3.5.1 无特殊处理

与其他需要特殊配置的第三方库不同，lazy_static 在 OpenHarmony 中没有进行任何特殊处理：

- **无自定义 defines**：未添加任何编译时宏定义
- **无特殊 flags**：未使用自定义的编译选项
- **无条件编译**：未针对 OpenHarmony 进行代码条件化
- **无后处理脚本**：未添加任何构建后处理步骤

### 3.5.2 配置简洁性的原因

这种配置简洁性源于以下因素：

**第一，库的功能通用**。lazy_static 提供的功能是 Rust 语言层面的抽象，不涉及操作系统相关的逻辑。

**第二，Rust 的跨平台设计**。Rust 标准库和 crates 生态强调跨平台兼容性，代码通常无需修改即可在不同平台上编译。

**第三，构建模板的成熟度**。`ohos_cargo_crate` 模板已经能够正确处理大多数 Rust crates 的构建需求。

## 3.6 版本升级注意事项

### 3.6.1 升级时的配置检查清单

当需要升级 lazy_static 版本时，需要检查以下配置项：

```
□ cargo_pkg_version：更新为新版本号
□ sources：如果新增源文件，需要添加到列表
□ edition：如果新版本要求更新的 Edition，需要同步更新
□ 依赖项版本：检查 Cargo.toml 中依赖的版本变化
```

### 3.6.2 常见的潜在问题

**问题一：Edition 不兼容**

如果上游升级到更新的 Rust Edition（如 2021），需要同步更新 BUILD.gn 中的 edition 字段。同时需要确保 OpenHarmony 的 Rust 工具链支持该 Edition。

**问题二：依赖冲突**

新版本可能引入新的依赖或更新现有依赖版本。需要检查这些依赖是否与 OpenHarmony 中的其他 Rust 组件兼容。

**问题三：宏变更**

如果新版本修改了 lazy_static! 宏的行为或语法，可能影响依赖该库的所有模块的代码。

## 3.7 与其他 Rust crates 的构建对比

| 库名称 | BUILD.gn 复杂度 | 特殊处理 | Patch 数量 |
|-------|----------------|---------|----------|
| lazy_static | 低 | 无 | 0 |
| bindgen | 中 | clang 依赖配置 | 0 |
| proc-macro2 | 低 | 无 | 0 |
| quote | 低 | 无 | 0 |
| syn | 中 | feature 配置 | 0 |

## 3.8 小结

lazy_static 在 OpenHarmony 中的构建适配体现了标准化和简洁化的设计原则。BUILD.gn 配置清晰、文档完善，展示了 OpenHarmony 对成熟 Rust crates 的良好支持。

由于该库无需任何特殊配置或 Patch，构建适配工作主要关注版本升级时的兼容性检查。建议维护者定期关注上游版本更新，并在升级前进行充分的测试验证。
