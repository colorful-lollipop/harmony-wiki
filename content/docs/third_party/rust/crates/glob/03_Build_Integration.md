# OpenHarmony 构建适配详解

本文档详细说明 glob 库在 OpenHarmony 中的构建集成配置。

---

## 1. 概述

glob 库在 OpenHarmony 中通过 **BUILD.gn** 文件进行 GN 构建系统集成，使用 `ohos_cargo_crate` 模板将 Cargo 项目映射到 GN 构建系统。

**适配特点**:
- ✅ 无需修改源代码
- ✅ 完整保留 Cargo 元数据
- ✅ 使用标准 ohos_cargo_crate 模板
- ✅ 无特殊编译选项或 features
- ✅ 静态库输出（.rlib）

---

## 2. BUILD.gn 结构说明

### 2.1 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0
# ...
import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "glob"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "0.3.1"
    cargo_pkg_authors = "The Rust Project Developers"
    cargo_pkg_name = "glob"
    module_output_extension = ".rlib"
    part_name = "rust_glob"
    subsystem_name = "thirdparty"
}
```

### 2.2 配置项详解

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `glob` | Crate 名称，必须与 Cargo.toml 一致 |
| `crate_type` | `rlib` | 生成 Rust 静态库（.rlib） |
| `crate_root` | `src/lib.rs` | 库入口文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |
| `edition` | `2015` | Rust 版本 |
| `cargo_pkg_version` | `0.3.1` | Cargo 版本号 |
| `cargo_pkg_authors` | `The Rust Project Developers` | Cargo 作者信息 |
| `cargo_pkg_name` | `glob` | Cargo 包名 |
| `module_output_extension` | `.rlib` | 输出文件扩展名 |
| `part_name` | `rust_glob` | OH 部件名称 |
| `subsystem_name` | `thirdparty` | OH 子系统名称 |

---

## 3. Cargo 到 GN 的映射

### 3.1 Cargo.toml vs BUILD.gn

| Cargo.toml 字段 | BUILD.gn 字段 | 说明 |
|----------------|---------------|------|
| `[package].name` | `crate_name`, `cargo_pkg_name` | 包名称 |
| `[package].version` | `cargo_pkg_version` | 版本号 |
| `[package].authors` | `cargo_pkg_authors` | 作者列表 |
| `[package].edition` | `edition` | Rust 版本 |
| `lib.path` | `crate_root` | 库入口 |
| `src/**/*.rs` | `sources` | 源文件列表 |

### 3.2 元数据保留

BUILD.gn 完整保留了 Cargo 包的元数据，这些信息在构建过程中被使用：

**用途**:
- 版本检查和兼容性验证
- 生成文档时的信息展示
- 依赖解析和冲突检测
- 构建产物命名规范

---

## 4. 与上游构建系统的差异

### 4.1 Cargo vs GN

| 特性 | Cargo | GN (OH) |
|------|-------|---------|
| **依赖管理** | 自动下载 crates.io | 手动指定 `deps` |
| **配置文件** | Cargo.toml | BUILD.gn + bundle.json |
| **构建输出** | target/ | out/ |
| **目标平台** | 自动检测 | 通过 target_os 控制 |
| **工作空间** | 支持 workspace | 使用 `gn` 工作区 |
| **条件编译** | `#[cfg(...)]` | GN 条件配置 |

### 4.2 glob 适配细节

**无差异的部分**:
- ✅ 源代码完全未修改
- ✅ 依赖管理（glob 无外部依赖）
- ✅ 条件编译（无平台特定代码）

**OH 特定的配置**:
- 🆕 `part_name` 和 `subsystem_name` - OH 部件化要求
- 🆕 `module_output_extension` - 指定 .rlib 扩展名

### 4.3 构建流程

```
Cargo 构建:
1. 解析 Cargo.toml
2. 下载依赖（glob 无依赖）
3. 编译 src/lib.rs
4. 生成 target/deps/libglob-*.rlib

GN 构建:
1. 解析 BUILD.gn
2. 读取 bundle.json 组件配置
3. 使用 rustc 编译 src/lib.rs
4. 生成 out/.../libglob.rlib
```

---

## 5. 特殊处理

### 5.1 无特殊处理说明

glob 库在 OH 适配中**没有特殊处理**，完全遵循标准流程：

**无需特殊处理的原因**:
1. **无外部依赖**: glob 仅依赖 Rust 标准库
2. **无平台特定代码**: 纯 Rust 实现的跨平台库
3. **无 features**: 不需要启用任何 Cargo features
4. **无 build.rs**: 不需要构建脚本
5. **无条件编译**: 无 `#[cfg(target_os = "...")]` 代码

### 5.2 与其他库对比

| 库 | 特殊处理 | 说明 |
|----|---------|------|
| **glob** | ❌ 无 | 简单直接 |
| **clang-sys** | ✅ 有 | 需要查找 libclang 路径 |
| **bindgen** | ✅ 有 | 复杂依赖，需要 features |
| **openssl-sys** | ✅ 有 | 需要配置 OpenSSL 路径 |

**结论**: glob 是最简单的集成案例。

---

## 6. 依赖关系

### 6.1 依赖声明

glob 在 BUILD.gn 中**无外部依赖**：

```gn
ohos_cargo_crate("lib") {
    # 无 deps 字段
    # 无 build_deps 字段
    # 无 features 字段
}
```

### 6.2 被依赖关系

glob 被以下模块依赖：

```gn
# clang-sys/BUILD.gn
ohos_cargo_crate("lib") {
    deps = [
        "//third_party/rust/crates/glob:lib",  # 运行时依赖
        # ...
    ]
    build_deps = [
        "//third_party/rust/crates/glob:lib",  # 构建时依赖
    ]
}
```

**依赖类型**:
- **运行时依赖**: clang-sys 在运行时使用 glob 的 API
- **构建时依赖**: clang-sys 的 build.rs 脚本使用 glob 查找文件

---

## 7. 编译选项

### 7.1 Rust 编译器选项

glob 使用默认的 Rust 编译选项，无特殊配置：

- **优化级别**: 继承 OH 系统默认（通常为 `opt` 或 `debug`）
- **目标平台**: 由 OH 构建系统控制
- **LTO**: 未启用
- **panic 策略**: 继承系统默认

### 7.2 GN 构建选项

```gn
# 默认 GN 选项（从 ohos_cargo_crate 模板继承）
- enable_rust = true
- rust_std = true
- use_rust_clippy = false  # 未启用 clippy
```

---

## 8. 构建产物

### 8.1 输出文件

| 构建类型 | 输出路径 | 文件名 |
|---------|---------|--------|
| Debug | `out/.../obj/third_party/rust/crates/glob/libglob.rlib` | `libglob.rlib` |
| Release | `out/.../obj/third_party/rust/crates/glob/libglob.rlib` | `libglob.rlib` |

### 8.2 产物特性

- **类型**: Rust 静态库（.rlib）
- **大小**: 约 50-100 KB（取决于优化级别）
- **符号**: 完整保留 Rust 符号信息
- **可重定位**: 是，可被其他 Rust crate 链接

---

## 9. bundle.json 配置

### 9.1 组件注册

```json
{
  "name": "@ohos/rust_glob",
  "version": "6.1",
  "component": {
    "name": "rust_glob",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "inner_kits": [
      {
        "name": "//third_party/rust/crates/glob:lib"
      }
    ]
  }
}
```

### 9.2 配置项说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | `@ohos/rust_glob` | OH 组件唯一标识 |
| `version` | `6.1` | OH 组件版本（非 crate 版本） |
| `component.name` | `rust_glob` | 部件名称 |
| `component.subsystem` | `thirdparty` | 所属子系统 |
| `component.adapted_system_type` | `["standard"]` | 适配的系统类型 |
| `inner_kits` | `["//third_party/rust/crates/glob:lib"]` | 提供的内部接口 |

---

## 10. 测试与验证

### 10.1 单元测试

glob 包含上游的单元测试，位于 `tests/` 目录：

```rust
// tests/glob-std.rs
// 标准库兼容性测试
```

**OH 集成**:
- ✅ 测试文件包含在源代码中
- ⚠️  OH 构建系统可能未自动运行这些测试
- 💡 建议：考虑添加 OH 单元测试集成

### 10.2 构建验证

```bash
# 验证构建
gn gen out/default
ninja -C out/default //third_party/rust/crates/glob:lib

# 验证产物
ls -l out/default/obj/third_party/rust/crates/glob/libglob.rlib
```

---

## 11. 版本升级指南

### 11.1 升级步骤

当上游发布新版本时（如果存在），升级流程如下：

1. **下载新版本**:
   ```bash
   git pull https://github.com/rust-lang/glob.git
   ```

2. **更新 Cargo.toml**:
   ```toml
   [package]
   version = "x.y.z"  # 新版本号
   ```

3. **更新 BUILD.gn**:
   ```gn
   cargo_pkg_version = "x.y.z"  # 同步版本号
   ```

4. **更新 README.OpenSource**:
   ```json
   {
     "Version Number": "x.y.z"
   }
   ```

5. **验证构建**:
   ```bash
   gn gen out/default
   ninja -C out/default //third_party/rust/crates/glob:lib
   ```

6. **测试验证**:
   ```bash
   # 运行上游单元测试
   cargo test
   ```

### 11.2 回滚策略

如果升级后出现问题：

```bash
git revert <commit-hash>
# 或
git checkout <old-version>
```

---

## 12. 常见问题

### Q1: 为什么不启用 Rust LTO（Link Time Optimization）?

**A**: glob 是小型工具库，LTO 带来的性能提升有限，反而增加构建时间。如果需要优化，可以在依赖它的 crate 中启用。

### Q2: 为什么不提供 `dylib` 输出?

**A**:
- Rust 生态主要使用 `rlib`（静态库）
- 动态库在 Rust 中存在版本兼容性问题
- OH 中 glob 仅用于编译时，不需要动态库

### Q3: 为什么没有 `features` 配置?

**A**: glob 在 Cargo.toml 中未定义任何 features，因此无需在 BUILD.gn 中配置。

### Q4: 如何添加 OH 特定功能?

**A**:
1. 评估是否可以推向上游
2. 如果是 OH 特有需求，创建 OH 特定的条件分支：
   ```rust
   #[cfg(target_os = "ohos")]
   // OH 特定代码
   ```
3. 更新 BUILD.gn 添加必要的配置

---

## 13. 最佳实践

### 13.1 维护建议

1. **定期同步上游**:
   - 关注上游 releases
   - 评估新版本的兼容性
   - 及时更新安全补丁

2. **保持配置简单**:
   - 避免添加不必要的 features
   - 不启用实验性选项
   - 保持与上游版本同步

3. **文档更新**:
   - 升级版本时同步更新 README.OpenSource
   - 记录重要变更
   - 更新 bundle.json 版本号

### 13.2 开发建议

1. **使用上游 API**:
   - 不要修改上游代码
   - 通过封装实现 OH 特定需求
   - 向上游提交改进建议

2. **测试覆盖**:
   - 确保上游测试通过
   - 添加 OH 特定测试用例
   - 考虑集成测试

---

## 14. 总结

glob 的 OpenHarmony 构建适配非常简单直接，展示了理想的零 Patch 集成案例：

**核心特点**:
- ✅ 无源代码修改
- ✅ 标准的 ohos_cargo_crate 模板
- ✅ 完整保留 Cargo 元数据
- ✅ 无特殊编译选项
- ✅ 维护成本低

**适用场景**:
- 纯 Rust 实现的跨平台库
- 无外部依赖或简单依赖
- 无平台特定代码
- 基础工具库

---

**最后更新**: 2026-02-08
