# 概述

## 原始库信息

| 项目 | 内容 |
|------|------|
| **库名称** | cfg-if |
| **版本** | 1.0.0 |
| **上游地址** | https://github.com/rust-lang/cfg-if |
| **许可证** | Apache License V2.0, MIT |
| **作者** | Alex Crichton |

## 功能简介

`cfg-if` 是一个轻量级的 Rust 条件编译宏库，提供了 `cfg_if!` 宏来简化基于 `#[cfg]` 属性的条件编译逻辑。该宏的结构类似于 C 语言的 `if/elif` 预处理器语句，允许定义一系列条件分支，编译器只会发射第一个匹配分支的代码。

## 核心功能

### cfg_if! 宏

`cfg_if!` 宏的主要特性：

1. **链式条件**：支持多个 `#[cfg]` 条件的级联判断
2. **自动互斥**：确保只有一个分支被编译
3. **嵌套支持**：可以在函数、结构体、impl 块等任何作用域中使用
4. **纯宏实现**：无需运行时开销，完全在编译期处理

## 使用示例

### 基本用法

```rust
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(unix)] {
        fn get_error_code() -> i32 { 1 }
    } else if #[cfg(windows)] {
        fn get_error_code() -> i32 { 2 }
    } else {
        fn get_error_code() -> i32 { 0 }
    }
}
```

### 平台检测

```rust
cfg_if! {
    if #[cfg(target_os = "linux")] {
        const PLATFORM: &str = "Linux";
    } else if #[cfg(target_os = "android")] {
        const PLATFORM: &str = "Android";
    } else if #[cfg(target_os = "ohos")] {
        const PLATFORM: &str = "OpenHarmony";
    } else if #[cfg(target_os = "windows")] {
        const PLATFORM: &str = "Windows";
    } else if #[cfg(target_os = "macos")] {
        const PLATFORM: &str = "macOS";
    } else {
        const PLATFORM: &str = "Unknown";
    }
}
```

### 特性条件

```rust
cfg_if! {
    if #[cfg(feature = "derive")] {
        use some_crate::derive_macro;
    } else if #[cfg(feature = "serde")] {
        use serde::{Serialize, Deserialize};
    } else {
        use plain_data_type;
    }
}
```

## 在 OpenHarmony 中的作用

### 定位

`cfg-if` 在 OpenHarmony Rust 生态系统中扮演**基础设施**角色，作为其他 Rust 库的底层依赖，提供统一的条件编译抽象。

### 核心价值

1. **简化条件编译**：为复杂的多平台代码提供清晰的语法
2. **提升代码可读性**：避免嵌套的 `#[cfg]` 属性和重复的条件判断
3. **标准化模式**：在 OH 的 Rust 第三方库中建立统一的条件编译风格

### 典型使用场景

- 平台检测（`target_os`、`target_arch`）
- 特性条件编译（`feature`）
- 调试与发布模式区分
- 不同 Rust 版本兼容性

## 与其他库的关系

```
用户代码
    ↓ 依赖
log、openssl、libloading、nix 等库
    ↓ 依赖
cfg-if（基础设施层）
```

`cfg-if` 位于依赖链的底层，被多个核心 Rust 库依赖，体现了其基础设施的重要性。

## 进一步阅读

- [构建集成](03_Build_Integration.md) - 了解如何在 OH 中构建
- [在 OH 中的使用](04_Usage_in_OH.md) - 查看依赖关系
- [上游文档](https://docs.rs/cfg-if) - 完整的 API 文档
