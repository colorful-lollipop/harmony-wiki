# 01 - 原始库简介

## 1.1 基本信息

| 属性 | 内容 |
|------|------|
| **库名称** | rust-cexpr |
| **crate 名称** | cexpr |
| **版本** | 0.6.0 |
| **作者** | Jethro Beekman |
| **许可证** | Apache-2.0 / MIT (双许可) |
| **上游仓库** | https://github.com/jethrogb/rust-cexpr |
| **文档** | https://docs.rs/cexpr/0.6.0/cexpr/ |

## 1.2 原始功能

rust-cexpr 是一个用于解析和求值 **C 语言常量表达式** 的 Rust 库。

### 支持的功能

| 功能类型 | 具体支持 |
|---------|---------|
| **数值类型** | 整数 (i64)、浮点数 (f64) |
| **字符类型** | C 字符常量 |
| **字符串类型** | C 字符串常量 |
| **算术运算** | `+`, `-`, `*`, `/`, `%` |
| **位运算** | `&`, `\|`, `^`, `~<<`, `>>` |
| **一元运算** | `+`, `-`, `~` |
| **括号分组** | `(...)` |
| **字符串拼接** | `"str1" "str2"` → `"str1str2"` |

### 不支持的功能

| 功能 | 说明 |
|------|------|
| `sizeof` 运算符 | 设计限制 |
| 类型转换 | 不支持 C 的类型转换语法 |
| 函数调用 | 非设计目标 |
| 复杂表达式 | 仅限于常量表达式 |

## 1.3 架构设计

### 模块结构

```
cexpr/
├── lib.rs       # 入口，错误类型定义
├── token.rs     # Token 定义（对应 libclang CXToken）
├── expr.rs      # 表达式解析和求值
└── literal.rs   # 字面量解析
```

### 核心 API

```rust
// 1. Token 表示
pub struct Token {
    pub kind: Kind,      // Punctuation, Keyword, Identifier, Literal, Comment
    pub raw: Box<[u8]>,  // 原始字节内容
}

// 2. 表达式求值结果
pub enum EvalResult {
    Int(Wrapping<i64>),
    Float(f64),
    Char(CChar),
    Str(Vec<u8>),
    Invalid,
}

// 3. 标识符解析器
pub struct IdentifierParser<'ident> {
    identifiers: &'ident HashMap<Vec<u8>, EvalResult>,
}
```

### 依赖关系

```
rust-cexpr
    └── nom 7.x (parser combinator 框架)
```

## 1.4 上游版本历史

| 版本 | 发布日期 | 主要变更 |
|------|---------|---------|
| 0.6.0 | 2022-03 | 升级 nom 到 7.x，API 稳定 |
| 0.5.0 | 2021-06 | nom 6.x 支持 |
| 0.4.0 | 2020-05 | 初始稳定版本 |

## 1.5 该库在 OpenHarmony 中的作用

### 核心定位

rust-cexpr 在 OpenHarmony 中的角色是 **bindgen 的基础设施库**，不直接面向应用开发者。

### 为什么需要它？

bindgen 自动生成 Rust FFI 绑定，需要解析 C/C++ 头文件中的宏定义：

```c
// C 头文件中的宏
#define BUFFER_SIZE 1024
#define VERSION "1.0.0"
#define FLAG_A 0x01
#define FLAG_B 0x02
#define FLAGS (FLAG_A | FLAG_B)
```

bindgen 使用 rust-cexpr 将这些宏转换为 Rust 代码：

```rust
// 生成的 Rust 代码
pub const BUFFER_SIZE: u32 = 1024;
pub const VERSION: &[u8; 6] = b"1.0.0\0";
pub const FLAG_A: u32 = 1;
pub const FLAG_B: u32 = 2;
pub const FLAGS: u32 = 3;
```

### 工作流程

```
C/C++ 头文件
    ↓
libclang 解析 → CXToken 序列
    ↓
bindgen/clang.rs → cexpr::token::Token
    ↓
cexpr::expr::IdentifierParser 求值
    ↓
cexpr::expr::EvalResult
    ↓
bindgen/ir/var.rs → Rust 常量定义
```

## 1.6 OpenHarmony 版本历史

| OH 版本 | cexpr 版本 | 主要变更 |
|---------|-----------|---------|
| 6.1 | 0.6.0 | 当前版本 |
| 早期 | 0.5.x | 初始导入版本 |

### OH 维护记录

1. **初始导入** (`f6bcd8f`)
   - 添加 GN 构建文件
   - 集成到 OH 构建系统

2. **版本升级** (`67c3a3e`)
   - 从早期版本升级到 0.6.0
   - 适配 nom 7.x

3. **部件化** (`432b77e`)
   - 添加 bundle.json
   - 纳入 OH 部件管理体系

## 1.7 技术特点总结

| 特点 | 说明 |
|------|------|
| **代码规模** | 小（约 800 行） |
| **依赖数量** | 单一（仅 nom） |
| **功能边界** | 清晰（仅表达式解析） |
| **API 稳定性** | 高（版本 0.6.0 后稳定） |
| **维护活跃度** | 中（功能完整，维护模式） |

---

*本文档描述原始库功能，OH 特有的适配内容见其他文档*
