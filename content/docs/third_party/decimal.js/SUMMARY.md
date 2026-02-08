# SUMMARY - 阅读路线建议

本文档提供不同角色的阅读路线建议，帮助你快速找到所需信息。

---

## 阅读路线

### 路线 1: 快速了解（5 分钟）

适合人群：初次接触该库，想了解基本信息

1. [README.md](./README.md) - 快速概览
2. [01_Overview.md](./01_Overview.md) - 库简介和 OH 定位
3. [04_Usage_in_OH.md#快速使用示例](./04_Usage_in_OH.md#快速使用示例) - 代码示例

**收获**: 了解这是什么库、在 OH 中的作用、如何使用

---

### 路线 2: 应用开发者（10 分钟）

适合人群：需要在 ArkTS 应用中使用 Decimal 的开发者

1. [README.md](./README.md) - 快速概览
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 完整使用指南
   - 依赖方式
   - API 使用示例
   - 常见问题

**收获**: 学会在应用中正确使用 Decimal 进行高精度计算

---

### 路线 3: 系统开发者（20 分钟）

适合人群：需要修改构建配置或升级版本的开发者

1. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估
2. [03_Build_Integration.md](./03_Build_Integration.md) - 构建详解
3. [02_Patches.md](./02_Patches.md) - Patch 分析（了解升级影响）
4. [04_Usage_in_OH.md#依赖关系](./04_Usage_in_OH.md#依赖关系) - 了解依赖者

**收获**: 掌握构建原理、升级流程、影响范围

---

### 路线 4: 版本升级（15 分钟）

适合人群：需要升级到新版本的维护者

1. [README.md#版本状态](./README.md#版本状态) - 当前版本
2. [02_Patches.md](./02_Patches.md) - Patch 清单（本库无 Patch）
3. [03_Build_Integration.md#升级指南](./03_Build_Integration.md#升级指南) - 升级步骤
4. [06_Security.md](./06_Security.md) - 安全风险检查

**收获**: 了解升级步骤、测试重点、风险点

---

### 路线 5: 安全评估（15 分钟）

适合人群：进行安全审计或评估的工程师

1. [06_Security.md](./06_Security.md) - 安全分析
2. [02_Patches.md](./02_Patches.md) - 确认无 Patch 引入攻击面
3. [_work/ASSESSMENT.md#评估总结](./_work/ASSESSMENT.md#评估总结) - 复杂度评估

**收获**: 了解 CVE 状态、安全升级策略

---

## 文档分类

### 按角色

| 角色 | 主要阅读文档 |
|------|-------------|
| **应用开发者** | 01_Overview.md, 04_Usage_in_OH.md |
| **系统开发者** | 03_Build_Integration.md, _work/ASSESSMENT.md |
| **维护者/升级** | 02_Patches.md, 03_Build_Integration.md, 06_Security.md |
| **安全审计** | 06_Security.md, 02_Patches.md |

### 按任务

| 任务 | 阅读文档 |
|------|----------|
| **了解库是什么** | README.md, 01_Overview.md |
| **学习如何使用** | 04_Usage_in_OH.md |
| **修改构建配置** | 03_Build_Integration.md |
| **升级版本** | 02_Patches.md, 03_Build_Integration.md#升级指南, 06_Security.md |
| **分析依赖关系** | 04_Usage_in_OH.md |
| **安全评估** | 06_Security.md |

---

## 文档依赖图

```
README.md (入口)
    │
    ├── 01_Overview.md (基础)
    │       └── 04_Usage_in_OH.md (使用)
    │
    ├── 02_Patches.md (Patch 分析)
    │       └── 03_Build_Integration.md (构建)
    │
    ├── 03_Build_Integration.md (构建)
    │       └── 04_Usage_in_OH.md (使用)
    │
    ├── 04_Usage_in_OH.md (使用)
    │
    ├── 05_API_Differences.md (API)
    │
    └── 06_Security.md (安全)

_work/ (工作文档)
    ├── ASSESSMENT.md (评估报告)
    ├── NOTES.md (分析记录)
    └── PLAN.md (任务进度)
```

---

## 关键信息速查

| 问题 | 答案 | 详见 |
|------|------|------|
| 这是什么库？ | JavaScript 高精度计算库 | [01_Overview.md](./01_Overview.md) |
| 有 Patch 吗？ | **没有** | [02_Patches.md](./02_Patches.md) |
| 如何在应用中使用？ | `import { Decimal } from '@kit.ArkTS'` | [04_Usage_in_OH.md](./04_Usage_in_OH.md) |
| 如何升级版本？ | 替换源码，测试 es2abc 编译 | [03_Build_Integration.md#升级指南](./03_Build_Integration.md#升级指南) |
| 有风险吗？ | CVE 需检查，无 Patch 引入攻击面 | [06_Security.md](./06_Security.md) |

---

*选择适合你的路线，开始阅读吧！*
