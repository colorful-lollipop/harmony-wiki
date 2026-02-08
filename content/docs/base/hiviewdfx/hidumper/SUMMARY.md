# HiDumper Wiki 导航

## 双路线阅读指南

### 新人学习路线 (推荐)

按以下顺序阅读，30 分钟内建立完整认知：

1. **[项目概览](./00_Overview.md)** (5分钟) - 了解 HiDumper 定位、模块划分、运行环境
2. **[系统架构](./01_Architecture.md)** (10分钟) - 理解组件图、数据流、线程模型
3. **[代码地图](./03_CodeMap.md)** (10分钟) - 快速定位核心代码位置
4. **[API 参考](./02_API_Reference.md)** (5分钟) - 查阅 Native API 接口
5. **[构建系统](./03_Build_System.md)** - 了解 GN targets 与编译配置
6. **[常见问题](./07_FAQ.md)** - 构建/运行/调试问题速查

### 安全研究路线 (推荐)

按以下顺序阅读，快速定位安全关注点：

1. **[攻击面分析](./05_AttackSurface.md)** - 识别所有外部输入入口 (CLI/IPC/文件/配置)
2. **[安全风险评估](./06_SecurityReview.md)** - 深度分析每个风险点的防护机制
3. **[系统架构](./01_Architecture.md)** - 理解信任边界和数据流
4. **[API 参考](./02_API_Reference.md)** - 了解 IPC 接口定义

---

## 文档索引

### 概览与架构
| 文档 | 说明 | 受众 |
|------|------|------|
| [README](./README.md) | 文档说明与双路线导航 | 全部 |
| [项目概览](./00_Overview.md) | 定位、能力边界、运行环境 | 新人 |
| [系统架构](./01_Architecture.md) | 组件图、数据流、线程模型 | 新人 |
| [代码地图](./03_CodeMap.md) | 目录结构、核心文件定位 | 新人 |

### API 与接口
| 文档 | 说明 | 受众 |
|------|------|------|
| [API 参考](./02_API_Reference.md) | Native API 清单 | 开发者 |
| - DumpManagerClient 接口 | - | - |
| - DumpUsage 接口 | - | - |
| - IPC 接口定义 | - | - |

### 安全分析
| 文档 | 说明 | 受众 |
|------|------|------|
| [攻击面分析](./05_AttackSurface.md) | 外部输入、信任边界、攻击向量 | 安全研究员 |
| [安全风险评估](./06_SecurityReview.md) | 风险点、利用路径、修复建议 | 安全研究员 |

### 构建与编译
| 文档 | 说明 | 受众 |
|------|------|------|
| [构建系统](./03_Build_System.md) | GN targets 与编译配置 | 开发者 |
| [编译产物](./04_Outputs.md) | .so/.a/可执行文件及路径 | 开发者 |

### 运维与问题
| 文档 | 说明 | 受众 |
|------|------|------|
| [常见问题](./07_FAQ.md) | 问题定位与解决方案 | 开发者 |

### 工作文档
| 文档 | 说明 |
|------|------|
| [项目评估](./_work/ASSESSMENT.md) | 项目类型、受众分析、文档策略 |
| [代码证据库](./_work/NOTES.md) | 代码路径、符号位置、证据引用 |
| [任务计划](./_work/PLAN.md) | 任务进度追踪 |

---

## 相关链接

- [HiDumper README (代码仓库)](../README_zh.md)
- [OpenHarmony DFX 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- [HiSysEvent 组件](https://gitee.com/openharmony/hiviewdfx_hilog)
- [FaultLogger 组件](https://gitee.com/openharmony/hiviewdfx_faultloggerd)
