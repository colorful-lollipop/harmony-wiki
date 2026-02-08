# API/接口差异

> OpenHarmony 版本与上游 env_logger 的 API 对比

---

## 概述

**结论：OpenHarmony 版本与上游 env_logger 完全一致，无 API 差异。**

由于 OpenHarmony 未对 env_logger 进行任何源代码修改，API 接口与上游版本 v0.10.2 完全相同。

---

## API 差异总览

| 类型 | OH 版本 | 上游版本 | 差异 |
|------|---------|----------|------|
| 新增 API | 0 | 0 | ❌ 无 |
| 行为变更 API | 0 | 0 | ❌ 无 |
| 废弃 API | 0 | 0 | ❌ 无 |
| 禁用功能 | 0 | 0 | ❌ 无 |

**结论**：100% 与上游一致

---

## OH 新增 API

**无新增 API**

OpenHarmony 未对 env_logger 添加任何新的 API 或接口。

---

## 行为变更 API

**无行为变更**

OpenHarmony 未对任何现有 API 的行为进行修改。

---

## 废弃或禁用的功能

**无废弃或禁用**

所有在上游版本中可用的功能在 OH 版本中均可使用。

---

## 完整 API 列表

### 核心初始化函数

#### `env_logger::init()`

```rust
pub fn init()
```

**功能**：初始化 logger，从环境变量读取配置

**参数**：无

**返回值**：无

**使用示例**：
```rust
use log::info;

fn main() {
    env_logger::init();
    info!("应用程序启动");
}
```

---

#### `env_logger::try_init()`

```rust
pub fn try_init() -> Result<(), SetLoggerError>
```

**功能**：尝试初始化 logger，允许手动处理错误

**参数**：无

**返回值**：`Result<(), SetLoggerError>`

**使用示例**：
```rust
use log::info;

fn main() {
    match env_logger::try_init() {
        Ok(_) => info!("Logger 初始化成功"),
        Err(e) => eprintln!("Logger 初始化失败: {}", e),
    }
}
```

---

### Builder 模式

#### `env_logger::Builder`

```rust
pub struct Builder
```

**功能**：提供灵活的 logger 配置方式

**常用方法**：

##### `new()`

```rust
pub fn new() -> Builder
```

**功能**：创建新的 Builder 实例

##### `from_env()`

```rust
pub fn from_env<'a>() -> Builder
```

**功能**：从环境变量创建 Builder

##### `from_env_with_env()`

```rust
pub fn from_env_with_env(env: Env<'a>) -> Builder
```

**功能**：使用自定义 Env 创建 Builder

##### `filter()`

```rust
pub fn filter(mut self, filter: Filter) -> Builder
```

**功能**：设置日志过滤器

##### `filter_level()`

```rust
pub fn filter_level(mut self, level: LevelFilter) -> Builder
```

**功能**：设置日志级别过滤器

##### `format()`

```rust
pub fn format<F>(mut self, format: F) -> Builder
where
    F: Fn(&mut Formatter, &Record) -> std::io::Result<()> + Send + Sync + 'static,
```

**功能**：自定义日志格式

##### `target()`

```rust
pub fn target(mut self, target: Target) -> Builder
```

**功能**：设置日志输出目标

##### `write_style()`

```rust
pub fn write_style(mut self, write_style: WriteStyle) -> Builder
```

**功能**：设置输出样式

##### `is_test()`

```rust
pub fn is_test(mut self, is_test: bool) -> Builder
```

**功能**：设置为测试模式

##### `try_init()`

```rust
pub fn try_init(self) -> Result<(), SetLoggerError>
```

**功能**：尝试初始化 logger

##### `init()`

```rust
pub fn init(self)
```

**功能**：初始化 logger（panic 失败）

**使用示例**：
```rust
use env_logger::{Builder, Env};
use log::info;

fn main() {
    Builder::from_env(Env::default().default_filter_or("info"))
        .filter_level(log::LevelFilter::Info)
        .init();

    info!("应用程序启动");
}
```

---

### Env 配置

#### `env_logger::Env`

```rust
pub struct Env<'a>
```

**功能**：配置环境变量读取方式

**常用方法**：

##### `new()`

```rust
pub fn new<'a>() -> Env<'a>
```

**功能**：创建新的 Env 实例

##### `default_filter_or()`

```rust
pub fn default_filter_or(mut self, filter: impl Into<String>) -> Env<'a>
```

**功能**：设置默认日志级别过滤器

##### `filter()`

```rust
pub fn filter(mut self, filter: impl Into<String>) -> Env<'a>
```

**功能**：设置日志过滤器变量名

##### `write_style()`

```rust
pub fn write_style(mut self, write_style: impl Into<String>) -> Env<'a>
```

**功能**：设置输出样式变量名

**使用示例**：
```rust
use env_logger::Env;
use log::info;

fn main() {
    let env = Env::default()
        .filter("MY_APP_LOG")
        .write_style("MY_APP_LOG_STYLE");

    env_logger::Builder::from_env(env)
        .default_filter_or("info")
        .init();

    info!("应用程序启动");
}
```

---

### Formatter

#### `env_logger::fmt::Formatter`

```rust
pub struct Formatter
```

**功能**：提供日志格式化功能

**常用方法**：

##### `default_level_style()`

```rust
pub fn default_level_style(&self) -> Style
```

**功能**：获取默认级别样式

##### `style()`

```rust
pub fn style(&self) -> Style
```

**功能**：创建自定义样式

##### `timestamp()`

```rust
pub fn timestamp(&self) -> Timestamp
```

**功能**：格式化时间戳

##### `timestamp_seconds()`

```rust
pub fn timestamp_seconds(&self) -> Timestamp
```

**功能**：格式化时间戳（秒）

##### `timestamp_millis()`

```rust
pub fn timestamp_millis(&self) -> Timestamp
```

**功能**：格式化时间戳（毫秒）

##### `timestamp_nanos()`

```rust
pub fn timestamp_nanos(&self) -> Timestamp
```

**功能**：格式化时间戳（纳秒）

**使用示例**：
```rust
use env_logger::Builder;
use log::info;

fn main() {
    Builder::new()
        .format(|buf, record| {
            writeln!(
                buf,
                "{}: {}",
                buf.timestamp(),
                record.args()
            )
        })
        .init();

    info!("应用程序启动");
}
```

---

### Filter

#### `env_logger::filter::Filter`

```rust
pub struct Filter
```

**功能**：日志过滤器

**常用方法**：

##### `new()`

```rust
pub fn new() -> Filter
```

**功能**：创建新的过滤器

##### `filter()`

```rust
pub fn filter(&self, module: &str, level: &Level) -> bool
```

**功能**：检查日志是否应该被输出

---

### 样式相关

#### `env_logger::fmt::WriteStyle`

```rust
pub enum WriteStyle {
    Auto,
    Always,
    Never,
}
```

**功能**：控制输出样式

**选项**：
- `Auto`：自动检测
- `Always`：总是启用
- `Never`：从不启用

---

#### `env_logger::fmt::Style`

```rust
pub struct Style
```

**功能**：样式控制

**常用方法**：

##### `set_color()`

```rust
pub fn set_color(&mut self, spec: &ColorSpec) -> Result<(), std::io::Error>
```

**功能**：设置颜色

##### `set_intense()`

```rust
pub fn set_intense(&mut self, intense: bool)
```

**功能**：设置强度

##### `set_dim()`

```rust
pub fn set_dim(&mut self, dim: bool)
```

**功能**：设置暗色

##### `set_bold()`

```rust
pub fn set_bold(&mut self, bold: bool)
```

**功能**：设置粗体

##### `set_underline()`

```rust
pub fn set_underline(&mut self, underline: bool)
```

**功能**：设置下划线

---

#### `env_logger::fmt::Color`

```rust
pub enum Color
```

**功能**：颜色枚举

**选项**：
- `Black`
- `Red`
- `Green`
- `Yellow`
- `Blue`
- `Magenta`
- `Cyan`
- `White`

---

#### `env_logger::fmt::ColorSpec`

```rust
pub struct ColorSpec
```

**功能**：颜色规范

**常用方法**：

##### `new()`

```rust
pub fn new() -> ColorSpec
```

**功能**：创建新的颜色规范

##### `set_fg()`

```rust
pub fn set_fg(&mut self, color: Option<Color>) -> &mut ColorSpec
```

**功能**：设置前景色

##### `set_bg()`

```rust
pub fn set_bg(&mut self, color: Option<Color>) -> &mut ColorSpec
```

**功能**：设置背景色

##### `set_bold()`

```rust
pub fn set_bold(&mut self, bold: bool) -> &mut ColorSpec
```

**功能**：设置粗体

---

### 环境变量配置

#### `RUST_LOG`

**功能**：控制日志级别

**语法**：
```bash
RUST_LOG=[target][=][level][,...]
```

**示例**：
```bash
RUST_LOG=info
RUST_LOG=my_crate=debug
RUST_LOG=error,my_crate=debug,other=info
```

---

#### `RUST_LOG_STYLE`

**功能**：控制输出样式

**选项**：
- `auto`：自动检测
- `always`：总是启用
- `never`：从不启用

**示例**：
```bash
RUST_LOG_STYLE=always
RUST_LOG_STYLE=never
```

---

## 功能对比表

| 功能 | 上游版本 | OH 版本 | 状态 |
|------|----------|---------|------|
| 环境变量配置 | ✅ | ✅ | ✅ 一致 |
| Builder 模式 | ✅ | ✅ | ✅ 一致 |
| 自定义格式 | ✅ | ✅ | ✅ 一致 |
| 模块过滤 | ✅ | ✅ | ✅ 一致 |
| 正则过滤 | ✅ | ✅ | ✅ 一致 |
| 彩色输出 | ✅ | ✅ | ✅ 一致 |
| 人类可读时间 | ✅ | ✅ | ✅ 一致 |
| 测试模式 | ✅ | ✅ | ✅ 一致 |

---

## 使用建议

### 推荐用法

由于 OH 版本与上游完全一致，建议：

1. **参考上游文档**：使用官方文档
2. **遵循最佳实践**：参考社区经验
3. **使用标准 API**：避免使用 unstable API

### 文档资源

- **上游文档**：https://docs.rs/env_logger
- **示例代码**：https://github.com/rust-cli/env_logger/tree/main/examples
- **log crate 文档**：https://docs.rs/log

---

## 迁移指南

### 从旧版本迁移

如果需要从旧版本的 env_logger 迁移到当前版本（v0.10.2），请参考上游的 [CHANGELOG](https://github.com/rust-cli/env_logger/blob/main/CHANGELOG.md)。

**主要变更**（从 0.9.x 到 0.10.x）：

1. **MSRV 升级**：Rust 最低版本要求从 1.41 升级到 1.60
2. **feature 名称变更**：
   - `atty` → `auto-color`
   - `termcolor` → `color`
3. **默认格式变更**：现在打印 target 而不是 module

### 迁移示例

#### 旧版本（0.9.x）

```rust
use env_logger;

fn main() {
    env_logger::init();
}
```

#### 新版本（0.10.x）

```rust
use log::info;

fn main() {
    env_logger::init();
    info!("应用程序启动");
}
```

**注意**：
- 需要添加 `log` crate 依赖
- 使用 `log` crate 提供的宏（`info!`, `debug!`, 等）

---

## 常见问题

### Q1: 为什么需要同时依赖 `log` 和 `env_logger`？

**A**：`log` crate 提供日志门面（facade），`env_logger` 是具体的实现。这种分离允许使用不同的日志实现而不需要修改应用代码。

### Q2: OH 版本是否支持所有上游 API？

**A**：是的，OH 版本与上游完全一致，支持所有 API。

### Q3: 如何自定义日志格式？

**A**：使用 `Builder::format()` 方法：

```rust
env_logger::Builder::new()
    .format(|buf, record| {
        writeln!(buf, "[{}] {}", record.level(), record.args())
    })
    .init();
```

### Q4: 如何在测试中使用日志？

**A**：使用 `is_test(true)`：

```rust
env_logger::Builder::new()
    .is_test(true)
    .try_init()
    .unwrap();
```

---

## 总结

### 关键结论

1. ✅ **完全兼容**：OH 版本与上游 API 完全一致
2. ✅ **无差异**：无新增、修改或废弃的 API
3. ✅ **可直接使用**：所有官方文档和示例都适用
4. ✅ **易于升级**：可直接跟随上游版本更新

### 维护建议

| 任务 | 说明 |
|------|------|
| API 兼容性检查 | 与上游版本对比，确保一致性 |
| 文档同步 | 跟随上游更新文档和示例 |
| API 变更跟踪 | 关注上游的 breaking changes |

---

**文档版本**：1.0
**更新时间**：2026-02-08
**评估版本**：env_logger v0.10.2
