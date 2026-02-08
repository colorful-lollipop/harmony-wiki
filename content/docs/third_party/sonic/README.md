# Sonic Wiki 目录

## Sonic 开源库在 OpenHarmony 中的集成与适配文档

本文档目录包含 Sonic 语音变速算法库在 OpenHarmony (OH) 中的详细分析，重点关注 OH 的定制化内容，包括 Patch、构建适配、依赖关系和使用方式。

---

## 快速导航

### 📋 基础信息
- [**01_Overview.md**](01_Overview.md) - 库概览、功能介绍、在 OH 中的作用和定位
- [**02_Patches.md**](02_Patches.md) - **核心文档** - Patch 详细分析（双声道 Bug 修复）

### 🔧 技术适配
- [**03_Build_Integration.md**](03_Build_Integration.md) - OH 构建适配、BUILD.gn 配置分析
- [**04_Usage_in_OH.md**](04_Usage_in_OH.md) - 依赖关系与使用方式、依赖关系图
- [**05_API_Differences.md**](05_API_Differences.md) - API 接口说明、与上游版本的差异

### 🔒 安全分析
- [**06_Security.md**](06_Security.md) - 安全风险分析、CVE 评估、安全建议

---

## 关键信息速览

### 库基本信息

| 属性 | 内容 |
|------|------|
| **库名称** | Sonic |
| **上游地址** | https://github.com/waywardgeek/sonic |
| **OH 版本** | 0.2.0 |
| **许可证** | Apache 2.0 |
| **功能** | 语音变速算法（加速/减速） |

### OH 关键变更

| 变更项 | 内容 |
|--------|------|
| **Patch 数量** | 1 个（双声道处理 Bug 修复） |
| **BUILD.gn** | 新增，适配 OH 构建系统 |
| **安全增强** | 启用 PAC 分支保护 |
| **API 级别** | platformsdk（平台 SDK 内部 API） |

### 依赖关系

```
third_party/sonic (源码)
    ↓
third_party/pulseaudio/sonic (封装)
    ↓
pulseaudio:sonic (external_deps)
    ↓
audio_framework/services/audio_service (条件编译)
    ↓
音频播放器/语音消息应用
```

---

## 阅读建议

### 按角色阅读

**架构师/技术负责人**:
1. [01_Overview.md](01_Overview.md) - 了解库的整体定位
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 理解依赖关系和架构位置
3. [06_Security.md](06_Security.md) - 评估安全风险

**开发工程师**:
1. [05_API_Differences.md](05_API_Differences.md) - 掌握 API 使用
2. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建配置
3. [02_Patches.md](02_Patches.md) - 了解 Bug 修复详情

**维护工程师**:
1. [02_Patches.md](02_Patches.md) - 了解 OH 特有修改
2. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建适配
3. [06_Security.md](06_Security.md) - 了解安全注意事项

### 按任务阅读

**了解整体情况**: [01_Overview.md](01_Overview.md)

**升级上游版本**:
1. [02_Patches.md](02_Patches.md) - 了解需要保留的修改
2. [05_API_Differences.md](05_API_Differences.md) - 检查 API 兼容性
3. [03_Build_Integration.md](03_Build_Integration.md) - 确认 BUILD.gn 配置

**排查问题**:
1. [02_Patches.md](02_Patches.md) - 检查已知 Bug 修复
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 确认依赖关系
3. [03_Build_Integration.md](03_Build_Integration.md) - 检查构建配置

---

## 文档质量说明

### 完成度

| 检查项 | 状态 |
|--------|------|
| Patch 分析覆盖全部 | ✅ 是（1 个 Bug 修复） |
| 主要依赖者列出 | ✅ 是（audio_service, pulseaudio modules, 50+ FuzzTest） |
| 依赖关系图 | ✅ 已提供 |
| BUILD.gn 配置分析 | ✅ 已提供 |
| API 文档 | ✅ 已提供 |
| 安全分析 | ✅ 已提供 |
| 技术结论有证据支撑 | ✅ 基于源代码分析 |

### 证据来源

本文档的所有结论均基于以下证据：
- ✅ 源代码分析（sonic.h, sonic.c, BUILD.gn）
- ✅ Patch 文件分析（patch.txt）
- ✅ 依赖关系搜索（OH 代码库全局搜索）
- ✅ 上游仓库调研（GitHub waywardgeek/sonic）

---

## 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02-07 | v1.0 | 初始版本，完成所有核心文档 |

---

## 反馈与贡献

如发现文档中的错误或需要补充的内容，请联系：
- 维护者: guoyichen2@huawei.com (bundle.json 中的 OH 维护者)

---

**本文档遵循证据优先原则，所有关于 Patch 和适配的结论均有直接代码或配置支持。**
