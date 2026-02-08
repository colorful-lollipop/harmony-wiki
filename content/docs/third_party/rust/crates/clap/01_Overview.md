# 01 - 原始库简介

## 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | clap |
| **版本** | 4.1.13 |
| **许可证** | Apache-2.0 OR MIT (双许可) |
| **上游地址** | https://github.com/clap-rs/clap |
| **官方文档** | https://docs.rs/clap |

## 功能简介

clap 是 Rust 生态中最流行的命令行参数解析库之一，提供：

### 核心功能
- **声明式 API**: 通过 `#[derive(Parser)]` 宏定义 CLI
- **构建器 API**: 程序化构建命令行接口
- **子命令支持**: 类似 `git commit` 的子命令结构
- **自动生成帮助**: 基于代码生成 `-h/--help` 输出
- **智能错误提示**: 拼写错误提示、上下文错误信息
- **Shell 补全**: 支持 Bash/Zsh/Fish/PowerShell 自动补全生成

### 特性标志

| 特性 | 说明 |
|------|------|
| `derive` | 启用派生宏 |
| `color` | 终端彩色输出 |
| `help` | 帮助文档生成 |
| `suggestions` | 错误时提供建议 |
| `std` | 标准库支持 |

## 使用示例

### Derive API (推荐)
```rust
use clap::Parser;

#[derive(Parser)]
#[command(name = "myapp")]
struct Cli {
    /// Name of the person to greet
    #[arg(short, long)]
    name: String,
    
    /// Number of times to greet
    #[arg(short, long, default_value_t = 1)]
    count: u8,
}

fn main() {
    let cli = Cli::parse();
    for _ in 0..cli.count {
        println!("Hello {}!", cli.name);
    }
}
```

### Builder API
```rust
use clap::{Command, Arg};

let cmd = Command::new("myapp")
    .version("1.0")
    .arg(Arg::new("name")
        .short('n')
        .long("name")
        .help("Name to greet"));
```

## 在 OpenHarmony 中的作用

### 定位
clap 在 OH 中是**构建时依赖**而非运行时依赖，主要用于：

| 工具 | 用途 |
|------|------|
| bindgen-cli | 解析用户提供的 C/C++ 头文件路径和选项 |
| cxxbridge-cmd | 解析代码生成配置参数 |

### 为什么选用 clap？

1. **生态标准**: Rust FFI 工具的标准 CLI 解析方案
2. **功能丰富**: 自动生成帮助、错误提示，提升开发者体验
3. **维护活跃**: 隶属于 WG-CLI 工作组，有持续维护保障
4. **API 稳定**: 语义化版本控制，升级风险低

### OH 版本适配

| 上游版本 | OH 引入时间 | 升级说明 |
|----------|------------|----------|
| 4.1.4 | 2023-04 | 初始引入 |
| 4.1.13 | 2023-XX | bitflags 升级配套 |

---

*文档版本: v1.0*
