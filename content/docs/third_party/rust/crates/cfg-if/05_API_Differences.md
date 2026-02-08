# API/接口差异

## 差异概述

**结论：该库在 OpenHarmony 中没有任何 API 或接口差异。**

`cfg-if` 是一个纯宏定义的库，其 API 非常简单且稳定，在 OpenHarmony 中的使用方式与上游完全一致。

## 核心 API

### cfg_if! 宏

该库只提供一个核心 API：`cfg_if!` 宏。

```rust
// 上游定义
#[macro_export]
macro_rules! cfg_if {
    // ... 宏实现
}

// OH 使用方式（完全相同）
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(condition)] {
        // 代码分支 1
    } else if #[cfg(condition2)] {
        // 代码分支 2
    } else {
        // 默认分支
    }
}
```

## API 一致性分析

### 对比表

| API 元素 | 上游行为 | OH 行为 | 差异 |
|---------|----------|---------|------|
| `cfg_if!` 宏 | 可用 | 可用 | 无差异 |
| 宏参数语法 | `if #[cfg(...)] {} else if ... else {}` | 相同 | 无差异 |
| 嵌套支持 | 支持 | 支持 | 无差异 |
| `#[cfg]` 条件支持 | 所有标准条件 | 相同 | 无差异 |
| OH 特有条件 | 不支持 | 不支持 | 无差异（该库本身不处理 OH 特有逻辑） |

### 详细说明

#### 1. 宏签名一致性

```rust
// 上游代码
#[macro_export]
macro_rules! cfg_if {
    ($(
        if #[cfg($meta:meta)] { $($tokens:tt)* }
    ) else * else {
        $($tokens2:tt)*
    }) => { /* ... */ };
}

// OH 代码（完全相同）
// 无任何修改
```

#### 2. 条件支持一致性

| 条件类型 | 上游支持 | OH 支持 | 说明 |
|---------|---------|---------|------|
| `target_os` | ✅ | ✅ | 平台检测 |
| `target_arch` | ✅ | ✅ | 架构检测 |
| `target_family` | ✅ | ✅ | 系统族检测 |
| `target_pointer_width` | ✅ | ✅ | 指针宽度检测 |
| `feature` | ✅ | ✅ | 特性检测 |
| `debug_assertions` | ✅ | ✅ | 调试模式检测 |
| `test` | ✅ | ✅ | 测试模式检测 |
| `target_os = "ohos"` | ✅ | ✅ | OpenHarmony 平台检测 |

## 使用方式对比

### 基本用法（完全相同）

```rust
// 上游示例
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(unix)] {
        fn run() { /* Unix */ }
    } else {
        fn run() { /* Other */ }
    }
}

// OH 使用（完全相同）
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(unix)] {
        fn run() { /* Unix */ }
    } else {
        fn run() { /* Other */ }
    }
}
```

### 复杂条件（完全相同）

```rust
// 上游示例
cfg_if! {
    if #[cfg(all(target_os = "linux", feature = "ssl"))] {
        use openssl;
    } else if #[cfg(all(target_os = "windows", feature = "ssl"))] {
        use win_crypto;
    }
}

// OH 使用（完全相同）
cfg_if! {
    if #[cfg(all(target_os = "linux", feature = "ssl"))] {
        use openssl;
    } else if #[cfg(all(target_os = "windows", feature = "ssl"))] {
        use win_crypto;
    }
}
```

## OH 特有用法说明

### OpenHarmony 平台检测

由于该库没有任何 OH 特有修改，检测 OpenHarmony 平台需要依赖 Rust 编译器的内置支持：

```rust
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(target_os = "ohos")] {
        fn platform_init() {
            // OpenHarmony 特定初始化
            println!("Initializing for OpenHarmony");
        }
    } else {
        fn platform_init() {
            // 其他平台
            println!("Initializing for generic platform");
        }
    }
}
```

### 自定义 OH 特性

如需添加 OH 特有的条件编译支持，可以通过 Cargo.toml 特性：

```toml
# Cargo.toml
[features]
ohos-extended = []
```

```rust
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(target_os = "ohos")] {
        cfg_if::cfg_if! {
            if #[cfg(feature = "ohos-extended")] {
                fn ohos_feature() { /* 扩展功能 */ }
            } else {
                fn ohos_feature() { /* 基础功能 */ }
            }
        }
    }
}
```

## 导出内容对比

### 上游导出

```rust
// src/lib.rs
#[macro_export]
macro_rules! cfg_if { /* ... */ }
```

### OH 导出（完全相同）

```rust
// OH 中通过 BUILD.gn 导出
// inner_kits 配置指向 :lib 目标
// 导出内容与上游完全一致
```

## 文档与注释

### 上游文档

```rust
//! A macro for defining `#[cfg]` if-else statements.
//!
//! The macro provided by this crate, `cfg_if`, is similar to the `if/elif` C
//! preprocessor macro by allowing definition of a cascade of `#[cfg]` cases,
//! emitting the implementation which matches first.
```

### OH 文档（完全相同）

该库的文档注释与上游完全一致，OH 侧的文档由 `README.md` 和源码注释提供。

## 迁移指南

### 从上游迁移到 OH

**无需任何迁移工作**。上游代码可以直接在 OH 中使用。

```rust
// ✅ 上游代码，可以直接在 OH 中使用
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(feature = "my-feature")] {
        // 功能实现
    }
}
```

### 从 OH 迁移到其他平台

**无需任何修改**。OH 中的 `cfg-if` 使用方式与其他平台完全一致。

```rust
// ✅ OH 代码，可以直接在其他平台使用
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(target_os = "ohos")] {
        // OH 特定代码
    } else {
        // 其他平台代码
    }
}
```

## 总结

| 维度 | 结论 |
|------|------|
| **API 完整性** | 100% 完整，无缺失 |
| **行为一致性** | 100% 一致，无行为差异 |
| **用法差异** | 无差异，完全相同 |
| **OH 特有 API** | 无（该库定位为通用库） |
| **迁移成本** | 零成本 |

该库在 OpenHarmony 中的 API 使用与上游完全一致，是 OH 中适配成本最低的 Rust 第三方库之一。
