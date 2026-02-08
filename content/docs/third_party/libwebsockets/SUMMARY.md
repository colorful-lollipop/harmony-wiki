# SUMMARY - 阅读路线建议

本文档提供不同角色的阅读路线建议。

---

## 路线 A：快速了解（5分钟）

适合：需要快速了解该库在 OH 中作用的人员

1. **README.md** - 了解库概览和定位
2. **ASSESSMENT.md 第 0.1 节** - 查看基础信息表
3. **ASSESSMENT.md 第 0.3 节** - 了解主要依赖者

---

## 路线 B：Patch 分析（15分钟）

适合：需要了解 OH 定制内容、进行升级评估的人员

1. **README.md** - 库概览
2. **02_Patches.md** - **核心文档**：Patch 详细分析
3. **03_Build_Integration.md** - 构建系统适配
4. **ASSESSMENT.md 第 0.2 节** - Patch 清单

---

## 路线 C：完整理解（30分钟）

适合：需要全面了解该库在 OH 中的集成情况的人员

按顺序阅读：
1. **README.md** - 库概览
2. **01_Overview.md** - 原始库简介
3. **02_Patches.md** - Patch 详细分析
4. **03_Build_Integration.md** - 构建适配
5. **04_Usage_in_OH.md** - 依赖关系与使用
6. **06_Security.md** - 安全风险（可选）

---

## 路线 D：升级评估（20分钟）

适合：计划升级上游版本的人员

1. **ASSESSMENT.md** - 完整阅读
2. **02_Patches.md** - 重点关注 Patch 升级建议
3. **03_Build_Integration.md** - 检查 BUILD.gn 兼容性
4. **04_Usage_in_OH.md** - 确认依赖关系

---

## 路线 E：安全审计（10分钟）

适合：进行安全评估的人员

1. **06_Security.md** - 安全风险分析
2. **02_Patches.md** - 检查 Patch 引入的攻击面
3. **ASSESSMENT.md 第 0.5 节** - 风险评估

---

*选择适合你的阅读路线，高效获取所需信息。*
