# OH 构建适配

## 概述

once_cell 在 OpenHarmony 中的构建适配属于**最简单的集成模式**：

- ✅ **零源代码修改**: 仅添加构建系统配置文件
- ✅ **使用标准模板**: 通过 `ohos_cargo_crate` GN 模板编译
- ✅ **无特殊选项**: 仅使用标准特性，无 OH 特定编译标志

**构建系统对比**:
| 项目 | 上游 (Cargo) | OH (GN/Ninja) |
|------|--------------|--------------|
| **构建工具** | `cargo build` | `ninja` |
| **构建描述** | `Cargo.toml` | `BUILD.gn` + `bundle.json` |
| **依赖管理** | `Cargo.toml` [dependencies] | GN `deps` 字段 |
| **特性控制** | `cargo build --features xxx` | BUILD.gn `features` 列表 |
| **输出类型** | 自动选择 | 强制 `rlib` |

---

## BUILD.gn 结构说明

### 文件路径
`third_party/rust/crates/once_cell/BUILD.gn`

### 完整内容

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
    crate_name = "once_cell"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "1.17.0"
    cargo_pkg_authors = "Aleksey Kladov <aleksey.kladov@gmail.com>"
    cargo_pkg_name = "once_cell"
    cargo_pkg_description = "Single assignment cells and lazy values."
    features = [
        "alloc",
        "race",
        "std",
    ]
    module_output_extension = ".rlib"
    part_name = "rust_once_cell"
    subsystem_name = "thirdparty"
}
```

### 字段详解

#### 1. 版权声明
```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0...
```
- 标准 OH 第三方库版权声明
- Apache License 2.0

#### 2. 导入 OH GN 规则
```gn
import("//build/ohos.gni")
```
- 导入 OH 的基础 GN 规则
- 提供 `ohos_cargo_crate` 模板

#### 3. `ohos_cargo_crate` 目标
```gn
ohos_cargo_crate("lib") {
    ...
}
```
- OH 提供的 Rust crate 编译模板
- 内部调用 `rustc` 编译器
- 处理依赖解析和链接

#### 4. Crate 元数据

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | once_cell | Crate 名称（不带版本） |
| `crate_type` | rlib | 输出类型：Rust 静态库 |
| `crate_root` | src/lib.rs | 源代码入口文件 |
| `sources` | ["src/lib.rs"] | 源文件列表 |
| `edition` | 2021 | Rust Edition 版本 |
| `cargo_pkg_version` | 1.17.0 | Cargo.toml 版本号 |
| `cargo_pkg_authors` | Aleksey Kladov | 作者信息 |
| `cargo_pkg_name` | once_cell | Cargo 包名 |
| `cargo_pkg_description` | Single assignment cells... | 包描述 |

#### 5. 特性配置
```gn
features = [
    "alloc",  # ✅ 启用: 提供堆内存分配支持
    "race",   # ✅ 启用: 提供 first one wins 初始化模式
    "std",    # ✅ 启用: 提供标准库集成
]
```

**特性说明**:

| 特性 | 启用原因 | 功能 | 对应的模块 |
|------|----------|------|-----------|
| **std** | OH 使用标准库 | 提供 `std` API 支持 | `sync::OnceCell`, `sync::Lazy` |
| **alloc** | `std` 的依赖 | 提供堆内存分配 | 所有模块 |
| **race** | 多线程场景 | first one wins 初始化 | `race::OnceBox`, `race::OnceRef` |

**未启用的特性**:
- ❌ `parking_lot`: 使用 parking_lot 优化内存大小（可选性能优化）
- ❌ `critical-section`: no_std 平台的 critical-section 实现（嵌入式场景）

**选择策略**:
- OH 使用标准功能集，避免引入额外依赖
- `parking_lot` 功能对 OH 价值有限（OH 不追求极致内存优化）
- `critical-section` 是嵌入式场景，OH 标准系统不需要

#### 6. OH 组件信息
```gn
module_output_extension = ".rlib"
part_name = "rust_once_cell"
subsystem_name = "thirdparty"
```

| 字段 | 值 | 说明 |
|------|-----|------|
| `module_output_extension` | .rlib | 输出文件扩展名：Rust 静态库 |
| `part_name` | rust_once_cell | OH 部件名称 |
| `subsystem_name` | thirdparty | 所属子系统 |

---

## 关键编译选项

### 默认编译选项

once_cell 使用 `ohos_cargo_crate` 模板的默认编译选项：

| 选项 | 值 | 来源 | 说明 |
|------|-----|------|------|
| `--edition` | 2021 | BUILD.gn | Rust Edition |
| `--crate-type` | rlib | BUILD.gn | 静态库输出 |
| `--opt-level` | 2 (Release) / 0 (Debug) | 模板 | 优化级别 |
| `--debug` | true/false | 模板 | Debug 信息 |

### OH 特定编译选项

**无额外编译选项**: once_cell 的 BUILD.gn **不包含** OH 特定的编译标志：

```gn
# ❌ 不存在以下选项：
#   rustflags = [
#       "--cfg=ohos",
#       "--target=...",
#   ]
```

**原因**:
- once_cell 是纯 Rust 库，不依赖平台特定 API
- 标准库 API 在 OH 环境中完全可用
- 无需条件编译或平台适配

### 与上游构建系统的差异

| 项目 | Cargo (上游) | GN (OH) | 差异说明 |
|------|--------------|-----------|----------|
| **构建命令** | `cargo build` | `ninja` | GN 生成 ninja 文件 |
| **依赖管理** | `Cargo.toml` [dependencies] | GN `deps` | GN 显式声明依赖 |
| **工作空间** | `Cargo.toml` [workspace] | GN `:` 引用 | GN 通过路径引用 |
| **特性激活** | `--features` 命令行 | `features` 列表 | GN 在配置中指定 |
| **构建模式** | `--release` / `--debug` | GN 模板选择 | OH 通过 GN 参数控制 |
| **目标平台** | `--target` | `target_cpu` / `target_os` | GN 环境变量 |

---

## 特殊处理

### 禁用的特性

#### 1. parking_lot 特性

**未启用原因**:
```gn
# ❌ 未添加到 features 列表
# "parking_lot",
```

**parking_lot 的作用**:
- 使用 `parking_lot_core` 替代 `std::sync::Once`
- 每个 `OnceCell<T>` 实例节省约 16 字节内存
- 性能相当，但内存占用更小

**OH 不启用的原因**:
1. **优化价值有限**: OH 主要追求稳定性和安全性，非极致内存优化
2. **增加依赖**: 需要额外依赖 `parking_lot_core`
3. **兼容性**: 标准 `std::sync::Once` 已经满足 OH 的所有场景

**何时考虑启用**:
- 如果 OH 的内存敏感场景（如嵌入式设备）有性能瓶颈
- 需要通过性能测试验证收益

#### 2. critical-section 特性

**未启用原因**:
```gn
# ❌ 未添加到 features 列表
# "critical-section",
```

**critical-section 的作用**:
- 在 `no_std` 环境下提供类似 `std::sync::Once` 的功能
- 用于嵌入式场景

**OH 不启用的原因**:
1. **标准系统**: OH 标准系统提供完整的 `std`
2. **不适用**: `critical-section` 是嵌入式场景，OH 标准系统不需要
3. **限制**: `critical-section` 会影响性能

### 添加 OH 特定源文件

**无 OH 特定源文件**: once_cell **不包含** OH 特定的源代码

```
src/
├── lib.rs          # 与上游一致
├── imp_pl.rs       # parking_lot 实现（未启用）
├── imp_std.rs      # 标准库实现（使用中）
└── imp_cs.rs       # critical-section 实现（未启用）
```

**说明**:
- 所有源代码与上游 1.17.0 版本完全一致
- OH 不添加、修改或删除任何源文件
- 实现选择由 `features` 编译条件控制

---

## 与上游构建系统的差异

### 文件对比

| 文件 | 上游 | OH | 说明 |
|------|------|-----|------|
| **Cargo.toml** | ✅ 存在 | ✅ 存在（相同） | 用于元数据，不参与 OH 构建 |
| **BUILD.gn** | ❌ 不存在 | ✅ 存在 | OH 专用的 GN 构建脚本 |
| **bundle.json** | ❌ 不存在 | ✅ 存在 | OH 组件元数据 |
| **OAT.xml** | ❌ 不存在 | ✅ 存在 | OSS 审计配置 |
| **README.OpenSource** | ❌ 不存在 | ✅ 存在 | OH 开源声明 |

### 编译流程对比

#### 上游编译流程
```
1. 运行 cargo build
   ↓
2. Cargo 解析 Cargo.toml
   ↓
3. Cargo 调用 rustc 编译 src/lib.rs
   ↓
4. 生成 target/rlib/libonce_cell-xxxxx.rlib
```

#### OH 编译流程
```
1. GN 解析 BUILD.gn
   ↓
2. GN 调用 ohos_cargo_crate 模板
   ↓
3. 模板调用 rustc 编译 src/lib.rs
   ↓
4. 生成 out/.../libonce_cell.rlib
   ↓
5. Ninja 负责依赖管理和并行编译
```

**关键差异**:
- OH 使用 GN 生成 Ninja 构建图，支持大规模并行编译
- OH 的所有模块使用统一的构建系统，便于集成
- Cargo.toml 仅作为元数据参考，不参与实际构建

### 特性启用对比

| 特性 | Cargo 命令 | OH BUILD.gn | 功能 |
|------|------------|-------------|------|
| **std** | 默认启用 | `"std"` | 标准库支持 |
| **alloc** | 默认启用 | `"alloc"` | 内存分配 |
| **race** | 默认启用 | `"race"` | First one wins |
| **parking_lot** | `--features parking_lot` | ❌ 未启用 | 内存优化 |
| **critical-section** | `--features critical-section` | ❌ 未启用 | 嵌入式支持 |

---

## 构建示例

### 编译 once_cell 本身

```bash
# 进入 OH 源码目录
cd /path/to/openharmony

# 使用 GN 生成构建文件
gn gen out/default

# 编译 once_cell
ninja -C out/default //third_party/rust/crates/once_cell:lib
```

**输出文件**:
```
out/default/obj/third_party/rust/crates/once_cell/libonce_cell.rlib
```

### 在其他模块中使用 once_cell

**方式 1: GN 依赖**
```gn
# your_module/BUILD.gn
import("//build/ohos.gni")

ohos_cargo_crate("your_lib") {
    deps = [
        "//third_party/rust/crates/once_cell:lib",
    ]
    features = ["std"]
}
```

**方式 2: Cargo.toml 依赖**
```toml
# your_module/Cargo.toml
[dependencies]
once_cell = "1.17.0"
```

---

## 故障排查

### 常见构建问题

#### 问题 1: 找不到 once_cell 头文件

**错误信息**:
```
error: could not find `once_cell` in `{{root}}`
```

**原因**: 未在 BUILD.gn 中添加依赖

**解决方案**:
```gn
deps = [
    "//third_party/rust/crates/once_cell:lib",
]
```

#### 问题 2: 特性不匹配

**错误信息**:
```
error: `once_cell::sync` is not enabled
```

**原因**: once_cell 的 BUILD.gn 未启用对应特性

**解决方案**: 确认 BUILD.gn 包含所需特性
```gn
features = [
    "std",  # sync 模块需要 std
]
```

#### 问题 3: 版本冲突

**错误信息**:
```
error: multiple versions of crate `once_cell` found
```

**原因**: 依赖者使用了不同版本的 once_cell

**解决方案**: 统一所有依赖者使用 OH 提供的 once_cell 版本
```gn
# ❌ 不要这样做
once_cell = { version = "1.12.0" }  # Cargo.toml

# ✅ 正确做法
deps = [
    "//third_party/rust/crates/once_cell:lib",  # BUILD.gn
]
```

---

## 升级指南

### 升级上游版本

**步骤**:

1. **下载新版本**
   ```bash
   cd third_party/rust/crates/once_cell
   git fetch upstream
   git checkout v1.18.0  # 新版本
   ```

2. **更新 BUILD.gn**
   ```gn
   cargo_pkg_version = "1.18.0"  # 更新版本号
   ```

3. **检查特性列表**
   - 查看上游 CHANGELOG.md
   - 确认是否有新增或移除的特性
   - 相应调整 `features` 列表

4. **编译验证**
   ```bash
   ninja -C out/default //third_party/rust/crates/once_cell:lib
   ```

5. **编译依赖者**
   ```bash
   ninja -C out/default //third_party/rust/crates/clap:lib
   ninja -C out/default //third_party/rust/crates/rustix:lib
   ```

6. **运行测试**（如果有）
   ```bash
   # 在 once_cell 目录运行上游测试
   cargo test
   ```

### 注意事项

⚠️ **重要**:
1. **不要修改源代码**: 保持与上游完全一致
2. **更新版本号同步更新**: BUILD.gn 和 README.OpenSource 都要更新
3. **特性审查**: 仔细阅读上游 CHANGELOG，确认特性变更
4. **编译测试**: 确保所有依赖者仍能正常编译

---

## 总结

once_cell 在 OpenHarmony 中的构建适配体现了 OH 第三方库集成的最佳实践：

✅ **最小化配置**: 仅添加 BUILD.gn 和 bundle.json
✅ **零侵入**: 不修改源代码，完全遵循上游
✅ **标准化**: 使用统一的 `ohos_cargo_crate` 模板
✅ **易维护**: 升级只需更新版本号和特性列表
✅ **低风险**: 无平台特定代码，回归风险极低

这是一个值得其他 Rust crate 学习的参考案例。
