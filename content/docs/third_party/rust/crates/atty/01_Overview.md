# 01 - 原始库简介

## 库基本信息

| 属性 | 内容 |
|------|------|
| **名称** | atty |
| **版本** | 0.2.14 |
| **作者** | softprops (Doug Tangren) |
| **许可证** | MIT |
| **上游仓库** | https://github.com/softprops/atty |
| **crates.io** | https://crates.io/crates/atty |
| **API 文档** | https://docs.rs/atty |

## 原始功能描述

**atty** 是一个简单的 Rust 库，用于回答一个简单的问题：

> "这是否是一个 TTY（终端）？"

TTY（Teletypewriter）检测是命令行工具开发中的常见需求。通过检测程序的标准输入、输出或错误流是否连接到终端，程序可以：
- 在交互式终端中使用颜色、进度条等富文本格式
- 在非交互式环境（如管道、重定向）中使用纯文本格式
- 根据交互性调整日志级别或提示方式

### 核心 API

```rust
use atty::Stream;

// 检测 stdout 是否为终端
if atty::is(Stream::Stdout) {
    println!("输出到终端");
} else {
    println!("输出被重定向");
}

// 使用 isnt 函数（语义更清晰）
if atty::isnt(Stream::Stdin) {
    println!("stdin 被重定向（可能是管道输入）");
}
```

### 支持的流类型

```rust
pub enum Stream {
    Stdout,  // 标准输出
    Stderr,  // 标准错误
    Stdin,   // 标准输入
}
```

## 平台支持

| 平台 | 支持状态 | 实现方式 |
|------|----------|----------|
| Linux / Unix | ✅ 支持 | `libc::isatty()` |
| Windows | ✅ 支持 | Win32 Console API + MSYS/Cygwin 检测 |
| WebAssembly | ✅ 支持 | 始终返回 `false` |
| Hermit OS | ✅ 支持 | `hermit_abi::isatty()` |
| macOS | ✅ 支持 | Unix 代码路径 |

## 该库在 OpenHarmony 中的作用和定位

### 功能定位

在 OpenHarmony 中，atty 库的定位是：

1. **基础设施库**: 作为 Rust 生态系统的基础组件，为命令行工具提供终端检测能力
2. **跨平台抽象**: 为 Rust 应用提供统一的 TTY 检测接口，屏蔽 Linux/Unix 底层差异
3. **CLI 工具支撑**: 支持 OH 中基于 Rust 的命令行工具开发

### 为什么选择该库

| 优势 | 说明 |
|------|------|
| 代码极简 | 核心逻辑仅约 50 行代码 |
| 零依赖 | 仅依赖系统 libc（标准库） |
| 跨平台 | 支持主流桌面和嵌入式平台 |
| 稳定成熟 | 版本 0.2.14，API 长期稳定 |
| 广泛采用 | Rust CLI 工具的事实标准 |

### 在 OH 架构中的位置

```
应用层
  │
  ├── Rust CLI 工具
  │      │
  │      └── atty (TTY 检测)
  │             │
  └── ─ ─ ─ ─ ─ └── libc (系统调用)
                       │
                  Linux Kernel
                       │
               OpenHarmony 系统
```

### 与 OH 组件的关系

- **依赖**: rust_libc（OH 的 libc crate）
- **被依赖**: 暂无发现直接依赖者（可能作为间接依赖使用）
- **子系统归属**: thirdparty（第三方组件）

## 技术实现细节

### Unix/Linux 实现

```rust
#[cfg(all(unix, not(target_arch = "wasm32")))]
pub fn is(stream: Stream) -> bool {
    let fd = match stream {
        Stream::Stdout => libc::STDOUT_FILENO,
        Stream::Stderr => libc::STDERR_FILENO,
        Stream::Stdin => libc::STDIN_FILENO,
    };
    unsafe { libc::isatty(fd) != 0 }
}
```

**关键点**:
- 使用 POSIX 标准 `isatty()` 系统调用
- OpenHarmony 基于 Linux 内核，完全兼容此实现
- 无需任何 OH 特定适配

### 版本历史（与 OH 相关）

| 版本 | 日期 | 重要变更 |
|------|------|----------|
| 0.2.14 | 2020-05 | 当前 OH 版本，新增 Hermit OS 支持 |
| 0.2.13 | 2019-11 | 支持旧版 Rust（2015 edition） |
| 0.2.11 | 2018-06 | MSYS 检测修复 |
| 0.2.0 | 2016-05 | 支持多种流类型 |

## 相关资源

- **上游仓库**: https://github.com/softprops/atty
- **crates.io**: https://crates.io/crates/atty
- **文档**: https://docs.rs/atty/0.2.14/atty/
- **Cargo.toml**: [../Cargo.toml](../Cargo.toml)
