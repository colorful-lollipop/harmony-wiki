# 01 - 原始库简介

## 1.1 库基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | static_assertions |
| **版本** | 1.1.0 |
| **作者** | Nikolai Vazquez |
| **许可证** | MIT OR Apache-2.0 |
| **上游地址** | https://github.com/nvzqz/static-assertions-rs |
| **crates.io** | https://crates.io/crates/static_assertions |

---

## 1.2 原始功能简介

static-assertions-rs 是一个 Rust 编译时断言宏库，提供在编译期验证代码假设的能力。

### 核心功能

1. **类型断言**: 验证类型实现特定 trait 或满足特定条件
2. **常量断言**: 验证编译期常量表达式的值
3. **大小/对齐断言**: 验证类型的大小和对齐方式
4. **Trait 对象安全**: 验证 trait 是否对象安全

### 提供的宏

| 类别 | 宏名称 | 用途 |
|------|--------|------|
| 类型 | `assert_impl_all!` | 断言类型实现所有指定 trait |
| 类型 | `assert_impl_any!` | 断言类型实现任一指定 trait |
| 类型 | `assert_not_impl_all!` | 断言类型未实现所有指定 trait |
| 类型 | `assert_type_eq_all!` | 断言所有类型相同 |
| 类型 | `assert_type_ne_all!` | 断言所有类型不同 |
| 常量 | `const_assert!` | 编译期布尔断言 |
| 常量 | `const_assert_eq!` | 编译期相等断言 |
| 常量 | `const_assert_ne!` | 编译期不等断言 |
| 大小 | `assert_eq_size!` | 断言类型大小相同 |
| 对齐 | `assert_eq_align!` | 断言类型对齐相同 |
| Trait | `assert_obj_safe!` | 断言 trait 对象安全 |
| Trait | `assert_trait_sub_all!` | 断言 trait 包含关系 |
| 字段 | `assert_fields!` | 断言结构体/枚举有指定字段 |
| 配置 | `assert_cfg!` | 断言编译配置 |

---

## 1.3 典型使用示例

### 类型 Trait 断言

```rust
use static_assertions::assert_impl_all;

struct MyType {
    data: String,
}

// 确保 MyType 实现 Send 和 Sync，可在多线程安全使用
assert_impl_all!(MyType: Send, Sync);
```

### 常量断言

```rust
use static_assertions::const_assert;

const BUFFER_SIZE: usize = 1024;

// 确保缓冲区大小合理
const_assert!(BUFFER_SIZE >= 512);
const_assert!(BUFFER_SIZE <= 4096);
```

### 类型大小断言

```rust
use static_assertions::assert_eq_size;

// 确保两个类型大小相同，便于内存操作
assert_eq_size!(u32, i32);
```

---

## 1.4 在 OpenHarmony 中的作用和定位

### 定位

在 OpenHarmony 的 Rust 生态系统中，static-assertions-rs 定位为**基础编译时验证工具**。

### 主要使用场景

1. **开发依赖验证**
   - 被 `clap`、`linux-raw-sys`、`libloading` 等 crate 作为开发依赖使用
   - 用于单元测试和编译期验证

2. **类型安全保证**
   - 确保跨 FFI 边界的类型满足特定 trait 约束
   - 验证结构体布局假设

3. **API 兼容性检查**
   - 在编译期捕获类型不匹配问题
   - 防止破坏性变更进入代码库

### 在 OH 中的特殊性

| 方面 | 说明 |
|------|------|
| **关键性** | 低（不影响运行时） |
| **使用频率** | 中（多 crate 开发依赖） |
| **修改需求** | 无（纯上游代码） |
| **维护成本** | 极低 |

### 与运行时无关

该库的一个重要特性是**纯编译期宏**，不生成任何运行时代码：

- ✅ 零运行时开销
- ✅ 不增加二进制大小
- ✅ 只在编译期执行验证
- ✅ 失败时产生编译错误而非运行时错误

---

## 1.5 版本历史

根据 CHANGELOG.md，v1.1.0 的主要变更：

### v1.1.0 (2019-11-03)

- 新增 `assert_impl_any!` 宏
- 新增 `assert_impl_one!` 宏
- 新增 `assert_trait_sub_all!` 宏
- 新增 `assert_trait_super_all!` 宏
- 修复内部宏导出问题

### v1.0.0 (2019-10-02)

- 新增 `assert_eq_align!` 宏
- **破坏性变更**: 移除宏标签（使用 `const _` 语法，Rust 1.37+）
- **破坏性变更**: 移除 `assert_impl!` 宏
- 修改 `const_assert!` 只接受单个表达式

---

## 1.6 技术特点

### no_std 支持

该库支持 `no_std` 环境，适用于嵌入式和内核开发：

```rust
#![no_std]
use static_assertions::const_assert;

const_assert!(true);
```

### Rust Edition

- 上游使用 Rust 2015 Edition
- 兼容 Rust 1.37.0+
- 利用 `const _` 语法实现无需标签的宏

### 零成本抽象

所有断言在编译期完成，不产生运行时开销：

```rust
// 这段代码不生成任何运行时指令
const_assert_eq!(MAX_CONNECTIONS, 100);

// 运行时只有实际业务代码
fn main() {
    println!("Running with max {} connections", MAX_CONNECTIONS);
}
```
