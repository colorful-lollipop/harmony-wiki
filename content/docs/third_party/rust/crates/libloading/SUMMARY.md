# 阅读路线指南

本文档提供 libloading 库在 OpenHarmony 中集成的完整阅读指南，帮助不同需求的读者快速定位所需信息。

## 读者类型与推荐阅读路径

### 1. 快速了解（5 分钟）

如果您只需要了解 libloading 在 OpenHarmony 中的基本定位：

1. **README.md** - 库概览和 OH 适配概述
2. **01_Overview.md** - 原始库功能简介
3. **03_Build_Integration.md** - 构建适配要点

### 2. 深入理解（15-30 分钟）

如果您需要全面理解 libloading 的 OH 集成细节：

1. 按顺序阅读 **01_Overview.md** 到 **04_Usage_in_OH.md**
2. 重点关注 **03_Build_Integration.md** 中的构建配置说明
3. 查看 **04_Usage_in_OH.md** 了解实际使用场景

### 3. 安全评估（10 分钟）

如果您关注安全相关问题：

1. **06_Security.md** - 安全风险分析
2. **02_Patches.md** - Patch 安全审查（本库无 Patch）
3. **03_Build_Integration.md** - 构建配置安全性

### 4. 维护者视角（20 分钟）

如果您需要维护或升级此库：

1. **README.md** - 了解整体架构
2. **02_Patches.md** - Patch 维护建议（本库无 Patch）
3. **03_Build_Integration.md** - 构建系统适配
4. **04_Usage_in_OH.md** - 依赖关系和升级影响评估

## 文档依赖关系

```
01_Overview.md (基础)
    ↓
02_Patches.md (Patch 分析，需要了解基础)
    ↓
03_Build_Integration.md (构建适配，可独立阅读)
04_Usage_in_OH.md (使用情况，可独立阅读)
    ↓
05_API_Differences.md (API 差异，需要了解基础)
06_Security.md (安全分析，综合文档)
```

## 快速参考

### 关键信息速查

| 问题 | 答案 |
|------|------|
| 是否有 OH Patch？ | 否，该库为原生移植 |
| 主要依赖者？ | 待进一步分析 |
| 构建类型？ | rlib（静态库） |
| 平台支持？ | Unix/Linux/macOS/Windows |

### 常见问题

**Q: libloading 在 OH 中的主要用途是什么？**
A: 提供 Rust 动态库加载能力，用于加载.so/.dylib/.dll 格式的动态链接库。

**Q: 为什么 libloading 没有 OH Patch？**
A: 因为 libloading 通过 cfg-if 实现了良好的平台抽象，Unix 平台（包括 OH 基于的 Linux 内核）的动态库加载 API 与上游代码兼容。

**Q: 如何在 OH 中使用 libloading？**
A: 通过 OH 的 Rust 构建系统，在 BUILD.gn 中添加依赖即可。

## 反馈与贡献

如发现文档错误或需要补充内容，请联系组件维护者或在 OpenHarmony 相关仓库提 Issue。
