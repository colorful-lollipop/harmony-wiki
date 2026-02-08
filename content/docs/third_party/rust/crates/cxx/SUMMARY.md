# 阅读路线指南

本文档提供 cxx 库在 OpenHarmony 中集成的完整文档，读者可根据以下路线选择阅读。

## 快速了解（5 分钟）

如果只需要了解 cxx 库在 OH 中的定位和作用：

1. 阅读 [README.md](README.md) 前两章（库概述、核心功能）
2. 查看 [01_Overview.md](01_Overview.md) 的"核心特性"章节

## 深入理解（15 分钟）

如果需要了解 OH 对该库的完整适配：

1. [README.md](README.md) - 完整阅读
2. [01_Overview.md](01_Overview.md) - 了解原始库功能
3. [03_Build_Integration.md](03_Build_Integration.md) - 重点关注 BUILD.gn 配置
4. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看依赖关系和使用场景

## 开发者参考（按需查阅）

### 想要贡献代码或提交 PR

1. [02_Patches.md](02_Patches.md) - 了解 OH Patch 策略
2. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置详情

### 想要升级上游版本

1. [02_Patches.md](02_Patches.md) - 检查是否有未合并到上游的 Patch
2. [03_Build_Integration.md](03_Build_Integration.md) - 验证新版本的构建兼容性

### 想要添加新依赖

1. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看现有依赖模式
2. [03_Build_Integration.md](03_Build_Integration.md) - 了解依赖添加流程

## 文档结构概览

```
wiki/
├── README.md              # 库概览、OH 适配概述、文档导航
├── SUMMARY.md             # 阅读路线建议（本文档）
├── _work/
│   ├── ASSESSMENT.md      # 项目评估结果
│   ├── NOTES.md           # 分析过程记录
│   └── PLAN.md            # 任务进度
├── 01_Overview.md         # 原始库简介
├── 02_Patches.md          # Patch 详细分析
├── 03_Build_Integration.md # OH 构建适配
└── 04_Usage_in_OH.md      # 依赖关系与使用
```

## 关键术语

| 术语 | 说明 |
|-----|------|
| FFI | Foreign Function Interface，外部函数接口 |
| rlib | Rust 静态库格式 |
| 过程宏 | Rust 编译时代码生成机制 |
| Inner Kit | OH 组件对外提供的编程接口 |
| GN | OH 构建系统（Generate Ninja） |

## 相关资源

- **上游文档**：https://cxx.rs
- **上游代码**：https://github.com/dtolnay/cxx
- **OH Rust 工具链**：请参阅 Rust crates 整体文档
