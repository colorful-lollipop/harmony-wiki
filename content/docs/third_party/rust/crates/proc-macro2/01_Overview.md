# 原始库概述

## 基本信息

| 项目 | 值 |
|------|-----|
| **库名称** | proc-macro2 |
| **上游地址** | https://github.com/dtolnay/proc-macro2 |
| **当前版本** | 1.0.92 |
| **许可证** | MIT OR Apache-2.0 |
| **最低 Rust 版本** | 1.56 |
| **维护者** | David Tolnay, Alex Crichton |

## 原始功能

proc-macro2 是一个 Rust 库，为编译器 `proc_macro` crate 提供替代实现。其核心功能包括:

### 1. Token 和 TokenStream 类型

提供 `TokenStream`、`TokenTree`、`Span`、`Punct`、`Ident`、`Literal` 等类型的替代实现，这些类型可以在**非过程宏代码**中使用。

### 2. 过程宏支持

支持在 `build.rs`、`main.rs` 等非宏上下文中使用类似过程宏的功能。

### 3. 位置信息

通过 `span-locations` feature 提供 token 的行/列位置信息，用于代码诊断和错误报告。

### 4. 与 syn/quote 生态集成

作为 Rust 过程宏生态的基础层，被以下核心库使用:

- **syn**: Rust 代码解析库
- **quote**: Rust 代码生成库
- **proc-macro-error**: 过程宏错误处理

## 在 OpenHarmony 中的定位

### 依赖链位置

```
proc-macro2 (基础设施层)
    │
    ├── quote ───→ syn ───→ bindgen
    │
    ├── cxx (C++/Rust FFI)
    │
    ├── serde_derive (序列化 derive)
    │
    └── openssl-macros (OpenSSL 派生宏)
```

### 作用

proc-macro2 在 OH Rust 生态中扮演**基础设施**角色:

1. **所有 Rust 派生宏（derive）的基础**: serde_derive、openssl_derive 等
2. **代码生成工具的核心依赖**: bindgen、cxx-code-gen 等
3. **过程宏测试支持**: 使得过程宏的单元测试成为可能

### 为什么 OH 需要它

Rust 生态的许多核心工具（bindgen、serde、cxx 等）都依赖 proc-macro2 来处理 Token 和生成代码。这些工具在 OH 中用于:

- 绑定 C/C++ 库（bindgen）
- 序列化框架（serde）
- C++/Rust FFI（cxx）
- 命令行工具（clap）

## 版本信息

| 版本 | 发布日期 | 变更类型 |
|------|----------|----------|
| 1.0.92 | 2024年 | 当前 OH 版本 |
| 1.0.91 | 2024年 | 补丁版本 |
| 1.0.90 | 2024年 | 补丁版本 |

## 与上游差异

| 方面 | 状态 |
|------|------|
| **源代码修改** | 无差异 |
| **API 变更** | 无 |
| **新增功能** | 无 |
| **移除功能** | 无 |

**结论**: OH 使用的 proc-macro2 与上游版本完全一致，未进行任何修改。
