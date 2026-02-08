# either - 原始库简介与 OH 定位

## 1. 原始库信息

### 1.1 基本信息

| 属性 | 值 |
|------|-----|
| **名称** | either |
| **版本** | 1.8.1 |
| **作者** | bluss |
| **许可证** | Apache-2.0 OR MIT |
| **仓库** | https://github.com/bluss/either |
| **文档** | https://docs.rs/either/1/ |

### 1.2 功能描述

**either** 是一个 Rust 库，提供 `Either<L, R>` 枚举类型，这是一个通用的**求和类型 (sum type)**，包含两个变体：

- `Left(L)` - 包含左值
- `Right(R)` - 包含右值

与 `Result<T, E>` 不同，`Either` **不区分成功/失败语义**，两个变体地位平等。

### 1.3 设计哲学

```rust
/// Either 的定义
pub enum Either<L, R> {
    Left(L),
    Right(R),
}
```

**适用场景**：
- 两种可能类型的值
- 需要统一处理不同类型的迭代器
- 需要统一处理不同的 IO 源
- 需要表示两种配置选项之一

**不适用场景**：
- 表示成功/失败结果 (应使用 `Result`)
- 可选值 (应使用 `Option`)

---

## 2. 核心功能

### 2.1 基本操作

```rust
use either::{Either, Left, Right};

let value: Either<i32, String> = Left(42);

// 判断变体
assert!(value.is_left());
assert!(!value.is_right());

// 转换为 Option
assert_eq!(value.left(), Some(42));
assert_eq!(value.right(), None);

// 映射
let doubled = value.map_left(|x| x * 2);
assert_eq!(doubled, Left(84));
```

### 2.2 宏支持

#### `for_both!` - 统一处理两侧

```rust
use either::Either;

fn length(e: Either<String, &'static str>) -> usize {
    either::for_both!(e, s => s.len())
}

assert_eq!(length(Left("hello".to_string())), 5);
assert_eq!(length(Right("world")), 5);
```

#### `try_left!` / `try_right!` - 短路返回

```rust
use either::{Either, Left, Right, try_left, try_right};

fn process(e: Either<i32, String>) -> Either<i32, String> {
    // 如果是 Right，提前返回
    let value = try_left!(e);
    Left(value * 2)
}

assert_eq!(process(Left(21)), Left(42));
assert_eq!(process(Right("error".to_string())), Right("error".to_string()));
```

### 2.3 Trait 实现

`Either<L, R>` 在 L 和 R 都实现某个 trait 时，自动实现该 trait：

| Trait | 说明 |
|-------|------|
| `Iterator` | 支持迭代 (next, fold, etc.) |
| `Read` | 支持读取 |
| `Write` | 支持写入 |
| `Seek` | 支持定位 |
| `BufRead` | 支持缓冲读取 |
| `Future` | 支持异步 |
| `Error` | 支持错误处理 |
| `Display` | 支持格式化输出 |
| `Clone`, `Copy` | 支持复制 |
| `AsRef`, `AsMut`, `Deref` | 支持引用转换 |

---

## 3. OpenHarmony 中的定位

### 3.1 在 OH 中的角色

**either** 在 OpenHarmony 中定位为**基础设施库**：

| 维度 | 说明 |
|------|------|
| **功能定位** | 基础数据类型库，提供 Either 类型 |
| **系统层级** | 底层支撑库 |
| **依赖范围** | 被工具链和少量第三方库依赖 |
| **维护策略** | 零修改，跟随上游 |

### 3.2 为什么需要这个库

1. **Rust 标准库缺失**: Rust 标准库未提供 Either 类型
2. **生态广泛**: Rust 生态中广泛使用的类型
3. **编译器依赖**: Rust 编译器内部依赖该库

### 3.3 典型使用场景

#### 场景 1：统一迭代器类型

```rust
use either::Either;

fn get_iter(condition: bool) -> Either<Vec<i32>::IntoIter, std::ops::Range<i32>> {
    if condition {
        Either::Left(vec![1, 2, 3].into_iter())
    } else {
        Either::Right(0..10)
    }
}

// 统一处理不同类型的迭代器
for item in get_iter(true) {
    println!("{}", item);
}
```

#### 场景 2：统一 IO 源

```rust
use either::Either;
use std::io::{self, Read};

fn get_reader(from_file: bool) -> Either<std::fs::File, &[u8]> {
    if from_file {
        Either::Left(std::fs::File::open("data.txt").unwrap())
    } else {
        Either::Right(b"static data")
    }
}

// 统一读取
let mut buf = [0u8; 1024];
get_reader(false).read(&mut buf).unwrap();
```

#### 场景 3：which-rs 中的实际使用

在 `which-rs` (Unix which 命令的 Rust 实现) 中，either 用于：

```rust
// 表示两种查找策略的结果
// - Left: 从环境变量 PATH 查找
// - Right: 从指定目录查找

pub fn which_in<T, U, V>(
    binary_name: T,
    paths: Option<U>,
    cwd: V,
) -> Either<PathBuf, WhichError>
where
    T: AsRef<OsStr>,
    U: AsRef<OsStr>,
    V: AsRef<Path>,
{
    // 实现...
}
```

---

## 4. 版本信息

### 4.1 版本对应关系

| 类型 | 版本 | 说明 |
|------|------|------|
| **上游 crate** | 1.8.1 | GitHub 发布的版本 |
| **OH 组件** | 6.1 | bundle.json 中定义的 OH 组件版本 |
| **Cargo.toml** | 1.8.1 | 源码中的 crate 版本 |

### 4.2 版本差异说明

**OH 组件版本 (6.1)** 与 **上游版本 (1.8.1)** 的差异：

- OH 组件版本是 OpenHarmony 内部的组件版本号
- 上游版本是 Rust crate 的版本号
- 两者独立演进

### 4.3 最低 Rust 版本要求

- **MSRV**: Rust 1.36+
- **OH 当前**: 使用 Rust 1.7x+ (满足要求)

---

## 5. 许可证合规

### 5.1 许可证信息

- **主许可证**: Apache-2.0 OR MIT (双许可)
- **许可证文件**: LICENSE-APACHE, LICENSE-MIT
- **兼容性**: ✅ 与 OpenHarmony 兼容

### 5.2 OAT 配置

```xml
<licensefile>LICENSE-APACHE|LICENSE-MIT</licensefile>
```

OAT (OSS Audit Tool) 已正确配置，扫描通过。

---

## 6. 相关资源

- [上游仓库](https://github.com/bluss/either)
- [crates.io 页面](https://crates.io/crates/either)
- [API 文档](https://docs.rs/either/1/)
- [CHANGELOG](./CHANGELOG.md) (如有)
