# 05 - API/接口差异

## 概述

由于 atty 库在 OpenHarmony 中**没有任何 Patch**，其 API 与上游版本完全一致。

本文档说明：
1. 上游提供的标准 API
2. OH 是否有扩展或修改
3. 使用注意事项

## 上游 API 完整列表

### 核心 API

```rust
/// 检测指定流是否为 TTY（终端）
pub fn is(stream: Stream) -> bool

/// 检测指定流是否不是 TTY
pub fn isnt(stream: Stream) -> bool
```

### 枚举类型

```rust
/// 可检测的流类型
#[derive(Clone, Copy, Debug)]
pub enum Stream {
    Stdout,  // 标准输出
    Stderr,  // 标准错误
    Stdin,   // 标准输入
}
```

## API 详细说明

### `atty::is()`

**功能**: 检测指定流是否连接到终端

**签名**:
```rust
pub fn is(stream: Stream) -> bool
```

**参数**:
- `stream`: 要检测的流（`Stream::Stdout`, `Stream::Stderr`, `Stream::Stdin`）

**返回值**:
- `true`: 该流连接到终端（TTY）
- `false`: 该流被重定向（管道、文件等）

**示例**:
```rust
use atty::Stream;

if atty::is(Stream::Stdout) {
    // 在终端中运行，可以使用颜色、进度条等
    println!("\x1B[32m绿色文字\x1B[0m");
} else {
    // 输出被重定向（如：./program > output.txt）
    println!("纯文本，无颜色代码");
}
```

**平台行为**:

| 平台 | 实现 | 说明 |
|------|------|------|
| Unix/Linux (OH) | `libc::isatty()` | POSIX 标准 |
| Windows | `GetConsoleMode()` | Win32 API |
| WebAssembly | 始终返回 `false` | WASM 无 TTY 概念 |

### `atty::isnt()`

**功能**: `is()` 的否定版本，语义更清晰

**签名**:
```rust
pub fn isnt(stream: Stream) -> bool
```

**等价实现**:
```rust
pub fn isnt(stream: Stream) -> bool {
    !is(stream)
}
```

**示例**:
```rust
use atty::Stream;

// 检测 stdin 是否被重定向（如：echo "input" | ./program）
if atty::isnt(Stream::Stdin) {
    println!("从管道或文件读取输入");
    // 批量处理模式
} else {
    println!("交互式模式，等待用户输入");
}
```

### `Stream` 枚举

**定义**:
```rust
#[derive(Clone, Copy, Debug)]
pub enum Stream {
    Stdout,  // 标准输出（文件描述符 1）
    Stderr,  // 标准错误（文件描述符 2）
    Stdin,   // 标准输入（文件描述符 0）
}
```

**对应关系**:

| Stream | Unix FD | Windows Handle | 用途 |
|--------|---------|----------------|------|
| `Stream::Stdout` | 1 (STDOUT_FILENO) | STD_OUTPUT_HANDLE | 程序输出 |
| `Stream::Stderr` | 2 (STDERR_FILENO) | STD_ERROR_HANDLE | 错误信息 |
| `Stream::Stdin` | 0 (STDIN_FILENO) | STD_INPUT_HANDLE | 程序输入 |

## OH 与上游 API 对比

### 差异表

| API | 上游版本 | OH 版本 | 差异 |
|-----|----------|---------|------|
| `atty::is()` | ✅ 可用 | ✅ 可用 | 无差异 |
| `atty::isnt()` | ✅ 可用 | ✅ 可用 | 无差异 |
| `Stream` enum | ✅ 可用 | ✅ 可用 | 无差异 |
| 新增 API | ❌ 无 | ❌ 无 | 无新增 |
| 废弃 API | ❌ 无 | ❌ 无 | 无废弃 |

**结论**: API 100% 一致，无任何差异。

## 典型使用模式

### 模式 1: 条件颜色输出

```rust
use atty::Stream;

fn print_colored(text: &str, color_code: &str) {
    if atty::is(Stream::Stdout) {
        // 终端支持 ANSI 颜色
        println!("{}\x1B[0m", color_code, text);
    } else {
        // 重定向到文件，不输出颜色代码
        println!("{}", text);
    }
}
```

### 模式 2: 交互式 vs 批处理模式

```rust
use atty::Stream;
use std::io::{self, BufRead};

fn process_input() {
    if atty::is(Stream::Stdin) {
        // 交互式模式
        println!("请输入内容（按 Ctrl+D 结束）:");
    }
    
    let stdin = io::stdin();
    for line in stdin.lock().lines() {
        match line {
            Ok(content) => process_line(content),
            Err(_) => break,
        }
    }
}
```

### 模式 3: 错误输出格式化

```rust
use atty::Stream;

fn print_error(msg: &str) {
    if atty::is(Stream::Stderr) {
        eprintln!("\x1B[31m[ERROR] {}\x1B[0m", msg);
    } else {
        eprintln!("[ERROR] {}", msg);
    }
}
```

## 使用注意事项

### 1. 平台行为一致性

虽然 API 一致，但不同平台的底层行为可能略有差异：

| 场景 | Unix/Linux | Windows | WASM |
|------|------------|---------|------|
| 正常终端 | true | true | false |
| 管道 | false | false | false |
| 重定向到文件 | false | false | false |
| SSH 远程终端 | true | N/A | false |
| Docker 无 TTY | false | false | false |

### 2. 异步环境

atty 检测是同步操作，不适用于异步上下文：

```rust
// ✅ 正确：在同步代码中使用
fn main() {
    if atty::is(Stream::Stdout) {
        // ...
    }
}

// ❌ 不推荐：在异步上下文中调用（虽然可以工作，但设计上有更好的方式）
async fn async_task() {
    if atty::is(Stream::Stdout) {  // 可以工作，但...
        // ...
    }
}
```

### 3. 缓存检测结果

如果多次检测同一流，建议缓存结果：

```rust
use atty::Stream;
use std::sync::OnceLock;

static IS_TTY: OnceLock<bool> = OnceLock::new();

fn is_stdout_tty() -> bool {
    *IS_TTY.get_or_init(|| atty::is(Stream::Stdout))
}
```

## 与其他库的集成

### 与 clap 集成

```rust
use atty::Stream;
use clap::Parser;

#[derive(Parser)]
#[command(name = "myapp")]
struct Cli {
    #[arg(short, long, help = "强制彩色输出")]
    force_color: bool,
    
    #[arg(short, long, help = "禁用彩色输出")]
    no_color: bool,
}

fn main() {
    let cli = Cli::parse();
    
    let use_color = if cli.no_color {
        false
    } else if cli.force_color {
        true
    } else {
        atty::is(Stream::Stdout)
    };
    
    // 根据 use_color 决定输出格式
}
```

### 与 log/env_logger 集成

```rust
use atty::Stream;
use env_logger;

fn init_logger() {
    let mut builder = env_logger::Builder::from_default_env();
    
    // 如果输出不是终端，添加时间戳
    if atty::isnt(Stream::Stderr) {
        builder.format(|buf, record| {
            use std::io::Write;
            writeln!(buf, "[{}] {}", 
                record.level(),
                record.args()
            )
        });
    }
    
    builder.init();
}
```

## 总结

| 项目 | 状态 |
|------|------|
| OH 特定 API 扩展 | 无 |
| API 废弃 | 无 |
| 行为变更 | 无 |
| 与上游兼容性 | 100% |

**结论**: atty 在 OpenHarmony 中保持与上游完全一致的 API 和行为。开发者可以直接参考上游文档使用，无需考虑 OH 特定差异。
