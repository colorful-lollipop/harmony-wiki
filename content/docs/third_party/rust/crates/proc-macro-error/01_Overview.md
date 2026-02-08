# 原始库介绍

## 1.1 库基本信息

| 项目 | 信息 |
|-----|------|
| **库名称** | proc-macro-error |
| **上游地址** | https://gitlab.com/CreepySkeleton/proc-macro-error |
| **当前版本** | 1.0.4 |
| **许可证** | MIT OR Apache-2.0 |
| **作者** | CreepySkeleton <creepy-skeleton@yandex.ru> |
| **支持 Rust 版本** | 1.31+ |

## 1.2 库的功能定位

### 解决的问题

在 Rust 过程宏（Procedural Macro）开发中，错误处理一直是一个痛点。传统方法有两种：

| 方法 | 问题 |
|-----|------|
| `panic!` | 无法携带 span（代码位置）信息，用户无法定位错误 |
| `compile_error!` | 需要手动 unwrap Result 代码，冗长且繁琐 |

### proc-macro-error 的解决方案

该库提供了一套**友好且统一的错误报告 API**：

```rust
// 类似 panic! 的语法，但携带 span 信息
abort!(error_span, "Invalid input: {}", msg);

// 支持多个错误信息
emit_error!(span, "Error: {}", msg; note = "Did you mean...?");

// 批量收集和报告错误
abort_if_dirty();  // 如果有错误则立即终止
```

## 1.3 核心功能

### 错误报告宏

| 宏/函数 | 功能描述 |
|-------|---------|
| `abort!` | 立即终止宏执行，报告错误（携带 span 信息） |
| `abort_call_site!` | 在宏调用点报告错误（不携带具体 span） |
| `emit_error!` | 收集错误，不立即终止（可收集多个错误） |
| `emit_warning!` | 发出警告（仅夜间版 Rust 支持，稳定版忽略） |
| `abort_if_dirty!` | 如果有已收集的错误，立即终止 |
| `abort_call_site!` | 在调用点报告错误 |

### 示例代码

#### Panic-like 用法

```rust
use proc_macro_error::{proc_macro_error, abort, abort_call_site};

#[proc_macro]
#[proc_macro_error]
pub fn my_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);

    // 解析失败，使用 abort! 报告错误
    if let Err(err) = parse_attributes(&input.attrs) {
        abort!(err.span(), "Failed to parse attributes: {}", err);
    }

    // 业务逻辑验证
    if !validate_fields(&input.data) {
        // 没有具体位置信息，使用 abort_call_site!
        abort_call_site!("Field validation failed");
    }

    quote!(/* ... */).into()
}
```

#### Diagnostic-like 用法

```rust
use proc_macro_error::*;

#[proc_macro]
#[proc_macro_error]
pub fn complex_macro(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as ItemStruct);

    // 收集多个错误
    for attr in &input.attrs {
        if let Err(msg) = validate_attr(attr) {
            emit_error!(attr, "Invalid attribute: {}", msg);
        }
    }

    // 处理字段
    for field in &input.fields {
        if let Some(msg) = check_field(field) {
            emit_error!(field, "Field error: {}", msg);
        }
    }

    // 检查是否有错误，有则终止
    abort_if_dirty!();

    // 没有错误，正常返回
    quote!(/* ... */).into()
}
```

### Span 信息支持

该库的核心价值在于**保留并传递代码位置信息**：

| Rust 版本 | 行为 |
|----------|------|
| 稳定版 | 使用 `compile_error!` + span 消息 |
| 夜间版 | 优先使用 `proc_macro::Diagnostic`，支持更丰富的错误格式 |

```rust
// 这段代码在不同 Rust 版本上的表现

abort!(token_span, 
    "Unexpected token: expected identifier"; 
    note = "Check the token type";
    help = "Did you forget a semicolon?");
```

**效果对比**：

| 场景 | 显示效果 |
|-----|---------|
| 稳定版 | 显示错误消息 + 位置下划线 |
| 夜间版 | 显示错误消息 + 位置高亮 + 辅助提示 |

## 1.4 依赖与特性

### Cargo 依赖

```toml
[dependencies]
proc-macro-error = "1.0"
```

### 可选依赖

| 依赖 | 用途 | 特性 |
|-----|------|-----|
| `syn` | 解析 Rust 代码结构 | `syn-error` (默认启用) |
| `proc-macro2` | TokenStream 抽象 | 默认包含 |
| `quote` | 代码生成 | 默认包含 |

### Cargo Features

| 特性 | 默认 | 说明 |
|-----|------|-----|
| `syn-error` | ✅ | 启用 `syn` 依赖，提供 `From<syn::Error>` 实现 |
| (无) | - | 仅使用 `syn::Error` 的基本功能 |

```toml
# 禁用 syn 特性
[dependencies]
proc-macro-error = { version = "1.0", default-features = false }
```

## 1.5 与 proc_macro::Diagnostic 的关系

### 设计哲学

该库的 API 设计**兼容 `proc_macro::Diagnostic`**：

```
用户代码 
    ↓
proc_macro_error (shim 层)
    ↓
稳定版: compile_error! + span
夜间版: proc_macro::Diagnostic
```

### 未来迁移

当 `proc_macro::Diagnostic` 正式稳定后：

1. 该库的 API 保持不变
2. 底层实现会自动切换到标准库
3. 用户代码无需修改

## 1.6 典型使用场景

### 场景 1：自定义 Derive 宏

```rust
// serde_derive, validator_derive 等类似
#[proc_macro_derive(MyDerive)]
pub fn derive_my_feature(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);

    if !validate_derive_attrs(&input.attrs) {
        abort_call_site!("Missing required #[my_attr]");
    }

    quote!(/* 生成代码 */).into()
}
```

### 场景 2：属性宏

```rust
#[proc_macro_attribute]
#[proc_macro_error]
pub fn my_attr(args: TokenStream, input: TokenStream) -> TokenStream {
    let item = syn::parse2::<ItemFn>(input).unwrap();

    if args.is_empty() {
        emit_error!(args, "Expected arguments");
    }

    quote!(/* 修改后的函数 */).into()
}
```

### 场景 3：函数式宏

```rust
#[proc_macro]
#[proc_macro_error]
pub fn my_func_macro(input: TokenStream) -> TokenStream {
    let tokens = TokenStream2::from(input).into_iter().collect::<Vec<_>>();

    if tokens.len() < 2 {
        abort!(proc_macro::Span::call_site(), "Need at least 2 arguments");
    }

    quote!(/* 生成代码 */).into()
}
```

## 1.7 已知限制

| 限制 | 说明 | 影响 |
|-----|------|-----|
| 警告仅夜间版 | `emit_warning!` 在稳定版被忽略 | 调试时需使用夜间版 |
| Help span | 稳定版上 help 消息无法携带独立 span | 影响 IDE 显示效果 |
| Panic 捕获 | 过程宏 panic 时不显示错误 | 需避免在宏内使用 panic |

## 1.8 在 OpenHarmony 中的作用

该库在 OpenHarmony Rust 生态中主要作为：

1. **开发工具库**：为 OH 开发者提供编写高质量过程宏的能力
2. **依赖传递**：被其他 Rust proc-macro crate 间接依赖
3. **错误体验**：帮助生成更好的编译错误信息，提升开发者体验

**定位**：开发工具层 → 支持 Rust 过程宏开发
