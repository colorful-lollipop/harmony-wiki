# either - Wiki 阅读指南

本文档提供 Wiki 的阅读路线建议。

---

## 快速了解 (5 分钟)

只想快速了解这个库？阅读以下文档：

1. **[README.md](./README.md)** - 快速概览、适配概述
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 谁在用它

---

## 深度了解 (30 分钟)

需要全面了解这个库在 OH 中的集成？按以下顺序阅读：

### Phase 1: 基础认知
1. **[01_Overview.md](./01_Overview.md)** - 原始库功能、在 OH 中的定位
2. **[03_Build_Integration.md](./03_Build_Integration.md)** - 如何构建

### Phase 2: 适配分析
3. **[02_Patches.md](./02_Patches.md)** - Patch 分析 (本库无 Patch)
4. **[05_API_Differences.md](./05_API_Differences.md)** - API 差异

### Phase 3: 使用与安全
5. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 依赖关系、使用场景
6. **[06_Security.md](./06_Security.md)** - 安全分析

---

## 特定目的阅读

### 我是开发者，想用这个库

阅读：
- [01_Overview.md](./01_Overview.md) - 了解功能
- [03_Build_Integration.md](./03_Build_Integration.md) - 如何添加依赖
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用示例

### 我是维护者，需要升级版本

阅读：
- [02_Patches.md](./02_Patches.md) - 确认无 Patch
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 评估报告

### 我是审计人员，检查安全风险

阅读：
- [06_Security.md](./06_Security.md) - 完整安全分析
- [02_Patches.md](./02_Patches.md) - 修改情况
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 风险评估

### 我想了解为什么无 Patch

阅读：
- [02_Patches.md](./02_Patches.md) - 详细说明
- [README.md](./README.md) - 适配概述
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 评估结论

---

## 文档关系图

```
                    ┌─────────────┐
                    │  README.md  │
                    │   (入口)    │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌──────────────┐ ┌──────────┐ ┌──────────────┐
    │ 01_Overview  │ │ 02_Patch │ │ 03_Build_    │
    │              │ │          │ │ Integration  │
    └──────┬───────┘ └────┬─────┘ └──────┬───────┘
           │              │              │
           └──────────────┼──────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │    04_Usage_in_OH     │
              └───────────┬───────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
       ┌──────────────┐       ┌──────────────┐
       │ 05_API_Diff  │       │ 06_Security  │
       └──────────────┘       └──────────────┘
```

---

## 附录：评估工作流

如果您需要重新评估这个库，按以下流程：

1. **Phase 0**: 信息收集 → [_work/ASSESSMENT.md](./_work/ASSESSMENT.md)
2. **Phase 1**: 基础文档 → [01_Overview.md](./01_Overview.md)
3. **Phase 2**: Patch 分析 → [02_Patches.md](./02_Patches.md)
4. **Phase 3**: 依赖分析 → [04_Usage_in_OH.md](./04_Usage_in_OH.md)
5. **Phase 4**: 文档整合 → 更新所有文档
6. **Phase 5**: 质量校验 → 验证完整性
