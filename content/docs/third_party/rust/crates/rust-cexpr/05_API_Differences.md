# 05 - API/接口差异

## 5.1 概述

### 结论：无 API 差异

rust-cexpr 在 OpenHarmony 中 **完全保持上游 API 不变**，没有任何添加、修改或废弃的 API。

这是该库的一个重要特点：**零侵入集成**。

---

## 5.2 上游 API 完整列表

### 公共模块

```rust
// lib.rs
pub mod nom;       // nom 错误类型重导出
pub mod expr;      // 表达式解析和求值
pub mod literal;   // 字面量解析
pub mod token;     // Token 定义
```

### token 模块

```rust
// token.rs
#[derive(Debug, Copy, Clone, PartialEq, Eq)]
pub enum Kind {
    Punctuation,
    Keyword,
    Identifier,
    Literal,
    Comment,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Token {
    pub kind: Kind,
    pub raw: Box<[u8]>,
}

impl From<(Kind, &[u8])> for Token { ... }
pub fn remove_comments(v: &mut Vec<Token>) -> &mut Vec<Token>;
```

### expr 模块

```rust
// expr.rs
pub struct IdentifierParser<'ident> { ... }
pub type CResult<'a, R> = IResult<&'a [Token], R, crate::Error<&'a [Token]>>;

#[derive(Debug, Clone, PartialEq)]
pub enum EvalResult {
    Int(Wrapping<i64>),
    Float(f64),
    Char(CChar),
    Str(Vec<u8>),
    Invalid,
}

impl EvalResult {
    pub fn as_int(self) -> Option<Wrapping<i64>>;
    pub fn as_float(self) -> Option<f64>;
    pub fn as_char(self) -> Option<CChar>;
    pub fn as_str(self) -> Option<Vec<u8>>;
}

impl<'ident> IdentifierParser<'ident> {
    pub fn new(identifiers: &HashMap<Vec<u8>, EvalResult>) -> IdentifierParser<'_>;
    pub fn expr(&self, input: &[Token]) -> CResult<'_< EvalResult>;
    pub fn macro_definition(&self, input: &[Token]) -> CResult<'_< (&[u8], EvalResult)>;
}

// 便捷函数
pub fn expr(input: &[Token]) -> CResult<'_< EvalResult>;
pub fn macro_definition(input: &[Token]) -> CResult<'_< (&[u8], EvalResult)>;
pub fn fn_macro_declaration(input: &[Token]) -> CResult<'_< (&[u8], Vec<&[u8]>)>;
```

### literal 模块

```rust
// literal.rs
#[derive(Debug, Clone, PartialEq)]
pub enum CChar {
    Char(char),
    Raw(u64),
}

pub fn parse(input: &[u8]) -> IResult<&[u8], EvalResult>;
```

### 错误类型

```rust
// lib.rs
#[derive(Debug)]
pub enum ErrorKind {
    ExactToken(token::Kind, &'static [u8]),
    ExactTokens(token::Kind, &'static [&'static str]),
    TypedToken(token::Kind),
    UnknownIdentifier,
    InvalidLiteral,
    Partial,
    Parser(nom::ErrorKind),
}

#[derive(Debug)]
pub struct Error<I> {
    pub input: I,
    pub error: ErrorKind,
}

pub fn assert_full_parse<'i, I: 'i, O, E>(
    result: nom::IResult<&'i [I], O, E>,
) -> nom::IResult<&'i [I], O, Error<&'i [I]>>;
```

---

## 5.3 OH 使用 vs 上游使用对比

### 使用方式完全一致

| 场景 | 上游代码 | OH 代码 | 差异 |
|------|---------|---------|------|
| 基本求值 | `cexpr::expr::expr(&tokens)` | 相同 | 无 |
| 宏定义解析 | `parser.macro_definition(&tokens)` | 相同 | 无 |
| 带标识符求值 | `IdentifierParser::new(&idents).expr(&tokens)` | 相同 | 无 |
| 错误处理 | `match result { Ok(...) => ..., Err(...) => ... }` | 相同 | 无 |

### bindgen 中的实际使用（OH 与上游一致）

```rust
// 此代码在 OH 和上游 bindgen 中完全一致

use cexpr::expr::{EvalResult, IdentifierParser};
use cexpr::token::Token;

// 解析宏定义
fn parse_macro(tokens: &[Token]) -> Option<(&[u8], EvalResult)> {
    match cexpr::expr::macro_definition(tokens) {
        Ok((_, result)) => Some(result),
        Err(_) => None,
    }
}

// 带上下文解析
fn parse_with_context(
    tokens: &[Token],
    known_macros: &HashMap<Vec<u8>, EvalResult>,
) -> Option<EvalResult> {
    let parser = IdentifierParser::new(known_macros);
    match parser.expr(tokens) {
        Ok((_, result)) => Some(result),
        Err(_) => None,
    }
}
```

---

## 5.4 为什么没有 API 差异？

### 原因分析

#### 1. 功能完整

上游 API 设计已满足 bindgen 的所有需求：
- ✅ 表达式求值
- ✅ 宏定义解析
- ✅ 标识符替换
- ✅ 多种结果类型

无需扩展。

#### 2. 职责分离

cexpr 只负责"解析表达式"，其他功能由 bindgen 处理：
- Token 化 → bindgen 使用 libclang
- 代码生成 → bindgen 自行实现
- 类型推断 → bindgen 自行实现

#### 3. 接口稳定

cexpr 0.6.0 的 API 是稳定的，bindgen 依赖的接口不会变更：
- `EvalResult` 枚举
- `IdentifierParser` 结构体
- `macro_definition` 函数

#### 4. 纯 Rust 实现

无需平台特定的适配层。

---

## 5.5 与其他库的对比

| 库 | API 差异 | 原因 |
|---|---------|------|
| curl | 大量 | 平台适配、功能扩展 |
| openssl | 中等 | 安全修复、平台适配 |
| rust-cexpr | **无** | 功能完整、纯 Rust、无平台依赖 |

---

## 5.6 对开发者的影响

### 正面影响

1. **学习成本低**：参考上游文档即可
2. **代码可移植**：OH 代码可直接在其他 Rust 项目中使用
3. **文档丰富**：上游 docs.rs 文档完全适用

### 升级影响

升级 cexpr 时无需担心 API 变更影响 OH 代码：
```
上游 API 不变 ──→ OH 代码不变
     ↓
升级简单，风险低
```

---

## 5.7 总结

| 项目 | 结论 |
|------|------|
| API 添加 | **无** |
| API 修改 | **无** |
| API 废弃 | **无** |
| 行为变更 | **无** |
| 文档差异 | **无**（使用上游文档） |

rust-cexpr 是 OH 第三方库中 API 完全对齐上游的典范，这种"零差异"设计是：
- 上游设计优秀的体现
- 功能边界清晰的体现
- 纯 Rust 跨平台优势的体现

---

*零 API 差异意味着零学习成本，开发者可以直接参考上游文档使用此库*
