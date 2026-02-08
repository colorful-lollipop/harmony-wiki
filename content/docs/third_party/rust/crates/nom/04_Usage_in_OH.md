# 依赖关系与使用

> nom 库在 OpenHarmony 中的依赖关系和使用场景

## 依赖关系概览

### 直接依赖者

经过全面搜索，nom 库在 OpenHarmony 中的**直接依赖者**只有 **1 个**：

| 依赖模块 | BUILD.gn 路径 | 依赖方式 | 主要用途 |
|---------|--------------|---------|---------|
| rust-cexpr | //third_party/rust/crates/rust-cexpr/BUILD.gn | 直接依赖 | C 表达式解析器 |

### 依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│                  OpenHarmony Rust 生态                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌───────────────────────────────────────────────────┐    │
│   │              third_party/rust/crates               │    │
│   ├───────────────────────────────────────────────────┤    │
│   │                                                   │    │
│   │   ┌───────────────┐                               │    │
│   │   │  rust-cexpr   │                               │    │
│   │   │  (C表达式解析) │                               │    │
│   │   └───────┬───────┘                               │    │
│   │           │                                        │    │
│   │           ▼ 依赖                                  │    │
│   │   ┌───────────────┐                               │    │
│   │   │     nom       │◀─────────────────────────────┤    │
│   │   │ (解析器组合子) │                               │    │
│   │   └───────┬───────┘                               │    │
│   │           │                                        │    │
│   │   ┌───────┴───────┐                               │    │
│   │   │               │                               │    │
│   ▼   ▼               ▼                               │    │
│ ┌──────────┐   ┌──────────────┐                       │    │
│ │  memchr  │   │minimal-lexical│                      │    │
│ └──────────┘   └──────────────┘                       │    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## rust-cexpr 详细分析

### 库信息

| 属性 | 值 |
|------|-----|
| **库名称** | rust-cexpr |
| **上游地址** | https://github.com/jethrogb/rust-cexpr |
| **版本** | 0.6.0 |
| **用途** | C 表达式解析器和求值器 |

### BUILD.gn 配置

```gn
# third_party/rust/crates/rust-cexpr/BUILD.gn

ohos_cargo_crate("lib") {
    crate_name = "cexpr"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.6.0"
    cargo_pkg_authors = "Jethro Beekman <jethro@jbeekman.nl>"
    cargo_pkg_name = "cexpr"
    cargo_pkg_description = "A C expression parser and evaluator"

    deps = [
        "//third_party/rust/crates/nom:lib",  // ← 依赖 nom
        "//third_party/rust/crates/charset:lib",
    ]

    module_output_extension = ".rlib"
    part_name = "rust_rust_cexpr"
    subsystem_name = "thirdparty"
}
```

### rust-cexpr 的功能

rust-cexpr 是一个用于解析和求值 C 表达式的库，支持：

- 基本类型解析 (int, float, char, etc.)
- 运算符优先级
- 宏定义解析
- sizeof 表达式
- 条件编译表达式

### rust-cexpr 对 nom 的使用方式

```rust
// rust-cexpr/src/tokenizer.rs (示例)

use nom::{
    IResult,
    bytes::complete::{tag, take_while},
    character::complete::digit0,
    combinator::map,
};

fn parse_number(input: &str) -> IResult<&str, Token> {
    // 使用 nom 解析数字
    map(digit0, |s: &str| Token::Number(s.parse().unwrap()))(input)
}
```

---

## 潜在使用场景

虽然当前只有 rust-cexpr 直接依赖 nom，但 nom 在 OpenHarmony 中有广泛的应用潜力：

### 1. 配置文件解析

| 格式 | 适用性 | 说明 |
|------|--------|------|
| JSON | ✅ 适合 | nom 有丰富的 JSON 解析示例 |
| XML | ✅ 适合 | 适合结构化标记语言 |
| TOML | ✅ 适合 | 适合配置格式 |
| YAML | ✅ 适合 | 适合数据序列化格式 |

### 2. 网络协议解析

| 协议 | 适用性 | 说明 |
|------|--------|------|
| HTTP | ✅ 适合 | nom 有 HTTP 解析器示例 |
| DNS | ✅ 适合 | 二进制协议解析 |
| MQTT | ✅ 适合 | IoT 常用协议 |
| CoAP | ✅ 适合 | 轻量级 IoT 协议 |

### 3. 文件格式解析

| 格式 | 适用性 | 说明 |
|------|--------|------|
| 图像格式 | ✅ 适合 | PNG, GIF, WebP 等 |
| 音频格式 | ✅ 适合 | WAV, FLAC 等 |
| 文档格式 | ✅ 适合 | 各种二进制格式 |
| 序列化格式 | ✅ 适合 | Protocol Buffers, MessagePack |

### 4. 领域特定语言 (DSL)

| DSL | 适用性 | 说明 |
|-----|--------|------|
| 查询语言 | ✅ 适合 | 数据库查询解析 |
| 规则引擎 | ✅ 适合 | 业务规则定义 |
| 模板引擎 | ✅ 适合 | 模板语法解析 |
| 配置 DSL | ✅ 适合 | 自定义配置语法 |

---

## 在 OH Rust 项目中使用 nom

### 添加依赖

#### 方式 1: 通过 BUILD.gn (推荐)

```gn
# your_component/BUILD.gn

ohos_rust_lib("my_parser") {
    crate_name = "my_parser"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    deps = [
        "//third_party/rust/crates/nom:lib",
    ]
}
```

#### 方式 2: 通过 Cargo.toml (如果支持)

```toml
[dependencies]
nom = "7.1.3"
```

### 使用示例

#### 示例 1: 简单的字节解析

```rust
// src/lib.rs

use nom::{
    IResult,
    bytes::complete::tag,
    combinator::map,
    sequence::tuple,
};

/// 解析固定格式: "PREFIX" + 2字节长度 + 数据
#[derive(Debug)]
struct Packet<'a> {
    data: &'a [u8],
}

fn parse_packet(input: &[u8]) -> IResult<&[u8], Packet> {
    let (input, _) = tag(b"PREFIX")(input)?;
    let (input, data) = map(tuple((
        take_2_bytes,
        take_slice,
    )))(input)?;

    Ok((input, Packet { data }))
}

fn take_2_bytes(input: &[u8]) -> IResult<&[u8], &[u8]> {
    // 假设前2字节是长度
    Ok((&input[2..], &input[..2]))
}

fn take_slice(input: &[u8]) -> IResult<&[u8], &[u8]> {
    Ok((&input[0..0], input)) // 简化示例
}
```

#### 示例 2: JSON 风格解析

```rust
use nom::{
    IResult,
    bytes::complete::{tag, is_not},
    character::complete::{char, alphanumeric1},
    sequence::delimited,
};

/// 解析简单的 JSON 字符串值
fn parse_string_value(input: &str) -> IResult<&str, &str> {
    delimited(
        char('"'),
        is_not("\""),
        char('"'),
    )(input)
}

/// 解析 JSON 对象中的键值对
fn parse_key_value(input: &str) -> IResult<&str, (&str, &str)> {
    let (input, key) = parse_string_value(input)?;
    let (input, _) = char(':')(input)?;
    let (input, value) = parse_string_value(input)?;

    Ok((input, (key, value)))
}
```

---

## 依赖关系管理

### 依赖链

```
应用层
    │
    ▼
rust-cexpr (depends on nom)
    │
    ▼
nom 7.1.3
    │
    ├── memchr 2.3
    └── minimal-lexical 0.2.0
```

### 依赖版本策略

| 库 | OH 版本 | 上游版本 | 同步状态 |
|----|---------|---------|---------|
| nom | 7.1.3 | 7.1.3 | ✅ 同步 |
| memchr | OH internal | 2.3 | ✅ 同步 |
| minimal-lexical | OH internal | 0.2.0 | ✅ 同步 |

---

## 最佳实践

### 1. 解析器组织

```rust
// src/parsers/mod.rs

mod json;
mod http;
mod custom;

pub use json::parse_json;
pub use http::parse_http_request;
pub use custom::{parse_my_format, MyParser};
```

### 2. 错误处理

```rust
use nom::{IResult, error::VerboseError};

type NomResult<T> = IResult<&[u8], T, VerboseError<&[u8]>>;

/// 统一的解析函数签名
pub trait Parser<T> {
    fn parse(&self, input: &[u8]) -> NomResult<T>;
}
```

### 3. 性能优化

```rust
// 1. 使用零拷贝解析
use nom::bytes::complete::{tag, take_while_m_n};

// 2. 避免不必要的分配
// 好的做法: 返回切片引用
fn parse_efficient(input: &[u8]) -> IResult<&[u8], &[u8]> {
    take_while_m_n(4, 4, |c| c == b'X')(input)
}

// 3. 流式解析大文件
use nom::combinator::rest;
use std::fs::File;

fn parse_file_chunks(file: &mut File) -> std::io::Result<()> {
    let mut chunk = [0u8; 1024];
    while let Ok(n) = file.read(&mut chunk) {
        if n == 0 { break; }
        let data = &chunk[..n];
        // 逐块解析
    }
    Ok(())
}
```

---

## 常见问题

### Q1: nom 和正则表达式选哪个？

| 场景 | 推荐 | 原因 |
|------|------|------|
| 简单模式匹配 | Regex | 更简洁，性能优化更好 |
| 结构化数据解析 | nom | 强类型保证，易于测试 |
| 复杂嵌套结构 | nom | 组合子易于处理嵌套 |
| 性能关键 | 两者都需评估 | 根据具体场景测试 |

### Q2: 如何处理解析错误？

```rust
use nom::{
    IResult,
    error::{VerboseError,VerboseErrorKind},
    Err,
};

fn parse_with_error_handling(input: &[u8]) -> Result<Vec<u8>, String> {
    match parse_something(input) {
        Ok((_, result)) => Ok(result),
        Err(VerboseError { errors }) => {
            let messages: Vec<String> = errors
                .into_iter()
                .map(|(i, e)| format!("{:?}: {:?}", i, e))
                .collect();
            Err(messages.join("; "))
        }
    }
}
```

### Q3: nom 在 OH 中的性能如何？

nom 以**高性能**著称：
- 基准测试显示优于 Parsec、attoparsec
- 可与手写 C 解析器媲美
- 零拷贝特性减少内存分配

### Q4: 如何调试 nom 解析器？

```rust
use nom::branch::alt;

// 使用 alt! 进行分支调试
fn debug_parse(input: &[u8]) -> IResult<&[u8], &str> {
    alt((
        |i| { dbg!(i); tag("option1")(i) },
        |i| { dbg!(i); tag("option2")(i) },
    ))(input)
}
```

---

## 未来扩展建议

### 可能依赖 nom 的模块

| 模块 | 可能性 | 用途 |
|------|--------|------|
| 配置文件解析 | 🔜 高 | JSON/TOML/YAML 支持 |
| 网络协议栈 | 🔜 中 | HTTP/DNS/CoAP 解析 |
| 文件格式支持 | 🔜 中 | 图片/音频/文档解析 |
| DSL 引擎 | 🔜 低 | 业务规则/查询语言 |

### 生态建设建议

1. **创建 OH 专用解析器**: 针对 OH 特有格式开发解析器
2. **贡献上游**: 将通用解析器贡献回 nom 社区
3. **文档完善**: 编写 OH 环境下的使用文档和示例
4. **性能测试**: 建立 nom 在 OH 平台的性能基准

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - nom 库功能介绍
- [02_Patches.md](./02_Patches.md) - Patch 分析
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置
- [rust-cexpr](../rust-cexpr) - nom 的直接依赖者

---

**最后更新**: 2024年
