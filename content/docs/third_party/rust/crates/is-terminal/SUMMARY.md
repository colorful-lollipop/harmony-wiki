# 阅读路线建议

本文档提供针对不同读者的阅读路线，帮助你快速找到需要的信息。

---

## 阅读者分类

### A. 快速了解者（推荐阅读顺序）

**目标**: 5 分钟内了解 is-terminal 在 OH 中的基本情况

**阅读路径**:
1. `README.md` → 快速概览章节
2. `_work/ASSESSMENT.md` → 0.5 总结章节

---

### B. 架构师/技术决策者

**目标**: 理解库在 OH 生态中的定位、依赖关系、升级风险

**阅读路径**:
1. `README.md` → 快速概览、OH 适配概述
2. `01_Overview.md` → 在 OH 中的作用和定位
3. `04_Usage_in_OH.md` → 依赖关系图、典型使用场景
4. `_work/ASSESSMENT.md` → 0.5 总结（风险评估、升级建议）

---

### C. 构建系统开发者

**目标**: 了解 OH 构建适配细节，如何升级或修改构建配置

**阅读路径**:
1. `README.md` → OH 适配概述
2. `03_Build_Integration.md` → BUILD.gn 结构、与上游差异
3. `_work/ASSESSMENT.md` → 0.4 特殊适配识别

---

### D. 库维护者

**目标**: 了解代码状态、Patch 情况、升级流程

**阅读路径**:
1. `README.md` → OH 适配概述
2. `02_Patches.md` → Patch 清单（确认无 Patch）
3. `_work/ASSESSMENT.md` → 0.2 Patch 分析、0.5 总结（升级建议）
4. `03_Build_Integration.md` → BUILD.gn 关键配置

---

### E. 使用该库的 OH 开发者

**目标**: 了解如何在项目中使用 is-terminal

**阅读路径**:
1. `README.md` → 快速开始章节
2. `01_Overview.md` → 库功能简介
3. `04_Usage_in_OH.md` → 典型使用场景

---

## 文档深度指南

| 文档 | 深度 | 适合读者 |
|------|------|---------|
| `README.md` | 浅 | 所有人 |
| `SUMMARY.md` | 浅 | 所有人 |
| `01_Overview.md` | 中 | 开发者、架构师 |
| `02_Patches.md` | 深 | 维护者 |
| `03_Build_Integration.md` | 深 | 构建系统开发者 |
| `04_Usage_in_OH.md` | 中 | 架构师、开发者 |
| `_work/ASSESSMENT.md` | 深 | 所有人（技术细节） |

---

## 常见问题快速查找

### Q: 这个库有 OH 特有的修改吗？
**A**: 无。查看 `02_Patches.md` 或 `_work/ASSESSMENT.md` 第 0.2 节。

### Q: 哪些 OH 模块使用了这个库？
**A**: 查看 `04_Usage_in_OH.md` 或 `_work/ASSESSMENT.md` 第 0.3 节。

### Q: 如何升级到新版本？
**A**: 查看 `_work/ASSESSMENT.md` 第 0.5 节（升级建议）。

### Q: BUILD.gn 的配置有什么特别之处？
**A**: 查看 `03_Build_Integration.md`。

### Q: 这个库在 OH 中的作用是什么？
**A**: 查看 `README.md` → 快速概览，或 `01_Overview.md`。

---

## 文档维护说明

本文档根据 `_work/ASSESSMENT.md` 中的 Phase 0 评估结果生成。

如果发现文档与实际代码不符，请：
1. 检查 `ASSESSMENT.md` 是否为最新版本
2. 重新运行 Phase 0 信息收集流程
3. 更新相应文档
