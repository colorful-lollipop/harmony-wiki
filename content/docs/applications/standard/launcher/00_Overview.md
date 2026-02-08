# 概览

> 项目定位、技术栈与核心能力说明

## 项目定位

**applications_launcher** 是 OpenHarmony 系统的桌面应用，作为系统人机交互的首要入口。

### 核心职责

| 功能 | 说明 |
|------|------|
| 应用图标显示 | 展示系统中已安装的应用图标 |
| 应用启动 | 点击图标启动对应应用 |
| 应用卸载 | 提供卸载功能入口 |
| 桌面布局设置 | 个性化桌面布局管理 |
| 最近任务管理 | 展示和管理最近使用的任务 |
| 桌面卡片 | 支持 Form 卡片展示 |

### 目标设备

| 设备类型 | 支持状态 | 说明 |
|----------|----------|------|
| Phone | ✅ 支持 | 手机形态 |
| Tablet | ✅ 支持 | 平板形态 |

## 技术栈

### 开发语言

| 语言 | 版本/说明 |
|------|-----------|
| ArkTS | TypeScript 的扩展方言 |
| TypeScript | 基础语言版本 |

### SDK 与框架

| 组件 | 版本/说明 |
|------|-----------|
| OpenHarmony SDK | API Version 10 |
| DevEco Studio | ≥ 3.0.0.900 |
| hvigor | 类 Gradle 构建工具 |
| Stage 模型 | 应用模型 |

### 依赖的系统 API

| 模块 | 用途 |
|------|------|
| `@ohos.app.ability.ServiceExtensionAbility` | ServiceExtension 基类 |
| `@ohos.window` | 窗口管理 |
| `@ohos.display` | 显示管理 |
| `@ohos.multimodalInput.inputConsumer` | 输入事件 |
| `@ohos.hilog` | 日志输出 |

## 架构风格

### 三层架构

```
┌─────────────────────────────────────┐
│           product/                   │  ← 业务形态层
│    (phone / pad 特定实现)            │
├─────────────────────────────────────┤
│            feature/                  │  ← 公共特性层
│  (可复用的功能组件)                  │
├─────────────────────────────────────┤
│            common/                   │  ← 公共能力层
│     (所有桌面必须依赖的基础能力)      │
└─────────────────────────────────────┘
```

### 应用模型

- **Stage 模型**：使用 `ServiceExtension` 作为入口
- **HAR 打包**：feature 模块以 HAR 形式被引用
- **Entry 打包**：product 模块打包为 HAP

## 运行环境

### 开发环境要求

| 要求 | 说明 |
|------|------|
| IDE | DevEco Studio for OpenHarmony ≥ 3.0.0.900 |
| SDK | ohos-sdk-full / mac-sdk-full |
| 系统 | Windows / macOS / Linux |

### 构建工具

| 工具 | 说明 |
|------|------|
| hvigor | 项目构建工具 |
| npm | 依赖管理 |
| ArkTS 编译器 | 编译 ArkTS 代码 |

## 关键概念

### MainAbility

Launcher 的入口组件，继承 `ServiceExtension`，负责：

- 初始化桌面上下文
- 创建桌面窗口
- 启动各功能模块
- 处理系统事件

### PreLoader 模式

各 feature 模块通过 `Pre延迟Loader` 进行加载：

- `pageDesktopPreLoader`
- `formPreLoader`
- `smartDockPreLoader`
- `appCenterPreLoader`
- `bigFolderPreLoader`
- `formPreLoader`

### 样式配置

支持多级样式配置：

| 级别 | 说明 |
|------|------|
| `LAYOUT_CONFIG_LEVEL_COMMON` | 公共默认配置 |
| `LAYOUT_CONFIG_LEVEL_FEATURE` | Feature 级配置 |
| `LAYOUT_CONFIG_LEVEL_PRODUCT` | Product 级自定义配置 |

## 相关文档

| 文档 | 链接 |
|------|------|
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构说明 | [02_Architecture.md](02_Architecture.md) |
| 构建配置 | [05_Build.md](05_Build.md) |
| OpenHarmony 文档 | [开发者官网](https://developer.harmonyos.com/cn/) |
