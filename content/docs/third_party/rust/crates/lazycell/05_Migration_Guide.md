# 迁移指南

> **说明**: 本文档说明如何从 lazycell 迁移到现代替代方案（once_cell、LazyLock）。
> **核心内容**: 迁移步骤、API 对比、最佳实践。

---

## 迁移决策

### 是否需要迁移？

| 场景 | 建议 |
|------|------|
| **现有稳定代码** | ❌ 无需迁移，继续使用 lazycell |
| **新项目** | ✅ 使用 once_cell 或 LazyLock |
| **大规模重构** | ⚠️ 可考虑迁移，但需评估收益 |
| **工具链代码** | ⚠️ 依赖 Rust 版本，需谨慎评估 |

### 迁移收益

| 收益 | lazycell | once_cell | std::sync::LazyLock |
|------|----------|-----------|---------------------|
| **现代 API** | ❌ | ✅ | ✅ |
| **标准库支持** | ❌ | ❌ | ✅（Rust 1.80+） |
| **外部依赖** | 是 | 是 | 否 |
| **维护成本** | 低 | 低 | 最低 |
| **社区支持** | 低 | 高 | 最高 |

---

## 迁移路径

### 路径 A: lazycell → once_cell

**适用场景**:
- 新项目
- 需要更丰富的 API
- Rust 版本 < 1.80

#### 1. 添加依赖

**Cargo.toml**

```toml
[dependencies]
once_cell = "1.18"
```

**BUILD.gn**

```gn
ohos_cargo_crate("my_crate") {
    deps = [
        "//third_party/rust/crates/once_cell:lib"
    ]
}
```

#### 2. API 迁移映射

| lazycell | once_cell |
|----------|-----------|
| `LazyCell` | `once_cell::unsync::LazyCell` |
| `AtomicLazyCell` | `once_cell::sync::LazyCell` |
| `LazyCell::new()` | `once_cell::unsync::LazyCell::new()` |
| `AtomicLazyCell::new()` | `once_cell::sync::LazyCell::new()` |
| `LazyCell::fill()` | 不支持（使用 `new()` + 闭包） |
| `AtomicLazyCell::fill()` | 不支持（使用 `new()` + 闭包） |
| `LazyCell::borrow()` | `Deref`（直接 `*lazy_cell`） |
| `AtomicLazyCell::borrow()` | `Deref`（直接 `*lazy_cell`） |
| `LazyCell::borrow_with()` | `once_cell::unsync::LazyCell::get_or_init()` |
| `AtomicLazyCell::borrow_with()` | `once_cell::sync::LazyCell::get_or_init()` |
| `LazyCell::filled()` | 无直接替代 |
| `AtomicLazyCell::filled()` | 无直接替代 |
| `LazyCell::into_inner()` | `once_cell::unsync::LazyCell::take()` |
| `AtomicLazyCell::into_inner()` | `once_cell::sync::LazyCell::take()` |

#### 3. 迁移示例

##### 单线程：lazycell → once_cell::unsync

**lazycell**

```rust
use lazycell::LazyCell;

struct ResourceManager {
    config: LazyCell<Config>,
}

impl ResourceManager {
    fn get_config(&self) -> &Config {
        self.config.borrow_with(|| {
            Config::load_from_file()
        })
    }
}
```

**once_cell::unsync**

```rust
use once_cell::unsync::Lazy;

struct ResourceManager {
    config: Lazy<Config>,
}

impl ResourceManager {
    fn new() -> Self {
        Self {
            config: Lazy::new(|| Config::load_from_file())
        }
    }

    fn get_config(&self) -> &Config {
        &self.config
    }
}
```

**关键差异**:
- once_cell 在创建时提供初始化闭包，lazycell 在访问时提供
- once_cell 通过 `Deref` 自动解引用

##### 多线程：lazycell → once_cell::sync

**lazycell**

```rust
use lazycell::AtomicLazyCell;

static CONFIG: AtomicLazyCell<Config> = AtomicLazyCell::NONE;

fn get_config() -> &'static Config {
    CONFIG.borrow_with(|| {
        Config::load_from_file()
    })
}
```

**once_cell::sync**

```rust
use once_cell::sync::Lazy;

static CONFIG: Lazy<Config> = Lazy::new(|| {
    Config::load_from_file()
});

fn get_config() -> &'static Config {
    &CONFIG
}
```

**关键差异**:
- once_cell 的语法更简洁
- 自动线程安全，无需手动管理原子操作

---

### 路径 B: lazycell → std::sync::LazyLock（Rust 1.80+）

**适用场景**:
- Rust 版本 >= 1.80
- 新项目
- 希望使用标准库

#### 1. 无需添加依赖

`std::sync::LazyLock` 是 Rust 标准库的一部分，无需添加依赖。

#### 2. API 迁移映射

| lazycell | std::sync::LazyLock |
|----------|---------------------|
| `AtomicLazyCell` | `std::sync::LazyLock` |
| `AtomicLazyCell::new()` | `std::sync::LazyLock::new()` |
| `AtomicLazyCell::borrow_with()` | 不支持（使用 `new()` + 闭包） |
| `AtomicLazyCell::borrow()` | `Deref`（直接 `*lazy_lock`） |
| `AtomicLazyCell::filled()` | 无直接替代 |
| `AtomicLazyCell::into_inner()` | 无直接替代 |

**注意**: `LazyLock` 仅提供线程安全的静态初始化，无单线程版本。

#### 3. 迁移示例

**lazycell**

```rust
use lazycell::AtomicLazyCell;

static CONFIG: AtomicLazyCell<Config> = AtomicLazyCell::NONE;

fn get_config() -> &'static Config {
    CONFIG.borrow_with(|| {
        Config::load_from_file()
    })
}
```

**std::sync::LazyLock**

```rust
use std::sync::LazyLock;

static CONFIG: LazyLock<Config> = LazyLock::new(|| {
    Config::load_from_file()
});

fn get_config() -> &'static Config {
    &CONFIG
}
```

---

## 迁移检查清单

### 迁移前检查

- [ ] 确认 Rust 版本是否支持目标方案
- [ ] 检查是否使用了 lazycell 特有 API（如 `fill()`、`filled()`）
- [ ] 评估迁移的工作量和收益
- [ ] 确认依赖方（bindgen、compiletest）的迁移需求

### 迁移步骤

- [ ] 添加新依赖（once_cell）或确认标准库支持（LazyLock）
- [ ] 更新导入语句
- [ ] 替换 API 调用
- [ ] 运行测试
- [ ] 检查编译警告
- [ ] 代码审查

### 迁移后验证

- [ ] 所有测试通过
- [ ] 无性能退化
- [ ] 无新的编译警告
- [ ] 更新文档（如有）

---

## 特殊场景处理

### 1. 使用 `fill()` 方法

**lazycell**

```rust
let cell = LazyCell::new();

// 后续手动填充
if some_condition {
    cell.fill(value).ok();
}

if let Some(value) = cell.borrow() {
    // 使用 value
}
```

**once_cell**

```rust
use once_cell::unsync::OnceCell;

let cell = OnceCell::new();

// 后续手动填充
if some_condition {
    cell.set(value).ok();
}

if let Some(value) = cell.get() {
    // 使用 value
}
```

**说明**: `once_cell::OnceCell` 的 `set()` 方法对应 lazycell 的 `fill()`。

### 2. 使用 `filled()` 方法

**lazycell**

```rust
if !cell.filled() {
    // 延迟初始化
    cell.fill(value).ok();
}
```

**once_cell**

```rust
if cell.get().is_none() {
    // 延迟初始化
    cell.set(value).ok();
}
```

**说明**: once_cell 通过 `get()` 返回 `Option` 来替代 `filled()`。

### 3. 使用 `borrow_mut_with()`

**lazycell**

```rust
let mut cell = LazyCell::new();

cell.borrow_mut_with(|| {
    compute_mutable_value()
});
```

**once_cell**

```rust
use once_cell::unsync::OnceCell;

let cell = OnceCell::new();

cell.get_or_init(|| {
    compute_mutable_value()
});

// once_cell 不支持可变借用
// 需要重新设计代码逻辑
```

**说明**: once_cell 不支持可变借用，需要重新设计代码。

---

## 最佳实践

### 1. 新项目

```rust
// 使用 once_cell（推荐）
use once_cell::sync::Lazy;

static CONFIG: Lazy<Config> = Lazy::new(|| Config::load());

fn main() {
    let config = &*CONFIG;
}
```

### 2. 现有代码

```rust
// 继续使用 lazycell（如果代码稳定）
use lazycell::AtomicLazyCell;

static CONFIG: AtomicLazyCell<Config> = AtomicLazyCell::NONE;

fn get_config() -> &'static Config {
    CONFIG.borrow_with(|| Config::load())
}
```

### 3. Rust 1.80+ 项目

```rust
// 使用标准库（最佳）
use std::sync::LazyLock;

static CONFIG: LazyLock<Config> = LazyLock::new(|| Config::load());

fn main() {
    let config = &*CONFIG;
}
```

---

## 常见问题

### Q1: lazycell 和 once_cell 哪个更好？

**A**: 取决于场景：
- **新项目**: once_cell（API 更现代）
- **现有代码**: lazycell（无需迁移）
- **Rust 1.80+**: std::sync::LazyLock（标准库）

### Q2: 迁移后性能会变化吗？

**A**: 理论上性能相同或略优：
- once_cell 实现更优化
- LazyLock 是标准库优化版本
- 实际性能差异通常可忽略

### Q3: 如何选择迁移时机？

**A**: 建议在以下时机迁移：
- 大规模重构时
- API 清理时
- 升级 Rust 版本时

### Q4: bindgen 和 compiletest 需要迁移吗？

**A**:
- **短期内**: 无需强制迁移
- **长期**: 建议跟随上游更新
- **工作量大**: 需要修改 2 个模块

---

## 参考资源

- **once_cell 文档**: https://docs.rs/once_cell
- **std::sync::LazyLock 文档**: https://doc.rust-lang.org/std/sync/struct.LazyLock.html
- **lazycell 文档**: https://indiv0.github.io/lazycell/lazycell
- **完整评估报告**: [_work/ASSESSMENT.md](_work/ASSESSMENT.md)
