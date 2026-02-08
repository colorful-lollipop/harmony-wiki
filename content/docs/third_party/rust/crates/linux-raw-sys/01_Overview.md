# 原始库功能介绍

## 一、库基本信息

### 1.1 项目信息

| 项目 | 内容 |
|------|------|
| **库名称** | linux-raw-sys |
| **当前版本** | 0.1.4 |
| **上游地址** | https://github.com/sunfishcode/linux-raw-sys |
| **许可证** | Apache-2.0 WITH LLVM-exception OR Apache-2.0 OR MIT |
| **维护者** | Dan Gohman (sunfishcode) |
| **首次发布** | 约 2020 年 |
| **Rust 版本要求** | 1.48 或更高 |

### 1.2 功能简述

linux-raw-sys 是一个 Rust 库，提供 Linux 内核用户态 API 的 bindgen 生成绑定。该库的核心功能是将 Linux 内核头文件中的类型定义、常量和结构体转换为 Rust 代码，使 Rust 程序能够直接访问 Linux 系统调用接口。

该库的设计目标是提供**完整的** Linux 用户态 API 绑定，比传统的 libc 库提供更多的定义，包括 Linux 内核头文件中存在但未被 libc 导出的补充定义。

## 二、核心功能模块

### 2.1 general 模块

**功能描述**：提供 Linux 通用数据类型和结构定义，包括：

- **系统调用相关**：sysinfo、timespec、itimerval 等时间相关结构
- **进程相关**：pid_t、uid_t、gid_t 等 ID 类型
- **文件系统相关**：stat、statfs 等文件状态结构
- **网络相关**：sockaddr、msghdr、cmsghdr 等 socket 编程结构
- **共享内存**：shmid_ds 等共享内存相关结构

**默认启用**：是

### 2.2 errno 模块

**功能描述**：提供 Linux 错误码定义，包括：

- **标准错误码**：E2BIG、EACCES、EADDRINUSE 等数百个错误码常量
- **错误码与字符串映射**：errno 值的符号名称定义

**默认启用**：是

### 2.3 ioctl 模块

**功能描述**：提供设备控制接口定义，包括：

- **通用 ioctl 请求码**：IO、IOR、IOW、IORW 等宏定义
- **终端控制**：termios、winsize 等终端相关结构
- **套接字控制**：socket ioctl 请求码

**默认启用**：是（在 OH 适配中启用）

### 2.4 netlink 模块

**功能描述**：提供 Netlink 套接字相关定义，包括：

- **Netlink 协议族**：NETLINK_ROUTE、NETLINK_FIREWALL 等
- **Netlink 消息头**：nlmsghdr、nlattr 等
- **路由消息**：rtmsg、ifinfomsg、ifaddrmsg 等路由相关信息

**默认启用**：否

## 三、技术特点

### 3.1 代码生成机制

该库采用**离线代码生成**方式，与 linux-sys 等其他库的在构建时生成方式不同。代码生成工具位于 `gen/` 目录，主要流程如下：

1. **头文件收集**：从 Linux 内核源码收集相关头文件
2. **bindgen 处理**：使用 bindgen 工具生成 Rust 绑定
3. **后处理**：对生成的代码进行清理和优化
4. **多架构拆分**：按 CPU 架构拆分到不同目录

这种离线生成方式的优势是：
- **构建简单**：下游项目无需运行 bindgen
- **可预测性**：生成的代码是确定的
- **完整性**：可以包含更多 libc 未导出的定义

### 3.2 多架构支持

该库支持广泛的 CPU 架构，每种架构有独立的绑定文件：

| 架构 | 目录 | 支持状态 |
|------|------|----------|
| 32位 ARM | arm/ | 活跃 |
| 64位 ARM | aarch64/ | 活跃 |
| 32位 x86 | x86/ | 活跃 |
| 64位 x86 | x86_64/ | 活跃 |
| x32 | x32/ | 活跃 |
| 32位 MIPS | mips/ | 活跃 |
| 64位 MIPS | mips64/ | 活跃 |
| 32位 PowerPC | powerpc/ | 活跃 |
| 64位 PowerPC | powerpc64/ | 活跃 |
| 32位 RISC-V | riscv32/ | 活跃 |
| 64位 RISC-V | riscv64/ | 活跃 |
| SPARC | sparc/ | 活跃 |
| SPARC64 | sparc64/ | 活跃 |
| s390x | s390x/ | 活跃 |

### 3.3 条件编译

代码使用 Rust 的 `#[cfg]` 属性进行条件编译，主要条件包括：

```rust
#[cfg(feature = "errno")]
#[cfg(target_arch = "arm")]
#[path = "arm/errno.rs"]
pub mod errno;
```

- **feature 条件**：根据启用的功能特性编译对应模块
- **arch 条件**：根据目标 CPU 架构选择对应绑定
- **pointer_width 条件**：根据指针宽度区分 32/64 位变体

## 四、在 Rust 生态中的位置

### 4.1 与同类库的比较

| 库名称 | 生成方式 | 完整性 | 维护状态 |
|--------|----------|--------|----------|
| linux-raw-sys | 离线生成 | 高 | 活跃 |
| linux-sys | 构建时生成 | 中 | 活跃 |
| libc | 手动维护 | 中 | 非常活跃 |

### 4.2 与 rustix 的关系

**重要提示**：虽然 linux-raw-sys 提供了原始绑定，但**不建议直接在应用代码中使用**。上游社区推荐使用建立在该库之上的 rustix 库：

```
应用代码
    ↓
rustix（推荐使用，安全封装层）
    ↓
linux-raw-sys（底层绑定）
    ↓
Linux 内核
```

rustix 提供了：
- **类型安全**：Rust 枚举和结构体
- **内存安全**：避免原始指针操作
- **错误处理**：使用 Result 类型
- **更友好的 API**：更符合 Rust 惯用法

## 五、使用场景

### 5.1 适用场景

linux-raw-sys 适用于以下场景：

1. **底层系统编程**：需要直接进行系统调用的场景
2. **绑定层开发**：开发与其他语言交互的 FFI 绑定
3. **库开发**：开发需要访问 Linux 内核 API 的库（如 rustix）

### 5.2 不适用场景

对于大多数应用程序开发，推荐使用 rustix 而非直接使用 linux-raw-sys。

## 六、版本历史

### 6.1 主要版本

| 版本 | 发布日期 | 主要变更 |
|------|----------|----------|
| 0.1.4 | 约 2023 年 | 当前 OH 使用版本 |
| 0.1.3 | 更早 | 历史版本 |
| 0.1.0 | 初始版本 | 初始发布 |

### 6.2 版本策略

该库采用语义化版本控制（Semantic Versioning）：
- **主版本**：不兼容的 API 变更
- **次版本**：向后兼容的功能新增
- **修订版本**：向后兼容的 bug 修复

## 七、参考资料

- **上游 GitHub**：[https://github.com/sunfishcode/linux-raw-sys](https://github.com/sunfishcode/linux-raw-sys)
- **Crates.io**：[https://crates.io/crates/linux-raw-sys](https://crates.io/crates/linux-raw-sys)
- **API 文档**：[https://docs.rs/linux-raw-sys](https://docs.rs/linux-raw-sys)
- **相关项目 rustix**：[https://github.com/bytecodealliance/rustix](https://github.com/bytecodealliance/rustix)

---

**文档版本**：1.0
**最后更新**：2024年
