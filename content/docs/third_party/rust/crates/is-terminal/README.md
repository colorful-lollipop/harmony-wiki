# is-terminal - OpenHarmony 第三方库文档

> is-terminal 是一个用于检测给定流是否为终端（tty）的 Rust 库

**OpenHarmony 组件**: `@ohos/rust_is_terminal` | **上游版本**: v0.4.3 | **许可证**: MIT

---

## 文档导航

本文档重点说明 `is-terminal` 在 OpenHarmony 中的集成与适配情况。

### 核心文档

| 文档 | 描述 | 读者 |
|------|------|------|
| **[SUMMARY.md](SUMMARY.md)** | 阅读路线建议 | 所有人 |
| **[01_Overview.md](01_Overview.md)** | 库概览、OH 适配概述 | 新手入门 |
| **[02_Patches.md](02_Patches.md)** | Patch 详细分析 | 维护者 |
| **[03_Build_Integration.md](03_Build_Integration.md)** | OH 构建适配 | 构建系统开发者 |
| **[04_Usage_in_OH.md](04_Usage_in_OH.md)** | OH 依赖关系与使用 | 架构师、开发者 |

### 工作文档

| 文档 | 描述 |
|------|------|
| **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** | 项目评估结果（Phase 0 收集的信息） |
| **[_work/NOTES.md](_work/NOTES.md)** | 分析过程记录 |
| **[_work/PLAN.md](_work/PLAN.md)** | 任务进度 |

---

## 快速概览

### 库功能
is-terminal 提供一个简单但强大的功能：**检测给定的流（stream）是否为终端**。

这在命令行工具中非常有用，用于：
- 根据是否在终端中运行，决定是否输出彩色文本
- 区分交互式与非交互式运行环境
- 调整日志输出格式

### 在 OpenHarmony 中的定位

**类型**: 基础设施库（Infrastructure Library）

**作用**: 为 OpenHarmony 的 Rust 生态提供终端检测能力

**集成方式**: 纯净上游集成（无代码修改）

**主要使用者**:
- `clap` (命令行参数解析器) - 用于终端颜色支持
- `env_logger` (日志库) - 用于自动彩色日志
- 间接支持: HDC、bindgen、cxx 等开发工具

---

## OH 适配概述

### 适配类型
**纯净上游集成** - 该库以最小改动集成到 OpenHarmony

### 适配内容
| 文件 | 用途 | 状态 |
|------|------|------|
| `BUILD.gn` | OpenHarmony GN 构建系统适配 | ✅ 已添加 |
| `bundle.json` | OH 组件化配置 | ✅ 已添加 |
| `README.OpenSource` | OH 归属文档 | ✅ 已添加 |
| 源代码 (`src/lib.rs`) | 上游原始代码 | ✅ 未修改 |

### 关键特性
- ✅ **无 Patch** - 源代码与上游 v0.4.3 完全一致
- ✅ **无 OH 特定代码** - 没有任何 `#ifdef OHOS` 或类似宏
- ✅ **易于升级** - 升级上游版本只需更新构建配置文件

---

## 快速开始

### 在 OH Rust 项目中使用

```rust
use is_terminal::IsTerminal;

fn main() {
    if std::io::stdout().is_terminal() {
        println!("正在终端中运行");
    } else {
        println!("未在终端中运行");
    }
}
```

### 在 OH BUILD.gn 中依赖

```gn
ohos_rust_executable("my_tool") {
    deps = [
        "//third_party/rust/crates/is-terminal:lib",
    ]
    sources = ["src/main.rs"]
}
```

---

## 相关资源

- **上游仓库**: https://github.com/sunfishcode/is-terminal
- **上游文档**: https://docs.rs/is-terminal
- **OH 负责人**: fangting12@huawei.com
- **OH 路径**: `third_party/rust/crates/is-terminal`
