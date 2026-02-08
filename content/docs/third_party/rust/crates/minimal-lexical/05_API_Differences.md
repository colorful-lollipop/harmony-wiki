# 05 - API/接口差异分析

## 5.1 概述

### 5.1.1 核心结论

**minimal-lexical 在 OpenHarmony 中的 API 与上游完全一致，无任何差异。**

由于 OH 版本直接采用上游源码，未应用任何 Patch，也未添加 OH 特有的 API，因此：

- ✅ 无新增 API
- ✅ 无删除 API
- ✅ 无修改 API
- ✅ 无行为变更

### 5.1.2 为什么无差异？

| 原因 | 说明 |
|------|------|
| **无 Patch** | 未对源码进行任何修改 |
| **功能完整** | 上游版本已满足 OH 需求 |
| **无需扩展** | 浮点数解析功能无需平台特定扩展 |
| **保持兼容** | 与上游保持一致有利于升级维护 |

## 5.2 公开 API 清单

### 5.2.1 核心 API

| API | 类型 | 签名 | 说明 |
|-----|------|------|------|
| `parse_float` | 函数 | `fn parse_float<F: Float>(integer, fraction, exponent) -> F` | 解析浮点数（唯一入口） |
| `Float` | Trait | `trait Float` | 浮点数类型 trait |

### 5.2.2 公开模块

| 模块 | 公开项 | 说明 |
|------|--------|------|
| `parse` | `parse_float` | 核心解析函数 |
| `num` | `Float` trait | 浮点数抽象 |
| `number` | `Number` struct | 数字内部表示 |
| `lemire` | 算法实现 | Eisel-Lemire 算法 |
| `bellerophon` | 算法实现 | Bellerophon 算法 |
| `slow` | 算法实现 | 慢速路径 |
| `bigint` | 大整数 | 大整数运算 |
| `extended_float` | 扩展浮点 | 扩展精度浮点 |
| `rounding` | 舍入 | 舍入处理 |
| `fpu` | FPU 控制 | 浮点单元控制 |
| `libm` | 数学函数 | 数学函数实现 |
| `mask` | 位掩码 | 位操作工具 |
| `heapvec` | 堆向量 | 堆分配向量 |
| `stackvec` | 栈向量 | 栈分配向量 |
| `table` | 查找表 | 算法查找表 |

### 5.2.3 模块可见性

```rust
// src/lib.rs

// 公开模块
pub mod bellerophon;
pub mod bigint;
pub mod extended_float;
pub mod fpu;
pub mod heapvec;
pub mod lemire;
pub mod libm;
pub mod mask;
pub mod num;
pub mod number;
pub mod parse;
pub mod rounding;
pub mod slow;
pub mod stackvec;
pub mod table;

// 私有模块（内部使用）
mod table_bellerophon;
mod table_lemire;
mod table_small;

// 导出核心 API
pub use self::num::Float;
pub use self::parse::parse_float;
```

## 5.3 API 详细说明

### 5.3.1 parse_float 函数

```rust
/// 解析浮点数
/// 
/// # 类型参数
/// - `F`: 目标浮点数类型（f32 或 f64）
/// 
/// # 参数
/// - `integer`: 整数部分的数字迭代器（ASCII 字节）
/// - `fraction`: 小数部分的数字迭代器（ASCII 字节）
/// - `exponent`: 指数值（已解析为 i32）
/// 
/// # 返回值
/// - 解析后的浮点数
/// 
/// # 前置条件
/// - 整数部分的前导零必须已被去除
/// - 小数部分的尾随零必须已被去除
/// - 迭代器产生的必须是 ASCII 数字字符（b'0'..=b'9'）
/// 
/// # 示例
/// ```
/// use minimal_lexical::parse_float;
/// 
/// let integer = b"3";
/// let fraction = b"14159";
/// let pi: f64 = parse_float(integer.iter(), fraction.iter(), 0);
/// assert_eq!(pi, 3.14159);
/// ```
pub fn parse_float<F: Float>(
    integer: impl Iterator<Item = &u8>,
    fraction: impl Iterator<Item = &u8>,
    exponent: i32
) -> F
```

### 5.3.2 Float Trait

```rust
/// 浮点数类型 trait
/// 
/// 实现了此 trait 的类型可以被 parse_float 解析。
/// 标准库中的 f32 和 f64 已自动实现此 trait。
pub trait Float {
    // 内部方法，对用户透明
    // ...
}

// 为 f32 和 f64 实现 Float trait
impl Float for f32 { /* ... */ }
impl Float for f64 { /* ... */ }
```

## 5.4 OH 与上游功能对比

### 5.4.1 Feature 对比

| Feature | 上游默认 | OH 配置 | 差异 |
|---------|---------|---------|------|
| `std` | ✅ 启用 | ✅ 启用 | 无差异 |
| `compact` | ❌ 未启用 | ❌ 未启用 | 无差异 |
| `alloc` | ❌ 未启用 | ❌ 未启用 | 无差异 |
| `nightly` | ❌ 未启用 | ❌ 未启用 | 无差异 |
| `lint` | ❌ 未启用 | ❌ 未启用 | 无差异 |

### 5.4.2 功能完整性

| 功能 | 上游 | OH | 说明 |
|------|------|-----|------|
| f32 解析 | ✅ | ✅ | 支持 |
| f64 解析 | ✅ | ✅ | 支持 |
| Eisel-Lemire 算法 | ✅ | ✅ | 快速路径 |
| Bellerophon 算法 | ✅ | ✅ | 中速路径 |
| 慢速路径（大整数） | ✅ | ✅ | 慢速路径 |
| no_std 支持 | ✅ | ✅ | 支持（通过 feature） |

## 5.5 使用示例对比

### 5.5.1 上游使用示例

```rust
// 来自上游文档的示例
extern crate minimal_lexical;

let integer = b"1";
let fraction = b"2345";
let float: f64 = minimal_lexical::parse_float(
    integer.iter(), 
    fraction.iter(), 
    0
);
println!("float={:?}", float);    // 1.2345
```

### 5.5.2 OH 中使用示例

```rust
// 在 OH 中的使用方式（完全相同）
extern crate minimal_lexical;

let integer = b"1";
let fraction = b"2345";
let float: f64 = minimal_lexical::parse_float(
    integer.iter(), 
    fraction.iter(), 
    0
);
println!("float={:?}", float);    // 1.2345
```

**结论**：使用方式完全一致，无任何差异。

## 5.6 与其他 Rust 解析库的对比

### 5.6.1 API 设计对比

| 库 | 核心 API | 复杂度 | 特点 |
|------|---------|--------|------|
| **minimal-lexical** | `parse_float(iterator, iterator, i32)` | 低 | 极简，高性能 |
| **std::str::parse** | `"1.23".parse::<f64>()?` | 低 | 标准库，易用 |
| **lexical-core** | `parse_float(string)` | 中 | 功能完整 |
| **fast-float** | `parse_float(string)` | 低 | 纯解析 |

### 5.6.2 minimal-lexical 的设计哲学

minimal-lexical 采用**低层 API 设计**：

```rust
// minimal-lexical：需要预处理
let input = "123.456e-7";
// 需要外部解析器分离出：
// - 整数部分："123"
// - 小数部分："456"
// - 指数部分：-7
let value = parse_float(
    b"123".iter(),
    b"456".iter(),
    -7
);

// 标准库：直接解析
let value: f64 = "123.456e-7".parse().unwrap();
```

**优势**：
- 更灵活：可以集成到各种解析框架中
- 更高性能：避免重复解析
- 零拷贝：无需创建中间字符串

**劣势**：
- 使用复杂：需要预处理
- 不适合直接使用：通常需要配合解析器框架（如 nom）

## 5.7 兼容性说明

### 5.7.1 与上游版本兼容

| OH 版本 | 上游版本 | 兼容性 |
|---------|---------|--------|
| 6.1 | 0.2.1 | ✅ 完全一致 |

### 5.7.2 未来升级兼容性

由于 OH 版本与上游完全一致，升级时只需关注上游的 CHANGELOG：

- **0.2.0 → 0.2.1**：API 兼容（patch 版本）
- **0.1.x → 0.2.x**：需检查 breaking changes

### 5.7.3 跨平台兼容性

API 在所有支持的平台上完全一致：

- ✅ x86_64 / x86
- ✅ ARM64 / ARMv7 / ARMv6
- ✅ MIPS / MIPS64
- ✅ PowerPC
- ✅ s390x

## 5.8 无差异的优势

### 5.8.1 维护优势

| 优势 | 说明 |
|------|------|
| **升级简单** | 直接替换源码即可 |
| **文档通用** | 可直接参考上游文档 |
| **测试复用** | 可直接运行上游测试 |
| **社区支持** | 可利用上游社区资源 |

### 5.8.2 开发优势

| 优势 | 说明 |
|------|------|
| **学习成本低** | 参考上游文档和示例 |
| **代码可移植** | 在 OH 和非 OH 环境使用相同代码 |
| **生态兼容** | 与 crates.io 生态完全兼容 |

## 5.9 总结

### 关键结论

1. **API 完全一致**：OH 版本与上游 0.2.1 版本 API 完全相同
2. **无新增功能**：未添加 OH 特有的 API
3. **无行为变更**：解析行为与上游一致
4. **完全兼容**：代码可在 OH 和非 OH 环境无缝迁移

### 对开发者的影响

- 可以参考 https://docs.rs/minimal-lexical 的文档
- 可以使用 crates.io 上的示例代码
- 无需学习 OH 特定的 API

---

*本文档最后更新：2026-02-07*
