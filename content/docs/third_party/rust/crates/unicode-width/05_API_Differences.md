# 05 - API 差异分析

## 核心结论

**unicode-width 在 OpenHarmony 中的 API 与上游完全一致，无任何差异。**

## 差异总览

| 差异类型 | 状态 | 说明 |
|---------|------|------|
| 新增 API | ❌ 无 | 未添加任何 OH 特有接口 |
| 删除 API | ❌ 无 | 未删除任何上游接口 |
| 修改行为 | ❌ 无 | 行为与上游完全一致 |
| 废弃 API | ❌ 无 | 未废弃任何接口 |
| Feature 差异 | ❌ 无 | features 与上游默认一致 |

## 与上游 API 对比

### 核心 Trait

| Trait | OH 状态 | 上游状态 | 差异 |
|-------|---------|----------|------|
| `UnicodeWidthChar` | ✅ 完整 | ✅ 完整 | 无 |
| `UnicodeWidthStr` | ✅ 完整 | ✅ 完整 | 无 |

### 方法对比

#### UnicodeWidthChar

| 方法 | OH 可用性 | 上游可用性 | 说明 |
|------|----------|-----------|------|
| `width()` | ✅ | ✅ | 计算字符宽度 |
| `width_cjk()` | ✅ | ✅ | CJK 上下文宽度（需 cjk feature） |

#### UnicodeWidthStr

| 方法 | OH 可用性 | 上游可用性 | 说明 |
|------|----------|-----------|------|
| `width()` | ✅ | ✅ | 计算字符串宽度 |
| `width_cjk()` | ✅ | ✅ | CJK 上下文宽度（需 cjk feature） |

### 常量

| 常量 | OH 可用性 | 上游可用性 |
|------|----------|-----------|
| `UNICODE_VERSION` | ✅ | ✅ |

## Feature 配置

### Cargo.toml features

```toml
[features]
cjk = []
default = ["cjk"]
rustc-dep-of-std = ['std', 'core', 'compiler_builtins']
no_std = []  # Legacy, no-op
```

### OH 构建配置

当前 BUILD.gn 未显式覆盖 features，因此：
- 使用默认 features：`["cjk"]`
- CJK 支持已启用

### Feature 差异说明

| Feature | OH 状态 | 上游默认 | 差异 |
|---------|---------|----------|------|
| `cjk` | ✅ 启用 | ✅ 启用 | 无差异 |
| `rustc-dep-of-std` | ❌ 未启用 | ❌ 可选 | 无差异 |
| `no_std` | ✅ 隐式 | ✅ 隐式 | 库本身即 no_std |

## 源码一致性验证

### lib.rs 对比

OH 版本与上游 v0.1.14 的 `src/lib.rs` 文件：

| 属性 | OH 版本 | 上游版本 | 结果 |
|------|---------|----------|------|
| 文件大小 | 相同 | 相同 | ✅ |
| MD5 哈希 | 相同 | 相同 | ✅ |
| 行数 | 259 | 259 | ✅ |
| 内容 | 完全相同 | 完全相同 | ✅ |

### 关键代码段

```rust
// OH 版本 - 与上游完全一致
#![forbid(unsafe_code)]
#![deny(missing_docs)]
#![doc(
    html_logo_url = "https://unicode-rs.github.io/unicode-rs_sm.png",
    html_favicon_url = "https://unicode-rs.github.io/unicode-rs_sm.png"
)]
#![no_std]

pub use tables::UNICODE_VERSION;

mod tables;

// ... trait 定义完全一致
```

## 行为一致性

### 宽度计算规则

OH 版本与上游遵循完全相同的 Unicode 宽度计算规则：

| 字符类型 | 计算方式 | 与上游一致性 |
|---------|---------|-------------|
| ASCII 字符 | 宽度 1 | ✅ 一致 |
| CJK 字符 | 宽度 2 | ✅ 一致 |
| Emoji | 宽度 2 | ✅ 一致 |
| 零宽字符 | 宽度 0 | ✅ 一致 |
| 组合字符 | 宽度 0 | ✅ 一致 |

### 测试用例一致性

所有上游测试用例在 OH 构建中均可正常运行：

| 测试文件 | OH 状态 | 说明 |
|---------|--------|------|
| `tests/tests.rs` | ✅ 通过 | 全部测试通过 |
| `tests/emoji-test.txt` | ✅ 通过 | Emoji 测试通过 |

## 无差异的原因

### 1. 功能完整性

unicode-width 的功能已经非常完整：
- 覆盖 Unicode TR11 规范
- 支持 Emoji、CJK、组合字符等复杂场景
- 无需 OH 特定扩展

### 2. 设计简洁

```rust
// 核心接口极其简洁
pub trait UnicodeWidthStr {
    fn width(&self) -> usize;
    fn width_cjk(&self) -> usize;  // feature = "cjk"
}
```

- 仅两个核心方法
- 无需扩展新功能

### 3. 通用性设计

库本身就是通用的 Unicode 工具：
- 不依赖特定平台
- 不依赖特定应用
- 纯算法实现

## 对开发者的影响

### 使用上游文档

由于 API 完全一致，开发者可以直接参考：
- 上游文档：https://docs.rs/unicode-width
- 上游示例：https://github.com/unicode-rs/unicode-width

### 代码可移植性

基于 unicode-width 编写的代码：
- 在 OH 和上游环境中行为一致
- 无需条件编译或特殊处理
- 可轻松迁移

### 示例代码

```rust
// 这段代码在 OH 和上游环境中完全一致
use unicode_width::UnicodeWidthStr;

fn align_text(text: &str, width: usize) -> String {
    let text_width = text.width();
    if text_width >= width {
        text.to_string()
    } else {
        let padding = width - text_width;
        format!("{}{}", text, " ".repeat(padding))
    }
}

fn main() {
    let s = "Hello, 世界!";
    assert_eq!(s.width(), 12);
    println!("{}", align_text(s, 20));
}
```

## 版本兼容性

### 语义化版本

unicode-width 遵循 SemVer 规范：
- 当前版本：v0.1.14
- API 向后兼容：✅

### 升级影响

升级到新版本时：
- API 保持不变
- 行为保持一致
- 仅需更新版本号

## 总结

| 项目 | 结论 |
|------|------|
| **API 差异** | **无** |
| **行为差异** | **无** |
| **Feature 差异** | **无** |
| **源码差异** | **无** |
| **兼容性** | **完全兼容** |

unicode-width 在 OpenHarmony 中的使用与上游完全一致，开发者可以放心使用上游文档和示例，无需担心平台差异。
