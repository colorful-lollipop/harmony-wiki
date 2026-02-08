# 文档导航

## 新人阅读路线

建议按以下顺序阅读：

```
1. 项目概览 → 2. 目录结构 → 3. 架构说明 → 4. API 参考 → 5. 构建系统 → 6. 安全评审
```

## 文档列表

| 章节 | 标题 | 说明 |
|------|------|------|
| 00 | [README](./README.md) | 文档说明、覆盖范围、更新方式 |
| 01 | [项目概览](./01_Overview.md) | 项目定位、核心能力、运行环境 |
| 02 | [目录结构](./02_Directory_Structure.md) | 模块职责、代码组织 |
| 03 | [架构说明](./03_Architecture.md) | Reactor-Executor、调度器、线程模型 |
| 04 | [API 参考](./04_API_Reference.md) | Rust API 清单、使用示例 |
| 05 | [构建系统](./05_Build_System.md) | GN targets、feature flags、产物路径 |
| 06 | [安全风险评审](./06_Security_Review.md) | 攻击面分析、风险点、修复建议 |

## 附录

| 文件 | 说明 |
|------|------|
| [附录 A：关键调用链](./appendix/A_Callgraphs.md) | 入口→核心逻辑调用链 |
| [附录 B：Feature Flags](./appendix/B_Feature_Flags.md) | 编译开关详解 |

## 代码证据索引

本文档中所有关键结论均可追溯到以下证据来源：

- **文件路径**: 精确到文件路径
- **符号名称**: 函数/结构体/宏/target 名称
- **行号引用**: 关键代码片段位置

示例证据引用格式：
> 证据：`ylong_runtime/src/lib.rs:45` → `pub use crate::task::{block_on, spawn, spawn_blocking}`
