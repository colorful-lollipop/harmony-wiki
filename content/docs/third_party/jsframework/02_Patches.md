# 02_Patches - Patch 详细分析

## 概述

**本库无传统 Patch 文件**。

与大多数 OpenHarmony 第三方库采用 `.patch` 文件的方式不同，jsframework 采用**独立源码分支**的方式进行 OH 适配，所有适配代码直接包含在 `runtime/` 目录中。

---

## Patch 分析结论

### 搜索结果

```bash
# 搜索命令
find . -name "*.patch" -o -name "patches" -type d
# 结果: 无匹配项
```

### 适配方式对比

| 方式             | 说明                         | 优缺点                                   |
| ---------------- | ---------------------------- | ---------------------------------------- |
| **传统 Patch**   | 在上游源码上应用 .patch 文件 | ✅ 易于追踪差异<br>❌ 可能产生冲突       |
| **独立源码分支** | 完全独立的 OH 适配代码       | ✅ 无冲突风险<br>❌ 升级上游时需手动合并 |

---

## 适配策略说明

### 为什么选择独立源码分支

1. **代码量较大**: jsframework 包含完整的 TypeScript 源码和构建配置，使用 Patch 难以管理
2. **深度定制**: OH 适配涉及大量新增模块和组件
3. **构建差异**: 使用 GN 构建系统，与上游 Webpack/Rollup 完全不同

### 与上游 Weex 的差异

由于采用独立源码分支，本库与上游 Weex 0.30.0 存在显著差异，详见：

- [01_Overview.md](./01_Overview.md) - 适配差异概览
- [05_API_Differences.md](./05_API_Differences.md) - 详细 API 差异

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - 库概览
- [05_API_Differences.md](./05_API_Differences.md) - API 差异
