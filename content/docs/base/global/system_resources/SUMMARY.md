# 文档导航

本文档为 OpenHarmony `system_resources` 模块的全站导航。

## 快速开始

| 主题 | 描述 | 预计阅读时间 |
|------|------|--------------|
| [README.md](README.md) | Wiki 使用指南 | 3 分钟 |
| [00_Overview.md](00_Overview.md) | 项目概览与定位 | 5 分钟 |
| [06_FAQ.md](06_FAQ.md) | 常见问题解答 | 5 分钟 |

## 完整文档列表

### 入门指南

1. [README.md](README.md) - Wiki 使用说明、更新方式、阅读建议
2. [00_Overview.md](00_Overview.md) - 项目定位、核心能力、运行环境、关键概念

### 结构与架构

3. [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构、模块职责、不含测试说明
4. [02_Architecture.md](02_Architecture.md) - 组件图、数据流、线程模型、关键时序

### 构建与资源

5. [03_Build_System.md](03_Build_System.md) - GN Targets、编译产物、产物映射
6. [04_Resources.md](04_Resources.md) - 系统字体、系统资源包、权限定义

### 安全与运维

7. [05_Security.md](05_Security.md) - 攻击面、信任边界、风险清单、修复建议
8. [06_FAQ.md](06_FAQ.md) - 构建、运行、调试问题与定位

## 新人阅读路线

```
Day 1: 快速入门
├── 00_Overview.md (项目定位)
├── 01_Directory_Structure.md (目录结构)
└── 06_FAQ.md (常见问题)

Day 2: 深入理解
├── 02_Architecture.md (架构设计)
├── 03_Build_System.md (构建系统)
└── 04_Resources.md (系统资源)

Day 3: 安全与实践
└── 05_Security.md (安全评审)
```

## 文档索引

### 按类型分类

| 类型 | 文档 |
|------|------|
| 入门 | README.md, 00_Overview.md, 06_FAQ.md |
| 结构 | 01_Directory_Structure.md |
| 架构 | 02_Architecture.md |
| 构建 | 03_Build_System.md |
| 资源 | 04_Resources.md |
| 安全 | 05_Security.md |

### 按功能分类

| 功能 | 文档 |
|------|------|
| 系统字体 | 04_Resources.md, 03_Build_System.md |
| 权限定义 | 04_Resources.md, 05_Security.md |
| GN 构建 | 03_Build_System.md |
| 安全评审 | 05_Security.md |

## 关键链接速查

| 主题 | 链接 |
|------|------|
| 根 README | [README.md](../README.md) |
| bundle.json | [bundle.json](../bundle.json) |
| 构建配置 | [BUILD.gn](../BUILD.gn) |
| GN 变量 | [systemres.gni](../systemres.gni) |
| 权限定义 | [module.json](systemres/main/module.json) |
| Git 仓库 | https://gitee.com/openharmony/resources |

## 版本兼容性

| Wiki 版本 | 适用 OpenHarmony |
|-----------|------------------|
| 1.0 | 4.0+ |
| 未来版本 | 待更新 |

## 贡献指南

Wiki 内容存放在 `wiki/` 目录下，采用 Markdown 格式。

**提交变更**:
```bash
git add wiki/
git commit -m "docs: update [文档名] for [变更描述]"
```

## 最后更新

- **时间**: 2026-02-06
- **生成工具**: OpenHarmony Wiki Generator
- **状态**: 初始版本
