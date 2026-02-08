# SystemUI Wiki

> OpenHarmony SystemUI 工程项目文档

## 项目简介

SystemUI 是 OpenHarmony 中预置的系统应用，为用户提供系统相关信息展示及交互界面，包括系统状态、系统提示、系统提醒等，例如系统时间、电量信息、通知管理等。

## 技术栈

- **开发语言**: ArkTS (TypeScript 超集)
- **UI 框架**: ArkUI (声明式 UI)
- **构建系统**: hvigor + @ohos/hvigor-ohos-plugin
- **SDK 版本**: OpenHarmony SDK 12

## 文档覆盖范围

| 文档 | 状态 | 说明 |
|------|------|------|
| [README](README.md) | ✅ | 本文档 |
| [SUMMARY](SUMMARY.md) | ✅ | 全站导航 |
| [01_Overview](01_Overview.md) | ✅ | 项目概览 |
| [02_Directory_Structure](02_Directory_Structure.md) | ✅ | 目录结构 |
| [03_Architecture](03_Architecture.md) | ✅ | 架构说明 |
| [04_API_Inner](04_API_Inner.md) | ✅ | 内部 API |
| [05_Build](05_Build.md) | ✅ | 构建指南 |
| [06_Security](06_Security.md) | ✅ | 安全评审 |

## 快速开始

### 新人阅读顺序

1. [01_Overview](01_Overview.md) - 了解项目定位和功能
2. [02_Directory_Structure](02_Directory_Structure.md) - 熟悉代码结构
3. [03_Architecture](03_Architecture.md) - 理解系统架构
4. [05_Build](05_Build.md) - 了解构建流程
5. [06_Security](06_Security.md) - 安全注意事项

### 关键概念

- **HAR (Harmony Archive)**: 共享模块，用于 features 和 product
- **HAP (Harmony Ability Package)**: 可安装的应用包
- **ServiceExtensionAbility**: 后台服务能力，用于系统 UI 组件
- **EventBus**: 内部事件总线，用于组件间通信

## 重要说明

### N-API 状态

⚠️ **本项目为纯 ArkTS/ArkUI 项目，不包含 N-API (Native API) 层。**

- 无 C/C++ 源代码
- 无 napi_ 符号调用
- 直接使用 `@ohos.*` 系统 API
- 相关文档见 [04_API_Inner](04_API_Inner.md)

### 模块类型

| 类型 | 说明 | 示例 |
|------|------|------|
| entry | 主入口模块 | `entry/phone`, `entry/pc` |
| feature | 功能模块 | `features/batterycomponent` |
| har | 共享模块 | `common`, `features/*` |

## 构建与运行

### 环境要求

- OpenHarmony SDK 12
- hvigor 构建工具
- Node.js (用于 hvigor)

### 构建命令

```bash
# 安装依赖
npm install

# 构建手机版本
hvigor --product phone --target targetName assembleHap

# 构建 PC 版本  
hvigor --product pc --target targetName assembleHap
```

详细构建文档: [05_Build](05_Build.md)

## 贡献指南

### 文档更新

1. 所有文档位于 `wiki/` 目录
2. 使用 Markdown 格式
3. 遵循现有文档风格
4. 添加代码证据（文件路径 + 符号）

### 证据要求

- 关键结论必须有代码证据
- 格式: `文件路径:行号` 或 `文件路径`
- 示例: `entry/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts:24`

## 版本信息

- **文档版本**: 1.0
- **生成时间**: 2026-02-06
- **仓库**: `/Volumes/lexar/code/d/work/oh/applications/standard/systemui`
