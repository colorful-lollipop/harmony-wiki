# minimal-lexical Wiki

## 库概览

**minimal-lexical** 是 OpenHarmony 引入的第三方 Rust 库，用于提供快速、精确的浮点数字符串解析功能。

| 属性 | 值 |
|------|-----|
| **上游名称** | minimal-lexical |
| **上游版本** | 0.2.1 |
| **OH 组件名** | @ohos/rust_minimal_lexical |
| **OH 版本** | 6.1 |
| **许可证** | Apache 2.0 / MIT 双许可 |
| **上游地址** | https://github.com/Alexhuszagh/minimal-lexical |
| **Patch 数量** | 0（无 Patch） |
| **依赖者** | 1 个（nom） |

## 快速导航

### 基础信息
- [01_Overview.md](01_Overview.md) - 库简介与 OH 定位
- [02_Patches.md](02_Patches.md) - Patch 详细分析（本库无 Patch）
- [03_Build_Integration.md](03_Build_Integration.md) - OH 构建系统集成

### 使用分析
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中的依赖关系与使用场景
- [05_API_Differences.md](05_API_Differences.md) - API 差异分析
- [06_Security.md](06_Security.md) - 安全风险分析

### 工作文档
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估报告
- [_work/NOTES.md](_work/NOTES.md) - 分析过程笔记
- [_work/PLAN.md](_work/PLAN.md) - 任务进度计划

## OH 适配概述

### 适配类型：直接集成（无修改）

minimal-lexical 是 OpenHarmony 中**最简洁的第三方库集成案例**之一：

1. **无代码修改**：直接采用上游 0.2.1 版本源码，未应用任何 Patch
2. **标准构建**：使用 OH 标准的 `ohos_cargo_crate` GN 模板
3. **功能完整**：启用 `std` feature，提供完整功能
4. **单一依赖**：仅被 `nom` 库依赖，作为其浮点数解析后端

### 核心算法

该库实现了三种浮点数解析算法，确保精度和性能的平衡：

- **Eisel-Lemire 算法**：快速路径，处理大多数常见浮点数
- **Bellerophon 算法**：中等路径，处理需要更高精度的数字
- **慢速路径（Slow Path）**：使用大整数运算，处理极端情况

### 在 OH 中的作用

```mermaid
graph LR
    A[应用/组件] --> B[nom 解析器库]
    B --> C[minimal-lexical]
    C --> D[浮点数解析]
```

minimal-lexical 虽然不直接暴露给应用开发者，但作为 `nom` 的依赖，间接支持了 OpenHarmony 中各种解析需求：

- 配置文件解析（JSON、TOML、YAML 等）
- 网络协议解析（HTTP、自定义协议等）
- 数据格式解析（二进制格式、文本格式等）

## 文档阅读建议

### 快速了解（5 分钟）
阅读本文档 + [01_Overview.md](01_Overview.md)

### 深入了解（15 分钟）
阅读 [03_Build_Integration.md](03_Build_Integration.md) + [04_Usage_in_OH.md](04_Usage_in_OH.md)

### 维护者必读（30 分钟）
完整阅读所有文档，特别关注 [02_Patches.md](02_Patches.md)（无 Patch 的说明）和 [06_Security.md](06_Security.md)

## 关键结论

| 方面 | 结论 |
|------|------|
| **维护难度** | ⭐ 极低（无 Patch，直接跟随上游） |
| **升级策略** | 直接替换源码，更新版本号即可 |
| **安全风险** | 低（经过广泛测试和模糊测试） |
| **依赖影响** | 影响 nom 库，间接影响使用 nom 的组件 |

## 相关链接

- [上游文档](https://docs.rs/minimal-lexical)
- [上游仓库](https://github.com/Alexhuszagh/minimal-lexical)
- [相关项目 rust-lexical](https://github.com/Alexhuszagh/rust-lexical)（完整版）
- [nom 解析器库](https://github.com/rust-bakery/nom)

---

*本文档最后更新：2026-02-07*
