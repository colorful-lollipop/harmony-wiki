# proc-macro-error | OpenHarmony 第三方库文档

> Rust 过程宏错误处理库 - OpenHarmony 适配说明

## 库概览

**proc-macro-error** 是一个 Rust 语言的过程宏（Procedural Macro）错误处理库，旨在让过程宏中的错误报告变得简单且易于使用。它提供了类似 `panic!` 的语法，但能够携带完整的**代码位置信息（span）**，帮助开发者精确定位错误发生的位置。

### 核心特性

- **语法友好**：提供 `abort!`、`abort_call_site!`、`emit_error!` 等宏，语法风格类似标准 `panic!`
- **Span 信息**：错误消息可以携带精确的代码位置，支持 IDE 高亮显示
- **跨版本兼容**：自动检测 Rust 版本，在稳定版和夜间版上选择最佳错误报告方式
- **未来兼容**：API 设计兼容 `proc_macro::Diagnostic`，未来可直接迁移

### 在 OpenHarmony 中的定位

在 OpenHarmony 的 Rust 生态系统中，该库作为**开发工具库**存在，用于：

1. **提升宏开发体验**：编写自定义 derive 宏或其他过程宏时，提供更好的错误信息
2. **依赖传递**：作为其他过程宏 crate 的传递依赖，间接服务于 OH 应用开发

---

## OH 适配概述

### Patch 状态：**无**

该库在 OpenHarmony 中**没有使用任何 Patch**，原因如下：

| 原因 | 说明 |
|-----|------|
| 纯 Rust 实现 | 100% Rust 代码，无 C/C++ 平台代码 |
| 平台无关 | 过程宏在编译时执行，不涉及运行时平台差异 |
| 上游稳定 | v1.0.4 版本 API 已稳定，无 OH 特需修改 |

### 构建适配

该库通过标准的 `BUILD.gn` 文件集成到 OpenHarmony 的 Rust 构建系统：

- **主 crate**：编译为 `rlib` 静态库
- **属性 crate**：编译为 `proc-macro` 类型（供其他宏使用）
- **依赖映射**：使用 OH `third_party/rust/crates/` 中的 Rust 生态库

---

## 文档导航

| 文档 | 内容 | 适合读者 |
|-----|------|---------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 | 首次阅读 |
| [01_Overview.md](01_Overview.md) | 原始库功能介绍 | 了解库的基本用途 |
| [02_Patches.md](02_Patches.md) | Patch 分析 | 了解 OH 对库的修改 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建配置说明 | 开发者、构建维护者 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用 | 想要使用该库的开发者 |

---

## 快速信息

| 项目 | 值 |
|-----|-----|
| **上游地址** | https://gitlab.com/CreepySkeleton/proc-macro-error |
| **上游版本** | 1.0.4 |
| **OH 版本** | 6.1 |
| **许可证** | Apache 2.0 / MIT |
| **Rust 版本要求** | 1.31+ |
| **Part Name** | rust_proc_macro_error |
| **Subsystem** | thirdparty |

---

## 使用示例

### 基本用法

```rust
use proc_macro_error::{proc_macro_error, abort};

#[proc_macro]
#[proc_macro_error]
pub fn my_macro(input: TokenStream) -> TokenStream {
    let parsed = parse_input(input);
    
    if let Err(e) = validate(&parsed) {
        // 错误会携带 span 信息，高亮显示问题位置
        abort!(e, "Validation failed: {}", e.message());
    }
    
    // ... 处理逻辑
    quote!(/* ... */).into()
}
```

### 错误消息效果

```rust
// 在 stable Rust 上的效果
abort!(span, "Expected identifier, found keyword"; 
    note = "Check the syntax reference");
```

---

## 相关资源

- **上游文档**: https://docs.rs/proc-macro-error
- **上游仓库**: https://gitlab.com/CreepySkeleton/proc-macro-error
- **OH Rust crates**: /third_party/rust/crates/
