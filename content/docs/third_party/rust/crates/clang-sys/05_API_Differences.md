# API/接口差异

## 核心结论

**clang-sys 在 OpenHarmony 中未对 API 做任何修改。**

由于未应用任何 Patch，且未添加 OH 特有代码，clang-sys 的 API 与上游完全一致。

## API 状态

### 接口变更清单

| API 类型 | 变更 | 说明 |
|---------|-----|-----|
| 函数 | 无 | 与上游 v1.4.0 完全一致 |
| 类型 | 无 | 与上游 v1.4.0 完全一致 |
| 常量 | 无 | 与上游 v1.4.0 完全一致 |
| 特性 (features) | 配置差异 | BUILD.gn 中启用的 features 不同 |

### 与上游版本对比

```
上游 clang-sys v1.4.0
    ↓
无代码变更
    ↓
OH clang-sys (BUILD.gn 配置适配)
    ↓
API 100% 兼容
```

## 可用 API 概览

### 核心模块

虽然 OH 未修改 API，但了解 clang-sys 提供的 API 有助于理解其使用方式。

#### 1. lib.rs - FFI 绑定

```rust
// 主要类型定义
pub type CXClientData = *mut c_void;
pub type CXCursorVisitor = extern "C" fn(CXCursor, CXCursor, CXClientData) -> CXChildVisitResult;
pub type CXFieldVisitor = extern "C" fn(CXCursor, CXClientData) -> CXVisitorResult;
pub type CXInclusionVisitor = extern "C" fn(CXFile, *mut CXSourceLocation, c_uint, CXClientData);

// 枚举定义示例
cenum! {
    enum CXAvailabilityKind {
        const CXAvailability_Available = 0,
        const CXAvailability_Deprecated = 1,
        const CXAvailability_NotAvailable = 2,
        const CXAvailability_NotAccessible = 3,
    }
}
```

#### 2. support.rs - 版本支持

```rust
pub fn is_available(clang: Clang) -> bool;
pub fn get_version() -> Option<CXVersion>;
```

#### 3. link.rs - 链接宏

提供内部使用的宏定义，用于条件编译。

### Feature-gated API

由于 OH BUILD.gn 启用了特定 features，以下 API 可用：

#### 启用的 Features

| Feature | 说明 | OH 状态 |
|--------|-----|---------|
| `clang_3_5` | Clang 3.5 API | ✓ 启用 |
| `clang_3_6` | Clang 3.6 新增 API | ✓ 启用 |
| `clang_3_7` | Clang 3.7 新增 API | ✓ 启用 |
| `clang_3_8` | Clang 3.8 新增 API | ✓ 启用 |
| `clang_3_9` | Clang 3.9 新增 API | ✓ 启用 |
| `clang_4_0` | Clang 4.0 新增 API | ✓ 启用 |
| `clang_5_0` | Clang 5.0 新增 API | ✓ 启用 |
| `clang_6_0` | Clang 6.0 新增 API | ✓ 启用 |
| `libloading` | 动态库加载 | ✓ 启用 |
| `static` | 静态链接 | ✓ 启用 |

#### 未启用的 Features

| Feature | 说明 | OH 状态 |
|--------|-----|---------|
| `clang_7_0` - `clang_16_0` | Clang 7.0+ API | ✗ 未启用 |
| `runtime` | 运行时加载模式 | ✗ 未启用 |

### Feature 差异导致的 API 可用性

#### CXCursor 相关 API

```rust
// clang_3_7+ 启用
#[cfg(feature = "clang_3_7")]
pub type CXFieldVisitor = extern "C" fn(CXCursor, CXClientData) -> CXVisitorResult;
```

OH 可用（启用了 clang_3_7）。

#### Clang 6.0 特有 API

```rust
// clang_6_0 feature 启用的 API
#[cfg(feature = "clang_6_0")]
extern "C" {
    pub fn clang_Cursor_getObjCPropertyAttributes(C: CXCursor, reserved: c_uint) -> CXString;
    // ...
}
```

OH 可用（启用了 clang_6_0）。

#### Clang 15.0+ API

```rust
// clang_15_0 feature
#[cfg(feature = "clang_15_0")]
extern "C" {
    pub fn clang_getUnqualifiedType(CT: CXType) -> CXType;
    // ...
}
```

OH **不可用**（未启用 clang_15_0）。

## 行为差异

### 无行为差异

由于无代码修改，clang-sys 的行为与上游完全一致。

### 配置导致的行为差异

虽然 API 相同，但 BUILD.gn 配置会影响运行行为：

| 方面 | 上游默认 | OH BUILD.gn | 影响 |
|-----|---------|-------------|-----|
| 链接方式 | 动态链接 | 静态链接（通过 feature） | 运行时依赖不同 |
| 版本支持 | 3.5 默认 | 3.5-6.0 | API 可用范围 |

## 废弃功能

### 上游废弃

clang-sys 作为绑定库，遵循 libclang 的废弃策略：

| 功能 | 状态 | 说明 |
|-----|-----|-----|
| `gte_clang_*` features | 已移除 | v1.0.0 移除，曾是实现细节 |

OH 未使用这些已移除的功能。

## 开发者指南

### 如何判断 API 是否可用

在 OH 中使用 clang-sys 时，检查以下两点：

1. **上游文档**: 参考 https://docs.rs/clang-sys/1.4.0/clang_sys/
2. **Feature 条件**: 确认 API 所需的 feature 是否在 BUILD.gn 中启用

### 代码示例

```rust
use clang_sys::*;

fn main() {
    // 检查 libclang 是否可用
    if support::is_available() {
        println!("libclang is available");
    }
    
    // 使用基本 API（clang_3_5+）
    unsafe {
        let index = clang_createIndex(0, 0);
        // ...
        clang_disposeIndex(index);
    }
    
    // 使用 clang_3_7+ API
    #[cfg(feature = "clang_3_7")]
    {
        // 这里可以使用 clang_3_7 特有的 API
    }
}
```

### 添加新 API 支持

如果需要使用更高版本的 Clang API：

1. **修改 BUILD.gn**:
```gn
features = [
    # ... 现有 features
    "clang_7_0",  # 添加新 feature
]
```

2. **重新构建**:
```bash
hb build //third_party/rust/crates/clang-sys:lib
```

3. **验证 bindgen** 兼容性

## 与上游兼容性

### 代码兼容性

| 场景 | 兼容性 | 说明 |
|-----|-------|-----|
| 上游示例代码 | 100% | 直接可用 |
| crates.io 依赖 | 100% | 版本匹配即可 |
| 文档示例 | 100% | 直接可用 |

### 版本锁定

OH 当前使用：
- **上游版本**: v1.4.0
- **API 版本**: 对应 Clang 3.5-6.0

如需使用 bindgen 等下游库，注意版本匹配。
