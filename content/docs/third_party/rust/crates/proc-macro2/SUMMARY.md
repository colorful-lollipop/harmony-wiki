# 阅读指南

本文档提供了 proc-macro2 在 OpenHarmony 中的集成与适配说明。

## 推荐阅读路线

### 场景 1: 快速了解
如果只需要了解该库的基本信息，建议阅读:

```
README.md → 01_Overview.md
```

### 场景 2: 构建适配
如果需要了解如何在 OH 中构建或修改该库:

```
README.md → 03_Build_Integration.md → 04_Usage_in_OH.md
```

### 场景 3: 升级维护
如果需要进行版本升级或维护工作:

```
README.md → 02_Patches.md → 03_Build_Integration.md → 04_Usage_in_OH.md
```

## 文档结构

| 文档 | 内容 | 目标读者 |
|------|------|----------|
| README.md | 项目概览、导航、OH 适配状态 | 所有读者 |
| 01_Overview.md | 原始库功能、OH 中的定位 | 需要了解该库用途的开发者 |
| 02_Patches.md | OH Patch 详细分析 | 版本升级维护者 |
| 03_Build_Integration.md | BUILD.gn 配置、编译选项 | 构建系统维护者 |
| 04_Usage_in_OH.md | 依赖关系、使用场景 | 需要使用该库的开发者 |

## 关键结论

**本库在 OpenHarmony 中没有使用任何 Patch 文件**，因为:

1. proc-macro2 是纯 Rust 实现，不涉及平台特定代码
2. 库的核心 API 稳定，与 Rust 编译器版本解耦
3. OH 使用的功能是库的标准功能，无需修改

升级时可直接使用上游最新版本，无需额外适配工作。
