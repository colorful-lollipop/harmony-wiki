# TEE Client Wiki 导航

## 新人阅读路线

建议阅读顺序：

1. [README](README.md) - 文档说明与覆盖范围
2. [00_Overview](00_Overview.md) - 项目定位与核心概念
3. [01_Architecture](01_Architecture.md) - 整体架构与模块职责
4. [03_CodeMap](03_CodeMap.md) - 目录结构与代码地图
5. [02_API_Reference](02_API_Reference.md) - API 接口规范
6. [04_Interface](04_Interface.md) - IPC 接口与配置文件
7. [03_Build_System](03_Build_System.md) - 构建系统与编译产物
8. [08_Internals](08_Internals.md) - 内部实现细节
9. [05_Troubleshooting](05_Troubleshooting.md) - 故障排查指南

## 安全研究路线

安全研究员建议阅读顺序：

1. [README](README.md) - 文档说明
2. [00_Overview](00_Overview.md) - 项目概述
3. [01_Architecture](01_Architecture.md) - 架构理解
4. [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
5. [04_Security_Review](04_Security_Review.md) - 安全风险评审
6. [04_Interface](04_Interface.md) - 接口文档（输入点）
7. [03_CodeMap](03_CodeMap.md) - 代码地图（快速定位）
8. [08_Internals](08_Internals.md) - 内部实现（深入分析）

## 文档索引

### 快速入门

| 文档 | 说明 |
|------|------|
| [README](README.md) | 文档说明与更新方式 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、运行环境 |

### 架构与设计

| 文档 | 说明 |
|------|------|
| [01_Architecture](01_Architecture.md) | 组件架构、数据流、线程模型 |
| [03_CodeMap](03_CodeMap.md) | 目录结构、核心文件定位、代码导航 |

### API 参考

| 文档 | 说明 |
|------|------|
| [02_API_Reference](02_API_Reference.md) | TEEC_* API 详解、错误码 |
| [04_Interface](04_Interface.md) | IPC 接口、数据类型、配置文件 |

### 构建与部署

| 文档 | 说明 |
|------|------|
| [03_Build_System](03_Build_System.md) | GN Targets、编译产物、安装路径 |

### 安全与运维

| 文档 | 说明 |
|------|------|
| [05_AttackSurface](05_AttackSurface.md) | 攻击面、外部输入、敏感操作、信任边界 |
| [04_Security_Review](04_Security_Review.md) | 安全风险评审、修复建议 |
| [05_Troubleshooting](05_Troubleshooting.md) | 常见问题、调试方法 |

### 工程实现

| 文档 | 说明 |
|------|------|
| [08_Internals](08_Internals.md) | 核心类职责、内部 API、资源生命周期 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 关键编译宏与 Feature Flags |

### 工作区文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES](_work/NOTES.md) | 事实记录（代码证据汇总） |
| [_work/PLAN](_work/PLAN.md) | 任务进度追踪 |

## 相关链接

- **代码仓库**：当前仓库 base/tee/tee_client
- **驱动仓库**：[tee_tzdriver](https://gitee.com/openharmony-sig/tee_tee_tzdriver)
- **GP 标准**：[TEE Client API Specification v1.0](https://globalplatform.org/specs-library/?filter-committee=tee)
