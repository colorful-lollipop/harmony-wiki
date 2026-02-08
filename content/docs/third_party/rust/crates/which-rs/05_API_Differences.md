# API 差异

## 概述

which-rs 在 OpenHarmony 中的 API 与上游保持一致，**无 OH 特定的 API 修改**。

但由于 BUILD.gn 中未启用 `regex` feature，部分功能在 OH 中不可用。

## 可用 API

### 核心函数（OH 可用）

```rust
// 在 PATH 中查找可执行文件
pub fn which<T: AsRef<OsStr>>(binary_name: T) -> Result<path::PathBuf>

// 查找所有匹配的可执行文件
pub fn which_all<T: AsRef<OsStr>>(binary_name: T) -> Result<impl Iterator<Item = path::PathBuf>>

// 在指定路径中查找（忽略当前目录）
pub fn which_global<T: AsRef<OsStr>>(binary_name: T) -> Result<path::PathBuf>

// 在指定路径中查找所有匹配
pub fn which_all_global<T: AsRef<OsStr>>(binary_name: T) -> Result<impl Iterator<Item = path::PathBuf>>

// 在自定义路径列表中查找
pub fn which_in<T, U, V>(binary_name: T, paths: Option<U>, cwd: V) -> Result<path::PathBuf>

// 在自定义路径列表中查找所有匹配
pub fn which_in_all<T, U, V>(binary_name: T, paths: Option<U>, cwd: V) -> Result<impl Iterator<Item = path::PathBuf>>

// 在自定义路径列表中查找（忽略当前目录）
pub fn which_in_global<T, U>(binary_name: T, paths: Option<U>) -> Result<impl Iterator<Item = path::PathBuf>>
```

### 类型包装器（OH 可用）

```rust
// 表示已验证的可执行文件路径
pub struct Path { ... }

impl Path {
    pub fn new<T: AsRef<OsStr>>(binary_name: T) -> Result<Path>
    pub fn all<T: AsRef<OsStr>>(binary_name: T) -> Result<impl Iterator<Item = Path>>
    pub fn new_in<T, U, V>(binary_name: T, paths: Option<U>, cwd: V) -> Result<Path>
    pub fn all_in<T, U, V>(binary_name: T, paths: Option<U>, cwd: V) -> Result<impl Iterator<Item = Path>>
    pub fn as_path(&self) -> &path::Path
    pub fn into_path_buf(self) -> path::PathBuf
}

// 表示已验证且规范化的可执行文件路径
pub struct CanonicalPath { ... }

impl CanonicalPath {
    pub fn new<T: AsRef<OsStr>>(binary_name: T) -> Result<CanonicalPath>
    pub fn all<T: AsRef<OsStr>>(binary_name: T) -> Result<impl Iterator<Item = Result<CanonicalPath>>>
    pub fn new_in<T, U, V>(binary_name: T, paths: Option<U>, cwd: V) -> Result<CanonicalPath>
    pub fn all_in<T, U, V>(binary_name: T, paths: Option<U>, cwd: V) -> Result<impl Iterator<Item = Result<CanonicalPath>>>
    pub fn as_path(&self) -> &path::Path
    pub fn into_path_buf(self) -> path::PathBuf
}
```

### 配置构建器（OH 可用）

```rust
pub struct WhichConfig { ... }

impl WhichConfig {
    pub fn new() -> Self
    pub fn system_cwd(self, use_cwd: bool) -> Self
    pub fn custom_cwd(self, cwd: path::PathBuf) -> Self
    pub fn binary_name(self, name: OsString) -> Self
    pub fn custom_path_list(self, custom_path_list: OsString) -> Self
    pub fn system_path_list(self) -> Self
    pub fn first_result(self) -> Result<path::PathBuf>
    pub fn all_results(self) -> Result<impl Iterator<Item = path::PathBuf>>
}
```

### 错误类型（OH 可用）

```rust
pub enum Error {
    BadAbsolutePath,
    BadRelativePath,
    CannotFindBinaryPath,
    CannotGetCurrentDir,
    CannotCanonicalize,
}
```

## 不可用 API（OH 未启用 regex feature）

以下 API 需要启用 `regex` feature，在 OH 中不可用：

```rust
// 使用正则表达式在 PATH 中查找
#[cfg(feature = "regex")]
pub fn which_re(regex: impl Borrow<Regex>) -> Result<impl Iterator<Item = path::PathBuf>>

// 使用正则表达式在指定路径中查找
#[cfg(feature = "regex")]
pub fn which_re_in<T>(regex: impl Borrow<Regex>, paths: Option<T>) -> Result<impl Iterator<Item = path::PathBuf>>

// WhichConfig 的正则配置方法
impl WhichConfig {
    #[cfg(feature = "regex")]
    pub fn regex(self, regex: Regex) -> Self
}
```

## 上游与 OH 的 API 对比表

| API | 上游 (全功能) | OH (当前配置) | 差异说明 |
|-----|--------------|---------------|----------|
| `which()` | ✅ 可用 | ✅ 可用 | 一致 |
| `which_all()` | ✅ 可用 | ✅ 可用 | 一致 |
| `which_global()` | ✅ 可用 | ✅ 可用 | 一致 |
| `which_in()` | ✅ 可用 | ✅ 可用 | 一致 |
| `which_re()` | ✅ 可用 | ❌ 不可用 | 未启用 regex feature |
| `which_re_in()` | ✅ 可用 | ❌ 不可用 | 未启用 regex feature |
| `Path` | ✅ 可用 | ✅ 可用 | 一致 |
| `CanonicalPath` | ✅ 可用 | ✅ 可用 | 一致 |
| `WhichConfig::regex()` | ✅ 可用 | ❌ 不可用 | 未启用 regex feature |
| `WhichConfig::binary_name()` | ✅ 可用 | ✅ 可用 | 一致 |
| `WhichConfig::custom_path_list()` | ✅ 可用 | ✅ 可用 | 一致 |

## 使用示例

### 基础查找（OH 可用）

```rust
use which::which;
use std::path::PathBuf;

fn main() -> Result<(), which::Error> {
    // 查找 rustc
    let path = which("rustc")?;
    println!("rustc: {:?}", path);
    
    // 查找 python
    if let Ok(path) = which("python3") {
        println!("python3: {:?}", path);
    }
    
    Ok(())
}
```

### 在自定义路径中查找（OH 可用）

```rust
use which::which_in;
use std::path::Path;

fn find_in_custom_path(binary: &str, custom_paths: &str) -> Option<PathBuf> {
    which_in(binary, Some(custom_paths), ".").ok()
}

// 使用示例
let path = find_in_custom_path("my_tool", "/opt/tools:/usr/local/bin");
```

### 使用 WhichConfig（OH 可用）

```rust
use which::WhichConfig;
use std::ffi::OsString;

fn find_with_config() -> Result<path::PathBuf, which::Error> {
    WhichConfig::new()
        .binary_name(OsString::from("clang"))
        .system_path_list()
        .system_cwd(false)  // 不使用当前目录
        .first_result()
}
```

### 正则查找（OH 不可用，但可模拟）

**上游方式**（OH 不支持）：
```rust
use which::which_re;
use regex::Regex;

// 查找所有 python 版本
let re = Regex::new(r"python\d(\.\d)?$").unwrap();
let pythons: Vec<_> = which_re(re).unwrap().collect();
```

**OH 替代方案**（手动过滤）：
```rust
use which::which_all;
use std::path::Path;

// 手动实现正则查找
fn which_re_like(pattern: &str) -> Vec<path::PathBuf> {
    which_all("python")  // 先找出所有 python
        .unwrap()
        .filter(|p| {
            if let Some(name) = p.file_name().and_then(|n| n.to_str()) {
                name.starts_with("python") && 
                name.chars().skip(6).next().map(|c| c.is_ascii_digit()).unwrap_or(false)
            } else {
                false
            }
        })
        .collect()
}
```

## 行为差异

### PATH 处理

| 场景 | 上游 | OH | 说明 |
|------|------|-----|------|
| 空 PATH | 返回 CannotFindBinaryPath | 相同 | 一致 |
| 相对路径 | 相对当前目录解析 | 相同 | 一致 |
| 绝对路径 | 直接检查存在性和可执行性 | 相同 | 一致 |

### 可执行性检查

| 平台 | 上游 | OH | 说明 |
|------|------|-----|------|
| Linux | 使用 libc::access(X_OK) | 相同 | 一致 |
| Windows | 检查文件扩展名 | N/A | OH 不支持 Windows |

## 迁移指南

### 从上游代码迁移到 OH

**无需修改**，所有基础 API 完全兼容。

### 如需 regex 功能

如果在 OH 中需要正则查找功能，有以下选择：

**选项 1: 手动过滤**（推荐，无额外依赖）
```rust
// 手动实现模式匹配
let matches: Vec<_> = which_all("python")
    .unwrap()
    .filter(|p| matches_pattern(p, regex))
    .collect();
```

**选项 2: 启用 regex feature**（需修改 BUILD.gn）
需要：
1. 在 BUILD.gn 中添加 regex 依赖
2. 启用 regex feature
3. 确保 regex 及其依赖 crates 已导入 OH

**选项 3: 使用 glob 模式**（如已引入 glob crate）
```rust
use glob::glob;

let matches: Vec<_> = glob("/usr/bin/python*")
    .unwrap()
    .filter_map(Result::ok)
    .filter(|p| p.is_file())
    .collect();
```

## 总结

| 类别 | OH 支持情况 |
|------|------------|
| 核心查找 API | ✅ 完整支持 |
| 类型包装器 | ✅ 完整支持 |
| 配置构建器 | ✅ 完整支持（不含 regex） |
| 正则查找 | ❌ 不支持（feature 未启用） |
| Windows 支持 | N/A（OH 基于 Linux） |

**建议**: 对于绝大多数使用场景，OH 当前提供的 API 已足够。如需正则查找，建议通过手动过滤或使用其他模式匹配库实现。
