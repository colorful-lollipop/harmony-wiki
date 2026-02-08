# 原始库简介

> `peeking_take_while` Rust Crate 概览

---

## 基本信息

| 项目 | 详情 |
|------|------|
| **库名称** | peeking_take_while |
| **作者** | Nick Fitzgerald <fitzgen@gmail.com> |
| **上游地址** | https://github.com/fitzgen/peeking_take_while |
| **许可证** | Apache-2.0 OR MIT |
| **当前版本** | 0.1.2 (OH BUILD.gn) / 1.0.0 (Cargo.toml) ⚠️ |
| **上游最新** | 1.0.0 (2021-09-03) |

---

## 功能描述

### 一句话概括

提供类似标准库 `take_while` 的迭代器适配器，但通过**预查看（peek）避免消耗首个不满足条件的元素**。

### 核心问题

标准库的 `Iterator::take_while` 有一个关键限制：

```rust
use std::iter::Peekable;

let data: Vec<i32> = (0..10).collect();
let mut iter = data.into_iter().peekable();

// 使用 take_while 处理小于 5 的元素
let small: Vec<i32> = iter.by_ref()
    .take_while(|&x| x < 5)
    .collect();

// 问题：元素 5 被消耗掉了！
assert_eq!(iter.next(), Some(6));  // 5 丢失了！
```

### 解决方案

`peeking_take_while` 通过预查看解决此问题：

```rust
use peeking_take_while::PeekableExt;

let data: Vec<i32> = (0..10).collect();
let mut iter = data.into_iter().peekable();

// 使用 peeking_take_while 处理小于 5 的元素
let small: Vec<i32> = iter.by_ref()
    .peeking_take_while(|&x| x < 5)
    .collect();

// 解决：元素 5 被保留！
assert_eq!(iter.next(), Some(5));  // 5 还在！
```

---

## 核心 API

### Trait: PeekableExt

```rust
pub trait PeekableExt<I>: Iterator
where
    I: Iterator,
{
    fn peeking_take_while<P>(&mut self, predicate: P) -> PeekingTakeWhile<'_, I, P>
    where
        P: FnMut(&Self::Item) -> bool;
}
```

**实现**: 为 `core::iter::Peekable<I>` 实现

### 方法: peeking_take_while

| 参数 | 类型 | 说明 |
|------|------|------|
| `predicate` | `P: FnMut(&Self::Item) -> bool` | 条件判断函数 |

**返回值**: `PeekingTakeWhile<'_, I, P>` - 新的迭代器适配器

### Struct: PeekingTakeWhile

迭代器适配器结构体，持有对 `Peekable` 的可变引用和谓词。

---

## 在 OpenHarmony 中的作用

### 定位

**类别**: 基础 Rust 工具库

**角色**:
- 提供 Rust 标准库的补充功能
- 支持需要精确控制迭代器消费的场景
- 特别适用于嵌入式和系统编程

### 价值主张

1. **精确控制**: 在 `by_ref()` 使用中保留分界元素
2. **无开销**: 零成本抽象，编译后无额外运行时开销
3. **安全可靠**: `no_std` 支持，`forbid(unsafe_code)`
4. **轻量级**: 单文件 220 行，无外部依赖

---

## 典型使用场景

### 场景 1: 词法分析器

```rust
use peeking_take_while::PeekableExt;

fn parse_number(chars: &mut std::iter::Peekable<impl Iterator<Item = char>>) -> String {
    chars.by_ref()
        .peeking_take_while(|c| c.is_ascii_digit())
        .collect()
}

let input = "123abc";
let mut chars = input.chars().peekable();
let number = parse_number(&mut chars);
assert_eq!(number, "123");
assert_eq!(chars.next(), Some('a'));  // 保留 'a'
```

### 场景 2: 前导空白处理

```rust
use peeking_take_while::PeekableExt;

fn skip_whitespace(chars: &mut std::iter::Peekable<impl Iterator<Item = char>>) {
    chars.by_ref()
        .peeking_take_while(|c| c.is_whitespace())
        .count();  // 消费但不使用
}

let input = "   hello";
let mut chars = input.chars().peekable();
skip_whitespace(&mut chars);
assert_eq!(chars.next(), Some('h'));  // 保留 'h'
```

### 场景 3: 二进制数据解析

```rust
use peeking_take_while::PeekableExt;

fn parse_header(bytes: &mut std::iter::Peekable<impl Iterator<Item = u8>>) -> Vec<u8> {
    bytes.by_ref()
        .peeking_take_while(|&b| b != 0)
        .collect()
}

let data: Vec<u8> = vec![1, 2, 3, 0, 4, 5];
let mut iter = data.into_iter().peekable();
let header = parse_header(&mut iter);
assert_eq!(header, vec![1, 2, 3]);
assert_eq!(iter.next(), Some(0));  // 保留分隔符 0
```

---

## 技术特性

### 代码质量

| 特性 | 状态 | 说明 |
|------|------|------|
| **no_std 支持** | ✅ | 适合 OpenHarmony 环境 |
| **unsafe 代码** | ❌ | `forbid(unsafe_code)` |
| **外部依赖** | ❌ | 零依赖 |
| **测试覆盖** | ✅ | 5 个测试用例 |
| **文档完整性** | ✅ | 完整注释和示例 |

### 性能特性

| 特性 | 说明 |
|------|------|
| **零成本抽象** | `#[inline]` 标记，编译优化后无开销 |
| **内存高效** | 仅持有引用，无额外分配 |
| **迭代器融合** | 支持与其他迭代器适配器组合 |

### 版本对比

| 特性 | 0.1.2 (OH BUILD.gn) | 1.0.0 (Cargo.toml) |
|------|---------------------|-------------------|
| Edition | 2015 | 2018 |
| no_std | ❌ | ✅ |
| 代码行数 | 41 行 | 120 行 |
| 文档 | 基础 | 完整 |
| 测试 | 基础 | 完善 |
| 实现方式 | 手动 | 使用 `Iterator::next_if` |

---

## 与标准库对比

### take_while vs peeking_take_while

| 特性 | take_while | peeking_take_while |
|------|-----------|-------------------|
| **分界元素** | 消耗 | 保留 |
| **与 by_ref 兼容** | ❌ 丢失元素 | ✅ 保留元素 |
| **运行时开销** | 低 | 低 |
| **适用场景** | 一次性处理 | 分段处理 |

### 何时使用哪个

```rust
// 场景 1: 一次性处理，不需要保留分界元素
// → 使用 take_while
let data = (0..100).filter(|&x| x < 10).collect();

// 场景 2: 分段处理，需要保留分界元素
// → 使用 peeking_take_while
use peeking_take_while::PeekableExt;

let mut iter = (0..100).peekable();
let first_part: Vec<i32> = iter.by_ref()
    .peeking_take_while(|&x| x < 10)
    .collect();
let second_part: Vec<i32> = iter.by_ref()
    .take(10)
    .collect();
```

---

## 在 Rust 生态中的位置

### 替代方案

| 方案 | 优点 | 缺点 |
|------|------|------|
| **peeking_take_while** | 轻量、专注、零依赖 | 维护不活跃 |
| **itertools** | 功能全面、维护活跃 | 依赖较大 |

### 使用统计

- **crates.io 下载量**: 约 1.07 亿次
- **直接依赖项目**: 约 10+ 个
- **通过 itertools 间接使用**: 更广泛

### 主要用户

- [Vector (Datadog)](https://github.com/vectordotdev/vector) - 日志处理
- [DataFusion](https://github.com/apache/datafusion) - SQL 解析
- [RisingWave](https://github.com/risingwavelabs/risingwave) - 分布式数据库

---

## 版本历史

### 0.1.2 (2017-05-17)
- OpenHarmony BUILD.gn 使用版本
- 代码行数: 41 行
- Edition: Rust 2015

### 1.0.0 (2021-09-03)
- 当前上游最新版本
- 重大更新
- Edition: Rust 2018
- 代码行数: 120 行
- 改进:
  - 添加 `no_std` 支持
  - 改进文档和测试
  - 使用 `Iterator::next_if` 简化实现

---

## 总结

`peeking_take_while` 是一个**简单、专注、高质量**的 Rust 工具库，解决了标准库 `take_while` 在分段处理场景中的局限。虽然在 OpenHarmony 中的使用版本（0.1.2）较旧，但代码本身稳定且安全，适合作为基础工具库保留。建议后续升级到 1.0.0 版本以获得更好的语言特性支持和文档。

---

**相关文档**:
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - OH 中的使用情况
- [06_Security.md](./06_Security.md) - 安全性分析

**外部链接**:
- [上游仓库](https://github.com/fitzgen/peeking_take_while)
- [API 文档](https://docs.rs/peeking_take_while)

---

**文档版本**: 1.0.0
**最后更新**: 2026-02-08
