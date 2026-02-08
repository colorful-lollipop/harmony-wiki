# 01 - 原始库简介

## 库基本信息

| 属性 | 内容 |
|------|------|
| **名称** | unicode-width |
| **版本** | v0.1.14 |
| **上游地址** | https://github.com/unicode-rs/unicode-width |
| **文档** | https://docs.rs/unicode-width |
| **许可证** | Apache-2.0 OR MIT |
| **维护者** | kwantam, Manish Goregaokar |

## 原始功能

unicode-width 是一个 Rust 库，用于确定 Unicode 字符和字符串的**显示宽度**（displayed width），即字符在等宽终端中所占的列数。

### 核心功能

1. **字符宽度计算**
   - 计算单个 `char` 的显示宽度
   - 根据 Unicode Standard Annex #11 规则
   - 处理控制字符、零宽字符、全角字符等

2. **字符串宽度计算**
   - 计算 `str` 的总显示宽度
   - 处理 Emoji 序列（ZWJ 序列）
   - 处理组合字符和连字

3. **CJK 上下文支持**
   - 提供 `width_cjk()` 方法
   - 对 East Asian Ambiguous 字符按 2 列计算
   - 适用于中日韩等东亚语言环境

### 使用示例

```rust
use unicode_width::UnicodeWidthStr;

// 基本字符串宽度
let s = "Hello, 世界!";
assert_eq!(s.width(), 12);  // 英文 1 列，中文 2 列

// CJK 上下文
assert_eq!(s.width_cjk(), 12);  // 包含中文字符

// Emoji 宽度
assert_eq!("👩‍🔬".width(), 2);  // Emoji 显示为 2 列

// 全角字符
assert_eq!("Ｈｅｌｌｏ".width(), 10);  // 全角英文字母各 2 列
```

## 技术特点

### 1. no_std 设计
```rust
#![no_std]
```
- 无标准库依赖
- 适用于嵌入式和资源受限环境
- 仅需 `core` crate

### 2. 安全代码
```rust
#![forbid(unsafe_code)]
```
- 完全使用安全 Rust 编写
- 无 unsafe 代码块

### 3. 算法实现

库使用 Unicode 标准数据表来计算宽度：

| 字符类型 | 宽度 | 说明 |
|---------|------|------|
| East Asian Wide/Fullwidth | 2 | 全角字符、CJK 字符 |
| East Asian Ambiguous | 1/2 | 根据上下文（CJK feature） |
| Emoji Presentation | 2 | Emoji 字符 |
| Default Ignorable | 0 | 零宽字符、控制字符 |
| Grapheme Extend | 0 | 组合用字符 |
| 其他 | 1 | 默认宽度 |

### 4. Emoji 支持

库对 Emoji 的处理遵循 Unicode 标准：

```rust
// Emoji 基础字符
assert_eq!("👩".width(), 2);        // Woman: 2 列

// ZWJ 序列（零宽连接符序列）
assert_eq!("👩‍🔬".width(), 2);      // Woman scientist: 2 列

// Emoji 修饰符
assert_eq!("👨🏻".width(), 2);        // Man + light skin tone: 2 列

// 旗帜序列
assert_eq!("🏳️‍🌈".width(), 2);       // Rainbow flag: 2 列
```

## 在 OpenHarmony 中的作用

### 定位

在 OpenHarmony 中，unicode-width 是一个**基础工具库**，主要用于：

1. **诊断信息显示**：Rust 编译器和相关工具的错误消息格式化
2. **源代码对齐**：源代码位置标记（`^~~~~`）的精确定位
3. **帮助文本格式化**：命令行工具的帮助信息对齐

### 使用场景

由于 OpenHarmony 使用 Rust 作为系统开发语言之一，unicode-width 被以下场景依赖：

| 场景 | 说明 |
|------|------|
| 编译错误提示 | 精确定位错误位置，考虑中英文混排 |
| 代码格式化 | rustfmt 工具对齐代码 |
| 诊断报告 | codespan-reporting 美化诊断输出 |
| CLI 工具 | 命令行帮助信息格式化 |

### OH 特定考量

1. **中文字符支持**：OH 作为中国主导的系统，需要正确处理中文字符宽度
2. **嵌入式友好**：no_std 设计适合 OH 的轻量级设备
3. **稳定依赖**：被 Rust 编译器依赖，质量有保障

## 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| v0.1.14 | 2023 | 当前 OH 使用版本，支持 Unicode 15.0 |
| v0.1.13 | 2023 | 新增若干 Emoji 序列支持 |
| v0.1.12 | 2022 | 改进 CJK 处理 |
| v0.1.11 | 2022 | 更早版本 |

## 上游活跃度

- **维护状态**：活跃维护
- **更新频率**：每年 2-4 次，跟随 Unicode 版本更新
- **Issue 响应**：及时
- **PR 接受度**：中等（需要符合 Unicode 标准）

## 参考资源

1. **GitHub**: https://github.com/unicode-rs/unicode-width
2. **文档**: https://docs.rs/unicode-width
3. **Crates.io**: https://crates.io/crates/unicode-width
4. **Unicode TR11**: http://www.unicode.org/reports/tr11/
5. **Unicode Emoji**: https://unicode.org/reports/tr51/
