# 依赖关系与使用

## 4.1 依赖关系概览

### 直接依赖者

rustix 在 OpenHarmony 中的依赖关系相对简单：

| 模块 | 类型 | OH 组件名 | 依赖方式 |
|------|------|----------|----------|
| **is-terminal** | 直接依赖 | rust_is_terminal | `deps = ["//third_party/rust/crates/rustix:lib"]` |

### 间接依赖者

| 模块 | 依赖链 | OH 组件名 |
|------|--------|----------|
| **clap** | clap → is-terminal → rustix | rust_clap |
| **env_logger** | env_logger → is-terminal → rustix | rust_env_logger |

### 依赖统计

| 指标 | 数值 |
|------|------|
| **直接依赖 rustix 的模块数** | 1 |
| **间接依赖 rustix 的模块数** | 2 |
| **依赖 rustix 的 OH 组件总数** | 3 |

## 4.2 依赖关系图

### Mermaid 依赖图

```mermaid
graph TB
    subgraph "应用层"
        CLI[CLI 工具]
        LOG[日志工具]
    end

    subgraph "工具库层"
        CLAP[clap<br/>rust_clap]
        ENV[env_logger<br/>rust_env_logger]
    end

    subgraph "系统调用层"
        ISO[is-terminal<br/>rust_is_terminal]
        RUSTIX[rustix<br/>rust_rustix<br/>@ohos/rust_rustix]
    end

    subgraph "基础设施层"
        BITFLAGS[bitflags<br/>rust_bitflags]
        IOLIFE[io-lifetimes<br/>rust_io_lifetimes]
        LIBC[libc<br/>rust_libc]
        LNXRAW[linux-raw-sys<br/>rust_linux_raw_sys]
    end

    CLI --> CLAP
    LOG --> ENV
    CLAP --> ISO
    ENV --> ISO
    ISO --> RUSTIX
    RUSTIX --> BITFLAGS
    RUSTIX --> IOLIFE
    RUSTIX --> LIBC
    RUSTIX --> LNXRAW
```

### 文字依赖链

```
clap ────────► is-terminal ───────► rustix ──► libc
                                      │
                                      ├──► io-lifetimes
                                      │
                                      ├──► bitflags
                                      │
                                      └──► linux-raw-sys

env_logger ───► is-terminal ───────► rustix ──► libc
                                        │
                                        ├──► io-lifetimes
                                        │
                                        ├──► bitflags
                                        │
                                        └──► linux-raw-sys
```

## 4.3 直接依赖者详解

### is-terminal (rust_is_terminal)

| 属性 | 值 |
|------|-----|
| **OH 组件名** | rust_is_terminal |
| **上游仓库** | https://github.com/sunfishcode/is_terminal.rs |
| **版本** | 0.4.3 |
| **BUILD.gn 路径** | `/Volumes/lexar/code/d/work/oh/third_party/rust/crates/is-terminal/BUILD.gn` |

#### 功能描述

is-terminal 是一个简单的 Rust 库，用于检测给定的 I/O 流是否为终端设备。

#### rustix 使用方式

```rust
// is-terminal 内部使用 rustix 进行终端检测
use rustix::fd::AsFd;

pub fn is_terminal(fd: &impl AsFd) -> bool {
    rustix::fs::isatty(fd.as_fd()).is_ok()
}
```

#### 主要使用场景

1. **彩色输出控制**：检测 stdout 是否为终端，决定是否启用 ANSI 颜色
2. **交互式检测**：判断程序是否以交互式方式运行
3. **TTY 特定行为**：对终端和非终端环境采取不同行为

#### 依赖声明

```gn
deps = [
  "//third_party/rust/crates/rustix:lib",
  "//third_party/rust/crates/io-lifetimes:lib",
]
```

## 4.4 间接依赖者详解

### clap (rust_clap)

| 属性 | 值 |
|------|-----|
| **OH 组件名** | rust_clap |
| **上游仓库** | https://github.com/clap-rs/clap |
| **版本** | 4.1.13 |

#### 功能描述

clap 是一个命令行参数解析库，提供了声明式的命令行界面定义方式。

#### 依赖链

```
clap → is-terminal → rustix
```

#### is-terminal 的用途

clap 使用 is-terminal 检测终端，以：

- 决定是否启用彩色帮助信息
- 调整提示信息的显示格式
- 支持交互式命令补全

### env_logger (rust_env_logger)

| 属性 | 值 |
|------|-----|
| **OH 组件名** | rust_env_logger |
| **上游仓库** | https://github.com/env-logger-rs/env_logger |
| **版本** | 0.10.2 |

#### 功能描述

env_logger 是一个通过环境变量配置日志级别的日志库。

#### 依赖链

```
env_logger → is-terminal → rustix
```

#### is-terminal 的用途

env_logger 使用 is-terminal 的 `auto-color` 特性：

```rust
// env_logger 自动检测终端以决定是否启用彩色日志
if std::io::stdout().is_terminal() {
    // 终端环境：启用 ANSI 颜色
} else {
    // 管道/重定向：禁用颜色
}
```

## 4.5 rustix 的使用方式

### 链接方式

| 维度 | 配置 |
|------|------|
| **链接类型** | 静态链接 |
| **库类型** | rlib (Rust static library) |
| **输出扩展名** | .rlib |
| **依赖声明** | `deps = ["//third_party/rust/crates/rustix:lib"]` |

### 典型使用场景

#### 1. 文件描述符操作

```rust
use rustix::fd::AsFd;
use rustix::fs::{openat, O_RDONLY};

// 打开文件
let cwd = rustix::fs::cwd();
let file = openat(&cwd, "test.txt", O_RDONLY)
    .expect("Failed to open file");
```

#### 2. 终端检测

```rust
use is_terminal::IsTerminal;

// 检测 stdout 是否为终端
if std::io::stdout().is_terminal() {
    println!("\x1b[32mSuccess\x1b[0m");  // 绿色输出
} else {
    println!("Success");  // 普通输出
}
```

#### 3. 进程操作

```rust
use rustix::process::{fork, Fork};

// 进程 fork（如果启用了 process feature）
let pid = fork().expect("Fork failed");
// ...
```

### 启用功能的使用

#### 当前启用功能的使用

```rust
// 基础 I/O（默认启用）
use rustix::fd::OwnedFd;

// 终端操作（termios feature 启用）
use rustix::termios::{tcgetattr, tcsetattr};

// 文件描述符管理（io-lifetimes feature 启用）
use rustix::fd::AsFd;
```

#### 可选功能的使用（需要启用对应 feature）

```rust
// 文件系统操作（需要启用 fs feature）
use rustix::fs::{openat, unlinkat, mkdirat};

// 网络操作（需要启用 net feature）
use rustix::net::{socket, bind, connect};

// 进程操作（需要启用 process feature）
use rustix::process::{fork, execve, waitpid};
```

## 4.6 OpenHarmony 特有使用说明

### Linux 兼容模式下的行为

由于 rustix 在 OH 中通过 `CARGO_CFG_TARGET_OS=linux` 被识别为 Linux，其行为与在 Linux 上完全一致：

- 使用标准的 Linux 系统调用封装
- 遵循 POSIX 语义
- 支持 Linux 特有的扩展（如 epoll、eventfd）

### 注意事项

1. **功能限制**：当前 OH 构建未启用 fs、net、process 等功能
2. **平台检测**：rustix 检测到的平台是 Linux，而非 OpenHarmony
3. **系统调用兼容性**：依赖 OH libc 对系统调用的支持

### 扩展使用建议

如需使用更多 rustix 功能，可考虑：

1. **启用 fs feature**：支持文件系统操作
2. **启用 net feature**：支持网络编程
3. **启用 process feature**：支持进程管理
4. **启用 thread feature**：支持线程操作

```gn
features = [
  "io-lifetimes",
  "libc",
  "std",
  "use-libc-auxv",
  "termios",
  "fs",        // 启用文件系统操作
  "net",       // 启用网络操作
  "process",   // 启用进程操作
]
```

## 4.7 依赖升级影响

### 依赖版本信息

| 依赖 | OH 版本 | 上游版本 | 兼容性 |
|------|---------|---------|--------|
| bitflags | 1.2.1 | 最新 | 兼容 |
| io-lifetimes | 1.0.0 | 最新 | 兼容 |
| libc | 0.2.133 | 最新 | 需验证 |
| linux-raw-sys | 0.1.2 | 最新 | 兼容 |

### 升级注意事项

1. **libc 版本**：确保 OH libc 与 rustix 要求的版本兼容
2. **io-lifetimes**：检查 API 变更对 is-terminal 的影响
3. **功能开关**：新增 features 可能需要更新 BUILD.gn 配置
