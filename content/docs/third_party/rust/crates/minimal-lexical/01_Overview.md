# 01 - minimal-lexical 库简介

## 1.1 原始库信息

### 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | minimal-lexical |
| **上游版本** | 0.2.1 |
| **Rust Edition** | 2018 |
| **MSRV** | Rust 1.36+ |
| **作者** | Alex Huszagh |
| **许可证** | Apache-2.0 / MIT 双许可 |
| **上游地址** | https://github.com/Alexhuszagh/minimal-lexical |
| **文档** | https://docs.rs/minimal-lexical |

### 功能描述

**minimal-lexical** 是 [rust-lexical](https://github.com/Alexhuszagh/rust-lexical) 的精简版本，专注于提供**快速且正确的浮点数字符串解析**。

核心特点：
- **快速编译**：代码量小，编译速度快
- **小体积**：生成的二进制文件体积小
- **正确性**：实现了业界领先的解析算法，确保 IEEE 754 标准的正确舍入
- **跨平台**：支持多种架构（x86, ARM, MIPS, PowerPC, s390x 等）
- **no_std 支持**：可以在无标准库环境下使用

### 设计目标

> "minimal-lexical is designed for fast compile times and small binary sizes, at the expense of a minor amount of performance."

与完整版 rust-lexical 相比，minimal-lexical 牺牲少量性能来换取更快的编译速度和更小的二进制体积。

---

## 1.2 核心 API

### 主要函数

```rust
/// 解析浮点数
/// 
/// # 参数
/// - `integer`: 整数部分的数字迭代器
/// - `fraction`: 小数部分的数字迭代器
/// - `exponent`: 指数值（已解析为 i32）
///
/// # 注意
/// - 整数部分的前导零必须已被去除
/// - 小数部分的尾随零必须已被去除
pub fn parse_float<F: Float>(
    integer: impl Iterator<Item = &u8>,
    fraction: impl Iterator<Item = &u8>,
    exponent: i32
) -> F
```

### 使用示例

```rust
extern crate minimal_lexical;

// 解析 "1.2345"
// 1. 使用外部解析器提取整数部分 "1"
// 2. 提取小数部分 "2345"
// 3. 解析指数为 0
let integer = b"1";
let fraction = b"2345";
let float: f64 = minimal_lexical::parse_float(
    integer.iter(),
    fraction.iter(),
    0
);
assert_eq!(float, 1.2345);
```

### 公开模块

| 模块 | 说明 |
|------|------|
| `parse` | 核心解析逻辑 |
| `num` | Float trait 定义 |
| `number` | 数字内部表示 |
| `lemire` | Eisel-Lemire 算法实现 |
| `bellerophon` | Bellerophon 算法实现 |
| `slow` | 慢速路径（大整数运算） |
| `bigint` | 大整数运算 |
| `extended_float` | 扩展精度浮点 |
| `rounding` | 舍入处理 |
| `fpu` | FPU 控制 |
| `table` | 查找表 |

---

## 1.3 核心算法

minimal-lexical 实现了**三层解析策略**，在性能和精度之间取得平衡：

### 第一层：Eisel-Lemire 算法（快速路径）

**处理范围**：大多数常见的浮点数

**算法特点**：
- 基于 Daniel Lemire 的研究成果
- 使用 128 位中间表示
- 无需大整数运算
- 速度极快（比标准库快数倍）

**成功率**：约 99% 的数字可以通过此路径解析

### 第二层：Bellerophon 算法（中速路径）

**处理范围**：快速路径失败的数字

**算法特点**：
- 扩展精度整数运算
- 处理需要更高精度的边界情况
- 比慢速路径快，比快速路径慢

### 第三层：慢速路径（Slow Path）

**处理范围**：极端情况（极大或极小的指数）

**算法特点**：
- 使用大整数（bigint）运算
- 确保 100% 正确性
- 速度较慢，但极少触发

---

## 1.4 在 OpenHarmony 中的定位

### 子系统归属

| 属性 | 值 |
|------|-----|
| **子系统** | thirdparty |
| **组件名** | rust_minimal_lexical |
| **发布方式** | code-segment |
| **适配系统类型** | standard |

### 在 OH 中的作用

minimal-lexical 是 OpenHarmony **基础设施层**的组件，作为**解析器生态的基础能力**提供浮点数解析服务。

```
┌─────────────────────────────────────────────────────────┐
│                      应用层                              │
│         (配置文件解析、数据处理、网络通信等)               │
├─────────────────────────────────────────────────────────┤
│                    框架层 (nom 等)                        │
│         (Parser Combinator 库、序列化库等)                │
├─────────────────────────────────────────────────────────┤
│                   基础库层 (本库)                          │
│              minimal-lexical (浮点数解析)                  │
├─────────────────────────────────────────────────────────┤
│                     运行时层                              │
│              Rust Standard Library                       │
└─────────────────────────────────────────────────────────┘
```

### 典型使用场景

1. **配置文件解析**
   - JSON、TOML、YAML 等配置格式中的数字字段
   - 示例：`{ "price": 19.99 }`

2. **网络协议解析**
   - HTTP 头中的浮点数字段
   - 自定义二进制/文本协议

3. **数据序列化/反序列化**
   - 日志解析
   - 数据导入/导出

4. **编译器/解释器**
   - 编程语言源码中的浮点数字面量解析

### 间接影响范围

虽然不直接暴露给应用开发者，但通过 `nom` 库，minimal-lexical 间接支持了 OpenHarmony 中大量解析相关的功能。

---

## 1.5 版本历史

### 上游 CHANGELOG

| 版本 | 日期 | 主要变更 |
|------|------|---------|
| 0.2.1 | - | 当前 OH 版本 |
| 0.2.0 | 2021-09-10 | `no_alloc` feature 替换为 `alloc` |
| 0.1.4 | 2021-10-02 | 添加缺失的许可证说明 |
| 0.1.3 | 2021-09-04 | 添加 `compact` 和 `nightly` features |
| 0.1.2 | 2021-05-09 | 移除 cached_float，优化指数推断 |
| 0.1.1 | 2021-05-08 | 添加 Eisel-Lemire 算法 |
| 0.1.0 | 2021-04-27 | 初始版本 |

### OH 版本信息

- **OH 组件版本**：6.1
- **上游版本**：0.2.1
- **同步时间**：2023 年（从 BUILD.gn 版权年份推断）

---

## 1.6 相关项目

### 上游生态

| 项目 | 关系 | 说明 |
|------|------|------|
| [rust-lexical](https://github.com/Alexhuszagh/rust-lexical) | 完整版 | 包含更多功能（数字到字符串转换等） |
| [nom](https://github.com/rust-bakery/nom) | 主要用户 | 使用 minimal-lexical 实现 float 解析器 |

### OH 中的相关库

| 库 | 关系 | 说明 |
|------|------|------|
| nom | 依赖者 | 唯一依赖 minimal-lexical 的 OH 组件 |
| memchr | 并列 | nom 的另一个依赖 |

---

## 1.7 平台支持

minimal-lexical 在以下平台上测试通过：

- **x86_64**: Linux, Windows, macOS, Android, iOS, FreeBSD, NetBSD
- **x86**: Linux, macOS, Android, iOS, FreeBSD
- **aarch64 (ARM64)**: Linux, Android, iOS
- **armv7**: Linux, Android, iOS
- **arm (ARMv6)**: Linux, Android
- **mips/mipsel**: Linux
- **mips64/mips64el**: Linux
- **powerpc/powerpc64/powerpc64le**: Linux
- **s390x**: Linux

---

*本文档最后更新：2026-02-07*
