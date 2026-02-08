# OpenHarmony 构建适配

## BUILD.gn 配置详解

### 配置文件结构

memchr 库的 OpenHarmony 构建配置位于 `//third_party/rust/crates/memchr/BUILD.gn`，使用 ohos_cargo_crate 模板进行集成。以下是完整的配置内容和详细说明：

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
    crate_name = "memchr"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "2.5.0"
    cargo_pkg_authors = "Andrew Gallant <jamslam@gmail.com>,  bluss"
    cargo_pkg_name = "memchr"
    cargo_pkg_description = "Safe interface to memchr."
    features = ["std"]
    build_root = "build.rs"
    build_sources = ["build.rs"]
    module_output_extension = ".rlib"
    part_name = "rust_memchr"
    subsystem_name = "thirdparty"
}
```

### 配置项逐项说明

#### 模板类型

```gn
ohos_cargo_crate("lib") { ... }
```

该配置使用 `ohos_cargo_crate` 模板，这是 OpenHarmony 为 Rust crates 提供的标准构建模板。该模板封装了 Rust crates 在 OHOS 环境下的构建逻辑，包括依赖解析、特性处理和输出产物管理。

#### crate 标识配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| crate_name | "memchr" | crate 在 Rust 代码中的名称 |
| cargo_pkg_name | "memchr" | 与 Cargo.toml 中 name 字段对应 |
| cargo_pkg_version | "2.5.0" | 与 Cargo.toml 中 version 字段对应 |
| cargo_pkg_authors | "Andrew Gallant <jamslam@gmail.com>, bluss" | 维护者信息 |
| cargo_pkg_description | "Safe interface to memchr." | 简短描述 |

#### 构建类型配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| crate_type | "rlib" | 生成 Rust 静态库，适用于静态链接 |
| crate_root | "src/lib.rs" | crate 的主入口文件 |
| edition | "2018" | Rust edition 版本 |
| sources | ["src/lib.rs"] | 源文件列表 |

crate_type 设置为 "rlib" 是 Rust crates 的标准静态库格式。这种格式的库可以被其他 Rust crate 静态链接，是 OpenHarmony 中 Rust 组件的标准分发形式。

#### 特性配置

```gn
features = ["std"]
```

该配置启用了 `std` 特性，这是 memchr 库的关键特性：

- **std 特性作用**：允许库使用 Rust 标准库，主要用于运行时 CPU 特性检测
- **性能影响**：启用后，库可以检测 CPU 是否支持 AVX/AVX2 等高级 SIMD 指令集，并自动选择最优的搜索实现
- **备选方案**：如果不启用 std 特性，库将回退到使用 SSE2（在 x86_64 上）或通用实现

#### 构建脚本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| build_root | "build.rs" | 构建脚本路径 |
| build_sources | ["build.rs"] | 构建脚本源文件 |

build.rs 是 Rust Cargo 的标准构建脚本机制。memchr 的构建脚本主要用于：

- 检测目标平台的 SIMD 能力
- 根据平台条件配置编译选项
- 生成必要的配置代码

#### 输出配置

```gn
module_output_extension = ".rlib"
```

指定输出产物的文件扩展名为 `.rlib`，这是 Rust 静态库的标准扩展名。

#### OpenHarmony 组织配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| part_name | "rust_memchr" | 部件名称 |
| subsystem_name | "thirdparty" | 子系统名称 |

这些配置将 memchr 库组织在 OpenHarmony 的部件和子系统架构中：
- **subsystem_name**: "thirdparty" 表示该库属于第三方组件子系统
- **part_name**: "rust_memchr" 是该库在构建系统中的唯一标识

## 与上游构建系统的对比

### Cargo.toml 配置

上游项目的 Cargo.toml 相关配置如下：

```toml
[package]
name = "memchr"
version = "2.5.0"
authors = ["Andrew Gallant <jamslam@gmail.com>", "bluss"]
edition = "2018"

[lib]
bench = false

[features]
default = ["std"]
std = []

[dependencies]
libc = { version = "0.2.18", default-features = false, optional = true }
```

### 差异对比表

| 配置维度 | 上游 Cargo.toml | OH BUILD.gn | 差异说明 |
|----------|-----------------|-------------|----------|
| crate 名称 | name = "memchr" | crate_name = "memchr" | 一致 |
| 版本 | version = "2.5.0" | cargo_pkg_version = "2.5.0" | 一致 |
| edition | edition = "2018" | edition = "2018" | 一致 |
| crate 类型 | 默认 rlib | crate_type = "rlib" | 一致 |
| 特性 | default = ["std"] | features = ["std"] | 一致 |
| 构建脚本 | build.rs | build_sources = ["build.rs"] | 功能一致，形式不同 |
| 依赖 | libc（可选） | 未指定 | OH 未启用 libc 特性 |

### 主要差异分析

1. **依赖配置差异**
   - 上游：Cargo.toml 中定义了 libc 可选依赖
   - OH：BUILD.gn 中未显式指定依赖
   - 说明：OH 环境未启用 libc 特性，memchr 使用自身的搜索实现

2. **构建系统映射**
   - Cargo.toml 的 `[features]` 映射到 BUILD.gn 的 `features` 字段
   - Cargo.toml 的 `[package]` 字段映射到 BUILD.gn 的 `cargo_pkg_*` 字段
   - Cargo.toml 的 `[lib]` 配置由 ohos_crate 模板自动处理

## 编译选项说明

### 启用的编译选项

当前配置启用的编译选项及其影响：

| 选项 | 来源 | 影响 |
|------|------|------|
| std 特性 | BUILD.gn: features | 启用标准库，获得 CPU 特性检测能力 |
| edition 2018 | BUILD.gn: edition | 使用 Rust 2018 edition 语法 |

### 隐式编译行为

基于当前配置，以下行为是隐式启用的：

1. **SIMD 自动检测**
   - 运行时检测 CPU 指令集支持
   - 自动选择最优 SIMD 实现（SSE2、AVX、AVX2、AVX512）

2. **平台自适应**
   - 根据目标平台选择合适的代码路径
   - 在非 x86_64 平台上使用通用实现

3. **发布构建优化**
   - 基于 Cargo.toml 中的 release profile 配置
   - 启用 LTO 和 codegen-unit 优化

## 产物说明

### 输出产物

构建完成后，memchr 库生成以下产物：

| 产物类型 | 文件名 | 说明 |
|----------|--------|------|
| 静态库 | libmemchr.rlib | Rust 静态库文件 |
| 元数据 | - | 由 ohos_crate 模板管理 |

### 产物使用方式

其他 OH 模块通过以下方式使用该库：

```gn
deps = ["//third_party/rust/crates/memchr:lib"]
```

该依赖路径指向 BUILD.gn 中定义的目标 `"lib"`。

## 特殊处理说明

### 无特殊处理

memchr 库在 OpenHarmony 中的构建集成不涉及以下特殊处理：

- **未禁用任何上游特性**：所有上游默认启用的特性在 OH 中保持启用
- **未添加 OH 特定源文件**：完全使用上游源代码
- **未修改编译标志**：使用上游默认的优化级别
- **未添加条件编译**：源代码中无 `#ifdef OHOS` 或类似宏

### 构建脚本行为

build.rs 在 OH 构建环境中的行为：

- **正常工作**：build.rs 中的平台检测逻辑在 OH 环境下正常执行
- **无特殊路径**：无需针对 OH 环境进行特殊配置
- **SIMD 能力检测**：正确识别目标平台的 SIMD 支持情况

## 故障排除

### 常见构建问题

1. **SIMD 编译错误**
   - 症状：编译时出现 "unavailable target feature" 错误
   - 原因：目标平台不支持请求的 SIMD 指令集
   - 解决方案：检查目标设备的 CPU 特性支持情况

2. **特性冲突**
   - 症状：构建时报告特性冲突
   - 原因：依赖链中有模块禁用了 std 特性
   - 解决方案：确保所有依赖保持特性一致性

### 性能相关问题

1. **未达到预期性能**
   - 检查：确认 std 特性已启用
   - 检查：确认目标平台支持 SIMD
   - 检查：确认构建的是 release 版本

## 相关文档

- [01_Overview.md](./01_Overview.md) - 库功能概述
- [02_Patches.md](./02_Patches.md) - Patch 分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景和依赖关系
- [ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整评估报告
