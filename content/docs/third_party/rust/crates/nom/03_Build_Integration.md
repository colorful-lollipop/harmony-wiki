# OH 构建适配

> nom 库在 OpenHarmony 构建系统中的配置详情

## 构建配置概览

### BUILD.gn 文件

nom 库通过 OpenHarmony 的 `ohos_cargo_crate` 模板集成到构建系统中：

```gn
# 文件位置: third_party/rust/crates/nom/BUILD.gn

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "nom"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "7.1.3"
    cargo_pkg_authors = "contact@geoffroycouprie.com"
    cargo_pkg_name = "nom"
    cargo_pkg_description = "A byte-oriented, zero-copy, parser combinators library"

    deps = [
        "//third_party/rust/crates/memchr:lib",
        "//third_party/rust/crates/minimal-lexical:lib",
    ]

    features = [
        "alloc",
        "std",
    ]

    module_output_extension = ".rlib"
    part_name = "rust_nom"
    subsystem_name = "thirdparty"
}
```

### 配置详解

#### 核心配置字段

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | "nom" | Rust crate 名称 |
| `crate_type` | "rlib" | Rust 静态库类型 |
| `crate_root` | "src/lib.rs" | 库入口文件 |
| `edition` | "2018" | Rust edition |
| `part_name` | "rust_nom" | OH 部件名称 |
| `subsystem_name` | "thirdparty" | 所属子系统 |

#### 版本配置

| 字段 | 值 | 说明 |
|------|-----|------|
| `cargo_pkg_version` | "7.1.3" | 与上游版本一致 |
| `cargo_pkg_name` | "nom" | 上游 crate 名称 |
| `cargo_pkg_authors` | 上游作者 | contact@geoffroycouprie.com |

---

## Feature 配置

### 启用的 Feature

```gn
features = [
    "alloc",
    "std",
]
```

#### Feature 详解

| Feature | 状态 | 用途 |
|---------|------|------|
| `alloc` | ✅ 启用 | 支持动态内存分配 (Vec, String 等) |
| `std` | ✅ 启用 | 支持标准库 (启用 alloc + memchr/std + minimal-lexical/std) |
| `docsrs` | ❌ 未启用 | 文档构建专用 |
| `default` | ✅ 等同于 std | 上游默认 feature |

#### Feature 依赖关系

```
std (启用时)
├── alloc
├── memchr/std
└── minimal-lexical/std
```

### 与上游 Cargo.toml 对比

**上游 Cargo.toml**:
```toml
[features]
alloc = []
std = ["alloc", "memchr/std", "minimal-lexical/std"]
default = ["std"]
```

**OH BUILD.gn**: `features = ["alloc", "std"]`

**分析**: OH 配置与上游默认配置一致，保持了 nom 的完整功能。

---

## 依赖配置

### 外部依赖

```gn
deps = [
    "//third_party/rust/crates/memchr:lib",
    "//third_party/rust/crates/minimal-lexical:lib",
]
```

| 依赖 | 上游版本 | OH 内部版本 | 用途 |
|------|---------|------------|------|
| memchr | 2.3 | OH internal | 高性能字节搜索 |
| minimal-lexical | 0.2.0 | OH internal | 高性能数字解析 |

### 依赖版本锁定

**上游 Cargo.toml**:
```toml
[dependencies.memchr]
version = "2.3"
default-features = false

[dependencies.minimal-lexical]
version = "0.2.0"
default-features = false
```

**分析**: OH 使用上游指定的版本，通过 OH 内部的 Rust crates 仓库进行管理。

---

## 构建产物配置

```gn
module_output_extension = ".rlib"
```

| 配置 | 值 | 说明 |
|------|-----|------|
| 输出类型 | `.rlib` | Rust 静态库 |
| 构建类型 | release | OH 默认构建类型 |

---

## 部件化配置

### bundle.json 配置

```json
{
  "name": "@ohos/rust_nom",
  "description": "A Rust library that provides support for parsing byte streams.",
  "version": "6.1",
  "license": "MIT License",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/nom"
  },
  "component": {
    "name": "rust_nom",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/nom:lib"
        }
      ],
      "test": []
    }
  }
}
```

### 关键配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | "@ohos/rust_nom" | OH NPM 风格名称 |
| `component.name` | "rust_nom" | 内部部件标识 |
| `subsystem` | "thirdparty" | 属于 thirdparty 子系统 |
| `inner_kits` | lib 目标 | 对外提供的构建目标 |

---

## 构建命令

### 独立构建

```bash
# 在 OpenHarmony 构建环境中
hb set
hb build -T //third_party/rust/crates/nom:lib
```

### 查看构建产物

```bash
# 构建产物位置 (取决于 out 目录)
out/xxx/packages/third_party/rust/crates/nom/
```

### 构建验证

```bash
# 检查 nom 是否正确构建
ls -la out/*/packages/third_party/rust/crates/nom/

# 应包含:
# - libnom.rlib
# - Cargo.toml (如果有对外可见)
```

---

## 与上游构建系统的差异

### 上游构建方式

上游 nom 使用标准 Cargo 构建：

```bash
# 开发构建
cargo build

# 发布构建
cargo build --release

# 测试
cargo test

# 文档
cargo doc
```

### OH 构建方式

OH 使用 GN + Cargo 混合构建：

```bash
# GN 配置
hb build -T //third_party/rust/crates/nom:lib

# 或使用 REPO 构建
./build.sh --product xxx --build-target //third_party/rust/crates/nom:lib
```

### 差异总结

| 维度 | 上游 (Cargo) | OH (GN) |
|------|-------------|---------|
| 构建工具 | cargo | gn + cargo |
| 配置格式 | Cargo.toml | BUILD.gn |
| 依赖管理 | Cargo.lock | OH 内部 crates 仓库 |
| 输出格式 | .rlib / .so | .rlib |
| 部件化 | 无 | 有 (bundle.json) |

---

## 常见问题

### Q1: 如何更新 nom 版本？

1. 修改 `BUILD.gn` 中的 `cargo_pkg_version`
2. 修改 `bundle.json` 中的 `version`
3. 同步更新 OH 内部 crates 仓库中的 nom 源码
4. 重新构建验证

### Q2: 能否禁用某些 feature？

可以，但需要修改 `features` 配置：

```gn
features = [
    "alloc",
    // 禁用 "std" 会导致 nom 以 no_std 模式运行
    // 此时依赖也需要调整
]
```

**注意**: 禁用 `std` 会使 nom 只能在 `no_std` 环境下使用，需要自备内存分配器。

### Q3: 如何添加新的依赖？

在 OH 中添加 Rust crate 依赖需要：

1. 将依赖 crate 添加到 OH third_party/rust/crates/ 目录
2. 创建对应的 BUILD.gn
3. 在 nom 的 BUILD.gn 中添加依赖路径

### Q4: 构建失败怎么办？

1. **检查版本兼容性**: 确保 Rust 版本 >= 1.48
2. **检查依赖**: 确保 memchr 和 minimal-lexical 已正确集成
3. **清理构建**: `hb clean && hb build -T //third_party/rust/crates/nom:lib`
4. **查看日志**: 检查详细的错误信息

---

## 维护建议

### 版本更新流程

```
nom 版本更新:
├── 1. 检查上游版本
├── 2. 更新 BUILD.gn 版本号
├── 3. 更新 bundle.json 版本号
├── 4. 同步上游源码 (如果有变更)
├── 5. 构建验证
├── 6. 运行依赖者测试 (rust-cexpr)
└── 7. 提交变更
```

### 配置验证清单

| 检查项 | 状态 |
|--------|------|
| 版本号与上游一致 | ☐ |
| features 配置正确 | ☐ |
| 依赖路径正确 | ☐ |
| bundle.json 完整 | ☐ |
| 构建成功 | ☐ |
| 依赖者测试通过 | ☐ |

---

## 相关文档

- [02_Patches.md](./02_Patches.md) - Patch 分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景
- [bundle.json](../bundle.json) - OH 组件配置
- [BUILD.gn](../BUILD.gn) - 构建配置

---

**最后更新**: 2024年
