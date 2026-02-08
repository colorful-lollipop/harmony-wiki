# OH 构建适配

> 本文档说明 syn 库在 OpenHarmony 构建系统中的集成方式
>
> 重点：BUILD.gn 配置、Features 启用策略、与上游构建系统的差异

---

## 3.1 BUILD.gn 配置说明

### 3.1.1 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
  crate_name = "syn"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2021"
  cargo_pkg_version = "2.0.48"
  cargo_pkg_authors = "David Tolnay <dtolnay@gmail.com>"
  cargo_pkg_name = "syn"
  cargo_pkg_description = "Parser for Rust source code"

  # 依赖
  deps = [
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
    "//third_party/rust/crates/unicode-ident:lib",
  ]

  # 启用的 features - 几乎所有 features 都被启用
  features = [
    "clone-impls",
    "derive",
    "extra-traits",
    "full",
    "parsing",
    "printing",
    "proc-macro",
    "quote",
    "visit",
    "visit-mut",
  ]

  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  module_output_extension = ".rlib"
  part_name = "rust_syn"
  subsystem_name = "thirdparty"
}
```

### 3.1.2 配置解析

| 配置项 | 值 | 说明 |
|-------|-----|------|
| **模板类型** | `ohos_cargo_crate` | OH 提供的 Cargo.toml → GN 转换模板 |
| **crate_name** | `syn` | Rust crate 名称 |
| **crate_type** | `rlib` | Rust 静态库 |
| **crate_root** | `src/lib.rs` | 库入口文件 |
| **edition** | `2021` | Rust edition |
| **version** | `2.0.48` | 与 Cargo.toml 一致 |

---

## 3.2 关键编译选项

### 3.2.1 Features 配置

OH 启用了 syn 的所有主要 features：

| Feature | 说明 | 为什么启用 |
|---------|------|----------|
| `clone-impls` | 为所有语法树类型实现 Clone | 需要克隆语法树 |
| `derive` | 支持 derive 宏的输入解析 | derive 宏的基础 |
| `extra-traits` | Debug、Eq、PartialEq、Hash | 调试和测试支持 |
| `full` | 完整的 Rust 语法树 | 解析任意 Rust 代码 |
| `parsing` | 解析功能 | 将 tokens 转换为语法树 |
| `printing` | 打印功能 | 将语法树转换为 tokens |
| `proc-macro` | 运行时依赖 libproc_macro | 在过程宏中使用 |
| `quote` | 启用 quote 集成 | 与 quote 库配合使用 |
| `visit` | 语法树遍历 trait | 只读遍历语法树 |
| `visit-mut` | 语法树可变遍历 | 修改语法树 |

**未启用的 Features**：
- `fold`：转换 owned 语法树（OH 不需要）
- `test`：测试相关（OH 不使用）
- `default`：默认 features（OH 显式启用所有需要的 features）

### 3.2.2 为什么启用所有 Features？

**原因**：

1. **功能完整性**：OH 的过程宏需要完整的 syn 功能
2. **避免遗漏**：显式启用所有需要的 features，避免隐式依赖
3. **性能优化**：虽然编译时间增加，但运行时性能不受影响
4. **简化依赖**：不需要每个依赖者单独指定 features

**权衡**：
- ✅ **优点**：功能完整，依赖管理简单
- ⚠️ **缺点**：编译时间稍长（但对于基础设施库可接受）

### 3.2.3 与上游 Cargo.toml 对比

上游 Cargo.toml：
```toml
[features]
default = ["derive", "parsing", "printing", "clone-impls", "proc-macro"]
derive = []
full = []
parsing = []
printing = ["quote"]
visit = []
visit-mut = []
fold = []
clone-impls = []
extra-traits = []
proc-macro = ["proc-macro2/proc-macro", "quote/proc-macro"]
```

OH BUILD.gn：
```gn
features = [
  "clone-impls",
  "derive",
  "extra-traits",
  "full",
  "parsing",
  "printing",
  "proc-macro",
  "quote",
  "visit",
  "visit-mut",
]
```

**对比结果**：
- OH 启用了所有主要 features
- 比上游默认 features 多了：`extra-traits`, `full`, `visit`, `visit-mut`
- 功能完全覆盖 OH 的需求

---

## 3.3 依赖管理

### 3.3.1 直接依赖

```gn
deps = [
  "//third_party/rust/crates/proc-macro2:lib",
  "//third_party/rust/crates/quote:lib",
  "//third_party/rust/crates/unicode-ident:lib",
]
```

| 依赖 | 版本 | 用途 |
|-----|------|-----|
| proc-macro2 | 1.0.75 | Token 流处理 |
| quote | 1.0.35 | 代码生成 |
| unicode-ident | 1 | Unicode 标识符解析 |

### 3.3.2 依赖版本策略

**上游 Cargo.toml**：
```toml
[dependencies]
proc-macro2 = { version = "1.0.75", default-features = false }
quote = { version = "1.0.35", optional = true, default-features = false }
unicode-ident = "1"
```

**OH 策略**：
- 使用 OH 统一管理的版本
- 通过 GN 路径引用，避免版本冲突
- 与 OH 中的其他 crates 保持一致

### 3.3.3 无条件编译选项

syn 的 BUILD.gn **没有**使用以下 GN 特性：
- `defines`：无宏定义
- `configs`：无额外配置
- `public_configs`：无公共配置
- `cflags`：无编译标志
- `include_dirs`：无额外头文件

**原因**：
- syn 是纯 Rust 库，不需要 C 交互
- 无平台特定逻辑
- 使用标准 Rust edition 2021

---

## 3.4 与上游构建系统的差异

### 3.4.1 构建系统对比

| 方面 | 上游（Cargo） | OH（GN） |
|-----|-------------|-----------|
| **配置文件** | Cargo.toml | BUILD.gn |
| **依赖管理** | crates.io | OH 内部 crates |
| **版本锁定** | Cargo.lock | 手动指定 |
| **Features** | 按需启用 | 显式启用所有需要的 |
| **构建工具** | rustc/cargo | OHGN + rustc |
| **目标平台** | 任意 | OH 平台（Linux/Windows 等） |

### 3.4.2 构建流程对比

**上游构建流程**：
```
Cargo.toml
    ↓
cargo build
    ↓
cargo（调用 rustc）
    ↓
编译 syn
```

**OH 构建流程**：
```
BUILD.gn
    ↓
ohos_cargo_crate 模板
    ↓
解析 Cargo.toml → 生成 GN 配置
    ↓
GN（调用 rustc）
    ↓
编译 syn
```

### 3.4.3 关键差异

| 差异点 | 说明 | 影响 |
|-------|------|-----|
| **配置方式** | 声明式 vs 命令式 | BUILD.gn 需要手动配置 features |
| **依赖来源** | crates.io vs OH 内部 | OH 需要明确指定依赖路径 |
| **版本管理** | 自动 vs 手动 | OH 需要手动同步版本 |
| **平台检测** | 自动 vs 需配置 | syn 无平台逻辑，无影响 |

---

## 3.5 特殊处理

### 3.5.1 禁用的功能

**无** - OH 启用了 syn 的所有主要功能。

理论上可以禁用的 features（但 OH 选择启用）：
- `extra-traits`：如果不需要调试和测试
- `visit` / `visit-mut`：如果不需要遍历语法树

**OH 选择启用的原因**：
- `extra-traits`：调试和单元测试需要
- `visit` / `visit-mut`：高级过程宏需要

### 3.5.2 添加的 OH 特定源文件

**无** - syn 没有添加任何 OH 特定源文件。

这与许多其他第三方库不同：
- ❌ curl：添加了 OH 证书适配代码
- ❌ openssl：添加了 OH 加密适配代码
- ✅ syn：纯语法树解析，无需平台代码

### 3.5.3 OH 特定的构建选项

**无** - syn 的 BUILD.gn 没有任何 OH 特定的编译选项。

这体现在：
- 无 `#ifdef OHOS` 条件编译
- 无 `#[cfg(target_os = "ohos")]` 属性
- 无 OH 特定的 `defines` 或 `configs`

---

## 3.6 构建适配策略总结

### 3.6.1 OH 的适配哲学

对于纯逻辑库（如 syn），OH 采用**最小化适配**策略：
1. 保持源码与上游完全一致
2. 仅通过 BUILD.gn 适配构建系统
3. 启用完整 features 以满足所有需求
4. 不添加 OH 特定代码

### 3.6.2 为什么这种策略适合 syn？

| 原因 | 说明 |
|-----|------|
| **功能纯粹** | 仅解析 Rust 语法，无平台逻辑 |
| **依赖清晰** | 依赖 proc-macro2、quote，都是纯逻辑库 |
| **API 稳定** | syn 2.0 API 已经非常稳定 |
| **广泛使用** | 生态系统中广泛使用，社区支持好 |

### 3.6.3 与其他库的对比

| 库 | 适配策略 | 原因 |
|-----|---------|------|
| **syn** | 最小化适配 | 纯语法树解析 |
| **curl** | 大量 Patch | 网络接口差异、证书路径 |
| **openssl** | 大量 Patch | 加密算法适配、硬件加速 |
| **zlib** | 少量 Patch | 性能优化、内存对齐 |
| **serde** | 最小化适配 | 纯序列化逻辑（类似 syn） |

---

## 3.7 使用 syn 的 BUILD.gn 示例

### 3.7.1 OH 模块中引用 syn

**示例 1：过程宏库**
```gn
ohos_rust_proc_macro("my_macros") {
  part_name = "my_part"
  subsystem_name = "my_subsystem"

  crate_name = "my_macros"
  edition = "2021"

  deps = [
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
    "//third_party/rust/crates/syn:lib",
  ]

  sources = [ "src/lib.rs" ]
}
```

**示例 2：普通 Rust 库**
```gn
ohos_cargo_crate("my_lib") {
  crate_name = "my_lib"
  crate_type = "rlib"

  deps = [
    "//third_party/rust/crates/syn:lib",
  ]

  sources = [ "src/lib.rs" ]
}
```

### 3.7.2 Features 选择建议

**如果只需要部分功能**：
```gn
# 只需要解析 derive 宏输入
features = [
  "derive",
  "parsing",
]

# 需要完整功能
features = [
  "clone-impls",
  "derive",
  "extra-traits",
  "full",
  "parsing",
  "printing",
  "visit",
  "visit-mut",
]
```

**建议**：
- 对于过程宏：参考 OH 的配置，启用完整 features
- 对于简单工具：根据实际需求选择 features

---

## 3.8 构建性能

### 3.8.1 编译时间

syn 是一个较大的库，编译时间较长：
- **源码行数**：约 20,000+ 行
- **编译时间**：约 10-30 秒（取决于硬件）

**优化策略**：
- ✅ OH 选择编译为 `rlib`（静态库），可被多个 crate 复用
- ✅ 使用 Rust 2021 edition，利用最新编译器优化
- ⚠️ 启用所有 features 会增加编译时间（但可接受）

### 3.8.2 二进制大小

syn 作为静态库，会被链接到使用它的 crate 中：
- **rlib 大小**：约 1-2 MB（未优化）
- **LTO 优化后**：约 0.5-1 MB

**影响**：
- 由于 syn 是基础设施，大部分 crate 都会链接它
- 使用 LTO（Link Time Optimization）可以显著减小最终二进制大小

---

## 3.9 常见问题

### Q1：为什么 syn 不需要配置 `cflags`？

**A**：因为 syn 是纯 Rust 库：
- 没有 C/C++ 代码
- 不需要特殊编译标志
- 使用标准的 Rust edition 2021

### Q2：为什么启用所有 features？

**A**：
- OH 的过程宏需要完整功能
- 避免遗漏，简化依赖管理
- 编译时间增加可接受（基础设施库）

### Q3：如何减小 syn 的编译时间？

**A**：
- 使用更快的硬件
- 启用编译缓存（OH 构建系统已支持）
- 如果只需要部分功能，减少 features（不推荐）

### Q4：OH 会不会未来添加 syn 的 Patch？

**A**：可能性很低：
- syn 是纯逻辑库，无平台相关代码
- 社区活跃，上游维护良好
- 添加 Patch 会增加维护成本

### Q5：如何跟踪 syn 的版本？

**A**：
- bundle.json 中指定版本
- 定期检查上游更新
- 参考 [安全风险分析](06_Security.md) 的升级建议

---

## 3.10 总结

### 核心要点

1. **BUILD.gn 配置**：使用 `ohos_cargo_crate` 模板，配置完整 features
2. **Features 策略**：启用所有主要 features，提供完整功能
3. **依赖管理**：依赖 proc-macro2、quote、unicode-ident
4. **与上游差异**：仅构建系统不同，源码完全一致
5. **特殊处理**：无 OH 特定代码，无需 Patch

### 最佳实践

✅ **DO**：
- 使用 `ohos_cargo_crate` 模板
- 显式启用所有需要的 features
- 保持版本与上游一致
- 定期检查上游更新

❌ **DON'T**：
- 不要添加 OH 特定代码
- 不要随意禁用 features
- 不要手动修改源码
- 不要跳过版本兼容性检查

---

## 3.11 相关文档

- [原始库简介](01_Overview.md) - syn 的功能介绍
- [Patch 详细分析](02_Patches.md) - 无 Patch 的原因
- [依赖关系与使用](04_Usage_in_OH.md) - 谁在使用 syn
- [安全风险分析](06_Security.md) - 升级建议

---

**文档最后更新**：2026-02-08
**适用版本**：syn 2.0.48
**构建模板**：ohos_cargo_crate
