# termcolor - OpenHarmony 第三方库文档

## 库概述

**termcolor** 是 Rust 生态中一个轻量级的跨平台终端颜色输出库。它通过 ANSI 转义序列（Unix/Linux 系统）或 Windows Console API（Windows 系统）为终端应用程序提供彩色文本输出能力。

在 OpenHarmony 中，termcolor 作为多个 Rust 命令行工具和日志系统的底层依赖，为 `clap`（命令行参数解析器）、`env_logger`（日志格式化器）等库提供颜色支持。

## OpenHarmony 适配状态

| 状态 | 说明 |
|------|------|
| ✅ **已适配** | 完全适配，零 Patch |
| 📦 **版本** | 1.2.0（上游） / 6.1（OH bundle） |
| 🔧 **维护者** | fangting12@huawei.com |
| 📅 **评估日期** | 2026-02-08 |

## 文档导航

### 快速入口

- [阅读摘要](SUMMARY.md) - 快速了解文档结构和阅读建议
- [01_Overview.md](01_Overview.md) - 原始库功能介绍
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景和依赖关系

### 详细信息

- [02_Patches.md](02_Patches.md) - Patch 分析（本库无 Patch）
- [03_Build_Integration.md](03_Build_Integration.md) - 构建系统适配
- [05_API_Differences.md](05_API_Differences.md) - API 差异（本库无差异）
- [06_Security.md](06_Security.md) - 安全风险分析

## 快速开始

### 在 OH Rust 项目中使用

```toml
# Cargo.toml
[dependencies]
termcolor = "1.2.0"  # OH 版本与上游一致
```

```rust
use termcolor::{Color, ColorChoice, ColorSpec, StandardStream, WriteColor};

fn example() -> std::io::Result<()> {
    let mut stdout = StandardStream::stdout(ColorChoice::Auto);
    stdout.set_color(ColorSpec::new().set_fg(Some(Color::Green)))?;
    println!("绿色文本");
    Ok(())
}
```

### 构建命令

```bash
# 构建 termcolor 库
hb build //third_party/rust/crates/termcolor:lib

# 查看构建产物
ls out/.../libs/libtermcolor.rlib
```

## 关键信息速查

| 项目 | 值 |
|------|-----|
| **上游仓库** | https://github.com/BurntSushi/termcolor |
| **许可证** | MIT / Unlicense |
| **OH 路径** | //third_party/rust/crates/termcolor |
| **构建目标** | lib.rlib |
| **直接依赖者** | clap, env_logger, codespan-reporting 等 6 个 |
| **上游兼容性** | 完全兼容，无 OH 特定代码 |

## 常见问题

**Q: termcolor 在 OpenHarmony 上需要特殊配置吗？**

A: 不需要。termcolor 是一个纯粹的用户态库，不涉及任何系统级调用。OpenHarmony 的标准终端输出行为与 Linux 完全一致，因此 termcolor 无需任何修改即可正常工作。

**Q: 为什么没有 Patch？其他库通常都有一些适配？**

A: termcolor 的设计非常简洁且跨平台友好。它的核心功能是格式化 ANSI 颜色转义序列并写入标准输出，这在所有 POSIX 兼容系统上行为一致。OpenHarmony 作为 Linux 内核衍生系统，完全兼容这一行为。

**Q: 如何升级 termcolor 版本？**

A: 直接更新 Cargo.toml 中的版本号即可。由于没有 OH 特定修改，升级过程与上游完全一致。建议定期跟随上游版本更新以获取最新功能和修复。

## 相关资源

- [上游文档](https://docs.rs/termcolor)
- [上游 GitHub](https://github.com/BurntSushi/termcolor)
- [OH Rust 工具链文档]()
- [OH 第三方库集成指南]()
