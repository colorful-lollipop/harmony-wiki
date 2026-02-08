# ylong_json Wiki

## 简介

本文档是 OpenHarmony `ylong_json` 组件的工程 Wiki，提供从代码层面深入理解该项目的完整资料。

## 覆盖范围

本文档覆盖以下内容：

- **项目概览**: 定位、边界、核心能力
- **架构设计**: 组件图、数据流、关键时序
- **目录结构**: 模块职责、文件组织
- **对外接口**: C FFI API 完整清单
- **内部实现**: 核心数据结构、算法
- **构建系统**: GN targets、编译产物
- **安全风险**: 攻击面分析、可被利用点
- **问题排查**: 常见构建/运行问题

## 未覆盖内容

- 测试相关代码（`tests/` 目录）
- 性能基准测试（`benches/` 目录）
- 第三方库内部实现（serde）

## 阅读路径

### 快速导航

查看 [SUMMARY.md](SUMMARY.md) 获取完整的文档导航和推荐阅读路径。

### 新人学习路线（30 分钟理解）

适合：系统服务层开发者、需要使用 JSON 解析能力的 C/Rust 开发者

**第一步（5 分钟）**：[项目概览](00_Overview.md)
- 了解项目定位和核心能力
- 查看性能对比数据

**第二步（10 分钟）**：[架构说明](01_Architecture.md)
- 理解整体架构设计
- 掌握数据流和组件关系

**第三步（10 分钟）**：
- [对外 API (C FFI)](03_Public_API.md) - 如果使用 C 接口
- [内部 API (Rust)](04_Internal_API.md) - 如果使用 Rust 接口

**第四步（5 分钟）**：[GN 构建](05_GN_Targets.md)
- 了解如何集成到 OpenHarmony 项目
- 选择合适的 Feature 配置

### 安全研究路线（深度审计）

适合：安全审计人员、安全架构师

**第一步（10 分钟）**：[安全风险分析](07_Security_Analysis.md) - 从"攻击面清单"开始
**第二步（30 分钟）**：[安全风险分析](07_Security_Analysis.md) - 阅读"可被利用点"
**第三步（10 分钟）**：[安全风险分析](07_Security_Analysis.md) - 阅读"修复建议汇总"
**第四步**：深入源代码审计

详细阅读路径请参考 [SUMMARY.md](SUMMARY.md) 中的完整说明。

## 文档特色

- **证据优先**：每个技术结论都附带代码路径和行号
- **双路线导航**：为新人学习者（API 导向）和安全研究员（安全导向）提供不同的阅读路径
- **质量标准**：满足 Definition of Done 的所有要求

## 更新方式

本文档基于代码证据生成，当代码发生以下变更时需要更新：

- C FFI 接口变更（`src/adapter.rs`）
- 公共 API 变更（`src/lib.rs` 导出）
- 构建配置变更（`BUILD.gn`, `Cargo.toml`）
- 架构重大调整

## 工作区

- [项目评估](wiki/_work/ASSESSMENT.md) - 项目类型判定、受众分析、文档策略
- [工作笔记](wiki/_work/NOTES.md) - 代码证据汇总、关键发现
- [任务计划](wiki/_work/PLAN.md) - 任务进度追踪
- [文档评估](wiki/_work/DOC_EVALUATION.md) - 文档覆盖情况和质量评估

## 生成信息

- **生成时间**: 2026-02-07
- **代码版本**: 基于仓库 HEAD 版本
- **文档语言**: 中文
- **评估版本**: Phase 0 项目评估已完成

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [项目 README](../README_zh.md)
- [用户指南](../docs/user_guide_zh.md)
- [示例代码](../examples/)
