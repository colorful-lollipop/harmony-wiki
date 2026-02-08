# rustix OpenHarmony Wiki

## 库概览

| 属性 | 值 |
|------|-----|
| **库名称** | rustix |
| **版本** | 0.36.16 |
| **许可证** | Apache-2.0 WITH LLVM-exception、Apache-2.0、MIT |
| **上游地址** | https://github.com/bytecodealliance/rustix |
| **OH 组件名** | @ohos/rust_rustix |
| **OH 版本** | 6.1 |
| **所属子系统** | thirdparty |

rustix 是由 Bytecode Alliance 维护的 Rust 库，提供对 POSIX/Unix/Linux/Winsock2 系统调用的安全绑定。在 OpenHarmony 中，该库通过 Linux 兼容模式进行适配，为 Rust 代码提供底层的系统调用接口。

## OpenHarmony 适配概述

### 适配策略

OpenHarmony 对 rustix 的适配采用了 **Linux 兼容模式**：

1. **无 OH 特定代码**：rustix 源代码中没有任何 `target_os = "ohos"` 的条件编译
2. **构建层适配**：通过 BUILD.gn 设置 `CARGO_CFG_TARGET_OS=linux`，将 OH 识别为 Linux 系统
3. **libc 后端**：强制使用 libc 而非 linux_raw 后端，确保系统调用兼容性

### Patch 情况

- **Patch 数量**：3 个
- **Patch 类型**：全部为 CI/QEMU 环境 Bugfix
- **直接修改 rustix**：否（Patch 修改的是 CI 环境中的 QEMU）

## 文档导航

### 核心文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](01_Overview.md) | 原始库功能简介及在 OH 中的定位 |
| [02_Patches.md](02_Patches.md) | **核心文档** - Patch 详细分析 |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 配置及构建适配 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系及使用场景 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异分析（如有） |
| [06_Security.md](06_Security.md) | 安全风险分析 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果（必读） |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度跟踪 |

## 快速开始

### OH 中使用 rustix

```rust
use rustix::fs::openat;
use rustix::fd::AsFd;

fn example() {
    let cwd = rustix::fs::cwd();
    let file = openat(&cwd, "test.txt", rustix::fs::O_RDONLY)
        .expect("openat failed");
}
```

### OH 构建配置

rustix 作为 OpenHarmony 的 thirdparty 组件，通过以下方式集成：

```gn
deps = [
    "//third_party/rust/crates/rustix:lib",
]
```

## 依赖关系

```
is-terminal ─────> rustix ──> libc
                         └──> io-lifetimes
                         └──> bitflags
                         └──> linux-raw-sys
```

详细依赖关系请参阅 [04_Usage_in_OH.md](04_Usage_in_OH.md)

## 版本信息

| 维度 | 版本 |
|------|------|
| **上游版本** | 0.36.16 |
| **OH bundle 版本** | 6.1 |
| **MSRV** | Rust 1.48 |
| **OH Rust 支持** | edition 2018 |

## 关键特性

### 已启用功能

| Feature | 状态 | 说明 |
|---------|------|------|
| `std` | ✓ | 标准库支持 |
| `libc` | ✓ | libc 后端 |
| `io-lifetimes` | ✓ | 文件描述符生命周期管理 |
| `termios` | ✓ | 终端 I/O 操作 |
| `use-libc-auxv` | ✓ | auxv 读取 |

### 未启用功能

| Feature | 状态 | 说明 |
|---------|------|------|
| `fs` | ✗ | 文件系统操作 |
| `net` | ✗ | 网络操作 |
| `process` | ✗ | 进程操作 |
| `io_uring` | ✗ | Linux io_uring |

## 相关链接

- [上游仓库](https://github.com/bytecodealliance/rustix)
- [上游文档](https://docs.rs/rustix)
- [OpenHarmony Rust crates]()
