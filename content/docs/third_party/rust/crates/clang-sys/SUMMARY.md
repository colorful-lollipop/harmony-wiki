# 阅读路线建议

## 根据角色的阅读路线

### 如果你是：第三方库维护者

**阅读顺序**:
1. [ASSESSMENT.md](_work/ASSESSMENT.md) - 了解整体评估结论
2. [02_Patches.md](02_Patches.md) - 确认无 Patch，了解原因
3. [03_Build_Integration.md](03_Build_Integration.md) - 学习 BUILD.gn 配置

**重点**: 理解为什么该库无需 Patch 即可在 OH 中工作。

### 如果你是：Rust 开发者/FFI 绑定开发者

**阅读顺序**:
1. [01_Overview.md](01_Overview.md) - 了解库的功能和定位
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解在 OH 中的使用方式
3. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建配置

**重点**: 理解如何将 libclang 功能集成到 Rust 项目中。

### 如果你是：安全/维护工程师

**阅读顺序**:
1. [02_Patches.md](02_Patches.md) - 确认 Patch 状态
2. [06_Security.md](06_Security.md) - 了解安全风险
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解影响范围

**重点**: 评估 FFI 带来的安全风险，制定升级策略。

### 如果你是：新成员/学习者

**阅读顺序**:
1. [README.md](README.md) - 获取整体概览
2. [01_Overview.md](01_Overview.md) - 了解原始库
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解在 OH 中的作用

**重点**: 建立对 FFI 绑定库在构建系统中作用的整体认知。

## 按主题的阅读路线

### 主题：理解 FFI 绑定模式

1. [01_Overview.md](01_Overview.md) - 什么是 FFI 绑定
2. [03_Build_Integration.md](03_Build_Integration.md) - 如何配置构建
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 典型使用场景

### 主题：学习构建适配

1. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 详解
2. [ASSESSMENT.md](_work/ASSESSMENT.md) - 评估报告中的构建分析
3. 对比上游 `Cargo.toml` 与 `BUILD.gn`

### 主题：依赖关系分析

1. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 完整依赖分析
2. [ASSESSMENT.md](_work/ASSESSMENT.md) - 评估报告第 0.4 节
3. 搜索 OH 代码库中的 `rust_bindgen` 使用

## 文档间关联图

```
README.md (起点)
    ├── 01_Overview.md (基础)
    │       └── 了解库的基本功能
    ├── 02_Patches.md (适配)
    │       └── 理解为何无 Patch
    ├── 03_Build_Integration.md (构建)
    │       └── BUILD.gn 配置详解
    │       └── 与 01_Overview 中的上游构建对比
    ├── 04_Usage_in_OH.md (应用)
    │       └── 了解 bindgen 如何使用该库
    │       └── 与 03_Build_Integration 中的 features 关联
    ├── 05_API_Differences.md (差异)
    │       └── 确认无 API 变更
    └── 06_Security.md (安全)
            └── 与 04_Usage_in_OH 中的影响范围关联
```

## 推荐阅读时间

- **快速浏览**（5 分钟）: README.md + 02_Patches.md
- **标准阅读**（15 分钟）: README.md + 01_Overview.md + 03_Build_Integration.md + 04_Usage_in_OH.md
- **深度阅读**（30 分钟）: 全部文档 + ASSESSMENT.md
