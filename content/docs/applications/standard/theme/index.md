# applications_theme - 主题应用

> OpenHarmony 系统预置的主题应用，提供系统主题和桌面壁纸设置能力。

## 项目简介

**applications_theme** 是 OpenHarmony 系统的预置系统应用，主要功能包括：

- ✅ **壁纸设置** - 为用户提供设置系统主题和桌面壁纸的基础能力
- ✅ **第三方支持** - 支持通过其他第三方应用设置用户自定义的主题与壁纸
- ✅ **多设备适配** - 支持手机（phone）和平板（pad）设备

## 快速定位

| 需求 | 文档 |
|------|------|
| 了解项目定位和功能 | [01_Project_Overview.md](01_Project_Overview.md) |
| 查看目录结构 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 理解系统架构 | [03_Architecture.md](03_Architecture.md) |
| 查看 API 接口 | [04_API.md](04_API.md) |
| 了解构建配置 | [06_Build.md](06_Build.md) |
| 安全风险评估 | [08_Security.md](08_Security.md) |

## 技术栈

| 类别 | 技术/版本 |
|------|----------|
| 开发语言 | ArkTS (ETS) |
| SDK 版本 | OpenHarmony SDK 9 |
| 构建系统 | hvigor (Node.js) |
| UI 框架 | ArkUI |
| 核心能力 | WallpaperExtension, Ability |

## 关键特征

### 无 N-API 暴露

本项目为纯前端 ETS 项目，未使用 C/C++ 开发，**不涉及 N-API 对外接口**。

### 轻量级架构

- 核心代码约 200 行（含注释）
- 仅依赖系统原生 API (@ohos.*)
- 模块结构简单清晰

### 安全设计

- 使用系统标准权限模型
- 遵循 OpenHarmony 安全开发规范

## 相关信息

- **仓库地址**: https://gitcode.com/openharmony/applications_theme
- **问题反馈**: OpenHarmony Issue Tracker
- **许可证**: Apache License 2.0
