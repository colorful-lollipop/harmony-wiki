# 阅读路线建议

本文档针对不同类型的读者提供阅读路线，帮助快速定位所需信息。

---

## 按角色分类

### 1. 系统集成者（需要了解如何集成到新模块）

**目标**: 了解如何在新模块中使用 optimized-routines

**预计时间**: 15 分钟

**阅读路线**:

```mermaid
graph LR
    A[README.md] --> B[04_Usage_in_OH.md]
    B --> C[03_Build_Integration.md]
```

**必读文档**:
1. **[README.md](README.md)** (5 分钟)
   - 快速概览
   - OH 适配特点
   - 常见问题

2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** (5 分钟)
   - 依赖方式（源码级集成）
   - 添加到新模块的步骤
   - 注意事项

3. **[03_Build_Integration.md](03_Build_Integration.md)** (5 分钟)
   - BUILD.gn 配置示例
   - 架构选择逻辑
   - 编译选项

**可选文档**:
- **[02_Adaptations.md](02_Adaptations.md)** - 如需了解适配机制

---

### 2. 开发者（需要了解 API 和使用方式）

**目标**: 了解 optimized-routines 提供的函数和性能特性

**预计时间**: 10 分钟

**阅读路线**:

```mermaid
graph LR
    A[README.md] --> B[01_Overview.md]
    B --> C[04_Usage_in_OH.md]
```

**必读文档**:
1. **[README.md](README.md)** (3 分钟)
   - 库定位
   - 关键概念

2. **[01_Overview.md](01_Overview.md)** (4 分钟)
   - 原始功能概述（字符串、数学函数）
   - 架构支持矩阵

3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** (3 分钟)
   - 使用场景
   - 性能提升

**可选文档**:
- **[03_Build_Integration.md](03_Build_Integration.md)** - 如需了解编译选项

---

### 3. 维护者（需要了解适配细节和升级策略）

**目标**: 掌握 OH 适配机制，能够升级上游版本

**预计时间**: 30 分钟

**阅读路线**:

```mermaid
graph LR
    A[_work/ASSESSMENT.md] --> B[02_Adaptations.md]
    B --> C[03_Build_Integration.md]
    C --> D[06_Security.md]
```

**必读文档**:
1. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** (5 分钟)
   - 项目评估结果
   - Patch 分析结果
   - 依赖者统计

2. **[02_Adaptations.md](02_Adaptations.md)** (10 分钟)
   - OH 适配方式详解
   - 源码级集成机制
   - 符号别名机制
   - OH 特定文件清单

3. **[03_Build_Integration.md](03_Build_Integration.md)** (10 分钟)
   - BUILD.gn 配置详解
   - ARMv7 / AArch64 配置
   - SVE / MTE 条件编译
   - 安全特性配置

4. **[06_Security.md](06_Security.md)** (5 分钟)
   - 安全特性（PAC-RET、栈保护、HWASAN）
   - 安全升级策略

**可选文档**:
- **[_work/NOTES.md](_work/NOTES.md)** - 分析过程记录
- **[_work/PLAN.md](_work/PLAN.md)** - 文档计划

---

### 4. 深入学习者（需要全面了解）

**目标**: 全面掌握 optimized-routines 在 OH 中的应用

**预计时间**: 60 分钟+

**阅读路线**:

```mermaid
graph LR
    A[README.md] --> B[01_Overview.md]
    B --> C[02_Adaptations.md]
    C --> D[03_Build_Integration.md]
    D --> E[04_Usage_in_OH.md]
    E --> F[06_Security.md]
```

**必读文档**:
1. **[README.md](README.md)** - 库概览和导航
2. **[01_Overview.md](01_Overview.md)** - 原始库简介
3. **[02_Adaptations.md](02_Adaptations.md)** - OH 适配说明
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建集成详解
5. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 使用情况和依赖
6. **[06_Security.md](06_Security.md)** - 安全风险分析

**可选文档**:
- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 项目评估结果
- **[_work/NOTES.md](_work/NOTES.md)** - 分析过程记录
- **[_work/PLAN.md](_work/PLAN.md)** - 文档计划

**扩展阅读**:
- 原始 upstream 文档（见 [01_Overview.md](01_Overview.md) 的"相关资源"）
- ARM PAC-RET / MTE 官方文档（见 [06_Security.md](06_Security.md) 的"参考资源"）

---

## 按需求分类

### 快速了解（5 分钟）

**目标**: 快速获取库的基本信息

**阅读路线**:
1. **[README.md](README.md)** - 快速概览
2. **[01_Overview.md](01_Overview.md)** - "在 OH 中的定位" 部分

**关键信息**:
- 库是什么
- OH 适配特点
- 系统覆盖范围

---

### 集成到新模块（15 分钟）

**目标**: 了解如何在新模块中使用

**阅读路线**:
1. **[README.md](README.md)** - 关键概念部分
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - "添加到新模块" 部分
3. **[03_Build_Integration.md](03_Build_Integration.md)** - "配置示例" 部分

**关键信息**:
- 源码级集成方式
- BUILD.gn 配置示例
- 注意事项

---

### 版本升级维护（30 分钟）

**目标**: 掌握升级上游版本的流程

**阅读路线**:
1. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - "0.2 Patch 分析" 和 "0.4 特殊适配识别"
2. **[02_Adaptations.md](02_Adaptations.md)** - "与上游的兼容性" 部分
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 完整阅读
4. **[06_Security.md](06_Security.md)** - "安全升级策略" 部分

**关键信息**:
- 升级步骤
- 验证方法
- 安全考虑

---

### 深入理解（60 分钟+）

**目标**: 全面掌握所有细节

**阅读路线**: 见"深入学习者"部分

**关键信息**:
- 所有文档的完整内容
- 分析过程记录
- 参考资源

---

## 文档索引

### 核心文档

| 文档 | 用途 | 阅读时间 |
|------|------|---------|
| [README.md](README.md) | 库概览和导航 | 5 分钟 |
| [01_Overview.md](01_Overview.md) | 原始库简介 | 5 分钟 |
| [02_Adaptations.md](02_Adaptations.md) | OH 适配说明 | 10 分钟 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建集成详解 | 10 分钟 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 使用情况和依赖 | 8 分钟 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 8 分钟 |

### 工作文档

| 文档 | 用途 | 阅读时间 |
|------|------|---------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果 | 5 分钟 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 | 5 分钟 |
| [_work/PLAN.md](_work/PLAN.md) | 文档计划 | 3 分钟 |

---

## 快速参考

### 常见问题速查

| 问题 | 参考文档 | 章节 |
|------|---------|------|
| 为什么没有 Patch？ | [02_Adaptations.md](02_Adaptations.md) | Patch 分析结果 |
| 如何在新模块中使用？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 添加到新模块 |
| 如何启用 SVE/MTE？ | [03_Build_Integration.md](03_Build_Integration.md) | 扩展特性控制 |
| 如何升级上游版本？ | [02_Adaptations.md](02_Adaptations.md) | 与上游的兼容性 |
| 安全特性有哪些？ | [06_Security.md](06_Security.md) | 安全特性 |

### 关键配置速查

| 配置项 | 值 | 参考文档 |
|--------|-----|---------|
| `musl_arch` | `"arm"` 或 `"aarch64"` | [03_Build_Integration.md](03_Build_Integration.md) |
| `ARM_FEATURE_SVE` | `true` 或 `false` | [03_Build_Integration.md](03_Build_Integration.md) |
| `ARM_FEATURE_MTE` | `true` 或 `false` | [03_Build_Integration.md](03_Build_Integration.md) |
| `use_hwasan` | `true` 或 `false` | [06_Security.md](06_Security.md) |

---

## 反馈与更新

如果您在阅读过程中有任何问题或建议，欢迎反馈：
- **Owner**: zhaotianyu9@huawei.com
- **上游 Issue**: https://github.com/ARM-software/optimized-routines/issues

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
