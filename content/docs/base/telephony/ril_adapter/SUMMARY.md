# 文档导航

> RIL Adapter Wiki 全站导航，包含新人学习路线和安全研究路线

---

## 新人学习路线 🔰

> 适合：初次接触 RIL Adapter，需要快速理解项目全貌

### 阶段一：建立认知（15 分钟）

| 顺序 | 文档 | 预计时间 | 核心收获 |
|------|------|----------|----------|
| 1 | [项目概览](01_Overview.md) | 5 分钟 | 理解项目定位、能力边界、依赖关系 |
| 2 | [架构与数据流](02_Architecture.md) | 10 分钟 | 掌握模块划分、数据流动方向 |

### 阶段二：深入代码（30 分钟）

| 顺序 | 文档 | 预计时间 | 核心收获 |
|------|------|----------|----------|
| 3 | [代码地图](03_CodeMap.md) | 10 分钟 | 能够快速定位核心代码文件 |
| 4 | [接口文档](04_Interface.md) | 20 分钟 | 理解 HDF 接口契约和请求流程 |

### 阶段三：工程实践（可选）

| 顺序 | 文档 | 预计时间 | 核心收获 |
|------|------|----------|----------|
| 5 | [构建配置](07_Build.md) | 15 分钟 | 理解构建流程和产物 |

---

## 安全研究路线 🔒

> 适合：安全研究员，需要评估攻击面和潜在风险

### 阶段一：攻击面识别（15 分钟）

| 顺序 | 文档 | 预计时间 | 核心收获 |
|------|------|----------|----------|
| 1 | [攻击面分析](05_AttackSurface.md) | 15 分钟 | 识别所有外部输入点、信任边界 |

### 阶段二：深度分析（45 分钟）

| 顺序 | 文档 | 预计时间 | 核心收获 |
|------|------|----------|----------|
| 2 | [架构与数据流](02_Architecture.md) | 15 分钟 | 理解数据流动中的安全检查点 |
| 3 | [安全风险评估](06_SecurityReview.md) | 30 分钟 | 了解具体漏洞点和利用路径 |

### 阶段三：接口审计（可选）

| 顺序 | 文档 | 预计时间 | 核心收获 |
|------|------|----------|----------|
| 4 | [接口文档](04_Interface.md) | 20 分钟 | 深入理解 IPC 接口安全性 |

---

## 完整文档索引

---

## 按主题导航

### 架构设计
- [03_Architecture - 组件图与数据流](03_Architecture.md#组件图)
- [03_Architecture - 线程模型](03_Architecture.md#线程模型)
- [03_Architecture - 关键时序](03_Architecture.md#关键时序)

### API 文档
- [04_Internal_API - HRilOps 接口](04_Internal_API.md#hrilops-接口)
- [04_Internal_API - HRilReport 回调](04_Internal_API.md#hrilreport-回调)
- [04_Internal_API - 请求与通知](04_Internal_API.md#请求与通知)

### 构建系统
- [05_GN_Targets - Targets 清单](05_GN_Targets.md#targets-清单)
- [05_GN_Targets - 依赖关系](05_GN_Targets.md#依赖关系)
- [06_Build_Artifacts - 产物清单](06_Build_Artifacts.md#产物清单)

### 安全
- [07_Security_Review - 攻击面](07_Security_Review.md#攻击面清单)
- [07_Security_Review - 信任边界](07_Security_Review.md#信任边界)
- [07_Security_Review - 可被利用点](07_Security_Review.md#可被利用点)

---

## 常见问题快速链接

| 问题 | 参考文档 |
|------|---------|
| 如何集成新的厂商库？ | [04_Internal_API - 厂商库接口](04_Internal_API.md#厂商库接口) |
| 编译产物有哪些？ | [06_Build_Artifacts - 产物清单](06_Build_Artifacts.md#产物清单) |
| HDF 服务如何注册？ | [03_Architecture - HDF 服务注册](03_Architecture.md#hdf-服务注册) |
| 支持哪些 RIL 请求？ | [04_Internal_API - 请求类型](04_Internal_API.md#请求类型) |
| 如何调试 RIL 问题？ | [08_FAQ - 调试指南](08_FAQ.md#调试指南) |
| 有哪些安全风险？ | [07_Security_Review - 可被利用点](07_Security_Review.md#可被利用点) |

---

## 核心文档

| 文件 | 标题 | 标签 |
|------|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 | 🔰 必读 |
| [02_Architecture.md](02_Architecture.md) | 架构与数据流 | 🔰 必读 / 🔒 重要 |
| [03_CodeMap.md](03_CodeMap.md) | 代码地图 | 🔰 推荐 |
| [04_Interface.md](04_Interface.md) | 接口文档 | 🔰 参考 / 🔒 审计 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 | 🔒 必读 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 | 🔒 必读 |
| [07_Build.md](07_Build.md) | 构建配置 | 🔰 参考 |

### 工作文档

| 文件 | 描述 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据库 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

---

## 术语表

| 术语 | 全称 | 描述 |
|------|------|------|
| **RIL** | Radio Interface Layer | 无线接口层，负责与 Modem 通信 |
| **HDF** | Hardware Driver Foundation | 硬件驱动框架 |
| **HRIL** | HDC RIL / Harmony RIL | OpenHarmony RIL 实现 |
| **Vendor** | 厂商实现层 | 屏蔽不同 Modem 厂商差异的抽象层 |
| **IPC** | Inter-Process Communication | 进程间通信 |
| **AT Command** | AT 命令 | Modem 通信协议 |

---

最后更新: 2026-02-07
