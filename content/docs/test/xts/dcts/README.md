# DCTS Wiki - OpenHarmony 分布式兼容性测试套件

## 文档说明

本文档为 OpenHarmony DCTS（Distributed Compatibility Test Suite，分布式兼容性测试套件）的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、测试覆盖范围及构建系统。

## 覆盖范围

本文档涵盖以下内容：

| 文档 | 内容描述 | 改进版本 |
|------|----------|----------|
| [README.md](README.md) | 本文档，说明文档覆盖范围和更新方式 | v2.0 ✅ |
| [SUMMARY.md](SUMMARY.md) | 全站导航 + 推荐阅读路径（新人/安全双路线） | v2.0 ✅ |
| [01_Overview.md](01_Overview.md) | 项目定位、核心能力、运行环境、关键概念 | v2.0 ✅ |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 | v2.0 ✅ |
| [03_Architecture.md](03_Architecture.md) | 组件图、数据流、线程模型、关键时序 | v1.0 |
| [04_Modules.md](04_Modules.md) | 各模块详细说明、API 测试清单 | v1.0 |
| [05_Build_System.md](05_Build_System.md) | GN Targets、编译产物、构建配置 | v1.0 |
| [06_AttackSurface.md](06_AttackSurface.md) | **NEW**: 攻击面分析、利用路径示例 | v2.0 ✅ |
| [06_Security_Review.md](06_Security_Review.md) | **NEW**: 安全风险评估、修复建议 | v2.0 ✅ |
| [appendix/Test_Frameworks.md](appendix/Test_Frameworks.md) | 测试框架说明 | v1.0 |

## 更新方式

本 Wiki 由代码分析自动生成。如需更新：

1. **代码变更后**：重新运行分析工具或手动更新相关章节
2. **新增模块**：在 `02_Directory_Structure.md` 添加模块说明
3. **构建配置变更**：更新 `05_Build_System.md` 中的 targets 列表
4. **安全漏洞发现**：更新 `06_Security_Review.md` 和 `06_AttackSurface.md`

## 生成信息

- **生成时间**: 2026-02-07
- **代码仓库**: `/Volumes/lexar/code/d/work/oh/test/xts/dcts`
- **OpenHarmony 版本**: 3.1+
- **文档版本**: 2.0（v1.0 的重大改进版）
- **证据完整性**: 200+ 代码证据片段
- **文档完成度**: 核心文档 100%，可选文档 0%

## 关键改进（v2.0 相比 v1.0）

| 改进项 | v1.0 | v2.0 |
|--------|------|------|
| **代码证据** | 少量 | 大量（200+ 证据）|
| **安全文档** | 基础 | 完整（2 个安全文档）|
| **读者视角** | 单一路线 | 双路线（新人+安全）|
| **API 示例** | 少量 | 中等（分散在各模块中）|
| **风险评估** | 无 | 详细（6 大类风险+修复建议）|
| **代码地图** | 基础 | 详细（功能→文件路径映射）|

## 关键链接

- [OpenHarmony 官方文档](https://www.openharmony.cn/)
- [DCTS 测试用例开发指南](../README.md)
- [测试框架文档](./appendix/Test_Frameworks.md)

---

**文档维护**: 本 Wiki 由自动化工具生成，建议定期重新分析代码库以保持文档与代码同步。
