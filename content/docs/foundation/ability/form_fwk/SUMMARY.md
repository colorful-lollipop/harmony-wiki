# Form Fmk Wiki - 文档导航

## 双路线阅读指南

本Wiki提供两条阅读路线，针对不同受众：

### 路线一：新人学习路线 ⭐

适合刚接触Form Framework的开发者，快速理解项目定位和基本使用。

**阅读顺序**：
1. [00_Overview](00_Overview.md) - 5分钟了解项目是什么
2. [01_Project_Overview](01_Project_Overview.md) - 项目定位、能力边界、核心概念
3. [02_Architecture](02_Architecture.md) - 架构设计与数据流
4. [03_NAPI_Reference](03_NAPI_Reference.md) - JS API使用方法
5. [06_Build_Artifacts](06_Build_Artifacts.md) - 编译产物与部署

**目标**：
- ✅ 5分钟内理解项目定位
- ✅ 15分钟内找到核心代码位置
- ✅ 30分钟内理解基本架构

---

### 路线二：安全研究路线 🔒

适合安全研究员，快速识别攻击面和评估安全风险。

**阅读顺序**：
1. [00_Overview](00_Overview.md) - 快速了解项目功能
2. [05_AttackSurface](05_AttackSurface.md) - 攻击面分析（外部输入、信任边界）
3. [07_Security_Review](07_Security_Review.md) - 深度安全风险评估
4. [02_Architecture](02_Architecture.md) - 理解数据流和组件关系
5. [04_Inner_API](04_Inner_API.md) - IPC接口详情

**目标**：
- ✅ 快速识别所有外部输入入口
- ✅ 定位敏感操作和权限检查点
- ✅ 每个风险都有可利用性评估和修复建议

---

## 完整文档索引

### 核心文档

| 文档 | 描述 |
|------|------|
| [README](README.md) | Wiki 说明与更新方式 |
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_Project_Overview](01_Project_Overview.md) | 项目定位、边界、关键概念 |
| [02_Architecture](02_Architecture.md) | 架构图、数据流、线程模型 |
| [03_NAPI_Reference](03_NAPI_Reference.md) | N-API 接口详情 |
| [04_Inner_API](04_Inner_API.md) | Inner API 接口详情 |
| [05_GN_Targets](05_GN_Targets.md) | GN 构建 Targets |
| [06_Build_Artifacts](06_Build_Artifacts.md) | 编译产物清单 |
| [07_Security_Review](07_Security_Review.md) | 安全风险分析 |

### 附录

| 文档 | 描述 |
|------|------|
| [Callgraphs](appendix/Callgraphs.md) | 关键调用链 |
| [Config_Flags](appendix/Config_Flags.md) | 关键宏与 Feature Flags |

## 快速跳转

### 按功能模块

- **卡片提供方** → `03_NAPI_Reference.md` (FormProvider)
- **卡片使用方** → `03_NAPI_Reference.md` (FormHost)
- **卡片管理服务** → `02_Architecture.md` / `04_Inner_API.md`
- **卡片渲染** → `02_Architecture.md` (FormRenderService)

### 按接口类型

- **JS API** → `03_NAPI_Reference.md`
- **Native SDK** → `04_Inner_API.md`
- **IPC 接口** → `04_Inner_API.md` (Inner APIs)
