# 05 - API/接口差异

## 概述

**clap 在 OH 中无任何 API 差异**。

由于本库采用零 Patch 集成，OH 中的 clap 与上游版本在 API 层面完全一致。开发者可以使用标准的 clap API，无需学习任何 OH 特有的用法。

## API 一致性声明

| 检查项 | 状态 |
|--------|------|
| 新增 OH 特有 API | 无 |
| 修改现有 API 行为 | 无 |
| 废弃/删除 API | 无 |
| 特性标志差异 | 仅启用 `derive`，无删减 |

## OH 与上游的功能一致性

### Derive API

完全兼容标准 clap 4.1.x：

```rust
use clap::Parser;

#[derive(Parser)]
#[command(name = "myapp")]
struct Cli {
    #[arg(short, long)]
    name: String,
}

// OH 中使用方式与上游完全一致
let cli = Cli::parse();
```

### Builder API

同样完全兼容：

```rust
use clap::{Command, Arg};

let cmd = Command::new("myapp")
    .version("1.0")
    .arg(Arg::new("config")
        .short('c')
        .long("config"));
```

## 特性标志对比

### OH 启用的特性

| 特性 | 说明 | 影响 |
|------|------|------|
| color | 终端彩色输出 | 正常 |
| error-context | 错误上下文 | 正常 |
| help | 帮助生成 | 正常 |
| std | 标准库 | 正常 |
| suggestions | 拼写建议 | 正常 |
| usage | 用法信息 | 正常 |
| derive | 派生宏 | **OH 显式启用** |

### 上游默认但未启用的特性

| 特性 | 说明 | OH 状态 |
|------|------|---------|
| cargo | Cargo 环境变量 | 未启用 |
| env | 环境变量支持 | 未启用 |
| unicode | Unicode 支持 | 未启用 |
| wrap_help | 帮助文本自动换行 | 未启用 |

**影响**: 这些特性缺失不影响 bindgen-cli 和 cxxbridge-cmd 的正常工作

## 升级兼容性

### 从 4.1.13 升级到更新版本

由于无 API 差异，升级时注意：

1. **检查依赖者兼容性**
   - bindgen-cli 使用的 API
   - cxxbridge-cmd 使用的 API

2. **验证 derive 宏兼容性**
   ```rust
   // 关键验证点
   #[derive(Parser)]
   struct Options { ... }
   ```

3. **构建测试**
   ```bash
   ninja -C out rust_bindgen:bindgen
   ninja -C out rust_cxx:cxxbridge
   ```

## 开发者指南

### 在 OH 中使用 clap

如果你正在开发 OH 的 Rust 工具，可以直接参考 [clap 官方文档](https://docs.rs/clap):

```rust
// 完全标准的 clap 用法
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(author, version, about)]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Generate bindings
    Bindgen {
        /// Input header
        input: String,
    },
}

fn main() {
    let cli = Cli::parse();
    // ...
}
```

### 特性需求

如果你需要使用未启用的特性（如 `cargo`、`unicode`）：

1. 检查 BUILD.gn 的 `features` 列表
2. 提交变更请求添加所需特性
3. 确保不影响现有依赖者

## 与 structopt 的关系

clap 3.0+ 吸收了 structopt 的功能，clap 4.x 的 derive API 等同于原 structopt。

| 库 | 状态 |
|----|------|
| structopt | 维护模式，建议迁移 |
| clap 4.x | 推荐，功能完整 |

OH 使用 clap 4.1.13，无需考虑 structopt 兼容性。

---

*文档版本: v1.0*
