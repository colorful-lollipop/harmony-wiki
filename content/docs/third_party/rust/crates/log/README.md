# log 库 - OpenHarmony Wiki

## 库概览

**log** 是 Rust 生态中最基础的日志门面（logging facade）库，为 Rust 应用程序和库提供统一的日志 API 抽象。

| 项目 | 信息 |
|-----|-----|
| **版本** | 0.4.17 |
| **许可证** | Apache License 2.0 / MIT |
| **上游地址** | https://github.com/rust-lang/log |
| **OH 组件** | rust_log |
| **维护者** | fangting12@huawei.com |

## 核心定位

log 库在 OpenHarmony 中的作用：

1. **日志门面标准**：定义 Rust 日志的标准接口（`info!`、`warn!`、`error!`、`debug!`、`trace!`）
2. **工具链基础组件**：为 hdc、bindgen 等 Rust 工具提供日志能力
3. **解耦设计**：库使用者依赖 `log` 接口，具体日志实现由应用层选择（如 env_logger）

## 文档导航

### 必读文档

- **[摘要与路线](SUMMARY.md)** - 阅读建议和文档结构
- **[库概述](01_Overview.md)** - 原始库功能和 OH 定位
- **[Patch 分析](02_Patches.md)** - OH 适配说明（本库无 Patch）
- **[构建适配](03_Build_Integration.md)** - BUILD.gn 配置详解
- **[OH 使用情况](04_Usage_in_OH.md)** - 依赖关系和使用场景

### 补充文档

- **[API 差异](05_API_Differences.md)** - API 变更说明（无差异）
- **[安全分析](06_Security.md)** - 安全风险评估

## 快速结论

✅ **该库无任何 OH 特定 Patch**。log 库作为纯逻辑层库，天然支持跨平台，无需任何修改即可在 OpenHarmony 上工作。

### 关键要点

1. **无需 Patch**：库本身不涉及平台代码，跨平台兼容性好
2. **最小适配**：仅需 BUILD.gn 构建配置，无代码层修改
3. **核心依赖**：是 OpenHarmony Rust 工具链的基础组件
4. **安全风险低**：代码简单，无已知高危漏洞

## 相关资源

- [上游文档](https://docs.rs/log/)
- [Cargo.toml](https://crates.io/crates/log)
- [OH 组件配置](../../bundle.json)
