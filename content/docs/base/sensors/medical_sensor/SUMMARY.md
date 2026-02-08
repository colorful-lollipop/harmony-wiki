# Medical_Sensor Wiki 目录

> 本页面提供完整的 Wiki 导航和推荐阅读路径。
> 
> **更新时间**: 2026-02-07
> **适用版本**: 3.1

---

## 🎯 双路线导航

根据您的角色选择适合的阅读路线：

### 📚 新人学习路线（推荐 3 小时）

适合：应用开发者、系统集成工程师、新手安全研究员

| 顺序 | 文档 | 预估时间 | 重点内容 |
|------|------|---------|---------|
| 1 | **[项目概览](00_Overview.md)** | 15 分钟 | 项目定位、核心概念、快速开始 |
| 2 | **[目录结构](01_Directory_Structure.md)** | 20 分钟 | 代码组织、模块职责、文件索引 |
| 3 | **[架构说明](02_Architecture.md)** | 40 分钟 | 组件图、数据流、线程模型、时序图 |
| 4 | **[N-API 参考](03_N-API_Reference.md)** | 30 分钟 | JS API 使用、参数说明、代码示例 |
| 5 | **[内部 API](05_Internal_API.md)** | 30 分钟 | 模块接口、依赖关系、稳定性边界 |
| 6 | **[编译产物](07_Build_Artifacts.md)** | 15 分钟 | 输出文件、安装路径、运行时加载 |
| 7 | **[常见问题](09_Troubleshooting.md)** | 30 分钟 | 构建、运行、调试问题排查 |

**学习建议**：
- 先通读项目概览建立整体认知
- 结合架构说明理解数据流向
- 参考 N-API 文档进行实际开发
- 遇到问题时查阅 Troubleshooting

---

### 🔐 安全研究路线（推荐 4 小时）

适合：安全审计人员、渗透测试工程师、漏洞研究员

| 顺序 | 文档 | 预估时间 | 重点内容 |
|------|------|---------|---------|
| 1 | **[项目概览](00_Overview.md)** | 15 分钟 | 系统边界、信任域、关键概念 |
| 2 | **[攻击面分析](04_AttackSurface.md)** | 45 分钟 | 外部输入清单、敏感操作、攻击路径 |
| 3 | **[架构说明](02_Architecture.md)** | 30 分钟 | 信任边界、数据流、IPC 机制 |
| 4 | **[安全风险评估](08_Security_Review.md)** | 60 分钟 | 漏洞分析、利用路径、修复建议 |
| 5 | **[N-API 参考](03_N-API_Reference.md)** | 20 分钟 | 参数验证、权限检查点 |
| 6 | **[内部 API](05_Internal_API.md)** | 25 分钟 | IPC 接口、权限验证、数据通道 |
| 7 | **[GN Targets](06_GN_Targets.md)** | 15 分钟 | 构建目标、依赖关系、产物分析 |

**研究建议**：
- 重点阅读攻击面分析和安全风险评估
- 结合架构图理解信任边界跨越点
- 关注代码证据中的文件路径和行号
- 使用背景任务结果进行深入分析

---

## 📑 按主题导航

### 快速入门
- [项目概览](00_Overview.md) - 项目定位、核心能力、运行环境
- [目录结构](01_Directory_Structure.md) - 代码组织详解
- [常见问题](09_Troubleshooting.md) - 快速排查问题

### 架构与设计
- [项目概览](00_Overview.md) - 项目定位和边界
- [目录结构](01_Directory_Structure.md) - 模块职责
- [架构说明](02_Architecture.md) - 组件图、数据流、线程模型

### 接口文档
- [N-API 参考](03_N-API_Reference.md) - JS API 完整文档
- [内部 API](05_Internal_API.md) - 模块间接口契约

### 安全研究
- [攻击面分析](04_AttackSurface.md) - 攻击面和输入入口
- [安全风险评估](08_Security_Review.md) - 漏洞分析和修复建议

### 构建与部署
- [GN Targets](06_GN_Targets.md) - 构建目标和依赖图
- [编译产物](07_Build_Artifacts.md) - 输出文件和安装路径

### 构建与部署
- [GN Targets](06_GN_Targets.md) - 构建目标和依赖图
- [编译产物](07_Build_Artifacts.md) - 输出文件和安装路径

---

## 📋 文档清单

### 核心文档

| 编号 | 文档 | 说明 | 受众 |
|------|------|------|------|
| 00 | [项目概览](00_Overview.md) | 项目定位、核心能力、运行环境 | 全部 |
| 01 | [目录结构](01_Directory_Structure.md) | 模块职责、代码导航 | 全部 |
| 02 | [架构说明](02_Architecture.md) | 组件图、数据流、时序图 | 全部 |
| 03 | [N-API 参考](03_N-API_Reference.md) | JS API 完整文档 | 开发者 |
| 04 | [攻击面分析](04_AttackSurface.md) | 攻击面、输入入口、攻击路径 | 安全研究员 |
| 05 | [内部 API](05_Internal_API.md) | 模块接口、依赖关系 | 系统开发者 |
| 06 | [GN Targets](06_GN_Targets.md) | 构建系统、依赖图 | 系统开发者 |
| 07 | [编译产物](07_Build_Artifacts.md) | 输出文件、安装路径 | 系统开发者 |
| 08 | [安全风险评估](08_Security_Review.md) | 漏洞分析、修复建议 | 安全研究员 |
| 09 | [常见问题](09_Troubleshooting.md) | 故障排查、调试技巧 | 全部 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [README.md](README.md) | Wiki 介绍和导航 |
| [SUMMARY.md](SUMMARY.md) | 本文件：全站导航 |
| `_work/ASSESSMENT.md` | 项目评估报告（Phase 0）|
| `_work/NOTES.md` | 代码证据汇总 |
| `_work/PLAN.md` | 任务进度追踪 |

---

## 🏷️ 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 医疗传感器 | Medical Sensor | 健康类传感器（PPG、ECG、心率等） |
| 传感器服务 | Sensor Service | 运行在独立进程中的核心服务 |
| 系统能力 | SystemAbility | OpenHarmony 的系统服务机制 |
| 原生接口 | Native Interface | C/C++ 层的接口 |
| JS 插件 | NAPI Module | Node.js API 模块 |
| 硬件设备接口 | HDI | Hardware Device Interface |
| 进程间通信 | IPC | Inter-Process Communication |
| 权限令牌 | AccessToken | 访问控制令牌 |
| 信任边界 | Trust Boundary | 安全域之间的边界 |
| 攻击面 | Attack Surface | 系统暴露给攻击者的入口点 |

---

## 🔗 相关资源

### 官方文档
- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [泛 Sensor 服务子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/泛Sensor子系统.md)

### 相关仓库
- [sensors_sensor](https://gitee.com/openharmony/sensors_sensor) - 传统传感器组件
- [sensors_miscdevice](https://gitee.com/openharmony/sensors_miscdevice) - 杂项设备组件

### 参考规范
- [OpenHarmony 安全开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/安全开发指南.md)
- [HDF 驱动框架文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/HDF.md)

---

## 📝 更新说明

### 2026-02-07
- ✅ 创建 ASSESSMENT.md 项目评估报告
- ✅ 新增 04_AttackSurface.md 攻击面分析文档
- ✅ 更新 SUMMARY.md 双路线导航
- ✅ 优化文档编号逻辑（04 改为攻击面，原内部 API 改为 05）

### 2026-02-06
- ✅ 初始版本发布
- ✅ 完成 9 篇核心文档
- ✅ 建立 Wiki 基础架构

---

## 💡 使用提示

1. **跳转链接**: 所有文档链接使用相对路径，可直接点击跳转
2. **代码证据**: 关键结论均标注了代码路径和行号，可快速定位
3. **Mermaid 图**: 支持 Mermaid 语法的图表，可使用插件渲染
4. **术语统一**: 专业术语首次出现时标注英文，后续使用中文
5. **受众标注**: 每篇文档开头标注适用受众，便于选择阅读

---

*本文档由 OpenHarmony Wiki Agent 自动生成，遵循证据优先原则。*
