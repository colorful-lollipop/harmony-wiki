# OH 构建适配

## BUILD.gn 配置

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
  crate_name = "proc_macro2"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2021"
  cargo_pkg_version = "1.0.92"
  cargo_pkg_authors =
      "David Tolnay <dtolnay@gmail.com>,  Alex Crichton <alex@alexcrichton.com>"
  cargo_pkg_name = "proc-macro2"
  cargo_pkg_description = "A substitute implementation of the compiler's `proc_macro` API to decouple token-based libraries from the procedural macro use case."
  deps = [ "//third_party/rust/crates/unicode-ident:lib" ]
  features = [
    "proc-macro",
    "span-locations",
  ]
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  module_output_extension = ".rlib"
  part_name = "rust_proc-macro2"
  subsystem_name = "thirdparty"
}
```

## 关键配置项说明

### 1. 构建模板

```gn
ohos_cargo_crate("lib")
```

使用 OH 的 Rust crates 标准构建模板 `ohos_crate_crate`。

### 2. crate 类型

```gn
crate_type = "rlib"
```

生成 Rust 静态库（.rlib），用于静态链接到其他 Rust crates。

### 3. Rust Edition

```gn
edition = "2021"
```

使用 Rust 2021 Edition，与上游版本一致。

### 4. 依赖配置

```gn
deps = [ "//third_party/rust/crates/unicode-ident:lib" ]
```

唯一依赖是 `unicode-ident`，用于生成 Unicode 标识符。

### 5. Features

```gn
features = [
  "proc-macro",      # 启用过程宏 API
  "span-locations",  # 启用位置信息
]
```

启用的 features 均为上游标准配置，用于提供完整的 Token 处理能力。

## 构建配置详解

### build.rs

OH 使用上游原生的 `build.rs`：

```gn
build_root = "build.rs"
build_sources = [ "build.rs" ]
```

`build.rs` 的主要功能：

1. **Unicode 标识符检查**: 验证目标 Rust 版本支持 Unicode 标识符
2. **配置检测**: 检测目标平台的 Rust 编译器功能
3. **Cargo 配置**: 设置 Cargo.toml 的 patch 配置

### module_output_extension

```gn
module_output_extension = ".rlib"
```

指定输出文件扩展名为 `.rlib`（Rust 静态库）。

## 与上游构建差异

| 配置项 | 上游 | OH | 差异说明 |
|--------|------|-----|----------|
| 构建系统 | cargo | ohos_cargo_crate | OH Rust 标准模板 |
| crate_type | rlib/dylib | rlib | 仅使用静态库 |
| edition | 2021 | 2021 | 一致 |
| features | proc-macro, default | proc-macro, span-locations | OH 额外启用 span-locations |

**注意**: `span-locations` feature 的启用是因为 OH 的某些下游库（如 cxx）需要位置信息用于代码诊断。

## 构建产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libproc_macro2.rlib | out/.../libproc_macro2.rlib | Rust 静态库 |
| libproc_macro2.so | out/.../libproc_macro2.so | 动态库（如果需要） |

## 版本管理

### OH 版本号映射

| 上游版本 | OH 版本 | 说明 |
|----------|---------|------|
| 1.0.92 | 5.0 | 当前版本 |
| 1.0.91 | 4.0 | 上一版本 |

OH 版本号采用独立的版本管理方案，与上游版本解耦。

### 版本升级流程

1. 更新 `cargo_pkg_version` 为新版本号
2. 更新 `part_name` 中的版本号（如需要）
3. 运行 `cargo update` 更新 Cargo.lock
4. 执行 OH 构建验证
5. 更新本文档的版本信息
