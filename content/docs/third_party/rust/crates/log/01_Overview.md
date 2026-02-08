# 原始库概述

## 库基本信息

| 属性 | 值 |
|-----|-----|
| **库名称** | log |
| **当前版本** | 0.4.17 |
| **许可证** | MIT OR Apache-2.0 |
| **上游地址** | https://github.com/rust-lang/log |
| **上游文档** | https://docs.rs/log/ |
| **最低 Rust 版本** | 1.31.0 |

## 原始功能描述

log 库是 Rust 生态系统的标准日志门面（logging facade），提供：

1. **日志级别定义**：Error、Warn、Info、Debug、Trace
2. **日志宏接口**：`error!`、`warn!`、`info!`、`debug!`、`trace!`
3. **日志门面模式**：库使用者依赖 log 接口，具体实现由应用层决定
4. **结构化日志支持**：可选的 kv_unstable 特性（实验性）

### 核心设计理念

```
┌─────────────────────────────────────────────────────┐
│                    应用程序                          │
│  ┌─────────────────────────────────────────────┐   │
│  │           具体日志实现 (如 env_logger)        │   │
│  └─────────────────────────────────────────────┘   │
│                         ▲                            │
│  ┌─────────────────────────────────────────────┐   │
│  │              log (日志门面)                   │   │
│  │  • 定义日志宏                                │   │
│  │  • 定义日志级别                              │   │
│  │  • 提供 Log trait                           │   │
│  └─────────────────────────────────────────────┘   │
│                         ▲                            │
│  ┌─────────────────────────────────────────────┐   │
│  │           库 (如 curl, serde)                │   │
│  │  • 使用 log! 宏输出日志                      │   │
│  │  • 不依赖具体日志实现                         │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

## 在 OpenHarmony 中的定位

### 工具链基础组件

log 库在 OpenHarmony 中承担 **Rust 工具链基础设施** 的角色：

| 角色 | 说明 |
|-----|-----|
| **日志标准** | 为所有 Rust 工具提供统一的日志接口 |
| **依赖基础** | bindgen、env_logger 等库的基础依赖 |
| **解耦设计** | 工具代码依赖 log 接口，具体实现可选 |

### 与其他组件的关系

```
OpenHarmony Rust 工具链
│
├── hdc (Huawei Device Connector)
│   └── log (日志输出)
│
├── bindgen (绑定生成器)
│   └── log (构建日志)
│
├── env_logger (日志实现)
│   ├── log (日志门面)
│   └── (提供具体日志输出到 stderr/系统日志)
│
└── 其他工具
    └── log (统一日志接口)
```

## 功能特性详解

### 日志级别

| 级别 | 数值 | 用途 |
|-----|-----|-----|
| Error | 1 | 错误信息，需要关注的问题 |
| Warn | 2 | 警告信息，可能有问题 |
| Info | 3 | 一般信息，重要操作确认 |
| Debug | 4 | 调试信息，开发时使用 |
| Trace | 5 | 跟踪信息，最详细的诊断 |

### 核心 API

#### 日志宏

```rust
use log::{error, warn, info, debug, trace};

error!("This is an error message");
warn!("This is a warning");
info!("Information message");
debug!("Debug information");
trace!("Detailed trace");
```

#### Log Trait

```rust
pub trait Log {
    fn enabled(&self, metadata: &Metadata) -> bool;
    fn log(&self, record: &Record);
    fn flush(&self);
}
```

#### 记录器初始化

```rust
use log::LevelFilter;
use env_logger::Env;

env_logger::Builder::from_env(Env::default())
    .filter_level(LevelFilter::Info)
    .init();
```

## 版本信息

### OH 版本与上游版本对应

| OH 版本 | log 版本 | 上游版本 | 差异 |
|-------|---------|---------|-----|
| 当前 | 6.1 | 0.4.17 | 完全一致 |

### 版本更新历史

OH 使用的版本与上游保持同步，无特殊版本定制。

## 许可证信息

log 库采用 **双重许可证**：MIT OR Apache-2.0

| 许可证 | 适用范围 |
|-------|---------|
| MIT | 自由使用，需保留版权声明 |
| Apache-2.0 | 自由使用，需保留许可证文件 |

在 OpenHarmony 中以 **Apache License 2.0** 为主许可证标识。
