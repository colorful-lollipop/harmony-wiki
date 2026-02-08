# OH 构建适配

## 构建配置概述

rustc-hash 在 OpenHarmony 中使用 `ohos_cargo_crate` 模板进行构建。构建配置与上游 Cargo.toml 高度一致，仅进行了必要的 OH 平台适配。

## BUILD.gn 文件详解

**文件路径**：`third_party/rust/crates/rustc-hash/BUILD.gn`

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
    crate_name = "rustc_hash"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "1.1.0"
    cargo_pkg_authors = "The Rust Project Developers"
    cargo_pkg_name = "rustc-hash"
    cargo_pkg_description = "speed, non-cryptographic hash used in rustc"
    features = ["std"]
    module_output_extension = ".rlib"
    part_name = "rust_rustc_hash"
    subsystem_name = "thirdparty"
}
```

## 关键配置参数解析

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | rustc_hash | Rust crate 内部名称（下划线格式） |
| `crate_type` | rlib | 生成 Rust 静态库文件 |
| `crate_root` | src/lib.rs | 库入口文件路径 |
| `edition` | 2015 | Rust 版本（较旧的 2015 edition） |
| `cargo_pkg_version` | 1.1.0 | 对应上游 crates.io 版本 |
| `features` | ["std"] | 启用的特性列表 |
| `part_name` | rust_rustc_hash | OH 部件名称 |
| `subsystem_name` | thirdparty | 所属子系统 |

## 与上游构建系统的差异

### 配置对比

| 配置项 | 上游 (Cargo.toml) | OH (BUILD.gn) |
|--------|-------------------|---------------|
| 版本 | 1.1.0 | 1.1.0 |
| Edition | 2015 | 2015 |
| Features | std (default) | std |
| License | Apache-2.0/MIT | Apache-2.0/MIT |

### 差异分析

**结论**：OH 的 BUILD.gn 配置与上游 Cargo.toml 几乎完全一致，无任何 OH 特定的定制配置。

具体差异说明：

1. **无额外 defines**：未添加任何编译宏定义
2. **无额外 configs**：未添加任何构建配置
3. **无额外 deps**：无额外的依赖项
4. **无额外 sources**：源文件列表与上游一致
5. **无额外 flags**：无额外的编译器标志

## 特性配置详情

### std 特性

```gn
features = ["std"]
```

该配置启用标准库支持，提供以下功能：

- `FxHashMap<K, V>` 类型别名
- `FxHashSet<V>` 类型别名
- `std::collections` 相关导入

### no_std 模式

如需在 `no_std` 环境中使用，可修改配置为：

```gn
features = []  # 不启用任何特性
```

但当前 OH 配置中启用了 std 特性，以支持 FxHashMap 和 FxHashSet 类型别名。

## 编译产物

### 输出文件

| 文件类型 | 扩展名 | 说明 |
|----------|--------|------|
| 静态库 | .rlib | Rust 静态库文件 |

### 库依赖方式

该库以**静态链接**方式被依赖者使用：
- 依赖者将 rustc-hash 编译为 `.rlib` 文件
- 在链接阶段将 rustc-hash 的代码静态链接到依赖者的最终产物中

## 部件注册信息

根据 bundle.json 配置：

```json
{
  "component": {
    "name": "rust_rustc_hash",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "build": {
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/rustc-hash:lib"
        }
      ]
    }
  }
}
```

### 部件信息

| 属性 | 值 |
|------|-----|
| 部件名称 | rust_rustc_hash |
| 子系统 | thirdparty |
| 适配系统 | standard |
| 内部kit | //third_party/rust/crates/rustc-hash:lib |

## 构建注意事项

### Rust Edition 升级建议

该库当前使用 **Rust 2015 edition**，这是一个相对较旧的版本。建议在后续升级中考虑：

1. **升级至 2018 edition**：获得更好的模块系统和宏支持
2. **升级至 2021 edition**：获得更现代的语言特性

升级时需要确保：
- OH 构建工具链支持新的 edition
- 依赖者（bindgen）兼容新的 edition
- 相关的 Clippy 配置更新

### 验证构建

可通过以下命令验证构建：

```bash
# 使用 OH 构建系统
hb build -p rust_rustc_hash

# 或直接验证 crate 编译
cargo check --manifest-path third_party/rust/crates/rustc-hash/Cargo.toml
```

## 相关文档

- [库概述](./01_Overview.md)
- [OH 使用情况](./04_Usage_in_OH.md)
- [安全风险分析](./06_Security.md)
