# OpenHarmony State Registry 模块 Wiki

## 文档覆盖范围

本文档全面描述 OpenHarmony `state_registry` 模块的工程实践，包括架构设计、API 接口、构建系统和安全分析。

### 已覆盖内容

- **项目概述**：模块定位、核心能力、运行环境
- **目录结构**：模块划分、职责边界
- **架构设计**：组件图、数据流、线程模型
- **对外 API**：N-API 接口规范、JS API 清单
- **内部 API**：Inner API 接口、模块依赖 (含精确行号)
- **GN 构建**：Targets 列表、编译产物
- **攻击面分析**：外部输入入口、信任边界、敏感操作、攻击路径
- **安全评审**：风险识别、修复建议

### 文档质量

- ✅ **代码证据**：所有关键结论均有文件路径和行号支撑
- ✅ **攻击面覆盖**：完整的外部输入入口和权限检查点分析
- ✅ **双路线导航**：新人学习路线 + 安全研究路线
- ✅ **Mermaid 图表**：架构图、数据流图、信任边界图

### 未覆盖内容

- 详细测试代码分析（测试代码不在 Wiki 范围内）
- 运行时性能基准数据
- 历史版本变更记录
- 贡献者指南

### 更新方式

本文档基于代码仓库 `HEAD` 分支生成。更新步骤：

1. 修改对应源文件（`frameworks/`、`services/`、`interfaces/`）
2. 运行完整构建验证
3. 人工审查 Wiki 相关章节是否需要更新
4. 提交 PR 时附带文档更新

### 生成信息

- **代码仓库**：`//base/telephony/state_registry`
- **最后更新**：2026-02-07
- **维护者**：Telephony SIG
- **Wiki 版本**：v2.0 (优化版)
- **问题反馈**：https://gitee.com/openharmony/telephony_state_registry/issues

---

## 快速导航

| 主题 | 链接 |
|------|------|
| 新人入门 | [00_Overview.md](00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| JS API | [03_JS_API.md](03_JS_API.md) |
| Native API | [04_Native_API.md](04_Native_API.md) |
| GN 构建 | [05_GN_Build.md](05_GN_Build.md) |
| 编译产物 | [06_Build_Artifacts.md](06_Build_Artifacts.md) |
| 攻击面分析 | [05_AttackSurface.md](05_AttackSurface.md) |
| 安全评审 | [07_Security_Review.md](07_Security_Review.md) |
| 故障排查 | [08_Troubleshooting.md](08_Troubleshooting.md) |
| 全站导航 | [SUMMARY.md](SUMMARY.md) |
