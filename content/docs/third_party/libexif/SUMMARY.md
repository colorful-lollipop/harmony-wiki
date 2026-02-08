# libexif Wiki 文档阅读路线

## 文档结构

```
wiki/
├── README.md              # 库概览、OH 适配概述、文档导航
├── SUMMARY.md            # 本文件：阅读路线建议
├── 01_Overview.md       # 原始库简介及在 OH 中的作用
├── 02_Patches.md       # OH 扩展实现分析（华为 Maker Note）
├── 03_Build_Integration.md  # BUILD.gn 构建系统适配
├── 04_Usage_in_OH.md   # 依赖关系与使用模式
├── 05_API_Differences.md # API 差异与 OH 特定功能
├── 06_Security.md       # 安全风险分析、CVE 修复状态
└── _work/
    ├── ASSESSMENT.md     # 项目评估结果
    ├── NOTES.md         # 分析过程记录
    └── PLAN.md         # 任务进度跟踪
```

## 阅读路线建议

### 路线 A：快速了解 OH 适配（推荐）

**目标**: 5 分钟内了解 libexif 在 OH 中的关键适配点

1. **[README.md](README.md)** - 阅读导航和核心特性
2. **[01_Overview.md](01_Overview.md)** - 了解原始库和 OH 定位
3. **[02_Patches.md](02_Patches.md)** - 重点阅读"华为 Maker Note 扩展"部分
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 查看"构建目标"和"安全加固"

**适合**: 想快速了解 OH 适配内容的开发者

---

### 路线 B：使用 libexif 的开发者

**目标**: 了解如何在自己的代码中使用 libexif

1. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看依赖关系和使用场景
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 了解如何在 BUILD.gn 中依赖 libexif
3. **[05_API_Differences.md](05_API_Differences.md)** - 查看可用的华为 Maker Note API

**适合**: 需要集成 libexif 到自己模块的 OH 开发者

---

### 路线 C：维护升级 libexif

**目标**: 了解版本升级注意事项和潜在风险

1. **[ASSESSMENT.md](_work/ASSESSMENT.md)** - 查看评估总结和升级建议
2. **[02_Patches.md](02_Patches.md)** - 了解华为扩展代码（升级时需保留）
3. **[06_Security.md](06_Security.md)** - 查看 CVE 修复状态和安全加固措施
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 了解构建配置差异

**适合**: 负责 libexif 维护和升级的工程师

---

### 路线 D：深度研究 OH 适配细节

**目标**: 全面理解 OH 的 libexif 定制化

**顺序阅读所有文档**:

1. **[01_Overview.md](01_Overview.md)** - 基础背景
2. **[02_Patches.md](02_Patches.md)** - 核心适配内容
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建系统集成
4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 实际使用情况
5. **[05_API_Differences.md](05_API_Differences.md)** - API 细节
6. **[06_Security.md](06_Security.md)** - 安全考虑

**适合**: 需要深入了解 OH 第三方库适配机制的研究人员

---

### 路线 E：安全审计

**目标**: 评估 libexif 的安全性

1. **[06_Security.md](06_Security.md)** - 重点关注：
   - 历史 CVE 修复状态
   - 华为代码的安全检查
   - OH 安全加固措施
   - 升级建议
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 查看安全配置
3. **[ASSESSMENT.md](_work/ASSESSMENT.md)** - 查看潜在风险

**适合**: 安全工程师、审计人员

---

## 关键问题快速导航

### 想了解...

| 问题 | 文档 | 章节 |
|------|------|------|
| libexif 是什么？ | [01_Overview.md](01_Overview.md) | 原始库简介 |
| OH 用它做什么？ | [01_Overview.md](01_Overview.md) | 在 OH 中的作用 |
| 有没有 Patch？ | [02_Patches.md](02_Patches.md) | Patch 分析概览 |
| 华为 Maker Note 是什么？ | [02_Patches.md](02_Patches.md) | 华为 Maker Note 扩展 |
| 如何在 BUILD.gn 中依赖？ | [03_Build_Integration.md](03_Build_Integration.md) | 依赖示例 |
| 谁在使用 libexif？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 直接依赖者列表 |
| 有什么安全风险？ | [06_Security.md](06_Security.md) | 安全风险分析 |
| 可以升级到新版本吗？ | [ASSESSMENT.md](_work/ASSESSMENT.md) | 升级建议 |

---

## 阅读提示

### 文档约定

- **证据优先**: 所有技术结论都有代码或配置文件证据支撑
- **版本标注**: 说明上游版本和 OH 组件版本
- **代码引用**: 关键代码片段有文件路径和行号引用
- **TODO 标注**: 不确定的内容标注 `TODO(需确认)`

### 专业术语

- **Maker Note**: 相机厂商在 EXIF 中嵌入的私有元数据
- **XMAGE**: 华为的影像技术品牌
- **XTStyle**: 华为的滤镜和风格系统
- **bounds_checking_function**: OH 的边界检查库
- **Fuzzer**: 模糊测试工具，用于发现安全漏洞

### 版本说明

- **上游版本**: libexif v0.6.25 (2025-01-08)
- **OH 组件版本**: 3.1
- **注意**: config.h 显示 0.6.24.1，可能未同步

---

## 反馈与更新

如有疑问或发现文档错误，请联系 OH multimedia 子系统维护团队。

**最后更新**: 2026-02-08
