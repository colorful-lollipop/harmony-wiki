# OH 构建适配

## 3.1 构建系统概述

strsim-rs 在 OpenHarmony 中使用 **GN (Generate Ninja)** 构建系统，通过 `ohos_cargo_crate` 模板集成 Rust crate。

**构建模板**：`ohos_cargo_crate`
**输出类型**：`.rlib` (Rust 静态库)
**构建系统版本**：与 OpenHarmony 主构建一致

## 3.2 BUILD.gn 配置详解

### 3.2.1 完整配置文件

```gn
# 文件位置：/Volumes/lexar/code/d/work/oh/third_party/rust/crates/strsim-rs/BUILD.gn

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "strsim"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "0.10.0"
    cargo_pkg_authors = "Danny Guo <danny@dannyguo.com>"
    cargo_pkg_name = "strsim"
    cargo_pkg_description = "Implementations of string similarity metrics. Includes Hamming, Levenshtein,OSA, Damerau-Levenshtein, Jaro, Jaro-Winkler, and Sørensen-Dice."
    module_output_extension = ".rlib"
    part_name = "rust_strsim_rs"
    subsystem_name = "thirdparty"
}
```

### 3.2.2 配置字段说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | `"strsim"` | OH 内部使用的库名称 |
| `crate_type` | `"rlib"` | Rust 静态库类型 |
| `crate_root` | `"src/lib.rs"` | crate 根文件 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |
| `edition` | `"2015"` | Rust Edition（较旧但兼容） |
| `cargo_pkg_version` | `"0.10.0"` | 与上游版本一致 |
| `cargo_pkg_authors` | Danny Guo | 上游作者信息 |
| `cargo_pkg_name` | `"strsim"` | 上游 crate 名称 |
| `cargo_pkg_description` | 描述文本 | 上游描述 |
| `module_output_extension` | `".rlib"` | 输出文件扩展名 |
| `part_name` | `"rust_strsim_rs"` | OH 组件名称 |
| `subsystem_name` | `"thirdparty"` | 所属子系统 |

### 3.2.3 配置分析

**标准配置**：✅ 所有配置均为标准项，无 OH 特定修改

**配置特点**：
- ✅ 无 `defines` - 不需要定义 OH 特定宏
- ✅ 无 `configs` - 不需要特殊编译配置
- ✅ 无 `deps` - 无额外依赖
- ✅ 无 `features` - 不需要特性开关
- ✅ 无 `cflags` / `cxxflags` - Rust 项目不需要

## 3.3 与上游构建差异

| 维度 | 上游 (Cargo) | OpenHarmony (GN) | 差异 |
|------|--------------|------------------|------|
| **构建工具** | cargo | gn + ninja | 构建系统不同 |
| **配置文件** | Cargo.toml | BUILD.gn | 配置格式不同 |
| **输出格式** | .rlib | .rlib | 一致 |
| **版本同步** | cargo_pkg_version | 需手动同步 | 需要维护一致性 |
| **依赖管理** | Cargo.lock | GN 依赖管理 | 机制不同 |
| **测试** | cargo test | hmt run | 命令不同 |

### 3.3.1 版本同步要求

**关键点**：需要确保 `cargo_pkg_version` 与 `Cargo.toml` 中的版本一致。

```toml
# Cargo.toml
[package]
version = "0.10.0"
```

```gn
# BUILD.gn
cargo_pkg_version = "0.10.0"  # 必须与 Cargo.toml 一致
```

**建议**：在升级版本时，同时更新两个文件。

## 3.4 OH 特有配置

### 3.4.1 组件配置 (bundle.json)

```json
{
  "name": "@ohos/rust_strsim_rs",
  "version": "6.1",
  "license": "MIT License",
  "publishAs": "code-segment",
  "component": {
    "name": "rust_strsim_rs",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/strsim-rs:lib"
        }
      ],
      "test": []
    }
  }
}
```

### 3.4.2 组件信息

| 配置项 | 值 |
|--------|-----|
| **OH 组件名** | @ohos/rust_strsim_rs |
| **版本号** | 6.1 (OH 内部版本) |
| **子系统** | thirdparty |
| **适配系统** | standard |
| **组件类型** | inner_kit (内部套件) |

## 3.5 构建命令

### 3.5.1 独立构建

```bash
cd /Volumes/lexar/code/d/work/oh
hb build system:thirdparty:rust_strsim_rs
```

### 3.5.2 作为依赖构建

当 clap 等模块依赖 strsim-rs 时，构建系统会自动处理：

```bash
# 构建包含 strsim-rs 依赖的模块
hb build system:thirdparty:rust_clap
```

### 3.5.3 运行测试

```bash
# Rust 单元测试
cd /Volumes/lexar/code/d/work/oh/third_party/rust/crates/strsim-rs
cargo test
```

## 3.6 构建产物

### 3.6.1 输出文件

| 文件 | 类型 | 说明 |
|------|------|------|
| `out/release/rust_strsim_rs/obj/third_party/rust/crates/strsim-rs/lib.rlib` | .rlib | 静态库文件 |
| `out/debug/rust_strsim_rs/obj/third_party/rust/crates/strsim-rs/lib.rlib` | .rlib | Debug 版本 |

### 3.6.2 库信息

```bash
# 查看库信息
rustfilt < lib.rlib  # 或使用其他 Rust 工具
```

## 3.7 故障排查

### 3.7.1 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 版本不匹配 | cargo_pkg_version 与 Cargo.toml 不一致 | 同步版本号 |
| 构建失败 | 缺少 Rust 工具链 | 安装对应版本 Rust |
| 链接错误 | 依赖模块未构建 | 先构建依赖模块 |

### 3.7.2 调试命令

```bash
# 查看构建日志
cat out/build.log | grep strsim

# 检查依赖关系
hb deps --tree system:thirdparty:rust_strsim_rs

# 验证库文件
file out/release/obj/third_party/rust/crates/strsim-rs/lib.rlib
```

## 3.8 维护建议

### 3.8.1 版本升级检查清单

- [ ] 检查上游新版本发布
- [ ] 更新 `Cargo.toml` 版本号
- [ ] 同步更新 `BUILD.gn` 中的 `cargo_pkg_version`
- [ ] 运行 `cargo test` 验证测试通过
- [ ] 执行 OH 构建验证
- [ ] 更新 `bundle.json` 中的版本（如需要）
- [ ] 更新本 Wiki 文档

### 3.8.2 构建优化建议

strsim-rs 作为纯算法库：
- **优化等级**：可使用默认优化级别
- **LTO**：可启用链接时优化
- **增量构建**：支持

---

**结论**：strsim-rs 的 OH 构建适配非常简单，仅需标准 `ohos_cargo_crate` 模板配置，无任何 OH 特定修改。
