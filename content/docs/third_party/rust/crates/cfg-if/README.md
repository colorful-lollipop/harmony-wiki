# cfg-if 第三方库 Wiki

## 库概览

| 项目 | 内容 |
|------|------|
| **库名称** | cfg-if |
| **版本** | 1.0.0 |
| **许可证** | Apache License V2.0, MIT |
| **上游地址** | https://github.com/rust-lang/cfg-if |
| **OH 组件名称** | rust_cfg_if |
| **OH 版本号** | 6.1 |

## 什么是 cfg-if？

`cfg-if` 是一个 Rust 语言的条件编译宏库，提供了 `cfg_if!` 宏用于根据 `#[cfg]` 属性定义一系列条件分支。该宏允许开发者方便地提供一长串 `#[cfg]` 条件的代码块，而无需多次重写每个子句。

## OpenHarmony 适配概述

**该库在 OpenHarmony 中无需任何 Patch 即可正常工作。**

### 适配特点

1. **零 Patch**：该库是纯宏定义库，功能完全基于 Rust 编译器的 `#[cfg]` 属性，在所有平台上行为一致
2. **标准集成**：使用 `ohos_cargo_crate` 模板进行构建集成
3. **核心依赖**：被 `log`、`rust-openssl`、`libloading`、`nix` 等核心 Rust 库依赖

## 文档导航

### 快速开始

- [01_概述](01_Overview.md) - 原始库功能介绍
- [03_构建集成](03_Build_Integration.md) - BUILD.gn 配置说明

### 详细分析

- [02_Patch 分析](02_Patches.md) - Patch 文件说明（本库无 Patch）
- [04_在 OH 中的使用](04_Usage_in_OH.md) - 依赖关系和使用场景
- [05_API 差异](05_API_Differences.md) - API 差异分析（本库无差异）
- [06_安全风险分析](06_Security.md) - 安全评估

### 阅读路线建议

```
新手入门 → README.md → 01_Overview.md → 03_Build_Integration.md → 04_Usage_in_OH.md
深入了解 → 02_Patches.md → 05_API_Differences.md → 06_Security.md
```

## 快速参考

### OH 构建引用方式

```gn
deps = [ "//third_party/rust/crates/cfg-if:lib" ]
```

### Rust 代码使用方式

```rust
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(unix)] {
        fn platform_specific() { /* Unix 平台实现 */ }
    } else if #[cfg(target_pointer_width = "32")] {
        fn platform_specific() { /* 32位平台实现 */ }
    } else {
        fn platform_specific() { /* 默认实现 */ }
    }
}
```

## 相关资源

- [上游仓库](https://github.com/rust-lang/cfg-if)
- [Rust 文档](https://docs.rs/cfg-if)
- [OH bundle.json](../../bundle.json)
- [OH BUILD.gn](../../BUILD.gn)

---

**文档版本**：1.0  
**最后更新**：2024 年  
**维护者**：fangting12@huawei.com
