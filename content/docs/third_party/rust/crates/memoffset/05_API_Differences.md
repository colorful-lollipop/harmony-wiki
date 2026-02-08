# API/接口差异

## 差异状态总结

**该库没有 OpenHarmony 特定的 API 差异**

memoffset 在 OpenHarmony 中使用上游版本的完整功能，未进行任何 API 修改或扩展。

## 无差异说明

### 代码一致性

```rust
// src/lib.rs - 与上游版本完全一致
#![no_std]

#[macro_use]
mod raw_field;
#[macro_use]
mod offset_of;
#[macro_use]
mod span_of;

// 导出的宏
pub use offset_of::*;
pub use span_of::*;
```

**结论**: OH 中的 memoffset 源代码与上游版本**完全一致**。

### API 完整性

| API 类型 | OH 状态 | 说明 |
|---------|---------|------|
| `offset_of!` | ✓ 完整 | 结构体偏移量计算 |
| `offset_of_tuple!` | ✓ 完整 | 元组偏移量计算 |
| `offset_of_union!` | ✓ 完整 | 联合体偏移量计算 |
| `span_of!` | ✓ 完整 | 字段范围计算 |

## 上游 API 参考

### 核心宏

#### offset_of!

```rust
/// 计算结构体指定字段的内存偏移量
///
/// # 语法
/// offset_of!(Struct, field)
///
/// # 参数
/// - `Struct`: 结构体类型
/// - `field`: 要计算偏移量的字段标识符
///
/// # 返回值
/// 返回 `usize` 类型的偏移量（字节）
///
/// # 示例
/// ```rust
/// use memoffset::offset_of;
///
/// #[repr(C)]
/// struct Foo {
///     a: u32,
///     b: u64,
///     c: [u8; 4],
/// }
///
/// assert_eq!(offset_of!(Foo, a), 0);
/// assert_eq!(offset_of!(Foo, b), 8);
/// assert_eq!(offset_of!(Foo, c), 16);
/// ```
#[macro_export]
macro_rules! offset_of {
    ($struct_type:path, $($field:tt)*) => {
        // ...
    };
}
```

#### offset_of_tuple!

```rust
/// 计算元组指定元素的偏移量
///
/// # 要求
/// 需要 Rust 1.20+ (tuple_ty cfg)
///
/// # 语法
/// offset_of_tuple!(Tuple, index)
///
/// # 示例
/// ```rust
/// use memoffset::offset_of_tuple;
///
/// let tuple = (1u32, 2u64, 3u16);
///
/// assert_eq!(offset_of_tuple!(tuple, 0), 0);
/// assert_eq!(offset_of_tuple!(tuple, 1), 8);
/// assert_eq!(offset_of_tuple!(tuple, 2), 16);
/// ```
```

#### offset_of_union!

```rust
/// 计算联合体指定字段的偏移量
///
/// # 语法
/// offset_of_union!(Union, field)
///
/// # 示例
/// ```rust
/// use memoffset::offset_of_union;
///
/// #[repr(C)]
/// union MyUnion {
///     as_i32: i32,
///     as_u64: u64,
///     as_bytes: [u8; 8],
/// }
///
/// assert_eq!(offset_of_union!(MyUnion, as_i32), 0);
/// assert_eq!(offset_of_union!(MyUnion, as_u64), 0);
/// ```
```

#### span_of!

```rust
/// 计算字段或字段组的内存范围
///
/// # 语法
/// span_of!(Struct, field...)
/// span_of!(Struct, start_field..end_field)
/// span_of!(Struct, ..=end_field)
/// span_of!(Struct, start_field..)
///
/// # 返回值
/// 返回 `core::ops::Range<usize>` 类型的内存范围
///
/// # 示例
/// ```rust
/// use memoffset::span_of;
///
/// #[repr(C, packed)]
/// struct Packet {
///     header: [u8; 8],
///     body: [u8; 16],
///     footer: [u8; 4],
/// }
///
/// assert_eq!(span_of!(Packet, header), 0..8);
/// assert_eq!(span_of!(Packet, header..body), 0..24);
/// assert_eq!(span_of!(Packet, header..=footer), 0..28);
/// ```
```

## OH 特有功能

### 当前状态

**OH 未添加任何特有 API**

| 功能类型 | OH 状态 | 说明 |
|---------|---------|------|
| 新增 API | ✗ 无 | 未添加任何新 API |
| 修改 API | ✗ 无 | 未修改任何现有 API |
| 废弃 API | ✗ 无 | 未废弃任何 API |
| 扩展参数 | ✗ 无 | 未添加任何扩展参数 |

### 未来可能性

如 OH 未来需要添加特有功能，可能的方向：

| 功能方向 | 可能性 | 说明 |
|---------|-------|------|
| OH 特有错误处理 | 低 | 当前 API 无错误场景 |
| 架构特定优化 | 低 | 纯宏实现，无需特定优化 |
| 集成 OH API | 低 | 与 OH 运行时交互 |

## 版本演进

### Rust 1.77+ 变化

从 Rust 1.77 开始，标准库提供了 `core::mem::offset_of!`。memoffset v0.9.1 会自动检测并转发：

```rust
// build.rs 版本检测
if ac.probe_rustc_version(1, 77) {
    println!("cargo:rustc-cfg=stable_offset_of");
}
```

**转发机制**:

| Rust 版本 | 使用的实现 |
|-----------|-----------|
| 1.19 - 1.76 | memoffset 自有实现 |
| 1.77+ | core::mem::offset_of! |

### OH 版本策略

| OH 版本 | Rust 版本 | offset_of 来源 |
|---------|-----------|----------------|
| OH 6.x | 1.77+ | 自动使用 std |
| OH 旧版本 | 1.76- | 使用 memoffset 实现 |

## API 使用建议

### 推荐使用方式

```rust
// Rust 1.77+ 推荐
use core::mem::offset_of;

// 或通过 nix/rustix 间接使用
use nix::sys::socket::sockaddr_in;
```

### 兼容性考虑

| 场景 | 推荐 API |
|------|---------|
| 新代码（Rust 1.77+） | `core::mem::offset_of!` |
| 兼容老版本 | `memoffset::offset_of!` |
| 通过 nix/rustix | 使用库提供的抽象 |

## 相关文档

- [上游 API 文档](https://docs.rs/memoffset/)
- [Rust 标准库 offset_of](https://doc.rust-lang.org/std/mem/fn.offset_of.html)
- [Rust Reference: offset_of](https://doc.rust-lang.org/reference/items/associated-constant.html)
