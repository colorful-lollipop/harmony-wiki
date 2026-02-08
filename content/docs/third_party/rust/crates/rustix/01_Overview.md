# rustix 原始库简介

## 1.1 库基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | rustix |
| **当前版本** | 0.36.16 |
| **维护者** | Bytecode Alliance |
| **许可证** | Apache-2.0 WITH LLVM-exception OR Apache-2.0 OR MIT |
| **上游仓库** | https://github.com/bytecodealliance/rustix |
| **文档地址** | https://docs.rs/rustix |
| **MSRV** | Rust 1.48 |
| **Edition** | 2018 |

## 1.2 核心功能概述

rustix 是一个 **安全 Rust 系统调用绑定库**，提供对 POSIX/Unix/Linux/Winsock2 系统调用的内存安全封装。其设计目标是为 Rust 代码提供接近底层但又类型安全的系统编程接口。

### 核心设计原则

1. **内存安全**：使用 Rust 引用、切片和返回值替代原始指针
2. **I/O 安全**：通过 `io-lifetimes` 库管理文件描述符生命周期
3. **错误处理**：使用 `Result` 类型返回错误，而非设置全局 `errno`
4. **类型安全**：使用 `bitflags` 替代裸整数标志位
5. **灵活参数**：通过 `Arg` trait 支持多种字符串类型作为路径参数

### 双后端架构

rustix 实现了两个可切换的后端：

| 后端 | 描述 | 优势 | 劣势 |
|------|------|------|------|
| **linux_raw** | 直接使用 Linux 系统调用和 vDSO | 性能更高、可内联、内存安全保证更完整 | 仅支持特定架构 |
| **libc** | 通过标准 C 库进行系统调用 | 跨平台兼容性好、支持更多架构 | 轻微性能开销 |

**OpenHarmony 选择**：在 OH 环境中强制使用 **libc 后端**，以确保兼容性。

## 1.3 API 功能模块

### 完整 API 模块

| 模块 | Cargo Feature | 功能描述 |
|------|---------------|----------|
| `rustix::io` | 默认启用 | 通用 I/O 操作、文件描述符管理 |
| `rustix::fd` | 默认启用 | 文件描述符类型（OwnedFd、AsFd） |
| `rustix::ffi` | 默认启用 | FFI 工具函数 |
| `rustix::fs` | `fs` | 文件系统操作（openat、mkdirat、unlinkat 等） |
| `rustix::path` | `fs`/`net` | 路径处理 |
| `rustix::net` | `net` | 网络操作（socket、bind、connect 等） |
| `rustix::process` | `process` | 进程操作（fork、execve、waitpid 等） |
| `rustix::thread` | `thread` | 线程操作（pthread_create 等） |
| `rustix::mm` | `mm` | 内存管理（mmap、mprotect、madvise 等） |
| `rustix::time` | `time` | 时间操作（clock_gettime、nanosleep 等） |
| `rustix::param` | `param` | 进程参数（getpid、getuid、geteuid 等） |
| `rustix::termios` | `termios` | 终端 I/O（tcgetattr、tcsetattr 等） |
| `rustix::rand` | `rand` | 随机数（getrandom 等） |
| `rustix::io_uring` | `io_uring` | Linux io_uring 接口 |

### OpenHarmony 启用状态

| 模块 | OH 启用状态 | 说明 |
|------|------------|------|
| `io`, `fd`, `ffi` | ✓ 默认 | 基础 I/O 和文件描述符 |
| `termios` | ✓ 启用 | 终端操作 |
| `fs` | ✗ 未启用 | 文件系统操作 |
| `net` | ✗ 未启用 | 网络操作 |
| `process` | ✗ 未启用 | 进程操作 |
| `thread` | ✗ 未启用 | 线程操作 |
| `mm` | ✗ 未启用 | 内存管理 |
| `time` | ✗ 未启用 | 时间操作 |
| `rand` | ✗ 未启用 | 随机数 |
| `io_uring` | ✗ 未启用 | io_uring |

## 1.4 平台支持

### 架构支持

| 架构 | linux_raw 后端 | libc 后端 |
|------|---------------|----------|
| x86_64 | ✓ | ✓ |
| x86 | ✓ | ✓ |
| aarch64 | ✓ | ✓ |
| arm (v5+) | ✓ | ✓ |
| riscv64gc | ✓ | ✓ |
| powerpc64le | ✓ | ✓ |
| mipsel | ✓ | ✓ |
| mips64el | ✓ | ✓ |
| s390x | ✗ | ✓ |
| 其他 Unix | ✗ | ✓ |
| Windows (部分) | ✗ | Winsock2 |

### OH 适配方式

OpenHarmony 并非 rustix 的原生支持平台，而是通过以下方式适配：

```bash
# OH 构建时设置的环境变量
CARGO_CFG_TARGET_OS=linux        # 伪装为 Linux
CARGO_CFG_TARGET_ARCH=x86_64    # 架构设置
```

这种方式使得 rustix 复用 Linux 的代码路径，无需修改 rustix 源代码。

## 1.5 在 OpenHarmony 中的定位

### 系统角色

```
┌─────────────────────────────────────────────────────┐
│              OpenHarmony 应用层                     │
│  (ace_engine, distributed_ddata, etc.)              │
├─────────────────────────────────────────────────────┤
│              Rust 运行时层                           │
│          (is-terminal, clap, env_logger)            │
├─────────────────────────────────────────────────────┤
│           rustix (系统调用抽象层)                     │
│   提供：文件系统、网络、进程、线程等系统调用封装       │
├─────────────────────────────────────────────────────┤
│              OpenHarmony libc                        │
│           (musl libc 或 OH libc)                    │
├─────────────────────────────────────────────────────┤
│              OpenHarmony 内核                        │
└─────────────────────────────────────────────────────┘
```

### 核心价值

1. **安全抽象**：为 Rust 代码提供类型安全的系统调用接口，避免内存安全问题
2. **跨平台兼容**：统一不同 Unix 系统的系统调用差异（OH 通过 libc 后端复用此特性）
3. **I/O 安全**：使用现代 Rust 的 I/O 安全特性（RFC 3128）
4. **生态桥梁**：连接 Rust 生态系统和 OpenHarmony 系统调用

### 与类似库的比较

| 库 | 特点 | 与 rustix 的关系 |
|---|------|-----------------|
| `nix` | 成熟的 Unix 系统调用库 | 功能类似，但 rustix 更注重 I/O 安全 |
| `libc` | C 标准库绑定 | rustix 基于 libc 后端的依赖 |
| `syscall` | 直接系统调用库 | rustix 的 linux_raw 后端与之类似 |
| `uapi` | Linux uAPI 绑定 | rustix 使用 linux-raw-sys 作为依赖 |

## 1.6 上游社区

### 维护状态

- **活跃度**：高（Bytecode Alliance 主导）
- **版本发布**：定期发布新版本
- **安全响应**：及时修复安全漏洞
- **社区贡献**：接受外部贡献

### 与标准库的关系

rustix 正在被探索作为 Rust 标准库在 `no_std` 环境下的底层实现：

```toml
[features]
rustc-dep-of-std = ["core", "alloc", "compiler_builtins"]
```

## 1.7 小结

rustix 是一个高质量的 Rust 系统调用绑定库，提供了安全、现代的 POSIX/Unix/Linux 系统调用接口。在 OpenHarmony 中，该库通过 Linux 兼容模式进行适配，复用了上游代码而无需 OH 特定的修改。当前 OH 仅启用了基础功能模块，更多高级功能（文件系统、网络、进程等）可根据需要启用。

下一章将详细介绍 rustix 在 OpenHarmony 中的 Patch 情况。
