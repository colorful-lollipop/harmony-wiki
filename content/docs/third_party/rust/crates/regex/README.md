# regex 库 OpenHarmony 适配文档

## 库概述

**regex** 是 Rust 生态中广泛使用的正则表达式库，提供正则表达式的解析、编译和执行功能。在 OpenHarmony 系统中，该库作为基础依赖组件，为 `bindgen`、`env_logger` 等 Rust 工具提供正则表达式匹配能力。

| 属性 | 值 |
|------|-----|
| **上游版本** | 1.7.1 |
| **OH 版本号** | 6.1 |
| **许可证** | Apache License 2.0, MIT |
| **上游地址** | https://github.com/rust-lang/regex |
| **OH 组件名** | rust_regex |
| **所属子系统** | thirdparty |

## OpenHarmony 适配特点

### 无 Patch 集成

该库在 OpenHarmony 中**没有使用任何 Patch 文件**。这得益于 Rust 语言和 Cargo 构建系统的跨平台特性：

- Rust 代码天然支持跨平台编译
- regex 库完全通过 Rust 标准库实现功能，不依赖操作系统特定 API
- Cargo 与 OpenHarmony 的 GN 构建系统通过 `ohos_cargo_crate` 模板无缝集成

### 完整功能配置

OpenHarmony 版本启用了 regex 库的全部特性和优化选项：

- **性能优化**：启用 Aho-Corasick 算法、memchr 字节搜索、DFA 引擎、缓存优化等
- **Unicode 支持**：完整的 Unicode 字符属性、大小写转换、脚本识别、断字规则
- **标准库支持**：通过 `std` 特性确保完整的正则表达式功能

## 文档导航

### 核心文档

| 文档 | 说明 |
|------|------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议和文档结构概览 |
| [01_Overview.md](./01_Overview.md) | 库功能介绍和在 OH 中的定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建配置详解 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景 |

### 扩展文档

| 文档 | 说明 |
|------|------|
| [05_API_Differences.md](./05_API_Differences.md) | API 差异分析（无差异） |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 快速参考

### 依赖声明（GN）

```gn
deps = [
    "//third_party/rust/crates/regex:lib",
]
```

### 组件信息

- **Inner Kit 导出**：`//third_party/rust/crates/regex:lib`
- **Part 名称**：`rust_regex`
- **子系统**：`thirdparty`
- **输出类型**：`.rlib`（Rust 静态库）

## 相关资源

- [上游仓库](https://github.com/rust-lang/regex)
- [Crates.io 页面](https://crates.io/crates/regex)
- [上游文档](https://docs.rs/regex)
