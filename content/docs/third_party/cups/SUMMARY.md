# CUPS 文档阅读路线建议

## 针对不同角色的阅读建议

### 开发者 (需要集成 CUPS)

1. **[01_Overview.md](01_Overview.md)** - 5 分钟
   - 了解 CUPS 在 OH 中的作用
   - 明确集成点

2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 10 分钟
   - 查看依赖关系图
   - 了解如何使用 CUPS API
   - 查找相关模块的 BUILD.gn 示例

3. **[03_Build_Integration.md](03_Build_Integration.md)** - 10 分钟
   - 了解编译配置
   - 查看外部依赖

### 安全工程师

1. **[06_Security.md](06_Security.md)** - 15 分钟
   - 查看所有 CVE 修复记录
   - 评估安全风险
   - 制定升级策略

2. **[02_Patches.md](02_Patches.md)** - 重点关注安全相关 Patch
   - `cups-log-datamasking.patch`
   - `ohos-ipp-authenticate.patch`
   - 所有 `backport-CVE-*.patch`

### 维护者 (需要升级 CUPS)

1. **[02_Patches.md](02_Patches.md)** - 30 分钟
   - **必须阅读**
   - 记录所有 OH 特有 Patch
   - 标注可推向上游的 Patch
   - 标注 OH 专用 Patch

2. **[03_Build_Integration.md](03_Build_Integration.md)** - 15 分钟
   - 了解构建系统差异
   - 记录特殊编译选项

3. **升级检查清单**：
   - [ ] 所有 CVE Patch 是否已合并到新版本
   - [ ] OH 特有 Patch 是否需要重新适配
   - [ ] BUILD.gn 配置是否兼容
   - [ ] 依赖版本是否变化

### 测试工程师

1. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 理解测试范围
2. **[01_Overview.md](01_Overview.md)** - 了解测试场景

---

## 文档结构说明

```
wiki/
├── README.md              # 入口文档、导航
├── SUMMARY.md             # 本文档 - 阅读建议
├── 01_Overview.md         # 库概览
├── 02_Patches.md          # Patch 详解 (核心)
├── 03_Build_Integration.md # 构建适配
├── 04_Usage_in_OH.md      # OH 使用情况
├── 05_API_Differences.md  # API 差异 (如需)
├── 06_Security.md         # 安全分析
└── _work/
    ├── ASSESSMENT.md     # 评估结果
    ├── NOTES.md           # 分析记录
    └── PLAN.md           # 任务计划
```

---

## 快速定位

| 需求 | 跳转 |
|-----|------|
| CUPS 在 OH 中做什么？ | 01_Overview.md |
| OH 修改了哪些代码？ | 02_Patches.md |
| 如何编译 CUPS？ | 03_Build_Integration.md |
| 谁在使用 CUPS？ | 04_Usage_in_OH.md |
| 有什么安全漏洞？ | 06_Security.md |
| 如何升级 CUPS 版本？ | 02_Patches.md (升级建议章节) |
