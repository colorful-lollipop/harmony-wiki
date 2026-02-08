# 原始库简介

> env_logger v0.10.2 - Rust 日志系统实现

---

## 1. 库基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | env_logger |
| **版本** | 0.10.2 |
| **发布日期** | 2024-01-18 |
| **许可证** | Apache License 2.0 OR MIT |
| **上游地址** | https://github.com/rust-cli/env_logger |
| **Rust 版本要求** | 1.60.0 (MSRV) |

## 2. 功能简介

env_logger 是 Rust 社区最常用的日志实现之一，作为 [`log`](https://docs.rs/log) crate 的具体实现。它提供：

- ✅ **环境变量配置**：通过 `RUST_LOG` 环境变量灵活配置日志级别
- ✅ **模块级过滤**：支持按模块或包名过滤日志
- ✅ **正则表达式过滤**：使用正则表达式过滤日志内容
- ✅ **彩色输出**：自动检测终端并启用彩色日志
- ✅ **人类可读时间**：友好的时间格式显示
- ✅ **自定义格式**：支持自定义日志格式

**一句话描述**：
> 一个可通过环境变量配置的 Rust 日志系统实现，配合 `log` crate 使用。

## 3. 核心概念

### 3.1 日志级别

env_logger 支持以下日志级别（按优先级从高到低）：

```rust
error // 错误 - 默认启用
warn  // 警告
info  // 信息
debug // 调试
trace // 跟踪
off   // 关闭（伪级别，用于禁用日志）
```

### 3.2 环境变量配置

**基础语法**：
```bash
RUST_LOG=[target][=][level][,...]
```

**常见示例**：

```bash
# 启用所有日志
RUST_LOG=trace

# 启用 info 及以上级别
RUST_LOG=info

# 为特定模块启用 debug
RUST_LOG=my_crate=debug

# 组合配置
RUST_LOG=error,my_crate=debug,other=info

# 使用正则表达式过滤
RUST_LOG=my_crate/.*error.*/trace
```

### 3.3 基本用法

```rust
use log::{info, debug, error};

fn main() {
    // 初始化 logger（从环境变量读取配置）
    env_logger::init();

    info!("应用程序启动");
    debug!("调试信息");
    error!("发生错误！");
}
```

运行：
```bash
$ RUST_LOG=info ./my_app
[2024-01-18T10:30:00Z INFO my_app] 应用程序启动
[2024-01-18T10:30:01Z ERROR my_app] 发生错误！
```

## 4. 特性（Features）

env_logger 提供多个可选特性：

| Feature | 说明 | 依赖 | OH 启用 |
|---------|------|------|---------|
| `color` | 启用彩色输出 | termcolor | 通过 auto-color 间接启用 |
| `auto-color` | 自动检测终端并启用彩色 | is-terminal, termcolor | ✅ 启用 |
| `humantime` | 人类可读时间格式 | humantime | ✅ 启用 |
| `regex` | 正则表达式日志过滤 | regex | ✅ 启用 |

## 5. 在 OpenHarmony 中的作用

### 5.1 定位

env_logger 在 OpenHarmony 中扮演**基础日志库**的角色，为 Rust 开发工具和构建工具提供标准化的日志输出能力。

### 5.2 适用场景

env_logger 适合用于：

- ✅ 开发工具（如 HDC 设备调试工具）
- ✅ 构建工具（如 bindgen-cli）
- ✅ 命令行工具
- ✅ 需要灵活日志控制的调试程序

**不适合**的场景：

- ❌ 需要持久化存储的日志（应使用 OH HiLog 系统）
- ❌ 跨进程日志收集（应使用专门的日志系统）
- ❌ 生产环境应用日志（应使用 OH 日志框架）

### 5.3 在 OH 生态中的位置

```
OH Rust 应用
    ↓
log crate (门面/Facade)
    ↓
env_logger (具体实现)
    ↓
标准输出 / 标准错误
```

**对比 OH HiLog**：

| 维度 | env_logger | OH HiLog |
|------|------------|----------|
| 目标场景 | 开发工具、CLI | 应用程序、系统服务 |
| 日志存储 | 仅输出到 stdout/stderr | 持久化到文件系统 |
| 日志查询 | grep 文件 | hilog 工具查询 |
| 系统集成 | 无 | 深度集成 OH 日志系统 |

## 6. 依赖关系

### 6.1 运行时依赖

| Crate | 版本 | 用途 |
|-------|------|------|
| `log` | 0.4.8+ | Rust 日志门面 |
| `regex` | 1.0.3+ | 正则表达式过滤（可选） |
| `termcolor` | 1.1.1+ | 终端颜色输出（可选） |
| `humantime` | 2.0.0+ | 人类可读时间格式（可选） |
| `is-terminal` | 0.4.0+ | 终端检测（可选） |

### 6.2 编译时依赖

- 无额外编译时依赖

## 7. 版本历史（OH 相关）

| 版本 | 发布日期 | OH 集成时间 | 关键变更 |
|------|----------|------------|----------|
| 0.10.2 | 2024-01-18 | 2024 | 当前 OH 版本 |
| 0.10.1 | 2023-11-10 | - | 性能优化 |
| 0.10.0 | 2022-11-24 | 2023-04-12 | 从 atty 迁移到 is-terminal，MSRV 升级到 1.60 |
| 0.9.3 | 2022-11-07 | 2023-04 前 | 修复编译问题 |

**关键里程碑**：
- **2023-04-12**：OH 首次集成 env_logger，版本为 0.9.3
- **版本火车适配**：升级到 0.10.2，跟随 OH 版本火车计划

## 8. 参考资源

### 官方资源

- **上游仓库**：https://github.com/rust-cli/env_logger
- **API 文档**：https://docs.rs/env_logger
- **crates.io**：https://crates.io/crates/env_logger
- **log crate**：https://docs.rs/log

### 相关文档

- [构建集成](../03_Build_Integration.md) - BUILD.gn 配置说明
- [使用场景](../04_Usage_in_OH.md) - 在 OH 中的实际使用案例
- [项目评估](./_work/ASSESSMENT.md) - 完整的技术评估报告

### 示例代码

上游示例：https://github.com/rust-cli/env_logger/tree/main/examples

包含：
- 自定义格式示例
- 不同目标输出示例
- 高级过滤示例

---

**文档版本**：1.0
**更新时间**：2026-02-08
