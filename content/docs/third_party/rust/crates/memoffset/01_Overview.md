# 原始库简介

## 库基本信息

| 属性 | 值 |
|------|-----|
| 库名称 | memoffset |
| 版本 | v0.9.1 |
| 许可证 | Apache License V2.0 / MIT |
| 上游地址 | https://github.com/Gilnaa/memoffset |
| 上游维护者 | Gilad Naaman <gilad.naaman@gmail.com> |
| Rust 支持 | 1.19 及以上 |

## 原始功能描述

memoffset 是一个专注于**结构体成员内存偏移量计算**的 Rust 库，提供类似 C 语言 `offsetof` 宏的功能。在 Rust stable 1.77 之前，该库填补了标准库缺乏结构体偏移量访问能力的空白。

### 核心功能

memoffset 提供以下四个核心宏：

| 宏名称 | 功能描述 | 使用场景 |
|--------|---------|---------|
| `offset_of!(Struct, field)` | 计算结构体指定字段的内存偏移量（字节） | FFI 调用、内存映射、序列化 |
| `offset_of_tuple!(Tuple, N)` | 计算元组第 N 个元素的偏移量 | 元组处理、动态访问 |
| `offset_of_union!(Union, field)` | 计算联合体指定字段的偏移量 | 联合体操作、位域处理 |
| `span_of!(Struct, field...)` | 计算字段或字段组的内存范围 | 批量内存操作、校验和计算 |

### 使用示例

```rust
use memoffset::{offset_of, span_of};

#[repr(C, packed)]
struct MessageHeader {
    msg_type: u32,
    msg_len: u32,
    payload: [u8; 1024],
    checksum: u16,
}

// 计算字段偏移量
let type_offset = offset_of!(MessageHeader, msg_type);      // 0
let len_offset = offset_of!(MessageHeader, msg_len);         // 4
let payload_offset = offset_of!(MessageHeader, payload);      // 8
let checksum_offset = offset_of!(MessageHeader, checksum);    // 1032

// 计算字段范围
let header_span = span_of!(MessageHeader, msg_type..checksum); // 0..1034
let payload_span = span_of!(MessageHeader, payload);          // 8..1032
```

## 原始设计特点

### 1. no_std 支持

memoffset 完全支持 `no_std` 环境，不依赖标准库：

```rust
#![no_std]
```

这使得该库可以在嵌入式系统、内核模块等受限环境中使用。

### 2. 纯宏实现

该库的核心功能完全通过 Rust 宏实现，不涉及任何平台特定的代码：

- **无 C 代码**：不依赖任何 native 代码
- **无系统调用**：运行时无系统调用开销
- **无平台分支**：代码逻辑与平台无关

### 3. 编译时计算

偏移量计算在编译时完成，运行时零开销：

```rust
// 编译期计算，结果作为常量使用
const TYPE_OFFSET: usize = offset_of!(MessageHeader, msg_type);
```

### 4. Rust 版本演进

memoffset 的设计考虑了 Rust 版本的演进：

| Rust 版本 | 功能 | memoffset 行为 |
|-----------|------|---------------|
| 1.19 - 1.76 | 无 stable offset_of | 提供完整的 offset_of! 实现 |
| 1.77+ | stable offset_of | 转发到 std::mem::offset_of! |

## 库定位

memoffset 定位为**底层支撑库**：

1. **非直接使用**：通常作为其他库的依赖间接使用
2. **基础设施**：为 FFI、序列化、系统编程提供基础能力
3. **过渡性质**：随着 Rust 标准库功能的完善，该库的重要性逐渐降低

## 在 OpenHarmony 中的作用

memoffset 在 OpenHarmony 生态系统中扮演**基础设施**角色：

### 依赖链

```
应用层/Rust crates
    ↓
nix / rustix (Unix API 绑定)
    ↓
memoffset (偏移量计算)
```

### 主要用途

1. **支持系统调用绑定**：nix 和 rustix 使用 memoffset 构造系统调用参数结构体
2. **确保跨平台一致**：在不同 CPU 架构下提供一致的内存布局计算
3. **FFI 互操作**：在 Rust 与 C/C++ 代码交互时计算结构体偏移

## 技术原理

### 偏移量计算原理

memoffset 使用 Rust 的 `&(((StructType) nullptr).field)` 模式获取字段偏移量：

```rust
// 简化的原理示意
macro_rules! offset_of {
    ($StructType:path, $field:ident) => {{
        // 将 nullptr 转换为结构体指针，访问字段地址
        let ptr = 0usize as *const $StructType;
        let field_ptr = core::ptr::addr_of!((*ptr).$field);
        field_ptr as usize
    }};
}
```

### 跨平台兼容性

由于 Rust 语言的跨平台特性和 `repr(C)` 属性，偏移量计算在不同平台上保持一致：

| 架构 | 对齐规则 | 偏移量计算 | 兼容性 |
|------|---------|-----------|--------|
| x86_64 | 8 字节对齐 | 编译期确定 | ✓ |
| aarch64 | 8 字节对齐 | 编译期确定 | ✓ |
| armv7 | 4 字节对齐 | 编译期确定 | ✓ |
| riscv64 | 8 字节对齐 | 编译期确定 | ✓ |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v0.9.1 | - | 当前 OH 使用版本 |
| v0.9.0 | - | 支持 Rust 1.65+ const 上下文 |
| v0.8.0 | - | 添加 span_of! 宏 |
| v0.7.0 | - | 支持 Rust 1.51+ raw_ref_macros |

## 相关资源

- [上游 crates.io 页面](https://crates.io/crates/memoffset)
- [上游文档](https://docs.rs/memoffset/)
- [Rust 语言偏移量文档](https://doc.rust-lang.org/std/mem/fn.offset_of.html)
