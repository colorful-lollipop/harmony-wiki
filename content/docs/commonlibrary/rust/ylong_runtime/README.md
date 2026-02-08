# ylong_runtime 工程 Wiki

## 文档说明

本文档为 OpenHarmony `ylong_runtime` 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、使用方法、构建系统和安全注意事项。

### 覆盖范围

| 分类 | 覆盖内容 |
|------|----------|
| ✅ 项目定位 | 核心能力、模块边界、运行环境 |
| ✅ 目录结构 | 各模块职责（不含测试） |
| ✅ 架构设计 | Reactor-Executor 模式、调度器选择 |
| ✅ Rust API | 主要模块的公开接口 |
| ✅ 构建系统 | GN targets、feature flags、编译产物 |
| ✅ 安全分析 | 攻击面、风险点、修复建议 |
| ❌ N-API | **不存在**（纯 Rust 库） |
| ❌ 单元测试详情 | 按规范不引用测试代码 |

### 更新方式

本文档基于代码分析自动生成。如需更新：

1. 修改代码后重新运行文档生成脚本
2. 或手动更新对应章节

**最后更新**: 2026-02-06

### 文档导航

建议阅读顺序：

1. [项目概览](./01_Overview.md) - 快速了解项目定位
2. [目录结构](./02_Directory_Structure.md) - 理解模块划分
3. [架构说明](./03_Architecture.md) - 深入核心设计
4. [API 文档](./04_API_Reference.md) - Rust API 参考
5. [构建系统](./05_Build_System.md) - GN 构建配置
6. [安全风险评审](./06_Security_Review.md) - 安全注意事项

完整导航见 [SUMMARY.md](./SUMMARY.md)
