# humantime 原始库简介

## 库基本信息

### 官方描述

> A parser and formatter for `std::time::{Duration, SystemTime}`

humantime 是一个 Rust 库，提供人性化的**时间格式化和解析**功能，核心特性包括：

- **持续时间解析**: 解析人类友好的持续时间字符串（如 `"15days 2min 2s"`）
- **持续时间格式化**: 格式化持续时间为可读形式（如 `"2years 2min 12us"`）
- **RFC3339 时间戳**: 解析和格式化 RFC3339 标准时间戳
- **高性能**: 时间戳解析/格式化经过优化，性能优异

### 上游仓库

- **GitHub**: https://github.com/tailhook/humantime
- **Crates.io**: https://crates.io/crates/humantime
- **文档**: https://docs.rs/humantime

## 核心功能

### 1. 持续时间解析 (Duration Parsing)

```rust
use std::time::Duration;
use humantime::parse_duration;

let duration = parse_duration("15days 2min 2s")?;
assert_eq!(duration, Duration::from_secs(1297322));
```

支持的时间单位：
- `ns`, `us`/`µs`, `ms`, `sec`/`s`, `min`/`m`, `hours`/`h`, `days`/`d`, `weeks`/`w`, `months`, `years`

### 2. 持续时间格式化 (Duration Formatting)

```rust
use std::time::Duration;
use humantime::format_duration;

let duration = Duration::from_secs(1297322);
println!("{}", format_duration(duration));  // "15days 2min 2s"
```

### 3. RFC3339 时间戳 (Timestamp Formatting)

```rust
use std::time::SystemTime;
use humantime::{format_rfc3339_millis, parse_rfc3339};

// 格式化当前时间
let ts = format_rfc3339_millis(SystemTime::now());
println!("{}", ts);  // "2024-01-15T10:30:45.123Z"

// 解析时间戳
let time = parse_rfc3339("2024-01-15T10:30:45Z")?;
```

支持多种精度：
- `format_rfc3339_seconds()` - 秒级精度
- `format_rfc3339_millis()` - 毫秒级精度
- `format_rfc3339_micros()` - 微秒级精度
- `format_rfc3339_nanos()` - 纳秒级精度

## 在 OpenHarmony 中的作用

### 功能定位

在 OpenHarmony 中，humantime 主要作为**日志系统的基础设施**，提供标准化的 RFC3339 时间戳格式化能力。

```
┌─────────────────────────────────────────────────────────────┐
│                    日志时间戳格式化                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   日志消息 ────────┐                                        │
│                    │                                        │
│   时间戳 ──────────┼───>  [humantime::format_rfc3339_*]  ───>  格式化输出
│   (SystemTime)     │          RFC3339 标准格式              │
│                    │                                        │
│   日志级别 ────────┘                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 使用特点

| 特性 | 说明 |
|------|------|
| **主要功能** | 时间戳格式化 (`format_rfc3339_millis`) |
| **次要功能** | 暂未被使用 |
| **性能** | 高性能，适合高频日志场景 |
| **标准合规** | RFC3339 标准格式，便于日志解析 |

### 使用组件

1. **env_logger**: 作为依赖提供日志时间戳功能
2. **HDC (HarmonyOS Device Connector)**: 直接使用生成带时间戳的日志文件名

## 版本信息

### 当前集成版本

| 属性 | 值 |
|------|-----|
| **上游版本** | 2.1.0 |
| **Rust Edition** | 2018 |
| **API 稳定性** | 稳定版 (Status: stable) |
| **最小 Rust 版本** | 1.31.0+ |

### 版本历史要点

- **v2.1.0**: 当前 OH 集成的版本
- **v2.0.0**: 重大版本更新，API 重构
- **v1.x**: 早期版本，API 有所不同

## 技术特点

### 设计哲学

1. **零成本抽象**: 时间戳格式化经过优化，性能接近手写代码
2. **纯 Rust 实现**: `#![forbid(unsafe_code)]`，完全内存安全
3. **标准库兼容**: 直接使用 `std::time::Duration` 和 `SystemTime`
4. **无依赖**: 除标准库外零依赖

### 性能数据（上游基准测试）

| 操作 | 性能 |
|------|------|
| RFC3339 格式化 (秒级) | ~73 ns/iter |
| RFC3339 解析 (秒级) | ~24 ns/iter |
| RFC3339 解析 (毫秒级) | ~28 ns/iter |

*作为对比：chrono 库同等操作约需 700+ ns*

## 许可证

- **Apache-2.0** 或 **MIT** (双许可)
- 用户可任选其一

## 相关资源

- **humantime-serde**: serde 集成 https://docs.rs/humantime-serde
- **原始仓库**: https://github.com/tailhook/humantime
