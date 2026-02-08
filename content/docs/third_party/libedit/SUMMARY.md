# 阅读路线建议

根据你的需求选择合适的阅读路径。

---

## 路线一：快速了解（5 分钟）

### 目标
快速了解 libedit 在 OpenHarmony 中的状态和适配情况。

### 阅读顺序

1. **[wiki/README.md](README.md)** - 阅读"关键发现"和"关于 libedit"章节
2. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 阅读第 1 节（基础信息）和第 9 节（总结）

### 关键要点

- ✅ libedit 是 BSD 许可证的行编辑库
- ❌ 在 OH 中**未被实际使用**
- ✅ 唯一的 OH 修改是跨编译支持（已整合到上游）
- ❌ 无 BUILD.gn 构建配置

---

## 路线二：完整评估（15 分钟）

### 目标
深入了解 libedit 的详细评估结果和证据。

### 阅读顺序

1. **[wiki/README.md](README.md)** - 完整阅读
2. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 完整阅读（9 个章节）
3. **[02_Patches.md](02_Patches.md)** - 了解唯一的 OH Patch

### 关键章节

| 章节 | 内容 | 重要性 |
|------|------|--------|
| ASSESSMENT §2 | Patch 分析 | ⭐⭐⭐ |
| ASSESSMENT §3 | OH 使用情况 | ⭐⭐⭐ |
| ASSESSMENT §5 | 代码级适配 | ⭐⭐ |
| ASSESSMENT §9 | 总结与建议 | ⭐⭐⭐ |

---

## 路线三：维护者视角（30 分钟）

### 目标
了解如何维护 libedit、升级策略、安全考虑。

### 阅读顺序

1. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - §7 维护建议
2. **[02_Patches.md](02_Patches.md)** - 完整阅读
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建系统说明
4. **[06_Security.md](06_Security.md)** - 安全风险评估

### 关键决策点

| 决策点 | 文档位置 | 内容 |
|--------|----------|------|
| 是否需要 BUILD.gn | 03_Build_Integration.md | 构建配置方案 |
| 如何升级版本 | 02_Patches.md | 升级策略 |
| 安全审计 | 06_Security.md | CVE 跟踪 |

---

## 路线四：集成 libedit（如需）

### 目标
了解如何在 OH 中集成使用 libedit。

### 阅读顺序

1. **[01_Overview.md](01_Overview.md)** - 了解库功能
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 当前使用情况分析
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建适配方案
4. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - §7 维护建议

### 实施步骤

参考 `04_Usage_in_OH.md` 中的"集成建议"章节。

---

## 按主题查找

### 我想知道...

| 问题 | 相关文档 |
|------|----------|
| libedit 是什么？ | [01_Overview.md](01_Overview.md) |
| OH 对它做了哪些修改？ | [02_Patches.md](02_Patches.md) |
| 谁在用这个库？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| 如何编译它？ | [03_Build_Integration.md](03_Build_Integration.md) |
| 有安全漏洞吗？ | [06_Security.md](06_Security.md) |
| 为什么没用它？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| 升级版本需要注意什么？ | [02_Patches.md](02_Patches.md) |
| 废弃它会有影响吗？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |

---

## 文档优先级

### 必读（⭐⭐⭐）

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 完整评估结果
- **[01_Overview.md](01_Overview.md)** - 库概览
- **[02_Patches.md](02_Patches.md)** - 唯一的 OH 修改

### 推荐（⭐⭐）

- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 使用情况分析
- **[03_Build_Integration.md](03_Build_Integration.md)** - 构建适配

### 可选（⭐）

- **[06_Security.md](06_Security.md)** - 安全评估
- **[05_API_Differences.md](05_API_Differences.md)** - API 差异（如有）

---

## 常见问题

### Q1: 为什么 libedit 在 OH 中没有被使用？

**A**: 参考 [04_Usage_in_OH.md](04_Usage_in_OH.md) 中的"可能的原因"章节。

### Q2: libedit 的 OH 修改会引入回归风险吗？

**A**: 不会。唯一的 OH 修改已整合到上游版本。

### Q3: 我需要在 OH 中使用 libedit，该怎么办？

**A**: 参考 [04_Usage_in_OH.md](04_Usage_in_OH.md) 中的"集成建议"章节。

### Q4: libedit 可以废弃吗？

**A**: 需要确认是否有隐性依赖。参考 [04_Usage_in_OH.md](04_Usage_in_OH.md) 中的"废弃考虑"章节。

---

**文档最后更新**: 2025-02-07
