# 文档阅读路线建议

本文档提供 libtiff Wiki 的阅读路线建议，帮助不同角色的读者快速找到所需信息。

---

## 按角色阅读

### 1. 应用开发者

**目标**: 了解如何在应用中使用 libtiff

**推荐路线**:

1. [README.md](README.md) - 快速了解 libtiff
2. [01_Overview.md](01_Overview.md) - 库的基本介绍
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系和使用场景
4. [05_API_Differences.md](05_API_Differences.md) - API 和功能限制
5. [README_zh.md](../README_zh.md) - API 使用示例

**预计阅读时间**: 20-30 分钟

---

### 2. 构建工程师

**目标**: 了解 libtiff 的构建系统和编译配置

**推荐路线**:

1. [README.md](README.md) - 快速了解
2. [03_Build_Integration.md](03_Build_Integration.md) - **BUILD.gn 详解（核心文档）**
3. [01_Overview.md](01_Overview.md) - 库基本信息
4. [BUILD.gn](../BUILD.gn) - 查看完整构建配置

**预计阅读时间**: 30-45 分钟

---

### 3. 系统集成者

**目标**: 了解 libtiff 在 OpenHarmony 系统中的位置和依赖关系

**推荐路线**:

1. [README.md](README.md) - 快速了解
2. [01_Overview.md](01_Overview.md) - 在 OH 中的作用和定位
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - **依赖关系和使用场景（核心文档）**
4. [02_Patches.md](02_Patches.md) - 了解零 Patch 集成特点
5. [05_API_Differences.md](05_API_Differences.md) - 功能限制

**预计阅读时间**: 25-35 分钟

---

### 4. 版本维护者

**目标**: 评估升级上游版本或维护当前版本

**推荐路线**:

1. [README.md](README.md) - 快速了解
2. [02_Patches.md](02_Patches.md) - **了解零 Patch 集成**
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建系统适配
4. [06_Security.md](06_Security.md) - **版本差异和安全建议（重要）**
5. [ASSESSMENT.md](_work/ASSESSMENT.md) - 完整项目评估

**预计阅读时间**: 40-60 分钟

---

### 5. 安全工程师

**目标**: 了解 libtiff 的安全风险和升级建议

**推荐路线**:

1. [README.md](README.md) - 快速了解
2. [06_Security.md](06_Security.md) - **安全分析（核心文档）**
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解使用场景
4. [02_Patches.md](02_Patches.md) - 确认无 Patch 引入的新攻击面

**预计阅读时间**: 20-30 分钟

---

## 按任务阅读

### 任务：首次集成 libtiff

**推荐阅读**:

1. [README.md](README.md) - 快速了解
2. [01_Overview.md](01_Overview.md) - 库的基本介绍
3. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 配置
4. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系
5. [README_zh.md](../README_zh.md) - API 使用示例

**关键步骤**:
```gn
// 在 BUILD.gn 中添加依赖
external_deps += [ "libtiff:libtiff" ]
```

---

### 任务：排查构建问题

**推荐阅读**:

1. [03_Build_Integration.md](03_Build_Integration.md) - **重点查看构建配置**
2. [BUILD.gn](../BUILD.gn) - 检查编译选项
3. [install.sh](../install.sh) - 查看构建脚本
4. [README_zh.md](../README_zh.md) - 检查功能限制

**常见问题**:
- 压缩算法未启用 → 检查 `enable_*` 配置
- 依赖缺失 → 检查 `external_deps`
- 条件编译问题 → 检查 `has_libtiff` 开关

---

### 任务：升级上游版本

**推荐阅读**:

1. [06_Security.md](06_Security.md) - **版本差异和安全建议**
2. [02_Patches.md](02_Patches.md) - 确认零 Patch 集成
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建系统适配
4. [ASSESSMENT.md](_work/ASSESSMENT.md) - 完整评估
5. [NOTES.md](_work/NOTES.md) - 分析过程记录

**升级优势**:
- 零 Patch 集成，升级相对容易
- 仅需更新源码和版本号
- 可能需要调整 BUILD.gn 配置

---

### 任务：了解安全风险

**推荐阅读**:

1. [06_Security.md](06_Security.md) - **安全分析（核心）**
2. [ASSESSMENT.md](_work/ASSESSMENT.md) - 安全风险评估
3. [02_Patches.md](02_Patches.md) - 确认无 Patch 引入新风险

**安全建议**:
- 考虑升级至 4.7.1 以获取安全修复
- 审查压缩算法配置的安全性
- 继续使用模糊测试

---

## 文档优先级

### 核心文档（必读）

| 文档 | 内容 | 适用角色 |
|-----|------|---------|
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建系统详解 | 构建工程师、系统集成者、版本维护者 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用场景 | 应用开发者、系统集成者、安全工程师 |
| [README.md](README.md) | 库概览和导航 | **所有读者** |

### 重要文档（推荐）

| 文档 | 内容 | 适用角色 |
|-----|------|---------|
| [01_Overview.md](01_Overview.md) | 库概览和基本介绍 | **所有读者** |
| [06_Security.md](06_Security.md) | 版本差异和安全建议 | 版本维护者、安全工程师 |

### 参考文档（按需）

| 文档 | 内容 | 适用角色 |
|-----|------|---------|
| [02_Patches.md](02_Patches.md) | Patch 分析（零 Patch 说明） | 版本维护者、升级开发者 |
| [05_API_Differences.md](05_API_Differences.md) | API 和功能限制 | 应用开发者 |
| [ASSESSMENT.md](_work/ASSESSMENT.md) | 完整项目评估 | 版本维护者、系统集成者 |
| [NOTES.md](_work/NOTES.md) | 分析过程记录 | 版本维护者、开发者 |

---

## 快速索引

### 按主题查找

**构建与编译**
- BUILD.gn 配置 → [03_Build_Integration.md](03_Build_Integration.md)
- 压缩算法选项 → [03_Build_Integration.md](03_Build_Integration.md)
- 依赖库 → [03_Build_Integration.md](03_Build_Integration.md)
- 构建流程 → [03_Build_Integration.md](03_Build_Integration.md)

**使用与集成**
- 依赖关系 → [04_Usage_in_OH.md](04_Usage_in_OH.md)
- 使用场景 → [04_Usage_in_OH.md](04_Usage_in_OH.md)
- API 示例 → [README_zh.md](../README_zh.md)
- 功能限制 → [05_API_Differences.md](05_API_Differences.md)

**版本与安全**
- 版本差异 → [06_Security.md](06_Security.md)
- 安全风险 → [06_Security.md](06_Security.md)
- CVE 状态 → [06_Security.md](06_Security.md)
- 升级建议 → [06_Security.md](06_Security.md)

**集成与维护**
- Patch 分析 → [02_Patches.md](02_Patches.md)
- 零 Patch 说明 → [02_Patches.md](02_Patches.md)
- 项目评估 → [ASSESSMENT.md](_work/ASSESSMENT.md)

---

## 文档版本信息

- **文档版本**: 1.0
- **libtiff 版本**: 4.7.0
- **上游版本**: 4.7.1（截至文档更新时）
- **最后更新**: 2026年2月8日

---

**建议**: 首次阅读建议按"应用开发者"路线走一遍，然后根据具体任务深入阅读相关章节。
