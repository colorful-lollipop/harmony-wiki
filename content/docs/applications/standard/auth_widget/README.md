# OpenHarmony auth_widget Wiki

## 项目概述

OpenHarmony Authentication Widget（用户认证组件）是一个系统级 UI 扩展模块，与 [useriam_user_auth_framework](https://gitee.com/openharmony/useriam_user_auth_framework) 配合工作，为用户认证流程提供统一的交互界面。

## 目录

### 快速入门
- [README](README.md) - 本 Wiki 使用指南
- [README_ZH](README_ZH.md) - 本 Wiki 中文指南

### 架构与设计
- [概览](00_Overview.md) - 项目定位、核心能力、运行环境
- [架构说明](01_Architecture.md) - 组件图、数据流、线程模型

### 接口文档
- [认证框架 API](02_UserAuth_API.md) - 与 user_auth_framework 的接口集成
- [组件 API](03_Components_API.md) - 内部组件接口

### 构建与配置
- [GN 构建配置](10_GN_Build.md) - 构建 targets 与编译产物
- [模块配置](11_Module_Config.md) - bundle.json 与 module.json 配置

### 安全与风险
- [安全风险评审](20_Security_Review.md) - 攻击面分析与安全建议

### 附录
- [常见问题](90_FAQ.md) - 构建、运行、调试问题
- [术语表](91_Glossary.md) - 关键术语解释

## 快速链接

- **官方文档**: [User Authentication Framework](https://gitee.com/openharmony/useriam_user_auth_framework)
- **构建命令**: `./build.sh --product-name rk3568 --ccache --build-target auth_widget`
- **模块配置**: [bundle.json](bundle.json)
- **源码路径**: `entry/src/main/ets/`

## 贡献指南

本 Wiki 由代码自动生成，如有疑问请参考源码。
