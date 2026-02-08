# 项目概览

## 项目定位

`ylong_runtime` 是 OpenHarmony 系统中的 **Rust 异步运行时库**，为系统服务提供异步编程能力。

### 核心能力

| 能力 | 描述 | 源码位置 |
|------|------|----------|
| 异步任务调度 | 支持任务 spawn、join、取消 | `ylong_runtime/src/task/` |
| 异步 IO | TCP/UDP/文件非阻塞 IO | `ylong_runtime/src/io/`, `ylong_runtime/src/net/`, `ylong_runtime/src/fs/` |
| 同步原语 | Mutex、RWLock、Semaphore、Channel | `ylong_runtime/src/sync/` |
| 定时器 | 异步 sleep、interval | `ylong_runtime/src/time/` |
| 并行计算 | 数据自动分片并行处理 | `ylong_runtime/src/iter/` |
| 进程管理 | 子进程创建与管理 | `ylong_runtime/src/process/` |
| 信号处理 | Unix/Windows 信号处理 | `ylong_runtime/src/signal/` |

### 与标准库的关系

API 设计参考 Rust 标准库和 Tokio，将同步接口异步化：

| 标准库接口 | ylong_runtime 接口 | 用途 |
|------------|-------------------|------|
| `std::thread::spawn` | `ylong_runtime::spawn` | 异步任务创建 |
| `std::io::TcpStream` | `ylong_runtime::net::TcpStream` | 异步 TCP 连接 |
| `std::fs::File` | `ylong_runtime::fs::File` | 异步文件操作 |
| `std::sync::Mutex` | `ylong_runtime::sync::Mutex` | 异步互斥锁 |

**证据**: `ylong_runtime/src/lib.rs:45` → `pub use crate::task::{block_on, spawn, spawn_blocking}`

## 运行环境

| 属性 | 要求 |
|------|------|
| 操作系统 | Linux (主要), Windows (部分支持) |
| 目标设备 | OpenHarmony 标准设备 |
| 内存占用 | ROM ~100KB, RAM ~200KB |
| 依赖 | rust_libc, ffrt (可选) |

**证据**: `bundle.json:22-27`

## 双调度器架构

`ylong_runtime` 支持两种任务调度器，可通过 feature 选择：

### 1. ylong executor (Rust 原生)

```toml
[dependencies]
ylong_runtime = { features = ["current_thread_runtime"] }  # 单线程
ylong_runtime = { features = ["multi_instance_runtime"] }   # 多线程
```

- 纯 Rust 实现
- 完整的任务生命周期管理
- 适合独立应用

### 2. FFRT executor (OpenHarmony 默认)

```toml
[dependencies]
ylong_runtime = { features = ["ffrt"] }
```

- C++ 实现，通过 FFI 调用
- 利用 OpenHarmony 系统级调度优化
- 与系统服务深度集成

**注意**: `ffrt` 与 `current_thread_runtime`/`multi_instance_runtime` **互斥**

**证据**: `ylong_runtime/src/lib.rs:19-23`

## 模块依赖关系

```
用户代码
    ↓
ylong_runtime (主库)
    ├── ylong_io (IO 底层)
    ├── ylong_ffrt (FFRT 适配器)
    └── ylong_runtime_macros (过程宏)
```

**证据**: `ylong_runtime/BUILD.gn:32-35`
