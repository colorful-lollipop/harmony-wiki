# 01_Overview.md - 原始库简介

本文档简要介绍 `foreign-types` 库的基本功能、版本信息和在 OpenHarmony 中的定位。

> **一句话描述**: `foreign-types` 是一个用于 Rust FFI 编程的框架，专门为 C API 创建安全的 Rust 包装器。

---

## 📋 基础信息

### 库信息概览

| 属性 | 值 |
|------|-----|
| **库名称** | foreign-types |
| **上游仓库** | https://github.com/sfackler/foreign-types |
| **作者** | Steven Fackler <sfackler@gmail.com> |
| **维护者** | xuelei3@huawei.com (OH) |
| **crates.io** | https://crates.io/crates/foreign-types |
| **API 文档** | https://docs.rs/foreign-types |

### 版本信息

| 版本类型 | 版本号 | 说明 |
|---------|--------|------|
| **上游版本** | 0.3.2 | Rust crate 版本 |
| **OH 组件版本** | 6.1 | OH bundle.json 版本 |
| **foreign-types-shared** | 0.1.1 | 内部依赖 crate |

### 许可证

| 许可证类型 | 值 |
|-----------|-----|
| **上游许可证** | MIT / Apache-2.0 (双许可) |
| **OH 声明许可证** | Apache-2.0 |
| **OH 适配文件** | Apache-2.0 (华为版权) |

---

## 🎯 核心功能

### foreign-types 是什么？

`foreign-types` 是一个 **Rust FFI 编程框架**，用于为 C API 创建安全的 Rust 包装器。

### 设计理念

**C 和 Rust 的所有权语义差异**:
- C 的所有权通常是隐式的（通过文档约定）
- Rust 的所有权是显式的（通过类型系统）

**foreign-types 的解决方案**:
提供双类型体系，类似 Rust 的 `String`/`str` 或 `PathBuf`/`Path`：
- **Owned 类型** (如 `Foo`): 表示拥有所有权的值，负责释放资源
- **Borrowed 类型** (如 `FooRef`): 表示借用引用，不负责释放资源

### 核心组件

#### 1. `ForeignType` Trait

```rust
pub trait ForeignType: Sized {
    type CType;
    type Ref: ForeignTypeRef<CType = Self::CType>;

    unsafe fn from_ptr(ptr: *mut Self::CType) -> Self;
    fn as_ptr(&self) -> *mut Self::CType;
}
```

**功能**:
- 定义包装 C 类型的基本行为
- 管理所有权转换
- 提供指针操作

#### 2. `ForeignTypeRef` Trait

```rust
pub trait ForeignTypeRef: Sized {
    type CType;

    unsafe fn from_ptr<'a>(ptr: *mut Self::CType) -> &'a Self;
    unsafe fn from_ptr_mut<'a>(ptr: *mut Self::CType) -> &'a mut Self;
    fn as_ptr(&self) -> *mut Self::CType;
}
```

**功能**:
- 定义借用的行为
- 提供安全的指针转换

#### 3. `foreign_type!` 宏

```rust
foreign_type! {
    type CType = foo_sys::FOO;
    fn drop = foo_sys::FOO_free;
    fn clone = foo_sys::FOO_duplicate;

    pub struct Foo;       // Owned 类型
    pub struct FooRef;    // Borrowed 类型
}
```

**功能**:
- 自动生成 Owned 和 Borrowed 类型
- 实现 `Drop`、`Clone`、`Deref` 等 trait
- 减少样板代码

---

## 💡 使用示例

### 场景：包装 OpenSSL 的 X509 证书

**C API**:
```c
// OpenSSL C API
X509* X509_dup(X509 *x509);
void X509_free(X509 *x509);
```

**Rust 包装器**（使用 foreign-types）:
```rust
use foreign_types::foreign_type;

// 定义 C 类型（在 openssl-sys 中）
mod ffi {
    pub enum X509 {}

    extern "C" {
        pub fn X509_dup(x509: *mut X509) -> *mut X509;
        pub fn X509_free(x509: *mut X509);
    }
}

// 使用 foreign_types 宏创建包装器
foreign_type! {
    type CType = ffi::X509;
    fn drop = ffi::X509_free;
    fn clone = ffi::X509_dup;

    /// An X509 certificate.
    pub struct X509;
    /// A borrowed X509 certificate.
    pub struct X509Ref;
}
```

**使用方式**:
```rust
// Owned 类型 - 拥有所有权
let cert: X509 = unsafe {
    X509::from_ptr(ptr)  // 从 C 指针创建
};
// cert 离开作用域时自动调用 X509_free

// Borrowed 类型 - 借用
fn print_cert(cert: &X509Ref) {
    // cert 不拥有所有权，不会调用 X509_free
}
```

### 为什么需要 Owned 和 Borrowed 类型？

**Owned 类型**:
- ✅ 负责释放资源（通过 `Drop` trait）
- ✅ 可以移动和存储
- ✅ 可以克隆（如果实现了 `Clone`）
- ❌ 不能有多个引用

**Borrowed 类型**:
- ✅ 可以有多个引用
- ✅ 用于函数参数
- ❌ 不负责释放资源
- ❌ 不能移动或存储

**类比**:
```rust
// Rust 标准库的类似设计
String         -> Owned 类型，拥有数据
str            -> Borrowed 类型，借用数据

// foreign-types 的设计
Foo            -> Owned 类型，拥有 C 对象
FooRef         -> Borrowed 类型，借用 C 对象
```

---

## 📦 项目结构

### Workspace 结构

```
foreign-types/                    # Workspace 根目录
├── Cargo.toml                    # Workspace 配置
├── README.md
├── LICENSE-APACHE
├── LICENSE-MIT
├── foreign-types/                # 主 crate
│   ├── Cargo.toml               # 版本 0.3.2
│   ├── src/
│   │   └── lib.rs               # foreign_type! 宏（307 行）
│   └── BUILD.gn                 # OH 构建配置
├── foreign-types-shared/         # Shared crate
│   ├── Cargo.toml               # 版本 0.1.1
│   ├── src/
│   │   └── lib.rs               # ForeignType/ForeignTypeRef trait（52 行）
│   └── BUILD.gn                 # OH 构建配置
├── bundle.json                   # OH 组件配置
└── README.OpenSource             # OH 合规声明
```

### 为什么需要 Workspace？

**foreign-types** 是一个 Workspace，包含两个 crates：
1. `foreign-types`: 主 crate，提供 `foreign_type!` 宏
2. `foreign-types-shared`: 内部 crate，提供 trait

**分离的原因**:
- 避免循环依赖
- 共享 trait 定义
- 减少编译依赖

---

## 🔗 与 OpenHarmony 的关系

### 在 OH 中的定位

**角色**: 基础设施库（Infrastructure Library）

**用途**:
- 为 Rust FFI 编程提供框架
- 主要被 `rust-openssl` 使用
- 支持 OpenSSL C API 的安全封装

### OH 中的使用场景

**场景 1: OpenSSL 封装**
```rust
// rust-openssl 使用 foreign-types 封装 OpenSSL
use foreign_types::{ForeignType, ForeignTypeRef};

foreign_type! {
    type CType = ffi::SSL;
    fn drop = ffi::SSL_free;

    pub struct Ssl;
    pub struct SslRef;
}
```

**场景 2: TLS 连接管理**
```rust
// SSL 对象的 Owned 和 Borrowed 类型
let ssl: Ssl = Ssl::new(&ctx)?;  // Owned，拥有 SSL 对象
let stream = ssl.connect(stream)?;  // 移动所有权

// SSLRef 作为借用引用
fn read(ssl: &SslRef) -> Result<Vec<u8>> {
    // ssl 是借用引用，不负责释放
}
```

**场景 3: 证书管理**
```rust
// X509 证书的 Owned 和 Borrowed 类型
let cert: X509 = X509::from_pem(pem_data)?;  // Owned

fn verify_cert(cert: &X509Ref) -> bool {  // Borrowed
    // 验证逻辑
}
```

---

## 📊 技术特性

### 支持的特性

| 特性 | 支持情况 | 说明 |
|------|---------|------|
| **no_std** | ✅ 支持 | 可用于嵌入式环境 |
| **Owned 类型** | ✅ 支持 | 通过 `ForeignType` trait |
| **Borrowed 类型** | ✅ 支持 | 通过 `ForeignTypeRef` trait |
| **Clone 支持** | ✅ 可选 | 通过 `fn clone` 参数 |
| **Send/Sync** | ✅ 自动推导 | 基于 `CType` 的实现 |
| **Drop 语义** | ✅ 自动 | 自动调用 C 释放函数 |

### 不支持的特性

| 特性 | 不支持 | 原因 |
|------|--------|------|
| **异步支持** | ❌ | 该库专注于 FFI，不涉及异步 |
| **动态链接** | ❌ | 仅支持静态链接（rlib） |
| **C++ 支持** | ❌ | 专注于 C API |

---

## 🔍 与其他 FFI 库的对比

### vs bindgen

| 库 | 功能 | 适用场景 |
|---|------|---------|
| **foreign-types** | 提供框架，需要手动编写包装器 | 需要精细控制 FFI API 的行为 |
| **bindgen** | 自动生成 Rust 绑定 | 快速生成大型 C API 的绑定 |

**foreign-types 的优势**:
- ✅ 更清晰的 API 设计
- ✅ 更好的所有权语义
- ✅ 更灵活的封装方式

### vs cxx

| 库 | 功能 | 适用场景 |
|---|------|---------|
| **foreign-types** | 包装现有 C API | 与现有 C 代码集成 |
| **cxx** | 定义 C++/Rust 互操作 | 从头设计 C++/Rust 接口 |

**foreign-types 的优势**:
- ✅ 无需修改 C 代码
- ✅ 适用于纯 C API

---

## 📈 使用统计

### Crates.io 统计

| 指标 | 值 |
|------|-----|
| **总下载量** | > 50,000,000 (截至 2024) |
| **月下载量** | > 1,000,000 |
| **依赖者数量** | 300+ crates |

### 主要依赖者

| Crate | 用途 |
|-------|------|
| rust-openssl | OpenSSL 绑定 |
| rustls | TLS 实现 |
| libloading | 动态库加载 |

---

## 🎓 学习资源

### 官方文档

| 资源 | 链接 |
|------|------|
| API 文档 | https://docs.rs/foreign-types |
| GitHub 仓库 | https://github.com/sfackler/foreign-types |
| 源代码 | https://github.com/sfackler/foreign-types/tree/master/foreign-types/src |

### 推荐阅读

1. **Foreign Types in Rust**:
   - 了解 Rust 中包装外部类型的模式
   - 理解 Owned/Borrowed 类型的设计

2. **Rust FFI 最佳实践**:
   - https://doc.rust-lang.org/nomicon/ffi.html
   - Rust FFI 编程指南

3. **OpenSSL Rust 绑定示例**:
   - https://docs.rs/openssl
   - 查看实际使用 foreign-types 的案例

---

## 📝 快速入门

### 1. 添加依赖

**Cargo.toml**:
```toml
[dependencies]
foreign-types = "0.3"
```

### 2. 使用 foreign_type! 宏

```rust
use foreign_types::foreign_type;

// 定义 C 类型
mod sys {
    pub enum Foo {}

    extern "C" {
        pub fn foo_new() -> *mut Foo;
        pub fn foo_free(foo: *mut Foo);
        pub fn foo_clone(foo: *mut Foo) -> *mut Foo;
    }
}

// 创建包装器
foreign_type! {
    type CType = sys::Foo;
    fn drop = sys::foo_free;
    fn clone = sys::foo_clone;

    pub struct Foo;        // Owned 类型
    pub struct FooRef;     // Borrowed 类型
}
```

### 3. 使用包装器

```rust
use foreign_types::ForeignType;

// 创建 Owned 类型
let foo: Foo = unsafe {
    Foo::from_ptr(sys::foo_new())
};

// foo 离开作用域时自动调用 foo_free

// 使用 Borrowed 类型
fn do_something(foo: &FooRef) {
    // foo 是借用引用
}
```

---

## 📌 总结

### foreign-types 的价值

1. **安全**: 通过 Rust 类型系统确保正确的资源管理
2. **简洁**: 减少样板代码，提高可维护性
3. **灵活**: 支持多种 FFI 模式
4. **成熟**: 被广泛使用，经过充分测试

### 在 OpenHarmony 中的作用

1. **基础设施**: 为 Rust FFI 编程提供底层支持
2. **互操作性**: 促进 Rust 与 C 代码的集成
3. **安全性**: 确保外部资源的安全管理

### 为什么选择 foreign-types？

| 原因 | 说明 |
|------|------|
| **零成本抽象** | 无运行时开销 |
| **类型安全** | 利用 Rust 类型系统 |
| **社区支持** | 被广泛使用，有丰富的生态 |
| **易于维护** | 清晰的 API 设计 |
