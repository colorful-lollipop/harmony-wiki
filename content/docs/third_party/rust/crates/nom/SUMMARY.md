# 阅读路线建议

> nom 库 OpenHarmony Wiki - 快速导航指南

## 选择你的阅读路径

根据你的角色和需求，选择最适合的阅读路径：

---

## 🎯 开发者 (我要使用 nom)

如果你是想在 OpenHarmony Rust 项目中使用 nom 的开发者：

| 顺序 | 文档 | 预计时间 | 内容 |
|------|------|---------|------|
| 1 | [README.md](./README.md) | 2分钟 | 快速概览和关键发现 |
| 2 | [01_Overview.md](./01_Overview.md) | 5分钟 | nom 核心功能介绍 |
| 3 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 3分钟 | OH 中的使用方式和依赖关系 |
| 4 | 上游文档 | - | [nom docs.rs](https://docs.rs/nom) |

**快速跳转**: 直接查看 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 获取使用示例

---

## 🔧 维护者 (我要维护 nom)

如果你是负责 nom 库在 OH 中集成的维护者：

| 顺序 | 文档 | 预计时间 | 内容 |
|------|------|---------|------|
| 1 | [README.md](./README.md) | 2分钟 | 项目概览 |
| 2 | [02_Patches.md](./02_Patches.md) | 5分钟 | Patch 分析 (核心) |
| 3 | [03_Build_Integration.md](./03_Build_Integration.md) | 5分钟 | 构建配置详解 |
| 4 | [ASSESSMENT.md](./_work/ASSESSMENT.md) | 10分钟 | 完整评估报告 |

**核心文档**: [02_Patches.md](./02_Patches.md) - 了解 nom 是否需要 Patch

---

## 📊 系统架构师 (我要了解依赖关系)

如果你关心 nom 在 OH 生态系统中的位置：

| 顺序 | 文档 | 预计时间 | 内容 |
|------|------|---------|------|
| 1 | [README.md](./README.md) | 2分钟 | 概览 |
| 2 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 5分钟 | 依赖关系图 |
| 3 | [ASSESSMENT.md](./_work/ASSESSMENT.md) | 10分钟 | 完整依赖分析 |

**关键信息**: nom 目前只有 1 个直接依赖者 (rust-cexpr)

---

## 🔒 安全审计员 (我要检查安全风险)

如果你负责安全审计：

| 顺序 | 文档 | 预计时间 | 内容 |
|------|------|---------|------|
| 1 | [README.md](./README.md) | 2分钟 | 安全相关摘要 |
| 2 | [02_Patches.md](./02_Patches.md) | 5分钟 | Patch 安全检查 |
| 3 | [ASSESSMENT.md](./_work/ASSESSMENT.md) | 10分钟 | 安全风险评估章节 |

---

## 📝 文档结构总览

```
wiki/
├── README.md              # 项目概览 (必读)
├── SUMMARY.md             # 阅读路线 (本文档)
├── 01_Overview.md         # 原始库简介
├── 02_Patches.md          # Patch 分析 (核心)
├── 03_Build_Integration.md # 构建适配
├── 04_Usage_in_OH.md     # 依赖与使用
└── _work/
    ├── ASSESSMENT.md     # 完整评估报告
    ├── NOTES.md          # 分析过程记录
    └── PLAN.md           # 任务进度跟踪
```

---

## 快速参考表

### 关键指标速查

| 问题 | 答案 |
|------|------|
| **nom 需要 Patch 吗？** | ❌ 不需要 |
| **集成复杂度如何？** | ⭐☆☆☆☆ 极低 |
| **谁在使用 nom？** | rust-cexpr |
| **nom 提供什么功能？** | 解析器组合子 |
| **上游同步状态？** | ✅ 完全同步 |

### 关键文件位置

| 文件 | 路径 |
|------|------|
| OH 组件配置 | `bundle.json` |
| 构建配置 | `BUILD.gn` |
| 上游 Cargo 配置 | `Cargo.toml` |
| Wiki 根目录 | `wiki/` |

---

## 推荐阅读时间

| 场景 | 推荐阅读 | 时长 |
|------|---------|------|
| **首次了解 nom** | README + Overview | 10分钟 |
| **准备升级版本** | Patches + Build | 15分钟 |
| **解决依赖问题** | Usage + Assessment | 20分钟 |
| **完整审计** | 所有文档 | 45分钟 |

---

## 下一步行动

根据你的需求，点击对应的链接开始：

- [我要使用 nom](./04_Usage_in_OH.md)
- [我要维护 nom](./02_Patches.md)
- [我要了解架构](./04_Usage_in_OH.md)
- [查看完整文档列表](#文档结构总览)

---

**最后更新**: 2024年
