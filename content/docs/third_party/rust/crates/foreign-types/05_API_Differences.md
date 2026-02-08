# 05_API_Differences.md - API/接口差异

本文档说明 OpenHarmony 中的 `foreign-types` 库与上游版本的 API 差异。

> **核心结论**: OH 未修改任何源代码，API 与上游版本完全一致，无差异。

---

## 📋 差异概览

| 差异类型 | 数量 | 说明 |
|---------|------|------|
| **OH 新增 API** | 0 | 无 |
| **行为变更的 API** | 0 | 无 |
| **废弃的 API** | 0 | 无 |
| **禁用的功能** | 0 | 无 |
| **源代码修改** | 0 | 完全一致 |

**结论**: 🟢 **无 API 差异**

---

## 🔍 源代码对比

### 文件完整性检查

| 文件 | 上游 | OH | 状态 |
|------|------|----|:----:|
| `foreign-types/src/lib.rs` | ✅ | ✅ | ✅ 一致 |
| `foreign-types-shared/src/lib.rs` | ✅ | ✅ | ✅ 一致 |
| `foreign-types/Cargo.toml` | ✅ | ✅ | ✅ 一致 |
| `foreign-types-shared/Cargo.toml` | ✅ | ✅ | ✅ 一致 |

**说明**: 所有 Rust 源文件与上游完全一致。

### 代码对比结果

**方法**:
```bash
# 对比 OH 和上游的源文件
diff -u <upstream>/foreign-types/src/lib.rs \
        <oh>/foreign-types/src/lib.rs

# 结果：无差异
```

**结论**: 🟢 源代码零差异

---

## 📦 OH 特有的修改（非 API）

### 新增文件（不影响 API）

OH 仅新增了构建和合规文件，这些文件不影响 API：

| 文件 | 类型 | 影响 |
|------|------|------|
| `README.OpenSource` | 合规声明 | ❌ 不影响 API |
| `bundle.json` | 构建配置 | ❌ 不影响 API |
| `foreign-types/BUILD.gn` | 构建配置 | ❌ 不影响 API |
| `foreign-types-shared/BUILD.gn` | 构建配置 | ❌ 不影响 API |

**说明**: 这些文件是构建系统适配，不修改任何公开 API。

---

## 🔧 构建系统差异（非 API）

### BUILD.gn vs Cargo.toml

虽然构建系统不同，但 API 完全一致：

| 维度 | 上游（Cargo） | OH（GN） | API 影响 |
|------|---------------|----------|---------|
| **构建命令** | `cargo build` | `ninja` | ❌ 不影响 |
| **依赖路径** | 相对路径 | GN 路径 | ❌ 不影响 |
| **输出类型** | 自动识别 | 显式声明 | ❌ 不影响 |
| **API 行为** | 一致 | 一致 | ✅ 一致 |

**说明**: 构建系统差异是内部实现细节，不影响公开 API 的行为。

---

## 📋 API 完整性检查

### 公开的 Trait

#### ForeignType Trait

**签名**:
```rust
pub trait ForeignType: Sized {
    type CType;
    type Ref: ForeignTypeRef<CType = Self::CType>;

    unsafe fn from_ptr(ptr: *mut Self::CType) -> Self;
    fn as_ptr(&self) -> *mut Self::CType;
}
```

**OH vs 上游**:
| 方法 | 上游 | OH | 状态 |
|------|------|----|:----:|
| `from_ptr` | ✅ | ✅ | ✅ 一致 |
| `as_ptr` | ✅ | ✅ | ✅ 一致 |

#### ForeignTypeRef Trait

**签名**:
```rust
pub trait ForeignTypeRef: Sized {
    type CType;

    unsafe fn from_ptr<'a>(ptr: *mut Self::CType) -> &'a Self;
    unsafe fn from_ptr_mut<'a>(ptr: *mut Self::CType) -> &'a mut Self;
    fn as_ptr(&self) -> *mut Self::CType;
}
```

**OH vs 上游**:
| 方法 | 上游 | OH | 状态 |
|------|------|----|:----:|
| `from_ptr` | ✅ | ✅ | ✅ 一致 |
| `from_ptr_mut` | ✅ | ✅ | ✅ 一致 |
| `as_ptr` | ✅ | ✅ | ✅ 一致 |

### 宏

#### foreign_type! 宏

**签名**:
```rust
macro_rules! foreign_type {
    (
        $(#[$impl_attr:meta])*
        type CType = $ctype:ty;
        fn drop = $drop:expr;
        $(fn clone = $clone:expr;)*
        $(#[$owned_attr:meta])*
        pub struct $owned:ident;
        $(#[$borrowed_attr:meta])*
        pub struct $borrowed:ident;
    ) => { /* ... */ }
}
```

**OH vs 上游**:
| 参数 | 上游 | OH | 状态 |
|------|------|----|:----:|
| `CType` | ✅ | ✅ | ✅ 一致 |
| `drop` | ✅ | ✅ | ✅ 一致 |
| `clone` | ✅ | ✅ | ✅ 一致 |
| `owned` | ✅ | ✅ | ✅ 一致 |
| `borrowed` | ✅ | ✅ | ✅ 一致 |

### 自动实现的 Trait

| Trait | 上游 | OH | 状态 |
|-------|------|----|:----:|
| `Drop` | ✅ | ✅ | ✅ 一致 |
| `Clone` | ✅（可选） | ✅（可选） | ✅ 一致 |
| `Deref` | ✅ | ✅ | ✅ 一致 |
| `DerefMut` | ✅ | ✅ | ✅ 一致 |
| `Borrow` | ✅ | ✅ | ✅ 一致 |
| `AsRef` | ✅ | ✅ | ✅ 一致 |
| `From` | ✅ | ✅ | ✅ 一致 |
| `Send` | ✅（自动推导） | ✅（自动推导） | ✅ 一致 |
| `Sync` | ✅（自动推导） | ✅（自动推导） | ✅ 一致 |

---

## 🌓 版本差异

### 版本号对比

| 版本 | 值 | 说明 |
|------|-----|------|
| **上游版本** | 0.3.2 | Rust crate 版本 |
| **OH 版本** | 0.3.2 | Rust crate 版本 |
| **OH 组件版本** | 6.1 | bundle.json 版本 |

**结论**: Rust crate 版本一致，API 完全相同。

### 潜在的次要版本差异

如果未来 OH 使用与上游不同的版本，可能出现的差异：

| 差异类型 | 说明 | API 影响 |
|---------|------|---------|
| **补丁版本** (0.3.2 -> 0.3.3) | Bug 修复 | ❌ 无影响 |
| **次要版本** (0.3.2 -> 0.4.0) | 新增功能 | ❌ 向后兼容 |
| **主要版本** (0.3.2 -> 1.0.0) | 破坏性变更 | ⚠️ 可能影响 |

**当前状态**: OH 使用 0.3.2，与上游一致，无任何差异。

---

## 🚫 不存在的差异

### 常见但不适用于此库的差异

这些差异在其他 OH 库中可能出现，但**不适用于** foreign-types：

| 差异类型 | foreign-types 状态 | 说明 |
|---------|-------------------|------|
| **条件编译** | ❌ 无 | foreign-types 是纯 Rust 库 |
| **平台特定代码** | ❌ 无 | 与平台无关 |
| **OH 特定宏** | ❌ 无 | 未添加 OH 宏 |
| **API 扩展** | ❌ 无 | 未添加任何新 API |
| **行为修改** | ❌ 无 | 所有行为一致 |

---

## 📊 兼容性分析

### 向后兼容性

| 场景 | 兼容性 | 说明 |
|------|--------|------|
| **上游升级到 0.3.x** | ✅ 兼容 | 补丁版本向后兼容 |
| **上游升级到 0.4.x** | ✅ 可能兼容 | 次要版本通常向后兼容 |
| **上游升级到 1.0.x** | ⚠️ 需检查 | 主要版本可能有破坏性变更 |

### 前向兼容性

| 场景 | 兼容性 | 说明 |
|------|--------|------|
| **依赖方代码迁移** | ✅ 无需修改 | API 完全一致 |
| **构建系统切换** | ✅ 无影响 | 仅内部实现 |

---

## 📝 使用示例对比

### 示例 1: 定义 Foreign Type

**上游代码**:
```rust
use foreign_types::foreign_type;

foreign_type! {
    type CType = foo::FOO;
    fn drop = foo::FOO_free;

    pub struct Foo;
    pub struct FooRef;
}
```

**OH 代码**: ✅ **完全相同**

### 示例 2: 使用 Owned 类型

**上游代码**:
```rust
use foreign_types::ForeignType;

let foo: Foo = unsafe { Foo::from_ptr(ptr) };
```

**OH 代码**: ✅ **完全相同**

### 示例 3: 使用 Borrowed 类型

**上游代码**:
```rust
use foreign_types::ForeignTypeRef;

fn do_something(foo: &FooRef) { }
```

**OH 代码**: ✅ **完全相同**

---

## 🔍 实际使用验证

### rust-openssl 中的使用

**依赖声明**:
```toml
# rust-openssl/Cargo.toml
foreign-types = "0.3.1"
```

**使用代码**:
```rust
// rust-openssl/src/ssl/mod.rs
use foreign_types::{ForeignType, ForeignTypeRef};

foreign_type! {
    type CType = ffi::SSL;
    fn drop = ffi::SSL_free;

    pub struct Ssl;
    pub struct SslRef;
}
```

**验证**: rust-openssl 在 OH 中的使用与上游完全一致。

---

## 📌 结论

### 核心结论

1. **✅ 无 API 差异**: OH 版本与上游版本 API 完全一致
2. **✅ 源代码未修改**: 所有 Rust 源文件与上游相同
3. **✅ 行为一致**: 所有 trait 和宏的行为相同
4. **✅ 向后兼容**: OH 使用与上游相同的版本

### 维护建议

1. **升级简单**: 升级上游版本仅需更新版本号
2. **无需适配**: 无需为 OH 特殊修改代码
3. **保持同步**: 建议与上游版本保持一致

### 总结

foreign-types 是 OH 中**最干净**的第三方库之一：
- ✅ 零源代码修改
- ✅ 零 API 差异
- ✅ 零行为变更
- ✅ 零 OH 特有功能

这种"零侵入"模式是 OH 集成第三方库的理想状态。

---

## 🔗 相关文档

- [02_Patches.md](./02_Patches.md): Patch 详细分析（无源代码修改）
- [03_Build_Integration.md](./03_Build_Integration.md): 构建系统适配
- [01_Overview.md](./01_Overview.md): 原始库简介
