# nom 原始库简介

> nom - eating data byte by byte

## 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | nom |
| **当前版本** | 7.1.3 |
| **上游地址** | https://github.com/rust-bakery/nom |
| **许可证** | MIT |
| **作者** | Geoffroy Couprie <contact@geoffroycouprie.com> |
| **首次发布** | 2014年 |
| **维护状态** | 活跃维护 |

## 核心定位

nom 是一个用 Rust 编写的**解析器组合子库 (parser combinators library)**。它的设计目标是提供一套工具，让开发者能够构建**安全、高效、正确**的解析器，而无需在速度或内存消耗上做出妥协。

### 什么是解析器组合子？

解析器组合子是一种**声明式**的解析方法论。与传统的 lex/yacc 工具不同，解析器组合子让你使用非常小且功能单一的函数（如"取5字节"、"识别'HTTP'这个词"）来构建解析器，并将它们以有意义的方式组合起来（如"识别'HTTP'，然后一个空格，然后一个版本号"）。

**nom 的优势**：
- 解析器小而简单
- 组件易于重用
- 组件易于单独测试
- 代码接近自然语法
- 支持构建部分解析器

## 核心特性

### 1. 字节导向 (Byte-oriented)

nom 的基本类型是 `&[u8]`（字节切片引用），解析器尽可能在字节数组切片上工作：

```rust
use nom::bytes::complete::tag;

fn parse_http_method(input: &[u8]) -> IResult<&[u8], &[u8]> {
    tag(b"GET")(input)?  // 解析 "GET" 字节序列
}
```

### 2. 零拷贝 (Zero-copy)

如果解析器返回输入数据的子集，它会返回该输入的切片引用，**不进行数据拷贝**：

```rust
use nom::bytes::complete::take_while_m_n;

fn take_five_bytes(input: &[u8]) -> IResult<&[u8], &[u8]> {
    take_while_m_n(5, 5, |_| true)(input)?
    // 返回输入的前5字节的切片引用，无拷贝
}
```

### 3. 流式解析 (Streaming)

nom 可以在**不完整数据**上正确工作。如果数据不足以做出判断，nom 会明确告知需要更多数据：

```rust
use nom::combinator::rest;

fn parse_incomplete(input: &[u8]) -> IResult<&[u8], &[u8]> {
    rest(input)?  // 返回剩余所有数据，即使不完整
}
```

这对于网络协议解析和超大文件处理特别重要。

### 4. 位级别解析 (Bit-oriented)

nom 可以将字节切片作为位流来处理：

```rust
use nom::bits::complete::take;

fn parse_4_bits(input: (&[u8], usize)) -> IResult<(&[u8], usize), u8> {
    take(4usize)(input)?  // 从位流中取4位
}
```

### 5. 丰富的错误信息

nom 可以聚合错误代码列表，并指向错误输入切片，支持模式匹配以提供有用的错误消息：

```rust
use nom::{error::VerboseError, IResult};

fn parse_with_errors(input: &str) -> IResult<&str, &str, VerboseError<&str>> {
    // 错误信息会包含位置和期望的内容
    tag("Hello")(input)
}
```

## 技术特性总结

| 特性 | 支持 | 说明 |
|------|------|------|
| 字节导向 | ✅ | 基本类型 `&[u8]` |
| 位导向 | ✅ | 支持位流解析 |
| 字符串导向 | ✅ | UTF-8 字符串支持 |
| 零拷贝 | ✅ | 切片引用返回 |
| 流式解析 | ✅ | 支持不完整数据 |
| 错误描述 | ✅ | 详细错误信息 |
| 自定义错误 | ✅ | 可指定错误类型 |
| 安全解析 | ✅ | Rust 内存安全保证 |
| 高速解析 | ✅ | 性能优于大多数替代方案 |

## 典型应用场景

### 二进制格式解析

nom 从一开始就为正确解析二进制格式而设计：

- TLV (Type-Length-Value) 格式
- 位级别解析
- 网络协议
- 文件格式 (FLV, Matroska, TAR)

### 文本格式解析

nom 也可以很好地处理文本格式：

- CSV、JSON
- 配置文件 (TOML, INI)
- 复杂嵌套结构

### 编程语言解析

虽然通常手写解析器以获得更大的灵活性，但 nom 可以（并且已经成功地）被用作语言的原型解析器：

- PHP 虚拟机
- SQL 解析器
- 各种领域专用语言

## 上游资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.rs/nom |
| GitHub | https://github.com/rust-bakery/nom |
| 教程文档 | https://github.com/rust-bakery/nom/tree/main/doc |
| 解析器示例 | https://github.com/rust-bakery/nom/blob/main/doc/choosing_a_combinator.md |

## nom 在 OpenHarmony 中的定位

### 生态系统角色

```
┌─────────────────────────────────────────────┐
│          OpenHarmony Rust 生态               │
├─────────────────────────────────────────────┤
│                                             │
│   ┌──────────────┐     ┌──────────────┐    │
│   │ rust-cexpr   │────▶│     nom      │    │
│   │ (C表达式解析) │     │ (解析组合子)  │    │
│   └──────────────┘     └──────────────┘    │
│                             ▲               │
│                             │ 基础依赖      │
│                             ▼               │
│                     ┌──────────────┐       │
│                     │  其他解析需求   │       │
│                     │  (待扩展...)   │       │
│                     └──────────────┘       │
│                                             │
└─────────────────────────────────────────────┘
```

### 为什么 OpenHarmony 需要 nom

1. **Rust 生态完整性**：作为 Rust 解析器的事实标准库，nom 的存在使 OH 能够支持更多 Rust crates
2. **零成本抽象**：nom 的解析器组合子不会引入额外的运行时开销
3. **安全保证**：相比手写 C 解析器，nom 避免了缓冲区溢出等安全问题
4. **社区兼容**：许多现有的 Rust crates 依赖 nom，引入 nom 可以复用这些生态资源

### 版本策略

| 版本 | 状态 | 说明 |
|------|------|------|
| 7.1.3 | ✅ 当前 | OpenHarmony 当前使用的版本 |
| 7.x | ✅ 稳定 | API 稳定的成熟版本 |
| 8.x | 🔜 未来 | 关注上游开发进度 |

---

## 相关文档

- [Patch 分析](./02_Patches.md) - nom 在 OH 中的代码修改
- [构建适配](./03_Build_Integration.md) - 构建系统配置
- [依赖关系](./04_Usage_in_OH.md) - 使用方式和场景

---

**最后更新**: 2024年
