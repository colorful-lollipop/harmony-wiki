# either - API 差异

## 概述

**either** 库在 OpenHarmony 中的 API 与上游完全一致，**无任何 API 差异**。

---

## 1. API 状态

| 项目 | 状态 |
|------|------|
| **API 修改** | 无 |
| **API 扩展** | 无 |
| **API 废弃** | 无 |
| **行为变更** | 无 |

**结论**: 零修改集成，API 100% 与上游一致。

---

## 2. 可用 API

### 2.1 完整 API 清单

由于无修改，以下是 either 1.8.1 提供的完整 API：

#### 枚举定义

```rust
pub enum Either<L, R> {
    Left(L),
    Right(R),
}
```

#### 构造方法

| 方法 | 说明 |
|------|------|
| `Either::Left(l)` | 创建 Left 变体 |
| `Either::Right(r)` | 创建 Right 变体 |

#### 查询方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `is_left` | `fn is_left(&self) -> bool` | 判断是否为 Left |
| `is_right` | `fn is_right(&self) -> bool` | 判断是否为 Right |

#### 转换方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `left` | `fn left(self) -> Option<L>` | 转为 Option<L> |
| `right` | `fn right(self) -> Option<R>` | 转为 Option<R> |
| `as_ref` | `fn as_ref(&self) -> Either<&L, &R>` | 转为引用 |
| `as_mut` | `fn as_mut(&mut self) -> Either<&mut L, &mut R>` | 转为可变引用 |
| `as_pin_ref` | `fn as_pin_ref(self: Pin<&Self>) -> ...` | Pin 投影 |
| `as_pin_mut` | `fn as_pin_mut(self: Pin<&mut Self>) -> ...` | Pin 可变投影 |
| `flip` | `fn flip(self) -> Either<R, L>` | 交换左右 |

#### 映射方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `map_left` | `fn map_left<F, M>(self, f: F) -> Either<M, R>` | 映射 Left |
| `map_right` | `fn map_right<F, S>(self, f: F) -> Either<L, S>` | 映射 Right |
| `either` | `fn either<F, G, T>(self, f: F, g: G) -> T` | 分别映射两侧 |
| `either_with` | `fn either_with<Ctx, F, G, T>(self, ctx, f, g) -> T` | 带上下文的映射 |

#### 链式方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `left_and_then` | `fn left_and_then<F, S>(self, f: F) -> Either<S, R>` | 链式 Left 操作 |
| `right_and_then` | `fn right_and_then<F, S>(self, f: F) -> Either<L, S>` | 链式 Right 操作 |

#### 取值方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `left_or` | `fn left_or(self, other: L) -> L` | 获取 Left 或默认值 |
| `left_or_default` | `fn left_or_default(self) -> L` | 获取 Left 或 Default |
| `left_or_else` | `fn left_or_else<F>(self, f: F) -> L` | 获取 Left 或计算 |
| `right_or` | `fn right_or(self, other: R) -> R` | 获取 Right 或默认值 |
| `right_or_default` | `fn right_or_default(self) -> R` | 获取 Right 或 Default |
| `right_or_else` | `fn right_or_else<F>(self, f: F) -> R` | 获取 Right 或计算 |
| `unwrap_left` | `fn unwrap_left(self) -> L` | 解包 Left (panic if Right) |
| `unwrap_right` | `fn unwrap_right(self) -> R` | 解包 Right (panic if Left) |
| `expect_left` | `fn expect_left(self, msg: &str) -> L` | 解包 Left 带消息 |
| `expect_right` | `fn expect_right(self, msg: &str) -> R` | 解包 Right 带消息 |

#### 特殊方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `into_iter` | `fn into_iter(self) -> Either<L::IntoIter, R::IntoIter>` | 转为迭代器 |
| `factor_none` | `fn factor_none(self) -> Option<Either<L, R>>` | 分解 Option |
| `factor_err` | `fn factor_err(self) -> Result<Either<L, R>, E>` | 分解 Result (Err) |
| `factor_ok` | `fn factor_ok(self) -> Result<T, Either<L, R>>` | 分解 Result (Ok) |
| `factor_first` | `fn factor_first(self) -> (T, Either<L, R>)` | 分解元组 (first) |
| `factor_second` | `fn factor_second(self) -> (Either<L, R>, T)` | 分解元组 (second) |
| `into_inner` | `fn into_inner(self) -> T` | 提取值 (L == R) |
| `map` | `fn map<F, M>(self, f: F) -> Either<M, M>` | 统一映射 (L == R) |
| `either_into` | `fn either_into<T>(self) -> T` | 统一转换 |

#### 宏

| 宏 | 说明 |
|----|------|
| `for_both!` | 对两侧执行相同操作 |
| `try_left!` | 尝试获取 Left，否则提前返回 |
| `try_right!` | 尝试获取 Right，否则提前返回 |

### 2.2 Trait 实现

#### 自动派生

- `Copy`
- `Clone`
- `PartialEq`
- `Eq`
- `PartialOrd`
- `Ord`
- `Hash`
- `Debug`

#### 标准 Trait 实现

| Trait | 条件 |
|-------|------|
| `Iterator` | `L: Iterator, R: Iterator<Item = L::Item>` |
| `DoubleEndedIterator` | `L: DoubleEndedIterator, R: DoubleEndedIterator<...` |
| `ExactSizeIterator` | `L: ExactSizeIterator, R: ExactSizeIterator<...` |
| `FusedIterator` | `L: FusedIterator, R: FusedIterator<...` |
| `Read` | `L: Read, R: Read` + use_std |
| `Write` | `L: Write, R: Write` + use_std |
| `Seek` | `L: Seek, R: Seek` + use_std |
| `BufRead` | `L: BufRead, R: BufRead` + use_std |
| `Future` | `L: Future, R: Future<Output = L::Output>` |
| `Error` | `L: Error, R: Error` + use_std |
| `Display` | `L: Display, R: Display` |
| `AsRef<T>` | `L: AsRef<T>, R: AsRef<T>` |
| `AsMut<T>` | `L: AsMut<T>, R: AsMut<T>` |
| `Deref` | `L: Deref, R: Deref<Target = L::Target>` |
| `DerefMut` | `L: DerefMut, R: DerefMut<Target = L::Target>` |
| `Extend<A>` | `L: Extend<A>, R: Extend<A>` |

#### 转换 Trait

| Trait | 转换 |
|-------|------|
| `From<Result<R, L>>` | `Result<R, L>` -> `Either<L, R>` (Err -> Left, Ok -> Right) |
| `Into<Result<R, L>>` | `Either<L, R>` -> `Result<R, L>` (Left -> Err, Right -> Ok) |

---

## 3. 与上游的差异

### 3.1 功能差异

| 特性 | 上游 | OpenHarmony | 差异说明 |
|------|------|-------------|----------|
| `use_std` | 默认启用 | 启用 | 一致 |
| `serde` | 可选 | 未启用 | 减少依赖 |

**说明**: 仅特性配置差异，API 完全一致。

### 3.2 代码差异

```diff
# 无差异 - 源码 100% 相同
```

---

## 4. 使用示例

### 4.1 基础用法

```rust
use either::{Either, Left, Right};

// 创建
let left: Either<i32, String> = Left(42);
let right: Either<i32, String> = Right("hello".to_string());

// 查询
assert!(left.is_left());
assert!(right.is_right());

// 取值
assert_eq!(left.left(), Some(42));
assert_eq!(right.right(), Some("hello".to_string()));
```

### 4.2 统一迭代器

```rust
use either::Either;

fn get_iter(condition: bool) -> Either<std::ops::Range<i32>, Vec<i32>::IntoIter> {
    if condition {
        Either::Left(0..10)
    } else {
        Either::Right(vec![1, 2, 3].into_iter())
    }
}

// 统一处理
for item in get_iter(true) {
    println!("{}", item);
}
```

### 4.3 使用宏

```rust
use either::{Either, for_both, try_left};

// for_both!
fn length(e: Either<String, &str>) -> usize {
    for_both!(e, s => s.len())
}

// try_left!
fn double(e: Either<i32, String>) -> Either<i32, String> {
    let value = try_left!(e);
    Left(value * 2)
}
```

---

## 5. 版本兼容性

### 5.1 API 稳定性

- **当前版本**: 1.8.1
- **API 稳定性**: 1.x 版本保证向后兼容
- **MSRV**: Rust 1.36+

### 5.2 升级兼容性

| 升级路径 | 兼容性 | 说明 |
|----------|--------|------|
| 1.8.1 -> 1.x | ✅ 兼容 | 1.x 版本 API 稳定 |
| 1.x -> 2.0 | ⚠️ 待验证 | 需查看变更日志 |

---

## 6. 总结

| 项目 | 状态 |
|------|------|
| API 修改 | 无 |
| API 扩展 | 无 |
| API 废弃 | 无 |
| 行为变更 | 无 |
| 特性差异 | 仅 serde 未启用 |
| 代码差异 | 无 |

**结论**: either 在 OpenHarmony 中保持**零修改**，API 与上游 1.8.1 完全一致。唯一的差异是未启用可选的 `serde` 特性以减少依赖。
