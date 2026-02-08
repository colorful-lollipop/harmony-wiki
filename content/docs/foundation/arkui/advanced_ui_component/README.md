# OpenHarmony ArkUI 高级 UI 组件库 (advanced_ui_component)

## 项目概述

**advanced_ui_component** 是 OpenHarmony ArkUI 的高级 UI 组件库，提供基于使用场景设计的高效 UI 组合。该库采用 ArkTS 语言开发，接口封闭、风格一致、开箱即用。

| 属性 | 值 |
|------|-----|
| **项目名称** | advanced_ui_component |
| **版本** | 1.0.0 |
| **子系统** | arkui |
| **目标系统** | standard (标准版) |
| **ROM** | 5120KB |
| **RAM** | 10240KB |
| **编程语言** | ArkTS, C++ |
| **框架** | ArkUI |

**证据来源**: `bundle.json:2-18`

---

## 文档覆盖范围

本 Wiki 涵盖以下内容：

### 核心文档

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [README](README.md) | 项目整体说明与贡献指南 | 所有读者 |
| [SUMMARY](SUMMARY.md) | 全站导航与阅读路线 | 所有读者 |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境 | 新人学习者 |
| [02_Architecture](02_Architecture.md) | 架构设计、组件图、数据流 | 架构师、开发者 |
| [03_Components](03_Components.md) | 组件清单与 API 详细说明 | 开发者 |
| [04_Build](04_Build.md) | GN 构建配置与编译产物 | 构建工程师 |
| [05_Usage](05_Usage.md) | 组件使用说明与示例 | 应用开发者 |
| [06_Security](06_Security.md) | 安全风险评审 | 安全研究员 |
| [07_Troubleshooting](07_Troubleshooting.md) | 常见问题与调试指南 | 开发者 |

### 工作文档

| 文档 | 说明 | 维护者 |
|------|------|--------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 | 维护者 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据汇总 | 维护者 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 | 维护者 |

---

## 快速开始

### 环境要求

- OpenHarmony SDK (API 10+)
- GN 构建系统
- Ninja 构建工具
- C++17 编译器

### 编译命令

```bash
# 完整构建
hb set
hb build -f

# 单独构建本模块
hb build //foundation/arkui/advanced_ui_component:advanced_ui_component
```

---

## 组件清单

| 组件名 | 类型 | 说明 |
|--------|------|------|
| AtomServiceNavigation | 原子化服务 | 导航组件 |
| AtomServiceSearch | 原子化服务 | 搜索组件 |
| AtomServiceTabs | 原子化服务 | 标签页组件 |
| AtomServiceWeb | 原子化服务 | Web 组件 |
| CustomAppBar | 原子化服务 | 自定义应用栏 |
| CustomAppBarMenuBar | 原子化服务 | 菜单栏 |
| FullScreenLaunchComponent | 启动组件 | 全屏启动 |
| HalfScreenLaunchComponent | 启动组件 | 半屏启动 |
| InnerFullScreenLaunchComponent | 启动组件 | 内部全屏启动 |
| InterstitialDialogAction | 弹窗组件 | 插屏弹窗 |
| NavPushPathHelper | 导航助手 | HSP 路径管理 |

**详细说明**: 参见 [03_Components](03_Components.md)

---

## 技术架构

```
┌─────────────────────────────────────────────────────────┐
│                    应用层 (ArkTS)                        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐   │
│  │Navigation│ │ Search │ │  Tabs   │ │  WebView    │   │
│  └────┬────┘ └────┬────┘ └────┬────┘ └──────┬──────┘   │
└───────┼────────────┼────────────┼─────────────┼──────────┘
        │            │            │             │
        ▼            ▼            ▼             ▼
┌─────────────────────────────────────────────────────────┐
│                 N-API 绑定层 (C++)                       │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐   │
│  │  Module │ │  Module │ │  Module │ │  Module     │   │
│  │Register │ │Register │ │Register │ │Register     │   │
│  └────┬────┘ └────┬────┘ └────┬────┘ └──────┬──────┘   │
└───────┼────────────┼────────────┼─────────────┼──────────┘
        │            │            │             │
        ▼            ▼            ▼             ▼
┌─────────────────────────────────────────────────────────┐
│                 系统 API 层                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐   │
│  │ Ability │ │BundleMgr│ │ Window  │ │ API Policy │   │
│  │  Kit    │ │         │ │ Manager │ │             │   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**详细架构**: 参见 [02_Architecture](02_Architecture.md)

---

## 安全特性

- **URL 策略校验**: AtomServiceWeb 使用系统级 API 策略库进行 URL 校验
- **HSP 静默安装**: NavPushPathHelper 提供安全的 HSP 包管理
- **权限控制**: 基于 OpenHarmony 权限系统

**详细安全分析**: 参见 [06_Security](06_Security.md)

---

## 贡献指南

1. 遵循 OpenHarmony 代码规范
2. 所有 API 必须有完整的 TypeScript 类型定义
3. N-API 绑定需通过安全审查
4. 提交前运行静态检查工具

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2024-xx-xx | 1.0.0 | 初始版本 |

---

## 许可证

Apache License 2.0

**证据来源**: `LICENSE` 文件