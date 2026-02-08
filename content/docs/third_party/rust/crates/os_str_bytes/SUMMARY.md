# 文档阅读指南

## 文档定位

本文档集专注于说明 `os_str_bytes` 库在 **OpenHarmony (OH)** 中的集成与适配情况。

**重要说明**：OH 未对该库进行任何源代码修改，仅添加了构建系统适配。本文档的重点是构建集成和依赖关系分析，而非原始库的功能介绍。

---

## 阅读路线

### 路线 1：快速了解 OH 集成（推荐给新维护者）

**总时间**: ~15 分钟

1. **[README.md](README.md)** - 2 分钟
   - 了解关键结论（OH 适配复杂度、源代码修改情况）
   - 查看 OH 集成摘要

2. **[01_Overview.md](01_Overview.md)** - 5 分钟
   - 了解库的基本信息
   - 理解该库在 OH 中的作用和定位

3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 5 分钟
   - 查看依赖关系图
   - 了解 OH 中哪些工具使用了该库

4. **[03_Build_Integration.md](03_Build_Integration.md)** - 3 分钟
   - 了解 OH 如何将 Cargo.toml 映射到 BUILD.gn
   - 查看编译配置

---

### 路线 2：深度技术分析（推荐给需要修改构建配置的开发者）

**总时间**: ~25 分钟

1. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 10 分钟
   - Phase 0 评估结果的完整技术细节
   - Patch 分析、依赖分析、特殊适配识别

2. **[03_Build_Integration.md](03_Build_Integration.md)** - 5 分钟
   - BUILD.gn 详细配置
   - Cargo.toml 与 BUILD.gn 的字段映射关系

3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 5 分钟
   - 依赖关系详细分析
   - 使用场景说明

4. **[02_Patches.md](02_Patches.md)** - 2 分钟
   - 确认无 Patch 的事实
   - 了解为什么不需要 Patch

5. **[01_Overview.md](01_Overview.md)** - 3 分钟
   - 库的功能概述（可选）

---

### 路线 3：升级评估（推荐给准备升级该库版本的开发者）

**总时间**: ~10 分钟

1. **[02_Patches.md](02_Patches.md)** - 2 分钟
   - **核心信息**: 确认无 Patch，升级风险低
   - 了解升级建议

2. **[03_Build_Integration.md](03_Build_Integration.md)** - 5 分钟
   - 检查 BUILD.gn 中的版本号
   - 检查 features 配置是否与上游兼容
   - 检查依赖（memchr）的版本兼容性

3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 3 分钟
   - 了解哪些依赖者可能受影响
   - 评估升级范围

---

## 文档快速参考

### 我想知道...

| 问题 | 查看文档 |
|-----|---------|
| 这个库是干什么的？ | [01_Overview.md](01_Overview.md) - "原始库简介" |
| OH 修改了源代码吗？ | [02_Patches.md](02_Patches.md) - "核心结论" |
| 有哪些 Patch？ | [02_Patches.md](02_Patches.md) - "完整清单" |
| OH 如何构建这个库？ | [03_Build_Integration.md](03_Build_Integration.md) |
| OH 中谁在用这个库？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) - "直接依赖者" |
| 可以直接升级上游版本吗？ | [02_Patches.md](02_Patches.md) - "升级建议" |
| 版本号为什么不一致？ | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - "0.4 特殊适配识别" |

---

## 核心结论速查

| 方面 | 结论 |
|-----|------|
| **源代码修改** | 无（OH 未修改任何源文件） |
| **Patch 文件** | 无 |
| **构建适配** | 仅添加 BUILD.gn 和 bundle.json |
| **当前版本** | 6.4.1（bundle.json 显示 6.1，需修复） |
| **维护风险** | 低（可直接跟随上游升级） |
| **依赖者** | clap_lex → clap → bindgen-cli, cxxbridge-cmd |

---

## 文档结构图

```
wiki/
├── README.md                      # 入口文档（从这里开始）
├── SUMMARY.md                     # 本文件（阅读指南）
├── 01_Overview.md                 # 库概览 + OH 定位
├── 02_Patches.md                  # Patch 分析（说明无 Patch）
├── 03_Build_Integration.md       # BUILD.gn 构建适配
├── 04_Usage_in_OH.md              # OH 依赖关系与使用
└── _work/
    ├── ASSESSMENT.md              # Phase 0 技术评估结果
    ├── NOTES.md                   # 分析过程记录（可选）
    └── PLAN.md                    # 任务进度（可选）
```

---

**最后更新**: 2026-02-08
