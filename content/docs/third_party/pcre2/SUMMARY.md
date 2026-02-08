# SUMMARY - 阅读路线建议

本文档是 OpenHarmony third_party/pcre2 的 Wiki，专注于分析 PCRE2 库在 OH 中的集成、Patch 和使用方式。

## 文档清单

| 文档 | 内容 | 推荐阅读人群 |
|-----|------|-------------|
| [README.md](./README.md) | 库概览与文档导航 | 所有读者 |
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 | 新接触该库的开发者 |
| [02_Patches.md](./02_Patches.md) | **核心文档** - Patch 详细分析 | 维护者、升级负责人 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配 | 构建系统开发者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | 系统开发者 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 | 应用开发者 |
| [06_Security.md](./06_Security.md) | 安全分析与 CVE | 安全工程师 |

## 推荐阅读路线

### 路线 1: 快速了解（10分钟）

适合：初次接触该库的开发者

```
README.md 
    → 01_Overview.md（仅读"基本信息"和"OH 中的定位"）
    → 02_Patches.md（仅读"Patch 清单概览"）
```

**收获**: 
- 了解 PCRE2 是什么
- 知道 OH 有一个关键 Patch
- 了解主要使用场景

### 路线 2: Patch 深度理解（30分钟）

适合：需要维护或升级该库的开发者

```
README.md
    → 01_Overview.md
    → 02_Patches.md（完整阅读）
    → 03_Build_Integration.md（重点看 Patch 应用机制）
    → 05_API_Differences.md
```

**收获**:
- 完全理解换行符 Patch 的原理
- 掌握 Patch 应用机制
- 了解升级时的注意事项

### 路线 3: 系统开发者（45分钟）

适合：开发依赖 PCRE2 的组件（如 ArkCompiler、SELinux）

```
README.md
    → 01_Overview.md
    → 02_Patches.md
    → 03_Build_Integration.md
    → 04_Usage_in_OH.md（完整阅读）
    → 05_API_Differences.md
```

**收获**:
- 理解如何正确使用 PCRE2
- 了解静态库 vs 共享库的区别
- 掌握链接和头文件引用方式

### 路线 4: 安全审计（20分钟）

适合：安全工程师、版本升级审核

```
README.md
    → 02_Patches.md（了解 Patch 的安全影响）
    → 06_Security.md（完整阅读）
```

**收获**:
- 了解已知 CVE 和修复状态
- 掌握安全使用建议
- 建立升级安全检查清单

## 关键信息速查

### 基本信息

| 项目 | 值 |
|-----|-----|
| **库名称** | PCRE2 |
| **上游版本** | 10.46 |
| **OH 组件** | @ohos/pcre2 |
| **许可证** | BSD-3-Clause WITH PCRE2-exception |

### 关键 Patch

| Patch | 位置 | 目的 |
|-------|-----|-----|
| pcre2_newline.patch | `arkcompiler/runtime_core/.../patches/` | 修改换行符识别，适配 ArkTS/ETS |

### 构建目标

| 目标 | 类型 | 特殊配置 |
|-----|------|---------|
| libpcre2 | 共享库 | 原生行为，无 Unicode |
| libpcre2_static | 静态库 8-bit | 启用 Patch，Unicode |
| libpcre2_static_16 | 静态库 16-bit | 启用 Patch，Unicode |

### 主要依赖者

1. **ArkCompiler Runtime Core** - ArkTS/ETS 正则表达式
2. **SELinux Adapter** - 安全策略解析
3. **Cangjie Runtime** - 仓颉语言正则

### 安全状态

| CVE | 影响版本 | OH 状态 |
|-----|---------|---------|
| CVE-2025-58050 | 10.45 | ✅ 已修复（当前 10.46）|

## 文档更新记录

| 日期 | 版本 | 更新内容 | 维护者 |
|-----|------|---------|-------|
| 2025-02-07 | 1.0 | 初始版本 | Wiki Agent |

## 反馈与维护

如有问题或建议，请联系：
- **组件 Owner**: maliang34@huawei.com
- **OH 社区**: https://gitee.com/openharmony

---

**文档定位**: third_party/pcre2/wiki/

**下次更新时机**: 
- PCRE2 上游版本升级
- Patch 内容变更
- 新增 CVE 或安全问题
