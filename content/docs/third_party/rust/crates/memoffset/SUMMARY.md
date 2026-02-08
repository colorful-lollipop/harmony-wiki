# 阅读路线指南

本文档提供 memoffset Wiki 的阅读路线建议，帮助不同角色的读者快速找到所需信息。

## 文档结构概览

```
memoffset Wiki/
├── README.md              # 快速入门和导航
├── SUMMARY.md            # 阅读路线（当前）
├── 01_Overview.md        # 库功能简介
├── 02_Patches.md         # Patch 分析
├── 03_Build_Integration.md # OH 构建适配
├── 04_Usage_in_OH.md     # OH 使用情况
├── 05_API_Differences.md # API 差异
├── 06_Security.md        # 安全分析
└── _work/                # 工作文档
    ├── ASSESSMENT.md     # 项目评估
    ├── PLAN.md           # 任务计划
    └── NOTES.md          # 分析记录
```

## 读者专属路线

### 路线 1：快速了解（5 分钟）

适合：初次接触该库，需要快速了解基本信息

| 文档 | 阅读内容 | 预计时间 |
|------|---------|---------|
| README.md | 库概述、适配状态、关键信息 | 2 分钟 |
| 01_Overview.md | 功能简介、核心 API | 3 分钟 |

**产出**：了解 memoffset 是什么，在 OH 中起什么作用

### 路线 2：开发参考（15 分钟）

适合：开发者，需要了解如何在 OH 中使用或维护该库

| 文档 | 阅读内容 | 预计时间 |
|------|---------|---------|
| README.md | 全部内容 | 3 分钟 |
| 01_Overview.md | 全部内容 | 5 分钟 |
| 03_Build_Integration.md | 构建配置详解 | 5 分钟 |
| 04_Usage_in_OH.md | 依赖关系和使用场景 | 2 分钟 |

**产出**：理解构建配置，知道依赖关系，能进行基础维护

### 路线 3：深度分析（30 分钟）

适合：维护者或需要深入了解的开发者

| 文档 | 阅读内容 | 预计时间 |
|------|---------|---------|
| 所有文档 | 按顺序阅读 | 30 分钟 |

**产出**：完整理解该库在 OH 中的所有细节

### 路线 4：安全审计（20 分钟）

适合：安全审计人员，关注安全风险

| 文档 | 阅读内容 | 预计时间 |
|------|---------|---------|
| 06_Security.md | 安全风险分析 | 10 分钟 |
| 02_Patches.md | Patch 安全审查 | 5 分钟 |
| 05_API_Differences.md | API 安全考量 | 5 分钟 |

**产出**：了解安全风险状况和缓解措施

## 常见问题快速定位

| 问题 | 答案位置 |
|------|---------|
| 这个库是干什么的？ | 01_Overview.md |
| OH 有没有修改这个库？ | 02_Patches.md |
| 如何构建这个库？ | 03_Build_Integration.md |
| 哪些模块在用它？ | 04_Usage_in_OH.md |
| OH 添加了哪些 API？ | 05_API_Differences.md |
| 有安全风险吗？ | 06_Security.md |

## 核心信息速查

### 必须了解的关键点

1. **无需 Patch**：该库原生跨平台，无需 OH 特定修改
2. **间接使用**：主要通过 nix/rustix 间接被使用
3. **可被替代**：Rust 1.77+ 可使用标准库 `offset_of!`
4. **静态链接**：编译为 rlib 静态库

### 重要文件清单

| 文件 | 路径 | 说明 |
|------|------|-----|
| BUILD.gn | //third_party/rust/crates/memoffset/BUILD.gn | OH 构建配置 |
| Cargo.toml | third_party/rust/crates/memoffset/Cargo.toml | Rust 包配置 |
| bundle.json | third_party/rust/crates/memoffset/bundle.json | OH 组件配置 |

## 下一步

选择适合您的路线开始阅读：

- [返回 README.md](README.md)
- [开始阅读：01_Overview.md](01_Overview.md)
- [查看所有文档列表](#文档结构概览)
