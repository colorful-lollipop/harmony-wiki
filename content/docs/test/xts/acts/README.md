# OpenHarmony XTS ACTS 工程 Wiki

## 文档覆盖范围

本文档为 OpenHarmony XTS ACTS（Application Compatibility Test Suite，应用兼容性测试套件）仓库的完整工程文档，旨在帮助开发者快速理解项目架构、构建流程、测试框架使用方法以及相关安全注意事项。

### 覆盖内容

本文档涵盖以下核心主题：

- **项目概述**：XTS 子系统定位、ACTS 测试套件的核心能力与设计目标
- **系统类型支持**：Mini System、Small System、Standard System 三种系统类型的差异与适配
- **测试框架**：HCTest（C 语言）、HCPPTest（C++）、HJSUnit（JavaScript）三种测试框架的详细用法
- **目录结构**：subsystem、subsystem_lite、tools 等核心目录的职责划分
- **构建系统**：GN 构建配置、target 定义、编译产物说明
- **测试用例开发**：从用例编写到编译、执行、结果分析的完整流程
- **安全风险评估**：基于代码证据的安全问题分析与修复建议

### 未覆盖内容

本文档**不包含**以下内容：

- OpenHarmony 整体架构设计（请参考 OpenHarmony 主仓库文档）
- 特定子系统或模块的业务逻辑实现细节（请参考各子系统仓库）
- CI/CD 流水线配置与自动化测试执行细节
- 商业认证或合规性相关流程说明

## 文档更新方式

本文档基于仓库代码自动生成，最后更新时间为 **2026-02-06**。

### 手动更新

如需手动更新本文档，请遵循以下流程：

1. 修改 `wiki/` 目录下的对应 `.md` 文件
2. 确保所有关键结论都有代码证据支撑（文件路径、符号名、代码片段）
3. 更新 `SUMMARY.md` 以保持导航链接一致
4. 提交更改并推送至远程仓库

### 代码驱动更新

对于以下场景，建议在代码提交时同步更新文档：

- 新增或删除测试框架宏、API
- 修改 GN 构建配置（新增/删除 target、变更依赖关系）
- 调整目录结构或模块职责
- 修复安全相关问题（需同步更新 `06_Security_Review.md`）

## 快速索引

| 主题 | 文档路径 | 适用读者 |
|------|----------|----------|
| 项目概览 | [01_Overview.md](./01_Overview.md) | 所有开发者 |
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) | 新入职开发者 |
| 测试框架 | [03_Test_Frameworks.md](./03_Test_Frameworks.md) | 用例开发者 |
| 构建系统 | [04_GN_Build.md](./04_GN_Build.md) | 构建工程师 |
| 安全评估 | [06_Security_Review.md](./06_Security_Review.md) | 安全工程师 |
| 故障排查 | [07_Troubleshooting.md](./07_Troubleshooting.md) | 测试工程师 |

## 贡献指南

欢迎社区开发者为本 Wiki 贡献内容。请注意以下原则：

- 所有结论必须可追溯到代码证据
- 禁止引用测试代码作为业务逻辑证据
- 保持文档风格一致，使用中文
- 更新 `SUMMARY.md` 以反映文档变更

## 许可证

本文档遵循 Apache License 2.0 许可证，与源码仓库许可证一致。
