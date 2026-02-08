# 阅读路线建议

本文档为 Brotli 第三方库集成文档的阅读路线指南，帮助不同角色的读者快速定位所需信息。

## 读者类型与推荐阅读

### 1. 应用开发者

**目标**：了解如何在应用中集成和使用 Brotli 压缩功能

推荐阅读顺序：
1. [README.md](README.md) - 库概览
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用场景和依赖方式
3. 上游 Brotli 文档（API 使用）

### 2. 系统集成工程师

**目标**：了解 Brotli 在 OH 系统中的构建和集成方式

推荐阅读顺序：
1. [README.md](README.md) - 适配概述
2. [03_Build_Integration.md](03_Build_Integration.md) - GN 构建配置
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系

### 3. 安全工程师

**目标**：评估 Brotli 的安全风险和加固措施

推荐阅读顺序：
1. [06_Security.md](06_Security.md) - 安全风险分析
2. [README.md](README.md) - 版本和 CVE 状态

### 4. 版本维护者

**目标**：了解如何维护和升级 Brotli 版本

推荐阅读顺序：
1. [02_Patches.md](02_Patches.md) - Patch 策略和维护建议
2. [06_Security.md](06_Security.md) - 安全更新策略
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置兼容性

## 文档依赖关系

```
README.md
├── 01_Overview.md (可选，了解原始功能)
├── 02_Patches.md (必须，理解 Patch 策略)
├── 03_Build_Integration.md (集成必读)
├── 04_Usage_in_OH.md (依赖关系必读)
├── 05_API_Differences.md (可选，无 API 差异)
└── 06_Security.md (安全评估必读)
```

## 快速查找

### 我想了解...

| 问题 | 前往文档 |
|-----|---------|
| Brotli 是什么？ | 01_Overview.md |
| OH 对 Brotli 做了哪些修改？ | 02_Patches.md |
| 如何构建 Brotli？ | 03_Build_Integration.md |
| 谁在使用 Brotli？ | 04_Usage_in_OH.md |
| 有没有 API 差异？ | 05_API_Differences.md |
| 有什么安全风险？ | 06_Security.md |

## 关键信息速览

### 版本信息
- **OH 版本**：v1.1.0
- **上游版本**：v1.1.0
- **同步状态**：✅ 已同步

### Patch 状态
- **本地 Patch**：无
- **Patch 策略**：通过 productdefine_common 集中管理

### 依赖统计
- **直接依赖者**：3 个主要模块（curl, skia/freetype2, skia/libjxl）
- **安全状态**：无已知未修复漏洞
