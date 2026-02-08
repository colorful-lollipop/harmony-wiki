# 阅读路线指南

本文档为 OpenHarmony third_party/typescript 库的 Wiki 索引，帮助您快速找到需要的信息。

## 文档结构

```
wiki/
├── README.md              # 库概览、导航入口
├── SUMMARY.md             # 本阅读指南
├── 01_Overview.md         # 库概述与 OH 定位
├── 02_Patches.md          # OH 适配修改（核心）
├── 03_Build_Integration.md # 构建系统适配
├── 04_Usage_in_OH.md      # 依赖关系与使用场景
├── 05_API_Differences.md  # API 差异
└── 06_Security.md         # 安全考虑

_work/                      # 工作文档（内部使用）
├── ASSESSMENT.md          # 项目评估报告
├── NOTES.md               # 分析过程记录
└── PLAN.md                # 任务计划
```

## 推荐阅读路线

### 路线一：快速了解（5 分钟）

如果您只是想快速了解此库在 OH 中的作用：

1. 阅读 **README.md**（约 3 分钟）
   - 了解库的基本信息
   - 查看文档导航
   - 理解在 OH 中的定位

2. 阅读 **04_Usage_in_OH.md**（约 2 分钟）
   - 查看依赖关系图
   - 了解主要使用场景

### 路线二：深入理解（15 分钟）

如果您需要全面理解此库的 OH 适配：

1. **01_Overview.md**（3 分钟）
   - 原始 TypeScript 功能简介
   - 在 OH 生态中的角色

2. **02_Patches.md**（8 分钟）← 核心文档
   - OH 修改概述
   - eTS 特性详细分析
   - 修改目的和实现方式

3. **04_Usage_in_OH.md**（4 分钟）
   - 依赖关系详解
   - 各模块使用方式

### 路线三：开发者参考（30 分钟）

如果您需要基于此库进行开发或适配：

1. **完整阅读所有文档**
2. 重点关注：
   - **03_Build_Integration.md** - 构建配置详解
   - **05_API_Differences.md** - API 变更说明
   - **06_Security.md** - 安全注意事项

## 按角色查找

### 应用开发者

| 问题 | 答案文档 |
|------|---------|
| TypeScript 是什么？ | 01_Overview.md |
| 我的项目如何使用它？ | 04_Usage_in_OH.md |
| 有哪些 eTS 特性可用？ | 02_Patches.md |

### 工具链开发者

| 问题 | 答案文档 |
|------|---------|
| 如何构建此库？ | 03_Build_Integration.md |
| 如何集成到我的工具中？ | 04_Usage_in_OH.md |
| 有哪些 API 可用？ | 05_API_Differences.md |

### 系统维护者

| 问题 | 答案文档 |
|------|---------|
| OH 对上游做了哪些修改？ | 02_Patches.md |
| 升级上游版本要注意什么？ | 02_Patches.md（升级建议章节） |
| 安全风险有哪些？ | 06_Security.md |
| 构建配置说明 | 03_Build_Integration.md |

## 核心章节标识

以下章节被标记为核心文档，建议所有读者都应阅读：

- **02_Patches.md** ⭐⭐⭐ - **必读**
  - 说明 OH 对 TypeScript 的所有适配修改
  - 包含 eTS 特性的详细说明
  - 包含升级上游版本时的注意事项

- **04_Usage_in_OH.md** ⭐⭐ - **推荐**
  - 说明库在 OH 中的依赖关系
  - 帮助理解生态系统中的位置

## 快速参考

### 文件位置速查

| 目的 | 文件路径 |
|------|---------|
| 构建配置 | `BUILD.gn` |
| OH 修改日志 | `README.md` "Changes" 章节 |
| 组件配置 | `bundle.json` |
| 许可证 | `LICENSE` |

### 关键路径速查

| 目的 | OH 代码路径 |
|------|------------|
| 打包工具 | `//developtools/ace_ets2bundle` |
| Linter | `//arkcompiler/ets_frontend/ets2panda/linter` |
| 编译器前端 | `//arkcompiler/ets_frontend/es2panda` |

## 更新日志

本文档会随 TypeScript 库版本更新。如发现文档错误或遗漏，请联系维护者或提交 Issue。
