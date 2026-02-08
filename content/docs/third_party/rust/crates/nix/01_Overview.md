# 01 Nix 库概述

> Nix - Rust 对 *nix 系统 API 的友好绑定

## 基本信息

| 属性 | 值 |
|------|------|
| **库名称** | nix |
| **当前版本** | 0.30.1 |
| **上游版本** | 0.30.1 |
| **许可证** | MIT |
| **上游仓库** | [nix-rust/nix](https://github.com/nix-rust/nix) |
| **文档地址** | [docs.rs/nix](https://docs.rs/nix/) |
| **OH 组件** | @ohos/rust_nix |
| **版本号** | 5.0 |

## 原始库简介

Nix 是一个 **Rust 库**，旨在为各种类 Unix 操作系统提供 **友好的系统 API 绑定**。它的设计哲学是：

> "不追求 100% 统一的接口，而是统一可以统一的部分，同时保留平台特定的 API。"

### 核心设计目标

1. **安全替代方案**：为 `libc` crate 暴露的不安全 API 提供安全的 Rust 替代
2. **类型安全**：通过 Rust 类型系统和抽象强制执行合法、正确的 API 使用
3. **错误处理**：使用 Rust 的 `Result<_, Errno>` 类型处理系统调用错误，而非手动检查 errno
4. **跨平台兼容**：支持 Linux、Darwin、FreeBSD、OpenBSD 等多种 Unix 系统

### 与 libc 的对比示例

```rust
// libc API (不安全，需要手动处理返回码和 errno)
unsafe extern "C" fn gethostname(name: *mut c_char, len: size_t) -> c_int;

// nix API (返回 Result<OsString>，安全且 ergonomics)
pub fn gethostname() -> Result<OsString>;
```

## 功能模块详解

Nix 库按功能划分为多个模块，以下是主要模块及其功能：

### 进程与线程管理

| 模块 | 功能描述 | 关键类型 |
|------|----------|----------|
| `process` | 进程创建、执行、控制 | `Fork`, `Pwait`, `Exec` |
| `pthread` | POSIX 线程 | `pthread_t`, `pthread_attr_t` |
| `sched` | 调度策略控制 | `sched_param`, `CpuSet` |
| `signal` | 信号处理机制 | `Signal`, `SigAction` |

### 文件系统操作

| 模块 | 功能描述 | 关键类型 |
|------|----------|----------|
| `fs` | 文件属性、操作 | `File`, `OpenOptions`, `Stat` |
| `unistd` | Unix 标准函数 | `chown`, `unlink`, `symlink` |
| `dir` | 目录遍历 | `Dir`, `DirEntry` |
| `mount` | 挂载操作 | `mount`, `umount` |
| `mqueue` | POSIX 消息队列 | `MqAttr`, `mqueue` |

### 网络通信

| 模块 | 功能描述 | 关键类型 |
|------|----------|----------|
| `socket` | 套接字编程 | `Socket`, `SockFlag`, `SockAddr` |
| `net` | 网络地址解析 | `IpAddr`, `InetAddr` |
| `ifaddrs` | 网络接口枚举 | `Ifaddr`, `InterfaceFlags` |

### 系统调用与工具

| 模块 | 功能描述 | 关键类型 |
|------|----------|----------|
| `errno` | 错误码处理 | `Errno`, `Errno::ENOENT` |
| `mman` | 内存管理 | `mmap`, `mprotect`, `MapFlags` |
| `ucontext` | 用户上下文 | `UContext`, `Context` |
| `poll` | I/O 多路复用 | `PollFd`, `PollFlags` |
| `inotify` | Linux 文件系统监控 | `Inotify`, `WatchDescriptor` |

### 其他功能

| 模块 | 功能描述 |
|------|----------|
| `pty` | 伪终端操作 |
| `syslog` | 系统日志 |
| `term` | 终端控制 |
| `time` | 时间相关 |
| `quota` | 磁盘配额 |
| `reboot` | 系统重启 |
| `personality` | 进程人格设置 |
| `fanotify` | 文件系统事件监控 |
| `ioctl` | 设备控制 |
| `kmod` | 内核模块加载 |
| `acct` | 进程记账 |
| `ucontext` | 用户态上下文切换 |

## 平台支持

### 上游支持级别

Nix 对各平台的支持分为三个层级：

| 层级 | 描述 | OpenHarmony 状态 |
|------|------|------------------|
| **Tier 1** | CI 运行完整构建和测试，失败阻止代码合并 | ❌ 不支持 |
| **Tier 2** | CI 运行构建测试，失败阻止代码合并 | ✅ **Tier 2 支持** |
| **Tier 3** | CI 运行构建测试，失败不阻止代码合并 | ❌ 不支持 |

### OpenHarmony 支持详情

Nix 上游已将 OpenHarmony 列为 **Tier 2** 支持平台：

```
✅ aarch64-unknown-linux-ohos
✅ armv7-unknown-linux-ohos  
✅ x86_64-unknown-linux-ohos
```

这意味着：
- OpenHarmony 目标平台的构建会在 CI 中运行
- 构建失败会阻止代码合并到上游
- 核心功能已经过验证

### 条件编译

Nix 通过 `cfg-if` 和条件编译属性处理不同平台的差异：

```rust
#[cfg(target_os = "linux")]
pub mod inotify;

#[cfg(target_os = "ohos")]
pub mod inotify;  // OH 与 Linux 共享实现

#[cfg(target_os = "freebsd")]
pub mod sysctl;
```

## 在 OpenHarmony 中的定位

### 系统架构位置

```
┌─────────────────────────────────────┐
│     OpenHarmony 应用层              │
├─────────────────────────────────────┤
│     OpenHarmony 框架层              │
├─────────────────────────────────────┤
│     Rust 用户空间应用                │
│  (如 HDC, 其他 Rust 工具)           │
├─────────────────────────────────────┤
│         nix 库                       │  ◄── 系统调用绑定层
├─────────────────────────────────────┤
│         libc 库                      │
├─────────────────────────────────────┤
│     OpenHarmony 内核层              │
└─────────────────────────────────────┘
```

### OH 中的主要使用者

| 模块 | 用途 |
|------|------|
| **hdc** | 华为设备连接工具，用于进程管理、套接字通信等 |

### 为何选择 Nix？

对于 HDC 等 Rust 工具，Nix 提供了：

1. **类型安全**：避免 libc 的原始指针操作导致的内存错误
2. **错误处理**：`Result` 类型强制处理错误情况
3. **跨平台**：同一套代码可编译给 Linux 和 OH
4. **活跃维护**：上游社区活跃，持续更新

## 版本信息

### 当前版本特性 (0.30.1)

- MSRV (Minimum Supported Rust Version): **Rust 1.69**
- Edition: **2021**
- 发布日期：查看 [CHANGELOG.md](../../CHANGELOG.md)

### 版本变更历史

详见上游 [CHANGELOG.md](../../CHANGELOG.md)，主要变更包括：

- v0.30.x: 完善 OH 支持，添加更多系统调用绑定
- v0.29.x: 增加异步支持，完善错误处理
- v0.28.x: 大量新 API 添加，API 稳定性改进

## 许可证

Nix 使用 **MIT License**，详见 [LICENSE](../../LICENSE)。

```
Copyright (c) 2014-2024 The nix-rust Project Developers

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
...
```

## 相关资源

- [上游文档](https://docs.rs/nix/)
- [上游 GitHub](https://github.com/nix-rust/nix)
- [crates.io](https://crates.io/crates/nix)
- [OH Rust 开发指南](../../../../docs/develop/Tool/Rust/README.md)

---

**下一节**: [02_Patches.md](02_Patches.md) - Patch 分析
