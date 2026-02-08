# heck 库概述

## 1. 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | heck |
| **上游版本** | 0.4.1 |
| **上游地址** | https://github.com/withoutboats/heck |
| **许可证** | Apache-2.0 OR MIT (双许可) |
| **Rust Edition** | 2018 |
| **MSRV** | 1.32.0 |

## 2. 原始功能简介

heck 是一个**字符串大小写转换库**，用于在不同命名规范之间转换字符串。例如：

```rust
use heck::ToKebabCase;

assert_eq!("HelloWorld".to_kebab_case(), "hello-world");
assert_eq!("my_field_name".to_kebab_case(), "my-field-name");
```

### 2.1 支持的命名规范（共 8 种）

| 命名规范 | 示例输入 | 示例输出 |
|----------|----------|----------|
| `UpperCamelCase` / `PascalCase` | `hello_world` | `HelloWorld` |
| `lowerCamelCase` | `hello_world` | `helloWorld` |
| `snake_case` | `HelloWorld` | `hello_world` |
| `kebab-case` | `HelloWorld` | `hello-world` |
| `SHOUTY_SNAKE_CASE` | `HelloWorld` | `HELLO_WORLD` |
| `Title Case` | `hello_world` | `Hello World` |
| `SHOUTY-KEBAB-CASE` | `HelloWorld` | `HELLO-WORLD` |
| `Train-Case` | `hello_world` | `Hello-World` |

### 2.2 词边界定义

heck 定义了智能的词边界规则：

1. **下划线**始终视为词边界
2. **大写字母后跟小写字母**时，边界在大写字母前（`Hello|World`）
3. **连续大写字母**视为一个词，最后一个可归入下一个词（`XML|Http|Request`）

```
"HelloWorld"      →  Hello|World
"XMLHttpRequest"  →  XML|Http|Request
"hello__world"    →  hello|world （多个下划线合并）
```

### 2.3 API 设计

每个命名规范提供两种 Trait：

| Trait 类型 | 方法 | 用途 |
|------------|------|------|
| `ToXxxCase` | `to_xxx_case()` | 拥有字符串，返回转换后的 `String` |
| `AsXxxCase` | `as_xxx_case()` | 借用包装，返回实现了 `Display` 的结构体 |

```rust
use heck::{ToSnakeCase, AsSnakeCase};

// ToSnakeCase - 返回 String
let s = "HelloWorld".to_snake_case();  // "hello_world"

// AsSnakeCase - 返回包装器，零分配格式化
println!("{}", "HelloWorld".as_snake_case());  // "hello_world"
```

## 3. 在 OpenHarmony 中的作用和定位

### 3.1 角色定位

在 OH 生态中，heck 是一个**底层基础设施库**：

```
应用程序层
    ↓ 使用
clap (命令行解析，带 derive 特性)
    ↓ 依赖
clap_derive (过程宏 crate)
    ↓ 依赖
heck (字符串命名转换)
```

### 3.2 主要使用场景

heck 在 OH 中的核心用途是支持 **clap_derive** 宏：

1. **命令行参数映射**：将 `--my-argument` 转换为 Rust 结构体字段 `my_argument`
2. **宏属性处理**：处理 `#[clap(name = "xxx")]` 等属性中的命名转换
3. **代码生成**：为 derive 宏生成符合 Rust 命名规范的代码

### 3.3 为何不直接使用上游？

OH 使用第三方 crate 的标准化流程：

1. **版本锁定**：固定 0.4.1 版本，确保构建可重现
2. **源码内嵌**：嵌入 `third_party/rust/crates/heck`，支持离线构建
3. **构建统一**：通过 BUILD.gn 与 OH 构建系统集成

## 4. 版本历史与变更

### 4.1 上游版本演进

| 版本 | 重要变更 |
|------|----------|
| 0.4.1 | 新增 Train-Case 支持 |
| 0.4.0 | **破坏性变更**：Unicode 变为可选特性；trait 重命名 |
| 0.3.x | 旧版本 API |

### 4.2 v0.4.0 破坏性变更详情

**Trait 重命名**（符合 Rust 标准库命名约定）：
- `SomeCase` → `ToSomeCase`（以动词开头）
- `ToMixedCase` → `ToLowerCamelCase`
- `ToCamelCase` → `ToUpperCamelCase`
- 新增 `ToPascalCase` 别名（`ToUpperCamelCase` 的别名）

**Unicode 特性变化**：
- Unicode 支持变为**可选功能**（默认关闭）
- 需要 Unicode 时启用 `unicode` 特性

### 4.3 OH 当前状态

- **当前版本**: 0.4.1
- **Unicode 特性**: 未启用
- **兼容性**: 与 clap_derive 4.1.12 兼容

## 5. 技术特性

### 5.1 安全性

```rust
#![forbid(unsafe_code)]
```

heck **完全禁止 unsafe 代码**，确保内存安全。

### 5.2 依赖关系

- **外部依赖**: 0（默认特性）
- **可选依赖**: `unicode-segmentation`（启用 `unicode` 特性时）

### 5.3 性能特点

- **零拷贝格式化**：`AsXxxCase` 类型避免分配
- **迭代器处理**：单次遍历完成转换
- **Unicode 可选**：不启用时仅使用 ASCII 处理，性能更优

## 6. 相关资源

- **Crates.io**: https://crates.io/crates/heck
- **文档**: https://docs.rs/heck/0.4.1/heck/
- **源码**: https://github.com/withoutboats/heck
- **CHANGELOG**: [CHANGELOG.md](../CHANGELOG.md)
