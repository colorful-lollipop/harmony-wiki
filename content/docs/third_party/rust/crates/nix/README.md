# Nix - OpenHarmony 第三方库文档

> Rust 对 *nix 系统 API 的友好绑定库在 OpenHarmony 中的集成与适配说明

## 库概览

| 项目 | 内容 |
|------|------|
| **库名称** | nix |
| **版本** | 0.30.1 |
| **许可证** | MIT |
| **上游地址** | https://github.com/nix-rust/nix |
| **OH 组件** | @ohos/rust_nix |
| **所属子系统** | thirdparty |

## Nix 是什么？

Nix 是一个 Rust 库，为各种类 Unix 操作系统（Linux、Darwin、FreeBSD、OpenHarmony 等）提供友好的系统 API 绑定。它的核心价值在于：

1. **安全接口**：将不安全的 libc API 包装为类型安全的 Rust 接口
2. **跨平台统一**：统一不同 Unix 系统的调用方式，同时保留平台特定功能
3. **错误处理**：使用 Rust 的 `Result` 类型处理系统调用错误

## OpenHarmony 适配状态

### ✅ 原生支持

该库**无需任何 Patch** 即可在 OpenHarmony 上运行，因为：

- 上游代码已将 OpenHarmony (`target_os = "ohos"`) 列为 Tier 2 支持平台
- 支持的 OH 目标架构：
  - `aarch64-unknown-linux-ohos`
  - `armv7-unknown-linux-ohos`
  - `x86_64-unknown-linux-ohos`

### 构建配置

- **构建模板**：`ohos_cargo_crate`（OH Rust crates 标准模板）
- **输出类型**：rlib（Rust 静态库）
- **启用的 Features**：31 个核心功能模块

## 文档导航

### 快速开始

1. **[01_Overview.md](01_Overview.md)** - 了解 Nix 库的功能和定位
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 查看 OH 构建配置
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 学习如何在 OH 中使用

### 深入了解

4. **[02_Patches.md](02_Patches.md)** - Patch 分析（本库无需 Patch）
5. **[05_API_Differences.md](05_API_Differences.md)** - API 差异说明
6. **[06_Security.md](06_Security.md)** - 安全风险分析

### 完整阅读

查看 **[SUMMARY.md](SUMMARY.md)** 获取详细的阅读路线建议。

## 在 OH 中的使用示例

```rust
// 在 Rust 项目中添加依赖
use nix::unistd;
use nix::errno::Errno;

// 获取当前进程 ID
let pid = unistd::getpid();

// 使用 Result 类型处理系统调用
let result = unistd::chown("/path", uid, gid);
```

## 依赖关系

```
HDC (华为设备连接工具)
    │
    └──► nix (Rust bindings to *nix APIs)
            │
            ├──► libc (系统调用绑定)
            ├──► bitflags (类型安全标志)
            ├──► cfg-if (条件编译)
            ├──► memoffset (内存偏移)
            └──► pin-utils (Pin 类型工具)
```

## 维护信息

- **上游维护**：活跃开发中，GitHub stars: 2.5k+
- **OH 版本**：0.30.1（与上游同步）
- **OH Patch**：无
- **最后更新**：2026-02-07

## 相关资源

- [上游文档](https://docs.rs/nix/)
- [上游 GitHub](https://github.com/nix-rust/nix)
- [OH Rust 开发指南](../../../../docs/develop/Tool/Rust/README.md)
