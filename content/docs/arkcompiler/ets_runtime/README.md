# ArkCompiler ETS Runtime 文档

## 文档覆盖范围

本文档为 OpenHarmony ArkCompiler ETS Runtime 仓库的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、构建系统和安全特性。

### 涵盖内容

- **项目概述**：定位、边界、核心能力、运行环境
- **目录结构**：模块职责划分（不含测试代码）
- **架构说明**：组件图、数据流、线程模型、关键时序
- **对外 API**：N-API 接口清单、参数校验、错误码
- **内部 API**：模块接口、依赖方向、稳定性标注
- **GN 构建系统**：Targets 梳理、依赖关系、编译产物
- **安全风险评审**：攻击面分析、信任边界、修复建议

### 未涵盖内容

- 测试代码（test/、tests/、*_test.* 等）
- 详细的实现算法（超出工程文档范围）
- 历史变更记录（请查阅 git log）

## 快速开始

### 新人学习路线
1. 阅读 [项目概览](01_Project_Overview.md) 了解项目定位
2. 查看 [目录结构](02_Directory_Structure.md) 熟悉模块划分
3. 学习 [架构说明](03_Architecture.md) 理解组件关系
4. 参考 [N-API 参考](04_NAPI_Reference.md) 学习对外接口

### 安全研究路线
1. 阅读 [攻击面分析](07_Security_Review.md) 识别入口点
2. 了解 [信任边界](07_Security_Review.md#信任边界) 理解安全域
3. 参考 [安全风险评估](07_Security_Review.md#可被利用点与修复建议) 了解潜在风险

## 文档质量

本文档已完成 Phase 0-7 的所有评估工作：
- ✅ Phase 0：项目评估（`ASSESSMENT.md`）
- ✅ Phase 1-6：文档创建与内容填充
- ✅ Phase 7：一致性校验（`QUALITY_ASSESSMENT.md`）

**总体评级**：B级 (良好) - 基础文档完整,代码证据充足,受众适配良好

## 更新方式

本文档随代码同步更新。当新增模块、API 或架构变更时，请同步更新相关文档。

**文档更新检查清单**：

- [ ] 新增 N-API 模块 → 更新 `04_NAPI_Reference.md`
- [ ] 修改 GN 构建配置 → 更新 `06_Build_System.md`
- [ ] 新增安全相关代码 → 更新 `07_Security_Review.md`
- [ ] 新增核心模块 → 更新 `02_Directory_Structure.md` 和 `03_Architecture.md`

## 文档生成信息

- **生成时间**：2025年2月
- **更新时间**：2026年2月7日
- **仓库版本**：基于当前代码主干
- **维护者**：ArkCompiler 团队

## 工作文件

详细的工作记录和评估结果请查看：
- **项目评估**：`_work/ASSESSMENT.md`
- **事实记录**：`_work/NOTES.md`
- **任务计划**：`_work/PLAN.md`
- **质量评估**：`_work/QUALITY_ASSESSMENT.md`

## 相关链接

- [项目源码](/README.md)
- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [ArkCompiler 架构文档](https://gitee.com/openharmony/docs/blob/master/en/readme/ARK-Runtime-Subsystem.md)
