# 05 - API/接口差异

## 5.1 概述

static-assertions-rs 在 OpenHarmony 中**完全保留上游 API**，无任何修改或扩展。

| 属性 | 状态 |
|------|------|
| **OH 新增 API** | 无 |
| **API 行为变更** | 无 |
| **废弃功能** | 无 |
| **禁用功能** | 无（nightly 特性未启用） |

---

## 5.2 完整 API 列表

该库提供的所有宏（与上游完全一致）：

### 常量断言

| 宏 | 用途 |
|----|------|
| `const_assert!` | 编译期布尔断言 |
| `const_assert_eq!` | 编译期相等断言 |
| `const_assert_ne!` | 编译期不等断言 |

### 类型断言

| 宏 | 用途 |
|----|------|
| `assert_impl_all!` | 断言类型实现所有指定 trait |
| `assert_impl_any!` | 断言类型实现任一指定 trait |
| `assert_impl_one!` | 断言类型恰好实现一个 trait |
| `assert_not_impl_all!` | 断言类型未实现所有指定 trait |
| `assert_not_impl_any!` | 断言类型未实现任一指定 trait |
| `assert_type_eq_all!` | 断言所有类型相同 |
| `assert_type_ne_all!` | 断言所有类型不同 |

### 大小和对齐断言

| 宏 | 用途 |
|----|------|
| `assert_eq_size!` | 断言类型大小相同 |
| `assert_eq_size_ptr!` | 断言指针指向类型大小相同 |
| `assert_eq_size_val!` | 断言值类型大小相同 |
| `assert_eq_align!` | 断言类型对齐相同 |

### Trait 相关断言

| 宏 | 用途 |
|----|------|
| `assert_obj_safe!` | 断言 trait 对象安全 |
| `assert_trait_sub_all!` | 断言 trait 包含关系 |
| `assert_trait_super_all!` | 断言 trait 被包含关系 |

### 字段和配置断言

| 宏 | 用途 |
|----|------|
| `assert_fields!` | 断言结构体/枚举有指定字段 |
| `assert_cfg!` | 断言编译配置 |

---

## 5.3 未启用的功能

### nightly 特性

上游 Cargo.toml 定义了可选的 nightly 特性：

```toml
[features]
nightly = []
```

在 OpenHarmony 的 BUILD.gn 中，**该特性未启用**。

**影响**: 无影响，nightly 特性在当前版本（1.1.0）中为空特性，仅作为占位符。

---

## 5.4 API 使用示例

### 常量断言示例

```rust
use static_assertions::{const_assert, const_assert_eq};

const BUFFER_SIZE: usize = 4096;
const MIN_SIZE: usize = 1024;

// 常量布尔断言
const_assert!(BUFFER_SIZE > MIN_SIZE);

// 常量相等断言
const_assert_eq!(BUFFER_SIZE, 4096);
```

### 类型 Trait 断言示例

```rust
use static_assertions::assert_impl_all;

struct MyData {
    value: String,
}

// 确保 MyData 可安全跨线程传递
assert_impl_all!(MyData: Send, Sync);
```

### 类型大小断言示例

```rust
use static_assertions::assert_eq_size;

// 确保 u32 和 i32 大小相同
assert_eq_size!(u32, i32);

// 确保自定义类型与原始类型大小相同
struct Wrapper(u32);
assert_eq_size!(Wrapper, u32);
```

### Trait 对象安全示例

```rust
use static_assertions::assert_obj_safe;

trait MyTrait {
    fn method(&self);
}

// 确保 MyTrait 可被用作 trait 对象
assert_obj_safe!(MyTrait);
```

---

## 5.5 与上游文档的一致性

该库的 API 文档完全一致：

- **RustDoc**: https://docs.rs/static_assertions/1.1.0/static_assertions/
- **上游 README**: 完整保留在源码中
- **示例代码**: 所有上游示例在 OH 中均可正常使用

---

## 5.6 API 稳定性

### 版本兼容性

| 版本 | API 稳定性 | 说明 |
|------|-----------|------|
| 1.0.0 - 1.1.0 | 稳定 | 无破坏性变更 |
| 0.x - 1.0.0 | 有破坏性变更 | 标签语法变更等 |

### 未来兼容性

由于该库：
- 使用稳定版 Rust 特性
- 不依赖 nightly
- API 已成熟

预计未来版本升级不会有破坏性 API 变更。

---

## 5.7 总结

| 方面 | 结论 |
|------|------|
| API 完整性 | 100% 保留上游 API |
| 功能差异 | 无差异 |
| 使用方式 | 与上游完全一致 |
| 文档参考 | 可直接参考上游文档 |

**建议**: 开发者可直接参考[上游文档](https://docs.rs/static_assertions/)了解详细 API 用法。
