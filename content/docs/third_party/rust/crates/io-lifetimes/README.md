# io-lifetimes OpenHarmony Wiki

## 库概览

**io-lifetimes** 是 OpenHarmony 第三方库中的一个基础 Rust crate，提供低层 I/O 所有权和借用抽象。

| 属性 | 值 |
|------|-----|
| **上游版本** | 1.0.5 |
| **OH 组件名** | rust_io_lifetimes |
| **许可证** | Apache-2.0 WITH LLVM-exception / Apache-2.0 / MIT |
| **Patch 数量** | 0（无需 Patch） |
| **维护成本** | 极低 |

### 核心功能

- 提供 `OwnedFd` 和 `BorrowedFd` 等 I/O 安全类型
- 实现 I/O Safety RFC 3128
- 为 `rustix` 等系统调用库提供基础抽象
- 零开销抽象，FFI 安全

---

## OH 适配概述

### 无需 Patch 的原因

io-lifetimes 在 OpenHarmony 中**未应用任何 Patch**，原因如下：

1. **纯类型系统抽象**：无平台特定汇编代码，纯 Rust 实现
2. **完善的条件编译**：通过 `#[cfg]` 属性优雅处理跨平台差异
3. **标准 RFC 实现**：I/O Safety RFC 3128 的官方参考实现，API 稳定
4. **依赖简单**：仅依赖 `libc`，无复杂依赖链

### BUILD.gn 关键配置

```gn
ohos_cargo_crate("lib") {
  crate_name = "io_lifetimes"
  crate_type = "rlib"
  external_deps = [ "rust_libc:lib" ]
  features = [ "close", "libc", "windows-sys" ]
  # ...
}
```

---

## 在 OH 中的作用

### 定位

io-lifetimes 是 OH Rust 生态的**基础设施层**：

```
应用层 → rustix → io-lifetimes → libc
       → is-terminal → rustix → io-lifetimes
```

### 主要依赖者

| 组件 | 用途 |
|------|------|
| **rustix** | POSIX 系统调用封装，使用 io-lifetimes 实现 I/O 安全 |
| **is-terminal** | 终端检测，使用 `AsFilelike` trait |

---

## 文档导航

### 快速开始

- **[01_Overview.md](01_Overview.md)** - 了解 io-lifetimes 是什么、在 OH 中的作用
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看谁在使用这个库

### 深度分析

- **[02_Patches.md](02_Patches.md)** - Patch 分析（本库无 Patch，解释原因）
- **[03_Build_Integration.md](03_Build_Integration.md)** - BUILD.gn 配置详解

### 工作文档

- **`_work/ASSESSMENT.md`** - 项目评估报告，包含详细的依赖分析

---

## 技术亮点

### 1. I/O 安全类型

```rust
// 替代原始的 RawFd
pub struct OwnedFd { ... }              // 拥有所有权，drop 时自动 close
pub struct BorrowedFd<'fd> { ... }      // 借用，带生命周期检查
```

### 2. FFI 安全

```rust
#[repr(transparent)]
pub struct OwnedFd { fd: RawFd }

// Option<OwnedFd> 与 C 的 NULL 指针大小相同
extern "C" {
    pub fn open(...) -> Option<OwnedFd>;  // 安全！
}
```

### 3. 零开销抽象

- 编译后与普通整数文件描述符性能相同
- 编译期生命周期检查，运行时无额外开销

---

## 维护建议

### 当前状态

- ✅ **稳定**：1.0.5 为稳定版本，API 已固化
- ✅ **无需维护**：无 Patch，无额外适配代码
- ✅ **低风险**：纯类型系统，无运行时漏洞风险

### 升级建议

- **被动升级**：仅在上游发布安全修复时考虑升级
- **升级流程**：替换源码 → 更新版本号 → 测试下游依赖
- **未来趋势**：Rust 1.63+ 已将 I/O Safety 合入标准库，此 crate 将逐渐成为兼容性垫片

---

## 相关链接

- **上游仓库**：https://github.com/sunfishcode/io-lifetimes
- **crates.io**：https://crates.io/crates/io-lifetimes
- **RFC 3128**：https://github.com/rust-lang/rfcs/blob/master/text/3128-io-safety.md
- **依赖者 rustix**：https://github.com/bytecodealliance/rustix

---

*本文档由 OpenHarmony Wiki Generator 自动生成*
*生成时间：2026-02-08*
