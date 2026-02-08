# lazycell 原始库简介

> **说明**: 本文档简要介绍 lazycell 原始库的基本功能和 API。
> **重点**: 该库在 OH 中的作用和定位。
> **详细文档**: 请参考 [上游 API 文档](https://indiv0.github.io/lazycell/lazycell)

---

## 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | lazycell |
| **版本** | 1.2.1 |
| **许可证** | MIT / Apache-2.0（双重许可） |
| **上游地址** | https://github.com/indiv0/lazycell |
| **作者** | Alex Crichton, Nikita Pekin |
| **代码行数** | 650 行（含测试） |

---

## 核心功能

lazycell 提供延迟初始化的 Cell 结构体，允许在首次访问时才计算值，之后缓存结果供重复使用。

### 两种类型

| 类型 | 线程安全 | 内部实现 | 适用场景 |
|------|----------|----------|----------|
| `LazyCell<T>` | ❌ 否 | `UnsafeCell<Option<T>>` | 单线程延迟初始化 |
| `AtomicLazyCell<T>` | ✅ 是 | `UnsafeCell<Option<T>>` + `AtomicUsize` | 多线程延迟初始化 |

---

## 主要 API

### LazyCell API（单线程）

```rust
use lazycell::LazyCell;

let lazycell = LazyCell::new();

// 填充值
lazycell.fill(1).ok();

// 检查是否已填充
assert!(lazycell.filled());

// 借用值
assert_eq!(lazycell.borrow(), Some(&1));

// 延迟计算
let value = lazycell.borrow_with(|| {
    // 首次调用时执行
    expensive_computation()
});

// 消费并获取内部值
let value = lazycell.into_inner();
```

### AtomicLazyCell API（线程安全）

```rust
use lazycell::AtomicLazyCell;

static CONFIG: AtomicLazyCell<Config> = AtomicLazyCell::NONE;

// 线程安全地延迟初始化
fn get_config() -> &'static Config {
    CONFIG.borrow_with(|| {
        load_config_from_file()
    })
}

// 多个线程可以同时调用 get_config()
// 只有第一个线程会执行 load_config_from_file()
```

---

## 典型使用场景

### 1. 延迟初始化全局配置

```rust
use lazycell::AtomicLazyCell;

static DATABASE_CONFIG: AtomicLazyCell<Config> = AtomicLazyCell::NONE;

fn get_db_config() -> &'static Config {
    DATABASE_CONFIG.borrow_with(|| {
        Config::from_env()
    })
}
```

### 2. 缓存昂贵计算结果

```rust
use lazycell::LazyCell;

struct Parser {
    cache: LazyCell<SyntaxTree>,
}

impl Parser {
    fn parse(&self) -> &SyntaxTree {
        self.cache.borrow_with(|| {
            // 只在首次调用时解析
            expensive_parsing()
        })
    }
}
```

### 3. 按需加载资源

```rust
use lazycell::LazyCell;

struct ResourceManager {
    image: LazyCell<Image>,
    sound: LazyCell<Sound>,
}

impl ResourceManager {
    fn get_image(&self) -> Option<&Image> {
        self.image.borrow()
    }

    fn load_image(&self) -> Result<(), Image> {
        self.image.fill(Image::load("image.png"))
    }
}
```

---

## lazycell 在 OH 中的作用和定位

### OH 中的定位

lazycell 是 OH Rust 生态中的**基础设施库**，主要用于：

1. **工具链内部**: Rust 编译器测试工具（compiletest）
2. **FFI 绑定生成**: bindgen 库
3. **延迟初始化模式**: 在 OH 中提供标准的延迟初始化能力

### OH 中的特点

| 特点 | 说明 |
|------|------|
| **零 Patch 集成** | 无任何 OH 特定修改 |
| **最小化依赖** | 仅被少数模块使用 |
| **维护成本低** | 无本地修改，升级无障碍 |
| **稳定性高** | 代码量小、逻辑简单、历史记录良好 |

---

## API 对比表

| API | LazyCell | AtomicLazyCell |
|-----|-----------|----------------|
| `new()` | ✅ | ✅ |
| `fill(value)` | ✅ | ✅ |
| `filled()` | ✅ | ✅ |
| `borrow()` | ✅ | ✅ |
| `borrow_mut()` | ✅ | ❌ |
| `borrow_with(f)` | ✅ | ❌ |
| `borrow_mut_with(f)` | ✅ | ❌ |
| `try_borrow_with(f)` | ✅ | ❌ |
| `replace(value)` | ✅ (mut) | ✅ (mut) |
| `into_inner()` | ✅ | ✅ |
| `Sync` | ❌ | ✅ (if T: Send + Sync) |
| `Send` | ✅ (if T: Send) | ✅ (if T: Send) |

---

## 与标准库和 once_cell 的对比

| 特性 | lazycell | once_cell | std::sync::LazyLock |
|------|----------|-----------|---------------------|
| **单线程延迟初始化** | `LazyCell` | `unsync::LazyCell` | ❌ |
| **多线程延迟初始化** | `AtomicLazyCell` | `sync::LazyLock` | `LazyLock` |
| **线程安全** | 手动实现 | 手动实现 | 标准库保证 |
| **API 丰富度** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **稳定版本** | Rust 1.15+ | Rust 1.0+ | Rust 1.80+ |
| **外部依赖** | 是 | 是 | 否（标准库） |

**建议**: OH 内新代码优先使用 `once_cell` 或 `std::sync::LazyLock`。

详见 **[05_Migration_Guide.md](05_Migration_Guide.md)**。

---

## 参考资料

- **上游仓库**: https://github.com/indiv0/lazycell
- **API 文档**: https://indiv0.github.io/lazycell/lazycell
- **Cargo 页面**: https://crates.io/crates/lazycell
- **OH 中的使用**: 见 **[04_Usage_in_OH.md](04_Usage_in_OH.md)**
