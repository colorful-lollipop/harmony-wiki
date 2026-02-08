# 文档导航 (SUMMARY)

> bundle_tool Wiki 全站索引 + 新人阅读路线

## 推荐阅读顺序

### 路线 A: 快速入门 (5分钟)

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [02_Command_Reference.md](./02_Command_Reference.md) - 命令速查

### 路线 B: 开发者深入 (30分钟)

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [01_Architecture.md](./01_Architecture.md) - 架构设计
3. [02_Command_Reference.md](./02_Command_Reference.md) - 命令详解
4. [03_Inner_API.md](./03_Inner_API.md) - 内部接口
5. [04_Build.md](./04_Build.md) - 构建系统

### 路线 C: 安全评审 (15分钟)

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [05_Security.md](./05_Security.md) - 安全风险分析

---

## 文档索引

### 快速入口

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](./README.md) | 本文档 | 必读 |
| [SUMMARY.md](./SUMMARY.md) | 导航索引 | 必读 |
| [00_Overview.md](./00_Overview.md) | 项目概览 | 高 |

### 架构与设计

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [01_Architecture.md](./01_Architecture.md) | 系统架构 | 高 |
| [03_Inner_API.md](./03_Inner_API.md) | 内部 API | 中 |

### 使用指南

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [02_Command_Reference.md](./02_Command_Reference.md) | 命令参考 | **极高** |
| [06_Troubleshooting.md](./06_Troubleshooting.md) | 问题定位 | 中 |

### 构建与部署

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [04_Build.md](./04_Build.md) | GN 构建系统 | 高 |

### 安全

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [05_Security.md](./05_Security.md) | 安全风险评审 | 高 |

---

## 命令速查表

| 命令 | 功能 | 章节 |
|------|------|------|
| `bm help` | 显示帮助 | 2.1 |
| `bm install` | 安装 HAP/HSP | 2.2 |
| `bm uninstall` | 卸载应用 | 2.3 |
| `bm dump` | 查询信息 | 2.4 |
| `bm clean` | 清理数据 | 2.5 |
| `bm enable` | 使能应用 | 2.6 |
| `bm disable` | 禁用应用 | 2.7 |
| `bm get` | 获取 UDID | 2.8 |
| `bm quickfix` | 快速修复 | 2.9 |
| `bm compile` | AOT 编译 | 2.10 |
| `bm copy-ap` | 拷贝 AP 文件 | 2.11 |
| `bm dump-overlay` | 查询 Overlay | 2.12 |
| `bm dump-target-overlay` | 查询目标 Overlay | 2.13 |
| `bm dump-shared` | 查询 HSP | 2.14 |
| `bm dump-dependencies` | 查询依赖 | 2.15 |
| `bm install-plugin` | 安装插件 | 2.16 |
| `bm uninstall-plugin` | 卸载插件 | 2.17 |

---

## 术语表

| 术语 | 含义 |
|------|------|
| Bundle | 应用包 (HAP/HSP) |
| HAP | Harmony Ability Package |
| HSP | Harmony Shared Package |
| SA | System Ability |
| UDID | Unique Device ID |
| AOT | Ahead-Of-Time 编译 |

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2024-XX-XX | 初始版本 |
