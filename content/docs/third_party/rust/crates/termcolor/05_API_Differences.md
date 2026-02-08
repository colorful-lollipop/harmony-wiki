# API/接口差异

## 差异概述

**结论**：termcolor 在 OpenHarmony 中**未修改任何 API**，所有 API 与上游完全一致。

| 维度 | 状态 |
|------|------|
| **API 完整性** | ✅ 完整，无删减 |
| **API 新增** | ❌ 无新增 |
| **API 修改** | ❌ 无修改 |
| **行为变更** | ❌ 无变更 |

## API 清单

### 公共 API（完整列表）

#### 模块级别

```rust
pub mod imp;
```

#### 类型定义

```rust
pub struct Ansi<W: Write> { /* ... */ }
pub struct Buffer { /* ... */ }
pub struct BufferWriter { /* ... */ }
pub struct ColorSpec { /* ... */ }
pub struct NoColor<W: Write> { /* ... */ }
pub struct StandardStream { /* ... */ }
pub struct StandardStreamLock<'a> { /* ... */ }
```

#### 枚举定义

```rust
pub enum Color {
    Black,
    Red,
    Green,
    Yellow,
    Blue,
    Magenta,
    Cyan,
    White,
    Ansi(u8),
    Rgb(u8, u8, u8),
}

pub enum ColorChoice {
    Always,
    Auto,
    Never,
}

pub enum Intensity {
    Normal,
    Bright,
}
```

#### Trait 定义

```rust
pub trait WriteColor: io::Write {
    fn set_color(&mut self, color: &ColorSpec) -> io::Result<()>;
    fn reset(&mut self) -> io::Result<()>;
    fn is_supported(&self) -> bool;
    fn supports_reset(&self) -> bool;
    fn has_color(&self) -> bool;
}
```

#### 函数

```rust
pub fn parse_color(color: &str) -> Result<Color, ParseColorError>;
```

## 使用示例

所有 API 在 OpenHarmony 上的使用方式与上游完全一致：

### 基础颜色输出

```rust
use termcolor::{Color, ColorChoice, ColorSpec, StandardStream, WriteColor};

fn basic_usage() -> std::io::Result<()> {
    let mut stdout = StandardStream::stdout(ColorChoice::Auto);
    
    // 设置颜色
    stdout.set_color(ColorSpec::new().set_fg(Some(Color::Green)))?;
    writeln!(&mut stdout, "绿色文本")?;
    
    // 重置
    stdout.reset()?;
    
    Ok(())
}
```

### 缓冲区使用

```rust
use termcolor::{BufferWriter, Color, ColorChoice, ColorSpec, WriteColor};

fn buffered_usage() -> std::io::Result<()> {
    let bufwtr = BufferWriter::stderr(ColorChoice::Auto);
    let mut buffer = bufwtr.buffer();
    
    // 设置多种样式
    buffer.set_color(ColorSpec::new().set_fg(Some(Color::Red)).set_bold(true))?;
    writeln!(&mut buffer, "红色粗体")?;
    
    buffer.set_color(ColorSpec::new().set_fg(Some(Color::Blue)))?;
    writeln!(&mut buffer, "蓝色文本")?;
    
    // 输出缓冲区
    bufwtr.print(&buffer)?;
    
    Ok(())
}
```

### ANSI 直接控制

```rust
use termcolor::Ansi;

fn ansi_direct<W: std::io::Write>(writer: W) -> std::io::Result<()> {
    let mut ansi = Ansi::new(writer);
    
    // ANSI 转义序列
    ansi.set_color(ColorSpec::new().set_fg(Some(Color::Yellow)))?;
    write!(ansi, "黄色文本")?;
    ansi.reset()?;
    
    Ok(())
}
```

## 条件编译路径

termcolor 使用 Rust 的 `#[cfg(...)]` 属性区分平台实现：

```rust
#[cfg(windows)]
mod imp {
    pub struct Ansi<W: Write> {
        // Windows Console 实现
        console: Console,
        out: W,
    }
}

#[cfg(unix)]
mod imp {
    pub struct Ansi<W: Write> {
        // Unix/OH ANSI 实现
        out: W,
    }
}
```

**OpenHarmony 激活的路径**：`#[cfg(unix)]`

这意味着在 OpenHarmony 上：
- 使用 ANSI 转义序列进行颜色输出
- 无 Windows 特定代码被编译
- 行为与 Linux/macOS 完全一致

## 行为一致性验证

### 环境变量检测

termcolor 的 `ColorChoice::Auto` 模式会检测以下环境变量：

| 环境变量 | 值 | 行为 |
|---------|-----|------|
| `NO_COLOR` | 任意值 | 禁用颜色 |
| `TERM` | dumb | 禁用颜色 |
| `TERM` | 未设置（非 Windows） | 禁用颜色 |

**在 OpenHarmony 上的行为**：
- ✅ 与上游行为完全一致
- ✅ `TERM` 和 `NO_COLOR` 变量正常工作
- ✅ 无 OH 特定的环境变量处理

### 终端兼容性

termcolor 生成的 ANSI 转义序列兼容以下终端：

| 终端类型 | 兼容性 |
|---------|-------|
| Linux 终端 | ✅ 完全兼容 |
| macOS Terminal | ✅ 完全兼容 |
| OpenHarmony HDC | ✅ 完全兼容 |
| SSH 远程终端 | ✅ 兼容（取决于远端） |
| Windows CMD/PowerShell | ✅ 通过 Windows API 处理 |

## API 使用限制

### 已知限制

| 限制 | 说明 | 解决方案 |
|------|------|---------|
| 非 TTY 输出 | 在非终端环境中，颜色可能不被识别 | 使用 `ColorChoice::Always` 强制 |
| 终端类型检测 | TERM 环境变量可能未设置 | 考虑使用 `atty` crate |
| 缓冲区大小 | 依赖系统页面大小 | 通常无影响 |
| 线程同步 | BufferWriter 不是线程安全的 | 每线程独立 Buffer |

### 不支持的功能

termcolor **不支持**以下功能（上游设计如此，非 OH 限制）：

| 功能 | 原因 | 替代方案 |
|------|------|---------|
| 24-bit 真彩色 | 仅支持 ANSI 16 色 + 256 色 | 使用 RGB(u8, u8, u8) 支持部分 |
| 256 色优化 | 主要支持 16 色 | 使用 `Color::Ansi(u8)` |
| 动画效果 | 不在设计范围内 | 手动发送转义序列 |
| 光标控制 | 不在设计范围内 | 手动发送转义序列 |

## 总结

termcolor 在 OpenHarmony 中保持了 100% 的 API 兼容性和行为一致性。开发者可以完全按照上游文档使用此库，无需任何 OH 特定的适配或注意事项。

**核心结论**：
- ✅ 所有 API 完整可用
- ✅ 所有行为与上游一致
- ✅ 无 OH 特定扩展
- ✅ 无功能删减
