# API 差异分析

## 核心结论

**✅ 本库无任何 API 差异。**

log 库在 OpenHarmony 中使用的 API 与上游完全一致，未进行任何修改或扩展。

## API 对比表

| API 分类 | 上游 API | OH API | 差异 |
|---------|---------|--------|-----|
| **日志级别** | Level::{Error, Warn, Info, Debug, Trace} | 相同 | ❌ 无差异 |
| **日志宏** | error!, warn!, info!, debug!, trace! | 相同 | ❌ 无差异 |
| **Log Trait** | log::Log | 相同 | ❌ 无差异 |
| **记录器初始化** | log::logger() | 相同 | ❌ 无差异 |
| **过滤配置** | LevelFilter | 相同 | ❌ 无差异 |
| **结构化日志** | kv_unstable 特性 | **未启用** | ⚠️ 特性差异 |

## 日志级别 API

```rust
// 上游定义
pub enum Level {
    Error = 1,
    Warn = 2,
    Info = 3,
    Debug = 4,
    Trace = 5,
}

// OH 使用 - 完全相同
use log::Level;
let level = Level::Info;
```

**结论**：✅ 完整支持，无修改

## 日志宏 API

```rust
// 上游宏定义
macro_rules! error {
    ($($arg:tt)*) => { ... }
}

// OH 使用 - 完全相同
error!("Error message: {}", err);
warn!("Warning: {}", msg);
info!("Info: {}", info);
debug!("Debug: {}", detail);
trace!("Trace: {}", data);
```

**结论**：✅ 完整支持，无修改

## Log Trait API

```rust
// 核心 Trait 定义
pub trait Log: Sync + Send {
    fn enabled(&self, metadata: &Metadata) -> bool;
    fn log(&self, record: &Record);
    fn flush(&self);
}

// 可选扩展
pub trait LogFactory {
    fn logger(&self, name: &str) -> Box<dyn Log>;
}
```

**结论**：✅ 完整支持，无修改

## 特性启用差异

### 上游可用特性（OH 未全部启用）

| 特性 | 上游状态 | OH 状态 | 说明 |
|-----|---------|---------|-----|
| `std` | ✅ 可选 | ✅ 启用 | 标准库支持 |
| `max_level_*` | ✅ 可选 | ❌ 默认 | 最大日志级别限制 |
| `release_max_level_*` | ✅ 可选 | ❌ 默认 | 发布版级别限制 |
| `serde` | ✅ 可选 | ❌ 禁用 | serde 序列化支持 |
| `kv_unstable_std` | ✅ 可选 | ❌ 禁用 | 结构化日志（std） |
| `kv_unstable_serde` | ✅ 可选 | ❌ 禁用 | 结构化日志（serde） |

### OH 特性配置

```gn
# OH BUILD.gn
features = ["std"]  # 仅启用 std
```

```toml
# 上游 Cargo.toml (完整配置)
[features]
std = []
max_level_off = []
max_level_error = []
# ... 更多级别
release_max_level_* = []
kv_unstable = ["value-bag"]
kv_unstable_std = ["std", "kv_unstable", ...]
```

**影响**：
- OH 用户无法使用结构化日志 API（kv_unstable）
- OH 用户无法使用 serde 序列化
- 这些是上游的实验性/可选功能

## 未启用特性的 API 差异

### kv_unstable 特性（未启用）

如果启用 kv_unstable，可使用结构化日志：

```rust
// 需要 kv_unstable 特性
use log::{info, as_serde, as_error};

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

info!(user = as_serde!(&user); "User logged in");
warn!(error = as_error!(err); "Login failed");
```

**OH 现状**：
- ❌ 该 API 不可用
- ⚠️ 如需使用，需要在 BUILD.gn 中启用特性

### serde 特性（未启用）

```rust
// 需要 serde 特性
use log::Log;
use serde::ser::{Serialize, Serializer};

// 自定义序列化日志
impl Serialize for Record {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        // ...
    }
}
```

**OH 现状**：
- ❌ 该 API 不可用
- ⚠️ 如需使用，需要在 BUILD.gn 中启用特性

## API 使用建议

### 推荐的跨平台 API

```rust
// ✅ 安全使用的 API（上游 + OH 都支持）
use log::{error, warn, info, debug, trace};
use log::{Level, LevelFilter, Metadata};
use log::{Record, Log};

// ✅ 标准日志模式
error!("Error: {}", err);
warn!("Warning: {}", msg);
info!("Info: {}", info);
debug!("Debug: {}", debug_info);
trace!("Trace: {}", trace_info);

// ✅ 标准日志级别过滤
log::set_max_level(LevelFilter::Info);
```

### 不推荐的 API（OH 未启用）

```rust
// ❌ 结构化日志 - OH 不可用
info!(key = value; "message");

// ❌ serde 序列化 - OH 不可用
impl Serialize for MyType { ... }
```

### 如果需要高级特性

如需使用 `kv_unstable` 或 `serde`：

1. **修改 BUILD.gn**：
```gn
features = ["std", "kv_unstable_std"]
```

2. **验证依赖兼容性**：
```bash
cargo tree -p log -e features
```

3. **测试验证**：
```rust
#[cfg(feature = "kv_unstable")]
fn test_structured_log() {
    // ...
}
```

## 兼容性矩阵

| 功能 | OH 支持 | 上游支持 | 备注 |
|-----|--------|---------|-----|
| 基础日志 | ✅ | ✅ | 完全兼容 |
| 日志级别 | ✅ | ✅ | 完全兼容 |
| 日志过滤 | ✅ | ✅ | 完全兼容 |
| Log Trait | ✅ | ✅ | 完全兼容 |
| 结构化日志 | ❌ | ✅ | 需启用特性 |
| serde 集成 | ❌ | ✅ | 需启用特性 |

## 升级注意事项

### 未来可能的变化

| 变化 | 可能性 | 影响 |
|-----|-------|-----|
| kv_unstable 稳定化 | 中 | 可能启用 |
| API breaking changes | 低 | Rust 1.0 后稳定 |
| 新特性添加 | 高 | 可选启用 |

### 升级策略

1. **保持当前配置**：
   - 继续仅启用 `std` 特性
   - 避免引入上游实验性功能

2. **按需启用特性**：
   - 仅当依赖者需要时才启用
   - 评估特性和依赖的影响

3. **测试驱动升级**：
   - 先在测试环境验证
   - 再推广到生产环境
