# ASSET Wiki 站点导航

本页面提供 ASSET 服务文档的**双路线阅读指南**：
- 🎓 **新人学习路线** - 快速理解项目、上手开发
- 🔒 **安全研究路线** - 识别攻击面、评估风险

---

## 🎓 新人学习路线

**目标**: 5分钟理解定位 → 15分钟找到代码 → 30分钟理解架构

| 顺序 | 文档 | 重点内容 | 预计时间 |
|:----:|------|----------|----------|
| 1 | **[项目概述](00_Overview.md)** | 项目定位、核心能力、运行环境 | 5分钟 |
| 2 | **[目录结构与模块职责](01_Directory_Structure.md)** | 代码组织、模块边界 | 10分钟 |
| 3 | **[架构说明](02_Architecture.md)** | 组件图、数据流、线程模型 | 15分钟 |
| 4 | **[对外 N-API](03_NAPI_API.md)** | JS API 使用、参数、错误码 | 20分钟 |
| 5 | **[常见问题](08_QA.md)** | 集成问题、调试技巧 | 参考 |

---

## 🔒 安全研究路线

**目标**: 识别攻击面 → 定位敏感操作 → 评估可利用性

| 顺序 | 文档 | 重点内容 | 预计时间 |
|:----:|------|----------|----------|
| 1 | **[攻击面分析](05_AttackSurface.md)** | 4个入口点、信任边界、数据流 | 15分钟 |
| 2 | **[安全风险评审](07_Security_Review.md)** | 7个可利用点、修复建议 | 30分钟 |
| 3 | **[项目概述](00_Overview.md)** | 安全架构、加密体系 | 10分钟 |
| 4 | **[架构说明](02_Architecture.md)** | TEE集成、权限模型 | 15分钟 |
| 5 | **[对外 N-API](03_NAPI_API.md)** | 参数验证、权限要求 | 参考 |

---

## 快速导航（按角色）

<details>
<summary><b>📱 应用开发者</b> - 使用 ASSET API 开发应用</summary>

推荐阅读：
1. [项目概述](00_Overview.md) - 了解能做什么
2. [对外 N-API](03_NAPI_API.md) - 学习 API 用法
3. [安全风险评审](07_Security_Review.md) - 了解安全最佳实践
4. [常见问题](08_QA.md) - 解决集成问题
</details>

<details>
<summary><b>🔧 系统开发者</b> - 集成或修改 ASSET 服务</summary>

推荐阅读：
1. [项目概述](00_Overview.md) - 整体定位
2. [目录结构与模块职责](01_Directory_Structure.md) - 代码组织
3. [架构说明](02_Architecture.md) - 服务间交互
4. [内部 API](04_Inner_API.md) - 系统服务接口
5. [GN 目标梳理](05_GN_Targets.md) - 构建配置
</details>

<details>
<summary><b>🛡️ 安全研究员</b> - 审计安全风险</summary>

推荐阅读：
1. [攻击面分析](05_AttackSurface.md) - 完整攻击面清单
2. [安全风险评审](07_Security_Review.md) - 详细风险分析
3. [项目概述](00_Overview.md) - 加密体系
4. [架构说明](02_Architecture.md) - 信任边界
5. [代码证据库](_work/NOTES.md) - 代码引用汇总
</details>

<details>
<summary><b>📦 构建工程师</b> - 维护构建系统</summary>

推荐阅读：
1. [GN 目标梳理](05_GN_Targets.md) - 构建目标
2. [编译产物](06_Build_Artifacts.md) - 输出清单
3. [配置标志](appendix/Config_Flags.md) - 编译选项
4. [目录结构与模块职责](01_Directory_Structure.md) - 模块依赖
</details>

<details>
<summary><b>🏗️ 架构师/技术负责人</b> - 技术决策</summary>

推荐阅读：
1. [项目概述](00_Overview.md) - 整体设计
2. [架构说明](02_Architecture.md) - 详细架构
3. [攻击面分析](05_AttackSurface.md) - 安全设计
4. [安全风险评审](07_Security_Review.md) - 风险权衡
</details>

---

## 完整文档索引

### 核心文档

| 文档 | 描述 | 适用对象 | 路线 |
|------|------|---------|------|
| [项目概述](00_Overview.md) | 项目定位、边界、核心能力、运行环境、关键概念 | 所有人 | 🎓🔒 |
| [目录结构与模块职责](01_Directory_Structure.md) | 目录结构、各目录职责、模块边界 | 开发者 | 🎓 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型、关键时序（含 Mermaid） | 架构师、开发者 | 🎓🔒 |
| [对外 N-API](03_NAPI_API.md) | JS API 清单表、参数、返回值、同步/异步、绑定位置、错误码 | 应用开发者 | 🎓 |
| [内部 API](04_Inner_API.md) | 模块接口、依赖方向、稳定性、可替换点 | 系统开发者 | 🎓 |
| [攻击面分析](05_AttackSurface.md) | 4个入口点、敏感操作、信任边界、攻击面矩阵 | 安全工程师 | 🔒 |
| [GN 目标梳理](05_GN_Targets.md) | 关键 targets、类型、依赖、产物 | 构建工程师 | 🎓 |
| [编译产物](06_Build_Artifacts.md) | .so/.a/.hap/可执行文件、安装路径、加载关系 | 构建工程师 | 🎓 |
| [安全风险评审](07_Security_Review.md) | 攻击面、信任边界、可被利用点（含 7 条） | 安全工程师 | 🔒 |
| [常见问题](08_QA.md) | 构建/运行/调试问题与定位路径 | 所有人 | 🎓 |

### 附录

| 文档 | 描述 |
|------|------|
| [调用链示例](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [配置标志](appendix/Config_Flags.md) | 关键宏/feature flags |

---

## 工作文档

| 文档 | 描述 |
|------|------|
| [项目评估](_work/ASSESSMENT.md) | 项目类型、受众分析、文档策略 |
| [代码证据库](_work/NOTES.md) | 关键代码引用、符号位置、调用链 |

---

## 文档版本

- **生成时间**: 2026-02-06
- **源码版本**: master 分支（最新）
- **适用系统**: OpenHarmony 4.1+

---

[← 返回 Wiki 根目录](README.md)
