# 02_Patches.md - Patch 详细分析

本文档详细分析 OpenHarmony 对 `foreign-types` 库的定制修改。

> **核心结论**: OH 未使用传统 Patch 文件，通过 Git commit 直接新增 4 个构建和合规文件，零源代码修改。

---

## 📋 Patch 形式说明

### Patch 文件 vs Git Commit

**传统 Patch 文件**（如 curl、openssl）:
- 使用 `*.patch` 文件存储修改
- 在编译前通过 `git apply` 或 `patch` 命令应用
- 适合大型、复杂的定制修改

**Git Commit 形式**（foreign-types）:
- 直接在 Git 历史中记录修改
- 无需额外的 Patch 文件
- 适合零侵入的构建适配

**foreign-types 采用 Git Commit 的原因**:
1. ✅ 仅新增文件，无源代码修改
2. ✅ 修改量小（仅 4 个文件，~103 行）
3. ✅ 便于追溯修改历史

---

## 📦 Patch 文件清单

### 概览表

| 文件路径 | 类型 | 行数 | 目的 | 关联的 OH 需求 |
|---------|------|------|------|--------------|
| `README.OpenSource` | 合规声明 | ~12 | OH 开源合规要求 | 合规性声明 |
| `bundle.json` | 部件配置 | ~34 | OH 组件化配置 | 组件管理系统 |
| `foreign-types/BUILD.gn` | 构建配置 | ~32 | GN 构建系统适配 | GN 构建系统 |
| `foreign-types-shared/BUILD.gn` | 构建配置 | ~28 | GN 构建系统适配 | GN 构建系统 |

**总计**: 4 个文件，~106 行代码

---

## 🔍 详细分析

### 1. README.OpenSource - OH 开源合规声明

**文件路径**: `README.OpenSource`

**文件内容**:
```json
[
  {
    "Name": "foreign-types",
    "License": "Apache-2.0",
    "License File": "LICENSE-APACHE",
    "Version Number": "0.3.2",
    "Owner": "xuelei3@huawei.com",
    "Upstream URL": "https://github.com/sfackler/foreign-types",
    "Description": "A framework for Rust wrappers over C APIs."
  }
]
```

**修改目的**:
- 满足 OpenHarmony 的开源合规要求
- 明确声明上游来源、许可证和版本
- 指定 OH 维护者

**OH 需求**: OpenHarmony 第三方库合规性管理

**关键信息**:
- **Name**: foreign-types
- **License**: Apache-2.0（注：上游为 MIT/Apache-2.0 双许可）
- **Version**: 0.3.2
- **Owner**: xuelei3@huawei.com
- **Upstream**: https://github.com/sfackler/foreign-types

---

### 2. bundle.json - OH 组件化配置

**文件路径**: `bundle.json`

**文件内容**:
```json
{
  "name": "@ohos/rust_foreign_types",
  "description": "A framework for Rust wrappers over C APIs",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/foreign-types"
  },
  "dirs": {},
  "scripts": {},
  "readmePath": {
    "en": "README.md"
  },
  "component": {
    "name": "rust_foreign_types",
    "subsystem": "thirdparty",
    "adapted_system_type": [
      "standard"
    ],
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/foreign-types:lib"
        }
      ],
      "test": []
    }
  }
}
```

**修改目的**:
- 将 foreign-types 注册为 OH 组件
- 定义组件的元数据和构建目标
- 支持组件化管理和依赖解析

**OH 需求**: OpenHarmony 组件化构建系统

**关键配置**:
| 配置项 | 值 | 说明 |
|-------|-----|------|
| `name` | @ohos/rust_foreign_types | OH 组件完整名称 |
| `version` | 6.1 | OH 组件版本（不同于 crate 版本 0.3.2） |
| `publishAs` | code-segment | 以代码段形式发布 |
| `subsystem` | thirdparty | 所属子系统 |
| `adapted_system_type` | standard | 适配标准系统 |
| `inner_kits` | //third_party/rust/crates/foreign-types:lib | 暴露的内部库 |

**OH 特有概念**:
- `code-segment`: OH 的发布方式，表示这是一个代码段而非独立应用
- `subsystem`: OH 的分层架构设计
- `inner_kits`: OH 组件暴露给其他模块的接口

---

### 3. foreign-types/BUILD.gn - 主 crate 构建配置

**文件路径**: `foreign-types/BUILD.gn`

**文件内容**:
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
  cargo_pkg_name = "foreign_types"
  cargo_pkg_description = "A framework for Rust wrappers over C APIs"
  deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
  module_output_extension = ".rlib"
  part_name = "rust_foreign_types"
  subsystem_name = "thirdparty"
}
```

**修改目的**:
- 将 Cargo 构建系统适配到 GN 构建系统
- 指定 Rust 编译选项和依赖关系
- 注册为 OH 部件和子系统的一部分

**OH 需求**: OpenHarmony GN 构建系统

**关键配置对比**:

| 配置项 | OH BUILD.gn | 上游 Cargo.toml | 差异 |
|-------|------------|-----------------|------|
| **构建系统** | GN (ohos_cargo_crate) | Cargo | 系统替换 |
| **crate_name** | `foreign_types` | - | GN 特有 |
| **crate_type** | `"rlib"` | - | GN 特有（静态库） |
| **edition** | `"2015"` | - | GN 特有（Rust edition） |
| **deps** | GN 路径 | Cargo 路径 | 路径格式差异 |
| **part_name** | `rust_foreign_types` | - | OH 特有 |
| **subsystem_name** | `thirdparty` | - | OH 特有 |

**与上游 Cargo.toml 的差异**:

**上游** (`foreign-types/Cargo.toml`):
```toml
[package]
name = "foreign-types"
version = "0.3.2"
authors = ["Steven Fackler <sfackler@gmail.com>"]
license = "MIT/Apache-2.0"
description = "A framework for Rust wrappers over C APIs"
repository = "https://github.com/sfackler/foreign-types"

[dependencies]
foreign-types-shared = { version = "0.1", path = "../foreign-types-shared" }
```

**OH BUILD.gn**:
- `cargo_pkg_version` 对应 `version`
- `cargo_pkg_authors` 对应 `authors`
- `cargo_pkg_name` 对应 `name`
- `cargo_pkg_description` 对应 `description`
- `deps` 路径格式：Cargo 用相对路径 `../foreign-types-shared`，GN 用绝对路径 `//third_party/rust/crates/...`

**依赖声明对比**:
```toml
# Cargo.toml (相对路径)
foreign-types-shared = { version = "0.1", path = "../foreign-types-shared" }
```

```gn
# BUILD.gn (GN 绝对路径)
deps = [ "//third_party/rust/crates/foreign-types/foreign-types-shared:lib" ]
```

**Copyright Header**:
- OH 特有：添加 Huawei Device Co., Ltd. 2023 版权声明
- 目的：OH 合规性要求

---

### 4. foreign-types-shared/BUILD.gn - Shared crate 构建配置

**文件路径**: `foreign-types-shared/BUILD.gn`

**文件内容**:
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

**修改目的**:
- 为 foreign-types-shared 提供 GN 构建配置
- foreign-types-shared 是 foreign-types 的内部依赖，无外部依赖

**OH 需求**: OpenHarmony GN 构建系统

**与主 crate 的差异**:
- **无 deps**: foreign-types-shared 是叶节点，无依赖
- **版本不同**: `0.1.1` vs `0.3.2`
- **描述不同**: "An internal crate used by foreign-types"

**注意**: 该 crate 无 `part_name` 和 `subsystem_name`，因为它是一个内部 crate，仅作为 foreign-types 的依赖。

---

## 📊 修改分类

### 按类型分类

| 类型 | 文件数量 | 说明 |
|------|---------|------|
| **合规文件** | 1 (README.OpenSource) | OH 开源合规声明 |
| **部件配置** | 1 (bundle.json) | OH 组件化配置 |
| **构建配置** | 2 (BUILD.gn) | GN 构建系统适配 |

### 按功能分类

| 功能 | 文件 | OH 需求 |
|------|------|---------|
| **开源合规** | README.OpenSource | 第三方库合规性管理 |
| **组件管理** | bundle.json | 组件化构建系统 |
| **构建适配** | foreign-types/BUILD.gn | GN 构建系统 |
| **依赖管理** | foreign-types-shared/BUILD.gn | GN 构建系统 |

---

## 🎯 源代码修改情况

### 结论：无源代码修改

**证据**:
1. ✅ 搜索未发现 `*.patch` 文件
2. ✅ Git 历史显示仅新增文件，未修改源文件
3. ✅ 所有 `*.rs` 文件与上游一致

**受影响的文件**:
| 文件类型 | 修改数量 | 说明 |
|---------|---------|------|
| **新增文件** | 4 | README.OpenSource, bundle.json, 2×BUILD.gn |
| **修改文件** | 0 | 无 |
| **删除文件** | 0 | 无 |
| **Rust 源文件** | 0 | 无修改 |

---

## 📈 修改历史

### Git 提交记录

| Commit Hash | 描述 | 作者 | 日期 |
|------------|------|------|------|
| `dbfb254` | foreign-types新增bundle.json部件化 | OH 开发者 | 未知 |
| `463289a` | README.OpenSource License整改 | 龙剑吟 | 未知 |
| `b58e1e8` | Add GN Build Files and Custom Modifications | peizhe | 未知 |

### 修改时间线

```
[上游版本 0.3.2] (commit 553b6d7)
        ↓
[OH 适配开始]
        ↓
1. b58e1e8: 添加 GN Build Files 和自定义修改
        ↓
2. 463289a: README.OpenSource License 整改
        ↓
3. dbfb254: 新增 bundle.json 部件化
        ↓
[当前 OH 版本]
```

---

## 🔧 升级建议

### 升级上游版本流程

由于 OH 仅新增构建文件，无源代码修改，升级流程非常简单：

#### 步骤 1: 同步上游代码
```bash
# 在 foreign-types 目录
git remote add upstream https://github.com/sfackler/foreign-types.git
git fetch upstream
git checkout <new-version>
```

#### 步骤 2: 更新 BUILD.gn 版本号

**foreign-types/BUILD.gn**:
```gn
# 修改前
cargo_pkg_version = "0.3.2"

# 修改后
cargo_pkg_version = "<new-version>"
```

**foreign-types-shared/BUILD.gn**:
```gn
# 修改前
cargo_pkg_version = "0.1.1"

# 修改后
cargo_pkg_version = "<new-shared-version>"
```

#### 步骤 3: 更新 README.OpenSource
```json
{
  "Version Number": "<new-version>"
}
```

#### 步骤 4: 验证构建
```bash
# 在 OH 根目录
./build.sh --product-name <product> --build-target foreign_types
```

### 回归风险评估

| 风险类型 | 风险等级 | 说明 |
|---------|---------|------|
| **API 兼容性** | 🟢 低 | foreign-types 是基础框架，API 稳定 |
| **构建系统** | 🟢 低 | 仅需更新版本号，无 OH 特有逻辑 |
| **依赖关系** | 🟢 低 | rust-openssl 使用稳定版本 |
| **测试覆盖** | 🟡 中 | 需运行 rust-openssl 的测试套件 |

### 升级前检查清单

- [ ] 查看 upstream GitHub release notes
- [ ] 检查是否有 breaking changes
- [ ] 更新 BUILD.gn 版本号
- [ ] 更新 README.OpenSource 版本号
- [ ] 验证构建成功
- [ ] 运行依赖方（rust-openssl）的测试
- [ ] 更新 bundle.json 版本号（可选）

---

## 📌 总结

### 关键发现

1. **零侵入**: OH 没有修改任何 Rust 源代码
2. **构建适配**: 仅添加 GN 构建配置文件
3. **合规声明**: 添加 README.OpenSource 满足开源合规要求
4. **组件化**: 通过 bundle.json 集成到 OH 组件系统

### 维护建议

1. **升级简单**: 仅需更新版本号，无 OH 特有逻辑
2. **低风险**: 源代码未修改，回归风险低
3. **持续同步**: 建议定期检查上游更新，及时升级

### Patch 分类

| Patch | 分类 | 可推向上游？ |
|-------|------|------------|
| README.OpenSource | OH 特有 | ❌ |
| bundle.json | OH 特有 | ❌ |
| BUILD.gn | OH 特有 | ❌ |

**说明**: 所有修改均为 OH 特有，不适合推向上游。
