# strsim-rs 原始库简介

## 1.1 库概述

**strsim-rs**（简称 strsim）是一个纯 Rust 实现的字符串相似度度量库，提供多种经典的字符串距离和相似度算法。该库完全使用 safe Rust 编写，不依赖任何外部 crate，仅使用 Rust 标准库。

| 项目 | 值 |
|------|-----|
| **库名称** | strsim-rs / strsim |
| **当前版本** | 0.10.0 |
| **首次发布** | 2016 年 |
| **许可证** | Apache 2.0, MIT (双许可证) |
| **上游地址** | https://github.com/dguo/strsim-rs |
| **crates.io** | https://crates.io/crates/strsim |
| **文档** | https://docs.rs/strsim/ |

## 1.2 核心功能

strsim-rs 实现了以下字符串相似度算法：

### 1.2.1 Hamming 距离

计算两个**等长字符串**中对应位置字符不同的数量。

```rust
use strsim::hamming;

let distance = hamming("hamming", "hammers").unwrap();  // 返回 3
let distance = hamming("abc", "abc").unwrap();          // 返回 0
```

**特点**：
- 仅适用于等长字符串
- 返回 `Result<usize, StrSimError>`，长度不匹配时返回错误
- 时间复杂度：O(n)

### 1.2.2 Levenshtein 距离

计算将一个字符串转换为另一个字符串所需的**最少编辑次数**，允许以下操作：
- 插入一个字符
- 删除一个字符
- 替换一个字符

```rust
use strsim::levenshtein;

let distance = levenshtein("kitten", "sitting");  // 返回 3
// kitten → sitten (替换 k→s)
// sitten → sittin (替换 e→i)
// sittin → sitting (插入 g)
```

**归一化版本**：

```rust
use strsim::normalized_levenshtein;

let similarity = normalized_levenshtein("kitten", "sitting");  // 返回 ~0.57
```

### 1.2.3 OSA (Optimal String Alignment) 距离

Levenshtein 距离的变体，**允许相邻字符换位**，但每个子串只能编辑一次。

```rust
use strsim::osa_distance;

let distance = osa_distance("ab", "ba");  // 返回 1 (换位)
```

**与 Levenshtein 的区别**：
- OSA 不允许字符串中的同一个字符被编辑多次
- OSA 的三角形不等式不成立

### 1.2.4 Damerau-Levenshtein 距离

真正的换位算法，允许**相邻或不相邻的字符换位**，且字符串可以被编辑多次。

```rust
use strsim::damerau_levenshtein;

let distance = damerau_levenshtein("ab", "ba");  // 返回 1
let distance = damerau_levenshtein("CA", "ABC");  // 返回 3
```

### 1.2.5 Jaro 相似度

用于衡量两个字符串的相似程度，取值范围 0.0 到 1.0。

```rust
use strsim::jaro;

let similarity = jaro("Friedrich Nietzsche", "Jean-Paul Sartre");  // 返回 ~0.39
```

**特点**：
- 考虑字符换位
- 不对共同前缀给予额外权重

### 1.2.6 Jaro-Winkler 相似度

Jaro 相似度的变体，**对共同前缀给予更高的权重**，这使得该算法在拼写检查场景中表现更好。

```rust
use strsim::jaro_winkler;

let similarity = jaro_winkler("cheeseburger", "cheese fries");  // 返回 ~0.91
let similarity = jaro_winkler("ABC", "AB");                     // 返回 ~0.97 (前缀权重)
```

**与 Jaro 的区别**：
- 共同前缀越长，相似度越高
- 最大前缀长度无限制
- 更适合拼写纠正场景

### 1.2.7 Sørensen-Dice 相似度

基于字符 bigram（连续字符对）的相似度系数。

```rust
use strsim::sorensen_dice;

let similarity = sorensen_dice("web applications", "applications of the web");
// 返回 ~0.79
```

**特点**：
- 对词序不敏感
- 适用于文本相似度比较
- 常用于自然语言处理

## 1.3 泛型支持

strsim-rs 提供泛型版本的函数，可用于非字符串类型的序列。

```rust
use strsim::generic_levenshtein;

// 用于数字序列
let distance = generic_levenshtein(&[1, 2, 3], &[1, 2, 4]);  // 返回 1

// 用于自定义类型（需实现 PartialEq trait）
```

## 1.4 API 速查

### 距离函数（返回整数）

| 函数 | 用途 | 时间复杂度 |
|------|------|------------|
| `hamming(a, b)` | Hamming 距离 | O(n) |
| `levenshtein(a, b)` | Levenshtein 距离 | O(n×m) |
| `osa_distance(a, b)` | OSA 距离 | O(n×m) |
| `damerau_levenshtein(a, b)` | Damerau-Levenshtein 距离 | O(n×m) |

### 相似度函数（返回浮点数 0.0-1.0）

| 函数 | 用途 |
|------|------|
| `normalized_levenshtein(a, b)` | 归一化 Levenshtein |
| `normalized_damerau_levenshtein(a, b)` | 归一化 Damerau-Levenshtein |
| `jaro(a, b)` | Jaro 相似度 |
| `jaro_winkler(a, b)` | Jaro-Winkler 相似度 |
| `sorensen_dice(a, b)` | Sørensen-Dice 相似度 |

### 泛型函数

| 函数 | 用途 |
|------|------|
| `generic_hamming(a, b)` | 泛型 Hamming 距离 |
| `generic_levenshtein(a, b)` | 泛型 Levenshtein 距离 |
| `generic_damerau_levenshtein(a_elems, b_elems)` | 泛型 Damerau-Levenshtein 距离 |

## 1.5 错误处理

```rust
use strsim::{hamming, StrSimError};

// Hamming 距离需要等长字符串
match hamming("abc", "ab") {
    Ok(distance) => println!("Distance: {}", distance),
    Err(StrSimError::DifferentLengthArgs) => {
        println!("错误：字符串长度不同");
    }
}
```

## 1.6 在 OpenHarmony 中的定位

strsim-rs 在 OpenHarmony 生态中定位为：

1. **基础算法库**：提供字符串相似度计算的核心算法
2. **间接依赖**：通过 clap 被命令行工具使用
3. **透明组件**：用户通常不需要直接使用，而是通过高级 API 间接调用

**典型使用路径**：
```
用户 → 命令行工具 → clap → strsim
```

---

*文档参考：上游 README.md 和 src/lib.rs*
