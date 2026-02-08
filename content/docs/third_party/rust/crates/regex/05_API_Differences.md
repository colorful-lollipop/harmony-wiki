# API/接口差异

## 概述

**结论：regex 库在 OpenHarmony 中的 API 与上游版本完全一致，没有添加、修改或禁用任何 API。**

由于 regex 库在 OH 中没有使用任何 Patch，API 层面不存在差异。

## 无 API 差异的原因

### 1. 无代码修改

regex 库直接使用上游源代码，没有进行任何代码级别的修改：

| 对比项 | 说明 |
|--------|------|
| **源代码** | 与上游 1.7.1 版本完全一致 |
| **Cargo.toml** | 与上游配置一致 |
| **Features** | 上游的超集（启用更多优化选项） |

### 2. 标准 Cargo 集成

通过 `ohos_cargo_crate` 模板构建，不改变 crate 的导出接口：

```gn
ohos_cargo_crate("lib") {
    crate_name = "regex"
    crate_root = "src/lib.rs"
    # 标准配置，不添加额外导出
}
```

## 上游 API 清单

### 核心类型

| 类型 | 说明 |
|------|------|
| **Regex** | 主要正则表达式类型，用于字符串匹配 |
| **RegexSet** | 多正则表达式同时匹配 |
| **Match** | 单个匹配结果 |
| **Matches** | 匹配迭代器 |
| **Captures** | 捕获组结果 |
| **CaptureMatches** | 捕获组迭代器 |

### 字节匹配 API

| 类型 | 说明 |
|------|------|
| **bytes::Regex** | 用于 `&[u8]` 字节数组匹配 |

### 子 crate API

| crate | 类型/函数 | 说明 |
|-------|----------|------|
| **regex-syntax** | `Parser` | 正则表达式解析器 |
| **regex-syntax** | `Ast` | 抽象语法树 |
| **regex-syntax** | `hir::Hir` | 高级中间表示 |

### 主要方法（Regex）

```rust
impl Regex {
    // 工厂方法
    pub fn new(pattern: &str) -> Result<Regex, Error>
    pub fn is_match(&self, text: &str) -> bool
    pub fn find(&self, text: &str) -> Option<Match>
    pub fn find_iter(&self, text: &str) -> Matches
    pub fn capture_positions(&self, text: &str) -> Option<Vec<usize>>
    pub fn captures(&self, text: &str) -> Option<Captures>
    pub fn captures_iter(&self, text: &str) -> CaptureMatches
    pub fn split(&self, text: &str) -> Split
    pub fn splitn(&self, text: &str, n: usize) -> SplitN
    pub fn replace(&self, text: &str, rep: &str) -> String
    pub fn replace_all(&self, text: &str, rep: &str) -> String
}
```

## 完整 Feature 列表

### 默认启用的 Features

| Feature | 功能 |
|---------|------|
| **std** | 标准库支持 |
| **perf** | 综合性能优化 |
| **perf-cache** | 匹配缓存 |
| **perf-dfa** | DFA 引擎 |
| **perf-inline** | 函数内联 |
| **perf-literal** | 字面量加速 |
| **unicode** | Unicode 支持 |
| **unicode-age** | Unicode 字符年龄 |
| **unicode-bool** | Unicode 布尔属性 |
| **unicode-case** | Unicode 大小写 |
| **unicode-gencat** | Unicode 通用类别 |
| **unicode-perl** | Perl 兼容 Unicode |
| **unicode-script** | Unicode 脚本 |
| **unicode-segment** | Unicode 断字 |

### 可选 Features（未启用）

| Feature | 功能 | 未启用原因 |
|---------|------|-----------|
| **simd** | SIMD 优化（nightly） | 需要 nightly Rust |
| **pattern-literal** | 字面量模式优化 | 已包含在 perf-literal |

## 使用示例

### 基本用法

```rust
use regex::Regex;

fn main() {
    // 创建正则表达式
    let re = Regex::new(r"\d{4}-\d{2}-\d{2}").unwrap();
    
    // 检查匹配
    assert!(re.is_match("2024-01-15"));
    
    // 查找匹配
    let mat = re.find("2024-01-15").unwrap();
    assert_eq!(mat.as_str(), "2024-01-15");
    
    // 迭代匹配
    for mat in re.find_iter("2024-01-15 and 2025-02-20") {
        println!("Found: {}", mat.as_str());
    }
}
```

### 捕获组

```rust
use regex::Regex;

fn parse_date() {
    let re = Regex::new(r"(\d{4})-(\d{2})-(\d{2})").unwrap();
    let caps = re.captures("2024-01-15").unwrap();
    
    println!("Year: {}", caps.get(1).unwrap().as_str());
    println!("Month: {}", caps.get(2).unwrap().as_str());
    println!("Day: {}", caps.get(3).unwrap().as_str());
}
```

### 字节匹配

```rust
use regex::bytes::Regex;

fn match_bytes() {
    let re = Regex::new(r"\x00[^\x00]+").unwrap();
    let data = b"\x00hello\x00world\x00";
    
    for mat in re.find_iter(data) {
        println!("Found: {:?}", mat.as_bytes());
    }
}
```

### 多模式匹配

```rust
use regex::RegexSet;

fn multi_pattern() {
    let set = RegexSet::new(&[
        r"\d+",
        r"[a-z]+",
        r"[A-Z]+",
    ]).unwrap();
    
    let matches = set.matches("abc123");
    println!("Matched patterns: {:?}", matches.matched());
}
```

## 结论

regex 库在 OpenHarmony 中的 API 与上游完全一致，使用者可以直接参考上游文档：

- [上游文档](https://docs.rs/regex)
- [上游 README](https://github.com/rust-lang/regex#readme)

如有 OH 特定的功能需求，建议：
1. 先检查上游是否已有类似功能
2. 如需添加 OH 特有功能，考虑提交给上游或创建 Patch
