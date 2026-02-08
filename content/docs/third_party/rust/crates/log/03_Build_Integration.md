# OH 构建适配

## 构建配置概览

log 库在 OpenHarmony 中的构建配置极其简洁，体现了其 **零修改适配** 的特点。

### BUILD.gn 配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
  crate_name = "log"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2015"
  cargo_pkg_version = "0.4.17"
  cargo_pkg_authors = "The Rust Project Developers"
  cargo_pkg_name = "log"
  features = [ "std" ]
  deps = [ "//third_party/rust/crates/cfg-if:lib" ]
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  module_output_extension = ".rlib"
  part_name = "rust_log"
  subsystem_name = "thirdparty"
}
```

## 配置详解

### 核心字段说明

| 字段 | 值 | 说明 |
|-----|-----|-----|
| `crate_name` | "log" | 库名称，与 Cargo.toml 一致 |
| `crate_type` | "rlib" | Rust 静态库，用于静态链接 |
| `crate_root` | "src/lib.rs" | 库入口文件 |
| `edition` | "2015" | Rust 2015 edition |
| `cargo_pkg_version` | "0.4.17" | 与上游版本同步 |
| `features` | ["std"] | **仅启用 std 特性** |
| `deps` | [cfg-if] | 唯一依赖 |
| `part_name` | "rust_log" | OH 组件名 |
| `subsystem_name` | "thirdparty" | 所属子系统 |

### 特性配置说明

#### OH 启用的特性

| 特性 | 状态 | 说明 |
|-----|-----|-----|
| `std` | ✅ 启用 | 支持标准库，OH 环境满足 |
| `max_level_*` | ❌ 未配置 | 使用默认值（trace） |
| `release_max_level_*` | ❌ 未配置 | 使用默认值 |
| `kv_unstable` | ❌ 禁用 | 实验性功能，不引入 |
| `serde` | ❌ 禁用 | 可选序列化支持 |

#### 与上游 Cargo.toml 对比

```toml
# 上游 Cargo.toml - 所有可用特性
[features]
std = []
kv_unstable = ["value-bag"]
kv_unstable_sval = ["kv_unstable", ...]
kv_unstable_std = ["std", "kv_unstable", ...]
kv_unstable_serde = ["kv_unstable_std", ...]

# OH BUILD.gn - 仅启用 std
features = ["std"]
```

**决策理由**：
- `std` 是必需的（OH 环境有标准库支持）
- 其他特性为可选，禁用可减少编译时间和二进制大小
- `kv_unstable` 为实验性，不适合生产环境依赖

### 依赖配置

| 依赖 | 类型 | 用途 |
|-----|-----|-----|
| `cfg-if` | OH 内部 crate | 条件编译支持 |

#### cfg-if 依赖说明

cfg-if 是一个条件编译库，用于在不同平台启用不同的代码：

```rust
// src/lib.rs 中的典型使用
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(unix)] {
        // Unix 特定代码
    } else if #[cfg(windows)] {
        // Windows 特定代码
    }
}
```

**OH 适配**：
- cfg-if 本身支持 OHOS 平台
- log 库中使用 cfg-if 的地方在 OH 上表现正常

## 构建流程

### 标准构建流程

```
1. ohos.gni 加载
   ↓
2. ohos_cargo_crate 模板调用
   ↓
3. 读取 Cargo.toml 获取元数据
   ↓
4. 执行 build.rs（检测平台能力）
   ↓
5. 编译 src/lib.rs
   ↓
6. 生成 .rlib 静态库
   ↓
7. 打包为 OH 组件
```

### build.rs 分析

log 库的 `build.rs` 用于检测目标平台的原子操作支持：

```rust
fn target_has_atomics(target: &str) -> bool {
    match &target[..] {
        "thumbv4t-none-eabi"
        | "msp430-none-elf"
        | "riscv32i-unknown-none-elf"
        | "riscv32imc-unknown-none-elf" => false,
        _ => true,
    }
}
```

**OH 平台结果**：
- OHOS 目标平台不在禁用列表中
- 启用 `atomic_cas` 和 `has_atomics` 配置

## 构建产物

### 产物清单

| 产物 | 类型 | 用途 |
|-----|-----|-----|
| `liblog.rlib` | 静态库 | 静态链接到依赖者 |
| `liblog.so` | 动态库 | **未生成**（rlib 类型） |
| 符号文件 | .d | 链接依赖分析 |

### 模块输出

```gn
module_output_extension = ".rlib"  # 明确输出类型
```

## 与上游构建差异

| 维度 | 上游 (Cargo) | OH (BUILD.gn) |
|-----|-------------|---------------|
| 构建工具 | cargo | ohos_cargo_crate |
| 构建配置 | Cargo.toml | BUILD.gn |
| 特性控制 | Cargo features | features 字段 |
| 依赖声明 | [dependencies] | deps 字段 |
| 输出格式 | .rlib/.so | .rlib |
| 发布方式 | crates.io | OH 组件系统 |

### 差异总结

| 差异类型 | 影响 |
|---------|-----|
| 构建系统不同 | 透明（GN 模板封装） |
| 特性配置不同 | 特性子集（OH 更保守） |
| 依赖解析不同 | 透明（OH 使用内部镜像） |

**结论**：OH 构建适配对用户透明，库的行为与上游完全一致。

## 常见问题

### Q1：如何升级版本？

**步骤**：
1. 更新 `cargo_pkg_version` 为新版本号
2. 验证新版本的 Cargo.toml 兼容性
3. 测试依赖者的编译和功能

**注意事项**：
- 无需修改代码
- 确保 `features` 配置仍适用
- 检查新版本是否有安全更新

### Q2：如何添加新特性？

**修改 BUILD.gn**：
```gn
features = [ "std", "serde" ]  # 添加 serde 特性
```

**同时需要**：
- 检查 serde 依赖是否在 OH 中可用
- 评估二进制大小增加
- 确认依赖者兼容性

### Q3：构建失败排查？

**常见原因**：
1. cfg-if 版本不匹配
2. Rust 版本低于 MSRV (1.31.0)
3. 依赖循环

**排查方法**：
```bash
# 检查构建日志
hvigor --info :log@... --verbose

# 验证依赖
cargo tree -p log
```
