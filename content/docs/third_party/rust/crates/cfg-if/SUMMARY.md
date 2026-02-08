# 文档目录

本文档提供 cfg-if 库在 OpenHarmony 中的集成与适配说明。

## 文档结构

```
cfg-if/
├── README.md              # 库概览、OH 适配概述、文档导航
├── SUMMARY.md             # 阅读路线建议（本文档）
├── _work/
│   ├── ASSESSMENT.md      # 项目评估结果
│   ├── NOTES.md           # 分析过程记录
│   └── PLAN.md            # 任务进度
├── 01_Overview.md         # 原始库简介
├── 02_Patches.md          # Patch 详细分析
├── 03_Build_Integration.md # OH 构建适配
├── 04_Usage_in_OH.md      # 依赖关系与使用
├── 05_API_Differences.md  # API/接口差异
└── 06_Security.md         # 安全风险分析
```

## 阅读路线

### 路线一：快速了解（5 分钟）

适合只想了解该库在 OH 中基本情况的读者：

1. **README.md** - 查看库概览和快速参考
2. **01_Overview.md** - 了解原始库功能
3. **03_Build_Integration.md** - 查看构建配置

### 路线二：深入分析（15 分钟）

适合需要全面了解该库适配细节的开发者：

1. **README.md** - 库概览
2. **01_Overview.md** - 原始功能介绍
3. **02_Patches.md** - Patch 分析（本库无 Patch）
4. **03_Build_Integration.md** - 构建集成细节
5. **04_Usage_in_OH.md** - 依赖关系和使用场景
6. **05_API_Differences.md** - API 差异（本库无差异）
7. **06_Security.md** - 安全评估

### 路线三：维护参考（按需查阅）

适合需要特定信息的维护者：

- **Patch 相关** → 02_Patches.md
- **构建配置** → 03_Build_Integration.md
- **依赖关系** → 04_Usage_in_OH.md
- **安全升级** → 06_Security.md

## 文档状态

| 文档 | 状态 | 说明 |
|------|------|------|
| README.md | ✅ 完成 | 库概览和导航 |
| SUMMARY.md | ✅ 完成 | 阅读路线建议 |
| ASSESSMENT.md | ✅ 完成 | 项目评估 |
| 01_Overview.md | ✅ 完成 | 原始库简介 |
| 02_Patches.md | ✅ 完成 | 无 Patch |
| 03_Build_Integration.md | ✅ 完成 | 构建适配 |
| 04_Usage_in_OH.md | ✅ 完成 | 依赖关系 |
| 05_API_Differences.md | ✅ 完成 | 无差异 |
| 06_Security.md | ✅ 完成 | 安全分析 |

## 符号说明

- ✅ - 已完成文档
- 🔄 - 建设中
- ⚠️ - 需要注意的内容
- 💡 - 提示信息
