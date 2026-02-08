# DLP权限管理应用 - 工程Wiki

## 简介

本Wiki是 **DLP权限管理应用 (dlp_manager)** 的工程文档，旨在帮助开发者快速理解项目架构、核心流程和安全机制。

## 项目基本信息

| 属性 | 值 |
|------|-----|
| 项目名称 | DLP权限管理应用 (dlp_manager) |
| BundleName | `com.ohos.dlpmanager` |
| 版本 | 6.1.0.318 |
| 类型 | OpenHarmony 标准系统应用 |
| 开发语言 | ArkTS (ETS) |
| 所属子系统 | applications |

## 核心功能

数据防泄漏（DLP: Data Leak Prevention）通过安全保护技术实现文档权限管理：

1. **生成DLP文件** - 对指定文件添加权限保护，创建.dlp后缀文件
2. **设置只读/编辑权限** - 给指定终端用户分配域账号权限
3. **更改DLP文件权限** - 支持解除加密或修改加密设置
4. **安全打开DLP文件** - 通过沙箱机制打开受保护文件

## 文档覆盖范围

### 已覆盖

- ✅ 项目概览与核心概念
- ✅ 目录结构与模块职责
- ✅ 架构设计（组件图/数据流/线程模型）
- ✅ Ability生命周期与交互流程
- ✅ 对外接口（N-API/Ability调用）
- ✅ 内部API与模块依赖
- ✅ GN构建系统与编译产物
- ✅ 安全风险分析与修复建议
- ✅ 调试指南与常见问题

### 未覆盖

- ❌ 测试代码（遵循约束排除）
- ❌ 第三方库内部实现
- ❌ DLP权限服务（security_dlp_permission_service）实现细节

## 如何更新本文档

本文档通过代码分析自动生成。如需更新：

1. 修改 `_work/NOTES.md` 记录新发现
2. 更新相应Markdown文件
3. 同步更新 `SUMMARY.md` 导航
4. 修改本文件的生成时间

## 生成信息

- **生成时间**: 2026-02-05
- **代码版本**: 基于仓库HEAD分析
- **源文件统计**: 105个非测试源文件
- **分析工具**: AST-Grep + 静态代码分析

## 快速导航

- [项目概览](00_Overview.md) - 了解项目定位和核心能力
- [架构设计](10_Architecture.md) - 理解系统架构和数据流
- [目录结构](20_Directory_Structure.md) - 查看代码组织方式
- [对外接口](30_Public_API.md) - 查看Ability接口和调用方式
- [安全风险](60_Security_Analysis.md) - 查看安全评审结果

## 相关仓库

- [DLP权限管理应用](https://gitcode.com/openharmony/applications_dlp_manager) - 本仓库
- [DLP权限管理服务](https://gitcode.com/openharmony/security_dlp_permission_service) - 系统服务

---

*本Wiki遵循 Apache License 2.0 开源协议*
