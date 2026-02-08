# 阅读路线建议

本文档为不同角色的读者提供针对性的阅读路线建议。

---

## 目录

- [快速概览](#快速概览)
- [开发者路线](#开发者路线)
- [维护者路线](#维护者路线)
- [安全审计路线](#安全审计路线)
- [完整阅读顺序](#完整阅读顺序)

---

## 快速概览

### "我只想知道这个库是做什么的"

**预计阅读时间**: 2 分钟

1. 📖 [README.md](./README.md) - 库概览（必读）
2. 📖 [01_Overview.md](./01_Overview.md) - 原始库简介

### "这个库有没有 OH 特有的修改？"

**预计阅读时间**: 3 分钟

1. 📖 [README.md](./README.md) - 库概览
2. 📖 [02_Patches.md](./02_Patches.md) - Patch 分析（核心文档）

---

## 开发者路线

### "我要在 OH 中使用这个库"

**预计阅读时间**: 10 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | README.md | 全部 | 快速了解库的作用 |
| 2 | 01_Overview.md | 全部 | 理解原始功能 |
| 3 | 04_Usage_in_OH.md | 使用方式 | 依赖添加方法 |
| 3 | 04_Usage_in_OH.md | 直接依赖者 | 查看使用示例 |

### "我要修改这个库的构建配置"

**预计阅读时间**: 15 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | 03_Build_Integration.md | 全部 | BUILD.gn 结构 |
| 2 | 03_Build_Integration.md | 关键编译选项 | defines 和 flags |
| 3 | 03_Build_Integration.md | 与上游差异 | 对比 CMake/Bazel |

### "我要升级这个库的版本"

**预计阅读时间**: 20 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | 02_Patches.md | 升级建议 | OH 特有修改 |
| 2 | 03_Build_Integration.md | 与上游差异 | 构建配置差异 |
| 3 | 04_Usage_in_OH.md | 依赖关系 | 依赖者兼容性 |

---

## 维护者路线

### "我要维护这个库的 OH 适配"

**预计阅读时间**: 30 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | README.md | 全部 | 整体认知 |
| 2 | 01_Overview.md | 全部 | 原始功能 |
| 3 | 02_Patches.md | 全部 | 无 Patch 结论 |
| 4 | 03_Build_Integration.md | 全部 | 构建适配细节 |
| 5 | 04_Usage_in_OH.md | 全部 | 依赖关系 |

### "我要排查构建问题"

**预计阅读时间**: 15 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | 03_Build_Integration.md | 构建目标详解 | 目标依赖关系 |
| 2 | 03_Build_Integration.md | 关键编译选项 | 编译器 flags |
| 3 | ASSESSMENT.md | 特殊适配识别 | 配置检查清单 |

### "我要添加新的依赖者"

**预计阅读时间**: 10 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | 04_Usage_in_OH.md | 直接依赖者 | 现有依赖者列表 |
| 2 | 04_Usage_in_OH.md | 依赖关系图 | 集成位置 |
| 3 | 03_Build_Integration.md | 构建目标详解 | 目标选择 |

---

## 安全审计路线

### "我要审计这个库的安全性"

**预计阅读时间**: 25 分钟

**推荐阅读顺序**:

| 顺序 | 文档 | 章节 | 重点 |
|------|------|------|------|
| 1 | 06_Security.md | 全部 | CVE 和修复状态 |
| 2 | ASSESSMENT.md | 0.4 OH 使用情况 | 使用场景 |
| 3 | 03_Build_Integration.md | 编译配置 | 安全相关 flags |

---

## 完整阅读顺序

### 按文档编号顺序（推荐顺序）

```
1. README.md          → 库概览和导航
2. SUMMARY.md         → 本文档，阅读路线
3. 01_Overview.md     → 原始库介绍
4. 02_Patches.md      → Patch 分析（重要）
5. 03_Build_Integration.md → 构建适配（重要）
6. 04_Usage_in_OH.md  → 依赖和使用（重要）
7. 05_API_Differences.md → API 差异（如有）
8. 06_Security.md     → 安全分析
```

### 按任务类型分类

#### 新增依赖者
```
README.md → 04_Usage_in_OH.md → 03_Build_Integration.md
```

#### 版本升级
```
02_Patches.md → 03_Build_Integration.md → 04_Usage_in_OH.md
```

#### 问题排查
```
03_Build_Integration.md → ASSESSMENT.md → 相关代码
```

#### 安全审计
```
06_Security.md → 02_Patches.md → 依赖关系分析
```

---

## 文档速查表

### 核心文档（必读）

| 文档 | 何时阅读 | 重要性 |
|------|---------|--------|
| README.md | 首次接触 | ⭐⭐⭐ |
| 02_Patches.md | 了解 OH 差异 | ⭐⭐⭐ |
| 03_Build_Integration.md | 构建相关 | ⭐⭐⭐ |
| 04_Usage_in_OH.md | 集成相关 | ⭐⭐⭐ |

### 辅助文档（按需）

| 文档 | 何时阅读 | 重要性 |
|------|---------|--------|
| 01_Overview.md | 了解原始功能 | ⭐⭐ |
| 05_API_Differences.md | API 变更时 | ⭐⭐ |
| 06_Security.md | 安全审计时 | ⭐⭐ |
| SUMMARY.md | 选择阅读路线时 | ⭐ |

### 工作文档（内部使用）

| 文档 | 用途 |
|------|------|
| _work/ASSESSMENT.md | 项目评估记录 |
| _work/NOTES.md | 分析过程记录 |
| _work/PLAN.md | 任务进度跟踪 |

---

## 常见问题

### Q: 我应该从哪个文档开始？

**A**: 从 [README.md](./README.md) 开始，它提供了库的整体介绍和文档导航。

### Q: 这个库有没有 OH 特有的修改？

**A**: 请阅读 [02_Patches.md](./02_Patches.md)，该库采用无 Patch 适配方式。

### Q: 如何在 OH 中使用这个库？

**A**: 请阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 的"使用方式"章节。

### Q: 如何排查构建问题？

**A**: 请阅读 [03_Build_Integration.md](./03_Build_Integration.md) 的"关键编译选项"章节。

---

## 反馈与贡献

如有文档问题或建议，请联系维护者：

- **维护者**: zhangleiyu1@huawei.com
- **组件所有者**: zhangleiyu1@huawei.com

---

*最后更新: 2026-02-07*
