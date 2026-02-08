# nom 库 OpenHarmony Wiki

> nom - A Rust parser combinators library for byte stream parsing

## 概述

本 Wiki 记录了 **nom 7.1.3** 版本在 OpenHarmony third_party 生态系统中的集成与适配信息。

nom 是 Rust 生态系统中最受欢迎的解析器组合子库之一，为开发者提供了一套强大且类型安全的工具，用于构建各种格式的解析器。在 OpenHarmony 中，nom 主要用于支持 Rust  crates 的解析需求，如 rust-cexpr 等。

## 关键发现

| 指标 | 结果 |
|------|------|
| **OH Patch 数量** | 0 (无任何代码修改) |
| **集成复杂度** | ⭐☆☆☆☆ (极低) |
| **构建适配** | 标准适配 |
| **依赖者数量** | 1 (rust-cexpr) |
| **上游同步状态** | ✅ 完全同步 |

## 为什么 nom 不需要 Patch

nom 作为一个**纯算法库**，具有以下特性使其天然兼容 OpenHarmony：

1. **无平台依赖**：不涉及 I/O、网络、文件系统等系统调用
2. **no_std 兼容**：设计之初就考虑了嵌入式和资源受限环境
3. **纯 Rust 实现**：所有依赖 (memchr, minimal-lexical) 也是纯 Rust 库
4. **API 稳定**：nom 7.x 系列 API 成熟稳定

## 文档导航

### 必读文档

- **[README.md](./SUMMARY.md)** - 阅读路线建议
- **[01_Overview.md](./01_Overview.md)** - 库功能概述
- **[02_Patches.md](./02_Patches.md)** - Patch 分析 (核心文档)
- **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建系统适配
- **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 依赖关系与使用场景

### 工作文档

- **[ASSESSMENT.md](./_work/ASSESSMENT.md)** - 项目评估报告 (Phase 0)
- **[NOTES.md](./_work/NOTES.md)** - 分析过程记录

## 快速开始

### 在 OH Rust 项目中使用 nom

```rust
use nom::{
    IResult,
    bytes::complete::{tag, take_while_m_n},
    combinator::map_res,
    sequence::tuple,
};

fn parse_hex_color(input: &str) -> IResult<&str, (u8, u8, u8)> {
    let (input, _) = tag("#")(input)?;
    let (input, red) = take_while_m_n(2, 2, |c: char| c.is_digit(16))(input)?;
    let (input, green) = take_while_m_n(2, 2, |c: char| c.is_digit(16))(input)?;
    let (input, blue) = take_while_m_n(2, 2, |c: char| c.is_digit(16))(input)?;

    Ok((input, (
        u8::from_str_radix(red, 16).unwrap(),
        u8::from_str_radix(green, 16).unwrap(),
        u8::from_str_radix(blue, 16).unwrap(),
    )))
}
```

### 构建系统引用

```gn
# BUILD.gn
ohos_rust_lib("my_parser") {
    deps = [
        "//third_party/rust/crates/nom:lib",
    ]
}
```

## 版本信息

| 属性 | 值 |
|------|-----|
| **nom 版本** | 7.1.3 |
| **OH 组件版本** | 6.1 |
| **许可证** | MIT |
| **上游地址** | [rust-bakery/nom](https://github.com/rust-bakery/nom) |
| **OH 维护者** | fangting12@huawei.com |

## 相关资源

### 上游资源
- [nom 官方文档](https://docs.rs/nom)
- [nom GitHub](https://github.com/rust-bakery/nom)
- [nom Cookbooks](https://github.com/rust-bakery/nom/tree/main/doc)

### OpenHarmony 资源
- [nom bundle.json](../bundle.json)
- [nom BUILD.gn](../BUILD.gn)
- [rust-cexpr](../rust-cexpr) - nom 的直接依赖者

## 常见问题

### Q: nom 和正则表达式有什么区别？

nom 是**解析器组合子**，适用于结构化数据的解析，而正则表达式适用于简单模式匹配。nom 提供：
- 强类型保证
- 精确的错误位置报告
- 可组合的解析单元
- 支持复杂嵌套结构

### Q: 为什么 rust-cexpr 需要依赖 nom？

rust-cexpr 是一个 C 表达式解析器，需要解析 C 语言语法。nom 提供了构建解析器所需的组合子工具，使得 rust-cexpr 可以用声明式的方式定义 C 语法规则。

### Q: 如何在 OH 中升级 nom 版本？

由于 nom 没有 OH 特定 Patch，升级相对简单：
1. 更新 `Cargo.toml` 中的版本号
2. 同步更新 `BUILD.gn` 中的 `cargo_pkg_version`
3. 运行测试验证兼容性
4. 提交更新到 OH 仓库

## 贡献指南

如果发现文档错误或有改进建议，请：

1. 提交 Issue 到 OpenHarmony 第三方库仓库
2. 或直接提交 Pull Request 修改 `wiki/` 目录下的文档

---

**最后更新**: 2024年
**维护者**: OpenHarmony third_party 团队
