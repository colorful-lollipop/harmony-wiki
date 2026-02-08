# io-lifetimes 概览

## 库基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | io-lifetimes |
| **上游版本** | 1.0.5 |
| **上游仓库** | https://github.com/sunfishcode/io-lifetimes |
| **作者** | Dan Gohman <dev@sunfishcode.online> |
| **许可证** | Apache-2.0 WITH LLVM-exception / Apache-2.0 / MIT |
| **Rust Edition** | 2018 |

## 原始库功能

io-lifetimes 是一个 **Rust 低层 I/O 所有权和借用库**，实现了 I/O Safety RFC 3128。它通过引入显式的所有权和生命周期概念，提供比传统 `RawFd` API 更安全的 I/O 操作抽象。

### 核心类型

```rust
// Unix/Linux 平台
pub struct BorrowedFd<'fd> { ... }  // 借用的文件描述符
pub struct OwnedFd { ... }             // 拥有的文件描述符

// Windows 平台对应
pub struct BorrowedHandle<'handle>;
pub struct OwnedHandle;
pub struct BorrowedSocket<'socket>;
pub struct OwnedSocket;
```

### 核心 Trait

| Trait | 功能描述 | 对应传统 API |
|-------|---------|-------------|
| `AsFd` | 获取借用引用 | `AsRawFd` |
| `IntoFd` | 转换为所有权类型 | `IntoRawFd` |
| `FromFd` | 从所有权类型创建 | `FromRawFd` |

### 关键特性

1. **FFI 安全**：`#[repr(transparent)]` 确保与 C 的 `int` 文件描述符 ABI 兼容
2. **Niche 优化**：`Option<OwnedFd>` 和 `Option<BorrowedFd>` 与原始指针大小相同，可安全用于 FFI
3. **生命周期检查**：借用类型带生命周期参数，编译时防止 use-after-close
4. **自动资源管理**：`OwnedFd` 实现 `Drop`，自动调用 `close(2)`

### 代码示例

```rust
use io_lifetimes::{AsFd, OwnedFd};

// FFI 声明 - 可直接使用 io-lifetimes 类型
extern "C" {
    pub fn open(pathname: *const c_char, flags: c_int, ...) -> Option<OwnedFd>;
    pub fn read(fd: BorrowedFd<'_>, ptr: *mut c_void, size: size_t) -> ssize_t;
    pub fn write(fd: BorrowedFd<'_>, ptr: *const c_void, size: size_t) -> ssize_t;
    pub fn close(fd: OwnedFd) -> c_int;
}

// 使用示例
let fd: OwnedFd = unsafe { open(path.as_ptr(), O_RDONLY) }?;
// fd 会在作用域结束时自动关闭
```

---

## 在 OpenHarmony 中的作用

### OH 组件信息

| 属性 | 值 |
|------|-----|
| **OH 组件名** | rust_io_lifetimes |
| **OH 版本** | 6.1 |
| **所属子系统** | thirdparty |
| **适配系统类型** | standard |

### OH 中的定位

在 OpenHarmony 中，io-lifetimes 扮演 **基础设施层** 角色：

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 / 业务代码                          │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ is-terminal  │  │   其他工具库  │  │    应用代码       │  │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘  │
├─────────┼────────────────┼──────────────────┼────────────┤
│         │                │                  │              │
│         ▼                │                  │              │
│  ┌──────────────┐        │                  │              │
│  │    rustix    │◄───────┴──────────────────┘              │
│  └──────┬───────┘                                         │
├─────────┼──────────────────────────────────────────────────┤
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────────────────────────────────┐             │
│  │         io-lifetimes (本库)               │             │
│  │  - 提供 I/O 安全抽象类型                  │             │
│  │  - OwnedFd / BorrowedFd                  │             │
│  │  - AsFd / IntoFd / FromFd traits         │             │
│  └────────┬─────────────────────────────────┘             │
├───────────┼────────────────────────────────────────────────┤
│           ▼                                                │
│  ┌──────────────────────────────────────────┐             │
│  │         libc / linux-raw-sys              │             │
│  └──────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

### 主要使用场景

1. **系统调用安全封装**（通过 rustix）
   - 文件操作：`open`, `read`, `write`, `close`
   - 网络操作：`socket`, `bind`, `listen`, `accept`
   - 进程操作：`pipe`, `dup`, `fcntl`

2. **终端检测**（通过 is-terminal）
   - 检测 stdin/stdout/stderr 是否为 TTY
   - 用于决定是否启用彩色输出、交互式提示等

3. **类型安全保证**
   - 防止文件描述符双重关闭
   - 防止使用已关闭的文件描述符
   - 编译期资源泄漏防护

### 与 Rust 标准库的关系

Rust 1.63+ 已将 I/O Safety 类型合入标准库：

```rust
// Rust 1.63+ 标准库已包含
use std::os::unix::io::{OwnedFd, BorrowedFd, AsFd};

// io-lifetimes 会检测并 re-export 标准库类型
#[cfg(io_safety_is_in_std)]
pub use std::os::unix::io::{AsFd, BorrowedFd, OwnedFd};
```

在 OH 中，io-lifetimes 作为 **兼容性垫片** 存在，为使用 Rust 1.48+ 的代码提供统一的 I/O 安全抽象。

---

## 为什么在 OH 中不需要 Patch

### 1. 纯抽象层实现

该库是纯粹的 **类型系统层** 抽象：
- 无平台特定汇编代码
- 使用 `libc` crate 进行实际的系统调用
- 通过 `cfg` 属性优雅处理跨平台差异

### 2. 标准 RFC 实现

io-lifetimes 是 **I/O Safety RFC 3128** 的参考实现：
- API 设计经过社区充分讨论
- 已被 Rust 标准库采纳
- 向后兼容性保证

### 3. 依赖链简单

```
io-lifetimes
    └── libc (已存在于 OH)
```

无复杂依赖，无需额外适配。

### 4. 平台支持完善

原生支持 OH 目标平台：
- ✅ Linux (标准系统)
- ✅ Unix-like 系统
- ❌ Windows（OH 不支持，但库已适配）

---

## 版本历史与维护状态

### 上游状态

- **当前版本**：1.0.5（稳定版）
- **维护状态**：维护模式
- **未来趋势**：随着 Rust 1.63+ 普及，将逐渐转为兼容性 crate

### OH 集成状态

- **集成时间**：2023年（从 BUILD.gn 版权日期推断）
- **Patch 数量**：0
- **维护成本**：极低

---

## 文档导航

- [概述](01_Overview.md)（本页）
- [Patch 分析](02_Patches.md) - 无 Patch，说明为何不需要
- [构建适配](03_Build_Integration.md) - BUILD.gn 详细分析
- [依赖与使用](04_Usage_in_OH.md) - 谁在使用及如何使用
