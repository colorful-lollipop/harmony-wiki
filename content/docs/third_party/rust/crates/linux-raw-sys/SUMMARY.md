# 文档目录与阅读路线

## 文档结构

```
wiki/
├── README.md                    # 库概览、导航、关键特性
├── SUMMARY.md                   # 本文档，阅读路线建议
├── _work/
│   ├── ASSESSMENT.md           # 项目评估结果（必读）
│   ├── NOTES.md                # 分析过程记录（可选）
│   └── PLAN.md                 # 任务进度跟踪（内部使用）
├── 01_Overview.md              # 原始库功能介绍
├── 02_Patches.md               # Patch 详细分析
├── 03_Build_Integration.md     # 构建适配说明
├── 04_Usage_in_OH.md           # OH 使用情况与依赖关系
├── 05_API_Differences.md      # API 差异分析（不适用）
└── 06_Security.md              # 安全风险分析
```

## 阅读路线建议

### 路线一：快速概览（5分钟）

适合：时间有限，只需了解基本情况的读者

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 1 | README.md | 第一、二、三节 |
| 2 | SUMMARY.md | 本路线说明 |

**预计时间**：5 分钟
**产出**：了解该库在 OH 中的定位和适配特点

### 路线二：技术深入（15-30分钟）

适合：需要深入了解适配细节的技术人员

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 1 | README.md | 全部内容 |
| 2 | 01_Overview.md | 全部内容 |
| 3 | 02_Patches.md | 重点关注结论部分 |
| 4 | 03_Build_Integration.md | 重点关注适配策略 |
| 5 | 04_Usage_in_OH.md | 依赖关系图 |

**预计时间**：15-30 分钟
**产出**：理解该库在 OH 生态中的位置和集成方式

### 路线三：完整分析（30-60分钟）

适合：需要全面了解的维护者和贡献者

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 1 | README.md | 全部内容 |
| 2 | 01_Overview.md | 全部内容 |
| 3 | 02_Patches.md | 全部内容 |
| 4 | 03_Build_Integration.md | 全部内容 |
| 5 | 04_Usage_in_OH.md | 全部内容 |
| 6 | 06_Security.md | 全部内容 |
| 7 | _work/ASSESSMENT.md | 评估结论和建议 |
| 8 | _work/NOTES.md | 分析过程（可选） |

**预计时间**：30-60 分钟
**产出**：全面理解该库的 OH 适配情况，可进行维护和贡献

## 按角色分类

### 构建系统工程师

| 优先级 | 文档 | 关注点 |
|--------|------|--------|
| 高 | 03_Build_Integration.md | BUILD.gn 配置、编译选项 |
| 中 | 04_Usage_in_OH.md | 依赖关系 |
| 中 | README.md | 功能模块说明 |

### 应用开发者

| 优先级 | 文档 | 关注点 |
|--------|------|--------|
| 高 | 04_Usage_in_OH.md | 使用场景、依赖图 |
| 中 | 01_Overview.md | 功能介绍 |
| 低 | 03_Build_Integration.md | 构建细节（可选） |

### 安全工程师

| 优先级 | 文档 | 关注点 |
|--------|------|--------|
| 高 | 06_Security.md | CVE 情况、风险评估 |
| 中 | 02_Patches.md | Patch 引入的风险 |
| 中 | 04_Usage_in_OH.md | 使用场景 |

### 架构师

| 优先级 | 文档 | 关注点 |
|--------|------|--------|
| 高 | 04_Usage_in_OH.md | 依赖关系、在 OH 中的位置 |
| 高 | _work/ASSESSMENT.md | 评估结论 |
| 中 | 03_Build_Integration.md | 构建适配策略 |

## 文档变更日志

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 1.0 | 2024年 | 初始版本 |

## 相关链接

- **上游项目**：[linux-raw-sys](https://github.com/sunfishcode/linux-raw-sys)
- **相关项目**：[rustix](https://github.com/bytecodealliance/rustix)
- **OH Rust 生态**：[third_party/rust/crates](../)
