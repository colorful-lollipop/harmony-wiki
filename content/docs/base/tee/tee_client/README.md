# TEE Client Wiki 文档

## 文档说明

本文档描述 OpenHarmony TEE Client 组件的架构、API、构建系统和安全风险分析。

## 覆盖范围

本文档涵盖以下内容：

### 新人学习必备
- **项目概述** ([00_Overview](00_Overview.md))：组件定位、核心能力、运行环境
- **架构说明** ([01_Architecture](01_Architecture.md))：组件图、数据流、线程模型、关键时序
- **代码地图** ([03_CodeMap](03_CodeMap.md))：目录结构、核心文件定位、代码导航
- **API 参考** ([02_API_Reference](02_API_Reference.md))：TEEC_* API 接口规范、参数校验、错误码
- **接口文档** ([04_Interface](04_Interface.md))：IPC 接口、数据类型、配置文件

### 安全研究必备
- **攻击面分析** ([05_AttackSurface](05_AttackSurface.md))：外部输入清单、敏感操作、信任边界
- **安全风险评审** ([04_Security_Review](04_Security_Review.md))：风险点、可利用性评估、修复建议

### 工程实现深度
- **构建系统** ([03_Build_System](03_Build_System.md))：GN Targets、编译产物、安装路径
- **内部实现** ([08_Internals](08_Internals.md))：核心类职责、内部 API、资源生命周期
- **故障排查** ([05_Troubleshooting](05_Troubleshooting.md))：常见构建/运行/调试问题与定位路径

## 未覆盖范围

- 测试相关内容（test/、fuzztest/、unittest/ 等）
- N-API JS/ArkTS 接口层（本项目为 Native API）
- TEE 内核驱动（tee_tzdriver）内部实现
- 具体 TA（Trusted Application）开发指南

## 文档更新方式

当代码发生以下变更时，需同步更新对应章节：

| 变更类型 | 需更新章节 |
|----------|------------|
| 新增/删除 TEEC_* API | 02_API_Reference.md, 04_Interface.md |
| 修改 API 参数或返回值 | 02_API_Reference.md, 04_Interface.md |
| 新增/修改 GN target | 03_Build_System.md |
| 新增服务或模块 | 01_Architecture.md, 03_CodeMap.md |
| 修改目录结构 | 03_CodeMap.md |
| 安全相关修改 | 05_AttackSurface.md, 04_Security_Review.md |
| 修改内部数据结构 | 08_Internals.md |

## 参考资料

- GlobalPlatform TEE Client API Specification v1.0 (GPD_SPE_007)
- OpenHarmony TEE 子系统架构设计
- 相关仓库：[tee_tzdriver](https://gitee.com/openharmony-sig/tee_tee_tzdriver)

## 生成信息

- 生成时间：2026-02-06
- 代码版本：基于当前仓库 HEAD
- 文档语言：中文（简体）
