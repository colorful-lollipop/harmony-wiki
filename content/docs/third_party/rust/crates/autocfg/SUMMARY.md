# SUMMARY - 阅读路线建议

## 快速了解（5 分钟）

如果你只想快速了解 autocfg 在 OpenHarmony 中的情况：

1. **[README.md](./README.md)** - 阅读「快速概览」和「关键信息速查」部分
2. **[02_Patches.md](./02_Patches.md)** - 确认 Patch 状态（本库无 Patch）

## 标准了解（15 分钟）

如果你需要全面了解该库：

1. **[01_Overview.md](./01_Overview.md)** - 了解原始库功能和 OH 定位
2. **[02_Patches.md](./02_Patches.md)** - 了解为什么不需要 Patch
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 了解 BUILD.gn 适配
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 了解依赖关系和使用场景

## 深度分析（30 分钟）

如果你需要进行技术评估或维护工作：

1. **必读文档**（同上）
2. **[06_Security.md](./06_Security.md)** - 安全风险评估
3. **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 完整的项目评估报告
4. **[_work/NOTES.md](./_work/NOTES.md)** - 分析过程记录（含搜索过程）

---

## 按角色阅读

### 开发者（使用 autocfg）

推荐阅读：
- [01_Overview.md](./01_Overview.md) - 了解基本用法
- [03_Build_Integration.md](./03_Build_Integration.md) - 了解 BUILD.gn 如何引用

### 维护者（升级/维护 autocfg）

推荐阅读：
- [README.md](./README.md) - 快速概览
- [02_Patches.md](./02_Patches.md) - Patch 状态
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解影响范围
- **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 完整评估

### 审计者（代码审计/安全检查）

推荐阅读：
- [02_Patches.md](./02_Patches.md) - 变更分析
- [06_Security.md](./06_Security.md) - 安全分析
- **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 风险评估章节

### 架构师（技术选型/依赖评估）

推荐阅读：
- [01_Overview.md](./01_Overview.md) - 技术定位
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖图和影响范围
- **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 完整评估报告

---

## 文档依赖关系

```
README.md (入口)
    ├── 01_Overview.md
    ├── 02_Patches.md
    ├── 03_Build_Integration.md
    ├── 04_Usage_in_OH.md
    ├── 05_API_Differences.md (可选)
    └── 06_Security.md (可选)

_work/ (工作文档，辅助参考)
    ├── ASSESSMENT.md
    ├── NOTES.md
    └── PLAN.md
```

---

## 关键结论速览

| 问题 | 答案 |
|------|------|
| 有 Patch 吗？ | **无** - 零 Patch 库 |
| 需要特殊适配吗？ | **不需要** - 标准 BUILD.gn 即可 |
| 谁在使用？ | **memoffset** 等 build.rs |
| 进入运行时吗？ | **不** - 纯构建时工具 |
| 升级有风险吗？ | **低** - semver 兼容，无 Patch 负担 |

---

*阅读路线建议由 OpenHarmony Wiki Agent 生成于 2025-02-08*
