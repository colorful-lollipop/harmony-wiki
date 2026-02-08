# lazy_static 在 OpenHarmony 中的集成文档

## 库概述

lazy_static 是 Rust 语言生态中用于声明惰性初始化静态变量的核心宏库。在 OpenHarmony 系统中，该库被集成在 `third_party/rust/crates/lazy-static.rs` 目录下，版本为 1.4.0，作为 Rust 组件构建基础设施的重要组成部分。

该库的主要功能是允许开发者通过简洁的宏语法定义「首次访问时初始化」的静态变量。这对于需要运行时计算、堆分配或复杂初始化逻辑的静态数据尤为有用。在 OpenHarmony 的 Rust 生态中，lazy_static 被多个关键模块所依赖，包括开发工具（hdc、bindgen）和安全服务（资产模块、代码签名模块）。

## OpenHarmony 适配特点

**原生集成，无 Patch**：与其他需要大量平台适配的第三方库不同，lazy_static 在 OpenHarmony 中以完全原生状态运行，未进行任何代码修改。这得益于 Rust 语言的设计和该库本身的高度平台无关性。

**标准构建适配**：OpenHarmony 使用 BUILD.gn 和 ohos_cargo_crate 模板来构建该库，配置简洁，仅包含必要的构建元数据。

**适度使用范围**：该库被 6 个 OpenHarmony 模块直接依赖，体现了其在 Rust 静态数据管理方面的实用价值。

## 文档导航

| 文档 | 内容说明 | 推荐阅读人群 |
|-----|---------|------------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议和文档结构说明 | 所有读者 |
| [01_Overview.md](./01_Overview.md) | 原始库功能介绍和 OH 定位 | 需要了解背景的读者 |
| [02_Patches.md](./02_Patches.md) | Patch 分析和无 Patch 原因说明 | 维护者和升级相关人员 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 配置详解 | 构建系统和 CI 工程师 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖者列表和使用场景分析 | 集成开发者和架构师 |
| [06_Security.md](./06_Security.md) | 安全风险分析和升级建议 | 安全工程师和维护者 |

## 快速参考

### 基础信息

| 属性 | 值 |
|------|-----|
| OH 组件名称 | @ohos/rust_lazy_static |
| 版本 | 1.4.0 |
| 许可证 | Apache 2.0 / MIT |
| 所属子系统 | thirdparty |
| 构建类型 | rlib（静态库） |
| Rust Edition | 2015 |

### 关键路径

- 源码目录：`third_party/rust/crates/lazy-static.rs/src/`
- 构建配置：`third_party/rust/crates/lazy-static.rs/BUILD.gn`
- 组件定义：`third_party/rust/crates/lazy-static.rs/bundle.json`
- Wiki 目录：`third_party/rust/crates/lazy-static.rs/wiki/`

### 相关资源

- 上游仓库：https://github.com/rust-lang-nursery/lazy-static.rs
- crates.io：https://crates.io/crates/lazy_static
- 官方文档：https://docs.rs/lazy_static

## 版本与维护状态

**上游维护状态**：该库目前处于「被动维护」（passively-maintained）状态。Rust 标准库自 1.80.0 版本起提供了等效的 `std::sync::LazyLock` 功能，这可能影响未来的依赖策略。

**OpenHarmony 集成状态**：当前版本（1.4.0）在 OpenHarmony 中运行稳定，无已知兼容性问题。由于无 Patch 集成，升级上游新版本时应相对平滑。

## 贡献与反馈

如发现文档错误、遗漏或需要更新内容，请联系组件维护者（fangting12@huawei.com）或通过 OpenHarmony 贡献流程提交改进建议。
